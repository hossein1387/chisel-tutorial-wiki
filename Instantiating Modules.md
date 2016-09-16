## Module Instantiation

Like other hardware description languages, Chisel allows fairly straightforward module instantiation to enable modularity and hierarchy.
In Chisel, instantiating a Module class is the equivalent to instantiating a module in Verilog.
To do this, we simply use a call to `Module` with the module created with the Scala `new` keyword in order to indicate that we are instantiation a new module.
We want to make sure we assign this to a value so that we can reference its input and outputs, which we also need to connect.

For example, suppose we would like to construct a 4-bit adder using multiple copies of the  `FullAdder` module, as shown in the Figure 1.

![Figure 1: Block Diagram of 4-Bit Adder](figs/4_Bit_Adder.jpg)

The Chisel source code is shown below.

``` scala
// A 4-bit adder with carry in and carry out
class Adder4 extends Module {
  val io = new Bundle {
    val A    = UInt(INPUT, 4)
    val B    = UInt(INPUT, 4)
    val Cin  = UInt(INPUT, 1)
    val Sum  = UInt(OUTPUT, 4)
    val Cout = UInt(OUTPUT, 1)
  }
  // Adder for bit 0
  val Adder0 = Module(new FullAdder())
  Adder0.io.a   := io.A(0)
  Adder0.io.b   := io.B(0)
  Adder0.io.cin := io.Cin
  val s0 = Adder0.io.sum
  // Adder for bit 1
  val Adder1 = Module(new FullAdder())
  Adder1.io.a   := io.A(1)
  Adder1.io.b   := io.B(1)
  Adder1.io.cin := Adder0.io.cout
  val s1 = Cat(Adder1.io.sum, s0)
  // Adder for bit 2
  val Adder2 = Module(new FullAdder())
  Adder2.io.a   := io.A(2)
  Adder2.io.b   := io.B(2)
  Adder2.io.cin := Adder1.io.cout
  val s2 = Cat(Adder2.io.sum, s1)
  // Adder for bit 3
  val Adder3 = Module(new FullAdder())
  Adder3.io.a   := io.A(3)
  Adder3.io.b   := io.B(3)
  Adder3.io.cin := Adder2.io.cout
  io.Sum  := Cat(Adder3.io.sum, s2).toUInt()
  io.Cout := Adder3.io.cout
}
```

In this example, notice how when referencing each module I/O we must first reference `io` that contains the ports for the I/Os.
Again, note how all assignments to the module I/Os use a reassignment operator `:=`.
When instantiating modules, it is important to make sure that you connect all the input and output ports.
If a port is not connected, the Chisel compiler may optimize away portions of your design that it finds unnecessary due to the unconnected ports and throw errors or warnings.

## The Vec Class

The `Vec` class allows you to create an indexable vector in Chisel which can be filled with any expression that returns a chisel data type.
The general syntax for a `Vec` declaration is given by:
``` scala
val myVec = 
  Vec(Seq.fill( <number of elements> ) { <data type> })
```
Where `<number of elements>` corresponds to how long the vector is and `<data type>` corresponds to what type of class the vector contains.

For instance, if we wanted to instantiate a 10 entry vector of 5 bit UInt values, we would use:

``` scala
val ufix5_vec10 := Vec(Seq.fill(10) { UInt(width = 5) })
```

If we want to define a vector of registers...

``` scala
val reg_vec32 := Vec(Seq.fill(32){ Reg() })
```

In order to assign to a particular value of the `Vec`, we simply assign the target value to the vector at a specified index.
For instance, if we wanted to assign a UInt value of zero to the first register in the above example, the assignment would look like:

``` scala
reg_vec32(0) := UInt(0)
```

To access a particular element in the vector at some index, we specify the index of the vector.
For example, to extract the 5th element of the register vector in the above example and assign it to some value `reg5`, the assignment would look like:

``` scala
val reg5 = reg_vec(5)
```

The syntax for the `Vec` class is slightly different when instantiating a vector of modules.When instantiating a vector of modules the data type that is specified in the {} braces is slightly different than the usualy primitive types.
To specify a vector of modules, we use the `io` bundle when specifying the type of the vector.
For example, in order to specify a `Vec` with 16 modules , say `FullAdder`s in this case, we would use the following declaration:

``` scala
val FullAdders = 
  Vec(Seq.fill(16){ Module(new FullAdder()).io })
```

Notice we use the keyword `new` in the vector definition before the module name `FullAdder`.
For how to actually access the `io` on the vector modules, refer to the next section.

#### Vec Shift Reg

The next assignment is to construct a simple bit shift register.
The following is a the template from `$TUT_DIR/src/main/scala/problems/VecShiftRegisterSimple.scala`:

