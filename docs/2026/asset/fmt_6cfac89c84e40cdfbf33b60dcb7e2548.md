
# [原创][表格算法]电子小说浏览器 ReadBook v1.51 Plus 注册算法分析 ＋ delphi(kol/mck)完整注册机源代码(包含所有的5种注册情况)

这几天在看『诛仙』这本书，真好看，不过就是看网页看久了刺眼，便下了个ReadBook 1.51 plus版本的小说浏览器来看，发现这个共享软件是需要注册，爱物及乌，自然想看看它的注册算法是如何的。

可上来一搜索，却突然发现这么好的一个软件居然在论坛上没有一个与这版本相关的算法分析文章，而找到的以前版本的算法分析又不适用，这怎么可以嘛！这么好的软件没有一个相关的分析，心中不爽呀！于是...，希望有人喜欢，呵呵 :)

好，先分析这个程序，使用了`PEtite2.2`压缩，在注册框中输入注册名和假码后，会有提示：‘注册码错误,请重新填写注册码!’，顺着这个提示，将脱壳后的程序拖入OD中，先不运行，使用字符串参考，找到了提示所在的位置：

字符串参考 , 条目 346

地址=`00409F28`

反汇编地址=push    un-packe.`00475AAC`

字符内容=注册码错误,请重新填写注册码!

而在它的上面便是注册成功的提示：‘您已经成功地注册了!’

字符串参考 , 条目 341

地址=`00409EC6`

反汇编地址=push    un-packe.`00475A7C`

字符内容=祝贺

字符串参考 , 条目 342

地址=`00409ECB`

反汇编地址=push    un-packe.`00475A68`

字符内容=您已经成功地注册了!

双击错误提示、成功提示的字符串后，分别来到了如下的地方：

```
00409F21   .  68 CC5A4700   push    un-packe.00475ACC                 ;  错了啦!
00409F26   .  8BF1          mov     esi, ecx
00409F28   .  68 AC5A4700   push    un-packe.00475AAC                 ;  注册码错误,请重新填写注册码!
```

```
00409EBD  |.  3935 A4AC4C00 cmp     dword ptr [4CACA4], esi
00409EC3  |.  74 4D         je      short un-packe.00409F12
00409EC5  |.  56            push    esi
00409EC6  |.  68 7C5A4700   push    un-packe.00475A7C                 ;  祝贺
00409ECB  |.  68 685A4700   push    un-packe.00475A68                 ;  您已经成功地注册了!
```

顺着它们往上查看，来到开始处：

```
00409D58  |.  FF70 1C       push    dword ptr [eax+1C]                ; |hWnd
00409D5B  |.  FF15 74064600 call    dword ptr [<&USER32.PostMessageA>>; \PostMessageA
00409D61  |.  E9 B3010000   jmp     un-packe.00409F19
00409D66  |>  8D85 48FFFFFF lea     eax, dword ptr [ebp-B8]           ;  从这里开始分析
00409D6C  |.  50            push    eax
00409D6D  |.  E8 A4550300   call    un-packe.0043F316F7进入
00409D72  |.  8B85 54FFFFFF mov     eax, dword ptr [ebp-AC]
```

因此，我们在`00409D66`这个地方按F2键下断点，然后Ctrl＋F2重新载入程序，F9运行，这时会弹出一个询问框，告诉你文件被感染了，这是程序的自验校，不管它，继续运行，然后打开注册输入窗口，输入一组实验码：

name：aCaFeeL

code：12345678

点击‘填写完毕’按钮，程序便被中断在了如下位置（开始分析）：

