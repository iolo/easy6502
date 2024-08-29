## Creating a game

Now, let's put all this knowledge to good use, and make a game! We're going to
be making a really simple version of the classic game 'Snake'.

Even though this will be a simple version, the code will be substantially larger
than all the previous examples. We will need to keep track of several memory
locations together for the various aspects of the game. We can still do
the necessary bookkeeping throughout the program ourselves, as before, but
on a larger scale that quickly becomes tedious and can also lead to bugs that
are difficult to spot. Instead we'll now let the assembler do some of the
mundane work for us.

In this assembler, we can define descriptive constants (or symbols) that represent
numbers. The rest of the code can then simply use the constants instead of the
literal number, which immediately makes it obvious what we're dealing with.
You can use letters, digits and underscores in a name.

Here's an example. Note that immediate operands are still prefixed with a `#`.

```6502   
  define  sysRandom  $fe ; an address
  define  a_dozen    $0c ; a constant
 
  LDA sysRandom  ; equivalent to "LDA $fe"

  LDX #a_dozen   ; equivalent to "LDX #$0c"
```

The simulator widget below contains the entire source code of the game. I'll
explain how it works in the following sections.

[Willem van der Jagt](https://twitter.com/wkjagt) made a [fully annotated gist
of this source code](https://gist.github.com/wkjagt/9043907), so follow along
with that for more details.

```6502   
;  ___           _        __ ___  __ ___
; / __|_ _  __ _| |_____ / /| __|/  \_  )
; \__ \ ' \/ _` | / / -_) _ \__ \ () / /
; |___/_||_\__,_|_\_\___\___/___/\__/___|

; Change direction: W A S D

define appleL         $00 ; screen location of apple, low byte
define appleH         $01 ; screen location of apple, high byte
define snakeHeadL     $10 ; screen location of snake head, low byte
define snakeHeadH     $11 ; screen location of snake head, high byte
define snakeBodyStart $12 ; start of snake body byte pairs
define snakeDirection $02 ; direction (possible values are below)
define snakeLength    $03 ; snake length, in bytes

; Directions (each using a separate bit)
define movingUp      1
define movingRight   2
define movingDown    4
define movingLeft    8

; ASCII values of keys controlling the snake
define ASCII_w      $77
define ASCII_a      $61
define ASCII_s      $73
define ASCII_d      $64

; System variables
define sysRandom    $fe
define sysLastKey   $ff


  jsr init
  jsr loop

init:
  jsr initSnake
  jsr generateApplePosition
  rts


initSnake:
  lda #movingRight  ;start direction
  sta snakeDirection

  lda #4  ;start length (2 segments)
  sta snakeLength
  
  lda #$11
  sta snakeHeadL
  
  lda #$10
  sta snakeBodyStart
  
  lda #$0f
  sta $14 ; body segment 1
  
  lda #$04
  sta snakeHeadH
  sta $13 ; body segment 1
  sta $15 ; body segment 2
  rts


generateApplePosition:
  ;load a new random byte into $00
  lda sysRandom
  sta appleL

  ;load a new random number from 2 to 5 into $01
  lda sysRandom
  and #$03 ;mask out lowest 2 bits
  clc
  adc #2
  sta appleH

  rts


loop:
  jsr readKeys
  jsr checkCollision
  jsr updateSnake
  jsr drawApple
  jsr drawSnake
  jsr spinWheels
  jmp loop


readKeys:
  lda sysLastKey
  cmp #ASCII_w
  beq upKey
  cmp #ASCII_d
  beq rightKey
  cmp #ASCII_s
  beq downKey
  cmp #ASCII_a
  beq leftKey
  rts
upKey:
  lda #movingDown
  bit snakeDirection
  bne illegalMove

  lda #movingUp
  sta snakeDirection
  rts
rightKey:
  lda #movingLeft
  bit snakeDirection
  bne illegalMove

  lda #movingRight
  sta snakeDirection
  rts
downKey:
  lda #movingUp
  bit snakeDirection
  bne illegalMove

  lda #movingDown
  sta snakeDirection
  rts
leftKey:
  lda #movingRight
  bit snakeDirection
  bne illegalMove

  lda #movingLeft
  sta snakeDirection
  rts
illegalMove:
  rts


checkCollision:
  jsr checkAppleCollision
  jsr checkSnakeCollision
  rts


