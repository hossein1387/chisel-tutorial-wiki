_This section is advanced and can be skipped at first reading._

Scala is a strongly typed language and uses parameterized types to specify generic functions and classes.
In this section, we show how Chisel users can de-fine their own reusable functions and classes using parameterized classes.

# Parameterized Functions

Earlier we defined `Mux2` on `Bool`, but now we show how we can define a generic multiplexer function.
We define this function as taking a boolean condition and con and alt arguments (corresponding to then and else expressions) of type `T`:

```scala
def Mux[T <: Bits](c: Bool, con: T, alt: T): T = { ... }
```

where `T` is required to be a subclass of `Bits`.
Scala ensures that in each usage of `Mux`, it can find a common superclass of the actual con and alt argument types,
otherwise it causes a Scala compilation type error.
For example,