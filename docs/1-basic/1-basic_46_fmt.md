
# 很菜的问题，and esp, 0FFFFFFF8h 有什么作用？

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-194.htm`](https://www.pediy.com/kssd/pediy09/pediy09-194.htm)

> 这条指令总是跟在函数栈框架 `push ebp`、`mov ebp,esp` 之后，我搞不明白这样做有什么用，是对齐吗，还只是某个编译选项设置的？

> 修改一下，准确的讲，这条指令出现的地方，前面总是函数栈框架 `push ebp`、`mov ebp,esp` 指令。

编译器优化存放局部变量而已。VC 在可以省略的地方都会尽量省略 但是如果开了省略框架指针则省略更多而已

每个编译器使实现不一样。。你说的这个市 intel C++ 的实现方法。

`int _tmain(int argc, _TCHAR* argv[])`

```
{
0040105C  push        ebp
0040105D  mov         ebp,esp
0040105F  sub         esp,3
00401062  and         esp,0FFFFFFF8h
00401065  add         esp,4
00401068  sub         esp,38h
0040106B  mov         eax,dword ptr [__security_cookie (403000h)]
00401070  mov         dword ptr [ebp-4],eax
  wchar_t buf[16];
```

这是在 Intel C++ 10 的优化下 `and esp,FFFFFFF0` 可见这句。

可以推测是结构堆对齐

```
00401000  T>/$  55                   push ebp
00401001   |.  8BEC                  mov ebp,esp
00401003   |.  83E4 F0               and esp,FFFFFFF0
00401006   |.  83EC 20               sub esp,20
00401009   |.  6A 03                 push 3
0040100B   |.  E8 D4000000           call TEST_2.004010E4
00401010   |.  8D4424 04             lea eax,dword ptr ss:[esp+4]
00401014   |.  68 2E104000           push TEST_2.0040102E                         ; /<%d> = 40102E (4198446.)
00401019   |.  68 3C204000           push TEST_2.0040203C                         ; |Format = "%d"
0040101E   |.  50                    push eax                                     ; |s
0040101F   |.  FF15 08204000         call dword ptr ds:[<&USER32.wsprintfW>]       ; \\wsprintfW
00401025   |.  83C4 10               add esp,10
00401028   |.  33C0                  xor eax,eax
0040102A   |.  8BE5                  mov esp,ebp
0040102C   |.  5D                    pop ebp
0040102D   \.  C3                    retn
```

关闭优化下

```
0040105C  T>/$  55                   push ebp
0040105D   |.  8BEC                  mov ebp,esp
0040105F   |.  83EC 03               sub esp,3
00401062   |.  83E4 F8               and esp,FFFFFFF8
00401065   |.  83C4 04               add esp,4
00401068   |.  83EC 2C               sub esp,2C
0040106B   |.  A1 00204000           mov eax,dword ptr ds:[<&USER32.wsprintfW>]
00401070   |.  83C4 00               add esp,0
00401073   |.  8D55 E0               lea edx,[local.8]                              ; |
00401076   |.  891424                mov dword ptr ss:[esp],edx                      ; |
00401079   |.  C74424 04 00304000    mov dword ptr ss:[esp+4],TEST_2.00403000        ; |UNICODE "%d"
00401081   |.  C74424 08 00104000    mov dword ptr ss:[esp+8],TEST_2.00401000        ; |
00401089   |.  FFD0                  call eax                                        ; \\wsprintfW
0040108B   |.  83C4 0C               add esp,0C
0040108E   |.  33C0                  xor eax,eax
00401090   |.  C9                    leave
00401091   \.  C3                    retn
```

确实 Intel C++的优化达到了前所未有的高度。。令人叹为观止。

引用: *最初由 **foxabu**发布* ![查看帖子](img/showthread.php?p=331945#post331945) 每个编译器使实现不一样。。你说的这个市 intel C++ 的实现方法。

...

哈哈，结构对齐，有些明白了，我推测前面的 `push ebp`、`mov ebp,esp` 应该是为了用 `ebp` 对函数的参数进行寻址，后面的 `and esp,FFFFFFF8h` 应该为了对栈上的临时结构变量进行 8byte 对齐而采取的措施，如果要 16byte 对齐，就会象你的第 2 段代码那样是 `and esp,FFFFFFF0h`。也就是说如果编译器已经推测到栈上的临时变量可以进行 8 或 16byte 对齐寻址，在打开某个优化开关的情况下，应该会生产这样的代码的。

另外，我看的代码应该是 VC 的，可能我们还不知道 VC 的那个优化选项决定这个东西。

很佩服 foxabu 大侠能对不同编译器分析的这么透彻，有点象 K8 的那本书了，谢谢指教！

引用: *最初由 **justlovemm**发布* ![查看帖子](img/showthread.php?p=332001#post332001)

哈哈，结构对齐，有些明白了，我推测前面的 `push ebp`、`mov ebp,esp` 应该是为了用 ebp 对函数的参数进行寻址，后面的 `and esp,FFFFFFF8h` 应该为了对栈 pediy09-194.htm 上的临时结构变量进行 8byte 对齐而采取的措施，如果要 16byte 对齐，就会象你的第 2 段代码那样是 a...

‘如果要 16byte 对齐，就会象你的第 2 段代码 不是要，而是打开速度优化的前提下’

如果用第一种未优化 会花费三条指令。 而第二种显然只要一条。空间换时间。不过也谈不上 反正堆栈只要不溢出。内存也不会浪费。leave ret 以后反正还要恢复原始 esp 的。

另外纠正一个说法。准确地说你说的 VC 只是代码 是 C++的或者说 VC++的 只有编译器，只要符合 C++标准的都可以编译嘛。所以你这个代码是有可能是 Intel C++ 编译器编译的。VC 中基本上还看不到这种（不排除我见得太少。希望多指教），根据你说的每个函数 来推测。

关于编译器的问题，我可以确定是 VC 编译的，因为 IDA 已经识别出足够多的 MFC 和 VCrtl 函数。

不过关于临时变量存取的优化的方法，由于我看的代码在 `sub esp,xxx` 以后就被 VMProtect 掉了，所以也没有看出什么来。我只是猜想也许在对齐的情况下，对临时变量可能是如下访问的：

```
lea Exx,[esp+n*Exx]
push Exx
```

也许这样做比 `push [ebp+xxx]` 要快一些吧？

不过这都是瞎猜的，还请大家多指教。

引用: *最初由 **justlovemm**发布* ![查看帖子](img/showthread.php?p=332034#post332034) 关于编译器的问题，我可以确定是 VC 编译的，因为 IDA 已经识别出足够多的 MFC 和 VCrtl 函数。

不过关于临时变量存取的优化的方法，由于我看的代码在 sub esp,xxx 以后就被 VMProtect

掉了，所以也没有看出什么来。我只是猜想也许在对齐的情况下，对临时变量可能是如下访问的：

```
lea E...
```

,,,,,晕.....

MFC 的是可以用 Intel C++ 甚至 Borland C++ 编译的！！

你把同样的 VC 代码 原封不动 直接交给 Intel C++编译器编译就行了。再说库是微软编写好的静态或者动态库，Runtime Package 段是 VC++的也非常正常。。

\---------------------------------------------------------------------------

奥 建议楼主补充一些知识。MFC 就代表是 VC 的？难道 Borland C++不可以用？。。。如有得罪之处 多多包涵。我这个人比较偏激。

引用: *最初由 **foxabu**发布* ![查看帖子](img/showthread.php?p=332072#post332072) ,,,,,晕.....

MFC 的是可以用 Intel C++ 甚至 Borland C++ 编译的！！

你把同样的 VC 代码 原封不动 直接交绮..

多谢指导，没有感觉到有偏激的问题，欢迎多对我拍砖头，呵呵

不过我觉得我们说的是一个问题的两个方向，foxabu 是通过特征代码来确定编译器的，而我是通过其他的一些方法来确认编译器，然后确认这个编译器也有类似的特征代码。我明白 VC 的代码可以使用其他的编译器来编译，不过我之所以觉得是 VC 编译的，除了 IDA 识别到的函数以外，IDA 自身也自动识别这个程序是 VC 编译(我看的这个 EXE 只用了 VMProtect 保护了很小一部分的关键代码，没有再加其他壳，所以应该很容易识别的），同时我看到一些 SEH 函数，我认为对 SEH 的处理应该能表明编译器的特征。我没有看过 intel C++编译的代码，但我现在看到的 SEH 代码和以前看过的 VC 编译出来的是很相似的。

请多指正。

以下是 Hacker Disassembling Uncovered 中的原文:

**Alignment in the stack** In some cases, elements of a structure, an array, or even particular variables must be aligned by addresses that are multiples of a specific power of 2. However, the stack-top pointer value is not defined beforehand. How can the compiler, which does not know the index value, fulfill this requirement? It simply discards the lower bits of ESP.

The lower bit of even numbers is zero. To ensure that the value of the stack-top pointer is divisible by two without a remainder, simply force its lower bit to zero. If two lower bits are set to zero, the resulting value will be a multiple of four; if three lower bits are set to zero, the resulting value will be a multiple of eight; and so on.

In most cases, bits are reset using the AND instruction. For example, `AND ESP, FFFFFF0` makes ESP a multiple of 16. How do we obtain this? Let's convert `0xFFFFFFF0` to a binary form, which will give the following: 111111111 11111111 11110000. The four trailing zeroes mean that four lower bits of any number will be masked. The number will be divisible by 2 to the power of 4, which equals 16.

引用: *最初由 **justlovemm**发布* ![查看帖子](img/showthread.php?p=332320#post332320) 多谢指导，没有感觉到有偏激的问题，欢迎多对我拍砖头，呵呵

不过我觉得我们说的是一个问题的两个方向，foxabu 是通过特征代码来确定编译器的，而我是通过其他的一些方法来确认编译器，然后确认这个编译器也有类似的特征代码。我明白 VC 的代码可以使用其他的编译器来编译，不过我之所以觉得是 VC 编译的，除了...

我都说了 SEH 使用的库完全有可能是由 VC 编译好的

绝大多数都是这种情况

一般没有人去编译 MSVCRT.lib 吧？很显然 SEH 结构化处理 包括 try catch 的编译器 thunk 都是 VC 的 这个一点也不奇怪。

另外我的判断根据是：公开的 VC++编译器 至少 6 7 8 没有发现这种优化。虽然说我不一定说就是 VC。但是很显然不是 Borland C++ 至于是不是 G++或者 GCC 这个就不清楚（但是貌似无法编译 MFC 的程序，有些是 Microsoft Special 的代码。）。反正主流编译器就这么几种。不排除作者自己写编译器。而且我给出的示例代码也证明了 Intel C++ 可以生成。我只说普遍性问题 仅根据楼主的 每个函数头前面生成。这几个字。

况且 IDA 什么时候可以识别 Intel C++？，不能够识别，就无法推断就是 VC！况且那个东西也能够使人相信。。。。。

推荐楼主自己试验，把能够编译 MFC 的且符合你所说的程序环境的编译器都拿来测试就知道了。
