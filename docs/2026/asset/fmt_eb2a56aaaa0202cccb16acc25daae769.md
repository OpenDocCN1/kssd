
# Obsidium V1.3.0.0-V1.3.0.4 UnPacK Script

//////////////////////////////////////////////////////////
// FileName    :  Obsidium V1.3.0.0.osc
// Comment     :  Obsidium V1.3.0.0-V1.3.0.4 UnPacK Script
// Environment :  WinXP SP2,OllyDbg V1.10,OllyScript V0.92
// Author      :  heXer & fly
// WebSite     :  e82K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4g2F1M7r3q4U0K9#2)9J5k6h3y4F1
// Date        :  2005-11-01 16:00
//////////////////////////////////////////////////////////

```
#log
dbh
MSGYN "Plz Clear All BreakPoints  And  Set Debugging Option Ignore All Excepions Options  !"
cmp $RESULT, 0
je TryAgain
#inc "Get.eXe.PE.Information.osc"
var T0
var T1
var temp
var FixCode1
var FixCode2
var FixCode3
var FixCode4
var FixCode5
var FixCode6
var Skip
var EAX=0
var EAX=1
var EAX=2
var EAX=3
var EAX=4
var IsDebuggerPresent
var JmpAddress
var SpecialFiXed
var SpecialFiXedOver
var bpcnt
var VirtualAlloc
var AllocMemory
var AllocMemory2
var AllocMemory2Size
var AllocMemory3
var AllocMemory3Size
var LoadLibraryA
var CreateRemoteThread
var VirtualFree
var DecodeFinal
var StolenOEP
```

```
//UnhandledExceptionFilter――――――――――――――――――――――――――――――――
gpa "UnhandledExceptionFilter", "KERNEL32.dll"
cmp $RESULT, 0
je Only Win2K/XP
WinXP:
find $RESULT, #0F849600000064A1180000008B4030#
cmp $RESULT, 0
je Win2K
log $RESULT
mov [$RESULT],#E997000000#
jmp CheckRemoteDebuggerPresent
Win2K:
gpa "UnhandledExceptionFilter", "KERNEL32.dll"
find $RESULT, #395DC80F8549020000#
cmp $RESULT, 0
je Only Win2K/XP
log $RESULT
mov [$RESULT],#395DC8EB0490909090#
jmp CreateToolhelp32Snapshot
```

```
//CheckRemoteDebuggerPresent――――――――――――――――――――――――――――――――
CheckRemoteDebuggerPresent:
gpa "CheckRemoteDebuggerPresent", "KERNEL32.dll"
cmp $RESULT, 0
je CreateToolhelp32Snapshot
find $RESULT, #33C040#
cmp $RESULT, 0
je CreateToolhelp32Snapshot
mov [$RESULT], #33C090#
```

```
//CreateToolhelp32Snapshot――――――――――――――――――――――――――――――――
CreateToolhelp32Snapshot:
gpa "CreateToolhelp32Snapshot", "KERNEL32.dll"
cmp $RESULT, 0
je NoFind
mov [$RESULT], #B8FFFFFFFFC20800#
```

```
//CreateRemoteThread――――――――――――――――――――――――――――――――
gpa "CreateRemoteThread", "KERNEL32.dll"
cmp $RESULT, 0
je NoFind
mov [$RESULT], #33C0C21C00#
```

```
//FindWindowA――――――――――――――――――――――――――――――――
gpa "FindWindowA", "USER32.dll"
cmp $RESULT, 0
je NoFind
mov [$RESULT], #33C0C20800#
```

```
//CloseHandle――――――――――――――――――――――――――――――――
gpa "CloseHandle", "KERNEL32.dll"
cmp $RESULT, 0
je NoFind
mov [$RESULT], #C20400#
```

```
//VirtualAlloc――――――――――――――――――――――――――――――――
gpa "VirtualAlloc", "KERNEL32.dll"
cmp $RESULT, 0
findop $RESULT,#C21000#
cmp $RESULT, 0
je NoFind
mov VirtualAlloc,$RESULT
eob VirtualAlloc
bp VirtualAlloc
esto
GoOn0:
esto
VirtualAlloc:
cmp eip,VirtualAlloc
jne GoOn0
inc bpcnt
cmp bpcnt,2
log bpcnt
jb GoOn0
ja AllocMemory3
mov AllocMemory2,eax
mov temp,esp
add temp,08
mov AllocMemory2Size,[temp]
inc bpcnt
log AllocMemory2
log AllocMemory2Size
jmp GoOn0
AllocMemory3:
mov AllocMemory3,eax
mov temp,esp
add temp,08
mov AllocMemory3Size,[temp]
log AllocMemory3
log AllocMemory3Size
bc VirtualAlloc
mov bpcnt,0
```

```
//LoadLibraryA――――――――――――――――――――――――――――――――
FindChance:
gpa "LoadLibraryA", "KERNEL32.dll"
cmp $RESULT, 0
je NoFind
mov LoadLibraryA,$RESULT
eob LoadLibraryA
bpwm LoadLibraryA, 5
esto
LoadLibraryA:
inc bpcnt
find AllocMemory2,#66F7062000#
cmp $RESULT, 0
je FindChance
```

