# AIG: Assignment 1<br> _Creating an AI agent Sushi Go_

---

**Team project repository**: https://github.com/grahaminn/AIinGames-Assignment1

**Project report**: https://github.com/pranigopu/artificialIntelligence-in-games/blob/main/assignment1/REPORT.pdf

---

This was a team project consisting of Graham Innocent, Malo Hamon and myself (Pranav Gopalkrishna), The focus of this project was to implement and compare sampling methods that serve as alternative bandit methods to the default/common upper confidence bound (UCB1) tree policy. Our project also focused on finding ways to augment these methods with functional enhancements to the core Monte Carlo tree search (MCTS) algorithm, namely pruning (to manage the search space), multi-root MCTS and information set MCTS (ISMCTS) (both to account for partial observability), as well as additional heuristics such as all-moves-as-first (AMAF) heuristics and the related method rapid action value estimation (RAVE).

## TAG (the benchmark to be used in the project)
For our project, we were supposed to use the TAG (<ins>TA</ins>bletop <ins>G</ins>ames) framework. TAG is a Java-based benchmark for developing board games for AI research. TAG offers the following:

- A common API for AI agents to communicate with each other
- A set of modules and packages to help add new games
- A module for enabling data definition in JSON format
- A bunch of example games already created
- Logging functionality (to analyze the games in operation)

## Individual contributions
Graham Innocent's major contributions to the agent's code were the implementation and testing Bayes-UCB sampling[^1]  and hard pruning[^2] for all our MCTS variants. For the group report, he wrote the first draft of abstract, introduction, explanation of TAG and the method, experimentation and discussion sections on hard pruning and Bayes-UCB. Graham's other contributions (not included in the report) include:

- Setting up the team repository on GitHub
- Implementing breadth-first search to get an idea about the TAG framework used
-  Implemented a way of configuring alternative rollout opponents other than `RandomPlayer` and tested effect of swapping in OSLA or basic MCTS players with low timeout as the rollout opponent

Malo Hamon’s major contributions to the agent's code were the implementation and testing of Thompson sampling, AMAF  heuristic, RAVE  and multi-root MCTS. For the group report, he wrote the wrote the method, experimentation and discussion sections on Thompson sampling, AMAF and RAVE. He also wrote the experimentation and discussion sections for multi-root MCTS, the report's final conclusion and the README file of the agent.

I worked on the implementation and testing of ISMCTS , but my implementation was faulty, showed mediocre results, and thus, was not included in the submission. I also tested the effect of progressive bias for the tree policy, which was negligible and also left out of the final report. Hence, I did not contribute to the final agent's code. For the group report, I wrote the background section explaining the MCTS algorithm and the method section for multi-root MCTS along with the pseudocode. I also configured and ran the conclusive experiment that compared our team's MCTS variants, whose results were processed and presented by Malo. Finally, I wrote the sections explaining the exploration-exploitation dilemma (prelude to sampling methods) and the partial observability problem (prelude to multi-root MCTS).

This project pushed me to understand how the theoretical concepts learnt in lectures could be implemented, and my teammates’ initiative exposed me to new areas of reinforcement learning and bandit methods.

[^1]:As per: http://proceedings.mlr.press/v22/kaufmann12/kaufmann12.pdf
[^2]:As per: https://ceur-ws.org/Vol-2862/paper27.pdf

## Personal notes on the project
### About Sushi Go
Sushi Go's key features to consider...

- Strategy revolves around how to select the cards from the hand you are passed
  - **NOTE**: _The card groups being first distributed then passed can be considered random_
- You know about the deck overall & the card-based point system
- You can see all your own cards & hand as well as others' face-up cards
  - But not their hands (i.e. the cards the hold)
  - Adversarial aspect is based on what cards you let or don't let your opponents have
    - Such a decision is based on their face-up cards
- Each round is mostly independent of the previous rounds<br>(_except for the pastry card_)
- Collection of pastry must be evaluated against other options
  - $\implies$ Actions can have effects between rounds
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
   
### Potential route for creating agents (not followed)
The complexities listed can be best tackled by the general approaches of Monte Carlo tree search (MCTS) or rolling horizon evolutionary algorithms (RHEA). Why these?

MCTS uses random sampling of pathways to obtain an estimate of how good or bad an action is from a given state. This removes the need for exhaustive branching, reducing memory and time requirements by many orders of magnitude. Furthermore, the law of large numbers implies that with enough sampling (using the appropriate sampling methods to make this process more efficient), we can obtain a fairly accurate estimate of how good an action is from a given state.

**Further notes on MCTS usage**...

Since the domain (Sushi Go) is non-deterministic, we must use a closed-loop system (no storage of states within node objects), wherein we continually use the forward model (FM) on the current state to simulate and evaluate future states; thus, time complexity cannot be reduced in this aspect

RHEA uses evolutionary search of pathways to obtain the (supposed) best sequence of actions from a given state. Similarly to MCTS, RHEA also generates the populations of pathways using some kind of sampling. In other words, we are dealing with only a fraction of the search space, removing the need for exhaustive branching. However, unlike MCTS, instead of using the FM to create samples after the first one, we use evolutionary methods to create consequent generations of pathways derived from the (supposedly) best individual pathways from the previous generation. As in MCTS, we use certain functions (fitness functions, heuristics, reward functions) to help us obtain the goodness of both steps and whole pathways, but instead of using inferential statistics to help guide us, we use evolutionary methods.

The question is: _why would we choose one over the other? Which approach is better suited for us and why_?

The answer is not immediately obvious in theory, which raises the need for experimentation. However, we aim to experiment in a way that helps us answer the following questions:

- _What base approach (inferential statistics or evolutionary methods) is better suited to dealing with stochastic, partial information and discrete domains & why_?
- _What enhancements to each base approach would elevate the performance of each approach_?
- _Why do these enhancements have their effect_?
- _Which base approach has more potential (in theory) of dealing with stochastic, partial information and discrete domains, with the help of enhancements_?

