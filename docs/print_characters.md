# Printing Text to the Screen
A few different ways to print text to the screen, from using the ROM routines to hand-crafted
code that does the same thing.

## The 'Ultimate' way
Early ultimate games seem to use this method for printing strings to the screen. It involves
setting the MSB in the last character in the string to signify the end of the string. 

For example, the string "Hello World" would be encoded as such -

```
Hello:  defb "Hello Worl", 0xC4 ; C4 = 0x44 with msb set
```

The print routine expects the screen positon in HL and DE to point to the start of the string. Note 
that the first byte seems to be the atrribute colour for the string. It makes use of two other routines, 
which convert the position in HL into a screen memory address and also a Attribute memory address.

This is the routine for converting a print @ position into screen memory address:

```
; Convert position in HL to memory address
; [IN]  HL = Position
; [OUT] HL = Screen memory location
TOSCREENMEM:
    LD A,L
    RRCA        ; Divide by 8
    RRCA
    RRCA    
    AND 0x1F    ; Keep the bottom 5 bits
    LD L,A

    LD A,H
    RLCA        ; Multiply by 4
    RLCA
    AND 0xE0    ; Set the top 3 bits (the 0x40 in screen memory)
    OR L
    LD L,A
    LD A,H
    AND 0x07    ; Keep low 3 bits
    EX AF,AF'
    LD A,H
    RRCA        ; Divide by 8
    RRCA
    RRCA    
    AND 0X18    ; Keep bottom 5 bits
    OR 0x40     ; Or it with 0x40 (start of screen memory)
    LD H,A
    EX AF,AF'
    OR H
    LD H,A
    RET
```

And to conver the same print location in HL into an attribute memory address, you would

```
; [IN]  HL=Print Location
; [OUT] HL=Attribute memory
TOATTRMEM:
    LD A,L
    RRCA        ; Divide by 8
    RRCA
    RRCA
    AND 0x1F
    LD L,A
    LD A,H
    RLCA
    RLCA
    LD C,A
    AND 0xE0
    OR L
    LD L,A
    LD A,C
    AND 0x03
    OR 0x58     ; OR the value into attr memory (from 0x58)
    LD H,A
    RET
```
    

The actual print routine that uses these to routines, is as follows:

```
; Prints a string
; [IN] HL = Screen Position (ie 0x0018)
; [IN] DE = Ptr to string memory
PrintString:
    PUSH HL
    CALL TOSCREENMEM
    LD A,(DE)
    EX AF,AF'
    INC DE
    EXX
    POP HL
    CALL TOATTRMEM
.loop:
    EXX
    LD A,(DE)
    BIT 7,A
    JR NZ,LASTCHAR
    CALL PRINTCHAR
    INC DE
    EXX
    EX AF,AF'
    LD (HL),A
    INC L
    EX AF,AF'
    JR .loop

LASTCHAR:
    AND 0x7F
    CALL PRINTCHAR
    EXX
    EX AF,AF'
    LD (HL),A
    RET

CHARS EQU 0x5C36        ; System Var points 256 less than font bitmaps

PRINTCHAR:
    PUSH BC
    PUSH DE
    PUSH HL
    LD L,A
    LD H,0
    ADD HL,HL
    ADD HL,HL
    ADD HL,HL
    LD DE,(CHARS)
    ADD HL,DE
    EX DE,HL
    POP HL
    LD B,0x08
.chrloop:
    LD A,(DE)
    LD (HL),A
    INC DE
    INC H
    DJNZ .chrloop
    POP DE
    POP BC
    LD A,H
    SUB 0x08
    LD H,A
    INC L
    RET
```

An example snippet using this would be:

```
    LD HL,0x0018
    LD DE,HELLO
    CALL PrintString

HELLO:
    DEFB 0x47, "HELLO, WORL", 0xC4
```
