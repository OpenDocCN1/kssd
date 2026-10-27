
```
00507A33   .  FF15 CC114000 CALL DWORD PTR DS:[<&MSVBVM60.\_\_vbaVarCa>;  MSVBVM60.\_\_vbaVarCat

00507A39   .  50            PUSH EAX

00507A3A   .  FF15 20104000 CALL DWORD PTR DS:[<&MSVBVM60.\_\_vbaStrVa>;  MSVBVM60.\_\_vbaStrVarMove

00507A40   .  8BD0          MOV EDX,EAX                              ;  EAX=0016DCDC, (UNICODE  "MRG101K1C1KD7CMarch3000000000000")

00507A42   .  8D4D C8       LEA ECX,DWORD PTR SS:[EBP-38]

00507A45   .  FF15 88124000 CALL DWORD PTR DS:[<&MSVBVM60.\_\_vbaStrMo>;  MSVBVM60.\_\_vbaStrMove

00507A4B   .  8D4D 80       LEA ECX,DWORD PTR SS:[EBP-80]

00507A4E   .  51            PUSH ECX

00507A4F   .  8D55 90       LEA EDX,DWORD PTR SS:[EBP-70]

00507A52   .  52            PUSH EDX

00507A53   .  8D45 A0       LEA EAX,DWORD PTR SS:[EBP-60]

00507A56   .  50            PUSH EAX

00507A57   .  8D4D B0       LEA ECX,DWORD PTR SS:[EBP-50]

00507A5A   .  51            PUSH ECX

00507A5B   .  6A 04         PUSH 4

00507A5D   .  FF15 30104000 CALL DWORD PTR DS:[<&MSVBVM60.\_\_vbaFreeV>;  MSVBVM60.\_\_vbaFreeVarList

00507A63   .  83C4 14       ADD ESP,14

00507A66   .  C745 FC 06000>MOV DWORD PTR SS:[EBP-4],6

00507A6D   .  66:C785 4CFFF>MOV WORD PTR SS:[EBP-B4],0A              ;  上面获得原始机器码

00507A76   .  8D95 4CFFFFFF LEA EDX,DWORD PTR SS:[EBP-B4]

00507A7C   .  52            PUSH EDX

00507A7D   .  8D45 C8       LEA EAX,DWORD PTR SS:[EBP-38]

00507A80   .  50            PUSH EAX

00507A81   .  E8 FADDFFFF   CALL GtChrdHl.00505880

00507A86   .  8BD0          MOV EDX,EAX

00507A88   .  8D4D C0       LEA ECX,DWORD PTR SS:[EBP-40]

00507A8B   .  FF15 88124000 CALL DWORD PTR DS:[<&MSVBVM60.\_\_vbaStrMo>;  MSVBVM60.\_\_vbaStrMove

00507A91   .  68 EC7A5000   PUSH GtChrdHl.00507AEC                   ;  获得机器码
```

明白了。取得临时机器码后计算得出真正机器码。

