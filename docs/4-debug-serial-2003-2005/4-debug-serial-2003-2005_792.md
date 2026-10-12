
# SerialChk

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-357.htm`](https://www.pediy.com/kssd/pediy07/pediy7-357.htm)

**【破文标题】** 某黑客游戏最终关验证器注册算法与验证码生成算法分析

**【软件名称】** `SerialChk.exe`

**【软件介绍】** 这个是“挑战高手 2”在线黑客类游戏最终关 14 关的验证码计算程序，你首先需要通过用户名跟注册码的检验，然后你输入从前面某 5 个关卡收集的验证码碎片计算出最终的验证码，通过后你的姓名等其他信息会出现在网站上。这个游戏不错，至少在现在的在线黑客游戏里面我认为还是有一定的可玩度的。如果有兴趣可以去看看。你也可以当作一个入门级别的 crackme。

原介绍：高手挑战 2 总共 14 关，玩家将从第一关开始挑战。在每一关，玩家需要凭借自己的能力和尽可能多的工具根据当前所给出的线索找出下一关的地址，直到到达最后一关。在上面这个过程中，个别关卡还会给出叫做“验证码碎片”的数字或者字母的组合。玩家应该牢记这些验证码碎片和他们出现的关卡。在最后一关会要求玩家提供所有游戏中的验证码碎片组合成为最终的验证码。如果最终的验证码是正确的话，玩家就可以上榜，成为高手挑战榜的一员。CSKSOFT 高手挑战并非一般的黑客类游戏，它更侧重于计算机技术的方方面面。同时具有较高的趣味性。高手挑战 2 在总体水平上要比挑战 1 难。其中对专业技术有比较多的涉及，所以建议还在学习阶段的朋友先尝试高手挑战 1 ！

**【软件地址】** [`www.csksoft.net/NetCompet2/`](http://www.csksoft.net/NetCompet2/)

**【破文作者】** KiLlL\[DFCG\]

**【破解时间】** 2005-08-14 22:21

**【破解声明】** 仅为技术交流之用！

## 【破解过程】

先拿工具看看，`peid`，没有壳，我喜欢。`vc` 的咚咚，功能很简单但是个头却不小，300 多 `k`。

\=================================================================

### 第一部分，注册算法

随便输入看看，用户名 `KiLlL`，注册码 `123456`，弹出错误对话框“用户名至少是 6 位”，好可以下手了。

`od` 载入，`bpx messageboxa` 之后，输入用户名 `KiLlL[DFCG]`，注册码 `123456`，点确定，程序断下：

```
00402C20   .  64:A1 0000000>mov eax,dword ptr fs:[0]            ;  注册码验证函数
00402C26   .  6A FF         push -1
00402C28   .  68 DB5A4200   push SerialCh.00425ADB
00402C2D   .  50           push eax
00402C2E   .  64:8925 00000>mov dword ptr fs:[0],esp
00402C35   .  81EC 88000000 sub esp,88
00402C3B   .  56           push esi
00402C3C   .  6A 01         push 1
00402C3E   .  8BF1         mov esi,ecx
00402C40   .  E8 A67B0100   call SerialCh.0041A7EB
00402C45   .  8B46 74       mov eax,dword ptr ds:[esi+74]       ;  用户名 KiLlL[DFCG]
00402C48   .  8B40 F4       mov eax,dword ptr ds:[eax-C]        ;  用户名长度 B
00402C4B   .  85C0         test eax,eax
00402C4D   .  75 22         jnz short SerialCh.00402C71          ;  是否输入了用户名
00402C4F   .  50           push eax
00402C50   .  50           push eax
00402C51   .  68 887C4200   push SerialCh.00427C88
00402C56   .  E8 BFE00100   call SerialCh.00420D1A             ;  错误提示
00402C5B   .  5E           pop esi
00402C5C   .  8B8C24 880000>mov ecx,dword ptr ss:[esp+88]
00402C63   .  64:890D 00000>mov dword ptr fs:[0],ecx
00402C6A   .  81C4 94000000 add esp,94
00402C70   .  C3           retn
00402C71   >  83F8 06       cmp eax,6                           ;  验证用户名位数
00402C74   .  7D 24         jge short SerialCh.00402C9A         ;  小于六位就完蛋，否则跳
00402C76   .  6A 00         push 0
00402C78   .  6A 00         push 0
00402C7A   .  68 787C4200   push SerialCh.00427C78
00402C7F   .  E8 96E00100   call SerialCh.00420D1A             ;  用户名错误对话框
00402C84   .  5E           pop esi
00402C85   .  8B8C24 880000>mov ecx,dword ptr ss:[esp+88]
00402C8C   .  64:890D 00000>mov dword ptr fs:[0],ecx
00402C93   .  81C4 94000000 add esp,94
00402C99   .  C3           retn
00402C9A   >  8B4E 78       mov ecx,dword ptr ds:[esi+78]       ;  假码
00402C9D   .  8B41 F4       mov eax,dword ptr ds:[ecx-C]        ;  假码的长度
00402CA0   .  85C0         test eax,eax                        ;  测试假码的长度
00402CA2   .  75 24         jnz short SerialCh.00402CC8         ;  是否输入了注册码
00402CA4   .  6A 00         push 0
00402CA6   .  6A 00         push 0
00402CA8   .  68 687C4200   push SerialCh.00427C68
00402CAD   .  E8 68E00100   call SerialCh.00420D1A             ;  错误提示
00402CB2   .  5E           pop esi
00402CB3   .  8B8C24 880000>mov ecx,dword ptr ss:[esp+88]
00402CBA   .  64:890D 00000>mov dword ptr fs:[0],ecx
00402CC1   .  81C4 94000000 add esp,94
00402CC7   .  C3           retn
00402CC8   >  53           push ebx                           ;  111
00402CC9   .  8D5424 08     lea edx,dword ptr ss:[esp+8]      ;  123456
00402CCD   .  52           push edx                          ; /Arg1
00402CCE   .  8BCE         mov ecx,esi                       ; |
00402CD0   .  E8 BBFCFFFF   call SerialCh.00402990           ; \验证注册码函数
00402CD5   .  8B00         mov eax,dword ptr ds:[eax]
00402CD7   .  8B76 78       mov esi,dword ptr ds:[esi+78]
00402CDA   .  50           push eax                          ;  真码，此处可以做内存注册机
00402CDB   .  56           push esi
00402CDC   .  E8 F3980000   call SerialCh.0040C5D4           ;  注册码验证
00402CE1   .  83C4 08       add esp,8
00402CE4   .  85C0         test eax,eax
00402CE6   .  8B4424 08     mov eax,dword ptr ss:[esp+8]      ;  真码
00402CEA   .  0F95C3        setne bl
00402CED   .  83C0 F0       add eax,-10
00402CF0   .  8D48 0C       lea ecx,dword ptr ds:[eax+C]
00402CF3   .  83CA FF       or edx,FFFFFFFF
00402CF6   .  F0:0FC111     lock xadd dword ptr ds:[ecx],edx
00402CFA   .  4A           dec edx
00402CFB   .  85D2         test edx,edx
00402CFD   .  7F 08         jg short SerialCh.00402D07
00402CFF   .  8B08         mov ecx,dword ptr ds:[eax]
00402D01   .  8B11         mov edx,dword ptr ds:[ecx]
00402D03   .  50           push eax
00402D04   .  FF52 04       call dword ptr ds:[edx+4]
00402D07   >  84DB         test bl,bl
00402D09   .  5B           pop ebx
00402D0A   .  6A 00         push 0
00402D0C   .  74 22         je short SerialCh.00402D30        ;  关键跳转，爆破点，改成 jmp 就可以了
00402D0E   .  6A 00         push 0
00402D10   .  68 547C4200   push SerialCh.00427C54
00402D15   .  E8 00E00100   call SerialCh.00420D1A           ;  验证失败提示
00402D1A   .  5E           pop esi
00402D1B   .  8B8C24 880000>mov ecx,dword ptr ss:[esp+88]
00402D22   .  64:890D 00000>mov dword ptr fs:[0],ecx
00402D29   .  81C4 94000000 add esp,94
00402D2F   .  C3           retn
```

注册算法的关键就是 `00402CD0 call SerialCh.00402990`，让我们来看吧：

```
00402990  /$  6A FF         push -1                           ;  计算注册码
00402992  |.  68 B85A4200   push SerialCh.00425AB8            ;  SE 句柄安装
00402997  |.  64:A1 0000000>mov eax,dword ptr fs:[0]
0040299D  |.  50           push eax
0040299E  |.  64:8925 00000>mov dword ptr fs:[0],esp
004029A5  |.  83EC 10       sub esp,10
004029A8  |.  56           push esi
004029A9  |.  8BF1         mov esi,ecx
004029AB  |.  C74424 0C 000>mov dword ptr ss:[esp+C],0
004029B3  |.  E8 19700100   call SerialCh.004199D1
004029B8  |.  8B10         mov edx,dword ptr ds:[eax]
004029BA  |.  8BC8         mov ecx,eax
004029BC  |.  FF52 0C       call dword ptr ds:[edx+C]
004029BF  |.  83C0 10       add eax,10
004029C2  |.  894424 0C     mov dword ptr ss:[esp+C],eax
004029C6  |.  C74424 1C 000>mov dword ptr ss:[esp+1C],0
004029CE  |.  E8 FE6F0100   call SerialCh.004199D1
004029D3  |.  8B10         mov edx,dword ptr ds:[eax]
004029D5  |.  8BC8         mov ecx,eax
004029D7  |.  FF52 0C       call dword ptr ds:[edx+C]
004029DA  |.  83C0 10       add eax,10
004029DD  |.  894424 08     mov dword ptr ss:[esp+8],eax
004029E1  |.  8B46 74       mov eax,dword ptr ds:[esi+74]     ;  用户名
004029E4  |.  8378 F4 01    cmp dword ptr ds:[eax-C],1        ;  循环开始
004029E8  |.  C64424 1C 01  mov byte ptr ss:[esp+1C],1
004029ED  |.  7D 0A         jge short SerialCh.004029F9
004029EF  |.  68 57000780   push 80070057
004029F4  |.  E8 27E7FFFF   call SerialCh.00401120
004029F9  |>  8A40 01       mov al,byte ptr ds:[eax+1]        ;  验证 eax+1 就是第二位 i
004029FC  |.  884424 05     mov byte ptr ss:[esp+5],al       ;  i,ascii 69，放入 esP+5
00402A00  |.  8B46 74       mov eax,dword ptr ds:[esi+74]
00402A03  |.  8B48 F4       mov ecx,dword ptr ds:[eax-C]      ;  用户名长度，B
00402A06  |.  83F9 02       cmp ecx,2                         ;  是不是大于 2 位？
00402A09  |.  7D 0A         jge short SerialCh.00402A15
00402A0B  |.  68 57000780   push 80070057
00402A10  |.  E8 0BE7FFFF   call SerialCh.00401120
```



```
00402A15   |>  8A50 02       mov dl,byte ptr ds:\[eax+2\]         ;  第三位 L，放入 edx  

