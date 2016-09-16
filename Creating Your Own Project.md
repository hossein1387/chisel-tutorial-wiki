## Creating Your Own Projects

In order to create your own projects from scratch, you will need to create a directory, a Chisel source file, an optional tester, and a build.sbt configuration file.
You should clone the [chisel-template](https://github.com/ucb-bar/chisel-template) repo, which contains a minimal project suitable as a template.
In the first part of this tutorial we cover the basic calls to SBT in order generate appropriate files.

#### Directory Structure

Although the simplest project file organization is using a single directory containing your SBT file and your Chisel source file, we suggest you adopt the SBT convention of organization your project with `src`, and `test` sub-directories.
The simplest project directory structure would look like:

``` bash
Hello/
  build.sbt   # scala configuration file
  src/main/scala/hello/Hello.scala # your source file
```

We will refer to the path to the `Hello` directory as `$BASEDIR` from here on.
Consult the SBT documentation for more information.

#### The Source Directory and Chisel Main

The src directory `$BASEDIR/src/main/scala/<project>` contains Scala source files containing all of the Chisel module definitions and the main method for your circuit *<project>*.
In this simple example, we have one Scala source file as shown below:

``` scala
// See LICENSE.txt for license details.
package hello

import Chisel._
import Chisel.iotesters.{PeekPokeTester, Driver}

class Hello extends Module {
  val io = new Bundle { 
    val out = UInt(OUTPUT, 8)
  }
  io.out := UInt(42)
}

class HelloTests(c: Hello) extends PeekPokeTester(c) {
  step(1)
  expect(c.io.out, 42)
}

object Hello {
  def main(args: Array[String]): Unit = {
    if (!Driver(() => new Hello())(c => new HelloTests(c))) System.exit(1)
  }
}
```

In the above example, we have a module definition in package `Hello` for a `Hello` module.
The main method calls `Driver` with a generator that constructs a new Hello module, and a new tester to test the constructed module.
In addition to creating the module, the call to `Driver` also includes a call to execute the scala testbench defined in the class `HelloTests`.

#### The build.sbt Template

The `build.sbt` configuration file is located in the top folder and contains a number of settings used by `sbt` when building and compiling the Chisel sources.
The following shows the recommended `build.sbt` template that should be used:

``` scala
scalaVersion := "2.11

resolvers ++= Seq(
  Resolver.sonatypeRepo("snapshots"),
  Resolver.sonatypeRepo("releases")
)

libraryDependencies += 
  "edu.berkeley.cs" %% "chisel3" % "latest.release"
```

The SBT project file contains a reference to Scala version greater or equal to 2.11 and a dependency on the latest release of the Chisel (chisel3) library.

## Compiling the Chisel Source

#### Compiling the Emulation Files and Running the Chisel Tests

In order to launch SBT to compile the Chisel code we must first be in the directory `$BASEDIR/`.
The following call is then made to compile and run the Hello module, and drive it with `HelloTests` :

``` bash
sbt run
```

#### Compiling Verilog

Similarly to compile the Chisel code, generate the Verilog HDL, and generate, compile, and run the Verilator simulation, we execute the same command with the TESTER_BACKENDS environment variable set appropriately.
The call looks like:

``` bash
TESTER_BACKENDS=verilator sbt run
```

## Putting It All Together

In summary, the bare minimum project components that are necessary for your project to get off the ground are the following files:

* `$BASEDIR/build.sbt`
* `$BASEDIR/<Chisel source files>.scala`

Together, these files compose a Chisel project and can be used to generate the Verilog and C++ files.
It is strongly recommended that you supplement the file structure with appropriate Makefiles but is not strictly necessary (examples can be found in the Chisel tutorial project).

