# Lab 2 - Game states, forward models & AI agents
## Testing game runs
In the last session, we saw the use of the "Game" class to run the desired game with the GUI. Here, we faced two limitations:

1. We had to edit the arguments in the source code to get the desired changes (ex. game to play, number of players, etc.)
2. We could not obtain more detailed data about the games (ex. performance of various agents)

### Solving issue 1
The first issue is solvable in Intellij IDEA through the use of JSON configurations, wherein we apply the configurations (specifically, what arguments to use) for a piece of code to run by (1) detailing them in a JSON file and (2) applying these configurations through the "Edit Configurations" functionaity under the "Run" option in the taskbar.

**More on the "Edit Configurations" functionality**:<br>
In this functionality, we can (via a GUI, i.e. a popup window) add and edit configurations we want to apply to any desired file (ex. the source or object code) in the project directory. Note that we can specify our file under the "Build and run" heading (we can browse the desired file in the given input box). Importantly, we want to add (if not available already) a configuration of type "Application"; this kind of configuration handles the arguments of executable code, i.e. applications. We can give any name for our application. To specify the configuration (JSON) file, we provide the following argument:

`config=<path of JSON file>`

Of course, the path is given relative to our working directory, which is in our case our project directory. An example for this argument:

`config=json/experiments/rungames0.json`

### Solving issue 2
The second issue is easily solved by using the `evaluation.RunGames` class (instead of the `Game` class) found in the source file "src/main/evaluation/RunGames" (relative to the project directory of course). We can specify this in our configuration (in the "Edit Configurations" window) under the "Build and run" heading.