checkAppleCollision:
  lda appleL
  cmp snakeHeadL
  bne doneCheckingAppleCollision
  lda appleH
  cmp snakeHeadH
  bne doneCheckingAppleCollision

  ;eat apple
  inc snakeLength
  inc snakeLength ;increase length
  jsr generateApplePosition
doneCheckingAppleCollision:
  rts


checkSnakeCollision:
  ldx #2 ;start with second segment
snakeCollisionLoop:
  lda snakeHeadL,x
  cmp snakeHeadL
  bne continueCollisionLoop

maybeCollided:
  lda snakeHeadH,x
  cmp snakeHeadH
  beq didCollide

continueCollisionLoop:
  inx
  inx
  cpx snakeLength          ;got to last section with no collision
  beq didntCollide
  jmp snakeCollisionLoop

didCollide:
  jmp gameOver
didntCollide:
  rts


updateSnake:
  ldx snakeLength
  dex
  txa
updateloop:
  lda snakeHeadL,x
  sta snakeBodyStart,x
  dex
  bpl updateloop

  lda snakeDirection
  lsr
  bcs up
  lsr
  bcs right
  lsr
  bcs down
  lsr
  bcs left
up:
  lda snakeHeadL
  sec
  sbc #$20
  sta snakeHeadL
  bcc upup
  rts
upup:
  dec snakeHeadH
  lda #$1
  cmp snakeHeadH
  beq collision
  rts
right:
  inc snakeHeadL
  lda #$1f
  bit snakeHeadL
  beq collision
  rts
down:
  lda snakeHeadL
  clc
  adc #$20
  sta snakeHeadL
  bcs downdown
  rts
downdown:
  inc snakeHeadH
  lda #$6
  cmp snakeHeadH
  beq collision
  rts
left:
  dec snakeHeadL
  lda snakeHeadL
  and #$1f
  cmp #$1f
  beq collision
  rts
collision:
  jmp gameOver


drawApple:
  ldy #0
  lda sysRandom
  sta (appleL),y
  rts


drawSnake:
  ldx snakeLength
  lda #0
  sta (snakeHeadL,x) ; erase end of tail

  ldx #0
  lda #1
  sta (snakeHeadL,x) ; paint head
  rts


spinWheels:
  ldx #0
spinloop:
  nop
  nop
  dex
  bne spinloop
  rts


