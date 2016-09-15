### List of operators

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

