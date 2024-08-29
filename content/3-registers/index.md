## Registers and flags

We've already had a little look at the processor status section (the bit with
`A`, `PC` etc.), but what does it all mean?

The first line shows the `A`, `X` and `Y` registers (`A` is often called the
"accumulator"). Each register holds a single byte. Most operations work on the
contents of these registers.

`SP` is the stack pointer. I won't get into the stack yet, but basically this
register is decremented every time a byte is pushed onto the stack, and
incremented when a byte is popped off the stack.

`PC` is the program counter - it's how the processor knows at what point in the
program it currently is. It's like the current line number of an executing
script. In the JavaScript simulator the code is assembled starting at memory
location `$0600`, so `PC` always starts there.

The last section shows the processor flags. Each flag is one bit, so all seven
flags live in a single byte. The flags are set by the processor to give
information about the previous instruction. More on that later. [Read more
about the registers and flags here](https://web.archive.org/web/20210626024532/http://www.obelisk.me.uk/6502/registers.html).