``` scala
class VecShiftRegisterSimple extends Module {
  val io = new Bundle {
    val in  = UInt(INPUT,  8)
    val out = UInt(OUTPUT, 8)
  }
  val delays = Vec(Seq.fill(4){ Reg(UInt(width = 8)) })
  ...
  io.out := UInt(0)
}
```

where `out` is a four cycle delayed copy of values on `in`.

## Parametrization

In the previous Adder example, we explicitly instantiated four different copies of a `FullAdder` and wired up the ports.
But suppose we want to generalize this structure to an n-bit adder.
Like Verilog, Chisel allows you to pass parameters to specify certain aspects of your design.
In order to do this, we add a parameter in the Module declaration to our Chisel definition.
For a carry ripple adder, we would like to parametrize the width to some integer value `n` as shown in the following example:

``` scala

// A n-bit adder with carry in and carry out
class Adder(n: Int) extends Module {
  val io = new Bundle {
    val A    = UInt(INPUT, n)
    val B    = UInt(INPUT, n)
    val Cin  = UInt(INPUT, 1)
    val Sum  = UInt(OUTPUT, n)
    val Cout = UInt(OUTPUT, 1)
  }
  // create a vector of FullAdders
  val FAs = Vec(Seq.fill(n){ Module(new FullAdder()).io })

  // define carry and sum wires
  val carry = Vec(Seq.fill(n+1){ UInt(width = 1) })
  val sum   = Vec(Seq.fill(n){ Bool() } )

  // first carry is the top level carry in
  carry(0) := io.Cin

  // wire up the ports of the full adders
  for(i <- 0 until n) {
     FAs(i).a   := io.A(i)
     FAs(i).b   := io.B(i)
     FAs(i).cin := carry(i)
     carry(i+1) := FAs(i).cout
     sum(i)     := FAs(i).sum.toBool()
  }
  io.Sum  := sum.toBits().toUInt()
  io.Cout := carry(n)
}

```

Note that in this example, we keep track of the sum output in a `Vec` of `Bool`s.
This is because Chisel does not support bit assignment directly.
Thus in order to get the n-bit wide `sum` in the above example, we use an n-bit wide `Vec` of `Bool`s and then cast it to a UInt().
Note that it must first be casted to the `Bits()` type before casting it to `UInt()`.

You will notice that modules are instantiated in a Vec class which allows us to iterate through each module when assigning the ports connections to each `FullAdder`.
This is similar to the generate statement in Verilog.
However, you will see in more advanced tutorials that Chisel can offer more powerful variations.

Instantiating a parametrized module is very similar to instantiating an unparametrized module except that we must provide arguments for the parameter values.
For instance, if we wanted to instantiate a 4-bit version of the `Adder` module we defined above, it would look like:

``` scala
val adder4 = Module(new Adder(4))
```

% Martin: Shouldn't this be called ``named'' parameter instead of ``explicite''?
We can also instantiate the `Adder` by explicitly specifying the value of it parameter `n` like the this:

``` scala
val adder4 = Module(new Adder(n = 4))
```

Explicitly specifying the parameter is useful when you have a module with multiple parameters.
Suppose you have a parametrized FIFO module with the following module definition:

``` scala
class FIFO(width: Int, depth: Int) extends Module {...}
```

You can explicitly specify the parameter values in any order:

``` scala
val fifo1 = Module(new FIFO(16, 32))
val fifo2 = Module(new FIFO(width = 16, depth = 32))
val fifo3 = Module(new FIFO(depth = 32, width = 16))
```

All of the above definitions pass the same parameters to the FIFO module.
Notice that when you explicitly assign the parameter values, they can occur in any order you want such as the definition for fifo3.

## Advanced Parametrization

Although parameters can be passed explicitly through a Module's constructor, this technique does not scale when parameterizing large designs with many generic components.
For a more detailed explanation of why a better parameterization method is needed, please see the Advanced Parameterization Manual.
In addition, this manual explains heuristics for how to organize and parameterize large designs, which we highly recommend one reads prior to using this functionality in a design.
The following, however, is a basic introduction.

Every Module has its own `params` object, which acts as a dictionary.
Querying this object is shown below.

``` scala
val width = params[Int]('width')
```

If `params` is queried and no parameter matches the query, Chisel throws a `ParameterUndefinedException`.
Notice the query return type must be provided.

When a parent Module creates a child Module, the parent's `params` object is automatically cloned and passed to the child.
In the following example, suppose the parent's params object returns `10` when queried for width.
Because the `Parent` `params` object is automatically cloned for `Child`, the `Child` query also returns `10`.

``` scala
class Parent extends Module {
  val io = new Bundle { ...
}
  val width = params[Int]('width') // returns 10
  // create child Module implicitly passing params
  val child = Module(new Child) 
}
class Child extends Module {
  val io = new Bundle { ...
}
  val width = params[Int]('width') // returns 10
}
```

