# Interfaces & Bulk Connections

For more sophisticated modules it is often useful to define and instantiate interface classes while defining the IO for a module. First and foremost, interface classes promote reuse allowing users to capture once and for all common interfaces in a useful form.

Secondly, interfaces allow users to dramatically reduce wiring by supporting bulk connections between producer and consumer modules. Finally, users can make changes in large interfaces in one place reducing the number of updates required when adding or removing pieces of the interface.

## Ports: Subclasses & Nesting

As we saw earlier, users can define their own interfaces by defining a class that subclasses Bundle. For example, a user could define a simple link for hand-shaking data as follows:

```scala
class SimpleLink extends Bundle {
  val data = UInt(16, OUTPUT)
  val valid = Bool(OUTPUT)
}
```

We can then extend SimpleLink by adding parity bits using bundle inheritance:
```scala
class PLink extends SimpleLink {
  val parity = UInt(5, OUTPUT)
}
```
In general, users can organize their interfaces into hierarchies using inheritance.

From there we can define a filter interface by nesting two PLinks into a new FilterIO bundle:
```scala
class FilterIO extends Bundle {
  val x = new PLink().flip
  val y = new PLink()
}
```
where flip recursively changes the “gender” of a bundle, changing input to output and output to input.

We can now define a filter by defining a filter class extending module:
```scala
class Filter extends Module {
  val io = new FilterIO()
  ...
}
```
where the io field contains FilterIO.

## Bundle Vectors

Beyond single elements, vectors of elements form richer hierarchical interfaces. For example, in order to create a crossbar with a vector of inputs, producing a vector of outputs, and selected by a UInt input, we utilize the Vec constructor:
```scala
class CrossbarIo(n: Int) extends Bundle {
  val in = Vec(n, new PLink().flip())
  val sel = UInt(INPUT, sizeof(n))
  val out = Vec(n, new PLink())
}
```
where Vec takes a size as the first argument and a block returning a port as the second argument.

## Bulk Connections

We can now compose two filters into a filter block as follows:
```scala
class Block extends Module {
  val io = new FilterIO()
  val f1 = Module(new Filter())
  val f2 = Module(new Filter())
  f1.io.x <> io.x
  f1.io.y <> f2.io.x
  f2.io.y <> io.y
}
```
where <> bulk connects interfaces of opposite gender between sibling modules or interfaces of the same gender between parent/child modules.

Bulk connections connect leaf ports of the same name to each other. If the names do not match or are missing, Chisel does not generate a connection.