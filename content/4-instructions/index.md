## Instructions

Instructions in assembly language are like a small set of predefined functions.
All instructions take zero or one arguments. Here's some annotated
source code to introduce a few different instructions:

```6502
LDA #$c0  ;Load the hex value $c0 into the A register
TAX       ;Transfer the value in the A register to X
INX       ;Increment the value in the X register
ADC #$c4  ;Add the hex value $c4 to the A register
BRK       ;Break - we're done
```

Assemble the code, then turn on the debugger and step through the code, watching
the `A` and `X` registers. Something slightly odd happens on the line `ADC #$c4`.
You might expect that adding `$c4` to `$c0` would give `$184`, but this
processor gives the result as `$84`. What's up with that?

The problem is, `$184` is too big to fit in a single byte (the max is `$FF`),
and the registers can only hold a single byte.  It's OK though; the processor
isn't actually dumb. If you were looking carefully enough, you'll have noticed
that the carry flag was set to `1` after this operation. So that's how you
know.

In the simulator below **type** (don't paste) the following code:

    LDA #$80
    STA $01
    ADC $01

```6502
```

An important thing to notice here is the distinction between `ADC #$01` and
`ADC $01`. The first one adds the value `$01` to the `A` register, but the
second adds the value stored at memory location `$01` to the `A` register.

Assemble, check the **Monitor** checkbox, then step through these three
instructions. The monitor shows a section of memory, and can be helpful to
visualise the execution of programs. `STA $01` stores the value of the `A`
register at memory location `$01`, and `ADC $01` adds the value stored at the
memory location `$01` to the `A` register. `$80 + $80` should equal `$100`, but
because this is bigger than a byte, the `A` register is set to `$00` and the
carry flag is set. As well as this though, the zero flag is set. The zero flag
is set by all instructions where the result is zero.

A full list of the 6502 instruction set is [available
here](http://www.6502.org/tutorials/6502opcodes.html) and
[here](http://www.obelisk.me.uk/6502/reference.html) (I usually refer to
both pages as they have their strengths and weaknesses). These pages detail the
arguments to each instruction, which registers they use, and which flags they
set. They are your bible.

### Exercises ###

1. You've seen `TAX`. You can probably guess what `TAY`, `TXA` and `TYA` do,
   but write some code to test your assumptions.
2. Rewrite the first example in this section to use the `Y` register instead of
   the `X` register.
3. The opposite of `ADC` is `SBC` (subtract with carry). Write a program that
   uses this instruction.


