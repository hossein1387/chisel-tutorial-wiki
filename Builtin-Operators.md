### List of operators
Chisel defines a set of hardware operators

| Operation        | Explanation |
| ---------        | ---------           |
| **Bitwise operators**                     | **Valid on:** SInt, UInt, Bool    |
| val invertedX = ~x                        | Bitwise NOT |
| val hiBits = x & UInt("h_ffff_0000")      | AND reduction                     |     
| val flagsOut = flagsIn | overflow         | OR reduction                      |   
| val flagsOut = flagsIn ^ toggle           | XOR reduction                     |  
| **Bitwise reductions.**                   | **Valid on:** SInt and UInt. Returns Bool. |
| val allSet = andR(x)                      | AND reduction                     |   
| val anySet = orR(x)                       | OR reduction                      |  
| val parity = xorR(x)                      | XOR reduction                     |   
| **Equality comparison.**                  | **Valid on:** SInt, UInt, and Bool. Returns Bool. |
| val equ = x === y                         | Equality                          |
| val neq = x =/= y                         | Inequality                        |
| **Shifts**                                | **Valid on:** SInt and UInt       |
| val twoToTheX = 1.S << x                  | Logical shift left                |
| val hiBits = x >> 16.U                    | Right shift (logical on UInt and& arithmetic on SInt). |  
| **Bitfield manipulation**                 | **Valid on:** SInt, UInt, and Bool. |
| val xLSB = x(0)                           | Extract single bit, LSB has index 0.     |  
| val xTopNibble = x(15,12)                 | Extract bit field from end to start bit position.     |            
| val usDebt = Fill(3, "hA".U)              | Replicate a bit string multiple times.     |               
| val float = Cat(sign,exponent,mantissa)   | Concatenates bit fields, with first argument on left.     |
| **Logical Operations**                    | **Valid on:** Bool                                                                             
| val sleep = !busy                         | Logical NOT                       |       
| val hit = tagMatch && valid               | Logical AND                       |                 
| val stall = src1busy \|\| src2busy          | Logical OR                        |                     
| val out = Mux(sel, inTrue, inFalse)       | Two-input mux where sel is a Bool |                                               
| **Arithmetic operations**                 | **Valid on Nums:** SInt and UInt.  |
| val sum = a + b                           | Addition                           |           
| val diff = a - b                          | Subtraction                        |            
| val prod = a * b                          | Multiplication                     |            
| val div = a / b                           | Division                           |           
| val mod = a % b                           | Modulus                            |           
| **Arithmetic comparisons**                | **Valid on Nums:** SInt and UInt. Returns Bool. |
| val gt = a > b                            | Greater than                       |      
| val gte = a >= b                          | Greater than or equal              |                 
| val lt = a < b                            | Less than                          |   
| val lte = a <= b                          | Less than or equal                 |              


### BitWidth Inference
Users are required to set bit widths of ports and registers, but otherwise,
bit widths on wires are automatically inferred unless set manually by the user.
The bit-width inference engine starts from the graph's input ports and 
calculates node output bit widths from their respective input bit widths according to the following set of rules:

| operation        | bit width           |
| ---------        | ---------           |
| ```z = x + y```        | ```wz = max(w x, wy)```   |
| ```z = x - y```        | ```wz = max(wx, wy)```    |
| ```z = x & y```        | ```wz = min(wx, wy)```    |
| ```z = Mux(c, x, y)``` | ```wz = max(wx, wy)```    |
| ```z = w * y```        | ```wz = wx + w```y        |
| ```z = x << n```       | ```wz = wx + maxNum(n)``` |
| ```z = x >> n```       | ```wz = wx - minNum(n)``` |
| ```z = Cat(x, y)```    | ```wz = wx + w```y        |
| ```z = Fill(n, x)```   | ```wz = wx * maxNum(n)``` |
>where for instance *wz* is the bit width of wire *z*, and the *&*
rule applies to all bitwise logical operations.

The bit width inference process continues until no bit width changes.
Except for right shifts by known constant amounts, the bit-width
inference rules specify output bit widths that are never smaller than
the input bit widths, and thus, output bit widths either grow or stay
the same.  Furthermore, the width of a register must be specified by
the user either explicitly or from the bit width of the reset value or
the *next* parameter.
From these two requirements, we can show that the bit width inference
process will converge to a fixed point.

>Our choice of operator names was constrained by the Scala language.
We have to use triple equals```===``` for equality and ```=/=```
for inequality to allow the
native Scala equals operator to remain usable.

>We are also planning to add further operators that constrain bitwidth
to the larger of the two inputs.

[Prev (Combinational Circuits)](Combinational Circuits)  [Next (Functional Abstraction)](Functional Abstraction)