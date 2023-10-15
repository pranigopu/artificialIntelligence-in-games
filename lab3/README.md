# Lab 3: MCTS & stochastic games
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
