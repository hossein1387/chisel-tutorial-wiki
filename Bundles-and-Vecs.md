```Bundle``` and ```Vec``` are classes that allow the user to expand
the set of Chisel datatypes with aggregates of other types.

Bundles group together several named fields of potentially different
types into a coherent unit, much like a ```struct``` in C. Users
define their own bundles by defining a class as a subclass of ```Bundle```
```scala
class MyFloat extends Bundle {
  val sign        = Bool()
  val exponent    = UInt(width = 8)
  val significand = UInt(width = 23)
}

val x  = new MyFloat()
val xs = x.sign
```

A Scala convention is to capitalize the name of new classes and we
suggest you follow that convention in Chisel too.  The ```width```
named parameter to the ```UInt``` constructor specificies the number
of bits in the type.

Vecs create an indexable vector of elements, and are constructed as
follows:
```scala
// Vector of 5 23-bit signed integers.
val myVec = Vec.fill(5){ SInt(width = 23) } 

// Connect to one element of vector. 
val reg3  = myVec(3) 
```

\noindent
(Note that we have to specify the type of the ```Vec``` elements
inside the trailing curly brackets, as we have to pass the bitwidth
parameter into the ```SInt``` constructor.)

The set of primitive classes
(```SInt```, ```UInt```, and ```Bool```) plus the aggregate
classes (```Bundles``` and ```Vec```s) all inherit from a common
superclass, ```Data```.  Every object that ultimately inherits from
```Data``` can be represented as a bit vector in a hardware design.

Bundles and Vecs can be arbitrarily nested to build complex data
structures:
```scala
class BigBundle extends Bundle {
 // Vector of 5 23-bit signed integers.
 val myVec = Vec.fill(5) { SInt(width = 23) } 
 val flag  = Bool()
 // Previously defined bundle.
 val f     = new MyFloat()              
}
```

Note that the builtin Chisel primitive and aggregate classes do not
require the ```new``` when creating an instance, whereas new user
datatypes will.  A Scala ```apply``` constructor can be defined so
that a user datatype also does not require ```new```, as described in
[Function Constructor](Function Constructor)

[Prev (Functional Abstraction)](Functional Abstraction) [Next (Ports)](Ports)