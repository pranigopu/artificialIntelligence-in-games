# Lab 3: MCTS & stochastic games

$\lambda = 1$

## Exercise 1
### Introduction
Tree nodes are implemented in the class `players.mcts.BasicTreeNode` (i.e. the `BasicTreeNode` class in the directory "src/main/java/players/basicMCTS"). This class implements the different functions of MCTS required to be done from a tree node, namely:

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

### Responses
#### 1
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
