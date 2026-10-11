
# 计算机软件水平考试测试系统 2005 V5.0 网络工程师版

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-695.htm`](https://www.pediy.com/kssd/pediy07/pediy7-695.htm)

软件大小： 19465 KB  
软件语言： 简体中文  
软件类别： 国产软件 / 共享版 / 考试系统  
应用平台： `Win9x/NT/2000/XP`  
界面预览： 无  
加入时间： 2005-02-27 10:04:28  
下载次数： 27337  
下载地址： `http://www.skycn.com/soft/9191.html`  
加壳方式： `PE-Armor V0.46` -> `Hying *`

这个 `Hying` 旧版的，估计版本应该是最老的

`OD` 载入,略去所有异常：

入口：

```
0073F000 >  E8 AA000000     call NE_St.0073F0AF
0073F005    2D F0330000     sub eax,33F0
0073F00A    0000           add byte ptr ds:[eax],al
0073F00C    0000           add byte ptr ds:[eax],al
0073F00E    0000           add byte ptr ds:[eax],al
0073F010    003D F033002D  add byte ptr ds:[2D0033F0],bh
0073F016    F0:3300        lock xor eax,dword ptr ds:[eax] ; 不允许锁定前缀
0073F019    0000           add byte ptr ds:[eax],al
0073F01B    0000           add byte ptr ds:[eax],al
0073F01D    0000           add byte ptr ds:[eax],al
```

`Alt+M` 打开内存镜象，如下:

```
--------------------------------------------------------------------
内存镜像

地址    大小    Owner    区段        类型    访问    初始访问
00400000 00001000 MASM32  PE header   Imag  R     RWE
00401000 00003000 MASM32  code        Imag  R     RWE
00404000 00001000 MASM32  resources   Imag  R     RWE
00405000 00001000 MASM32  SFX,data,imports Imag R RWE
```

在 `code` 区段下内存写入断点，`F9` 运行，中断在如下:

```
0073F105    A4            movs byte ptr es:[edi],byte ptr ds>
0073F106    B3 02         mov bl,2
0073F108    E8 6D000000   call NE_St.0073F17A
0073F10D  ^ 73 F6         jnb short NE_St.0073F105
0073F10F    33C9          xor ecx,ecx
0073F111    E8 64000000   call NE_St.0073F17A
0073F116    73 1C         jnb short NE_St.0073F134
```

下命令 `bp GetProcAddress`, `F9` 运行，中断后取消断点，

`Alt+F9` 返回在如下:

```
0037302B    8907          mov dword ptr ds:[edi],eax  ; ntdll.RtlDeleteCriticalSection //中断在这里
0037302D    5A            pop edx
0037302E    0FB642 FF     movzx eax,byte ptr ds:[edx-1]
00373032    03D0          add edx,eax
00373034    42            inc edx
00373035    83C7 04       add edi,4
00373038    59            pop ecx
00373039  ^ E2 CA         loopd short 00373005
0037303B  ^ EB 93         jmp short 00372FD0//循环初始化 IAT
0037303D    8B85 BC020000 mov eax,dword ptr ss:[ebp+2BC]
00373043    83F8 01       cmp eax,1
00373046    75 27         jnz short 0037306F
00373048    8BBD C4020000 mov edi,dword ptr ss:[ebp+2C4]
0037304E    03FD          add edi,ebp
00373050    8DB5 4D020000 lea esi,dword ptr ss:[ebp+24D]
00373056    8B07          mov eax,dword ptr ds:[edi]
00373058    0BC0          or eax,eax
0037305A    75 02         jnz short 0037305E
0037305C    EB 11         jmp short 0037306F
0037305E    25 FFFFFF7F   and eax,7FFFFFFF
00373063    8BDE          mov ebx,esi///////这里开始打补丁，改为 JMP 00373185
00373065    2BD8          sub ebx,eax
00373067    8958 FC       mov dword ptr ds:[eax-4],ebx
0037306A    83C7 08       add edi,8
0037306D  ^ EB E7         jmp short 00373056
0037306F    64:FF35 3000000> push dword ptr fs:[30]
00373076    58            pop eax
00373077    85C0          test eax,eax
00373079    78 0F         js short 0037308A
0037307B    8B40 0C       mov eax,dword ptr ds:[eax+C]
0037307E    8B40 0C       mov eax,dword ptr ds:[eax+C]
00373081    C740 20 0010000> mov dword ptr ds:[eax+20],1000
00373088    EB 1C         jmp short 003730A6
0037308A    6A 00         push 0
0037308C    FF95 A8020000 call dword ptr ss:[ebp+2A8]
00373092    85D2          test edx,edx
00373094    79 10         jns short 003730A6
00373096    837A 08 FF    cmp dword ptr ds:[edx+8],-1
0037309A    75 0A         jnz short 003730A6
0037309C    8B52 04       mov edx,dword ptr ds:[edx+4]
0037309F    C742 50 0010000> mov dword ptr ds:[edx+50],1000
003730A6    89AD 58020000 mov dword ptr ss:[ebp+258],ebp
003730AC    8B85 C8020000 mov eax,dword ptr ss:[ebp+2C8]
003730B2    0385 B4020000 add eax,dword ptr ss:[ebp+2B4]
003730B8    FFE0          jmp eax //这里跳往 OEP          ; NE_St.00665AB4
```

