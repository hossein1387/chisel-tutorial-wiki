When you first define a value in Chisel, we use the *=*+ operator in order to tell Chisel to allocate the value for the first time. On every subsequent reassignment to the value, we must use a *:=* when reassigning the value.

Since we are constructing a digital circuit, the notion of reassignment does not make much sense since connections between circuit nodes only need to be specified once. However, there are some cases when we will need to perform reassignment to a value in Chisel since it is compiled sequentially unlike Verilog. Thus it may be necessary to perform reassignment when a value or connection is not known until later in the Chisel source. 

A simple example of when reassignment is necessary is in the construction of the top level I/O for your module; the values of the output are not immediately known at the time of declaration.

Consider the simple *FullAdder* circuit from previous tutorial that determines the sum *sum* and carry out *cout* given two values *a* and *b*, and a carry in *cin*.

```scala
class FullAdder extends Module {
  val io = new Bundle {
    val a    = UInt(INPUT, 1)
    val b    = UInt(INPUT, 1)
    val cin  = UInt(INPUT, 1)
    val sum  = UInt(OUTPUT, 1)
    val cout = UInt(OUTPUT, 1)
  }

  // Generate the sum
  val a_xor_b = io.a ^ io.b
  io.sum := a_xor_b ^ io.cin
  // Generate the carry
  val a_and_b = io.a & io.b
  val b_and_cin = io.b & io.cin
  val a_and_cin = io.a & io.cin
  io.cout := a_and_b | b_and_cin | a_and_cin
}
```


In the previous examples we have been using the UInt type which is an unsigned integer as the type for all of our values. For many of the basic computations in Chisel the UInt class is sufficient. The following example shows some of the commonly used UInt operations in the context of a simple *ALU*\footnote{We ignore overflow and underflow in this example.}:

```scala
class BasicALU extends Module {
  val io = new Bundle {
    val a = UInt(INPUT, 4)
    val b = UInt(INPUT, 4)
    val opcode = UInt(INPUT, 4)
    val out = UInt(OUTPUT, 4)
  }
  io.out := 0.U //THIS SEEMS LIKE A HACK/BUG
  when (io.opcode === 0.U) {
    io.out := io.a //pass A
  } .elsewhen (io.opcode === 1.U) {
    io.out := io.b //pass B
  } .elsewhen (io.opcode === 2.U) {
    io.out := io.a + 1.U //increment A by 1
  } .elsewhen (io.opcode === 3.U) {
    io.out := io.a - 1.U //increment B by 1
  } .elsewhen (io.opcode === 4.U) {
    io.out := io.a + 4.U //increment A by 4
  } .elsewhen (io.opcode === 5.U) {
    io.out := io.a - 4.U //decrement A by 4
  } .elsewhen (io.opcode === 6.U) {
    io.out := io.a + io.b //add A and B
  } .elsewhen (io.opcode === 7.U) {
    io.out := io.a - io.b //subtract B from A
  } .elsewhen (io.opcode === 8.U) {
    io.out := io.a < io.b //set on A less than B
  } .otherwise { 
    io.out :=  (io.a === io.b).asUInt() //set on A equal to B
  }
}
```

You will notice that there are multiple reassignments to *io.output*+ inside a *when*+ block which indicates that the value of *io.output*+ can take many different values depending on the *io.opcode*+ in this example. Also notice that in order to specify constants to add to our operands, we must also specify them as a UInt type as UInt operations on different type operands is not allowed.

```scala
// Specify that 1 is a UInt type
io.output := io.a + UInt(1) 
```

A list of commonly used UInt operations is given in the table below:

[Prev (The Basics)](The Basics)  [Next (Instantiating Modules)](Instantiating Modules)