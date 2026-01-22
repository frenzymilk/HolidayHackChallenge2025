---
icon: material/text-box-outline
---

# Act 2: Going in reverse

**Difficulty**: :fontawesome-solid-star::fontawesome-solid-star::fontawesome-regular-star::fontawesome-regular-star::fontawesome-regular-star:<br/>
**Direct link**: [Objective 1 terminal](https://.../)

## Objective

!!! question "Request"
    Kevin in the Retro Store needs help rewinding tech and going in reverse. Extract the flag and enter it here.

??? quote "Kevin McFarland"
    Finding an old Commodore 64 disk with a mysterious BASIC program on it? That's like discovering a digital time capsule. The C64 was an incredible machine for its time - 64KB of RAM seemed like an ocean of possibility back then. I spent countless hours as a kid typing in program listings from Compute! magazine, usually making at least a dozen typos along the way.<br/>
    The thing about BASIC programs from that era is they were often written by clever programmers who knew how to hide things in plain sight. Sometimes the most interesting discoveries come from reading the code itself rather than watching it execute. It's like being a digital archaeologist - you're not just looking at what the program does, you're understanding how the programmer thought.<br/>
    Take your time with this one. Those old-school programmers had to be creative within such tight constraints. You'll know the flag by the Christmas phrase that pays.

## Solution

To be honest, my assembly being pretty rusty, ChatGPT did most of the job there.

I just passed the code:
```
10 REM *** COMMODORE 64 SECURITY SYSTEM ***
20 ENC_PASS$ = "D13URKBT"
30 ENC_FLAG$ = "DSA|auhts*wkfi=dhjwubtthut+dhhkfis+hnkz" ' old "DSA|qnisf`bX_huXariz"
40 INPUT "ENTER PASSWORD: "; PASS$
50 IF LEN(PASS$) <> LEN(ENC_PASS$) THEN GOTO 90
60 FOR I = 1 TO LEN(PASS$)
70 IF CHR$(ASC(MID$(PASS$,I,1)) XOR 7) <> MID$(ENC_PASS$,I,1) THEN GOTO 90
80 NEXT I
85 FLAG$ = "" : FOR I = 1 TO LEN(ENC_FLAG$) : FLAG$ = FLAG$ + CHR$(ASC(MID$(ENC_FLAG$,I,1)) XOR 7) : NEXT I : PRINT FLAG$
90 PRINT "ACCESS DENIED"
100 END
```

Here is the output:
```
Nice little C64-style XOR challenge 🙂

What it does

The password check XORs each character with 7 and compares it to ENC_PASS$.

So the real password is simply ENC_PASS$ XOR 7.

Same logic for the flag.

Decoded results

Password: C64RULES

Flag: CTF{frost-plan:compressors,coolant,oil}

Why

D13URKBT XOR 7 → C64RULES

ENC_FLAG$ XOR 7 → the flag above

Classic retro crypto, simple but effective for its era.
```

####  And the flag is: CTF{frost-plan:compressors,coolant,oil}