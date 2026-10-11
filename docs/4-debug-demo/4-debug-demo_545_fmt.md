
# Remote Administrator 2.2 服务器端去自效验+文件名效验

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-76.htm`](https://www.pediy.com/kssd/pediy09/pediy09-76.htm)

> 【文章标题】: Remote Administrator 2.2 服务器端去自效验+文件名效验  
> 【文章作者】: KiD  
> 【作者邮箱】: iiedii@163.com  
> 【软件名称】: Remote Administrator 2.2  
> 【下载地址】: [`www.hanzify.org/index.php?Go=Show::List&ID=5887`](http://www.hanzify.org/index.php?Go=Show::List&ID=5887)  
> 【加壳方式】: 无壳  
> 【编写语言】: Microsoft Visual C++ 6.0  
> 【使用工具】: `OD`, `PEiD`, `HIEW`  
> 【软件介绍】: 一个效率和安全性都非常高的远程控制软件  
> 【作者声明】: 我是新菜，对破解只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!  
> \--------------------------------------------------------------------------------  
> 【详细过程】

Remote Administrator 2.2 是 Famatech 公司 1994 年出品的远程控制经典产品了。貌似从 1994 年的推出的这一个版本开始，Famatech 就着手自效验保护，所以服务器端一直没法汉化。估计现在它推出的新的版本保护雷同，就看 2.2 的算了。

**情况初探**：`r_server.exe` 是服务器端的主程序，不加壳看来是该公司的惯常伎俩，给它加个壳如何？我用 `ASPR` 加之，很快得到了答复“`Executable file is corrupted`”，这还算友好的。更可恶的是修改了文件名之后，比如我把文件名改为 `r_server_m.exe`，再运行就无提示自动退出了。

**保护分析**：它的自效验保护并不像传统软件的自效验那样，改个跳转了事。它把整个自效验过程连同其它的一些必要过程压缩存放，在运行的时候才当场解压出这段代码。这么做的好处是：你还没法直接修改这段代码，因为原始文件里面是密文存放的；你也没法屏蔽掉这段代码，因为它还包含一些软件运行必须的过程。整个保护有点像融入了一点壳的手段。

**破解分析**：对付这种保护，先从理论上入手，我们等它刚刚把这些代码解压完毕时，修改掉那些关键跳转不就了事拉。。。

下面从实道来

>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

这里大胆地不设任何断点，从程序入口开始顺序跟踪。。。嘿嘿，这样顺序地跟着程序走，对我们这些新菜来说思路会比较清晰

先看看平平常常的程序入口：

```
01406FDB >/$  55         push    ebp
01406FDC  |.  8BEC       mov     ebp, esp
01406FDE  |.  6A FF     push    -1
01406FE0  |.  68 B8EE4001 push  0140EEB8
01406FE5  |.  68 E86D4001 push  01406DE8                 ;  SE 处理程序安装
```

注意地址都是 `0140XXXX`，因为后面会出现微妙的变化。

单步执行不久，程序走到：

```
01407097 |>  50         push    eax
01407098  |.  FF75 9C    push    dword ptr [ebp-64]
0140709B  |.  56         push    esi
0140709C  |.  56         push    esi                      ; /pModule
0140709D  |.  FF15 00B04001 call dword ptr [<&KERNEL32.GetMod..    ; \\GetModuleHandleA
014070A3  |.  50         push    eax
014070A4  |.  E8 F7A1FFFF call  014012A0                   ;  弹出提示的 call
                                                           ;  如果修改了文件名程序就从这里直接退出
```

找到一个关键的 `call`，跟进 `014012A0` 看看，原来还复杂着：

---------- call `014012A0` 内部 ----------

在 `014012A0` 中跟过了几次无关紧要的判断程序是否出错，几步路来到：

```
01401333   > \\8B4D C0      mov     ecx, dword ptr [ebp-40]        ;  r_server.01417250
01401336   .  8B11         mov     edx, dword ptr [ecx]
01401338   .  8955 BC      mov     dword ptr [ebp-44], edx
0140133B   .  6A 40        push    40                              ; /Protect = PAGE_EXECUTE_READWRITE
0140133D   .  68 00100000  push    1000                            ; |AllocationType = MEM_COMMIT
01401342   .  8B45 BC      mov     eax, dword ptr [ebp-44]        ; |
01401345   .  50          push    eax                             ; |Size
01401346   .  6A 00       push    0                               ; |Address = NULL
01401348   .  FF15 0CB04001 call dword ptr [<&KERNEL32.VirtualA... ; \\VirtualAlloc 申请分配虚拟内存
```

后面还有：

```
........   （略去几个）
01401595   .  6A 40        push    40                              ; /Protect = PAGE_EXECUTE_READWRITE
01401597   .  68 00100000  push    1000                            ; |AllocationType = MEM_COMMIT
0140159C   .  8B45 E0      mov     eax, dword ptr [ebp-20]        ; |
0140159F   .  6BC0 18      imul    eax, eax, 18                   ; |
014015A2   .  8B4D A8      mov     ecx, dword ptr [ebp-58]        ; |
014015A5   .  8B5401 0C    mov     edx, dword ptr [ecx+eax+C]     ; |
014015A9   .  52          push    edx                             ; |Size
014015AA   .  8B45 E0      mov     eax, dword ptr [ebp-20]        ; |
014015AD   .  6BC0 18      imul    eax, eax, 18                   ; |
014015B0   .  8B4D A8      mov     ecx, dword ptr [ebp-58]        ; |
014015B3   .  8B15 08284101 mov     edx, dword ptr [1412808]      ; |
014015B9   .  035401 08    add     edx, dword ptr [ecx+eax+8]     ; |
014015BD   .  52          push    edx                             ; |Address
014015BE   .  FF15 0CB04001 call dword ptr [<&KERNEL32.VirtualA... ; \\VirtualAlloc 申请分配虚拟内存
```

哈哈，竟然找到一堆 `VirtualAlloc`，看来它企图请求分配虚拟内存，然后把一些代码写到这段内存里面。顺便查查资料：在 Win32 系统中，调用 `VirtualAlloc` 函数分配虚拟内存，申请成功后用 `VirtualFree` 可以释放掉。

穿插在这一堆 `VirtualAlloc` 之间的是大量的循环，诸如：

```
0140165C   > \\8B45 E0      mov     eax, dword ptr [ebp-20]
0140165F   .  3B45 B4      cmp     eax, dword ptr [ebp-4C]
01401662   .  0F83 99020000 jnb    01401901
01401668   .  C785 A8FEFFFF>mov     dword ptr [ebp-158], 0
01401672   .  C785 ACFEFFFF>mov     dword ptr [ebp-154], 0
0140167C   .  C785 ACFAFFFF>mov     dword ptr [ebp-554], 0
01401686   .  8B4D E0      mov     ecx, dword ptr [ebp-20]
01401689   .  69C9 07010000 imul    ecx, ecx, 107
0140168F   .  8B55 B0      mov     edx, dword ptr [ebp-50]
01401692   .  A1 08284101  mov     eax, dword ptr [1412808]
01401697   .  03840A FF0000>add     eax, dword ptr [edx+ecx+FF]
0140169E   .  8985 B4FAFFFF mov     dword ptr [ebp-54C], eax
014016A4   .  8B4D E0      mov     ecx, dword ptr [ebp-20]
014016A7   .  69C9 07010000 imul    ecx, ecx, 107
014016AD   .  8B55 B0      mov     edx, dword ptr [ebp-50]
014016B0   .  03D1         add     edx, ecx
014016B2   .  8D8D B8FAFFFF lea     ecx, dword ptr [ebp-548]
014016B8   .  E8 D3040000  call    01401B90
014016BD   .  8B55 E0      mov     edx, dword ptr [ebp-20]
014016C0   .  69D2 07010000 imul    edx, edx, 107
014016C6   .  8B45 B0      mov     eax, dword ptr [ebp-50]
014016C9   .  8B8C10 030100>mov     ecx, dword ptr [eax+edx+103]
014016D0   .  898D ACFAFFFF mov     dword ptr [ebp-554], ecx
014016D6   >  C785 B0FAFFFF>mov     dword ptr [ebp-550], 0
014016E0   .  8D8D B8FAFFFF lea     ecx, dword ptr [ebp-548]
014016E6   .  E8 25040000  call    01401B10
014016EB   .  83E8 01      sub     eax, 1
014016EE   .  8985 A4FEFFFF mov     dword ptr [ebp-15C], eax
014016F4   .  EB 0F        jmp     short 01401705
014016F6   >  8B95 A4FEFFFF mov     edx, dword ptr [ebp-15C]
014016FC   .  83EA 01      sub     edx, 1
014016FF   .  8995 A4FEFFFF mov     dword ptr [ebp-15C], edx
01401705   >  83BD A4FEFFFF>cmp     dword ptr [ebp-15C], 0
0140170C   .  7E 17        jle     short 01401725
0140170E   .  8B85 A4FEFFFF mov     eax, dword ptr [ebp-15C]
01401714   .  0FBE8C05 B8FA>movsx   ecx, byte ptr [ebp+eax-548]
0140171C   .  83F9 2E      cmp     ecx, 2E
0140171F   .  75 02        jnz     short 01401723
01401721   .  EB 02        jmp     short 01401725
01401723   >^ EB D1        jmp     short 014016F6
```

懒得管它们，反正大致是不厌其烦地将代码解压到申请的内存空间。。。这一过程会持续很久，单步显然是受不了了。向后看，注意到 `014019EB` 处，整个解压过程结束。

```
014019EB   > \\8B45 E0      mov     eax, dword ptr [ebp-20]        ; 代码解压过程完成，在这里设断
014019EE   .  3B45 AC      cmp     eax, dword ptr [ebp-54]
014019F1   .  75 1B        jnz     short 01401A0E
014019F3   .  6A 00       push    0                               ; /Style = MB_OK|MB_APPLMODAL
014019F5   .  68 10284101  push    01412810                        ; |Title = ""
```



## 反汇编代码段

```
014019FA   .  68 30004101   push    01410030                      ; |Text = "Err7"  
014019FF   .  6A 00         push    0                             ; |hOwner = NULL  
01401A01   .  FF15 A0B04001 call    dword ptr [<&USER32.MessageBoxA>] ; \MessageBoxA  
01401A07   .  B8 01000000   mov     eax, 1  
01401A0C   .  EB 2A         jmp     short 01401A38  
01401A0E   >  68 00800000   push    8000                          ; /FreeType = MEM_RELEASE  
01401A13   .  6A 00         push    0                             ; |Size = 0  
01401A15   .  8B4D E8       mov     ecx, dword ptr [ebp-18]      ; |  
01401A18   .  51            push    ecx                           ; |Address  
01401A19   .  FF15 08B04001 call    dword ptr [<&KERNEL32.VirtualF... ; \VirtualFree  
01401A1F   .  8B15 08284101 mov     edx, dword ptr [1412808]     ; 在我这里 EDX = 9F0000，它是申请到的内存段的基址  
01401A25   .  0395 D8FEFFFF add     edx, dword ptr [ebp-128]     ; 换算得到申请内存段的程序入口  
01401A2B   .  8955 B8       mov     dword ptr [ebp-48], edx     ; 保存该入口地址  
01401A2E   .  8B45 B8       mov     eax, dword ptr [ebp-48]     ;  
01401A31   .  FFE0          jmp     eax                           ; 跳到该入口地址，即跳到解压出的程序入口  
```

跳转过后：

### 解压出的代码内部

```
---------- 解压出的代码内部 ----------

00A234DC    55             push    ebp  
00A234DD    8BEC          mov     ebp, esp  
00A234DF    6A FF         push    -1  
00A234E1    68 1071A300   push    0A37110  
00A234E6    68 0434A200   push    0A23404  
```

晕。。。简直就是一个 `PE` 文件的开头。注意到地址已经变为 `00A2XXXX`，现在程序已经运行在解压出的代码处了。实际上，在我这里，这段内存空间的地址是从 `009F0000`-`00B30FFF`。在这段空间里面工作感觉不爽，因为 `OD` 的一些功能不能用了，比如代码分析，转换到其它模块等等（不知是不是我太笨没有找到）。

又开始单步，几步路后来到这里：

```
00A235AA    50             push    eax  
00A235AB    FF75 9C        push    dword ptr [ebp-64]  
00A235AE    56             push    esi  
00A235AF    56             push    esi  
00A235B0    FF15 98D2A200  call    dword ptr [A2D298]            ; kernel32.GetModuleHandleA  
00A235B6    50             push    eax  
00A235B7    E8 64C4FDFF    call    009FFA20                      ; 再走就弹出提示  
                                                          ; 如果修改了文件名程序就从这里直接退出  
```

看来这个才是真正要找的 `call`，它负责服务端程序的一些初始化工作以及一些例行的检查，想必自效验和文件名检测都在里面。进去看看：

### `call 009FFA20` 内部

```
---------- call 009FFA20 内部 ----------

009FFA20    8B4424 04      mov     eax, dword ptr [esp+4]  
009FFA24    6A 00          push    0  
009FFA26    A3 3C80A600    mov     dword ptr [A6803C], eax  
009FFA2B    E8 E2360200    call    00A23112  
009FFA30    50             push    eax  
009FFA31    E8 D9310200    call    00A22C0F  
009FFA36    8B4C24 14      mov     ecx, dword ptr [esp+14]  
009FFA3A    83C4 08        add     esp, 8  
009FFA3D    68 3480A600    push    0A68034  
009FFA42    BA 3880A600    mov     edx, 0A68038  
009FFA47    E8 D4FF0000    call    00A0FA20  
009FFA4C    85C0           test    eax, eax  
009FFA4E    75 03          jnz     short 009FFA53  
009FFA50    C2 1000        retn    10  
009FFA53    E8 081A0000    call    00A01460                      ; 这个 call 完成一些初始化和自效验以及文件名效验的工作  
009FFA58    85C0           test    eax, eax  
009FFA5A    75 03          jnz     short 009FFA5F                ; 不跳就挂了  
009FFA5C    C2 1000        retn    10  
009FFA5F    E8 ACCCFFFF    call    009FC710                      ; 这个 call 负责一些启动参数的判断，无关紧要:)  
009FFA64    E8 071C0000    call    00A01670  
009FFA69    B8 01000000    mov     eax, 1  
009FFA6E    C2 1000        retn    10  
```

关注 `009FFA53` 处的 `call`，我们来到了最终的地方。

### `call 00A01460` 内部

```
---------- call 00A01460 内部 ----------

00A01460  
........ （跳过一些初始化过程）  
00A01511    8B0D 3C80A600   mov     ecx, dword ptr [A6803C]       ; r_server.01400000  
00A01517    BA BC5EC995     mov     edx, 95C95EBC  
00A0151C    E8 9F8DFFFF     call    009FA2C0  
00A01521    85C0            test    eax, eax  
00A01523    0F84 21010000   je      00A0164A  
00A01529    8B0D 3C80A600   mov     ecx, dword ptr [A6803C]       ; r_server.01400000  
00A0152F    68 DA43F0D3     push    D3F043DA  
00A01534    8D9424 A8000000 lea     edx, dword ptr [esp+A8]  
00A0153B    E8 3091FFFF     call    009FA670                      ; 文件自效验  
00A01540    85C0            test    eax, eax                      ; 效验结果送到 eax  
00A01542    0F84 02010000   je      00A0164A                      ; 跳就死，弹出提示“可执行文件被破坏”  
00A01548    57              push    edi  
00A01549    83C9 FF         or      ecx, FFFFFFFF  
00A0154C    33C0            xor     eax, eax  
00A0154E    8DBC24 A8000000 lea     edi, dword ptr [esp+A8]  
00A01555    F2:AE           repne   scas byte ptr es:[edi]  
00A01557    F7D1            not     ecx  
00A01559    49              dec     ecx  
00A0155A    5F              pop     edi  
00A0155B    74 1F           je      short 00A0157C  
00A0155D    80BC0C A4000000>cmp     byte ptr [esp+ecx+A4], 2E  
00A01565    75 08           jnz     short 00A0156F  
00A01567    C6840C A4000000>mov     byte ptr [esp+ecx+A4], 0  
00A0156F    80BC0C A4000000>cmp     byte ptr [esp+ecx+A4], 5C  
00A01577    74 0D           je      short 00A01586  
00A01579    49              dec     ecx  
00A0157A  ^ 75 E1           jnz     short 00A0155D  
00A0157C    80BC0C A4000000>cmp     byte ptr [esp+ecx+A4], 5C  
00A01584    75 01           jnz     short 00A01587  
00A01586    41              inc     ecx  
00A01587    8D8C0C A4000000 lea     ecx, dword ptr [esp+ecx+A4]    ; ECX 装入当前的服务端程序的文件名  
00A0158E    68 9490A300     push    0A39094                         ; ASCII "r_server" 服务端原始文件名入栈  
00A01593    51              push    ecx                             ; 服务端当前文件名入栈  
00A01594    E8 37B70200     call    00A2CCD0                        ; 比较两个文件名是否相同  
00A01599    83C4 08         add     esp, 8  
00A0159C    85C0            test    eax, eax                        ; 比较结果送到 eax  
00A0159E    0F85 B5000000   jnz     00A01659                        ; 跳就死，什么提示都没有  
00A015A4    8D5424 04       lea     edx, dword ptr [esp+4]  
00A015A8    52              push    edx  
00A015A9    BE 00000100     mov     esi, 10000                      ; UNICODE "=::=::\\"  
00A015AE    68 3877A500     push    0A57738  
00A015B3    BA BD0D0000     mov     edx, 0DBD  
00A015B8    B9 D0D5A200     mov     ecx, 0A2D5D0  
00A015BD    897424 0C       mov     dword ptr [esp+C], esi  
00A015C1    FF15 6CA6A300   call    dword ptr [A3A66C]  
00A015C7    85C0            test    eax, eax  
00A015C9    0F84 8A000000   je      00A01659  
00A015CF    8D4424 04       lea     eax, dword ptr [esp+4]  
00A015D3    50              push    eax  
00A015D4    68 3877A400     push    0A47738  
00A015D9    BA 39060000     mov     edx, 639  
00A015DE    B9 90E3A200     mov     ecx, 0A2E390  
00A015E3    897424 0C       mov     dword ptr [esp+C], esi  
00A015E7    FF15 6CA6A300   call    dword ptr [A3A66C]  
00A015ED    85C0            test    eax, eax  
00A015EF    74 68           je      short 00A01659  
00A015F1    8B35 A0D2A200   mov     esi, dword ptr [A2D2A0]        ; kernel32.GetVersionExA  
00A015F7    68 887FA600     push    0A67F88  
00A015FC    C705 887FA600 9>mov     dword ptr [A67F88], 94  
00A01606    FFD6            call    esi  
00A01608    85C0            test    eax, eax  
```



```
 00A0160A    74 4D           je      short 00A01659  
 00A0160C    8D4C24 08       lea     ecx, dword ptr [esp+8]  
 00A01610    51             push    ecx  
 00A01611    C74424 0C 9C0000>mov    dword ptr [esp+C], 9C  
 00A01619    FFD6            call    esi  
 00A0161B    85C0            test    eax, eax  
 00A0161D    74 0A           je      short 00A01629  
 00A0161F    8B5424 0C       mov     edx, dword ptr [esp+C]  
 00A01623    8915 8C7FA600   mov     dword ptr [A67F8C], edx  
 00A01629    A1 987FA600     mov     eax, dword ptr [A67F98]  
 00A0162E    A3 1C80A600     mov     dword ptr [A6801C], eax  
 00A01633    E8 E83E0000     call    00A05520  
 00A01638    A3 4C80A600     mov     dword ptr [A6804C], eax  
 00A0163D    B8 01000000     mov     eax, 1  
 00A01642    5E             pop     esi  
 00A01643    81C4 A0040000   add     esp, 4A0  
 00A01649    C3             retn  
 00A0164A    68 349DA300     push    0A39D34                   ; ASCII "Executable file is corrupted"  
 00A0164F    68 C9000000     push    0C9  
 00A01654    E8 A7A5FFFF     call    009FBC00  
 00A01659    33C0            xor     eax, eax  
 00A0165B    5E             pop     esi  
 00A0165C    81C4 A0040000   add     esp, 4A0  
 00A01662    C3             retn  
```

>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

到这里我们终于清楚了。我们只需要把 `00A01540` 处的 `eax` 改为 `1`，就破解了文件完整性自效验，把 `00A0159C` 处的 `eax` 改为 `0`，文件名效验就破了。具体方法是弄成这样：

```
00A01540    40             inc     eax  
00A01541    90             nop  

00A0159C    33C0            xor     eax, eax  
```

**解决方案**：前面也提到，由于效验过程的代码是解压出来的，我们无法直接修改，我们需要等这些代码解压完毕后再修改。看来我们必须做一个动态的补丁，在程序执行到一定程度后才开始修改代码。下面详述这个动态补丁的制作方法：

我们再来回顾一下，在效验过程的代码解压出来之后，程序还未跳进解压出来的代码入口之前，有些什么？

```
01401A1F   .  8B15 08284101 mov     edx, dword ptr [1412808]       ; 在我这里 EDX = 9F0000，它是申请到的内存段的基址  
01401A25   .  0395 D8FEFFFF add     edx, dword ptr [ebp-128]       ; 换算得到申请内存段的程序入口  
01401A2B   .  8955 B8       mov     dword ptr [ebp-48], edx       ; 保存该入口地址  
01401A2E   .  8B45 B8       mov     eax, dword ptr [ebp-48]       ;  
01401A31   .  FFE0          jmp     eax                           ; 跳到该入口地址，即跳到解压出的程序入口  
```

由于 `01401A1F` 处，`EDX` 装入了申请到的内存段的基址，我们的修改又正好要用到这个地址，所以我们就在这句之后添加补丁代码。

我打算从 `01401A2B` 处开始改，事先在程序中找一个 `code cave`，写入我们的补丁代码，我就写到程序尾部的 `0140A29D` 处：

---------- `code cave` ----------

```
0140A29D   > \\60            pushad                          ; 保护现场  
0140A29E   .  2B95 D8FEFFFF sub     edx, dword ptr [ebp-128]       ; 得到程序申请到的内存段的基址  
0140A2A4   .  66:3E:C782 40>mov     word ptr [edx+11540], 9040     ; 修改 00A01540 处的代码  
0140A2AE   .  66:3E:C782 9C>mov     word ptr [edx+1159C], 0C033    ; 修改 00A0159C 处的代码  
0140A2B8   .  61             popad                           ; 恢复现场  
0140A2B9   .  8955 B8       mov     dword ptr [ebp-48], edx       ; 恢复前面抹去的指令  
0140A2BC   .  8B45 B8       mov     eax, dword ptr [ebp-48]       ;  
0140A2BF   . ^ E9 6D77FFFF   jmp     01401A31                    ; 跳回原位置继续执行  
```

现在我们就有条件大胆地修改 `01401A2B` 处的代码了：

```
01401A1F   .  8B15 08284101 mov     edx, dword ptr [1412808]  
01401A25   .  0395 D8FEFFFF add     edx, dword ptr [ebp-128]  
01401A2B   .  E9 6D880000   jmp     0140A29D                    ; 大大地跳到我们的 code cave 入口  
01401A30     90            nop  
01401A31   >  FFE0          jmp     eax  
```

>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

这样，一切就联系好了。。。整个破解似乎也完工了，貌似无懈可击。但事实并非如此，因为这样修改后出现了一个奇怪的问题：

还记得 `009FFA53` 处的 `call`，这个最终进行文件完整性效验和文件名检查的 `call`：

---------- `call 00A01460` 内部 ----------

```
00A01460  
........   （跳过一些初始化过程）  
00A01511    8B0D 3C80A600   mov     ecx, dword ptr [A6803C]       ; r_server.01400000  
00A01517    BA BC5EC995     mov     edx, 95C95EBC  
00A0151C    E8 9F8DFFFF     call    009FA2C0                      ; <-- 在这里发生异常  
........   （略）  
00A0153B    E8 3091FFFF     call    009FA670                      ; 文件自效验  
00A01540    85C0            test    eax, eax  
00A01542    0F84 02010000   je      00A0164A  
........   （略）  
00A01587    8D8C0C A4000000 lea     ecx, dword ptr [esp+ecx+A4]  
00A0158E    68 9490A300     push    0A39094  
00A01593    51             push    ecx  
00A01594    E8 37B70200     call    00A2CCD0                      ; 文件名效验  
00A01599    83C4 08         add     esp, 8  
00A0159C    85C0            test    eax, eax  
00A0159E    0F85 B5000000   jnz     00A01659  
........  
```

程序执行到 `00A0151C` 处时，突然异常退出，异常原因貌似访问违规：内存地址 `F03241DB` 不可读。怪了，我根本没有修改到 `00A0151C` 和 `call 009FA2C0` 里面的代码阿。我对异常具体细节不很了解，什么地方出了问题？我是新菜，还望各位大师赐教。。。

现在我只能把我的部分补丁代码 `nop` 掉：

```
0140A29D   > \\60            pushad  
0140A29E   .  2B95 D8FEFFFF sub     edx, dword ptr [ebp-128]  
0140A2A4   .  66:3E:C782 40>mov     word ptr [edx+11540], 9040     ; 修改 00A01540 处的代码  
0140A2AE   .  66:3E:C782 9C>mov     word ptr [edx+1159C], 0C033    ; <-- 把这句 nop 掉  
0140A2B8   .  61             popad  
0140A2B9   .  8955 B8       mov     dword ptr [ebp-48], edx  
0140A2BC   .  8B45 B8       mov     eax, dword ptr [ebp-48]  
0140A2BF   . ^ E9 6D77FFFF   jmp     01401A31  
```

这样，只破解了文件完整性检查（自效验），并不破解文件名效验，程序就能顺利执行。

破解未完成，望高人指点迷津。。。

--------------------------------------------------------------------------------

【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

2007 年 03 月 02 日 上午 03:08:10

引用: *最初由** KiD **发布*

还是想办法证明一下，呵呵：

[`iedimem.bokee.com/photo/view....6051852&mode=3`](http://iedimem.bokee.com/photo/view.fcgi?id=6051852&mode=3)

图片被反色处理了

........

你给的图片是客户端的吧？我是说服务端。客户端以前我就弄过了。服务端的以前也曾看过，改过后启动服务时总是超时错误，我对比过原版和我修改后的版本，前面一切都是正常的，当调用一个启动服务的 API 函数时，传入的参数及堆栈数据都是相同的，但修改过的就是会启动服务超时，估计可能是驱动中还有猫腻。因为没装 `SoftICE`，机器又比较老，用 `Windbg` 加虚拟机调试的话又太慢，就扔在那了。我看了你的修改方法，和我以前写的 `3.0` 客户端去校验的方法是一样的，应该直接用在服务端上是不行的。你说你那修改成功，难道是我的机器有问题？

我测试了一下服务端，的确还是存在问题，但不像是你描述滴那种。。。

我这里的表现是：我把 `rserver3.exe` 的程序代码做一个 `code cave` 改过之后，服务端能够正常运行各种功能，说明这样修改是可以的，并没有看到启动服务的时候超时错误的提示。但是我尝试加 `Aspack` 壳或者修改资源过后，再启动 `rserver3.exe` 过一会儿不弹出任何提示就退出了。由于跟踪过程遇到的异常太多，跟不下去，暂时还不知道怎样处理。感觉这个东东一调试就是很容易出现异常，诡异。。。我没有经验，还望各位大大帮忙。。。

`4dd466d0e1e0cc123afbe504da2462ca`
