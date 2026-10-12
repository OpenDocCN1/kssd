
```assembly
012718B3    EB 05          jmp short 012718BA  
012718B5    1BC0           sbb eax,eax  
012718B7    83D8 FF       sbb eax,-1  
012718BA    85C0           test eax,eax  
012718BC    75 0C          jnz short 012718CA  
012718BE    5F            pop edi  
012718BF    5E            pop esi  
012718C0    5D            pop ebp  
012718C1    B8 A01B2701   mov eax,1271BA0  
012718C6    5B            pop ebx  
012718C7    C2 0C00       retn 0C  
012718CA    57            push edi  
012718CB    55            push ebp  
012718CC    E8 AF000000   call 01271980  
012718D1    8BF0          mov esi,eax  
012718D3    8B4424 1C     mov eax,dword ptr ss:[esp+1C]  
012718D7    25 FF000000   and eax,0FF  
012718DC    48            dec eax  
012718DD    83F8 03       cmp eax,3  
012718E0    77 7C         ja short 0127195E  
```

```text
*/  
mov T0,eip  
and T0,0FFFF0000  
log T0  
find T0,#FFD0F7D81BC0F7D8C3#  
cmp $RESULT, 0  
je Magic Jmp1  
mov [$RESULT],#FFD0F7D81BC033C0C3#  
//Pass IsDebuggerPresent  
Magic Jmp1:  
find T0,#8B??????8B??????3BE8#  
cmp $RESULT, 0  
log $RESULT  
je Magic Jmp2  
mov T1,$RESULT  
add T1,A  
mov [T1],#E9#  
find T1,#81??000001000F82??000000#  
cmp $RESULT, 0  
log $RESULT  
je Magic Jmp2  
mov T2,$RESULT  
add T2,8  
mov T3, [T2]  
sub T2,T1  
log T2  
add T1,1  
add T3,T2  
sub T3,1  
mov [T1],T3  
log T3  
Magic Jmp2:  
find T0,#83F80377??FF24#  
cmp $RESULT, 0  
je NoFind  
log $RESULT  
mov [$RESULT],#83F803EB#  
//Fixed Importing Function  
//GetOEP————————————————————————————————  
/*  
```

```assembly
01273841    61            popad  
01273842    870424        xchg dword ptr ss:[esp],eax  
01273845    C3            retn  
```

```text
*/  
find T0, #61870424C3#  
cmp $RESULT, 0  
je NoFind  
add $RESULT,4  
eob GetOEP  
bp $RESULT  
esto  
GoOn1:  
esto  
GetOEP:  
cmp eip,$RESULT  
jne GoOn1  
log eip  
bc $RESULT  
sti  
log eip  
cmt eip, "This is the OEP! Found by fly"  
MSG "Just: OEP !  Dump and Fix IAT & Last Section Size.  Good Luck  "  
ret  
NoFind:  
MSG "Error! Maybe It's not Protection Plus V4.X  "  
ret 

4dd466d0e1e0cc123afbe504da2462ca
```
