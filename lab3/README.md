# Lab 3: MCTS & stochastic games
## Exercise 1
### Introduction
Tree nodes are implemented in the class `players.basicMCTS.BasicTreeNode` (i.e. the `BasicTreeNode` class in the directory "src/main/java/players/basicMCTS"). This class implements the different functions of MCTS required to be done from a tree node, namely:

- Tree policy
- Rollout/Monte Carlo simulation
- Expansion
- Backpropagation

The `mctsSearch` method of this class runs the main loop of the MCTS algorithm. This method in turn uses three methods to handle the four aforementioned steps of the algorithm. These methods are:

- `treePolicy`: Performs selection (using UCB1[^1]) & expansion
- `rollOut`: Performs the Monte Carlo simulation
- `backUp`: Performs the backpropagation

### Questions
Explore the three aforementioned methods and answer the following:

1. Explain the different lines of code in the UCB1 calculation
2. How does selection stop & move on to expansion?
3. How is an action chosen to add a new node (expansion step)?
4. How are actions chosen for rollout & when does a rollout terminate?
5. How is backpropagation done? _To elaborate, how is the final state of a rollout evaluated & how are each node's statistics then updated?_

### Response for question 1
For reference, here is the code for the `ucb` method of the `BasicTreeNode` class (this method calculates the UCB1 value):

```
private AbstractAction ucb() {
  // Find child with highest UCB value, maximising for ourselves and minimizing for opponent
  AbstractAction bestAction = null;
  double bestValue = -Double.MAX_VALUE;

  for (AbstractAction action : children.keySet()) {
      BasicTreeNode child = children.get(action);
      if (child == null)
          throw new AssertionError("Should not be here");
      else if (bestAction == null)
          bestAction = action;

      // Find child value
      double hvVal = child.totValue;
      double childValue = hvVal / (child.nVisits + player.params.epsilon);

      // default to standard UCB
      double explorationTerm = player.params.K * Math.sqrt(Math.log(this.nVisits + 1) / (child.nVisits + player.params.epsilon));
      // unless we are using a variant

      // Find 'UCB' value
      // If 'we' are taking a turn we use classic UCB
      // If it is an opponent's turn, then we assume they are trying to minimise our score (with exploration)
      boolean iAmMoving = state.getCurrentPlayer() == player.getPlayerID();
      double uctValue = iAmMoving ? childValue : -childValue;
      uctValue += explorationTerm;

      // Apply small noise to break ties randomly
      uctValue = noise(uctValue, player.params.epsilon, player.rnd.nextDouble());

      // Assign value
      if (uctValue > bestValue) {
          bestAction = action;
          bestValue = uctValue;
      }
  }

  if (bestAction == null)
      throw new AssertionError("We have a null value in UCT : shouldn't really happen!");

  root.fmCallsCount++;  // log one iteration complete
  return bestAction;
}
```

#### Main loop of the `ucb` method
In the code of the `ucb` method,  the current state is the `BasicTreeNode` object that calls this method; remember that this method is a method of the `BasicTreeNode` class. The currently considered action from the current state is given by the `action` variable defined in the initial line of the main for loop of the method. We consider the actions from the current state iteratively (one by one) using the for loop, calculating the UCB1 value for each of them and keeping track of which action is best (i.e. has the highest UCB1 value). The `bestAction` variable keeps track of the best action so far, and the `bestValue` variable keeps track of the highest UCB1 value so far (which should naturally be of the best action; these variables must be updated together).

**NOTE**: A "child" is technically a node, but since every child node is only reachable via a particular action from the current node, a "child" of the current node is also practically equivalent to a particular action from the current node. Thus, in exploring and evaluating the children of a node, we are evaluating essentially exploring and evaluating the actions possible from the current node being considered.

#### The minimax conceptual framework
When dealing with two-player adversarial zero-sum games, as the number of iterations tends to infinity, the MCTS search tree converges to the minimax tree. Minimaxing is an approach to two-player zero-sum adversarial games wherein we abstract, simplify and represent the players' goals as one maximizing a measure opposed to the other minimizing the same measure; i.e. a two-player adversarial game becomes a contest between a maximizer and a minimizer of a certain measure. The move's of each player alternate with each level of the search tree; hence, the decision made at a particular level of the tree depends on which player's turn it is.