到 `OEP` 后跟进任何一个 `CALL`，可以看到跳转表被加密了，不过解码再简单不过了

随便在此内存段末尾找片空白片，我找的是 `00373185` 这里，所以在 `00341EAA` 代码改成了 `JMP 00373185)`

```
00373185    8BF0          mov esi,eax                         ; NE_St.004014BA
00373187    83C6 FA       add esi,-6
0037318A    66:C706 FF25  mov word ptr ds:[esi],25FF
0037318F    8B47 04       mov eax,dword ptr ds:[edi+4]
00373192    8946 02       mov dword ptr ds:[esi+2],eax
00373195  ^ E9 D0FEFFFF   jmp 0037306A
```

补丁完后 `F4` 到 `003730B8` 这行就到 `OEP`，`DUMP`、修复之后就可以运行了。

用 `Peid` 检测为 `Borland Delphi 6.0 - 7.0`

### 2. 解除自校验

`BP CreateFileA`,`F9`,请看堆栈：

开始检测调试器了

```
0012FDB0   0065C1F3   /CALL 到 CreateFileA 来自 1_.0065C1EE
0012FDB4   0065C204   |FileName = "\\\\.\\SICE"
0012FDB8   C0000000   |Access = GENERIC_READ|GENERIC_WRITE
0012FDBC   00000003   |ShareMode = FILE_SHARE_READ|FILE_SHARE_WRITE
0012FDC0   00000000   |pSecurity = NULL
0012FDC4   00000003   |Mode = OPEN_EXISTING

0012FDB0   0065C3CB   /CALL 到 CreateFileA 来自 1_.0065C3C6
0012FDB4   0065C3DC   |FileName = "\\\\.\\TRWDEBUG"
0012FDB8   C0000000   |Access = GENERIC_READ|GENERIC_WRITE
0012FDBC   00000003   |ShareMode = FILE_SHARE_READ|FILE_SHARE_WRITE
0012FDC0   00000000   |pSecurity = NULL
0012FDC4   00000003   |Mode = OPEN_EXISTING

0012FDB0   0065C40B   /CALL 到 CreateFileA 来自 1_.0065C406
0012FDB4   0065C41C   |FileName = "\\\\.\\TRW2000"
0012FDB8   C0000000   |Access = GENERIC_READ|GENERIC_WRITE
0012FDBC   00000003   |ShareMode = FILE_SHARE_READ|FILE_SHARE_WRITE
0012FDC0   00000000   |pSecurity = NULL
0012FDC4   00000003   |Mode = OPEN_EXISTING
```

没了，看来只检测 2 个最出名的：）

```
0011FC30   00403384   /CALL 到 CreateFileA 来自 1_.0040337F
0011FC34   0012FCB4   |FileName = "C:\\Flyhua\\网络工程师\\1_.exe"
0011FC38   80000000   |Access = GENERIC_READ
0011FC3C   00000003   |ShareMode = FILE_SHARE_READ|FILE_SHARE_WRITE
0011FC40   00000000   |pSecurity = NULL
```

返回

