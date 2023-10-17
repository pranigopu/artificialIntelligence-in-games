
# Lab 3: MCTS & stochastic games
Below are the exercises and my responses to them. I aim to only included details (regarding implementation, core concepts, etc.) that I regarded as necessary to get a functional understanding of:

- The conceptual aims of the code
- The logic of the the code & its connection to the broader algorithm
- The flow of the logic, including how the broader algorithm's steps are implemented

No more details, and no less.

## Exercise 1
### Introduction
Tree nodes are implemented in the class `players.basicMCTS.BasicTreeNode` (i.e. the `BasicTreeNode` class in the directory "src/main/java/players/basicMCTS"). This class implements the different functions of MCTS required to be done from a tree node, namely:

- Tree policy
- Rollout/Monte Carlo simulation
- Expansion
- Backpropagation

The `mctsSearch` method of this class runs the main loop of the MCTS algorithm. This method in turn uses three methods to handle the four aforementioned steps of the algorithm. These methods are:

- `treePolicy`: Performs selection (using UCB1 (discussed later)) & expansion
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
The UCB1 value is generally calculated for a given state $s$ and given action $a$ as:

$UCB1(s,a)$<br>
$=ExploitationTerm + ExplorationTerm$<br>
$=Q(s,a)+K \sqrt{\ln(\frac{N(s)+1}{N(s,a)})}$

where

- $Q(s,a)$ is the exploitation term based on the reward of action $a$ from state $s$
- $N(s)$ is the number of times state $s$ has been visited
- $N(s,a)$ is the number of times action $a$ has been taken from state $s$
- $K$ is the exploration constant

**SIDE NOTE**: _The simplest exploitation term is the reward function itself (that returns the reward of a given action_ $a$ _from the given state_ $s$_). However, we could make the exploitation term use the reward function in different ways (as we shall see in the implementation in TAG as well)_.

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

While I was unable to decipher what `hvVal` meant as such, the value it was assigned to, i.e. `child.totValue`, is the "total value" statistic of the `child` object of the `BasicTreeNode` class, which (as we shall see later) is the total reward assigned to the node. We see that **_the exploitation term here is not given by the reward statistic of the child alone, but also based on the number of times the child was visited_** (this is different from the basic formulation of UCB1 we gave above).

It is also clear that the child's exploitation value is inversely proportionate to the number of times it was visited. To see why this is done, consider this: if a child has been visited more times yet its `hvVal` remains unchanged, we may have hit a plateau or a limit in child's measure of goodness, thus indicating that the child requires less further exploration. Again, I was unable to understand the purpose of the small constant parameter `player.params.epsilon`. 

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

### Response to question 2
For reference, here is the code for the `treePolicy` method (defined within the `BasicTreeNode` class) that, as the name suggests, enforces the tree policy, i.e. decides how the next node to be expanded must be selected. In this implementation, this function also performs the expansion step:

```
private BasicTreeNode treePolicy() {  
  
	BasicTreeNode cur = this;  

	// Keep iterating while the state reached is not terminal and the depth of the tree is not exceeded  
	while (cur.state.isNotTerminal() && cur.depth < player.params.maxTreeDepth) {  
	    if (!cur.unexpandedActions().isEmpty()) {  
	        // We have an unexpanded action  
	cur = cur.expand();  
	        return cur;  
	    } else {  
	        // Move to next child given by UCT function  
	AbstractAction actionChosen = cur.ucb();  
	        cur = cur.children.get(actionChosen);  
	    }  
	}  

	return cur;  
}
```
 As can be seen in the while loop, the selection step stops when the current node being considered is unexpanded. How is this node selected? To begin with, the current node is the the `BasicTreeNode` object that calls this method. This is seen here:

```
BasicTreeNode cur = this;  
```

 `cur` refers to the current node being considered and `this` refers to the `BasicTreeNode` object that calls this method. 

___

**_Small deviation from the main question_**...

**NOTE: What** `cur` **is initialized to in practice**: In practice, the object calling the `treePolicy`  method is the object that calls the implementation of the MCTS's main loop, i.e. the `mctsSearch` method (also defined within the `BasicTreeNode` class). This is clear when we look at  the code of `mctsSearch`, wherein we see these lines:

**_Viewing a code snippet of the_** `mctsSearch` **_method_**...

```
// Selection + expansion: navigate tree until a node not fully expanded is found, add a new node to the tree  
BasicTreeNode selected = treePolicy();
```

