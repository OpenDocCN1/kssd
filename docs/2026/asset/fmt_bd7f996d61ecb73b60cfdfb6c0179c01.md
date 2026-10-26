
### 汇编片段

```
0065F562    8A40 09         mov al,byte ptr ds:[eax+9]          //"7"

0065F565    8845 F6         mov byte ptr ss:[ebp-A],al

0065F568    8B45 FC         mov eax,dword ptr ss:[ebp-4]

0065F56B    8A40 10         mov al,byte ptr ds:[eax+10]         //"1"

0065F56E    8845 F7         mov byte ptr ss:[ebp-9],al

0065F571    8B45 FC         mov eax,dword ptr ss:[ebp-4]

0065F574    8A40 12         mov al,byte ptr ds:[eax+12]          //"9"

0065F577    8845 F8         mov byte ptr ss:[ebp-8],al

0065F57A    8B45 FC         mov eax,dword ptr ss:[ebp-4]

0065F57D    8A40 13         mov al,byte ptr ds:[eax+13]          //"B"

0065F580    8845 F9         mov byte ptr ss:[ebp-7],al

0065F583    8B45 FC         mov eax,dword ptr ss:[ebp-4]

0065F586    8A40 1B         mov al,byte ptr ds:[eax+1B]          //"6"

0065F589    8845 FA         mov byte ptr ss:[ebp-6],al

0065F58C    8B45 FC         mov eax,dword ptr ss:[ebp-4]

0065F58F    8A40 1D         mov al,byte ptr ds:[eax+1D]          //"F"

0065F592    8845 FB         mov byte ptr ss:[ebp-5],al

0065F595    8BC3            mov eax,ebx

0065F597    8D55 F4         lea edx,dword ptr ss:[ebp-C]

0065F59A    B9 08000000     mov ecx,8

0065F59F    E8 CC53DAFF     call 1\_.00404970
```

```
..............

...............

0065F5FC    8D55 F8         lea edx,dword ptr ss:[ebp-8]

0065F5FF    8B45 FC         mov eax,dword ptr ss:[ebp-4]

0065F602    E8 C9DAFFFF     call 1\_.0065D0D0

0065F607    8B45 F8         mov eax,dword ptr ss:[ebp-8]

0065F60A    33D2            xor edx,edx

0065F60C    52              push edx

0065F60D    50              push eax

0065F60E    8D55 F0         lea edx,dword ptr ss:[ebp-10]

0065F611    B8 08000000     mov eax,8

0065F616    E8 89A4DAFF     call 1\_.00409AA4               ;通过浮点运算产生数值“19F76797”
```

---------------------------------------------------------------------------------------

```
00409A0D    4E              dec esi

00409A0E    D9F8            fprem

00409A10    DF1C24          fistp word ptr ss:[esp]

00409A13    DCF9            fdiv st(1),st

00409A15    8A0424          mov al,byte ptr ss:[esp]

00409A18    04 30           add al,30

00409A1A    3C 3A           cmp al,3A

00409A1C    72 02           jb short 1\_.00409A20

00409A1E    04 07           add al,7

00409A20    8806            mov byte ptr ds:[esi],al

00409A22    D9C1            fld st(1)

00409A24    D8D3            fcom st(3)

00409A26    9B              wait

00409A27    DFE0            fstsw ax

00409A29    9E              sahf

00409A2A  ^ 73 E1           jnb short 1\_.00409A0D
```

------------------------------------------------------------------------------------------

```
0065F61B    8B45 F0         mov eax,dword ptr ss:[ebp-10]      ; 19F76797

0065F61E    8D55 F4         lea edx,dword ptr ss:[ebp-C]

0065F621    E8 86FEFFFF     call 1\_.0065F4AC  //经过下面与CA719B6F相同的运算

0065F626    8B45 F4         mov eax,dword ptr ss:[ebp-C]       ; 57856

0065F629    50              push eax

0065F62A    8D45 E8         lea eax,dword ptr ss:[ebp-18]

0065F62D    E8 F2FEFFFF     call 1\_.0065F524

0065F632    8B45 E8         mov eax,dword ptr ss:[ebp-18]      ; CA719B6F

0065F635    8D55 EC         lea edx,dword ptr ss:[ebp-14]

0065F638    E8 6FFEFFFF     call 1\_.0065F4AC

0065F63D    8B55 EC         mov edx,dword ptr ss:[ebp-14]      ; 61824

0065F640    58              pop eax                            ; 57856

0065F641    E8 5656DAFF     call 1\_.00404C9C

0065F646    EB 14           jmp short 1\_.0065F65C
```

取每个字符对应的16进制数值，进行运算

1  9  F  7  6  7  9  7

`49*128+57*128+70*128+55*128+54*128+55*128+57*128+55*128`

=`E200`--->转换成十进制值就是`57856`

C  A  7  1  9  B  6  F

=`67*128+65*128+55*128+49*128+57*128+66*128+54*128+70*128`

=`F180`--->转换成十进制值就是`61824`

两个进行比较是否相等，相等则注册成功。

### 申请码的由来

------------------------------------------------------------------------------------

