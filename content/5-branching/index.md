## Branching

So far we're only able to write basic programs without any branching logic.
Let's change that.

6502 assembly language has a bunch of branching instructions, all of which
branch based on whether certain flags are set or not. In this example we'll be
looking at `BNE`: "Branch on not equal".

```6502
  LDX #$08
decrement:
  DEX
  STX $0200
  CPX #$03
  BNE decrement
  STX $0201
  BRK
```

First we load the value `$08` into the `X` register. The next line is a label.
Labels just mark certain points in a program so we can return to them later.
After the label we decrement `X`, store it to `$0200` (the top-left pixel), and
then compare it to the value `$03`.
[`CPX`](http://www.obelisk.me.uk/6502/reference.html#CPX) compares the
value in the `X` register with another value. If the two values are equal, the
`Z` flag is set to `1`, otherwise it is set to `0`.

The next line, `BNE decrement`, will shift execution to the decrement label if
the `Z` flag is set to `0` (meaning that the two values in the `CPX` comparison
were not equal), otherwise it does nothing and we store `X` to `$0201`, then
finish the program.

In assembly language, you'll usually use labels with branch instructions. When
assembled though, this label is converted to a single-byte relative offset (a
number of bytes to go backwards or forwards from the next instruction) so
branch instructions can only go forward and back around 256 bytes. This means
they can only be used to move around local code. For moving further you'll need
to use the jumping instructions.

### Exercises ###

1. The opposite of `BNE` is `BEQ`. Try writing a program that uses `BEQ`.
2. `BCC` and `BCS` ("branch on carry clear" and "branch on carry set") are used
   to branch on the carry flag. Write a program that uses one of these two.

