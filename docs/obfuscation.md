# Obfuscation
A number of different ways to hide your code from prying eyes.

## Simple Methods
Simple method 1 - This is used the the game Jetpac, and basically moves the game code
down 4 bytes after the game loads so that the main code is in the correct memory. Things
to note that you must have some spare memory for the code to go, and room enough to move
your main code to where you want it.

```asm
Decrypt:
    LD HL, LOADADDRESS      ; Where the code is loaded 
    LD DE, MOVETOADDRESS    ; Address where the code should relocate to
    LD BC, SIZEOFBLOCK      ; Length of the code block in bytes
    LDIR                    ; Block transfer
    JP MAINSTART            ; Go the the main entry point
```