Here, we see that the object running the MCTS search also calls upon the `treePolicy` method to select nodes; thus, the `treePolicy` method begins from the node denoted by this object. In theory, this node can be any `BasicTreeNode` object, but in practice, it is generally the root node of the current MCTS (i.e. the tree search performed in the current move of the player) that alone calls upon the `mctsSearch`. This can be confirmed by checking the calls of `mctsSearch` in the project files of TAG.

**SIDE NOTE: The structure of a** `BasicTreeNode` **object**: The `BasicTreeNode` class is a way to encapsulate a node, and naturally, it contains a reference to its parent node and means to obtain its children. An object of this class does represent a node but not necessarily the root node; every object of this class contains a reference to the actual root of the current MCTS search tree in the `BasicTreeNode root` parameter.

___

**_Back to the main question_**...

The while loop of the `treePolicy` method does the following:

- If the current node is unexpanded, it expands it, i.e. generates one child & returns this child
- If the current node is expanded, one of its children is selected based on the which has the maximum UCB1 value
	- The UCB1 value is calculated in the `.ucb` method
	- The particular child is retrieved using `cur.children.get(actionChosen)`
	- The loop continues with the selected child as the current node `cur`

**RECALL NOTE**: _A child represents a node, but since it is obtainable from a particular action from the given state, it can also be used to represent the particular action (i.e. the particular branch). But technically, as such, it is a node_.

**CONCEPTUAL NOTE**: _A node is expanded if all its children have been visited/generated and unexpanded if even one of its children has not been visited/generated in the search tree_.

**TECHNICAL NOTE**: _The potential children of a node are known based on the available actions possible from this node, with the forward model enabling us to see what states these actions would lead to. However, to create or generate a child is to actually create the node object representing it, which is only done in the expansion stage. In the TAG framework's_  `BasicTreeNode` _class, the children of a node (created or not) are accessible via the parameter_ `Map<AbstractAction, BasicTreeNode> children = new HashMap<>();`.

**_Thus, the selection step stops as soon as we select an unexpanded node_**. This makes sense, because if a node is unexpanded, we have actions we have not even considered; thus, before further evaluation or expansion of the other actions, we must first at least initially evaluate all the actions of a given node.

### Response to question 3
To see how an action is selected in the expansion stage to add a new node, we must look at the `.expand` method of the `BasicTreeNode` class, which actually performs this step:

```
private BasicTreeNode expand() {  
	// Find random child not already created  
	Random r = new Random(player.params.getRandomSeed());  
	// pick a random unchosen action  
	List<AbstractAction> notChosen = unexpandedActions();  
	AbstractAction chosen = notChosen.get(r.nextInt(notChosen.size()));  

	// copy the current state and advance it using the chosen action  
	// we first copy the action so that the one stored in the node will not have any state changes
	AbstractGameState nextState = state.copy();  
	advance(nextState, chosen.copy());  

	// then instantiate a new node  
	BasicTreeNode tn = new BasicTreeNode(player, this, nextState, rnd);  
	children.put(chosen, tn);  
	return tn;  
}
```

From the list of unexpanded actions of the current node (these unexpanded actions being assigned to the variable `notChosen`), an action is chosen at random and assigned to the `chosen` variable:

```
AbstractAction chosen = notChosen.get(r.nextInt(notChosen.size()));
```

The identifier `state` in the line `AbstractGameState nextState = state.copy();` refers to the `state` parameter of the `BasicTreeNode` class, defined as:

**_Viewing some other code of the_** `BasicTreeSearch` **_class_**...

```
// State in this node (closed loop)  
private AbstractGameState state;
```

**TECHNICAL NOTE 1**: _In the_ `expand` _method, we make a deep copy of the state_ **_(every_** `.copy` **_method of any class in TAG creates a deep copy of the calling object)_** _and the action (of the given node, represented by its children) because the_ `advance` _function used to get the next state (given the chosen action) would actually alter the state variable itself as well as the actions available from this state (i.e. the children possible from this given state); we want to preserve the original parameters, as they belong to the main_ `BasicTreeNode` _object that is calling this_ `expand` _method_.

