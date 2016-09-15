| operation        | bit width           |   
| z = x + y        | wz = max(w x, wy)   |   
| z = x - y        | wz = max(wx, wy)    |   
| z = x & y        | wz = min(wx, wy)    |   
| z = Mux(c, x, y) | wz = max(wx, wy)    |   
| z = w * y        | wz = wx + wy        |   
| z = x << n       | wz = wx + maxNum(n) |   
| z = x >> n       | wz = wx - minNum(n) |   
| z = Cat(x, y)    | wz = wx + wy        |   
| z = Fill(n, x)   | wz = wx * maxNum(n) |   
                                             