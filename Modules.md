Chisel *modules* are very similar to Verilog *modules* in
defining a hierarchical structure in the generated circuit.

The hierarchical module namespace is accessible in downstream tools
to aid in debugging and physical layout.  A user-defined module is
defined as a *class* which:

 - inherits from ```Module```,
 - contains an interface stored in a port field named ```io```, and
 - wires together subcircuits in its constructor.

As an example, consider defining your own two-input multiplexer as a
module:
```scala
class Mux2 extends Module {
  val io = new Bundle{
    val sel = UInt(INPUT, 1)
    val in0 = UInt(INPUT, 1)
    val in1 = UInt(INPUT, 1)
    val out = UInt(OUTPUT, 1)
  }
  io.out := (io.sel & io.in1) | (~io.sel & io.in0)
}
```

The wiring interface to a module is a collection of ports in the
form of a ```Bundle```.  The interface to the module is defined
through a field named ```io```.  For ```Mux2```, ```io``` is
defined as a bundle with four fields, one for each multiplexer port.

The ```:=```ssignment operator, used here in the body of the
definition, is a special operator in Chisel that wires the input of
left-hand side to the output of the right-hand side.

### Module Hierarchy

We can now construct circuit hierarchies, where we build larger modules out
of smaller sub-modules.  For example, we can build a 4-input
multiplexer module in terms of the ```Mux2``` module by wiring
together three 2-input multiplexers:

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

We again define the module interface as ```io``` and wire up the
inputs and outputs.  In this case, we create three ```Mux2```
children modules, using the ```Module``` constructor function and 
the Scala ```new``` keyword to create a
new object.  We then wire them up to one another and to the ports of
the ```Mux4``` interface.