```
0065F60A    33D2          xor edx,edx
0065F60C    52            push edx
0065F60D    50            push eax
0065F60E    8D55 F0       lea edx,dword ptr ss:[ebp-10]
0065F611    B8 08000000   mov eax,8
0065F616    E8 89A4DAFF   call 1_.00409AA4
0065F61B    8B45 F0       mov eax,dword ptr ss:[ebp-10]       ; 19F76797
0065F61E    8D55 F4       lea edx,dword ptr ss:[ebp-C]
0065F621    E8 86FEFFFF   call 1_.0065F4AC
0065F626    8B45 F4       mov eax,dword ptr ss:[ebp-C]        ; 57856
0065F629    50            push eax
0065F62A    8D45 E8       lea eax,dword ptr ss:[ebp-18]
0065F62D    E8 F2FeFFFF   call 1_.0065F524
0065F632    8B45 E8       mov eax,dword ptr ss:[ebp-18]       ; CA719B6F
0065F635    8D55 EC       lea edx,dword ptr ss:[ebp-14]
0065F638    E8 6FFEFFFF   call 1_.0065F4AC
0065F63D    8B55 EC       mov edx,dword ptr ss:[ebp-14]       ; 61824
0065F640    58            pop eax                             ; 57856
0065F641    E8 5656DAFF   call 1_.00404C9C                    
0065F646    74 14         je short 1_.0065F65C                //改为 JMP 0065F65C
```

这里并非是自校验比较的关键地，而是注册成功与否的关键比较地，试想一下如果改为注册成功就能使软件运行，何乐而不为呢，

我才不管你是不是还在自身比较呢：）

### 3. 寻找算法

输入注册信息：

申请码：`NEBS-4088-C747-9238-8222`

认证码：`11111111-22222222`  
        `33333333-44444444`

提示重启验证注册码。注册信息保存在目录下`\DATA\friend.ini`中。

```
-------------------------------------
[Options]

aa=11111111222222223333333344444444
ab=F26C49BAD73D8B3F1E9BA973B646FFDF

--------------------------------------
```

用 `OLLYDB` 加载运行。搜索提示字符串 `DATA\friend.ini` 的找到两处，分别下断点



```
0065F54D    8B45 FC         mov eax,dword ptr ss:[ebp-4]        ;ASCII "F26C49BAD73D8B3F1E9BA973B646FFDF"
0065F550    8A40 03         mov al,byte ptr ds:[eax+3]          // "C"
0065F553    8845 F4         mov byte ptr ss:[ebp-C],al
0065F556    8B45 FC         mov eax,dword ptr ss:[ebp-4]
0065F559    8A40 07         mov al,byte ptr ds:[eax+7]          //"A"
0065F55C    8845 F5         mov byte ptr ss:[ebp-B],al
0065F55F    8B45 FC         mov eax,dword ptr ss:[ebp-4]
0065F562    8A40 09         mov al,byte ptr ds:[eax+9]           //"7"
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
0065F59F    E8 CC53DAFF     call 1_.00404970
..............
...............
0065F5FC    8D55 F8         lea edx,dword ptr ss:[ebp-8]
0065F5FF    8B45 FC         mov eax,dword ptr ss:[ebp-4]
0065F602    E8 C9DAFFFF     call 1_.0065D0D0
0065F607    8B45 F8         mov eax,dword ptr ss:[ebp-8]
0065F60A    33D2            xor edx,edx
0065F60C    52              push edx
0065F60D    50              push eax
0065F60E    8D55 F0         lea edx,dword ptr ss:[ebp-10]
0065F611    B8 08000000     mov eax,8
0065F616    E8 89A4DAFF     call 1_.00409AA4                     ;通过浮点运算产生数值“19F76797”
```

---------------------------------------------------------------------------------------

```
00409A0D    4E              dec esi
00409A0E    D9F8            fprem
00409A10    DF1C24          fistp word ptr ss:[esp]
00409A13    DCF9            fdiv st(1),st
00409A15    8A0424          mov al,byte ptr ss:[esp]
00409A18    04 30          add al,30
00409A1A    3C 3A          cmp al,3A
00409A1C    72 02          jb short 1_.00409A20
00409A1E    04 07          add al,7
00409A20    8806           mov byte ptr ds:[esi],al
00409A22    D9C1           fld st(1)
00409A24    D8D3           fcom st(3)
00409A26    9B             wait
00409A27    DFE0           fstsw ax
00409A29    9E             sahf
00409A2A   73 E1          jnb short 1_.00409A0D
```