```
00507C3C   .  FF15 18114000 CALL DWORD PTR DS:[<&MSVBVM60.\_\_vbaFileC>;  MSVBVM60.\_\_vbaFileClose

00507C42   .  E8 C9FCFFFF   CALL GtChrdHl.00507910                   ;  这个是一个关键函数，应该是获取机器码的。

00507C47   .  8BD0          MOV EDX,EAX                              ;  取得了机器码

00507C49   .  8D4D D8       LEA ECX,DWORD PTR SS:[EBP-28]

00507C4C   .  FFD7          CALL EDI

00507C4E   .  8B55 D8       MOV EDX,DWORD PTR SS:[EBP-28]

00507C51   .  8D4D E0       LEA ECX,DWORD PTR SS:[EBP-20]

00507C54   .  C745 C0 14000>MOV DWORD PTR SS:[EBP-40],14

00507C5B   .  C745 D8 00000>MOV DWORD PTR SS:[EBP-28],0

00507C62   .  FFD7          CALL EDI

00507C64   .  8B4D E4       MOV ECX,DWORD PTR SS:[EBP-1C]            ;  假码

00507C67   .  8D55 C0       LEA EDX,DWORD PTR SS:[EBP-40]            ;  机器码

00507C6A   .  51            PUSH ECX

00507C6B   .  8D45 E0       LEA EAX,DWORD PTR SS:[EBP-20]

00507C6E   .  52            PUSH EDX

00507C6F   .  50            PUSH EAX

00507C70   .  E8 0BDCFFFF   CALL GtChrdHl.00505880                   ;  计算注册码过程，得到真正的注册码

00507C75   .  8BD0          MOV EDX,EAX                              ;  真码，可作内存注册机

00507C77   .  8D4D DC       LEA ECX,DWORD PTR SS:[EBP-24]

00507C7A   .  FFD7          CALL EDI

00507C7C   .  50            PUSH EAX

00507C7D   .  FFD3          CALL EBX

00507C7F   .  8BF0          MOV ESI,EAX

00507C81   .  8D4D D8       LEA ECX,DWORD PTR SS:[EBP-28]

00507C84   .  F7DE          NEG ESI

00507C86   .  8D55 DC       LEA EDX,DWORD PTR SS:[EBP-24]

00507C89   .  51            PUSH ECX

00507C8A   .  1BF6          SBB ESI,ESI

00507C8C   .  8D45 E0       LEA EAX,DWORD PTR SS:[EBP-20]

00507C8F   .  52            PUSH EDX

00507C90   .  46            INC ESI

00507C91   .  50            PUSH EAX

00507C92   .  6A 03         PUSH 3

00507C94   .  F7DE          NEG ESI

00507C96   .  FF15 1C124000 CALL DWORD PTR DS:[<&MSVBVM60.\_\_vbaFreeS>;  MSVBVM60.\_\_vbaFreeStrList

00507C9C   .  83C4 10       ADD ESP,10

00507C9F   .  66:85F6       TEST SI,SI

00507CA2   .  74 07         JE SHORT GtChrdHl.00507CAB               ;  关键跳转
```

====================================================

### 下面来详细跟踪一下注册码生成过程吧：

```
00505880   $  55            PUSH EBP                                 ;  机器码与注册码共用函数

00505881   .  8BEC          MOV EBP,ESP

00505883   .  83EC 0C       SUB ESP,0C

00505886   .  68 06334000 push <jmp.&MSVBVM60.\_\_vbaExceptHandler>       ;  SE handler installation

0050588B   .  64:A1 00000>mov eax,dword ptr fs:[0]

00505891   .  50          push eax

00505892   .  64:8925 000>mov dword ptr fs:[0],esp

00505899   .  81EC 480100>sub esp,148

0050589F   .  53          push ebx

005058A0   .  56          push esi

005058A1   .  57          push edi

005058A2   .  8965 F4     mov dword ptr ss:[ebp-C],esp

005058A5   .  C745 F8 802>mov dword ptr ss:[ebp-8],GtChrdHl.00402F80

005058AC   .  8B45 08     mov eax,dword ptr ss:[ebp+8]

005058AF   .  33DB        xor ebx,ebx

005058B1   .  895D E0     mov dword ptr ss:[ebp-20],ebx

005058B4   .  895D DC     mov dword ptr ss:[ebp-24],ebx

005058B7   .  8B08        mov ecx,dword ptr ds:[eax]                    ;  硬件号"w2CsM2O2Q2"

005058B9   .  895D D8     mov dword ptr ss:[ebp-28],ebx

005058BC   .  51          push ecx

005058BD   .  68 18DB4400 push GtChrdHl.0044DB18
```

省略若干初始化代码...

