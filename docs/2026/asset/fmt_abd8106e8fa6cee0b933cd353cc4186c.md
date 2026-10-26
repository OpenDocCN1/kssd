
```asm
:004053AD 7520

             jne 004053CF

:004053AF

53                      push

ebx

:004053B0 53

   push ebx

:004053B1 6A01

       push 00000001

:004053B3 6840010000

         push 00000140

:004053B8 B9A8035600

           mov ecx, 005603A8

:004053BD E8DEB50C00

             call 004D09A0

:004053C2

50                      push

eax

:004053C3 E8499C1100              call

0051F011

:004053C8 33C0

       xor eax, eax

:004053CA E9BD090000

         jmp 00405D8C
```

异途同归。