In the implementation we see, the MCTS agent aims to maximize its UCB1 function as opposed to the other player aiming to minimize the same. We shall see the exact details below.

#### UCB1 calculation
The UCB1 value is calculated for a given state $s$ and given action $a$ as:

$UCB1(s,a)$<br>
$=ExploitationTerm + ExplorationTerm$<br>
$=Q(s,a)+K \sqrt{\ln(\frac{N(s)+1}{N(s,a)})}$

where

- $Q(s,a)$ is the reward function returning the reward of action $a$ from state $s$
- $N(s)$ is the number of times state $s$ has been visited
- $N(s,a)$ is the number of times action $a$ has been taken from state $s$
- $K$ is the exploration constant

The exploitation term shifts the value toward known high-reward actions, whereas the exploration term shifts the value toward less explored actions (_that could have greater potential reward; we need to explore more to know more_).  In the equation above as well as in the class `players.basicMCTS.BasicMCTSParams`, we see the parameter $K$; this is -- as stated before -- the exploration constant. It is a multiplier in the exploration term and determines the weightage given to the exploration term.

In the `ucb` method, the UCB1 value is calculated for each action of the current state; these actions are considered iteratively in the main for loop of the method. The exploration term is calculated similarly to the aforementioned formula, where:

- `this.nVisits` is $N(s)$
- `child.nVisits` is $N(s,a)$

**NOTE 1**: One small change from our formula is that the code adds a small constant `players.params.epsilon` to $N(s,a)$; I was unable to understand its purpose.

**NOTE 2**: **Representing** $N(s,a)$ **as** `child.nVisits`: The `child` variable defined in the `ucb` method denotes a particular child node of the current node, and nodes represent states not actions. However, since every child node is only reachable via a particular action from the current node, the number of times the child node was visited is the same as the number of times the particular action was taken from the current node.

The **_exploitation term_**, i.e. the child's value (practically also the action's value) is given by the following lines:

```
// Find child value
double hvVal = child.totValue;
double childValue = hvVal / (child.nVisits + player.params.epsilon);
```

While I was unable to decipher what `hvVal` meant as such, it is clear that the child's exploitation value is inversely proportionate to the number of times it was visited. To see why this is done, consider this: if a child has been visited more times yet its `hvVal` remains unchanged, we may have hit a plateau or a limit in child's measure of goodness, thus indicating that the child requires less further exploration. Again, I was unable to understand the purpose of the small constant parameter `player.params.epsilon`. 

The **_exploration term_** term is given by the following line:

```
double explorationTerm = player.params.K * Math.sqrt(Math.log(this.nVisits + 1) / (child.nVisits + player.params.epsilon));
```
Most of the calculation matches the formula given at the start of this subsection, with the addition of the small constant `player.params.epsilon` in the denominator (as always, I was unable to understand its purpose). 

The way in which the exploitation and exploration terms are added depends on which player's turn is being considered (since to simulate a game, the agent needs to consider what its opponent would do as well). If the opponent's turn is being considered, the UCB1 terms are added so as to minimize the total, whereas if the given agent's turn is being considered, the UCB1 terms are added so as to maximize the total. This process is given in the code snippet below:

```
// Find 'UCB' value
// If 'we' are taking a turn we use classic UCB
// If it is an opponent's turn, then we assume they are trying to minimise our score (with exploration)
boolean iAmMoving = state.getCurrentPlayer() == player.getPlayerID();
double uctValue = iAmMoving ? childValue : -childValue;
uctValue += explorationTerm;
```

To elaborate, if it is the opponent's turn, the UCB1 value is minimized by subtracting the exploitation term, i.e. the child's value `childValue` (assumed to be positive) from the exploration term `explorationTerm`. If it is the given agent's turn, these terms are added instead. Clearly, the `state.getCurrentPlayer() == player.getPlayerID()` return value (assigned to the variable `iAmMoving`) is a Boolean value that informs whether the given agent is playing at a certain level in the tree or not.