```
//FixedImportingFunction――――――――――――――――――――――――――――――――
log bpcnt
bpmc
mov FixCode1,$RESULT
log FixCode1
//jmp Final
mov [FixCode1],#66F7060800#
/\*
FixCode1:
00908035      66:F706 2000       test word ptr ds:[esi],20
Modified:     66:F706 0800       test word ptr ds:[esi],8    ★
\*/
find FixCode1,#0F84??000000#
cmp $RESULT, 0
je NoFind
mov FixCode2,$RESULT
log FixCode2
mov T0,$RESULT
add T0,2
mov T1,[T0]
add T0,4
add T0,T1
mov JmpAddress,T0
log JmpAddress
eval "jne {JmpAddress}"
asm FixCode2, $RESULT
/\*
FixCode2:
00908040      0F84 ??000000      je 009080DB
Modified:     0F85 95000000      jnz 009080DB  ★
\*/
find FixCode2,#0F84??000000#
cmp $RESULT, 0
je NoFind
mov FixCode3,$RESULT
log FixCode3
eval "je {JmpAddress}"
asm FixCode3, $RESULT
mov temp,FixCode3
add temp,2
fill temp, 4, 90
/\*
FixCode3:
00908085      0F84 88000000      je 00908113
Modified:     7454 90909090      je 009080DB   ★
\*/
find FixCode3,#74??EB??#
cmp $RESULT, 0
je NoFind
mov FixCode4,$RESULT
log FixCode4
eval "je {JmpAddress}"
asm FixCode4, $RESULT
/\*
FixCode4:
009080CE      74 43              je 00908113
Modified:     74 0B              je 009080DB   ★
\*/
find FixCode2,#75??EB#
cmp $RESULT, 0
je NoFind
mov Skip,$RESULT
log Skip
mov [Skip],#EB#
/\*
00908FAC         66:F706 0200       test word ptr ds:[esi],2
00908FB1         EB 03              jmp short 00908FB6
00908FB6         75 47              jnz short 00908FFF
Modified:        EB 47              jmp short 00908FFF   ★
00908FB8         EB 02              jmp short 00908FBC
\*/
find FixCode1,#891F83C30AE9#
cmp $RESULT, 0
je NoFind
mov FixCode5,$RESULT
log FixCode5
fill FixCode5, 2, 90
/\*
00909127      891F               mov dword ptr ds:[edi],ebx
Modified:     9090               NOP           ★
00909129      83C3 0A            add ebx,0A
0090912C      E9 49FFFFFF        jmp 0090907A
\*/
```

```
//IsDebuggerPresent――――――――――――――――――――――――――――――――
gpa "IsDebuggerPresent", "KERNEL32.dll"
cmp $RESULT, 0
je NoFind
find $RESULT,#C3#
cmp $RESULT, 0
je NoFind
mov IsDebuggerPresent,$RESULT
eob IsDebuggerPresent
bp IsDebuggerPresent
```

```
//SpecialImportingFunction――――――――――――――――――――――――――――――――
find FixCode1,#C214008B#
cmp $RESULT, 0
je NoFind
mov SpecialFiXed,$RESULT
log SpecialFiXed
find FixCode1,#FF501850#
cmp $RESULT, 0
je NoFind
mov EAX=3,$RESULT
log EAX=3
find EAX=3,#FF5018EB1C#
cmp $RESULT, 0
je NoFind
mov EAX=0,$RESULT
log EAX=0
find EAX=3,#FF5018EB0D#
cmp $RESULT, 0
je NoFind
mov EAX=1,$RESULT
log EAX=1
find EAX=3,#FF5018C603#
cmp $RESULT, 0
je NoFind
mov EAX=2,$RESULT
log EAX=2
EAX:
eob SpecialImportingFunction
bp SpecialFiXed
bp EAX=0
bp EAX=1
bp EAX=2
bp EAX=3
esto
GoOn1:
log eip
esto
/\*
009090FC     8B46 04            mov eax,dword ptr ds:[esi+4]
009090FF     83F8 00            cmp eax,0
00909102     74 45              je short 00909149
00909104     83F8 01            cmp eax,1
00909107     74 4F              je short 00909158
00909109     83F8 02            cmp eax,2
0090910C     74 59              je short 00909167
0090910E     83F8 03            cmp eax,3
00909111     74 12              je short 00909125
00909113     83F8 04            cmp eax,4
00909116     75 CA              jnz short 009090E2
00909118     8B45 14            mov eax,dword ptr ss:[ebp+14]
0090911B     8B90 E8000000      mov edx,dword ptr ds:[eax+E8]
00909121     8917               mov dword ptr ds:[edi],edx
00909123     EB BD              jmp short 009090E2
00909125     8B45 14            mov eax,dword ptr ss:[ebp+14]
00909128     68 C5B1662D        push 2D66B1C5
0090912D     6A 00              push 0
0090912F     FF50 18            call dword ptr ds:[eax+18]
00909132     50                 push eax
00909133     53                 push ebx
00909134     E8 98020000        call 009093D1
00909139     53                 push ebx
0090913A     E8 19020000        call 00909358
0090913F     8BCB               mov ecx,ebx
00909141     8D5C03 01          lea ebx,dword ptr ds:[ebx+eax+1]
00909145     8BC1               mov eax,ecx
00909147     EB 2B              jmp short 00909174
00909149     8B45 14            mov eax,dword ptr ss:[ebp+14]
0090914C     68 0F1ACF4C        push 4CCF1A0F
00909151     6A 00              push 0
00909153     FF50 18            call dword ptr ds:[eax+18]
00909156     EB 1C              jmp short 00909174
\*/
