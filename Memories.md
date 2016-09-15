Chisel provides facilities for creating both read only and read/write memories.  

## ROM

Users can define read only memories with a `Vec`:

``` scala
    Vec(inits: Seq[T])
    Vec(elt0: T, elts: T*)
```

where `inits` is a sequence of initial `Data` literals that initialize the ROM. For example,  users cancreate a small ROM initialized to 1, 2, 4, 8 and loop through all values using a counter as an address generator as follows:

``` scala
    val m = Vec(Array(1.U, 2.U, 4.U, 8.U))
    val r = m(counter(UInt(m.length)))
```

We can create an *n* value sine lookup table using a ROM initialized as follows:

``` scala
    def sinTable (amp: Double, n: Int) = {
      val times = 
        Range(0, n, 1).map(i => (i*2*Pi)/(n.toDouble-1) - Pi)
      val inits = 
        times.map(t => SInt(round(amp * sin(t)), width = 32))
      Vec(inits)
    }
    def sinWave (amp: Double, n: Int) = 
      sinTable(amp, n)(counter(UInt(n))
```

where `amp` is used to scale the fixpoint values stored in the ROM.

## Mem

Memories are given special treatment in Chisel since hardware
implementations of memory have many variations, e.g., FPGA memories
are instantiated quite differently from ASIC memories.  Chisel defines
a memory abstraction that can map to either simple Verilog behavioral
descriptions, or to instances of memory modules that are available
from external memory generators provided by foundry or IP vendors.  

Chisel supports random-access memories via the `Mem` construct.
Writes to Mems are positive-edge-triggered and reads are either
combinational or positive-edge-triggered.
*Current FPGA technology
does not support combinational (asynchronous) reads (anymore). The read address
needs to be registered.*


Ports into Mems are created by applying a `UInt` index.  A 32-entry
register file with one write port and two combinational read ports might be
expressed as follows:

``` scala
    val rf = Mem(32, UInt(width = 64))
    when (wen) { rf(waddr) := wdata }
    val dout1 = rf(waddr1)
    val dout2 = rf(waddr2)
```

If the optional parameter `seqRead` is set, Chisel will attempt to infer
sequential read ports when the read address is a `Reg`.  A one-read port,
one-write port SRAM might be described as follows:

``` scala
    val ram1r1w =
      Mem(1024, UInt(width = 32))
    val reg_raddr = Reg(UInt())
    when (wen) { ram1r1w(waddr) := wdata }
    when (ren) { reg_raddr := raddr }
    val rdata = ram1r1w(reg_raddr)
```

Single-ported SRAMs can be inferred when the read and write conditions are
mutually exclusive in the same `when` chain:

``` scala
    val ram1p = Mem(1024, UInt(width = 32))
    val reg_raddr = Reg(UInt())
    when (wen) { ram1p(waddr) := wdata }
    .elsewhen (ren) { reg_raddr := raddr }
    val rdata = ram1p(reg_raddr)
```

If the same `Mem` address is both written and sequentially read on the same clock
edge, or if a sequential read enable is cleared, then the read data is
undefined.

`Mem` also supports write masks for subword writes.  A given bit is written if
the corresponding mask bit is set.

``` scala
    val ram = Mem(256, UInt(width = 32))
    when (wen) { ram.write(waddr, wdata, wmask) }
```

<!---
For example, an
audio recorder could be defined as follows:

``` scala
  def audioRecorder(n: Int, button: Bool) = { 
    val addr   = counter(UInt(n))
    val ram    = Mem(n)
    ram(addr) := button
    ram(Mux(button(), 0.U, addr))
  } 
```

\noindent
where a counter is used as an address generator into a memory.  
The device records while \verb+button+ is \verb+true+, or plays back when \verb+false+.
--->

[Prev(State Elements)]  (State Elements)[Next(Interfaces \& Bulk Connections)](Interfaces \& Bulk Connections)