**TECHNICAL NOTE 2**: _The line_ `BasicTreeNode tn = new BasicTreeNode(player, this, nextState, rnd);` _creates the child node (denoted by the identifier_ `tn` _here, with_ `this` _(i.e. the current node) given as its_ `parent` _parameter value and_ `nextState` _given as its_ `state` _parameter value_.

Using the line...

```
advance(nextState, chosen.copy());
```

... we obtain the next state that corresponds to the action chosen `chosen` from the current state. **_In short, we see that the action from the current state is selected at random in the expansion step_**. _Note that just by creating this next node reached (representing the next state), we are also adding it to the search tree (since we give its parent as the current state, thus keeping preserving tree-hierarchy)_.

___

**_Deviating & going deeper into some the code_**...

Here, we shall see the nature and function of the following parameters and methods used in the `expand` method given above:

- The `children` parameter of the `BasicTreeNode` class
- The `unexpandedActions` method of the `BasicTreeNode` class
- The `unexpandedActions` method of the `BasicTreeNode` class
- The `children.put` method used in the `expand function`

The children of a node and by extension its available actions are represented by the `children` parameter of the `BasicTreeNode` class, defined as:

**_Viewing the definition of the_** `children` **_parameter of the_** `BasicTreeNode` **_class_**...
```
// Children of this node  
Map<AbstractAction, BasicTreeNode> children = new HashMap<>();
```

We see that `children` is in fact a hash map (more concretely, an object of class `HashMap`) whose keys are `AbstractAction` objects (_denoting available actions_) and whose values (to which these actions relate to one-to-one) are `BasicTreeNode` objects (_representing the states corresponding to each action_). Clearly, the `children` parameter is a hash map relating each available action from the current state (represented by the current node) to the state it will reach. Initially, when the current node is unexplored, the values of the hash map are empty, i.e. each key relates to a `null` value; a `null` value for a key (i.e. action) means that the respective action has not been expanded yet. Consequently, once a value (i.e. a node) is added for a key, we will know that the respective action _has_ been expanded. This is how we check for unexpanded actions.

**_Viewing the definition of the_** `unexpandedActions` **_method of the_** `BasicTreeNode` **_class_**...

```
private List<AbstractAction> unexpandedActions() {
	return children.keySet().stream().filter(a -> children.get(a) == null).collect(toList());  
}
```

Without going into every detail, we can see that the `unexpandedActions` method returns the list of `AbstractAction` objects representing the unexpanded actions (thus by extension the unexpanded children) of the `BasicTreeNode` object for which this method applies. Note that the `children.get` method is a utility function for hash maps that returns the value for the key `a` (remember that the `children` is a hash map); in our case, the key is a particular action and the value is either `null` (meaning the action has not been explored) or a particular node object (meaning that the action has been explored).


**_Viewing the definition of the_** `advance` **_method of the_** `BasicTreeNode` **_class_**...

```
private void advance(AbstractGameState gs, AbstractAction act) {  
	player.getForwardModel().next(gs, act);  
	root.fmCallsCount++;  
}
```

Without going into every detail, we see that this method uses the forward model to obtain the next state (represented by a node in turn represented by a`BasicTreeNode` object) based on a particular action `act` action applied to the current state `gs` (stands for "game state"). The forward model method used updates `gs` and `act`, wherein `gs` becomes the next node reached (_I am not sure how or why_ `act` _is updated;_ `AbstractAction` _objects can be updated and each is unique (and can serve as a unique key) even if they have identical parameter values_).

**SIDE NOTE**: _The_ `player` _identifier that calls the_ `getForwardModel` _method is in fact the_ `player` _parameter of the_ `BasicTreeNode` _object in question. This parameter is defined as_...

```
// Parameters guiding the search  
private BasicMCTSPlayer player;
```

... _and, as the comment says, it contains all the parameters needed to guide (or control) the search_.

**_The_** `children.put` **_method used in the_** `expand` **_method_**...<br>
`.put` used here is a utility method defined for hash maps (remember that `children` is a hash map), with the syntax `myHashMap.put(key, value)`. In our case, we are assigning the value of the key `chosen` (denoting a certain action from the current node) as `tn` defined in the `expand` function as `BasicTreeNode tn = new BasicTreeNode(player, this, nextState, rnd);`, i.e. the node reachable by the given action from the current node. In this way, we are marking the action denoted by `chosen` as expanded.

### Response to question 4
The rollout stage is carried out by the `rollout` method of the `BasicTreeNode` class, and this method in turn uses the `finishRollout`

**_The main_** `rollout method`:

```
/**  
* Perform a Monte Carlo rollout from this node. * * @return - value of rollout.  
*/private double rollOut() {  
	int rolloutDepth = 0; // counting from end of tree  
	 // If rollouts are enabled, select actions for the rollout in line with the rollout policy
	AbstractGameState rolloutState = state.copy();  
	if (player.params.rolloutLength > 0) {
		while (!finishRollout(rolloutState, rolloutDepth)) {
			AbstractAction next = randomPlayer.getAction(rolloutState, randomPlayer.getForwardModel().computeAvailableActions(rolloutState, randomPlayer.parameters.actionSpace));
			advance(rolloutState, next);
			rolloutDepth++;
			}
		}  
	// Evaluate final state and return normalised score  
	double value = player.params.getHeuristic().evaluateState(rolloutState, player.getPlayerID());  
	if (Double.isNaN(value))  
	  throw new AssertionError("Illegal heuristic value - should be a number");  
	return value;  
}  
```

**_The_** `finishRollout` **_method_**:

```
/**  
* Checks if rollout is finished. Rollouts end on maximum length, or if game ended. * * @param rollerState - current state  
* @param depth - current depth  
* @return - true if rollout finished, false otherwise  
*/
private boolean finishRollout(AbstractGameState rollerState, int depth) {  
	if (depth >= player.params.rolloutLength)  
		return true;  
	 
	 // End of game  
	return !rollerState.isNotTerminal();  
}
```

In the `rollout` method, we can see that the opponent model used in a random agent, and we in fact use this random agent to generate a random sequence of actions (which simulate both the basic MCTS agent's and the random agent's actions) to simulate a possible pathway of the game from the current game state. **_Hence, actions in rollout are selected randomly_**.

The `rollout` method keeps track of the rollout depth (i.e. the depth of simulated pathway), and loops rollout steps in a while loop with the terminating condition given by the return value of the `finishRollout` method. `finishRollout` returns true (i.e. a sign to continue the rollout) if the state at the end of the rollout so far is non-terminal AND if the rollout depth limit has not been reached; if either of these conditions is not met, it returns false, indicating that the rollout must end. This limit is given by the parameter `rolloutLength` of the `player` parameter of the `BasicTreeNode` (_this parameter is an object of the class_ `BasicMCTSPlayer` _and contains the parameters necessary to guide search_).

### Response to question 5
The backpropagation step is implemented using the `backUp` method of the `BasicTreeNode` class:

```
/**  
* Back up the value of the child through all parents. Increase number of visits and total value. * * @param result - value of rollout to backup  
 */private void backUp(double result) {  
	BasicTreeNode n = this;  
	while (n != null) {  
	    n.nVisits++;  
	    n.totValue += result;  
	    n = n.parent;  
	}  
}
```

For context, here is the main part of the loop of the `mctsSearch` method where `backUp` is also called:

**_Viewing a code snippet from the_** `mctsSearch` **_method of the_** `BasicTreeNode` **_class_**...

```
while (!stop) {  
	// New timer for this iteration  
	ElapsedCpuTimer elapsedTimerIteration = new ElapsedCpuTimer();  

	// Selection + expansion: navigate tree until a node not fully expanded is found, add a new node to the tree  
	BasicTreeNode selected = treePolicy();  
	// Monte carlo rollout: return value of MC rollout from the newly added node  
	double delta = selected.rollOut();  
	// Back up the value of the rollout through the tree  
	selected.backUp(delta);  
	// Finished iteration  
	numIters++;
	...
}
```

Hence, we see the following:

1.<br>
In every iteration of the MCTS loop, we select a new node to rollout from

2.<br>
At the end of the rollout, the `rollout` method returns a value based on the value estimated/calculated for its last state as follows:

**_Viewing a code snippet from the_** `rollout` **_method of the_** `BasicTreeNode` **_class_**...

```
double value = player.params.getHeuristic().evaluateState(rolloutState, player.getPlayerID());
```

In `mctsSearch`, this return value is stored in the variable `delta`.

3.<br>
`delta`  is used in the `backUp` function to add to the `totValue` parameter (denoting the reward statistic of the node) of the selected node and its parents (all the way up to the root, which has no parent; the `parent` parameter for the root will be given as `null`). The number of visits for the selected node as well as its parents is incremented by one each (since to visit the selected node, we had to visit its parents as well).

**NOTE**: In basic MCTS, the nodes generated during rollout are not added to the search tree. Also, except for the state at the end of the rollout, none of their statistics evaluated either. The state at the end of the rollout is evaluated to indicate the reward of the pathway that led to it, but this evaluation is not assigned to this end state; it is instead returned to help update the reward of the selected node of the MCTS search iteration.
