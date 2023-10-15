# Artificial Intelligence in Games
Code from my learnings in the first semester module "Artificial Intelligence in Games" of my MSc. AI course.

## Lab sessions list
- Lab 1: Getting started with TAG (Tabletop Games Framework)
- Lab 2: Game states, forward models & AI agents (TAG)
- Lab 3: MCTS & stochastic games

## GitHub-related notes
- Cloning a GitHub repository simply means downloading a copy of it on your system
- GitHub supports linked footnotes in its markdown files

# General conceptual & technical notes
## Common terms & term-related confusions
- CLI: Command Line Interface
- GUI: Graphical User Interface
- OOP: Object Oriented Progamming
- Note that in object-oriented programming, "method" means the same thing as "function"
- "Folder" means practically the same thing as "directory". Folder is used in a Windows context while directory in a Linux context.

## Useful computing concepts used in the labs
### Benchmark (in computing)
A benchmark in computing is a test used to compare performance between multiple objects/systems (hardware or software), either against each other or against an accepted standard. For example, it could be a framework that offers an environment that enables you to develop, test and compare software modules.

### Instantiation of a class (OOP)
Instantiation refers to (unless mentioned otherwise) the instantiation of a class, i.e. the creation of an object that concretizes (_i.e. realizes in execution_) the specifications of a class.

**NOTE**: A class is an encapsulation of attributes and methods (i.e. functions) that represents a programmable unit (i.e. an abstract representation of a member of a group of similar members). An object is a particular programmable entity that shares its defining characteristics (attributes and methods) with the class, i.e. it is a particular member of the category, rather than a general representation of members of that category.

**SIDE NOTE 1**: INSTANTIATING ABSTRACT CLASSES IN JAVA<br>
_In Java, an abstract class cannot be directly instantiated, but it can be instantiated indirectly through an concrete subclass_.

**SIDE NOTE 2**: INDIRECT INSTANTIATION OF A PARENT CLASS<br>
_An object of a subclass B of a parent class A is also an instantiation (indirectly, but still) of the parent class A, since a subclass is logically contained within the parent class. Java also considers it as such, since if a method specifies the object of a certain class as its parameter (i.e. argument), it also accepts an object of this class's subclass for this parameter_.

### Overloading
This is a feature in some programming languages (ex. Java) that allow the same name to be used for different functions provided that the function definitions have specified different parameters as arguments. Hence, these functions with the same name are distinguished by the parameters they accept.

### Overriding a parent class's method (OOP)
Overriding a parent class's method refers to a redefinition (by a subclass that inherits the parent class) of a certain function defined in the parent class (_note that in some programming languages (ex. Java), you can overload function names, so in such cases, both the name and the parameters must be the same to actually redefine the desired method_). Note that certain types of functions cannot be overridden (ex. a function of type "final" in Java).
