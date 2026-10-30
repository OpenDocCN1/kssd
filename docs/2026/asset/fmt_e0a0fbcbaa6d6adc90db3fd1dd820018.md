
```
004B8690    53              push ebx

004B8691    8BDC            mov ebx,esp

004B8693    83EC 08         sub esp,0x8

004B8696    83E4 F0         and esp,-0x10

004B8699    83C4 04         add esp,0x4

004B869C    55              push ebp

004B869D    8B6B 04         mov ebp,dword ptr ds:[ebx+0x4]           ; 九重妖塔.004B853C

004B86A0    896C24 04       mov dword ptr ss:[esp+0x4],ebp

004B86A4    8BEC            mov ebp,esp

004B86A6    83EC 68         sub esp,0x68

004B86A9    A1 44304B00     mov eax,dword ptr ds:[0x4B3044]

004B86AE    33C5            xor eax,ebp

004B86B0    8945 FC         mov dword ptr ss:[ebp-0x4],eax

004B86B3    8B15 E4374B00   mov edx,dword ptr ds:[0x4B37E4]

004B86B9    0315 F4384C00   add edx,dword ptr ds:[0x4C38F4]          ; 九重妖塔.00400000

004B86BF    8BC2            mov eax,edx                              ; 九重妖塔.004B2AA5

004B86C1    8915 C0374B00   mov dword ptr ds:[0x4B37C0],edx          ; 九重妖塔.004B2AA5

004B86C7    2B05 D4374B00   sub eax,dword ptr ds:[0x4B37D4]

004B86CD    A3 CC374B00     mov dword ptr ds:[0x4B37CC],eax

004B86D2    A1 DC374B00     mov eax,dword ptr ds:[0x4B37DC]

004B86D7    0305 F4384C00   add eax,dword ptr ds:[0x4C38F4]          ; 九重妖塔.00400000

004B86DD    56              push esi

004B86DE    57              push edi

004B86DF    8B3D B4374B00   mov edi,dword ptr ds:[0x4B37B4]

004B86E5    33F6            xor esi,esi

004B86E7    A3 B8374B00     mov dword ptr ds:[0x4B37B8],eax

004B86EC    33C0            xor eax,eax

004B86EE    8945 AC         mov dword ptr ss:[ebp-0x54],eax

004B86F1    85FF            test edi,edi

004B86F3    74 40           je short 九重妖塔.004B8735

004B86F5    8D0C32          lea ecx,dword ptr ds:[edx+esi]

004B86F8    894D A8         mov dword ptr ss:[ebp-0x58],ecx

004B86FB    8A09            mov cl,byte ptr ds:[ecx]

004B86FD    84C9            test cl,cl

004B86FF    74 2A           je short 九重妖塔.004B872B

004B8701    83E0 03         and eax,0x3

004B8704    C0C9 02         ror cl,0x2

004B8707    3288 C4374B00   xor cl,byte ptr ds:[eax+0x4B37C4]

004B870D    8A45 AC         mov al,byte ptr ss:[ebp-0x54]

004B8710    FEC0            inc al

004B8712    32C8            xor cl,al

004B8714    8B45 A8         mov eax,dword ptr ss:[ebp-0x58]

004B8717    8808            mov byte ptr ds:[eax],cl                 ; DLL名称解码

004B8719    8B45 AC         mov eax,dword ptr ss:[ebp-0x54]

004B871C    8B15 C0374B00   mov edx,dword ptr ds:[0x4B37C0]          ; 九重妖塔.004B2AA5

004B8722    40              inc eax

004B8723    8B3D B4374B00   mov edi,dword ptr ds:[0x4B37B4]

004B8729    EB 02           jmp short 九重妖塔.004B872D

004B872B    33C0            xor eax,eax

004B872D    46              inc esi

004B872E    8945 AC         mov dword ptr ss:[ebp-0x54],eax

004B8731    3BF7            cmp esi,edi

004B8733  ^ 72 C0           jb short 九重妖塔.004B86F5

004B8735    833D C8374B00 0>cmp dword ptr ds:[0x4B37C8],0x0

004B873C    C745 AC 0000000>mov dword ptr ss:[ebp-0x54],0x0

004B8743    0F86 9B020000   jbe 九重妖塔.004B89E4

004B8749    33FF            xor edi,edi

004B874B    897D A8         mov dword ptr ss:[ebp-0x58],edi

004B874E    66:90           nop

004B8750    8B35 B8374B00   mov esi,dword ptr ds:[0x4B37B8]          ; 九重妖塔.004C6000

004B8756    F6043E 01       test byte ptr ds:[esi+edi],0x1           ; DLL分为2类，用2种不同的方式解码

004B875A    8B743E 04       mov esi,dword ptr ds:[esi+edi+0x4]

004B875E    0F84 02010000   je 九重妖塔.004B8866

004B8764    03F2            add esi,edx                              ; 九重妖塔.004B2AA5

004B8766    56              push esi                                 ; ----第1种解码方式----

004B8767    FF15 D8384C00   call dword ptr ds:[0x4C38D8]             ; kernel32.GetModuleHandleA

004B876D    8BC8            mov ecx,eax

004B876F    85C9            test ecx,ecx

004B8771    75 09           jnz short 九重妖塔.004B877C

004B8773    56              push esi

004B8774    FF15 DC384C00   call dword ptr ds:[0x4C38DC]             ; kernel32.LoadLibraryA

004B877A    8BC8            mov ecx,eax

004B877C    A1 B8374B00     mov eax,dword ptr ds:[0x4B37B8]

004B8781    8B0407          mov eax,dword ptr ds:[edi+eax]

004B8784    D1E8            shr eax,1

004B8786    50              push eax

004B8787    51              push ecx

004B8788    FF15 E0384C00   call dword ptr ds:[0x4C38E0]             ; kernel32.GetProcAddress

004B878E    6A 40           push 0x40                                ; 上面获取到真实API地址

004B8790    68 00300000     push 0x3000

004B8795    6A 20           push 0x20

004B8797    6A 00           push 0x0

004B8799    8BF8            mov edi,eax

004B879B    C745 D0 E801000>mov dword ptr ss:[ebp-0x30],0x1E8        ; 以下为第1段空间代码

004B87A2    C745 D4 00E958E>mov dword ptr ss:[ebp-0x2C],0xEB58E900

004B87A9    66:C745 D8 01E8 mov word ptr ss:[ebp-0x28],0xE801

004B87AF    C645 DA B8      mov byte ptr ss:[ebp-0x26],0xB8

004B87B3    C745 DF EB01153>mov dword ptr ss:[ebp-0x21],0x351501EB

004B87BA    C745 E3 1906172>mov dword ptr ss:[ebp-0x1D],0x20170619

004B87C1    C745 E7 EB01FF5>mov dword ptr ss:[ebp-0x19],0x50FF01EB

004B87C8    C745 EB EB02FF1>mov dword ptr ss:[ebp-0x15],0x15FF02EB

004B87CF    C645 EF C3      mov byte ptr ss:[ebp-0x11],0xC3

004B87D3    FF15 D0384C00   call dword ptr ds:[0x4C38D0]             ; kernel32.VirtualAlloc

004B87D9    6A 40           push 0x40                                ; 申请第1段空间

004B87DB    68 00300000     push 0x3000

004B87E0    6A 20           push 0x20

004B87E2    6A 00           push 0x0

004B87E4    8BF0            mov esi,eax

004B87E6    C745 B0 E801000>mov dword ptr ss:[ebp-0x50],0x1E8        ; 以下为第2段空间代码

004B87ED    C745 B4 00E958E>mov dword ptr ss:[ebp-0x4C],0xEB58E900

004B87F4    66:C745 B8 01E8 mov word ptr ss:[ebp-0x48],0xE801

004B87FA    C645 BA B8      mov byte ptr ss:[ebp-0x46],0xB8

004B87FE    C745 BF EB01153>mov dword ptr ss:[ebp-0x41],0x351501EB

004B8805    C745 C3 2605891>mov dword ptr ss:[ebp-0x3D],0x19890526

004B880C    C745 C7 EB01FF5>mov dword ptr ss:[ebp-0x39],0x50FF01EB

004B8813    C745 CB EB02FF1>mov dword ptr ss:[ebp-0x35],0x15FF02EB

004B881A    C645 CF C3      mov byte ptr ss:[ebp-0x31],0xC3

004B881E    FF15 D0384C00   call dword ptr ds:[0x4C38D0]             ; kernel32.VirtualAlloc

004B8824    8BC8            mov ecx,eax                              ; 申请第2段空间

004B8826    81F1 19061720   xor ecx,0x20170619                       ; 第2段地址入口加密

004B882C    81F7 26058919   xor edi,0x19890526                       ; API地址加密

004B8832    894D DB         mov dword ptr ss:[ebp-0x25],ecx          ; 写入加密后的第2段地址入口

004B8835    0F1045 D0       movups xmm0,dqword ptr ss:[ebp-0x30]

004B8839    8B4D AC         mov ecx,dword ptr ss:[ebp-0x54]

004B883C    897D BB         mov dword ptr ss:[ebp-0x45],edi          ; 写入加密后的API地址

004B883F    0F1106          movups dqword ptr ds:[esi],xmm0          ; 第1段空间代码写入

004B8842    0F1045 E0       movups xmm0,dqword ptr ss:[ebp-0x20]

004B8846    0F1146 10       movups dqword ptr ds:[esi+0x10],xmm0

004B884A    0F1045 B0       movups xmm0,dqword ptr ss:[ebp-0x50]

004B884E    0F1100          movups dqword ptr ds:[eax],xmm0          ; 第2段空间代码写入

004B8851    0F1045 C0       movups xmm0,dqword ptr ss:[ebp-0x40]

004B8855    0F1140 10       movups dqword ptr ds:[eax+0x10],xmm0

004B8859    A1 E4384C00     mov eax,dword ptr ds:[0x4C38E4]

004B885E    893488          mov dword ptr ds:[eax+ecx*4],esi         ; 第1段地址入口写入到临时IAT中

004B8861    E9 56010000     jmp 九重妖塔.004B89BC

004B8866    03F2            add esi,edx                              ; 九重妖塔.004B2AA5

004B8868    56              push esi                                 ; ----第2种解码方式----

004B8869    FF15 D8384C00   call dword ptr ds:[0x4C38D8]             ; kernel32.GetModuleHandleA

004B886F    8BC8            mov ecx,eax

004B8871    894D A4         mov dword ptr ss:[ebp-0x5C],ecx

004B8874    85C9            test ecx,ecx

004B8876    75 0A           jnz short 九重妖塔.004B8882

004B8878    56              push esi

004B8879    FF15 DC384C00   call dword ptr ds:[0x4C38DC]             ; kernel32.LoadLibraryA

004B887F    8945 A4         mov dword ptr ss:[ebp-0x5C],eax

004B8882    A1 B8374B00     mov eax,dword ptr ds:[0x4B37B8]

004B8887    8B7407 0C       mov esi,dword ptr ds:[edi+eax+0xC]

004B888B    33C0            xor eax,eax

004B888D    0335 CC374B00   add esi,dword ptr ds:[0x4B37CC]          ; 九重妖塔.004B2000

004B8893    8975 9C         mov dword ptr ss:[ebp-0x64],esi

004B8896    8945 A0         mov dword ptr ss:[ebp-0x60],eax

004B8899    3806            cmp byte ptr ds:[esi],al

004B889B    74 26           je short 九重妖塔.004B88C3

004B889D    0f1f00          nop dword ptr ds:[eax]

004B88A0    8A1430          mov dl,byte ptr ds:[eax+esi]

004B88A3    8BC8            mov ecx,eax

004B88A5    83E1 03         and ecx,0x3

004B88A8    C0CA 03         ror dl,0x3

004B88AB    3291 D0374B00   xor dl,byte ptr ds:[ecx+0x4B37D0]

004B88B1    8D48 01         lea ecx,dword ptr ds:[eax+0x1]

004B88B4    32D1            xor dl,cl

004B88B6    881430          mov byte ptr ds:[eax+esi],dl             ; API名称解密

004B88B9    40              inc eax

004B88BA    803C30 00       cmp byte ptr ds:[eax+esi],0x0

004B88BE  ^ 75 E0           jnz short 九重妖塔.004B88A0

004B88C0    8945 A0         mov dword ptr ss:[ebp-0x60],eax

004B88C3    56              push esi

004B88C4    FF75 A4         push dword ptr ss:[ebp-0x5C]             ; kernel32.76900000

004B88C7    FF15 E0384C00   call dword ptr ds:[0x4C38E0]             ; kernel32.GetProcAddress

004B88CD    6A 40           push 0x40                                ; 上面获取到真实API地址

004B88CF    68 00300000     push 0x3000

004B88D4    6A 20           push 0x20

004B88D6    6A 00           push 0x0

004B88D8    8BF8            mov edi,eax

004B88DA    C745 B0 E801000>mov dword ptr ss:[ebp-0x50],0x1E8        ; 以下为第1段空间代码

004B88E1    C745 B4 00E958E>mov dword ptr ss:[ebp-0x4C],0xEB58E900

004B88E8    66:C745 B8 01E8 mov word ptr ss:[ebp-0x48],0xE801

004B88EE    C645 BA B8      mov byte ptr ss:[ebp-0x46],0xB8

004B88F2    C745 BF EB01153>mov dword ptr ss:[ebp-0x41],0x351501EB

004B88F9    C745 C3 1906172>mov dword ptr ss:[ebp-0x3D],0x20170619

004B8900    C745 C7 EB01FF5>mov dword ptr ss:[ebp-0x39],0x50FF01EB

004B8907    C745 CB EB02FF1>mov dword ptr ss:[ebp-0x35],0x15FF02EB

004B890E    C645 CF C3      mov byte ptr ss:[ebp-0x31],0xC3

004B8912    FF15 D0384C00   call dword ptr ds:[0x4C38D0]             ; kernel32.VirtualAlloc

004B8918    6A 40           push 0x40                                ; 申请第1段空间

004B891A    68 00300000     push 0x3000

004B891F    6A 20           push 0x20

004B8921    6A 00           push 0x0

004B8923    8BF0            mov esi,eax

004B8925    C745 D0 E801000>mov dword ptr ss:[ebp-0x30],0x1E8        ; 以下为第2段空间代码

004B892C    C745 D4 00E958E>mov dword ptr ss:[ebp-0x2C],0xEB58E900

004B8933    66:C745 D8 01E8 mov word ptr ss:[ebp-0x28],0xE801

004B8939    C645 DA B8      mov byte ptr ss:[ebp-0x26],0xB8

004B893D    C745 DF EB01153>mov dword ptr ss:[ebp-0x21],0x351501EB

004B8944    C745 E3 2605891>mov dword ptr ss:[ebp-0x1D],0x19890526

004B894B    C745 E7 EB01FF5>mov dword ptr ss:[ebp-0x19],0x50FF01EB

004B8952    C745 EB EB02FF1>mov dword ptr ss:[ebp-0x15],0x15FF02EB

004B8959    C645 EF C3      mov byte ptr ss:[ebp-0x11],0xC3

004B895D    FF15 D0384C00   call dword ptr ds:[0x4C38D0]             ; kernel32.VirtualAlloc

004B8963    8BC8            mov ecx,eax                              ; 申请第2段空间

004B8965    81F7 26058919   xor edi,0x19890526                       ; API地址加密

004B896B    81F1 19061720   xor ecx,0x20170619                       ; 第2段地址入口加密

004B8971    897D DB         mov dword ptr ss:[ebp-0x25],edi          ; 写入加密后的API地址

004B8974    894D BB         mov dword ptr ss:[ebp-0x45],ecx          ; 写入加密后的第2段地址入口

004B8977    0F1045 B0       movups xmm0,dqword ptr ss:[ebp-0x50]

004B897B    8B4D AC         mov ecx,dword ptr ss:[ebp-0x54]

004B897E    0F1106          movups dqword ptr ds:[esi],xmm0          ; 第1段空间代码写入

004B8981    0F1045 C0       movups xmm0,dqword ptr ss:[ebp-0x40]

004B8985    0F1146 10       movups dqword ptr ds:[esi+0x10],xmm0

004B8989    0F1045 D0       movups xmm0,dqword ptr ss:[ebp-0x30]

004B898D    0F1100          movups dqword ptr ds:[eax],xmm0          ; 第2段空间代码写入

004B8990    0F1045 E0       movups xmm0,dqword ptr ss:[ebp-0x20]

004B8994    0F1140 10       movups dqword ptr ds:[eax+0x10],xmm0

004B8998    A1 E4384C00     mov eax,dword ptr ds:[0x4C38E4]

004B899D    893488          mov dword ptr ds:[eax+ecx*4],esi         ; 第1段地址入口写入到临时IAT中

004B89A0    8B45 A0         mov eax,dword ptr ss:[ebp-0x60]

004B89A3    48              dec eax

004B89A4    0345 9C         add eax,dword ptr ss:[ebp-0x64]          ; 九重妖塔.004B228D

004B89A7    8038 00         cmp byte ptr ds:[eax],0x0

004B89AA    74 10           je short 九重妖塔.004B89BC

004B89AC    0f1f40 00       nop dword ptr ds:[eax]

004B89B0    8D40 FF         lea eax,dword ptr ds:[eax-0x1]

004B89B3    C640 01 00      mov byte ptr ds:[eax+0x1],0x0            ; 把API名称清0

004B89B7    8038 00         cmp byte ptr ds:[eax],0x0

004B89BA  ^ 75 F4           jnz short 九重妖塔.004B89B0

004B89BC    8B45 AC         mov eax,dword ptr ss:[ebp-0x54]

004B89BF    8B7D A8         mov edi,dword ptr ss:[ebp-0x58]

004B89C2    40              inc eax

004B89C3    8B15 C0374B00   mov edx,dword ptr ds:[0x4B37C0]          ; 九重妖塔.004B2AA5

004B89C9    83C7 10         add edi,0x10

004B89CC    8945 AC         mov dword ptr ss:[ebp-0x54],eax

004B89CF    897D A8         mov dword ptr ss:[ebp-0x58],edi

004B89D2    3B05 C8374B00   cmp eax,dword ptr ds:[0x4B37C8]          ; DLL为的API解码完毕？

004B89D8  ^ 0F82 72FDFFFF   jb 九重妖塔.004B8750

004B89DE    8B3D B4374B00   mov edi,dword ptr ds:[0x4B37B4]

004B89E4    33C0            xor eax,eax

004B89E6    85FF            test edi,edi

004B89E8    74 1B           je short 九重妖塔.004B8A05

004B89EA    66:0f1f4400 00  nop word ptr ds:[eax+eax]

004B89F0    C60410 00       mov byte ptr ds:[eax+edx],0x0            ; DLL名称清0

004B89F4    40              inc eax

004B89F5    3B05 B4374B00   cmp eax,dword ptr ds:[0x4B37B4]

004B89FB    73 08           jnb short 九重妖塔.004B8A05

004B89FD    8B15 C0374B00   mov edx,dword ptr ds:[0x4B37C0]          ; 九重妖塔.004B2AA5

004B8A03  ^ EB EB           jmp short 九重妖塔.004B89F0

004B8A05    8B4D FC         mov ecx,dword ptr ss:[ebp-0x4]

004B8A08    5F              pop edi

004B8A09    33CD            xor ecx,ebp

004B8A0B    5E              pop esi

004B8A0C    E8 AF010000     call 九重妖塔.004B8BC0

004B8A11    8BE5            mov esp,ebp

004B8A13    5D              pop ebp

004B8A14    8BE3            mov esp,ebx

004B8A16    5B              pop ebx

004B8A17    C3              retn
```