Suppose a parent Module wants to override or add parameters to its child's `params` object.
This case requires adding a partial function (a Scala way of defining key-value mappings) to the `Child` Module constructor:

``` scala
class Parent extends Module {
  val io = new Bundle { ...
}
  val width = params[Int]('width') // returns 10
  val n = params[Int]('n') // returns 20
  // Partial function is added to Module constructor
  val child = Module(new Child,{'n' => 40})
}
class Child extends Module {
  val io = new Bundle { ...
}
  val width = params[Int]('width') // returns 10
  val n = params[Int]('n') // returns 40
}
```

An example which is impossible to do with simple parameterization, but simple with the advanced parameterization, is when using a generic `Mesh` generator with a custom `MyRouter` Module.
The existing source code might look like:

``` scala
class Mesh(routerConstructor: () => Router, n:Int) extends Module {
  val io = new Bundle { ...
}
  val routers = Vec(Seq.fill(n){Module(routerConstructor())})
  hookUpRouters(routers)
}
```

However, our custom `MyRouter` Module requires a parameter, `RoutingFunction` that we want to sweep for a design space evaluation.
Using the simple parameterization method would require a change to the `Mesh` Module's constructor to include `RoutingFunction`.


Alternatively, one can use the `params` object to implicitly pass the `RoutingFunction`:

``` scala
class MyRouter extends Module {
  val io = new Bundle { ...
}
  val myRoutingFunction = params[RoutingFunction]('r')
  ...
}
class Top extends Module {
  val io = new Bundle { ...
}
  val mesh = Module(new Mesh(() => new MyRouter),{'r' => new RoutingFunction})
}
```

For more advanced uses, tips, and tricks, please see the Advanced Parameterization Manual, in the documentation section of the website or the doc/parameters directory of the git repo.
This is an evolving area and the documentation may not be as up-to-date as the code.

## Built In Primitives

Like other HDL, Chisel provides some very basic primitives.
These are constructs that are built in to the Chisel compiler and come for free.
The Reg, UInt, and Bundle classes are such primitives that have already been covered.
Unlike Module instantiations, primitive do not require explicit connections of their io ports to use.
Other useful primitive types include the Mem and Vec classes which will be discussed in a more advanced tutorial.
In this tutorial we explore the use of the `Mux` primitive.

#### The Mux Class

The `Mux` primitive is a two input multiplexer.
In order to use the `Mux` we first need to define the expected syntax of the `Mux` class.
As with any two input multiplexer, it takes three inputs and one output.
Two of the inputs correspond to the data values `A` and `B` that we would like to select which can be any width and data type as long as they are the same.
The first input `select`, which is a Bool type, determines which one to output.
A `select` value of `true` will output the value `A`, while a `select` value of `false` will pass `B`.

``` scala
val out = Mux(select, A, B)
```

Thus if `A=10`, `B=14`, and `select` was `true`, the value of `out` would be assigned 10.
Notice how using the `Mux` primitive type abstracts away the logic structures required if we had wanted to implement the multiplexer explicitly.

#### Parameterized Width Adder

The next assignment is to construct an adder with a parameterized width and using the built in addition operator `+`.
The following is a the template from `$TUT_DIR/src/main/scala/problems/Adder.scala`:

``` scala
class Adder(val w: Int) extends Module {
  val io = new Bundle {
    val in0 = UInt(INPUT,  1)
    val in1 = UInt(INPUT,  1)
    val out = UInt(OUTPUT, 1)
  }
  ...
  io.out := UInt(0)
}
```

where `out` is sum of `w` width unsigned inputs `in0` and `in1`.
 
Notice how `val` is added to the width parameter value to allow the width to be accessible from the tester as a field of the adder module object.

Edit your copy of `$TUT_DIR/src/main/scala/problems/Adder.scala` and run:

``` bash
./run-problem.sh Adder
```

until your circuit passes the tests.

<!---
% Martin: I would drop the following as it is just confusing in a tutorial
% state simple that there is a Mux primitive and that is fine.

%The instantiation would look something like this:
%
%``` scala
%// where n is the width of A and m is the width of B
%val mux = Module(new Mux(n, m))
%mux.io.select := select
%mux.io.A      := A
%mux.io.B      := B
%val out = mux.io.out
%```
%
%We see that clearly it is much cleaner to use the primitive `Mux` type instead of trying to write and implement our own general multiplexer since the `Mux` type does all the wiring for you.


%## Exercises
%
%#### n-bit Subtractor
%
%Earlier in this tutorial we demonstarted how to parametrize and instantiate an n-bit ripple adder.
<Finish this>
%
--->
