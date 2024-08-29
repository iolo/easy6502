## Introduction

In this tiny ebook I'm going to show you how to get started writing 6502
assembly language. The 6502 processor was massive in the seventies and
eighties, powering famous computers like the
[BBC Micro](http://en.wikipedia.org/wiki/BBC_Micro),
[Atari 2600](http://en.wikipedia.org/wiki/Atari_2600),
[Commodore 64](http://en.wikipedia.org/wiki/Commodore_64),
[Apple II](http://en.wikipedia.org/wiki/Apple_II), and the [Nintendo Entertainment
System](http://en.wikipedia.org/wiki/Nintendo_Entertainment_System). Bender in
Futurama [has a 6502 processor for a
brain](http://www.transbyte.org/SID/SID-files/Bender_6502.jpg). [Even the
Terminator was programmed in
6502](http://www.pagetable.com/docs/terminator/00-37-23.jpg).

So, why would you want to learn 6502? It's a dead language isn't it? Well,
so's Latin. And they still teach that.
[Q.E.D.](http://en.wikipedia.org/wiki/Q.E.D.)

(Actually, I've been reliably informed that 6502 processors are still being
produced by [Western Design Center](http://www.westerndesigncenter.com/wdc/w65c02s-chip.cfm)
and [sold to hobbyists](http://www.mouser.co.uk/Search/Refine.aspx?Keyword=65C02), so clearly 6502
*isn't* a dead language! Who knew?)

Seriously though, I think it's valuable to have an understanding of assembly
language. Assembly language is the lowest level of abstraction in computers -
the point at which the code is still readable. Assembly language translates
directly to the bytes that are executed by your computer's processor.
If you understand how it works, you've basically become a computer
[magician](http://skilldrick.co.uk/2011/04/magic-in-software-development/).

Then why 6502? Why not a *useful* assembly language, like
[x86](http://en.wikipedia.org/wiki/X86)? Well, I don't think learning x86 is
useful. I don't think you'll ever have to *write* assembly language in your day
job - this is purely an academic exercise, something to expand your mind and
your thinking. 6502 was originally written in a different age, a time when the majority of
developers were writing assembly directly, rather than in these new-fangled
high-level programming languages. So, it was designed to be written by humans.
More modern assembly languages are meant to written by compilers, so let's
leave it to them. Plus, 6502 is *fun*. Nobody ever called x86 *fun*.

