
```asm
004175F9  |.  E8 F6FE0200   call    004474F4

004175FE  |.  50            push    eax

004175FF  |.  68 2C010000   push    12C

00417604  |.  E8 909E0000   call    00421499

00417609  |.  83C4 0C       add     esp, 0C

0041760C  |.  6A FF         push    -1

0041760E  |.  8D4D EC       lea     ecx, dword ptr [ebp-14]

00417611  |.  E8 2DFF0200   call    00447543

00417616  |.  51            push    ecx

00417617  |.  8BCC          mov     ecx, esp

00417619  |.  8965 DC       mov     dword ptr [ebp-24], esp

0041761C  |.  8D45 EC       lea     eax, dword ptr [ebp-14]

0041761F  |.  50            push    eax

00417620  |.  E8 A1F80200   call    00446EC6

00417625  |.  8945 BC       mov     dword ptr [ebp-44], eax

00417628  |.  8B4D BC       mov     ecx, dword ptr [ebp-44]

0041762B  |.  894D B8       mov     dword ptr [ebp-48], ecx

0041762E  |.  C645 FC 04    mov     byte ptr [ebp-4], 4

00417632  |.  51            push    ecx

00417633  |.  8BCC          mov     ecx, esp

00417635  |.  8965 D8       mov     dword ptr [ebp-28], esp

00417638  |.  8D55 08       lea     edx, dword ptr [ebp+8]

0041763B  |.  52            push    edx

0041763C  |.  E8 85F80200   call    00446EC6

00417641  |.  8945 B4       mov     dword ptr [ebp-4C], eax

00417644  |.  8D45 F0       lea     eax, dword ptr [ebp-10]

00417647  |.  50            push    eax

00417648  |.  8B4D CC       mov     ecx, dword ptr [ebp-34]

0041764B  |.  C645 FC 03    mov     byte ptr [ebp-4], 3

0041764F  |.  E8 A9000000   call    004176FD                         ;  算法CALL

00417654  |.  8945 B0       mov     dword ptr [ebp-50], eax

00417657  |.  C645 FC 05    mov     byte ptr [ebp-4], 5

0041765B  |.  8D4D 0C       lea     ecx, dword ptr [ebp+C]

0041765E  |.  51            push    ecx

0041765F  |.  8D55 F0       lea     edx, dword ptr [ebp-10]

00417662  |.  52            push    edx

00417663  |.  E8 7805FFFF   call    00407BE0                         ;  上面计算得到注册码与试炼码比较

00417668  |.  25 FF000000   and     eax, 0FF

0041766D  |.  85C0          test    eax, eax

0041766F  |.  74 3F         je      short 004176B0

00417671  |.  C745 D4 01000>mov     dword ptr [ebp-2C], 1

00417678  |.  C645 FC 03    mov     byte ptr [ebp-4], 3

0041767C  |.  8D4D F0       lea     ecx, dword ptr [ebp-10]

0041767F  |.  E8 CDFA0200   call    00447151

00417684  |.  C645 FC 01    mov     byte ptr [ebp-4], 1

00417688  |.  8D4D EC       lea     ecx, dword ptr [ebp-14]

0041768B  |.  E8 C1FA0200   call    00447151

00417690  |.  C645 FC 00    mov     byte ptr [ebp-4], 0

00417694  |.  8D4D 08       lea     ecx, dword ptr [ebp+8]

00417697  |.  E8 B5FA0200   call    00447151

0041769C  |.  C745 FC FFFFF>mov     dword ptr [ebp-4], -1

004176A3  |.  8D4D 0C       lea     ecx, dword ptr [ebp+C]

004176A6  |.  E8 A6FA0200   call    00447151

004176AB  |.  8B45 D4       mov     eax, dword ptr [ebp-2C]

004176AE  |.  EB 3D         jmp     short 004176ED

004176B0  |>  C745 D0 00000>mov     dword ptr [ebp-30], 0

004176B7  |.  C645 FC 03    mov     byte ptr [ebp-4], 3

004176BB  |.  8D4D F0       lea     ecx, dword ptr [ebp-10]

004176BE  |.  E8 8EFA0200   call    00447151

004176C3  |.  C645 FC 01    mov     byte ptr [ebp-4], 1

004176C7  |.  8D4D EC       lea     ecx, dword ptr [ebp-14]

004176CA  |.  E8 82FA0200   call    00447151

004176CF  |.  C645 FC 00    mov     byte ptr [ebp-4], 0

004176D3  |.  8D4D 08       lea     ecx, dword ptr [ebp+8]

004176D6  |.  E8 76FA0200   call    00447151

004176DB  |.  C745 FC FFFFF>mov     dword ptr [ebp-4], -1

004176E2  |.  8D4D 0C       lea     ecx, dword ptr [ebp+C]

004176E5  |.  E8 67FA0200   call    00447151

004176EA  |.  8B45 D0       mov     eax, dword ptr [ebp-30]

004176ED  |>  8B4D F4       mov     ecx, dword ptr [ebp-C]

004176F0  |.  64:890D 00000>mov     dword ptr fs:[0], ecx

004176F7  |.  8BE5          mov     esp, ebp

004176F9  |.  5D            pop     ebp

004176FA  \.  C2 0800       retn    8
```