00402A18   |.  8B46 74       mov eax,dword ptr ds:\[esi+74\]  

00402A1B   |.  8B4E 74       mov ecx,dword ptr ds:\[esi+74\]  

00402A1E   |.  8B49 F4       mov ecx,dword ptr ds:\[ecx\-C\]      ;  用户名长度  

00402A21   |.  53            push ebx  

00402A22   |.  8B58 F4       mov ebx,dword ptr ds:\[eax\-C\]  

00402A25   |.  85DB          test ebx,ebx                     ;  用户名长度 B  

00402A27   |.  7D 0A         jge short SerialCh.00402A33  

00402A29   |.  68 57000780   push 80070057  

00402A2E   |.  E8 EDE6FFFF   call SerialCh.00401120  

00402A33   |>  8A18          mov bl,byte ptr ds:\[eax\]         ;  用户名第一位  

00402A35   |.  0FBEC2        movsx eax,dl                     ;  刚才的 L，4c  

00402A38   |.  0FBE5424 09   movsx edx,byte ptr ss:\[esp+9\]     ;  为什么不去 esp+5，而是+9？压入了 ebp  

00402A3D   |.  03C2          add eax,edx                     ;  累加 ascii 69+4c i+L  

00402A3F   |.  99            cdq                            ;  B5  

00402A40   |.  55            push ebp  

00402A41   |.  BD 09000000   mov ebp,9  

00402A46   |.  F7FD          idiv ebp                         ;  B5/9=14  

00402A48   |.  0FBEC3        movsx eax,bl                     ;  K,4B，此时 edx 存放的是余数 1  

00402A4B   |.  0FAFC1        imul eax,ecx                     ;  4B\*B=339  

00402A4E   |.  8BDD          mov ebx,ebp  

00402A50   |.  52            push edx                         ;  压入余数 1  

00402A51   |.  99            cdq  

00402A52   |.  F7FB          idiv ebx                         ;  339/9=5B，余数是 6  

00402A54   |.  8BC1          mov eax,ecx  

00402A56   |.  8BCD          mov ecx,ebp  

00402A58   |.  52            push edx                         ;  压入 余数 6  

00402A59   |.  99            cdq  

00402A5A   |.  F7F9          idiv ecx                         ;  B/9=1 余数是 2  

00402A5C   |.  52            push edx                         ;  压入结果 2  

00402A5D   |.  8D5424 20     lea edx,dword ptr ss:\[esp+20\]  

00402A61   |.  68 4C7C4200   push SerialCh.00427C4C           ;  ASCII "%d%d%d"  

00402A66   |.  52            push edx                         ;  变成十进制 261  

00402A67   |.  E8 04F1FFFF   call SerialCh.00401B70  

00402A6C   |.  8B46 74       mov eax,dword ptr ds:\[esi+74\]    ;  序列号第一部分 261  

00402A6F   |.  8B48 F4       mov ecx,dword ptr ds:\[eax\-C\]  

00402A72   |.  83C4 14       add esp,14  

注册码第一部分，长度 MOD 9,第一位\*len mod 9,( 第二位+第三位 ) mod 9  

00402A75   |.  83F9 05       cmp ecx,5                        ;  b  

00402A78   |.  5D            pop ebp  

00402A79   |.  7D 0A         jge short SerialCh.00402A85  

00402A7B   |.  68 57000780   push 80070057  

00402A80   |.  E8 9BE6FFFF   call SerialCh.00401120  

00402A85   |>  8B4E 74       mov ecx,dword ptr ds:\[esi+74\]  

00402A88   |.  8379 F4 04    cmp dword ptr ds:\[ecx\-C\],4     ;  4  

