# Working with the screen and attribute memory
Various routines and methods for working with screen memory. 

## Clearing the Screen Memory

This routine is actually two routines in one; Both make use of the same loop to do the
fill on the screen, but you have to call both seperately.

```
ClearScreen:
    LD HL,0x4000            ; Start of Display memory
    LD B, 0x58              ; 0x58 is the start of the Attribute memory
    LD C,0                  ; Value to fill memory with (0 no bits set)
LOOP:
    LD (HL),C               ; Write the value to the screen
    INC HL                  ; Next screen position
    LD A,H
    CP B                    ; Are we at the attribute file yet?
    JR C, LOOP              ; Nope keep going
    RET
ClearAttrs:
    LD HL,0x5800            ; Start of Attribute memory
    LD B, 0x5B              ; 0x5B is the start of the printer buffer
    LD C,0x47               ; Attribute fill colour
    JR LOOP
```

Another quick way to clear the screen memory that will clear both the Screen and Attribute
memory is to use the block transfer command:

```
; Clears both the screen and attribute memory
; Note that the 0 fill value will set the attributes to Black Ink/Paper
CLR_SCREEN:
    LD HL,0x4000            ; Start of Screen memory
    LD DE,0x4001            ; Next location
    LD BC,0x1AFF            ; Size of screen/attrs -1
    LD (HL),0               ; Always fill the first byte
    LDIR
    RET
```

Yet another way to clear them seperately:

```
; Clear Just the Display memory
CLR_SCREEN:
    LD HL,0x4000            ; Start of Screen memory
    LD DE,0x4001            ; Next screen byte
    LD BC,0x17FF            ; Size of screen memory less 1
    LD (HL),0               ; Set it to clear
    LDIR                    ; Block Transfer
    RET                     ; Done


; Clears the Attribute File
; [IN] Set A to the colour value required
CLR_ATTRS:
    LD HL,0x5800            ; Start of Attribute memory
    LD DE,0x5801            ; next byte
    LD BC,0x2FF             ; Size of attrs memory less 1
    LD (HL),A               ; Set the colour to value in accum
    LDIR                    ; Block transfer
    RET                     ; Finished
```


