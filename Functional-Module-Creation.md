Objects in Scala have a pre-existing creation function (method) called `apply`. 
When an object is used as value in an expression (which basically means that the constructor was called), this method determines the returned value.
When dealing with hardware modules, one would expected the module output to be representative of the hardware module's functionality.
Therefore, we would sometimes like the module output to be the value returned when using the object as a value in an expression.
Since hardware modules are represented as Scala objects, this can be done by defining the object's \code{apply} method to return the module's output.
This can be referred to as creating a functional interface for module construction.
If we apply this on the standard mux2 example, we would to return the mux2 output ports when we used mux2 in an expression.
Implementing this requires building a constructor that takes multiplexer inputs as parameters and returns the multiplexer output:

```scala
object Mux2 {
  def apply (sel: UInt, in0: UInt, in1: UInt) = {
    val m = new Mux2()
    m.io.in0 := in0
    m.io.in1 := in1
    m.io.sel := sel
    m.io.out`
  }
}
```

As we can see in the code example, we defined the `apply` method to take the mux2 inputs as the method parameters, and return the mux2 output as
the function's return value.

by defining modules in this way, it is easier to later implement larger and more complex version of this regular modules.
for example,previously implemented mux4 explicitly like this:

```scala
class Mux4 extends Module {
  val io = new Bundle {
    val in0 = UInt(INPUT, 1)
    val in1 = UInt(INPUT, 1)
    val in2 = UInt(INPUT, 1)
    val in3 = UInt(INPUT, 1)
    val sel = UInt(INPUT, 2)
    val out = UInt(OUTPUT, 1)
  }
  val m0 = Module(new Mux2())
  m0.io.sel := io.sel(0) 
  m0.io.in0 := io.in0; m0.io.in1 := io.in1

  val m1 = Module(new Mux2())
  m1.io.sel := io.sel(0) 
  m1.io.in0 := io.in2; m1.io.in1 := io.in3

  val m3 = Module(new Mux2())
  m3.io.sel := io.sel(1) 
  m3.io.in0 := m0.io.out; m3.io.in1 := m1.io.out

  io.out := m3.io.out
}
```

However, by using the creation function we redefined for mux2, we can now use the mux2 outputs as values of the modules themselves
when writing the mux4 output expression:

```scala
class Mux4 extends Module {
  val io = new Bundle {
    val in0 = UInt(INPUT, 1)
    val in1 = UInt(INPUT, 1)
    val in2 = UInt(INPUT, 1)
    val in3 = UInt(INPUT, 1)
    val sel = UInt(INPUT, 2)
    val out = UInt(OUTPUT, 1)
  }
  io.out := Mux2(io.sel(1),
                 Mux2(io.sel(0), io.in0, io.in1),
                 Mux2(io.sel(0), io.in2, io.in3))
}
```

This allows to write more intuitively readable hardware connection descriptions, which are similar to software expression evaluation.

[Next (Muxes and Input Selection)](Muxes and Input Selection)