------------------------------------------------------------------------------------------

```
0065F61B    8B45 F0         mov eax,dword ptr ss:[ebp-10]     ; 19F76797
0065F61E    8D55 F4         lea edx,dword ptr ss:[ebp-C]
0065F621    E8 86FEFFFF     call 1_.0065F4AC  //经过下面与 CA719B6F 相同的运算
0065F626    8B45 F4         mov eax,dword ptr ss:[ebp-C]       ; 57856
0065F629    50             push eax
0065F62A    8D45 E8         lea eax,dword ptr ss:[ebp-18]
0065F62D    E8 F2FeFFFF     call 1_.0065F524
0065F632    8B45 E8         mov eax,dword ptr ss:[ebp-18]      ; CA719B6F
0065F635    8D55 EC         lea edx,dword ptr ss:[ebp-14]
0065F638    E8 6FFEFFFF     call 1_.0065F4AC
0065F63D    8B55 EC         mov edx,dword ptr ss:[ebp-14]      ; 61824
0065F640    58             pop eax                             ; 57856
0065F641    E8 5656DAFF    call 1_.00404C9C
0065F646    EB 14          jmp short 1_.0065F65C
```

取每个字符对应的 16 进制数值，进行运算

```
1  9  F  7  6  7  9  7
49*128+57*128+70*128+55*128+54*128+55*128+57*128+55*128
=E200--->转换成十进制值就是 57856

C  A  7  1  9  B  6  F
=67*128+65*128+55*128+49*128+57*128+66*128+54*128+70*128
=F180--->转换成十进制值就是 61824
```

两个进行比较是否相等，相等则注册成功。

申请码的由来：

------------------------------------------------------------------------------------

```
0065EFA1    64:8920         mov dword ptr fs:[eax],esp
0065EFA4    8BC3            mov eax,ebx
0065EFA6    BA 50F06500     mov edx,1_.0065F050               ; ASCII "0123456789ABCDEF"
0065EFAB    E8 2459DAFF     call 1_.004048D4
0065EFB0    8D45 F8         lea eax,dword ptr ss:[ebp-8]
0065EFB3    E8 28DEFFFF     call 1_.0065CDE0
0065EFB8    8B45 F8         mov eax,dword ptr ss:[ebp-8]       ;ASCII "I845G/ICH4-P4B53"//取 CPU 的型号
0065EFBB    BA 6CF06500     mov edx,1_.0065F06C               ; ASCII "FF"
0065EFC0    E8 D75CDAFF     call 1_.00404C9C
0065EFC5    75 0A          jnz short 1_.0065EFD1             ;不知道这是比较什么东东？
..
..
0065EFD9    8D55 F0         lea edx,dword ptr ss:[ebp-10]
0065EFDC    8B45 FC         mov eax,dword ptr ss:[ebp-4]
0065EFDF    E8 28FEFFFF     call 1_.0065EE0C
0065EFE4    8B45 F0         mov eax,dword ptr ss:[ebp-10]    ;ASCII "02dd44e5eb5b891e49f83d7db745c540"
0065EFE7    8D4D F4         lea ecx,dword ptr ss:[ebp-C]
0065EFEA    BA 08000000     mov edx,8
0065EFEF    E8 4825DEFF     call 1_.0044153C
0065EFF4    8B45 F4         mov eax,dword ptr ss:[ebp-C]      ;ASCII"b745c540"
0065EFF7    50             push eax
...
0065F098    8D45 F8         lea eax,dword ptr ss:[ebp-8]
0065F09B    E8 40DDFFFF     call 1_.0065CDE0
0065F0A0    8B45 F8         mov eax,dword ptr ss:[ebp-8]
0065F0A3    BA F0F16500     mov edx,1_.0065F1F0               ; ASCII "FF"
0065F0A8    E8 EF5BDAFF     call 1_.00404C9C
0065F0AD    0F85 89000000  jnz 1_.0065F13C
0065F13C    68 18F26500    push 1_.0065F218                  ; ASCII "NEBS-"
0065F141    8D45 E4         lea eax,dword ptr ss:[ebp-1C]
0065F144    50             push eax
0065F145    B9 04000000    mov ecx,4
0065F14A    BA 01000000    mov edx,1
0065F14F    8B45 FC        mov eax,dword ptr ss:[ebp-4]
0065F152    E8 595CDAFF    call 1_.00404DB0
0065F157    FF75 E4        push dword ptr ss:[ebp-1C]
0065F15A    68 0CF26500    push 1_.0065F20C
0065F15F    8D45 E0        lea eax,dword ptr ss:[ebp-20]
0065F162    50             push eax
0065F163    B9 04000000    mov ecx,4
0065F141    8D45 E4        lea eax,dword ptr ss:[ebp-1C]
0065F144    50             push eax
0065F145    B9 04000000    mov ecx,4
0065F14A    BA 01000000    mov edx,1
0065F14F    8B45 FC        mov eax,dword ptr ss:[ebp-4]       ;ASCII "4088C74792388222"
0065F152    E8 595CDAFF    call 1_.00404DB0
0065F157    FF75 E4        push dword ptr ss:[ebp-1C]         ;ASCII "4088"
0065F15A    68 0CF26500    push 1_.0065F20C
0065F15F    8D45 E0        lea eax,dword ptr ss:[ebp-20]
0065F162    50             push eax
0065F163    B9 04000000    mov ecx,4
0065F168    BA 05000000    mov edx,5
0065F16D    8B45 FC        mov eax,dword ptr ss:[ebp-4]
0065F170    E8 3B5CDAFF    call 1_.00404DB0
0065F175    FF75 E0        push dword ptr ss:[ebp-20]         ;ASCII "C747"
0065F178    68 0CF26500    push 1_.0065F20C
0065F17D    8D45 DC        lea eax,dword ptr ss:[ebp-24]
0065F180    50             push eax
0065F181    B9 04000000    mov ecx,4
0065F186    BA 09000000    mov edx,9
0065F18B    8B45 FC        mov eax,dword ptr ss:[ebp-4]
```



