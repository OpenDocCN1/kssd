
# [原创]Remote Administrator 2.2 服务器端去自效验+文件名效验

- 【文章标题】: Remote Administrator 2.2 服务器端去自效验+文件名效验
- 【文章作者】: KiD
- 【作者邮箱】: iiedii@163.com
- 【软件名称】: Remote Administrator 2.2
- 【下载地址】: [288K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3S2S2L8Y4A6A6k6Y4W2Q4x3X3g2G2M7X3N6Q4x3V1k6A6L8X3c8W2P5q4)9J5k6i4m8Z5M7q4)9K6c8V1N6G2i4K6y4p5f1$3S2G2N6#2)9K6b7g2)9K6b7f1I4A6M7%4c8Q4x3U0k6u0c8q4)9K6c8o6f1^5z5o6M7`.](https://bbs.kanxue.com/elink@fb8K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3S2S2L8Y4A6A6k6Y4W2Q4x3X3g2G2M7X3N6Q4x3V1k6A6L8X3c8W2P5q4)9J5k6i4m8Z5M7q4)9K6c8V1N6G2i4K6y4p5f1$3S2G2N6#2)9K6b7g2)9K6b7f1I4A6M7%4c8Q4x3U0k6u0c8q4)9K6c8o6f1^5z5o6M7`.)
- 【加壳方式】: 无壳
- 【编写语言】: Microsoft Visual C++ 6.0
- 【使用工具】: OD, PEiD, HIEW
- 【软件介绍】: 一个效率和安全性都非常高的远程控制软件
- 【作者声明】: 我是新菜，对破解只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

---

## 详细过程

Remote Administrator 2.2是Famatech公司1994年出品的远程控制经典产品了。貌似从1994年的推出的这一个版本开始，Famatech就着手自效验保护，所以服务器端一直没法汉化。估计现在它推出的新的版本保护雷同，就看2.2的算了。

### 情况初探

`r_server.exe`是服务器端的主程序，不加壳看来是该公司的惯常伎俩，给它加个壳如何？我用ASPR加之，很快得到了答复“Executable file is corrupted”，这还算友好的。更可恶的是修改了文件名之后，比如我把文件名改为`r_server_m.exe`，再运行就无提示自动退出了。

### 保护分析

它的自效验保护并不像传统软件的自效验那样，改个跳转了事。它把整个自效验过程连同其它的一些必要过程压缩存放，在运行的时候才当场解压出这段代码。这么做的好处是：你还没法直接修改这段代码，因为原始文件里面是密文存放的；你也没法屏蔽掉这段代码，因为它还包含一些软件运行必须的过程。整个保护有点像融入了一点壳的手段。

### 破解分析

对付这种保护，先从理论上入手，我们等它刚刚把这些代码解压完毕时，修改掉那些关键跳转不就了事拉。。。

下面从实道来

---

这里大胆地不设任何断点，从程序入口开始顺序跟踪。。。嘿嘿，这样顺序地跟着程序走，对我们这些新菜来说思路会比较清晰

先看看平平常常的程序入口：

```
01406FDB >/$  55            push    ebp
01406FDC  |.  8BEC          mov     ebp, esp
01406FDE  |.  6A FF         push    -1
01406FE0  |.  68 B8EE4001   push    0140EEB8
01406FE5  |.  68 E86D4001   push    01406DE8                          ;  SE 处理程序安装
```

注意地址都是`0140XXXX`，因为后面会出现微妙的变化。

单步执行不久，程序走到：

```
01407097  |>  50            push    eax
01407098  |.  FF75 9C       push    dword ptr [ebp-64]
0140709B  |.  56            push    esi
0140709C  |.  56            push    esi                               ; /pModule
0140709D  |.  FF15 00B04001 call    dword ptr [<&KERNEL32.GetMod..    ; \GetModuleHandleA
014070A3  |.  50            push    eax
014070A4  |.  E8 F7A1FFFF   call    014012A0                          ;  弹出提示的call
                                                                        ;  如果修改了文件名程序就从这里直接退出
```

找到一个关键的`call`，跟进`014012A0`看看，原来还复杂着：

#### `call 014012A0` 内部

在`014012A0`中跟过了几次无关紧要的判断程序是否出错，几步路来到：

```
01401333   > \8B4D C0       mov     ecx, dword ptr [ebp-40]           ;  r_server.01417250
01401336   .  8B11          mov     edx, dword ptr [ecx]
01401338   .  8955 BC       mov     dword ptr [ebp-44], edx
0140133B   .  6A 40         push    40                                ; /Protect = PAGE_EXECUTE_READWRITE
0140133D   .  68 00100000   push    1000                              ; |AllocationType = MEM_COMMIT
01401342   .  8B45 BC       mov     eax, dword ptr [ebp-44]           ; |
01401345   .  50            push    eax                               ; |Size
01401346   .  6A 00         push    0                                 ; |Address = NULL
01401348   .  FF15 0CB04001 call    dword ptr [<&KERNEL32.VirtualA... ; \VirtualAlloc 申请分配虚拟内存
```

后面还有：

