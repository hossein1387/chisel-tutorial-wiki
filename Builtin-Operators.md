### List of operators

| operation        | bit width           |
| ---------        | ---------           |
| **Bitwise operators**                     | **Valid on:** SInt, UInt, Bool    |
| val invertedX = ~x                        | Bitwise NOT |
| val allSet = andR(x)                      | AND reduction                     |     
| val anySet = orR(x)                       | OR reduction                      |   
| val parity = xorR(x)                      | XOR reduction                     |  
| **Bitwise reductions.**                   | **Valid on:** SInt and UInt. Returns Bool. |
| val allSet = andR(x)                      | AND reduction                     |   
| val anySet = orR(x)                       | OR reduction                      |  
| val parity = xorR(x)                      | XOR reduction                     |   
| **Equality comparison.** |                | **Valid on:** SInt, UInt, and Bool. Returns Bool. |
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
| --- |
| **Logical Operations**                    | **Valid on:** Bool                                                                             
| val sleep = !busy                         | Logical NOT                       |       
| val hit = tagMatch && valid               | Logical AND                       |                 
| val stall = src1busy \|\| src2busy          | Logical OR                        |                     
| val out = Mux(sel, inTrue, inFalse)       | Two-input mux where sel is a Bool |                                               


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

