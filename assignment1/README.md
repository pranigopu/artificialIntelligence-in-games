# AIG: Assignment 1<br> _Creating an AI agent Sushi Go_
This was a team project consisting of Graham Innocent, Malo Hamon and myself (Pranav Gopalkrishna), The focus of this project was to implement and compare sampling methods that serve as alternative bandit methods to the default/common upper confidence bound (UCB1) tree policy. Our project also focused on finding ways to augment these methods with functional enhancements to the core Monte Carlo tree search (MCTS) algorithm, namely pruning (to manage the search space), multi-root MCTS and information set MCTS (ISMCTS) (both to account for partial observability), as well as additional heuristics such as all-moves-as-first (AMAF) heuristics and the related method rapid action value estimation (RAVE).

**Team project repository**: https://github.com/grahaminn/AIinGames-Assignment1

## TAG (the benchmark to be used in the project)
For our project, we were supposed to use the TAG (<ins>TA</ins>bletop <ins>G</ins>ames) framework. TAG is a Java-based benchmark for developing board games for AI research. TAG offers the following:

- A common API for AI agents to communicate with each other
- A set of modules and packages to help add new games
- A module for enabling data definition in JSON format
- A bunch of example games already created
- Logging functionality (to analyze the games in operation)

## Individual contributions
Graham Innocent's major contributions to the agent's code were the implementation and testing Bayes-UCB sampling  and hard pruning for all our MCTS  variants. For the group report, he wrote the first draft of abstract, introduction, explanation of TAG and the method, experimentation and discussion sections on hard pruning and Bayes-UCB. Graham's other contributions (not included in the report) include:

- Setting up the team repository on GitHub
- Implementing breadth-first search to get an idea about the TAG framework used
-  Implemented a way of configuring alternative rollout opponents other than `RandomPlayer` and tested effect of swapping in OSLA or basic MCTS players with low timeout as the rollout opponent

Malo Hamon’s major contributions to the agent's code were the implementation and testing of Thompson sampling, AMAF  heuristic, RAVE  and multi-root MCTS. For the group report, he wrote the wrote the method, experimentation and discussion sections on Thompson sampling, AMAF and RAVE. He also wrote the experimentation and discussion sections for multi-root MCTS, the report's final conclusion and the README file of the agent.

I worked on the implementation and testing of ISMCTS , but my implementation was faulty, showed mediocre results, and thus, was not included in the submission. I also tested the effect of progressive bias for the tree policy, which was negligible and also left out of the final report. Hence, I did not contribute to the final agent's code. For the group report, I wrote the background section explaining the MCTS algorithm and the method section for multi-root MCTS along with the pseudocode. I also configured and ran the conclusive experiment that compared our team's MCTS variants, whose results were processed and presented by Malo. Finally, I wrote the sections explaining the exploration-exploitation dilemma (prelude to sampling methods) and the partial observability problem (prelude to multi-root MCTS).

This project pushed me to understand how the theoretical concepts learnt in lectures could be implemented, and my teammates’ initiative exposed me to new areas of RL  and bandit methods.

## Personal notes on the project
### About Sushi Go
Sushi Go's key features to consider...

- Strategy revolves around how to select the cards from the hand you are passed
  - NOTE: The card groups being first distributed then passed can be considered random
- You know about the deck overall & the card-based point system
- You can see all your own cards & hand as well as others' face-up cards (but not their hands)
  - The adversarial aspect is based on what cards you let or don't let your opponents have based on their face-up cards
- Each round is mostly independent of the previous rounds, except for the pastry card
- Collection of pastry must be evaluated against other options; thus, actions can have effects between rounds
- Each decision tree is as follows (per turn)
  - Root node: Current game state
  - Branch: Action of picking one or more cards (minimum 1, maximum 2)
- Environment
  - Discrete (due to discrete-valued outcome parameters)
  - Partial information
  - Stochastic
    - Though point system is mostly fixed, parts of it rely on what others get
    - What everyone gets is randomized, thus creating a stochastic environment
- Complexity
  - High branching factor (many possible card combinations to pick)
  - Deep tree with long-ranging decisions
    - $\implies$ High-reward long-range strategies that may lead to seemingly lower advantages short-range
