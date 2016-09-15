_This section is advanced and can be skipped at first reading._

Scala is a strongly typed language and uses parameterized types to specify generic functions and classes.
In this section, we show how Chisel users can de-fine their own reusable functions and classes using parameterized classes.

# Parameterized Functions

Earlier we defined `Mux2` on `Bool`, but now we show how we can define a generic multiplexer function.
We define this function as taking a boolean condition and con and alt arguments (corresponding to then and else expressions) of type `T`:

```scala
def Mux[T <: Bits](c: Bool, con: T, alt: T): T = { ... }
```

where `T` is required to be a subclass of `Bits`.
Scala ensures that in each usage of `Mux`, it can find a common superclass of the actual con and alt argument types,
otherwise it causes a Scala compilation type error.
For example,

```scala
Mux(c, UInt(10), UInt(11))
```

yields a `UInt` wire because the `con` and `alt` arguments are each of type `UInt`.

<!---
Jack: I cannot seem to get this to actually work
      Scala does not like the * in FIR since it could be from UInt or SInt

We now present a more advanced example of parameterized functions for defining an inner product FIR digital filter generically over Chisel `Num`s.

The inner product FIR filter can be mathematically defined as:
\begin{equation}
y[t] = \sum_j w_j * x_j[t-j]
\end{equation}


where `x` is the input and `w` is a vector of weights.
In Chisel this can be defined as:


```scala
def delays[T <: Data](x: T, n: Int): List[T] = 
  if (n <= 1) List(x) else x :: delays(RegNext(x), n - 1)

def FIR[T <: Data with Num[T]](ws: Seq[T], x: T): T = 
  ws zip delays(x, ws.length) map { case (a, b) => a * b } reduce (_ + _)
```
 
where 
`delays` creates a list of incrementally increasing delays of its input and
`reduce` constructs a reduction circuit given a binary combiner function `f`.  
In this case, `reduce` creates a summation circuit.
Finally, the `FIR` function is constrained to work on inputs of type `Num` where Chisel multiplication and addition are defined.
--->

# Parameterized Classes

Like parameterized functions, we can also parameterize classes to make them more reusable.
For instance, we can generalize the Filter class to use any kind of link.
We do so by parameterizing the `FilterIO` class and defining the constructor to take a single argument `gen` of type `T` as below.

```scala
class FilterIO[T <: Data](gen: T) extends Bundle { 
  val x = gen.asInput
  val y = gen.asOutput
}
```

We can now define `Filter` by defining a module class that also takes a link type constructor argument and passes it through to the `FilterIO` interface constructor:

```scala
class Filter[T <: Data](gen: T) extends Module { 
  val io = new FilterIO(gen)
  ...
}
```

We can now define a `PLink`-based `Filter` as follows:

```scala
val f = Module(new Filter(new PLink))
```

A generic FIFO could be defined as follows:

```scala
class DataBundle extends Bundle {
  val a = UInt(width = 32)
  val b = UInt(width = 32)
}

class Fifo[T <: Data](gen: T, n: Int) extends Module {
  val io = new Bundle {
    val enqVal = Bool(INPUT)
    val enqRdy = Bool(OUTPUT)
    val deqVal = Bool(OUTPUT)
    val deqRdy = Bool(INPUT)
    val enqDat = gen.asInput
    val deqDat = gen.asOutput
  }
  val enqPtr     = Reg(init = UInt(0, sizeof(n)))
  val deqPtr     = Reg(init = UInt(0, sizeof(n)))
  val isFull     = Reg(init = Bool(false))
  val doEnq      = io.enqRdy && io.enqVal
  val doDeq      = io.deqRdy && io.deqVal
  val isEmpty    = !isFull && (enqPtr === deqPtr)
  val deqPtrInc  = deqPtr + UInt(1)
  val enqPtrInc  = enqPtr + UInt(1)
  val isFullNext = Mux(doEnq && ~doDeq && (enqPtrInc === deqPtr),
                         Bool(true), Mux(doDeq && isFull, Bool(false),
                         isFull))
  enqPtr := Mux(doEnq, enqPtrInc, enqPtr)
  deqPtr := Mux(doDeq, deqPtrInc, deqPtr)
  isFull := isFullNext
  val ram = Mem(n)
  when (doEnq) {
    ram(enqPtr) := io.enqDat
  }
  io.enqRdy := !isFull
  io.deqVal := !isEmpty
  ram(deqPtr) <> io.deqDat
}
```

An Fifo with 8 elements of type DataBundle could then be instantiated as:

```scala
val fifo = Module(new Fifo(new DataBundle, 8))
```

It is also possible to define a generic decoupled (ready/valid) interface:

```scala
class DecoupledIO[T <: Data](data: T) extends Bundle {
  val ready = Bool(INPUT)
  val valid = Bool(OUTPUT)
  val bits  = data.clone.asOutput
}
```

This template can then be used to add a handshaking protocol to any
set of signals:

```scala
class DecoupledDemo extends DecoupledIO(new DataBundle)
```

The FIFO interface can be now be simplified as follows: 

```scala
class Fifo[T <: Data](data: T, n: Int) extends Module {
  val io = new Bundle {
    val enq = new DecoupledIO(data).flip
    val deq = new DecoupledIO(data)
  }
  ...
}
```

[Prev(Muxes and Input Selection)](Muxes and Input Selection) [Next(Multiple Clock Domains)](Multiple Clock Domains)