00402A8C   |.  8A40 05       mov al,byte ptr ds:\[eax+5\]     ;  第六位\[，放入 al  

00402A8F   |.  7D 0A         jge short SerialCh.00402A9B  

00402A91   |.  68 57000780   push 80070057  

00402A96   |.  E8 85E6FFFF   call SerialCh.00401120  

00402A9B   |>  8A49 04       mov cl,byte ptr ds:\[ecx+4\]     ;  第五位 L  

00402A9E   |.  884C24 09     mov byte ptr ss:\[esp+9\],cl     ;  放入 esp+9  

00402AA2   |.  8B4E 74       mov ecx,dword ptr ds:\[esi+74\]  

00402AA5   |.  8379 F4 03    cmp dword ptr ds:\[ecx\-C\],3  

00402AA9   |.  7D 0A         jge short SerialCh.00402AB5  

00402AAB   |.  68 57000780   push 80070057  

00402AB0   |.  E8 6BE6FFFF   call SerialCh.00401120  

00402AB5   |>  8A51 03       mov dl,byte ptr ds:\[ecx+3\]     ;  第四位 l 6c  

00402AB8   |.  8B4E 74       mov ecx,dword ptr ds:\[esi+74\]  

00402ABB   |.  885424 0A     mov byte ptr ss:\[esp+A\],dl  

00402ABF   |.  8379 F4 02    cmp dword ptr ds:\[ecx\-C\],2  

00402AC3   |.  7D 0A         jge short SerialCh.00402ACF  

00402AC5   |.  68 57000780   push 80070057  

00402ACA   |.  E8 51E6FFFF   call SerialCh.00401120  

00402ACF   |>  8A49 02       mov cl,byte ptr ds:\[ecx+2\]     ;  第三位 L 4c  

00402AD2   |.  884C24 0B     mov byte ptr ss:\[esp+B\],cl  

00402AD6   |.  8B4E 74       mov ecx,dword ptr ds:\[esi+74\]  

00402AD9   |.  8379 F4 01    cmp dword ptr ds:\[ecx\-C\],1  

00402ADD   |.  7D 0A         jge short SerialCh.00402AE9  

00402ADF   |.  68 57000780   push 80070057  

00402AE4   |.  E8 37E6FFFF   call SerialCh.00401120  

00402AE9   |>  8B76 74       mov esi,dword ptr ds:\[esi+74\]  

00402AEC   |.  8A59 01       mov bl,byte ptr ds:\[ecx+1\]     ;  第二位 i  

00402AEF   |.  8B4E F4       mov ecx,dword ptr ds:\[esi\-C\]  

00402AF2   |.  85C9          test ecx,ecx  

00402AF4   |.  7D 0A         jge short SerialCh.00402B00  

00402AF6   |.  68 57000780   push 80070057  

00402AFB   |.  E8 20E6FFFF   call SerialCh.00401120  

00402B00   |>  8A0E          mov cl,byte ptr ds:\[esi\]       ;  用户名第一位  

00402B02   |.  0FBEC0        movsx eax,al                     ;  取出 al 的\[,5B  

00402B05   |.  C1E0 03       shl eax,3                        ;  5B\*2³=2D8  

00402B08   |.  99            cdq  

00402B09   |.  BE 4D000000   mov esi,4D  

00402B0E   |.  F7FE          idiv esi                         ;  2D8/4D=9，余数 23  

00402B10   |.  0FBE4424 09   movsx eax,byte ptr ss:\[esp+9\]     ;  取出 esp+9，L,4C  

00402B15   |.  8D0440        lea eax,dword ptr ds:\[eax+eax\*2\]    ;  4C+4C\*2=E4  

00402B18   |.  D1E0          shl eax,1                        ;  E4\*2=1C8  

00402B1A   |.  BE 43000000   mov esi,43  

00402B1F   |.  52            push edx                         ;  压入第一个余数 23  

00402B20   |.  99            cdq  

00402B21   |.  F7FE          idiv esi                         ;  1C8/43=6 余数 36  

00402B23   |.  0FBE4424 0E   movsx eax,byte ptr ss:\[esp+E\]     ;  第四位,l,6c  

00402B28   |.  8D0440        lea eax,dword ptr ds:\[eax+eax\*2\]    ;  6c+6c\*2=144  

00402B2B   |.  BE 37000000   mov esi,37  

00402B30   |.  52            push edx                         ;  压入第二个余数 36  

00402B31   |.  99            cdq  

00402B32   |.  F7FE          idiv esi                         ;  144/37=5 余数 31  

00402B34   |.  0FBE4424 13   movsx eax,byte ptr ss:\[esp+13\]     ;  第三位 L,4C  

00402B39   |.  BE 2C000000   mov esi,2C  

00402B3E   |.  52            push edx                         ;  压入第三个余数 31  

00402B3F   |.  99            cdq  

00402B40   |.  F7FE          idiv esi                         ;  4C/2c=1 余数 20  

00402B42   |.  0FBEC3        movsx eax,bl                     ;  第二位，i，69  

00402B45   |.  C1E0 02       shl eax,2                        ;  69\*2²＝1A4  

00402B48   |.  BE 21000000   mov esi,21                       ;  1a4  

00402B4D   |.  52            push edx                         ;  压入第四个余数 20  

00402B4E   |.  99            cdq  

00402B4F   |.  F7FE          idiv esi                         ;  1A4/21=C，余数 18  

00402B51   |.  0FBEC1        movsx eax,cl                     ;  第一位 K,4B  

00402B54   |.  8D0480        lea eax,dword ptr ds:\[eax+eax\*4\]    ;  4B+4B\*2=177  

00402B57   |.  B9 14000000   mov ecx,14  

00402B5C   |.  52            push edx                         ;  压入第五个 18  

00402B5D   |.  99            cdq  

00402B5E   |.  F7F9          idiv ecx                         ;  177/14=12 余数是 F  

00402B60   |.  52            push edx                         ;  压入最后一个余数 F  

00402B61   |.  8D5424 24     lea edx,dword ptr ss:\[esp+24\]  

00402B65   |.  68 3C7C4200   push SerialCh.00427C3C           ;  ASCII "%d%d%d%d%d%d"  

00402B6A   |.  52            push edx  

00402B6B   |.  E8 00F0FFFF   call SerialCh.00401B70           ;  转成十进制  

00402B70   |.  68 387C4200   push SerialCh.00427C38           ;  152432495435  

00402B75   |.  8D4424 34     lea eax,dword ptr ss:\[esp+34\]  

00402B79   |.  50            push eax  

00402B7A   |.  8D4C24 3C     lea ecx,dword ptr ss:\[esp+3C\]  

00402B7E   |.  51            push ecx  
```



## 汇编代码段

```
00402B7F  |.  E8 4CFDFFFF   call SerialCh.004028D0  
00402B84  |.  8B7424 54     mov esi,dword ptr ss:[esp+54]  
00402B88  |.  8D5424 38     lea edx,dword ptr ss:[esp+38]  
00402B8C  |.  52            push edx  
00402B8D  |.  50            push eax  
00402B8E  |.  56            push esi  
00402B8F  |.  C64424 58 02  mov byte ptr ss:[esp+58],2  
00402B94  |.  E8 F7EFFFFF   call SerialCh.00401B90          ; 261 变成了 261-  
00402B99  |.  8B4424 4C     mov eax,dword ptr ss:[esp+4C]  
00402B9D  |.  83C0 F0       add eax,-10  
00402BA0  |.  83C4 38       add esp,38  
00402BA3  |.  C64424 20 01  mov byte ptr ss:[esp+20],1  
00402BA8  |.  8D48 0C       lea ecx,dword ptr ds:[eax+C]  
00402BAB  |.  83CA FF       or edx,FFFFFFFF  
00402BAE  |.  F0:0FC111     lock xadd dword ptr ds:[ecx],edx  
00402BB2  |.  4A            dec edx  
00402BB3  |.  85D2          test edx,edx  
00402BB5  |.  7F 08         jg short SerialCh.00402BBF  
00402BB7  |.  8B08          mov ecx,dword ptr ds:[eax]  
00402BB9  |.  8B11          mov edx,dword ptr ds:[ecx]  
00402BBB  |.  50            push eax  
00402BBC  |.  FF52 04       call dword ptr ds:[edx+4]  
00402BBF  |> 8B4424 0C     mov eax,dword ptr ss:[esp+C]      ; 注册码第二部分 152432495435  
00402BC3  |.  83C0 F0       add eax,-10  
00402BC6  |.  C64424 20 00  mov byte ptr ss:[esp+20],0  
00402BCB  |.  8D48 0C       lea ecx,dword ptr ds:[eax+C]  
00402BCE  |.  83CA FF       or edx,FFFFFFFF  
00402BD1  |.  F0:0FC111     lock xadd dword ptr ds:[ecx],edx  
00402BD5  |.  4A            dec edx  
00402BD6  |.  85D2          test edx,edx  
00402BD8  |.  7F 08         jg short SerialCh.00402BE2  
00402BDA  |.  8B08          mov ecx,dword ptr ds:[eax]  
00402BDC  |.  8B11          mov edx,dword ptr ds:[ecx]  
00402BDE  |.  50            push eax  
00402BDF  |.  FF52 04       call dword ptr ds:[edx+4]  
00402BE2  |> 8B4424 10     mov eax,dword ptr ss:[esp+10]  
00402BE6  |.  83C0 F0       add eax,-10  
00402BE9  |.  C74424 20 FFF>mov dword ptr ss:[esp+20],-1  
00402BF1  |.  8D48 0C       lea ecx,dword ptr ds:[eax+C]  
00402BF4  |.  83CA FF       or edx,FFFFFFFF  
00402BF7  |.  F0:0FC111     lock xadd dword ptr ds:[ecx],edx  
00402BFB  |.  4A            dec edx  
00402BFC  |.  85D2          test edx,edx  
00402BFE  |.  7F 08         jg short SerialCh.00402C08  
00402C00  |.  8B08          mov ecx,dword ptr ds:[eax]  
00402C02  |.  8B11          mov edx,dword ptr ds:[ecx]  
00402C04  |.  50            push eax  
00402C05  |.  FF52 04       call dword ptr ds:[edx+4]  
00402C08  |> 8B4C24 18     mov ecx,dword ptr ss:[esp+18]  
00402C0C  |.  5B            pop ebx  
00402C0D  |.  8BC6          mov eax,esi  
00402C0F  |.  5E            pop esi  
00402C10  |.  64:890D 00000>mov dword ptr fs:[0],ecx  
00402C17  |.  83C4 1C       add esp,1C  
00402C1A  \.  C2 0400       retn 4  
```

### 注册码算法

注册码第二部分计算规则：

- 第一位：`*5 mod 14`
- 第二位：`*4 mod 21`
- 第三位：`mod 2c`
- 第四位：`*3 mod 37`
- 第五位：`*6 mod 43`
- 第六位：`*8 mod 4d`

### 结果示例

- 用户名：`KiLlL[DFCG]`
- 注册码：`261-152432495435`

---

## 第二部分：计算验证码

### 背景

既然注册过了，再顺便分析一下它的验证码组合过程吧。

五个验证码碎片分别输入 `1`，`2`，`3`，`4`，`5`，弹出组合后的验证码。但是怎么断下呢？

使用 `od` 的字符串插件：

### OllyDbg 字符串参考搜索

| 地址 | 反汇编 | 字符串 |
|------|--------|--------|
| `0040216F` | `push SerialCh.00427768` | `请输入第 1 关的验证码碎片` |
| `00402198` | `push SerialCh.00427750` | `请输入第 5 关的验证码碎片` |
| `004021C1` | `push SerialCh.00427738` | `请输入第 9 关的验证码碎片` |
| `004021EA` | `push SerialCh.0042771C` | `请输入第 11 关的验证码碎片` |
| `00402216` | `push SerialCh.00427700` | `请输入第 12 关的验证码碎片` |
| `00402243` | `push SerialCh.004276EC` | `CSKSOFT-Personal` |

发现了很多东西呀。好，就导第一个地方去看看了。

### 验证码生成部分汇编

```
0040213C      CC            int3  
0040213D      CC            int3  
0040213E      CC            int3  
0040213F      CC            int3  
00402140  .  64:A1 0000000>mov eax,dword ptr fs:[0]            ; 验证码生成部分  
00402146  .  6A FF          push -1  
00402148  .  68 E3594200    push SerialCh.004259E3  
0040214D  .  50            push eax  
0040214E  .  64:8925 00000>mov dword ptr fs:[0],esp  
00402155  .  83EC 78        sub esp,78  
00402158  .  56            push esi  
00402159  .  6A 01          push 1  
0040215B  .  8BF1           mov esi,ecx  
0040215D  .  E8 89860100    call SerialCh.0041A7EB  
00402162  .  8B46 70        mov eax,dword ptr ds:[esi+70]      ; 第一文本框的验证码  
00402165  .  8B40 F4        mov eax,dword ptr ds:[eax-C]  
00402168  .  85C0           test eax,eax                        ; 长度比较  
0040216A  .  6A 00          push 0  
0040216C  .  75 1E          jnz short SerialCh.0040218C         ; 不为空就跳  
0040216E  .  50            push eax  
0040216F  .  68 68774200    push SerialCh.00427768  
00402174  .  E8 A1EB0100    call SerialCh.00420D1A  
00402179  .  8B4C24 7C      mov ecx,dword ptr ss:[esp+7C]  
0040217D  .  64:890D 00000>mov dword ptr fs:[0],ecx  
00402184  .  5E            pop esi  
00402185  .  81C4 84000000  add esp,84  
0040218B  .  C3            retn  
0040218C  >  8B4E 74        mov ecx,dword ptr ds:[esi+74]  
0040218F  .  8B41 F4        mov eax,dword ptr ds:[ecx-C]  
00402192  .  85C0           test eax,eax  
00402194  .  75 1F          jnz short SerialCh.004021B5         ; 第二个  
00402196  .  6A 00          push 0  
00402198  .  68 50774200    push SerialCh.00427750  
0040219D  .  E8 78EB0100    call SerialCh.00420D1A  
004021A2  .  8B4C24 7C      mov ecx,dword ptr ss:[esp+7C]  
004021A6  .  64:890D 00000>mov dword ptr fs:[0],ecx  
004021AD  .  5E            pop esi  
004021AE  .  81C4 84000000  add esp,84  
004021B4  .  C3            retn  
004021B5  >  8B56 78        mov edx,dword ptr ds:[esi+78]      ; 第三个  
004021B8  .  8B42 F4        mov eax,dword ptr ds:[edx-C]  
004021BB  .  85C0           test eax,eax  
004021BD  .  75 1F          jnz short SerialCh.004021DE  
004021BF  .  6A 00          push 0  
004021C1  .  68 38774200    push SerialCh.00427738  
004021C6  .  E8 4FEB0100    call SerialCh.00420D1A  
004021CB  .  8B4C24 7C      mov ecx,dword ptr ss:[esp+7C]  
004021CF  .  64:890D 00000>mov dword ptr fs:[0],ecx  
004021D6  .  5E            pop esi  
004021D7  .  81C4 84000000  add esp,84  
004021DD  .  C3            retn  
004021DE  >  8B46 7C        mov eax,dword ptr ds:[esi+7C]      ; 第四个  
004021E1  .  8B48 F4        mov ecx,dword ptr ds:[eax-C]  
004021E4  .  85C9           test ecx,ecx  
004021E6  .  75 1F          jnz short SerialCh.00402207  
004021E8  .  6A 00          push 0  
004021EA  .  68 1C774200    push SerialCh.0042771C  
004021EF  .  E8 26EB0100    call SerialCh.00420D1A  
004021F4  .  8B4C24 7C      mov ecx,dword ptr ss:[esp+7C]  
004021F8  .  64:890D 00000>mov dword ptr fs:[0],ecx  
004021FF  .  5E            pop esi  
00402200  .  81C4 84000000  add esp,84  
00402206  .  C3            retn  
00402207  >  8B8E 80000000  mov ecx,dword ptr ds:[esi+80]      ; 第五个  
0040220D  .  8B41 F4        mov eax,dword ptr ds:[ecx-C]  
00402210  .  85C0           test eax,eax  
00402212  .  75 1F          jnz short SerialCh.00402233  
00402214  .  6A 00          push 0  
00402216  .  68 00774200    push SerialCh.00427700  
0040221B  .  E8 FAEA0100    call SerialCh.00420D1A  
00402220  .  8B4C24 7C      mov ecx,dword ptr ss:[esp+7C]  
00402224  .  64:890D 00000>mov dword ptr fs:[0],ecx  
0040222B  .  5E            pop esi  
0040222C  .  81C4 84000000  add esp,84  
00402232  .  C3            retn  
00402233  >  8D4C24 0C      lea ecx,dword ptr ss:[esp+C]       ; 当你输入了全部的验证码后  
00402237  .  E8 24010000    call SerialCh.00402360  
0040223C  .  51            push ecx  
0040223D  .  8BCC           mov ecx,esp  
0040223F  .  896424 08      mov dword ptr ss:[esp+8],esp  
00402243  .  68 EC764200    push SerialCh.004276EC             ; 固定字串"CSKSOFT-Personal"  
```



### 00402248 - 004022B6 反汇编代码

```
00402248   .   C78424 8C0000>mov dword ptr ss:[esp+8C],0  
00402253   .   E8 E8F9FFFF   call SerialCh.00401C40  
00402258   .   8D5424 08     lea edx,dword ptr ss:[esp+8]        ; |  
0040225C   .   52            push edx                            ; |Arg1 压入固定字串跟你输入的字串。  
0040225D   .   8BCE          mov ecx,esi                         ; |  
0040225F   .   E8 3CFBFFFF   call SerialCh.00401DA0              ; \\验证码生成函数  
00402264   .   50            push eax  
00402265   .   8D4C24 7C     lea ecx,dword ptr ss:[esp+7C]  
00402269   .   C68424 88000>mov byte ptr ss:[esp+88],1  
00402271   .   E8 4AF8FFFF   call SerialCh.00401AC0  
00402276   .   8D4C24 04     lea ecx,dword ptr ss:[esp+4]  
0040227A   .   C68424 84000>mov byte ptr ss:[esp+84],0  
00402282   .   E8 99F1FFFF   call SerialCh.00401420  
00402287   .   8D4C24 08     lea ecx,dword ptr ss:[esp+8]  
0040228B   .   E8 0C740100   call SerialCh.0041969C  
00402290   .   8D4C24 08     lea ecx,dword ptr ss:[esp+8]  
00402294   .   C78424 84000>mov dword ptr ss:[esp+84],-1  
0040229F   .   E8 6C000000   call SerialCh.00402310  
004022A4   .   8B4C24 7C     mov ecx,dword ptr ss:[esp+7C]  
004022A8   .   64:890D 00000>mov dword ptr fs:[0],ecx  
004022AF   .   5E            pop esi  
004022B0   .   81C4 84000000 add esp,84  
004022B6   .   C3            retn  
```

看来这个 `0040225F call SerialCh.00401DA0` 很关键，那就去看看了。

### 00401DA0 - 00401F05 反汇编代码

```
00401DA0  /$  6A FF         push -1  
00401DA2  |.  68 B8594200   push SerialCh.004259B8              ;  核心代码  
00401DA7  |.  64:A1 0000000>mov eax,dword ptr fs:[0]  
00401DAD  |.  50            push eax  
00401DAE  |.  64:8925 00000>mov dword ptr fs:[0],esp  
00401DB5  |.  83EC 18       sub esp,18  
00401DB8  |.  53            push ebx  
00401DB9  |.  55            push ebp  
00401DBA  |.  56            push esi  
00401DBB  |.  33C0          xor eax,eax  
00401DBD  |.  57            push edi  
00401DBE  |.  8BF1          mov esi,ecx  
00401DC0  |.  894424 24     mov dword ptr ss:[esp+24],eax  
00401DC4  |.  894424 30     mov dword ptr ss:[esp+30],eax  
00401DC8  |.  E8 047C0100   call SerialCh.004199D1  
00401DCF  |.  8B10          mov edx,dword ptr ds:[eax]  
00401DCF  |.  8BC8          mov ecx,eax  
00401DD1  |.  FF52 0C       call dword ptr ds:[edx+C]  
00401DD4  |.  8D68 10       lea ebp,dword ptr ds:[eax+10]  
00401DD7  |.  896C24 18     mov dword ptr ss:[esp+18],ebp  
00401DDB  |.  C64424 30 01  mov byte ptr ss:[esp+30],1  
00401DE0  |.  E8 EC7B0100   call SerialCh.004199D1  
00401DE5  |.  8B10          mov edx,dword ptr ds:[eax]  
00401DE7  |.  8BC8          mov ecx,eax  
00401DE9  |.  FF52 0C       call dword ptr ds:[edx+C]  
00401DEC  |.  8D78 10       lea edi,dword ptr ds:[eax+10]  
00401DEF  |.  897C24 10     mov dword ptr ss:[esp+10],edi  
00401DF3  |.  8B5C24 3C     mov ebx,dword ptr ss:[esp+3C]  
00401DF7  |.  68 E0764200   push SerialCh.004276E0  
00401DFC  |.  53            push ebx  
00401DFD  |.  C64424 38 02  mov byte ptr ss:[esp+38],2  
00401E02  |.  E8 CDA70000   call SerialCh.0040C5D4  
00401E07  |.  83C4 08       add esp,8  
00401E0A  |.  85C0          test eax,eax  
00401E0C  |.  75 3C         jnz short SerialCh.00401E4A  
00401E0E  |.  E8 BE7B0100   call SerialCh.004199D1  
00401E13  |.  8B10          mov edx,dword ptr ds:[eax]  
00401E15  |.  8BC8          mov ecx,eax  
00401E17  |.  FF52 0C       call dword ptr ds:[edx+C]  
00401E1A  |.  8B7424 38     mov esi,dword ptr ss:[esp+38]  
00401E1E  |.  83C0 10       add eax,10  
00401E21  |.  8906          mov dword ptr ds:[esi],eax  
00401E23  |.  8D47 F0       lea eax,dword ptr ds:[edi-10]  
00401E26  |.  C64424 30 01  mov byte ptr ss:[esp+30],1  
00401E2B  |.  8D48 0C       lea ecx,dword ptr ds:[eax+C]  
00401E2E  |.  83CA FF       or edx,FFFFFFFF  
00401E31  |.  F0:0FC111     lock xadd dword ptr ds:[ecx],edx  
00401E35  |.  4A            dec edx  
00401E36  |.  85D2          test edx,edx  
00401E38  |.  7F 08         jg short SerialCh.00401E42  
00401E3A  |.  8B08          mov ecx,dword ptr ds:[eax]  
00401E3C  |.  8B11          mov edx,dword ptr ds:[ecx]  
00401E3E  |.  50            push eax  
00401E3F  |.  FF52 04       call dword ptr ds:[edx+4]  
00401E42  |>  8D45 F0       lea eax,dword ptr ss:[ebp-10]  
00401E45  |.  E9 94020000   jmp SerialCh.004020DE  
00401E4A  |>  8D46 74       lea eax,dword ptr ds:[esi+74]  
00401E4D  |.  50            push eax  
00401E4E  |.  8D4E 70       lea ecx,dword ptr ds:[esi+70]  
00401E51  |.  51            push ecx  
00401E52  |.  8D5424 2C     lea edx,dword ptr ss:[esp+2C]  
00401E56  |.  52            push edx                            ;  下面把所有的字符串连接起来  
00401E57  |.  E8 34FDFFFF   call SerialCh.00401B90  
00401E5C  |.  8D4E 78       lea ecx,dword ptr ds:[esi+78]  
00401E5F  |.  51            push ecx  
00401E60  |.  50            push eax  
00401E61  |.  8D5424 34     lea edx,dword ptr ss:[esp+34]  
00401E65  |.  52            push edx  
00401E66  |.  C64424 48 03  mov byte ptr ss:[esp+48],3  
00401E6B  |.  E8 20FDFFFF   call SerialCh.00401B90  
00401E70  |.  8D4E 7C       lea ecx,dword ptr ds:[esi+7C]  
00401E73  |.  51            push ecx  
00401E74  |.  50            push eax  
00401E75  |.  8D5424 34     lea edx,dword ptr ss:[esp+34]  
00401E79  |.  52            push edx  
00401E7A  |.  C64424 54 04  mov byte ptr ss:[esp+54],4  
00401E7F  |.  E8 0CFDFFFF   call SerialCh.00401B90  
00401E84  |.  81C6 80000000 add esi,80  
00401E8A  |.  56            push esi  
00401E8B  |.  50            push eax  
00401E8C  |.  8D4424 48     lea eax,dword ptr ss:[esp+48]  
00401E90  |.  B3 05         mov bl,5  
00401E92  |.  50            push eax  
00401E93  |.  885C24 60     mov byte ptr ss:[esp+60],bl  
00401E97  |.  E8 F4FCFFFF   call SerialCh.00401B90              ;  连接起来 12345  
00401E9C  |.  83C4 30       add esp,30  
00401E9F  |.  50            push eax  
00401EA0  |.  8D4C24 1C     lea ecx,dword ptr ss:[esp+1C]  
00401EA4  |.  C64424 34 06  mov byte ptr ss:[esp+34],6  
00401EA9  |.  E8 12FCFFFF   call SerialCh.00401AC0  
00401EAE  |.  8B4424 1C     mov eax,dword ptr ss:[esp+1C]       ;  12345  
00401EB2  |.  83C0 F0       add eax,-10  
00401EB5  |.  885C24 30     mov byte ptr ss:[esp+30],bl         ;  长度 5  
00401EB9  |.  8D48 0C       lea ecx,dword ptr ds:[eax+C]  
00401EBC  |.  83CA FF       or edx,FFFFFFFF  
00401EBF  |.  F0:0FC111     lock xadd dword ptr ds:[ecx],edx  
00401EC3  |.  4A            dec edx  
00401EC4  |.  85D2          test edx,edx  
00401EC6  |.  7F 08         jg short SerialCh.00401ED0  
00401EC8  |.  8B08          mov ecx,dword ptr ds:[eax]  
00401ECA  |.  8B11          mov edx,dword ptr ds:[ecx]  
00401ECC  |.  50            push eax  
00401ECD  |.  FF52 04       call dword ptr ds:[edx+4]  
00401ED0  |>  8B4424 14     mov eax,dword ptr ss:[esp+14]       ;  下面把他们分开存放  
00401ED4  |.  83C0 F0       add eax,-10  
00401ED7  |.  C64424 30 04  mov byte ptr ss:[esp+30],4  
00401EDC  |.  8D48 0C       lea ecx,dword ptr ds:[eax+C]  
00401EDF  |.  83CA FF       or edx,FFFFFFFF  
00401EE2  |.  F0:0FC111     lock xadd dword ptr ds:[ecx],edx  
00401EE6  |.  4A            dec edx  
00401EE7  |.  85D2          test edx,edx  
00401EE9  |.  7F 08         jg short SerialCh.00401EF3  
00401EEB  |.  8B08          mov ecx,dword ptr ds:[eax]  
00401EED  |.  8B11          mov edx,dword ptr ds:[ecx]  
00401EEF  |.  50            push eax  
00401EF0  |.  FF52 04       call dword ptr ds:[edx+4]  
00401EF3  |>  8B4424 20     mov eax,dword ptr ss:[esp+20]  
00401EF7  |.  83C0 F0       add eax,-10  
00401EFA  |.  C64424 30 03  mov byte ptr ss:[esp+30],3  
00401EFF  |.  8D48 0C       lea ecx,dword ptr ds:[eax+C]  
00401F02  |.  83CA FF       or edx,FFFFFFFF  
00401F05  |.  F0:0FC111     lock xadd dword ptr ds:[ecx],edx  
```



```
00401F09  |.  4A            dec  edx  

00401F0A  |.  85D2          test  edx,edx  

00401F0C  |.  7F 08         jg   short  SerialCh.00401F16  

00401F0E  |.  8B08          mov  ecx,dword  ptr  ds:\[eax\]  

00401F10  |.  8B11          mov  edx,dword  ptr  ds:\[ecx\]  

00401F12  |.  50            push  eax  

00401F13  |.  FF52 04       call  dword  ptr  ds:\[edx+4\]  

00401F16  |>  8B4424 24     mov  eax,dword  ptr  ss:\[esp+24\]  

00401F1A  |.  83C0 F0       add  eax,-10  

00401F1D  |.  C64424 30 02  mov  byte  ptr  ss:\[esp+30\],2  

00401F22  |.  8D48 0C       lea  ecx,dword  ptr  ds:\[eax+C\]  

00401F25  |.  83CA FF       or   edx,FFFFFFFF  

00401F28  |.  F0:0FC111     lock  xadd  dword  ptr  ds:\[ecx\],edx  

00401F2C  |.  4A            dec  edx  

00401F2D  |.  85D2          test  edx,edx  

00401F2F  |.  7F 08         jg   short  SerialCh.00401F39  

00401F31  |.  8B08          mov  ecx,dword  ptr  ds:\[eax\]  

00401F33  |.  8B11          mov  edx,dword  ptr  ds:\[ecx\]  

00401F35  |.  50            push  eax  

00401F36  |.  FF52 04       call  dword  ptr  ds:\[edx+4\]  

00401F39  |>  8B5C24 3C     mov  ebx,dword  ptr  ss:\[esp+3C\]       ;  固定堆栈  "CSKSOFT-Personal"  

00401F3D  |.  8B4B F4       mov  ecx,dword  ptr  ds:\[ebx\-C\]  

00401F40  |.  8B6C24 18     mov  ebp,dword  ptr  ss:\[esp+18\]       ;  合成之后的 12345  

00401F44  |.  8B45 F4       mov  eax,dword  ptr  ss:\[ebp\-C\]  

00401F47  |.  33F6          xor  esi,esi  

00401F49  |.  56            push  esi  

00401F4A  |.  894C24 24     mov  dword  ptr  ss:\[esp+24\],ecx  

00401F4E  |.  68 E0764200   push  SerialCh.004276E0  

00401F53  |.  8D4C24 18     lea  ecx,dword  ptr  ss:\[esp+18\]  

00401F57  |.  894424 24     mov  dword  ptr  ss:\[esp+24\],eax  

00401F5B  |.  E8 30F8FFFF   call  SerialCh.00401790  

00401F60  |.  E8 6C7A0100   call  SerialCh.004199D1  

00401F65  |.  8B10          mov  edx,dword  ptr  ds:\[eax\]  

00401F67  |.  8BC8          mov  ecx,eax  

00401F69  |.  FF52 0C       call  dword  ptr  ds:\[edx+C\]  

00401F6C  |.  83C0 10       add  eax,10  

00401F6F  |.  894424 14     mov  dword  ptr  ss:\[esp+14\],eax  

00401F73  |.  8B4424 1C     mov  eax,dword  ptr  ss:\[esp+1C\]  

00401F77  |.  33FF          xor  edi,edi  

00401F79  |.  85C0          test  eax,eax  

00401F7B  |.  C64424 30 07  mov  byte  ptr  ss:\[esp+30\],7          ;  7  

00401F80  |.  0F86 A0000000 jbe  SerialCh.00402026  

00401F86  |>  85FF          /test  edi,edi  

00401F88  |.  0F8C C3000000 |jl  SerialCh.00402051  

00401F8E  |.  3B7D F4       |cmp  edi,dword  ptr  ss:\[ebp\-C\]       ;  长度 5，用作循环  

00401F91  |.  0F8F BA000000 |jg  SerialCh.00402051  

00401F97  |.  85F6          |test  esi,esi  

00401F99  |.  8A0C2F        |mov  cl,byte  ptr  ds:\[edi+ebp\]       ;  逐位读取他们的 ASCII  

00401F9C  |.  0F8C AF000000 |jl  SerialCh.00402051  

00401FA2  |.  3B73 F4       |cmp  esi,dword  ptr  ds:\[ebx\-C\]  

00401FA5  |.  0F8F A6000000 |jg  SerialCh.00402051  

00401FAB  |.  8A041E        |mov  al,byte  ptr  ds:\[esi+ebx\]       ;  由 esi 和循环变量一起控制取"CSKSOFT-Personal"某位  

00401FAE  |.  32C1          |xor  al,cl                            ;  固定串的某位跟验证码碎片的某位 XOR  

00401FB0  |.  0FB6C0        |movzx  eax,al                         ;  得到最终验证码的 ascii  

00401FB3  |.  50            |push  eax  

00401FB4  |.  8D4C24 18     |lea  ecx,dword  ptr  ss:\[esp+18\]  

00401FB8  |.  68 E4764200   |push  SerialCh.004276E4               ;  ASCII  "00%d"  

00401FBD  |.  51            |push  ecx  

00401FBE  |.  E8 ADFBFFFF   |call  SerialCh.00401B70  

00401FC3  |.  83C4 0C       |add  esp,0C                           ;  转成 10 进制，并加入前导 00  

00401FC6  |.  6A 03         |push  3  

00401FC8  |.  8D5424 28     |lea  edx,dword  ptr  ss:\[esp+28\]  

00401FCC  |.  52            |push  edx  

00401FCD  |.  8D4C24 1C     |lea  ecx,dword  ptr  ss:\[esp+1C\]  

00401FD1  |.  E8 7AF8FFFF   |call  SerialCh.00401850               ;  right(sn,3)  

00401FD6  |.  8B00          |mov  eax,dword  ptr  ds:\[eax\]         ;  从右边截取三位  

00401FD8  |.  8B48 F4       |mov  ecx,dword  ptr  ds:\[eax\-C\]  

00401FDB  |.  51            |push  ecx  

00401FDC  |.  50            |push  eax  

00401FDD  |.  8D4C24 18     |lea  ecx,dword  ptr  ss:\[esp+18\]  

00401FE1  |.  C64424 38 08  |mov  byte  ptr  ss:\[esp+38\],8  

00401FE6  |.  E8 D5F9FFFF   |call  SerialCh.004019C0  

00401FEB  |.  8B4424 24     |mov  eax,dword  ptr  ss:\[esp+24\]     ;  计算出来的数字。  

00401FEF  |.  83C0 F0       |add  eax,-10  

00401FF2  |.  C64424 30 07  |mov  byte  ptr  ss:\[esp+30\],7  

00401FF7  |.  8D48 0C       |lea  ecx,dword  ptr  ds:\[eax+C\]  

00401FFA  |.  83CA FF       |or   edx,FFFFFFFF  

00401FFD  |.  F0:0FC111     |lock  xadd  dword  ptr  ds:\[ecx\],edx  

00402001  |.  4A            |dec  edx  

00402002  |.  85D2          |test  edx,edx  

00402004  |.  7F 08         |jg   short  SerialCh.0040200E  

00402006  |.  8B08          |mov  ecx,dword  ptr  ds:\[eax\]  

00402008  |.  8B11          |mov  edx,dword  ptr  ds:\[ecx\]  

0040200A  |.  50            |push  eax  

0040200B  |.  FF52 04       |call  dword  ptr  ds:\[edx+4\]  

0040200E  |>  8B4424 20     |mov  eax,dword  ptr  ss:\[esp+20\]     ;  10，这个是固定串长度  

00402012  |.  46            |inc  esi  

00402013  |.  3BF0          |cmp  esi,eax                           ;  比较一下 esi（固定串长度），是不是小于 eax（碎片长度）  

00402015  |.  72 02         |jb   short  SerialCh.00402019  

00402017  |.  33F6          |xor  esi,esi                           ;  如果小于就 xor  

00402019  |>  8B4424 1C     |mov  eax,dword  ptr  ss:\[esp+1C\]  

0040201D  |.  47            |inc  edi                               ;  变量加一  

0040201E  |.  3BF8          |cmp  edi,eax                           ;  有没有处理完毕？  

00402020  |.^ 0F82 60FFFFFF \jb  SerialCh.00401F86  

00402026  |>  8B6C24 10     mov  ebp,dword  ptr  ss:\[esp+10\]       ;  "114097120103122"  

0040202A  |.  8B4D F0       mov  ecx,dword  ptr  ss:\[ebp\-10\]  

0040202D  |.  8B01          mov  eax,dword  ptr  ds:\[ecx\]  

0040202F  |.  83C5 F0       add  ebp,-10  

00402032  |.  FF50 10       call  dword  ptr  ds:\[eax+10\]  

00402035  |.  8B55 0C       mov  edx,dword  ptr  ss:\[ebp+C\]  

00402038  |.  85D2          test  edx,edx  

0040203A  |.  8D4D 0C       lea  ecx,dword  ptr  ss:\[ebp+C\]  

0040203D  |.  7C 1C         jl   short  SerialCh.0040205B  

0040203F  |.  3B45 00       cmp  eax,dword  ptr  ss:\[ebp\]  

00402042  |.  75 17         jnz  short  SerialCh.0040205B  

00402044  |.  8BC5          mov  eax,ebp  

00402046  |.  BA 01000000   mov  edx,1  

0040204B  |.  F0:0FC111     lock  xadd  dword  ptr  ds:\[ecx\],edx  

0040204F  |.  EB 3D         jmp  short  SerialCh.0040208E  

00402051  |>  68 57000780   push  80070057  

00402056  |.  E8 C5F0FFFF   call  SerialCh.00401120  

0040205B  |>  8B4D 04       mov  ecx,dword  ptr  ss:\[ebp+4\]  

0040205E  |.  8B10          mov  edx,dword  ptr  ds:\[eax\]  

00402060  |.  6A 01         push  1  

00402062  |.  51            push  ecx  

00402063  |.  8BC8          mov  ecx,eax  

00402065  |.  FF12          call  dword  ptr  ds:\[edx\]  

00402067  |.  85C0          test  eax,eax  

00402069  |.  75 05         jnz  short  SerialCh.00402070  

0040206B  |.  E8 00F2FFFF   call  SerialCh.00401270  

00402070  |>  8B55 04       mov  edx,dword  ptr  ss:\[ebp+4\]  

00402073  |.  8950 04       mov  dword  ptr  ds:\[eax+4\],edx  

00402076  |.  8B4D 04       mov  ecx,dword  ptr  ss:\[ebp+4\]  

00402079  |.  41            inc  ecx  

0040207A  |.  8BD1          mov  edx,ecx  

0040207C  |.  C1E9 02       shr  ecx,2  

0040207F  |.  8D75 10       lea  esi,dword  ptr  ss:\[ebp+10\]  

00402082  |.  8D78 10       lea  edi,dword  ptr  ds:\[eax+10\]  

00402085  |.  F3:A5         rep  movs  dword  ptr  es:\[edi\],dword  p>  

00402087  |.  8BCA          mov  ecx,edx  

00402089  |.  83E1 03       and  ecx,3  

0040208C  |.  F3:A4         rep  movs  byte  ptr  es:\[edi\],byte  ptr\>  

0040208E  |>  8B7424 38     mov  esi,dword  ptr  ss:\[esp+38\]  

00402092  |.  83C0 10       add  eax,10  

00402095  |.  8906          mov  dword  ptr  ds:\[esi\],eax          ;  再次出现真码，不过没有必要做内存注册机了吧！～  

00402097  |.  8B4424 14     mov  eax,dword  ptr  ss:\[esp+14\]  

0040209B  |.  83C0 F0       add  eax,-10  
```



```
0040209E  |.  C64424 30 02  mov byte ptr ss:[esp+30],2  

004020A3  |.  8D48 0C       lea ecx,dword ptr ds:[eax+C]  

004020A6  |.  83CA FF       or edx,FFFFFFFF  

004020A9  |.  F0:0FC111     lock xadd dword ptr ds:[ecx],edx  

004020AD  |.  4A            dec edx  

004020AE  |.  85D2          test edx,edx  

004020B0  |.  7F 08         jg short SerialCh.004020BA  

004020B2  |.  8B08          mov ecx,dword ptr ds:[eax]  

004020B4  |.  8B11          mov edx,dword ptr ds:[ecx]  

004020B6  |.  50            push eax  

004020B7  |.  FF52 04       call dword ptr ds:[edx+4]  

004020BA  |>  C64424 30 01  mov byte ptr ss:[esp+30],1  

004020BF  |.  8D45 0C       lea eax,dword ptr ss:[ebp+C]  

004020C2  |.  83C9 FF       or ecx,FFFFFFFF  

004020C5  |.  F0:0FC108     lock xadd dword ptr ds:[eax],ecx  

004020C9  |.  49            dec ecx  

004020CA  |.  85C9          test ecx,ecx  

004020CC  |.  7F 09         jg short SerialCh.004020D7  

004020CE  |.  8B4D 00       mov ecx,dword ptr ss:[ebp]  

004020D1  |.  8B11          mov edx,dword ptr ds:[ecx]  

004020D3  |.  55            push ebp  

004020D4  |.  FF52 04       call dword ptr ds:[edx+4]  

004020D7  |>  8B4424 18     mov eax,dword ptr ss:[esp+18]  

004020DB  |.  83C0 F0       add eax,-10  

004020DE  |>  C64424 30 00  mov byte ptr ss:[esp+30],0  

004020E3  |.  8D48 0C       lea ecx,dword ptr ds:[eax+C]  

004020E6  |.  83CA FF       or edx,FFFFFFFF  

004020E9  |.  F0:0FC111     lock xadd dword ptr ds:[ecx],edx  

004020ED  |.  4A            dec edx  

004020EE  |.  85D2          test edx,edx  

004020F0  |.  7F 08         jg short SerialCh.004020FA  

004020F2  |.  8B08          mov ecx,dword ptr ds:[eax]  

004020F4  |.  8B11          mov edx,dword ptr ds:[ecx]  

004020F6  |.  50            push eax  

004020F7  |.  FF52 04       call dword ptr ds:[edx+4]  

004020FA  |>  8D43 F0       lea eax,dword ptr ds:[ebx-10]  

004020FD  |.  C74424 30 FFF>mov dword ptr ss:[esp+30],-1  

00402105  |.  8D48 0C       lea ecx,dword ptr ds:[eax+C]  

00402108  |.  83CA FF       or edx,FFFFFFFF  

0040210B  |.  F0:0FC111     lock xadd dword ptr ds:[ecx],edx  

0040210F  |.  4A            dec edx  

00402110  |.  85D2          test edx,edx  

00402112  |.  7F 08         jg short SerialCh.0040211C  

00402114  |.  8B08          mov ecx,dword ptr ds:[eax]  

00402116  |.  8B11          mov edx,dword ptr ds:[ecx]  

00402118  |.  50            push eax  

00402119  |.  FF52 04       call dword ptr ds:[edx+4]  

0040211C  |>  8B4C24 28     mov ecx,dword ptr ss:[esp+28]  

00402120  |.  5F            pop edi  

00402121  |.  8BC6          mov eax,esi  

00402123  |.  5E            pop esi  

00402124  |.  5D            pop ebp  

00402125  |.  5B            pop ebx  

00402126  |.  64:890D 00000>mov dword ptr fs:[0],ecx  

0040212D  |.  83C4 24       add esp,24  

00402130  |\.  C2 0800      retn 8  

00402133      CC            int3  
```

嗯？怎么没有验证假码的部分？哦，原来只是计算出最终验证码，看来是习惯了真假码了。

明白了计算过程了没有，用碎片逐位跟固定串 `XOR`，如果固定串较短，就先 `xor`(就是循环逐位取出，到了最后，下一次从第一个取)。计算出的结果转成十进制，并加上前导 `00`，最后取右边三位。

`12345`->`114097120103122`

### 【算法总结】

**注册验证代码：**

```
sn = CStr(Len(user) Mod 9) & CStr(Asc(Mid$(user, 1, 1)) * Len(user) Mod 9) & CStr((Asc(Mid$(user, 2, 1)) + Asc(Mid$(user, 3, 1))) Mod 9)

sn = sn & "-" & CStr(Asc(Mid$(user, 1, 1)) * 5 Mod &H14) & CStr(Asc(Mid$(user, 2, 1)) * 4 Mod &H21) & CStr(Asc(Mid$(user, 3, 1)) Mod &H2C)

sn = sn & CStr(Asc(Mid$(user, 4, 1)) * 3 Mod &H37) & CStr(Asc(Mid$(user, 5, 1)) * 6 Mod &H43) & CStr(Asc(Mid$(user, 6, 1)) * 8 Mod &H4D)
```

**验证码计算代码：**

```
user = s1 + s2 + s3 + s4 + s5

code = "CSKSOFT-Personal"

For i = 1 To Len(user)
    sn = sn + Right$("00" & CStr(Asc(Mid$(user, i, 1)) Xor Asc(Mid$(code, (i - 1) Mod Len(code) + 1, 1))), 3)
Next
```

### 【破解心得】

流程很清楚，算法也很清楚，可以当作 `vc` 的入门练手 `crackme`。算法的关键是注意 `edx` 用来保存余数，`cdq` 将 `edx` 清零。掌握了这两点，算法就很容易懂了。原来写破文比破要难。

========================================

献给 `DFCG`，祝 `DFCG` 蒸蒸日上！

感谢 `DFCG` 的各位朋友！

感谢 `POJE` 论坛的各位朋友！

========================================

【`KiLlL[DFCG]` 于 2005-08-14 23:37 完成】

`4dd466d0e1e0cc123afbe504da2462ca`
