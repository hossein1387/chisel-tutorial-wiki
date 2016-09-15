# Running Examples
Now that we have defined modules, we will discuss how we actually run and test a circuit.
Testing is a crucial part of circuit design, and thus in Chisel we provide a mechanism for testing circuits by providing test vectors within Scala using tester method calls which binds a tester to a module and allows users to write tests using the given debug protocol. In particular, users utilize:
* poke to set input port and state values,
* step to execute the circuit one time unit,
* peek to read port and state values, and
* expect to compare peeked circuit values to ex- pected arguments.