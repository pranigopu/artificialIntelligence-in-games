# Lab 1 - Getting started with TAG
(_**TAG: Tabletop Games Framework**_)

TAG is a Java-based benchmark[^1] for developing board games for AI research. TAG offers the following:

- A common API for AI agents to communicate with each other
- A set of modules and packages to help add new games
- A module for enabling data definition in JSON format
- A bunch of example games already created
- Logging functionality (to analyze the games in operation)

## TAG installation notes

- Prerequisites:
	- Java version 8 or above
	- A Java IDE (preferably IntelliJ IDEA, for reasons discussed later)
	- GitHub Desktop (for enabling you to clone repositories) (_cloning a GitHub repository simply means downloading a copy of it on your system_)

Clone the [TAG GitHub repository](https://github.com/GAIGResearch/TabletopGames) on your system (in any desired location). The repository folder is your project folder.

### Notes for IntelliJ IDEA IDE

- In the "File" option of the IDE, go to "New", and within "New" click on "Project from Existing Sources" (since you already have the project folder in your system)
- Select the project folder in the file browser
- Next, you will be prompted to import the project; select the option "Import project from external model" (_this option enables you to create a model of your project using one of the listed tools and import this model along with the original source code files; the need for this is discussed next_)
- Among the tools listed, pick "Maven"; this tool will automatically build an executable project model (_compiling and combining all the required dependencies for the main source code(s)) that will enable you to test the code directly_)

_Maven (i.e. Apache Maven) is discussed further in the section below_.

**_IntelliJ IDEA IDE is preferred as it automates the process of installing and using Maven to build a testable project model_**.

### Notes for non-IntelliJ IDEA IDE's
#### Apache Maven
##### Introduction
To run and manage a whole Java project (consisting of a set of interrelated packages and modules) such as TAG, it is useful to create executables that compile and combine the various dependencies of the main source code(s). To help in this, we use Apache Maven.

Apache Maven (or simply Maven) is a tool to (1) automate the building of, (2) facilitate the management of, and (3) provide comprehensive project-related information _for Java-based software projects_. To help facilitate the management of the project, Maven provides a uniform build system, i.e. a common project-building approach, so that navigating a Maven project becomes easy with just a little familiarization. To be more specific, Maven uses a POM (project object model) and a set of plugins to build any project; each project directory must thus have the appropriate POM if it is to be handled by Maven.

For more information about Maven, including the kind of project-related information it can provide and the developer best-practices it implements, check about Maven in its [official website](https://maven.apache.org/what-is-maven.html).

##### Installation (Windows)

- Go to the [downloads page](https://maven.apache.org/download.cgi) and download any of the binary ZIP files (_if you download the TAR binary ZIP file, you will need the right software tools to extract the files within it, which you can look up_)
- Extract the ZIP file once downloaded (_you may first move the file to any location you like, but note that in certain folders such as "C:\Program Files", you may need admin privileges when extracting a TAR ZIP file using the command line. You can always move the extracted folder in any location as well_)
- In the now extracted folder, there is a "bin" folder; copy or note down the complete path of this "bin" folder within your system
- Add this path to the "Path" environment variable of your system by first opting to edit the "Path" variable, then adding the path of the aforementioned "bin" folder (_if unsure about how, look up how to edit environment variables in Windows_)
- This will make Maven functionalities accessible by CLI through the keyword `mvn`.

#### Using Maven for your project folder (CLI method)

- Navigate to your project folder in CLI
- Run the command `mvn verify`
- A new subfolder "target" will be created within the project folder and will contain the compiled code and executables built by Maven
- You can now run the appropriate executables (through CLI or GUI)

_For TAG, the appropriate executable to run existing games is the JAR file for the frontend_.

### Notes for finding the main source code

The main source code that can be executed is the "Games.java" file found in the subfolder `TabletopGames\src\main\java\core`. Simply run the code to execute it. To change the game being run, modify the `gameType` string variable in the main function (`public static void main`) (modify the third argument of the function call on the right hand side). Make sure that the game's name that you enter is valid (i.e. exists within the project framework).

[^1]:A benchmark in computing is a test used to compare performance between multiple objects/systems (hardware or software), either against each other or against an accepted standard. In the case of TAG, it is a framework that offers an environment that enables you to develop, test and compare AI-based board games.
