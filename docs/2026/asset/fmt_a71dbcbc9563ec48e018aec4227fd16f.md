
# [分享]Asprotect SKE 2.2 的Advanced Import protection保护技术

- 【文章标题】: Asprotect SKE 2.2 的Advanced Import protection保护技术浅析
- 【文章作者】: kanxue
- 【作者主页】: 11fK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4m8W2k6r3W2&6i4K6u0W2j5$3!0E0
- 【软件名称】: Asprotect SKE 2.2 build 03.19 Release 加壳的目标软件

---

首先感谢shoooo的帮助！春节前那段时间开始学习Asprotect SKE 2.x脱壳技术，由于长时间没接触Asprotect，感觉很是吃力，多亏shoooo无私地将他的许多经验技巧相授，这样才得以掌握这款猛壳的一些脱壳技术。

本篇文章以Asprotect SKE 2.2 build 03.19 Release加壳的目标为讲解实例（一个Microsoft Visual C++ 6.0编译的程序），压缩选项只选了一项“Advanced Import protection”，Asprotect保护的输入表情况多种，本文仅与大家探讨Advanced Import protection部分，其它更完整的输入表处理类型参考shoooo撰写的Asprotect SKE经典文章：[nspack3.5主程序脱壳分析(Aspr SKE 2.X)](http://bbs.pediy.com/showthread.php?s=&threadid=19313)，本文只是将shoooo这篇的文章提到的东西转换成实际操作过程而已。

脱Asprotect SKE 2.x壳会遇到如下情况：

1. Emulate standard system functions（易）
2. Advanced Import protection（中）
3. stolen OEP（脱Asprotect难点就在这，体力＋毅力＋技巧＋时间）
4. SDK（脱Asprotect难点就在这，体力＋毅力＋技巧＋时间）
5. 初始化表与库函数（Delphi程序）（易）

## 1.什么是Advanced Import protection

先来看一段加壳的代码，请注意API函数的调用部分：

```
00401EC5   FF7424 08             push    dword ptr [esp+8]
00401EC9   FF15 3C504000         call    [<&KERNEL32.GetCurrentProcess>]  //这里，即call    [40503C]
00401ECF   50                    push    eax
00401ED0   FF15 38504000         call    [<&KERNEL32.TerminateProcess>]   //这里，即call    [405038]
00401ED6   837C24 0C 00          cmp     dword ptr [esp+C], 0
```

用Advanced Import protection保护后的代码，其API调用转换成了`call 01200000`形式：

```
00401EC5   FF7424 08             push    dword ptr [esp+8]
00401EC9   E8 32E1DF00           call    01200000              //这里，调用KERNEL32.GetCurrentProcess
00401ECE   AA                    stos    byte ptr es:[edi]
00401ECF   50                    push    eax
00401ED0   E8 2BE1DF00           call    01200000              //这里，调用KERNEL32.TerminateProcess
00401ED5   CA                    db      CA
```

以下这段文字来自shoooo的文章（稍有改动）：

> *最初由 **shoooo** 发布*
>
> 出处：http://bbs.pediy.com/showthread.php?s=&threadid=19313
>
> `01200000`什么呢？
> 它是把导入函数调用的变形，原来的`call [IAT]` 和 `jmp [IAT]`的变形
> `01200000`壳用`VirtualAlloc`分配的空间，不在区段中
> 在我的机子上现在是`call 01200000`，在你的机子上就可能是`call 1230000`
> 也就是说，`call 01200000`是壳经过计算后写入的
> 于是我想看看，在它写入`call 01200000`前是什么样子
>
> OD载入实例程序，清除所有断点，
> 对`0401ECA`下内存写入断点 (因为`00401EC9` 是`E8`，我们要的是后4个字节)
>
> 若干次后会断在这里
>
> ```
> 00AA6EBF 8945 00 mov [ebp], eax // 断在这儿：ebp指向00401ECA，eax写入后，使那个地方变成call 01200000
> 00AA6EC2 6A 0A push 0A
> 00AA6EC4 E8 D3E7FDFF call 00A8569C
> 00AA6EC9 8BC8 mov ecx, eax
> 00AA6ECB 038B E4000000 add ecx, [ebx+E4]
> 00AA6ED1 8BD6 mov edx, esi
> 00AA6ED3 8BC3 mov eax, ebx
> 00AA6ED5 E8 AAE6FFFF call 00AA5584
> 00AA6EDA FF0C24 dec dword ptr [esp]
> 00AA6EDD 03B3 E4000000 add esi, [ebx+E4]
> 00AA6EE3 833C24 00 cmp dword ptr [esp], 0
> 00AA6EE7 ^ 0F87 55FEFFFF ja 00AA6D42 //如果还有需要处理就跳上去
> 00AA6EED 53 push ebx
> 00AA6EEE E8 5D000000 call 00AA6F50
> 00AA6EF3 0183 EC000000 add [ebx+EC], eax
> 00AA6EF9 B0 01 mov al, 1
> 00AA6EFB 83C4 24 add esp, 24
> 00AA6EFE 5D pop ebp
> 00AA6EFF 5F pop edi
> 00AA6F00 5E pop esi
> 00AA6F01 5B pop ebx
> 00AA6F02 C3 retn //这里结束
> ```
>
> 正如我所说，`call 01200000`完全是在代码段解码后，申请空间，现在我申请到的是`01200000`
> 那么它就将需要变形的地方计算后写成`call 01200000`，如果你申请到的是`1230000`，那么你
> 是`call 1230000`
>
> 断在这里，我当然想看一看在改写成`call 01200000`之前，那些地址是不是正常的
> 很可惜，那里在改写成`call 01200000`，本身就是乱掉的。
> 或者在某个时候能知道那些变形地址原先的真实情况，可惜我找不到。
> 也许只有作者知道在哪里
> 也许根本就找不到
> 因为根本就不需要
> 对于`call 01200000`，它加密前只要知道2件事，1.本身所在的地址 2.IAT中的位置
> 对于`call 01200000`，现在也只要知道2件事，1.本身所在的地址 2.最后要去的导入函数的地址
> 它没有理由记录IAT中的位置
> 我们要做的是找出最后到达的导入函数的地址，然后找出它在IAT中的位置
> 改成原先的`call [IAT]` 或 `jmp [IAT]`
>
> 回到正题，当我们断下时，前面可能已经处理若干个了
> 要想得到全部的表
> 你有好几种选择
> 1. 到oep后，写一段代码搜索出所有的`call 01200000`的地址
> 2. 想办法第一时间断在上面这个地方，即`00AA6EBF`，`ebp-1`就是变形的地址，保存所有的`ebp-1`
> 3. 也许内存中本身存在这张表，我没有去找，你可以找找

```
00AA6EBF    8945 00             mov     [ebp], eax
00AA6EC2    6A 0A               push    0A
00AA6EC4    E8 D3E7FDFF         call    00A8569C
00AA6EC9    8BC8                mov     ecx, eax
00AA6ECB    038B E4000000       add     ecx, [ebx+E4]
00AA6ED1    8BD6                mov     edx, esi
00AA6ED3    8BC3                mov     eax, ebx
00AA6ED5    E8 AAE6FFFF         call    00AA5584
00AA6EDA    FF0C24              dec     dword ptr [esp]
00AA6EDD    03B3 E4000000       add     esi, [ebx+E4]
00AA6EE3    833C24 00           cmp     dword ptr [esp], 0
00AA6EE7  ^ 0F87 55FEFFFF       ja      00AA6D42 //循环上去
```

在写入每一处的`call 01200000`时，上面的代码会经过这里：

```
00AA6D42    33C0                xor     eax, eax
00AA6D44    8A07                mov     al, [edi]
00AA6D46    8D0440              lea     eax, [eax+eax\*2]
00AA6D49    8B6C83 68           mov     ebp, [ebx+eax\*4+68]
00AA6D4D    8BC6                mov     eax, esi
00AA6D4F    FFD5                call    ebp
00AA6D51    8BE8                mov     ebp, eax
00AA6D53    036B 24             add     ebp, [ebx+24]
00AA6D56    03AB E0000000       add     ebp, [ebx+E0]
00AA6D5C    EB 01               jmp     short 00AA6D5F
00AA6D5E    E8 33C08A47         call    48352D96
00AA6D63    098D 04408B54       or      [ebp+548B4004], ecx
00AA6D69    8368 8B C6          sub     dword ptr [eax-75], -3A
00AA6D6D    FFD2                call    edx                       //call edx 结果在eax
00AA6D6F    807B 20 00          cmp     byte ptr [ebx+20], 0       //eax 可能是1或0
00AA6D73    0F85 3D010000       jnz     00AA6EB6
```

> *最初由 **shoooo** 发布*
>
> 如果是1，当前这个`call 01200000`处运行时，会重新回到调用地址，再进入导入函数
> 如果是0，当前这个`call 01200000`进入导入函数后出来，不过这种方式比较邪恶，它可能做更多的事情

## 2.获得地址表

这里想办法获得程序里 `call 01200000` 语句所有的地址，上面代码中 `00AA6D6D call edx` 这个点比较好，除了能得到地址表，还能得知晓 `call 01200000` 是按类型1来处理的，还是按类型2来处理的。

对 `00AA6D6D call edx` 下硬件执行断点，重新用OD加载实例程序，由于已下硬件断点，因此这次直接断在这里：

```
00AA6D6D    FFD2                call    edx                  //断在这里，此次EBP指向的就要处理的地址
00AA6D6F    807B 20 00          cmp     byte ptr [ebx+20], 0
00AA6D73    0F85 3D010000       jnz     00AA6EB6
00AA6D79    3C 01               cmp     al, 1
```

此时堆栈窗口的数据：

```
0012FEF8   0000001A   //注意这里的数据0x1A，表示26个地址要处理
0012FEFC   D6589EF1
0012FF00   9BCE5199
0012FF04   00000000
```

为了得到所有的地址表，必须写一小段补丁代码。首先用HideOD插件分配一个临时空间（`Alloc Memory`菜单命令），我这里分配得到的地址是`01240000`，在你的机子可能不是这值。

```
01240000  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
01240010  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
01240020  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
01240030  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
01240040  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
```

然后修改`00AA6D6F`这段代码如下：

```
00AA6D6D  - E9 8E927900         jmp     01240000    //跳到patch代码执行
00AA6D72    90                  nop
00AA6D73     0F85 3D010000       jnz     00AA6EB6
00AA6D79    3C 01               cmp     al, 1
```

在分配的空间里键入如下补丁代码：

```
01240000    FFD2                call    edx                //壳原来的代码
01240002    60                  pushad                     //开始补丁代码
01240003    8B1D 30002401       mov     ebx, [1240030]     //取缓存地址，我这里是用01240040这个地址
01240009    C1E0 1F             shl     eax, 1F            //将EAX的返回值0或1置高位
0124000C    03C5                add     eax, ebp           //ebp是所要的地址
0124000E    8903                mov     [ebx], eax
01240010    83C3 04             add     ebx, 4             //指向下一个缓存地址
01240013    891D 30002401       mov     [1240030], ebx
01240019    61                  popad
0124001A    807B 20 00          cmp     byte ptr [ebx+20], 0//壳原来的代码
0124001E  - E9 506D86FF         jmp     00AA6D73             //跳到壳里继续执行
```

对应的二进制代码如下：

```
FF D2 60 8B 1D 30 00 24 01 C1 E0 1F 03 C5 89 03 83 C3 04 89 1D 30 00 24 01 61 80 7B 20 00 E9 50
6D 86 FF
```

键入补丁后的数据窗口如下：

```
01240000  FF D2 60 8B 1D 30 00 24 01 C1 E0 1F 03 C5 89 03  ?`?0.$拎?
01240010  83 C3 04 89 1D 30 00 24 01 61 80 7B 20 00 E9 50  ??0.$a? .樾
01240020  6D 86 FF 00 00 00 00 00 00 00 00 00 00 00 00 00  m?.............
01240030  40 00 24 01  00 00 00 00 00 00 00 00 00 00 00 00  @.$............
```
