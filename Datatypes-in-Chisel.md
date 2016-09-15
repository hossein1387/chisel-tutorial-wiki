Chisel datatypes are used to specify the type of values held in state
elements or flowing on wires.  While hardware designs ultimately
operate on vectors of binary digits, other more abstract
representations for values allow clearer specifications and help the
tools generate more optimal circuits.  In Chisel, a raw collection of
bits is represented by the ```Bits``` type.  Signed and unsigned integers
are considered subsets of fixed-point numbers and are represented by
types ```SInt``` and ```UInt``` respectively. Signed fixed-point
numbers, including integers, are represented using two's-complement
format.  Boolean values are represented as type ```Bool```.  Note
that these types are distinct from Scala's builtin types such as
```Int``` or ```Boolean```.  Additionally, Chisel defines `Bundles` for making
collections of values with named fields (similar to ```structs``` in
other languages), and ``Vecs``` for indexable collections of
values.  Bundles and Vecs will be covered later.

Constant or literal values are expressed using Scala integers or
strings passed to constructors for the types:
```scala
1.U       // decimal 1-bit lit from Scala Int.
"ha".U    // hexadecimal 4-bit lit from string.
"o12".U   // octal 4-bit lit from string.
"b1010".U // binary 4-bit lit from string.

5.S    // signed decimal 4-bit lit from Scala Int.
-8.S   // negative decimal 4-bit lit from Scala Int.
5.U    // unsigned decimal 3-bit lit from Scala Int.

Bool(true) // Bool lits from Scala lits.
Bool(false)
```