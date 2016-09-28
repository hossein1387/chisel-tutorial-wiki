
This chapter is an installation guide for *Chisel* (Constructing
Hardware In a Scala Embedded Language) and is intended to prepare your system for subsequent tutorials.  Chisel is a hardware
construction language embedded in the high-level programming language
Scala.

## Development Tool Installation

The chisel tutorials should be able to be run on a variety of modern systems.  
For the most update instructions on
preparing your machine for Chisel see [Chisel Installation Preparation](https://github.com/ucb-bar/chisel3/wiki/Install)

## Setting Up the Tutorial

In subsequent tutorials, you will be using the files distributed in the chisel-tutorial repository. To obtain these tutorials files, \verb+cd+ to the directory = \verb+$DIR+ where you want to place the Chisel tutorial and type:

```bash
export DIR=~/chisel-workspace
mkdir -p $DIR
cd $DIR
git clone https://github.com/ucb-bar/chisel-tutorial.git
cd chisel-tutorial
```

You will now be in your copy of the Chisel Tutorial repository will then be in **$DIR/chisel-tutorial**.  
Define this as a variable in your bash environment named \verb+$TUT_DIR+.

This is the Chisel tutorial directory structure you should see, which is explained more in the next tutorial:

chisel-tutorial
 - build.sbt # project description
 - doc/
 - src/
   - main/
     - scala/
       - examples/
         - More advanced example circuits are found here
       - problems/
         - A series of problem circuits to be completed by the user
       - solutions/  # solutions to problems
         - Solutions to the problems circuits
   - test/
     - resources/
       - Resources required by some of the tests
     - scala/
       - examples/
         - Test harnesses for the example circuits
       - problems/
         - Test harnesses for the problem circuits
       - solutions/
         - Test harnesses for the solution circuits
       - util/
         - TutorialRunner.scala

 - [Next (The Basics)](The Basics)