```
........   （略去几个）

01401595   .  6A 40         push    40                                ; /Protect = PAGE_EXECUTE_READWRITE
01401597   .  68 00100000   push    1000                              ; |AllocationType = MEM_COMMIT
0140159C   .  8B45 E0       mov     eax, dword ptr [ebp-20]           ; |
0140159F   .  6BC0 18       imul    eax, eax, 18                      ; |
014015A2   .  8B4D A8       mov     ecx, dword ptr [ebp-58]           ; |
014015A5   .  8B5401 0C     mov     edx, dword ptr [ecx+eax+C]        ; |
014015A9   .  52            push    edx                               ; |Size
014015AA   .  8B45 E0       mov     eax, dword ptr [ebp-20]           ; |
014015AD   .  6BC0 18       imul    eax, eax, 18                      ; |
014015B0   .  8B4D A8       mov     ecx, dword ptr [ebp-58]           ; |
014015B3   .  8B15 08284101 mov     edx, dword ptr [1412808]          ; |
014015B9   .  035401 08     add     edx, dword ptr [ecx+eax+8]        ; |
014015BD   .  52            push    edx                               ; |Address
014015BE   .  FF15 0CB04001 call    dword ptr [<&KERNEL32.VirtualA... ; \VirtualAlloc 申请分配虚拟内存
```

哈哈，竟然找到一堆`VirtualAlloc`，看来它企图请求分配虚拟内存，然后把一些代码写到这段内存里面。顺便查查资料：在Win32系统中，调用`VirtualAlloc`函数分配虚拟内存，申请成功后用`VirtualFree`可以释放掉。

穿插在这一堆`VirtualAlloc`之间的是大量的循环，诸如：

```
0140165C   > \8B45 E0       mov     eax, dword ptr [ebp-20]
0140165F   .  3B45 B4       cmp     eax, dword ptr [ebp-4C]
01401662   .  0F83 99020000 jnb     01401901
01401668   .  C785 A8FEFFFF>mov     dword ptr [ebp-158], 0
01401672   .  C785 ACFEFFFF>mov     dword ptr [ebp-154], 0
0140167C   .  C785 ACFAFFFF>mov     dword ptr [ebp-554], 0
01401686   .  8B4D E0       mov     ecx, dword ptr [ebp-20]
01401689   .  69C9 07010000 imul    ecx, ecx, 107
0140168F   .  8B55 B0       mov     edx, dword ptr [ebp-50]
01401692   .  A1 08284101   mov     eax, dword ptr [1412808]
01401697   .  03840A FF0000>add     eax, dword ptr [edx+ecx+FF]
0140169E   .  8985 B4FAFFFF mov     dword ptr [ebp-54C], eax
014016A4   .  8B4D E0       mov     ecx, dword ptr [ebp-20]
014016A7   .  69C9 07010000 imul    ecx, ecx, 107
014016AD   .  8B55 B0       mov     edx, dword ptr [ebp-50]
014016B0   .  03D1          add     edx, ecx
014016B2   .  8D8D B8FAFFFF lea     ecx, dword ptr [ebp-548]
014016B8   .  E8 D3040000   call    01401B90
014016BD   .  8B55 E0       mov     edx, dword ptr [ebp-20]
014016C0   .  69D2 07010000 imul    edx, edx, 107
014016C6   .  8B45 B0       mov     eax, dword ptr [ebp-50]
014016C9   .  8B8C10 030100>mov     ecx, dword ptr [eax+edx+103]
014016D0   .  898D ACFAFFFF mov     dword ptr [ebp-554], ecx
014016D6   >  C785 B0FAFFFF>mov     dword ptr [ebp-550], 0
014016E0   .  8D8D B8FAFFFF lea     ecx, dword ptr [ebp-548]
014016E6   .  E8 25040000   call    01401B10
014016EB   .  83E8 01       sub     eax, 1
014016EE   .  8985 A4FEFFFF mov     dword ptr [ebp-15C], eax
014016F4   .  EB 0F         jmp     short 01401705
014016F6   >  8B95 A4FEFFFF mov     edx, dword ptr [ebp-15C]
014016FC   .  83EA 01       sub     edx, 1
014016FF   .  8995 A4FEFFFF mov     dword ptr [ebp-15C], edx
01401705   >  83BD A4FEFFFF>cmp     dword ptr [ebp-15C], 0
0140170C   .  7E 17         jle     short 01401725
0140170E   .  8B85 A4FEFFFF mov     eax, dword ptr [ebp-15C]
01401714   .  0FBE8C05 B8FA>movsx   ecx, byte ptr [ebp+eax-548]
0140171C   .  83F9 2E       cmp     ecx, 2E
0140171F   .  75 02         jnz     short 01401723
01401721   .  EB 02         jmp     short 01401725
01401723   >^ EB D1         jmp     short 014016F6
```

懒得管它们，反正大致是不厌其烦地将代码解压到申请的内存空间。。。这一过程会持续很久，单步显然是受不了了。向后看，注意到`014019EB`处，整个解压过程结束。

```
014019EB   > \8B45 E0       mov     eax, dword ptr [ebp-20]           ; 代码解压过程完成，在这里设断
014019EE   .  3B45 AC       cmp     eax, dword ptr [ebp-54]
014019F1   .  75 1B         jnz     short 01401A0E
014019F3   .  6A 00         push    0                                 ; /Style = MB_OK|MB_APPLMODAL
014019F5   .  68 10284101   push    01412810                          ; |Title = ""
014019FA   .  68 30004101   push    01410030                          ; |Text = "Err7"
014019FF   .  6A 00         push    0                                 ; |hOwner = NULL
01401A01   .  FF15 A0B04001 call    dword ptr [<&USER32.MessageBoxA>] ; \MessageBoxA
01401A07   .  B8 01000000   mov     eax, 1
01401A0C   .  EB 2A         jmp     short 01401A38
```
