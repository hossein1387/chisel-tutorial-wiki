# Interfaces & Bulk Connections

For more sophisticated modules it is often useful to define and instantiate interface classes while defining the IO for a module. First and foremost, interface classes promote reuse allowing users to capture once and for all common interfaces in a useful form.

Secondly, interfaces allow users to dramatically reduce wiring by supporting bulk connections between producer and consumer modules. Finally, users can make changes in large interfaces in one place reducing the number of updates required when adding or removing pieces of the interface.

## Ports: Subclasses & Nesting

As we saw earlier, users can define their own interfaces by defining a class that subclasses Bundle. For example, a user could define a simple link for hand-shaking data as follows:

```
class SimpleLink extends Bundle {
  val data = UInt(16, OUTPUT)
  val valid = Bool(OUTPUT)
}
```

We can then extend SimpleLink by adding parity bits using bundle inheritance:

class PLink extends SimpleLink {
  val parity = UInt(5, OUTPUT)
}

In general, users can organize their interfaces into hierarchies using inheritance.

From there we can define a filter interface by nesting two PLinks into a new FilterIO bundle:
```
class FilterIO extends Bundle {
  val x = new PLink().flip
  val y = new PLink()
}
```
where flip recursively changes the “gender” of a bundle, changing input to output and output to input.

We can now define a filter by defining a filter class extending module:
```
class Filter extends Module {
  val io = new FilterIO()
  ...
}
```
where the io field contains FilterIO.

## Bundle Vectors

Beyond single elements, vectors of elements form richer hierarchical interfaces. For example, in order to create a crossbar with a vector of inputs, producing a vector of outputs, and selected by a UInt input, we utilize the Vec constructor:
```
class CrossbarIo(n: Int) extends Bundle {
  val in = Vec(n, new PLink().flip())
  val sel = UInt(INPUT, sizeof(n))
  val out = Vec(n, new PLink())
}
```
where Vec takes a size as the first argument and a block returning a port as the second argument.

## Bulk Connections

We can now compose two filters into a filter block as follows:
```
class Block extends Module {
  val io = new FilterIO()
  val f1 = Module(new Filter())
  val f2 = Module(new Filter())
  f1.io.x <> io.x
  f1.io.y <> f2.io.x
  f2.io.y <> io.y
}
```
where <> bulk connects interfaces of opposite gender between sibling modules or interfaces of same gender between parent/child modules. Bulk connections connect leaf ports of the same name to each other.

After all connections are made and the circuit is being elaborated, Chisel warns users if ports have other than exactly one connection to them.

13.4 Interface Views

Consider a simple CPU consisting of control path and data path submodules and host and memory interfaces shown in Figure ??. In this CPU we can see that the control path and data path each connect only to a part of the instruction and data memory interfaces. Chisel allows users to do this with partial fulfillment of interfaces. A user first defines the complete interface to a ROM and Mem as follows:
```
class RomIo extends Bundle {
  val isVal = Bool(INPUT)
  val raddr = UInt(INPUT, 32)
  val rdata = UInt(OUTPUT, 32)
}

class RamIo extends RomIo {
  val isWr = Bool(INPUT)
  val wdata = UInt(INPUT, 32)
}
```
Now the control path can build an interface in terms of these interfaces:
```
class CpathIo extends Bundle {
  val imem = RomIo().flip()
  val dmem = RamIo().flip()
  ...
}
```
and the control and data path modules can be built by partially assigning to this interfaces as follows:
```
class Cpath extends Module {
  val io = new CpathIo()
  ...
  io.imem.isVal := ...
  io.dmem.isVal := ...
  io.dmem.isWr := ...
  ...
}

class Dpath extends Module {
  val io = new DpathIo()
  ...
  io.imem.raddr := ...
  io.dmem.raddr := ...
  io.dmem.wdata := ...
  ...
}
```
We can now wire up the CPU using bulk connects as we would with other bundles:
```
class Cpu extends Module {
  val io = new CpuIo()
  val c = Module(new CtlPath())
  val d = Module(new DatPath())
  c.io.ctl <> d.io.ctl
  c.io.dat <> d.io.dat
  c.io.imem <> io.imem
  d.io.imem <> io.imem
  c.io.dmem <> io.dmem
  d.io.dmem <> io.dmem
  d.io.host <> io.host
}
```
Repeated bulk connections of partially assigned control and data path interfaces completely connect up the CPU interface.