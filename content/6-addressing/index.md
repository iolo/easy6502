## Addressing modes

The 6502 uses a 16-bit address bus, meaning that there are 65536 bytes of
memory available to the processor. Remember that a byte is represented by two
hex characters, so the memory locations are generally represented as `$0000 -
$ffff`. There are various ways to refer to these memory locations, as detailed below.

With all these examples you might find it helpful to use the memory monitor to
watch the memory change. The monitor takes a starting memory location and a
number of bytes to display from that location. Both of these are hex values.
For example, to display 16 bytes of memory from `$c000`, enter `c000` and `10`
into **Start** and **Length**, respectively.

### Absolute: `$c000` ###

With absolute addressing, the full memory location is used as the argument to the instruction. For example:

    STA $c000 ;Store the value in the accumulator at memory location $c000

### Zero page: `$c0` ###

All instructions that support absolute addressing (with the exception of the jump
instructions) also have the option to take a single-byte address. This type of
addressing is called "zero page" - only the first page (the first 256 bytes) of
memory is accessible. This is faster, as only one byte needs to be looked up,
and takes up less space in the assembled code as well.

### Zero page,X: `$c0,X` ###

This is where addressing gets interesting. In this mode, a zero page address is given, and then the value of the `X` register is added. Here is an example:

    LDX #$01   ;X is $01
    LDA #$aa   ;A is $aa
    STA $a0,X ;Store the value of A at memory location $a1
    INX        ;Increment X
    STA $a0,X ;Store the value of A at memory location $a2

If the result of the addition is larger than a single byte, the address wraps around. For example:

    LDX #$05
    STA $ff,X ;Store the value of A at memory location $04

### Zero page,Y: `$c0,Y` ###

This is the equivalent of zero page,X, but can only be used with `LDX` and `STX`.

### Absolute,X and absolute,Y: `$c000,X` and `$c000,Y` ###

These are the absolute addressing versions of zero page,X and zero page,Y. For example:

    LDX #$01
    STA $0200,X ;Store the value of A at memory location $0201

Unlike zero page,Y, absolute,Y can't be used with `STX` but can be used with `LDA` and `STA`.

### Immediate: `#$c0` ###

Immediate addressing doesn't strictly deal with memory addresses - this is the
mode where actual values are used. For example, `LDX #$01` loads the value
`$01` into the `X` register. This is very different to the zero page
instruction `LDX $01` which loads the value at memory location `$01` into the
`X` register.

### Relative: `$c0` (or label) ###

Relative addressing is used for branching instructions. These instructions take
a single byte, which is used as an offset from the following instruction.

Assemble the following code, then click the **Hexdump** button to see the assembled code.

```6502
  LDA #$01
  CMP #$02
  BNE notequal
  STA $22
notequal:
  BRK
```

The hex should look something like this:

    a9 01 c9 02 d0 02 85 22 00

`a9` and `c9` are the processor opcodes for immediate-addressed `LDA` and `CMP`
respectively. `01` and `02` are the arguments to these instructions. `d0` is
the opcode for `BNE`, and its argument is `02`. This means "skip over the next
two bytes" (`85 22`, the assembled version of `STA $22`). Try editing the code
so `STA` takes a two-byte absolute address rather than a single-byte zero page
address (e.g. change `STA $22` to `STA $2222`). Reassemble the code and look at
the hexdump again - the argument to `BNE` should now be `03`, because the
instruction the processor is skipping past is now three bytes long.

### Implicit ###

Some instructions don't deal with memory locations (e.g. `INX` - increment the
`X` register). These are said to have implicit addressing - the argument is
implied by the instruction.

### Indirect: `($c000)` ###

Indirect addressing uses an absolute address to look up another address. The
first address gives the least significant byte of the address, and the
following byte gives the most significant byte. That can be hard to wrap your
head around, so here's an example:

```6502
LDA #$01
STA $f0
LDA #$cc
STA $f1
JMP ($00f0) ;dereferences to $cc01
```

In this example, `$f0` contains the value `$01` and `$f1` contains the value
`$cc`. The instruction `JMP ($f0)` causes the processor to look up the two
bytes at `$f0` and `$f1` (`$01` and `$cc`) and put them together to form the
address `$cc01`, which becomes the new program counter. Assemble and step
through the program above to see what happens. I'll talk more about `JMP` in
the section on [Jumping](#jumping).

### Indexed indirect: `($c0,X)` ###

This one's kinda weird. It's like a cross between zero page,X and indirect.
Basically, you take the zero page address, add the value of the `X` register to
it, then use that to look up a two-byte address. For example:

```6502
LDX #$01
LDA #$05
STA $01
LDA #$07
STA $02
LDY #$0a
STY $0705
LDA ($00,X)
```

Memory locations `$01` and `$02` contain the values `$05` and `$07`
respectively. Think of `($00,X)` as `($00 + X)`. In this case `X` is `$01`, so
this simplifies to `($01)`. From here things proceed like standard indirect
addressing - the two bytes at `$01` and `$02` (`$05` and `$07`) are looked up
to form the address `$0705`.  This is the address that the `Y` register was
stored into in the previous instruction, so the `A` register gets the same
value as `Y`, albeit through a much more circuitous route. You won't see this
much.


### Indirect indexed: `($c0),Y` ###

Indirect indexed is like indexed indirect but less insane. Instead of adding
the `X` register to the address *before* dereferencing, the zero page address
is dereferenced, and the `Y` register is added to the resulting address.

```6502
LDY #$01
LDA #$03
STA $01
LDA #$07
STA $02
LDX #$0a
STX $0704
LDA ($01),Y
```

In this case, `($01)` looks up the two bytes at `$01` and `$02`: `$03` and
`$07`. These form the address `$0703`. The value of the `Y` register is added
to this address to give the final address `$0704`.

### Exercise ###

1. Try to write code snippets that use each of the 6502 addressing modes.
   Remember, you can use the monitor to watch a section of memory.