```
0065F18E    E8 1D5CDAFF     call 1_.00404DB0

0065F193    FF75 DC        push dword ptr ss:[ebp-24]        ;ASCII "9238"

0065F196    68 0CF26500    push 1_.0065F20C

0065F19B    8D45 D8        lea eax,dword ptr ss:[ebp-28]

0065F19E    50             push eax

0065F19F    B9 04000000    mov ecx,4

0065F1A4    BA 0D000000    mov edx,0D

0065F1A9    8B45 FC        mov eax,dword ptr ss:[ebp-4]

0065F1AC    E8 FF5BDAFF    call 1_.00404DB0

0065F1B1    FF75 D8        push dword ptr ss:[ebp-28]        ;ASCII "8222"

0065F1B4    8BC3           mov eax,ebx

....

0066007E    8B55 FC        mov edx,dword ptr ss:[ebp-4]       ;ASCII "NEBS-4088-C747-9238-8222"组合成申请码的格式

00660081    8B83 14030000  mov eax,dword ptr ds:[ebx+314]

00660087    E8 481EE1FF    call 1_.00471ED4

0066008C    E8 1BF7FFFF    call 1_.0065F7AC

00660091    8BD0           mov edx,eax

00660093    8B83 FC020000  mov eax,dword ptr ds:[ebx+2FC]

00660099    8B08           mov ecx,dword ptr ds:[eax]

0066009B    FF51 64        call dword ptr ds:[ecx+64]

0066009E    33C0           xor eax,eax

006600A0    5A             pop edx

006600A1    59             pop ecx

006600A2    59             pop ecx
```

算法就是上面的大概呢，太累了，不想找下去了，哪位请继续一下![](img/710b5589904d0d2bf58a67748fca40e1.jpg) ![](img/710b5589904d0d2bf58a67748fca40e1.jpg)

很久没发帖了，心痒痒，所以![](img/710b5589904d0d2bf58a67748fca40e1.jpg)![](img/710b5589904d0d2bf58a67748fca40e1.jpg)

`4dd466d0e1e0cc123afbe504da2462ca`