gameOver:
```


### Overall structure ###

After the initial block of comments (lines starting with semicolons), the first
two lines are:

    jsr init
    jsr loop

`init` and `loop` are both subroutines. `init` initializes the game state, and
`loop` is the main game loop.

The `loop` subroutine itself just calls a number of subroutines sequentially,
before looping back on itself:

    loop:
      jsr readkeys
      jsr checkCollision
      jsr updateSnake
      jsr drawApple
      jsr drawSnake
      jsr spinwheels
      jmp loop

First, `readkeys` checks to see if one of the direction keys (W, A, S, D) was
pressed, and if so, sets the direction of the snake accordingly. Then,
`checkCollision` checks to see if the snake collided with itself or the apple.
`updateSnake` updates the internal representation of the snake, based on its
direction. Next, the apple and snake are drawn. Finally, `spinWheels` makes the
processor do some busy work, to stop the game from running too quickly. Think
of it like a sleep command. The game keeps running until the snake collides
with the wall or itself.


### Zero page usage ###

The zero page of memory is used to store a number of game state variables, as
noted in the comment block at the top of the game. Everything in `$00`, `$01`
and `$10` upwards is a pair of bytes representing a two-byte memory location
that will be looked up using indirect addressing.  These memory locations will
all be between `$0200` and `$05ff` - the section of memory corresponding to the
simulator display. For example, if `$00` and `$01` contained the values `$01`
and `$02`, they would be referring to the second pixel of the display (
`$0201` - remember, the least significant byte comes first in indirect addressing).

The first two bytes hold the location of the apple. This is updated every time
the snake eats the apple. Byte `$02` contains the current direction. `1` means
up, `2` right, `4` down, and `8` left.  The reasoning behind these numbers will
become clear later.

Finally, byte `$03` contains the current length of the snake, in terms of bytes
in memory (so a length of 4 means 2 pixels).


### Initialization ###

The `init` subroutine defers to two subroutines, `initSnake` and
`generateApplePosition`. `initSnake` sets the snake direction, length, and then
loads the initial memory locations of the snake head and body. The byte pair at
`$10` contains the screen location of the head, the pair at `$12` contains the
location of the single body segment, and `$14` contains the location of the
tail (the tail is the last segment of the body and is drawn in black to keep
the snake moving). This happens in the following code:

    lda #$11
    sta $10
    lda #$10
    sta $12
    lda #$0f
    sta $14
    lda #$04
    sta $11
    sta $13
    sta $15

This loads the value `$11` into the memory location `$10`, the value `$10` into
`$12`, and `$0f` into `$14`. It then loads the value `$04` into `$11`, `$13`
and `$15`. This leads to memory like this:

    0010: 11 04 10 04 0f 04

which represents the indirectly-addressed memory locations `$0411`, `$0410` and
`$040f` (three pixels in the middle of the display). I'm labouring this point,
but it's important to fully grok how indirect addressing works.

The next subroutine, `generateApplePosition`, sets the apple location to a
random position on the display. First, it loads a random byte into the
accumulator (`$fe` is a random number generator in this simulator). This is
stored into `$00`. Next, a different random byte is loaded into the
accumulator, which is then `AND`-ed with the value `$03`. This part requires a
bit of a detour.

The hex value `$03` is represented in binary as `00000011`. The `AND` opcode
performs a bitwise AND of the argument with the accumulator. For example, if
the accumulator contains the binary value `10101010`, then the result of `AND`
with `00000011` will be `00000010`.

The effect of this is to mask out the least significant two bits of the
accumulator, setting the others to zero. This converts a number in the range of
0&ndash;255 to a number in the range of 0&ndash;3.

After this, the value `2` is added to the accumulator, to create a final random
number in the range 2&ndash;5.

The result of this subroutine is to load a random byte into `$00`, and a random
number between 2 and 5 into `$01`. Because the least significant byte comes
first with indirect addressing, this translates into a memory address between
`$0200` and `$05ff`: the exact range used to draw the display.


### The game loop ###

Nearly all games have at their heart a game loop. All game loops have the same
basic form: accept user input, update the game state, and render the game
state. This loop is no different.


#### Reading the input ####

The first subroutine, `readKeys`, takes the job of accepting user input. The
memory location `$ff` holds the ascii code of the most recent key press in this
simulator. The value is loaded into the accumulator, then compared to `$77`
(the hex code for W), `$64` (D), `$73` (S) and `$61` (A). If any of these
comparisons are successful, the program branches to the appropriate section.
Each section (`upKey`, `rightKey`, etc.) first checks to see if the current
direction is the opposite of the new direction. This requires another little detour.

As stated before, the four directions are represented internally by the numbers
1, 2, 4 and 8. Each of these numbers is a power of 2, thus they are represented
by a binary number with a single `1`:

    1 => 0001 (up)
    2 => 0010 (right)
    4 => 0100 (down)
    8 => 1000 (left)

The `BIT` opcode is similar to `AND`, but the calculation is only used to set
the zero flag - the actual result is discarded. The zero flag is set only if the
result of AND-ing the accumulator with argument is zero. When we're looking at
powers of two, the zero flag will only be set if the two numbers are not the
same. For example, `0001 AND 0001` is not zero, but `0001 AND 0010` is zero.

So, looking at `upKey`, if the current direction is down (4), the bit test will
be zero. `BNE` means "branch if the zero flag is clear", so in this case we'll
branch to `illegalMove`, which just returns from the subroutine. Otherwise, the
new direction (1 in this case) is stored in the appropriate memory location.


#### Updating the game state ####

The next subroutine, `checkCollision`, defers to `checkAppleCollision` and
`checkSnakeCollision`. `checkAppleCollision` just checks to see if the two
bytes holding the location of the apple match the two bytes holding the
location of the head. If they do, the length is increased and a new apple
position is generated.

`checkSnakeCollision` loops through the snake's body segments, checking each
byte pair against the head pair. If there is a match, then game over.

After collision detection, we update the snake's location. This is done at a
high level like so: First, move each byte pair of the body up one position in
memory. Second, update the head according to the current direction. Finally, if
the head is out of bounds, handle it as a collision. I'll illustrate this with
some ascii art. Each pair of brackets contains an x,y coordinate rather than a
pair of bytes for simplicity.

      0    1    2    3    4
    Head                 Tail
    
    [1,5][1,4][1,3][1,2][2,2]    Starting position

    [1,5][1,4][1,3][1,2][1,2]    Value of (3) is copied into (4)
    
    [1,5][1,4][1,3][1,3][1,2]    Value of (2) is copied into (3)
    
    [1,5][1,4][1,4][1,3][1,2]    Value of (1) is copied into (2)
    
    [1,5][1,5][1,4][1,3][1,2]    Value of (0) is copied into (1)
    
    [0,5][1,5][1,4][1,3][1,2]    Value of (0) is updated based on direction

At a low level, this subroutine is slightly more complex. First, the length is
loaded into the `X` register, which is then decremented. The snippet below
shows the starting memory for the snake.

    Memory location: $10 $11 $12 $13 $14 $15

    Value:           $11 $04 $10 $04 $0f $04

The length is initialized to `4`, so `X` starts off as `3`. `LDA $10,x` loads the
value of `$13` into `A`, then `STA $12,x` stores this value into `$15`. `X` is
decremented, and we loop. Now `X` is `2`, so we load `$12` and store it into
`$14`. This loops while `X` is positive (`BPL` means "branch if positive").

Once the values have been shifted down the snake, we have to work out what to
do with the head. The direction is first loaded into `A`. `LSR` means "logical
shift right", or "shift all the bits one position to the right". The least
significant bit is shifted into the carry flag, so if the accumulator is `1`,
after `LSR` it is `0`, with the carry flag set.

To test whether the direction is `1`, `2`, `4` or `8`, the code continually
shifts right until the carry is set. One `LSR` means "up", two means "right",
and so on.

The next bit updates the head of the snake depending on the direction. This is
probably the most complicated part of the code, and it's all reliant on how
memory locations map to the screen, so let's look at that in more detail.

You can think of the screen as four horizontal strips of 32 &times; 8 pixels.
These strips map to `$0200-$02ff`, `$0300-$03ff`, `$0400-$04ff` and `$0500-$05ff`.
The first rows of pixels are `$0200-$021f`, `$0220-$023f`, `$0240-$025f`, etc.

As long as you're moving within one of these horizontal strips, things are
simple. For example, to move right, just increment the least significant byte
(e.g. `$0200` becomes `$0201`). To go down, add `$20` (e.g. `$0200` becomes
`$0220`). Left and up are the reverse.

Going between sections is more complicated, as we have to take into account the
most significant byte as well. For example, going down from `$02e1` should lead
to `$0301`. Luckily, this is fairly easy to accomplish. Adding `$20` to `$e1`
results in `$01` and sets the carry bit. If the carry bit was set, we know we
also need to increment the most significant byte.

After a move in each direction, we also need to check to see if the head
would become out of bounds. This is handled differently for each direction. For
left and right, we can check to see if the head has effectively "wrapped
around". Going right from `$021f` by incrementing the least significant byte
would lead to `$0220`, but this is actually jumping from the last pixel of the
first row to the first pixel of the second row. So, every time we move right,
we need to check if the new least significant byte is a multiple of `$20`. This
is done using a bit check against the mask `$1f`. Hopefully the illustration
below will show you how masking out the lowest 5 bits reveals whether a number
is a multiple of `$20` or not.

    $20: 0010 0000
    $40: 0100 0000
    $60: 0110 0000

    $1f: 0001 1111

I won't explain in depth how each of the directions work, but the above
explanation should give you enough to work it out with a bit of study.


#### Rendering the game ####

Because the game state is stored in terms of pixel locations, rendering the
game is very straightforward. The first subroutine, `drawApple`, is extremely
simple. It sets `Y` to zero, loads a random colour into the accumulator, then
stores this value into `($00),y`. `$00` is where the location of the apple is
stored, so `($00),y` dereferences to this memory location. Read the "Indirect
indexed" section in [Addressing modes](#addressing) for more details.

Next comes `drawSnake`. This is pretty simple too - we first undraw the tail
and then draw the head. `X` is set to the length of the snake, so we can index
to the right pixel, and we set `A` to zero then perform the write using the
indexed indirect addressing mode. Then we reload `X` to index to the head, set
`A` to one and store it at `($10,x)`. `$10` stores the two-byte location of
the head, so this draws a white pixel at the current head position. As only
the head and the tail of the snake move, this is enough to keep the snake
moving.

The last subroutine, `spinWheels`, is just there because the game would run too
fast otherwise. All `spinWheels` does is count `X` down from zero until it hits
zero again. The first `dex` wraps, making `X` `#$ff`.

