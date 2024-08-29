## Our first program

So, let's dive in! That thing below is a little [JavaScript 6502 assembler and
simulator](https://github.com/skilldrick/6502js) that I adapted for this book.
Click **Assemble** then **Run** to assemble and run the snippet of assembly language.

```6502
LDA #$01
STA $0200
LDA #$05
STA $0201
LDA #$08
STA $0202
```

Hopefully the black area on the right now has three coloured "pixels" at the
top left. (If this doesn't work, you'll probably need to upgrade your browser to
something more modern, like Chrome or Firefox.)

So, what's this program actually doing? Let's step through it with the
debugger. Hit **Reset**, then check the **Debugger** checkbox to start the
debugger. Click **Step** once. If you were watching carefully, you'll have
noticed that `A=` changed from `$00` to `$01`, and `PC=` changed from `$0600` to
`$0602`.

Any numbers prefixed with `$` in 6502 assembly language (and by extension, in
this book) are in hexadecimal (hex) format. If you're not familiar with hex
numbers, I recommend you read [the Wikipedia
article](http://en.wikipedia.org/wiki/Hexadecimal). Anything prefixed with `#`
is a literal number value. Any other number refers to a memory location.

Equipped with that knowledge, you should be able to see that the instruction
`LDA #$01` loads the hex value `$01` into register `A`. I'll go into more
detail on registers in the next section.

Press **Step** again to execute the second instruction. The top-left pixel of
the simulator display should now be white. This simulator uses the memory
locations `$0200` to `$05ff` to draw pixels on its display. The values `$00` to
`$0f` represent 16 different colours (`$00` is black and `$01` is white), so
storing the value `$01` at memory location `$0200` draws a white pixel at the
top left corner. This is simpler than how an actual computer would output
video, but it'll do for now.

So, the instruction `STA $0200` stores the value of the `A` register to memory
location `$0200`. Click **Step** four more times to execute the rest of the
instructions, keeping an eye on the `A` register as it changes.

### Exercises ###

1. Try changing the colour of the three pixels.
2. Change one of the pixels to draw at the bottom-right corner (memory location `$05ff`).
3. Add more instructions to draw extra pixels.