```
0065EFA1    64:8920         mov dword ptr fs:[eax],esp

0065EFA4    8BC3            mov eax,ebx

0065EFA6    BA 50F06500     mov edx,1\_.0065F050                ; ASCII "0123456789ABCDEF"

0065EFAB    E8 2459DAFF     call 1\_.004048D4

0065EFB0    8D45 F8         lea eax,dword ptr ss:[ebp-8]

0065EFB3    E8 28DEFFFF     call 1\_.0065CDE0

0065EFB8    8B45 F8         mov eax,dword ptr ss:[ebp-8]       ;ASCII "I845G/ICH4-P4B53"//取CPU的型号

0065EFBB    BA 6CF06500     mov edx,1\_.0065F06C                ; ASCII "FF"

0065EFC0    E8 D75CDAFF     call 1\_.00404C9C

0065EFC5    75 0A           jnz short 1\_.0065EFD1              ;不知道这是比较什么东东？
..

..

0065EFD9    8D55 F0         lea edx,dword ptr ss:[ebp-10]

0065EFDC    8B45 FC         mov eax,dword ptr ss:[ebp-4]

0065EFDF    E8 28FEFFFF     call 1\_.0065EE0C

0065EFE4    8B45 F0         mov eax,dword ptr ss:[ebp-10]   ;ASCII "02dd44e5eb5b891e49f83d7db745c540"

0065EFE7    8D4D F4         lea ecx,dword ptr ss:[ebp-C]

0065EFEA    BA 08000000     mov edx,8

0065EFEF    E8 4825DEFF     call 1\_.0044153C

0065EFF4    8B45 F4         mov eax,dword ptr ss:[ebp-C]     ;ASCII"b745c540"

0065EFF7    50              push eax
...
```

```
0065F098    8D45 F8         lea eax,dword ptr ss:[ebp-8]

0065F09B    E8 40DDFFFF     call 1\_.0065CDE0

0065F0A0    8B45 F8         mov eax,dword ptr ss:[ebp-8]

0065F0A3    BA F0F16500     mov edx,1\_.0065F1F0                ; ASCII "FF"

0065F0A8    E8 EF5BDAFF     call 1\_.00404C9C

0065F0AD    0F85 89000000   jnz 1\_.0065F13C

0065F13C    68 18F26500     push 1\_.0065F218                   ; ASCII "NEBS-"

0065F141    8D45 E4         lea eax,dword ptr ss:[ebp-1C]

0065F144    50              push eax

0065F145    B9 04000000     mov ecx,4

0065F14A    BA 01000000     mov edx,1

0065F14F    8B45 FC         mov eax,dword ptr ss:[ebp-4]

0065F152    E8 595CDAFF     call 1\_.00404DB0

0065F157    FF75 E4         push dword ptr ss:[ebp-1C]

0065F15A    68 0CF26500     push 1\_.0065F20C

0065F15F    8D45 E0         lea eax,dword ptr ss:[ebp-20]

0065F162    50              push eax

0065F163    B9 04000000     mov ecx,4

0065F141    8D45 E4         lea eax,dword ptr ss:[ebp-1C]

0065F144    50              push eax

0065F145    B9 04000000     mov ecx,4

0065F14A    BA 01000000     mov edx,1

0065F14F    8B45 FC         mov eax,dword ptr ss:[ebp-4]       ;ASCII "4088C74792388222"

0065F152    E8 595CDAFF     call 1\_.00404DB0

0065F157    FF75 E4         push dword ptr ss:[ebp-1C]         ;ASCII "4088"

0065F15A    68 0CF26500     push 1\_.0065F20C

0065F15F    8D45 E0         lea eax,dword ptr ss:[ebp-20]

0065F162    50              push eax

0065F163    B9 04000000     mov ecx,4

0065F168    BA 05000000     mov edx,5

0065F16D    8B45 FC         mov eax,dword ptr ss:[ebp-4]

0065F170    E8 3B5CDAFF     call 1\_.00404DB0

0065F175    FF75 E0         push dword ptr ss:[ebp-20]         ;ASCII "C747"

0065F178    68 0CF26500     push 1\_.0065F20C

0065F17D    8D45 DC         lea eax,dword ptr ss:[ebp-24]

0065F180    50              push eax

0065F181    B9 04000000     mov ecx,4

0065F186    BA 09000000     mov edx,9

0065F18B    8B45 FC         mov eax,dword ptr ss:[ebp-4]

0065F18E    E8 1D5CDAFF     call 1\_.00404DB0

0065F193    FF75 DC         push dword ptr ss:[ebp-24]         ;ASCII "9238"

0065F196    68 0CF26500     push 1\_.0065F20C

0065F19B    8D45 D8         lea eax,dword ptr ss:[ebp-28]

0065F19E    50              push eax

0065F19F    B9 04000000     mov ecx,4

0065F1A4    BA 0D000000     mov edx,0D

0065F1A9    8B45 FC         mov eax,dword ptr ss:[ebp-4]

0065F1AC    E8 FF5BDAFF     call 1\_.00404DB0

0065F1B1    FF75 D8         push dword ptr ss:[ebp-28]         ;ASCII "8222"

0065F1B4    8BC3            mov eax,ebx
....
```

```
0066007E    8B55 FC         mov edx,dword ptr ss:[ebp-4]       ;ASCII "NEBS-4088-C747-9238-8222"组合成申请码的格式

00660081    8B83 14030000   mov eax,dword ptr ds:[ebx+314]

00660087    E8 481EE1FF     call 1\_.00471ED4

0066008C    E8 1BF7FFFF     call 1\_.0065F7AC

00660091    8BD0            mov edx,eax

00660093    8B83 FC020000   mov eax,dword ptr ds:[ebx+2FC]
```
