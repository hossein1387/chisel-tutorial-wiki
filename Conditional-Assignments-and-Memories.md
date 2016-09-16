Often times parametrization requires instantiating multiple components which are connected in a very regular structure. A revisit to the parametrized *Adder* component definition shows the *for* loop construct in action:

```scala
//A n-bit adder with carry in and carry out
class Adder(val n:Int) extends Module {
  val io = new Bundle {
    val A    = UInt(INPUT, n)
    val B    = UInt(INPUT, n)
    val Cin  = UInt(INPUT, 1)
    val Sum  = UInt(OUTPUT, n)
    val Cout = UInt(OUTPUT, 1)
  }
  //create a vector of FullAdders
  val FAs   = Vec.fill(n)(Module(new FullAdder()).io)
  val carry = Wire(Vec(n+1, UInt(width = 1)))
  val sum   = Wire(Vec(n, Bool()))

  //first carry is the top level carry in
  carry(0) := io.Cin

  //wire up the ports of the full adders
  for (i <- 0 until n) {
    FAs(i).a := io.A(i)
    FAs(i).b := io.B(i)
    FAs(i).cin := carry(i)
    carry(i+1) := FAs(i).cout
    sum(i) := FAs(i).sum.toBool()
  }
  io.Sum := Cat(sum.reverse)
  io.Cout := carry(n)
}
```

Notice that a Scala integer *i* value is used in the *for* loop definition as the index variable. This indexing variable is specified to take values from 0 *until* n, which means it takes values 0, 1, 2..., n-1. If we wanted it to take values from 0 to n inclusive, we would use *for (i <- 0 to n)*.

It is also important to note, that the indexing variable *i* does not actually manifest itself in the generated hardware. It exclusively belongs to Scala and is only used in declaring how the connections are specified in the Chisel component definition.

The for loop construct is also very useful for assigning to arbitrarily long *Vec*s 

### Using If, Else If, Else

As previously mentioned, the *if, elseif,* and *else* keywords are reserved for Scala control structures. What this means for Chisel is that these constructs allow you to selectively generate different structures depending on parameters that are supplied. This is particularly useful when you want to turn certain features of your implementation "on" or "off", or if you want to use a different variant of some component.

For instance, suppose we have several simple counters that we would like to package up into a general purpose counter module: UpCounter, DownCounter, and OneHotCounter. From the definitions below, we notice that for these simple counters, the I/O interfaces and parameters are identical:

```scala
// Simple up counter that increments from 0 and wraps around
class UpCounter(CounterWidth:Int) extends Module {
  val io = new Bundle {
    val output = UInt(OUTPUT, CounterWidth)
    val ce     = Bool(INPUT)
  }...
}

// Simple down counter that decrements from 
// 2^CounterWidth-1 then wraps around
class DownCounter(CounterWidth:Int) extends Module{
  val io = new Bundle {
    val output = UInt(OUTPUT, CounterWidth)
    val ce     = Bool(INPUT)
  }...
}

// Simple one hot counter that increments from one hot 0 
// to CounterWidth-1 then wraps around
class OneHotCounter(CounterWidth:Int) extends Module {
  val io = new Bundle {
      val output = UInt(OUTPUT, CounterWidth)
      val ce     = Bool(INPUT)
  }...
}
```

We could just instantiate all three of these counters and multiplex between them but if we needed one at any given time this would be a waste of hardware. In order to choose between which of these three counters we want to instantiate, we can use Scala's *if, else if, else* statements to tell Chisel how to pick which component to instantiate based on a *CounterType* parameter:

```scala
class Counter(CounterWidth: Int, CounterType: String) 
    extends Module {
  val io = new Bundle {
    val output = UInt(OUTPUT, CounterWidth)
    val ce     = Bool(INPUT)
  }
  if (CounterType == "UpCounter") {
     val upcounter = new UpCounter(CounterWidth)
     upcounter.io.ce := io.ce
     io.output := upcounter.io.output
  } else if (CounterType == "DownCounter") {
    val downcounter = new DownCounter(CounterWidth)
    downcounter.io.ce := io.ce
    io.output := downcounter.io.output
  } else if (CounterType == "OneHotCounter") {
    val onehotcounter = new OneHotCounter(CounterWidth)
    onehotcounter.io.ce := io.ce
    io.output := onehotcounter.io.output
  } else {
    // default output 1
    io.output := UInt(1)
  }
}
```

By consolidating these three counter components into a single *Counter* module, we can instantiate a different counter by simply changing the parameter *CounterType*. For instance:

```scala
// instantiate a down counter of width 16
val downcounter = 
  Module(new Counter(16, "DownCounter"))

// instantiate an up counter of width 16
val upcounter = 
  Module(new Counter(16, "UpCounter"))

// instantiate a one hot counter of width 16
val onehotcounter = 
  Module(new Counter(16, "OneHotCounter"))
```

This allows seamless alternation between them.

### Using def

Chisel also allows the usage of the Scala *def* statement to define Chisel code that may be used frequently. These *def* statements can be packaged into a Scala Object and then called inside a Module. The following Chisel code shows an alternate implementation of an counter using *def* that increments by *amt* if the *inc* signal is asserted.

```scala 
object Counter {
  def wrapAround(n: UInt, max: UInt) = 
    Mux(n > max, UInt(0), n)

  def counter(max: UInt, en: Bool, amt: UInt) = {
    val x = Reg(init = UInt(0, max.getWidth))
    x := wrapAround(x + amt, max)
    x
  }
}

class Counter extends Module {
  val io = new Bundle {
    val inc = Bool(INPUT)
    val amt = UInt(INPUT,  4)
    val tot = UInt(OUTPUT, 8)
  }
  io.tot := counter(UInt(255), io.inc, io.amt)
}
```
 
In this example, we use calls to subroutines defined in the *Counter* object in order to perform the appropriate logic. 

### *Parameterized Vec Shift Reg*

The next assignment is to construct a bit shift register with delay parameter.
The following is a the template from *$TUT_DIR/problems/VecShiftRegisterParam.scala*:

```scala
class VecShiftRegisterParam(val n: Int, val w: Int) extends Module {
  val io = new Bundle {
    val in  = UInt(INPUT,  w)
    val out = UInt(OUTPUT, w)
  }
  ...
  io.out := UInt(0)
}
```

where *out* is a *n* cycle delayed copy of values on *in*.  
Also notice how *val* is added to each parameter value to 
allow those values to be accessible from the tester. Run 

```bash
make VecShiftRegisterParam.out
```
 
until your circuit passes the tests.

### *Mul Lookup Table*

The next assignment is to write a 16x16 multiplication table using *Vec*.
The following is a the template from *$TUT_DIR/problems/Mul.scala*:

```scala
class Mul extends Module {
  val io = new Bundle {
    val x   = UInt(INPUT,  4)
    val y   = UInt(INPUT,  4)
    val z   = UInt(OUTPUT, 8)
  }
  val muls = new ArrayBuffer[UInt]()

  // flush this out ...

  io.z := UInt(0)
}
```

As a hint build the lookup table using a rom constructed from the *tab* lookup table represented as a Scala ArrayBuffer with incrementally added elements (using *+=*):

```scala
val tab = Vec(muls)
```
and lookup the result using an address formed from the *x* and *y* inputs as follows:

```scala
io.z := tab(Cat(io.x, io.y))
```

Run 

```bash
make Mul.out
```
until your circuit passes the tests.

[Prev (Creating Your Own Project)](Creating Your Own Project) 
[Next (Scripting Hardware Generation)](Scripting Hardware Generation)