***

进入第3组注册码算法`CALL 004176FD`

***

```asm
004176FD  /$  55            push    ebp

004176FE  |.  8BEC          mov     ebp, esp

00417700  |.  6A FF         push    -1

00417702  |.  68 9FF24400   push    0044F29F                         ;  SE 处理程序安装

00417707  |.  64:A1 0000000>mov     eax, dword ptr fs:[0]

0041770D  |.  50            push    eax

0041770E  |.  64:8925 00000>mov     dword ptr fs:[0], esp

00417715  |.  83EC 30       sub     esp, 30

00417718  |.  894D CC       mov     dword ptr [ebp-34], ecx

0041771B  |.  C745 D0 00000>mov     dword ptr [ebp-30], 0

00417722  |.  C745 FC 02000>mov     dword ptr [ebp-4], 2

00417729  |.  8D4D E4       lea     ecx, dword ptr [ebp-1C]

0041772C  |.  E8 7FA4FEFF   call    00401BB0

00417731  |.  C645 FC 03    mov     byte ptr [ebp-4], 3

00417735  |.  8D4D D8       lea     ecx, dword ptr [ebp-28]

00417738  |.  E8 73A4FEFF   call    00401BB0

0041773D  |.  C645 FC 04    mov     byte ptr [ebp-4], 4

00417741  |.  C745 E8 00000>mov     dword ptr [ebp-18], 0

00417748  |.  8D45 10       lea     eax, dword ptr [ebp+10]

0041774B  |.  50            push    eax

0041774C  |.  8D4D 0C       lea     ecx, dword ptr [ebp+C]

0041774F  |.  51            push    ecx

00417750  |.  8D55 D4       lea     edx, dword ptr [ebp-2C]

00417753  |.  52            push    edx

00417754  |.  E8 9AFB0200   call    004472F3

00417759  |.  8945 C8       mov     dword ptr [ebp-38], eax

0041775C  |.  8B45 C8       mov     eax, dword ptr [ebp-38]

0041775F  |.  8945 C4       mov     dword ptr [ebp-3C], eax

00417762  |.  C645 FC 05    mov     byte ptr [ebp-4], 5

00417766  |.  8B4D C4       mov     ecx, dword ptr [ebp-3C]

00417769  |.  51            push    ecx

0041776A  |.  8D4D 0C       lea     ecx, dword ptr [ebp+C]

0041776D  |.  E8 CCFA0200   call    0044723E

00417772  |.  C645 FC 04    mov     byte ptr [ebp-4], 4

00417776  |.  8D4D D4       lea     ecx, dword ptr [ebp-2C]

00417779  |.  E8 D3F90200   call    00447151

0041777E  |.  8D4D 0C       lea     ecx, dword ptr [ebp+C]

00417781  |.  E8 4ABBFEFF   call    004032D0

00417786  |.  8945 EC       mov     dword ptr [ebp-14], eax

00417789  |.  68 00010000   push    100

0041778E  |.  8D4D 10       lea     ecx, dword ptr [ebp+10]

00417791  |.  E8 5EFD0200   call    004474F4

00417796  |.  50            push    eax

00417797  |.  E8 E1A60000   call    00421E7D

0041779C  |.  83C4 04       add     esp, 4

0041779F  |.  8945 DC       mov     dword ptr [ebp-24], eax

004177A2  |.  6A FF         push    -1

004177A4  |.  8D4D 10       lea     ecx, dword ptr [ebp+10]

004177A7  |.  E8 97FD0200   call    00447543

004177AC  |.  68 00010000   push    100

004177B1  |.  8D4D 0C       lea     ecx, dword ptr [ebp+C]

004177B4  |.  E8 3BFD0200   call    004474F4                         ;  试炼码与固定字符串“300”连接

004177B9  |.  8945 F0       mov     dword ptr [ebp-10], eax

004177BC  |.  C745 E0 00000>mov     dword ptr [ebp-20], 0

004177C3  |.  EB 09         jmp     short 004177CE

004177C5  |>  8B55 E0       /mov     edx, dword ptr [ebp-20]         ;  ebp-20的值传送回edx

004177C8  |.  83C2 01       |add     edx, 1                          ;  edx值增1

004177CB  |.  8955 E0       |mov     dword ptr [ebp-20], edx         ;  edx值放入ebp-20,以便于后面参与计算
```
