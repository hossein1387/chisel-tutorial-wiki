
Now that we have defined modules, we will discuss how we actually run and test a circuit.
Testing is a crucial part of circuit design, and thus in Chisel we provide a mechanism for testing circuits by providing test vectors within Scala using tester method calls which binds a tester to a module and allows users to write tests using the given debug protocol. In particular, users utilize:
* poke to set input port and state values
* step to execute the circuit one time unit
* peek to read port and state values
* expect to compare peeked circuit values to expected arguments.

For example, in the following:
```scala
class Mux2Tests(c: Mux2, b: Option[TesterBackend] = None) extends PeekPokeTester(c, _backend=b) {
  val n = pow(2, 3).toInt 
  for (s <- 0 until 2) {
    for (i0 <- 0 until 2) { for (i1 <- 0 until 2) {
      poke(c.io.sel, s)
      poke(c.io.in1, i1)
      poke(c.io.in0, i0)
      step(1)
      expect(c.io.out, (if (s == 1) i1 else i0))
    }
  }
}
```
assignments for each input of ```Mux2``` are set to the appropriate values using poke. For this particular example, we are testing the ```Mux2``` by hardcoding the inputs to some known values and checking if the output corresponds to the known one. To do this, on each iteration we generate appropriate inputs to the module and tell the simulation to assign these values to the inputs of the device we are testing c, step the circuit 1 clock cycle, and test the expected value. Steps are necessary to update registers and the combinational logic driven by registers. For pure combinational paths, poke alone is sufficient to update all combinational paths connected to the poked input wire.


[Next(State Elements)]  (State Elements)