```
00505B8D   .  6A 04       push 4

00505B8F   .  FFD7        call edi

00505B91   .  83C4 14     add esp,14

00505B94   >  E8 C7090000 call GtChrdHl.00506560

00505B99   .  8B35 881240>mov esi,dword ptr ds:[<&MSVBVM60.\_\_vbaStrMove>;  MSVBVM60.\_\_vbaStrMove

00505B9F   >  66:8B45 E0  mov ax,word ptr ss:[ebp-20]

00505BA3   .  8B4D 0C     mov ecx,dword ptr ss:[ebp+C]

00505BA6   .  66:05 0100  add ax,1

00505BAA   .  66:8B19     mov bx,word ptr ds:[ecx]

00505BAD   .  0F80 A20900>jo GtChrdHl.00506555

00505BB3   .  66:035D E0  add bx,word ptr ss:[ebp-20]

00505BB7   .  8985 DCFEFF>mov dword ptr ss:[ebp-124],eax

00505BBD   .  0F80 920900>jo GtChrdHl.00506555

00505BC3   >  66:C1F8 0F  sar ax,0F

00505BC7   .  8BD0        mov edx,eax

00505BC9   .  B9 00010000 mov ecx,100                                   ;  100位

00505BCE   .  33D1        xor edx,ecx

00505BD0   .  33C3        xor eax,ebx

00505BD2   .  66:3BC2     cmp ax,dx                                     ;  第15位

00505BD5   .  7F 66       jg short GtChrdHl.00505C3D

00505BD7   .  0FBFC3      movsx eax,bx

00505BDA   .  48          dec eax

00505BDB   .  3BC1        cmp eax,ecx

00505BDD   .  8985 E4FEFF>mov dword ptr ss:[ebp-11C],eax

00505BE3   .  72 0C       jb short GtChrdHl.00505BF1

00505BE5   .  FF15 201140>call dword ptr ds:[<&MSVBVM60.\_\_vbaGenerateBo>;  MSVBVM60.\_\_vbaGenerateBoundsError

00505BEB   .  8B85 E4FEFF>mov eax,dword ptr ss:[ebp-11C]

00505BF1   >  8B15 B06051>mov edx,dword ptr ds:[5160B0]                 ;  长字串
```

这里有个表，也就是长字符串了。

`str="N  Z^%@\*&(HGUINKNUIGBF%kfinkcG76U^GUI766f^%SyI@(Ifdkj(\*8g7^&f65fFdIUYf6I^GjkdBUDhuipdfyig% $fjL\*\*.x02ytPHDOIDTphHPPiofDmllfl44190$#&\*JKho4o00wjdhFDK\*jo:;JUIp(B[ApIuy^R&ubrYUi%6ge4uyU7OTd la;fyp0s0-2htdf!Jhfu7IK (\*$$#g%(m0FG[D688g^6fI8HDGF9[y44NT87%u9\*\*r4EU."`

```
00505BF7   .  8B4D D4     mov ecx,dword ptr ss:[ebp-2C]

00505BFA   .  51          push ecx

00505BFB   .  8A0410      mov al,byte ptr ds:[eax+edx]                  ;  逐位取ascii G->47

00505BFE   .  50          push eax

00505BFF   .  FF15 881140>call dword ptr ds:[<&MSVBVM60.\_\_vbaStrUI1>]   ;  MSVBVM60.\_\_vbaStrUI1

00505C05   .  8BD0        mov edx,eax                                   ;  变成字符串 47->71

00505C07   .  8D4D C4     lea ecx,dword ptr ss:[ebp-3C]

00505C0A   .  FFD6        call esi

00505C0C   .  50          push eax

00505C0D   .  FF15 641040>call dword ptr ds:[<&MSVBVM60.\_\_vbaStrCat>]   ;  MSVBVM60.\_\_vbaStrCat

00505C13   .  8BD0        mov edx,eax

00505C15   .  8D4D D4     lea ecx,dword ptr ss:[ebp-2C]

00505C18   .  FFD6        call esi

00505C1A   .  8D4D C4     lea ecx,dword ptr ss:[ebp-3C]

00505C1D   .  FF15 CC1240>call dword ptr ds:[<&MSVBVM60.\_\_vbaFreeStr>]  ;  MSVBVM60.\_\_vbaFreeStr

00505C23   .  66:8B8D DCF>mov cx,word ptr ss:[ebp-124]

00505C2A   .  8B85 DCFEFF>mov eax,dword ptr ss:[ebp-124]
```
