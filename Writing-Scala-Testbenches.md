Chisel's Scala based testbench is the first line of defense against simple bugs in your design. The Scala testbench uses several unique Chisel constructs to perform this. To see how this works, let's first explore a simple example.

### Scala Testbench Example

Below is the *ByteSelector.scala* module definition from the previous tutorial and the corresponding Chisel test harness.

```scala
package TutorialExamples

import Chisel._

class ByteSelector extends Module {
  val io = new Bundle {
    val in     = UInt(INPUT, 32)
    val offset = UInt(INPUT, 2)
    val out    = UInt(OUTPUT, 8)
  }
  io.out := UInt(0, width = 8)
  when (io.offset === UInt(0)) {
    io.out := io.in(7,0)
  } .elsewhen (io.offset === UInt(1)) {
    io.out := io.in(15,8)
  } .elsewhen (io.offset === UInt(2)) {
    io.out := io.in(23,16)
  } .otherwise {
    io.out := io.in(31,24)
  }    
}

class ByteSelectorTests(c: ByteSelector) 
    extends Tester(c) {
  val test_in = 12345678
  for (t <- 0 until 4) {
    poke(c.io.in,     test_in)
    poke(c.io.offset, t)
    step(1)
    expect(c.io.out, (test_in >> (t * 8)) & 0xFF)
  }
}
```

In the test harness *ByteSelectorTests* we see that the test portion is written in Scala with some Chisel constructs inside a *Tester* class definition. The device under test is passed to us as a parameter *c*. 

In the *for* loop, the assignments for each input of the *ByteSelector* is set to the appropriate values using *poke*. For this particular example, we are testing the *ByteSelector* by hardcoding the input to some known value and checking if each of the 4 offsets returns the appropriate byte. To do this, on each iteration we generate appropriate inputs to the module and tell the simulation to assign this value to the input of the device we are testing *c*:

```scala
val test_in = 12345678
for (t <- 0 until 4) {
  // set in of the DUT to be some known word
  poke(c.io.in,     test_in)
  // set the offset of the DUT
  poke(c.io.offset, t)
  ...
}
```

Next we step the circuit.  We next advance the simulation by calling the *step* function. This effectively advances the simulation one clock cycle in the presence of sequential logic. 
% Martin: Something needs to be said what step does relative to combinational circuits

```scala
step(1)
```

Finally, we check for expected outputs.
In this case, we check the expected output of *ByteSelector* as follows:

```scala
expect(c.io.out, (test_in >> (t * 8)) & 0xFF)
```

This defines the reference output expected for this particular cycle of the simulation. Since the circuit we are testing is purely combinational, we expected that the output we define appears on any advancement of the simulation.  The *expect* function will record either true or false after checking if the output generates the expected reference output. The results of successive *expect*'s are anded into a *Tester* field called *ok* which starts out as *true*.  The value of the *ok* field determines the success or failure of the tester execution.

Actually *expect* is defined in terms of *peek* roughly as follows:

```scala
def expect (data: Bits, expected: BigInt) = 
  ok = peek(data) == expected && ok
```

where *peek* gets the value of a signal from the DUT.

### Simulation Debug Output

Now suppose we run the testbench for the *ByteSelector* defined previously. To do this, *cd* into the *$DIR/problems* directory and run *make ByteSelector*.

When we run the testbench, we will notice that the simulation produces debug output every time the *step* function is called. Each of these calls gives the state of the inputs and outputs to the *ByteSelector* and whether the check between the reference output and expected output matched as shown below:

```bash
STARTING ../emulator/problems/ByteSelector
---
POKE ByteSelector__io_in <- 12345678
POKE ByteSelector__io_offset <- 0
STEP 1 <- 0
PEEK ByteSelector__io_out -> 0x4e
EXPECT ByteSelector__io_out <- 78 == 78 PASS
POKE ByteSelector__io_in <- 12345678
POKE ByteSelector__io_offset <- 1
STEP 1 <- 0
PEEK ByteSelector__io_out -> 0x61
EXPECT ByteSelector__io_out <- 97 == 97 PASS
...
POKE ByteSelector__io_in <- 12345678
POKE ByteSelector__io_offset <- 3
STEP 1 <- 0
PEEK ByteSelector__io_out -> 0x00
EXPECT ByteSelector__io_out <- 0 == 0 PASS
PASSED   // Final pass assertion
[success] Total time: 6 s, completed Feb 23, 2014 9:52:22 PM
```

Also notice that there is a final pass assertion "PASSED" at the end which corresponds to the *allGood* at the very end of the testbench. In this case, we know that the test passed since the allGood assertion resulted in a "PASSED". In the event of a failure, the assertion would result in a "FAILED" output message here.

### General Testbench

In general, the scala testbench should have the following rough structure:

 - Set inputs using *poke*
 - Advance simulation using *step*
 - Check expected values using *expect* (and/or *peek*)
 - Repeat until all appropriate test cases verified

For sequential modules we may want to delay the output definition to the appropriate time as the *step* function implicitly advances the clock one period in the simulation. Unlike Verilog, you do not need to explicitly specify the timing advances of the simulation; Chisel will take care of these details for you.

### *Max2 Testbench*

In this assignment, write a tester for the *Max2* circuit:

```scala
class Max2 extends Module {
  val io = new Bundle {
    val in0 = UInt(INPUT,  8)
    val in1 = UInt(INPUT,  8)
    val out = UInt(OUTPUT, 8)
  }
  io.out := Mux(io.in0 > io.in0, io.in0, io.in1)
}
```

found in *$TUT_DIR/problems/Max2.scala* by filling in the following tester:

```scala
class Max2Tests(c: Max2) extends Tester(c) {
  for (i <- 0 until 10) {
    // FILL THIS IN HERE
    poke(c.io.in0, 0)
    poke(c.io.in1, 0)
    // FILL THIS IN HERE
    step(1)
    expect(c.io.out, 1)
  }
}
```
 
using random integers generated as follows:

```scala
// returns random int in 0..lim-1
val in0 = rnd.nextInt(lim) 
```

Run 

```bash
make Max2.out
```
 
until the circuit passes your tests.

### Limitations of the Testbench

The Chisel testbench works well for simple tests and small numbers of simulation iterations. However, for larger test cases, the Chisel testbench quickly becomes more complicated and slower simply due to the inefficiency of the infrastructure. For these larger and more complex test cases, we recommend using the C++ emulator or Verilog test harnesses which run faster and can handle more rigorous test cases.

[Prev (Instantiating Modules)](Instantiating Modules) [Next (Creating Your Own Project)](Creating Your Own Project)