## The stack

The stack in a 6502 processor is just like any other stack - values are pushed
onto it and popped ("pulled" in 6502 parlance) off it. The current depth of the
stack is measured by the stack pointer, a special register. The stack lives in
memory between `$0100` and `$01ff`. The stack pointer is initially `$ff`, which
points to memory location `$01ff`. When a byte is pushed onto the stack, the
stack pointer becomes `$fe`, or memory location `$01fe`, and so on.

Two of the stack instructions are `PHA` and `PLA`, "push accumulator" and "pull
accumulator". Below is an example of these two in action.

```6502   
  LDX #$00
  LDY #$00
firstloop:
  TXA
  STA $0200,Y
  PHA
  INX
  INY
  CPY #$10
  BNE firstloop ;loop until Y is $10
secondloop:
  PLA
  STA $0200,Y
  INY
  CPY #$20      ;loop until Y is $20
  BNE secondloop
```

`X` holds the pixel colour, and `Y` holds the position of the current pixel.
The first loop draws the current colour as a pixel (via the `A` register),
pushes the colour to the stack, then increments the colour and position.  The
second loop pops the stack, draws the popped colour as a pixel, then increments
the position. As should be expected, this creates a mirrored pattern.

