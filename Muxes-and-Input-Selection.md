Selecting inputs is very useful in hardware description, and therefore Chisel provides several built-in generic input-selection implementations.
The first one is `Mux`. This is a 2-input selector. Unlike the `Mux2` example which was presented previously, the built-in `Mux` allows 
the inputs (`in0` and `in1`) to be any datatype as long as they are the same subclass of `Data`.

by using the functional module creation feature presented in the previous section, we can create multi-input selector in a simple way:

```scala
Mux(c1, a, Mux(c2, b, Mux(..., default)))
```

However, this is not necessary since Chisel also provides the built-in \code{MuxCase}, which implements that exact feature.
`MuxCase` is an n-way `Mux`, which can be used as follows:

```scala
MuxCase(default, Array(c1 -> a, c2 -> b, ...))
```
 
Where each selection dependency is represented as a tuple in a Scala
array [ condition -> selected_input_port ].


Chisel also provides `MuxLookup` which is an n-way indexed multiplexer:

```scala
MuxLookup(idx, default, 
          Array(0.U -> a, 1.U -> b, ...))
```

This is the same as a `MuxCase`, where the conditions are all index based selection:

```scala
MuxCase(default, 
        Array((idx === 0.U) -> a,
              (idx === 1.U) -> b, ...))
```

Note that the conditions/cases/selectors (eg. c1, c2) must be in parentheses.

[Next (Polymorphism and Parameterization)](Polymorphism and Parameterization)