```
『以上代码省略』
00409D5B  |.  FF15 74064600 call    dword ptr [<&USER32.PostMessageA>>; \PostMessageA
00409D61  |.  E9 B3010000   jmp     un-packe.00409F19                 ; ××下面便是被中断的开始位置××
00409D66  |>  8D85 48FFFFFF lea     eax, dword ptr [ebp-B8]           ;  注册码到eax中
00409D6C  |.  50            push    eax                               ;  压栈
00409D6D  |.  E8 A4550300   call    un-packe.0043F316                 ;  将注册名全部转为小写，写入0012FAB0开始的地方
00409D72  |.  8B85 54FFFFFF mov     eax, dword ptr [ebp-AC]           ; |注册名13～16位(hex反序) ->eax|不足用$00$20..$20填补
00409D78  |.  8B8D 4CFFFFFF mov     ecx, dword ptr [ebp-B4]           ; |注册名8～5位(hex反序)   ->ecx|不足用$00$20..$20填补
00409D7E  |.  8BB5 48FFFFFF mov     esi, dword ptr [ebp-B8]           ; |注册名4～1位(hex反序)   ->esi|不足用$00$20..$20填补
00409D84  |.  03C8          add     ecx, eax                          ; |ecx = ecx + eax
00409D86  |.  038D 50FFFFFF add     ecx, dword ptr [ebp-B0]           ; |ecx = ecx+注册名12～9位(反序)|不足用$00$20..$20填补
00409D8C  |.  69F6 31750000 imul    esi, esi, 7531                    ; |esi = esi \* $7531;
00409D92  |.  69C9 31750000 imul    ecx, ecx, 7531                    ; |ecx = ecx \* $7531;
00409D98  |.  C70424 605447>mov     dword ptr [esp], un-packe.0047546>; |ASCII "BIN\_OR\_TEXT"
00409D9F  |.  68 B5000000   push    0B5                               ; |ResourceName = B5
00409DA4  |.  6A 00         push    0                                 ; |hModule = NULL
00409DA6  |.  2BF1          sub     esi, ecx                          ; |esi = esi - ecx;//最终结果值
00409DA8  |.  FF15 38024600 call    dword ptr [<&KERNEL32.FindResourc>; \FindResourceA
00409DAE  |.  50            push    eax                               ; /hResource
00409DAF  |.  6A 00         push    0                                 ; |hModule = NULL
00409DB1  |.  FF15 40024600 call    dword ptr [<&KERNEL32.LoadResourc>; \LoadResource
00409DB7  |.  68 985A4700   push    un-packe.00475A98                 ;  ASCII "Register"
00409DBC  |.  57            push    edi
00409DBD  |.  8945 F0       mov     dword ptr [ebp-10], eax
00409DC0  |.  E8 F8ED0000   call    un-packe.00418BBD                 ;  获取ini文件的节点
00409DC5  |.  68 845A4700   push    un-packe.00475A84                 ;  ASCII "RegisterEncryptMode"
00409DCA  |.  57            push    edi
00409DCB  |.  8945 FC       mov     dword ptr [ebp-4], eax
00409DCE  |.  C745 F4 12320>mov     dword ptr [ebp-C], 3212
00409DD5  |.  C745 E8 34120>mov     dword ptr [ebp-18], 1234
00409DDC  |.  C745 EC 88880>mov     dword ptr [ebp-14], 8888
00409DE3  |.  C745 F8 23230>mov     dword ptr [ebp-8], 2323
00409DEA  |.  E8 CEED0000   call    un-packe.00418BBD                 ;  获取ini文件的节点
00409DEF  |.  83C4 10       add     esp, 10
00409DF2  |.  85C0          test    eax, eax
00409DF4  |.  75 24         jnz     short un-packe.00409E1A
00409DF6  |.  8D45 F8       lea     eax, dword ptr [ebp-8]            ;  2323 -> eax
00409DF9  |.  50            push    eax                               ; /pClusters
00409DFA  |.  8D45 EC       lea     eax, dword ptr [ebp-14]           ; |8888 -> eax
00409DFD  |.  50            push    eax                               ; |pFreeClusters
00409DFE  |.  8D45 E8       lea     eax, dword ptr [ebp-18]           ; |1234 -> eax
00409E01  |.  50            push    eax                               ; |pBytesPerSector
00409E02  |.  8D45 F4       lea     eax, dword ptr [ebp-C]            ; |3212 -> eax
00409E05  |.  50            push    eax                               ; |pSectorsPerCluster
00409E06  |.  68 34594700   push    un-packe.00475934                 ; |c:\
00409E0B  |.  FF15 FC014600 call    dword ptr [<&KERNEL32.GetDiskFree>; \GetDiskFreeSpaceA
00409E11  |.  8B45 F8       mov     eax, dword ptr [ebp-8]
00409E14  |.  0FAF45 F4     imul    eax, dword ptr [ebp-C]
00409E18  |.  EB 19         jmp     short un-packe.00409E33
00409E1A  |>  8D45 C8       lea     eax, dword ptr [ebp-38]
00409E1D  |.  C745 C8 20000>mov     dword ptr [ebp-38], 20
00409E24  |.  50            push    eax                               ; /pMemStat
00409E25  |.  FF15 EC014600 call    dword ptr [<&KERNEL32.GlobalMemor>; \GlobalMemoryStatus
00409E2B  |.  8B45 D4       mov     eax, dword ptr [ebp-2C]
00409E2E  |.  05 CCEDFFFF   add     eax, -1234
00409E33  |>  F7D0          not     eax                               ;  not eax
00409E35  |.  3145 FC       xor     dword ptr [ebp-4], eax
00409E38  |.  33C0          xor     eax, eax                          ;  eax 清零
00409E3A  |.  A3 B8774700   mov     dword ptr [4777B8], eax
00409E3F  |>  3975 FC       /cmp     dword ptr [ebp-4], esi
00409E42  |.  74 1D         |je      short un-packe.00409E61
00409E44  |.  8B55 F0       |mov     edx, dword ptr [ebp-10]          ;  [ebp-10] -> edx
00409E47  |.  8BC8          |mov     ecx, eax                         ;  eax -> ecx
00409E49  |.  83E1 7F       |and     ecx, 7F                          ;  ecx = ecx and $7F;
00409E4C  |.  03348A        |add     esi, dword ptr [edx+ecx\*4]       ;  esi = esi + key[i](由ecx值决定的 004E78D0~004E7ACC 一张表中的相应数值);
00409E4F  |.  40            |inc     eax                              ;  i = i + 1;
00409E50  |.  3D FF0F0000   |cmp     eax, 0FFF                        ;  ? > $0FFF-1
00409E55  |.  A3 B8774700   |mov     dword ptr [4777B8], eax
00409E5A  |.^ 72 E3         \jb      short un-packe.00409E3F          ;  小于$0FFF便继续循环
00409E5C  |.  3975 FC       cmp     dword ptr [ebp-4], esi            ;  esi的十进制无符号数便为真正的注册码
00409E5F  |.  75 78         jnz     short un-packe.00409ED9           ;  关键跳转,真假码相等不跳
00409E61  |>  8B35 EC054600 mov     esi, dword ptr [<&USER32.***GetMenu**>>;  以下便是注册成功后，保存注册信息到ini文件中的代码
00409E67  |.  C705 A0AC4C00>mov     dword ptr [4CACA0], 1
00409E71  |.  FF73 1C       push    dword ptr [ebx+1C]                ; /hWnd
```

『以下代码省略』

通过上面的分析，我们知道了其算法主要是由一张表的结果累加，再配上不等的逻辑和算术运算的结果决定的，而这张表可从：

```
00409E4C  |.  03348A        |add     esi, dword ptr [edx+ecx\*4]
```
