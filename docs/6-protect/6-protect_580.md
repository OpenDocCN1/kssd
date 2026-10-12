
# 我也来理解虚拟机

> 原文：[`www.pediy.com/kssd/pediy10/60436.html`](https://www.pediy.com/kssd/pediy10/60436.html)

现在的技术更新换代真是快哦，好多壳都还没有脱成功，就出现 `VM` 了。本来还不打算接触 `VM` 的，![](img/9d43c9aee695f95b024aafd1b69ffa5a.jpg "伤心") ![](img/9d43c9aee695f95b024aafd1b69ffa5a.jpg "伤心") 但是被某某鄙视不动手的懒人，所以今天就动手给他瞧瞧。不过话说回来，我确实有点懒，因为我把大部分时间都花在泡 MM、游戏，跟发呆上（我经常发呆冥想自己中了大奖，有个美女跟我搭讪什么的），做今天这样的事情完全是凭心血来潮。

看了些讨论虚拟机的帖子，“每个虚拟机都有自己的 `CONTEXT`”，“会在伪指令中修改 `EIP`”。

我纳闷了好久，按我目前的理解，我就知道有一个 `SetThreadContext` 可以修改 `CONTEXT` 结构，但是用 `OD` 打开 `VM` 过的程序，楞是没看到呀？？

入口参数：

```
00401166 > $ 68 514E4000 push 00404E51 //压入 PCODE 首地址
0040116B .- E9 4F3A0000 jmp 00404BBF
```

`VM` 初始化：

```
00404BBF 9C pushfd //压入 32 位的 EFLAGS
00404BC0 60 pushad //压入 eax, ecx, edx, ebx, esp, ebp, esi, edi
00404BC1 68 00000000 push 0 //我把他看作压入一个 DWORD 变量
00404BC6 8B7424 28 mov esi, dword ptr [esp+28] //[esp+28]=00404E51
00404BCA BF 00404000 mov edi, 00404000 //edi=CONTEXT 结构
00404BCF FC cld
00404BD0 89F3 mov ebx, esi
00404BD2 033424 add esi, dword ptr [esp]
00000000(变量) |
edi |
esi | ⌒
ebp | 堆
esp | 栈
ebx | ↓
edx |
ecx |
eax |
flags |
00404e51 |
mov edi, 00404000
```

对比了一些帖子，我也敢说 `00404000` 指向 `CONTEXT` 结构，刚开始的时候我觉得这个地址就是 `VMP` 区段的开头，没什么特别的。

`CTRL+G` 到 `00404000` 看看：

```
00404000 F5 cmc
00404001 808B 1C99DCE8 E2 or byte ptr [ebx+E8DC991C], 0E2
00404008 94 xchg eax, esp
00404009 3386 C61FD899 xor eax, dword ptr [esi+99D81FC6]
0040400F 8136 24CFFA5B xor dword ptr [esi], 5BFACF24
00404015 7E 68 jle short 0040407F
00404017 48 dec eax
00404018 2B8C14 DDA08B9B sub ecx, dword ptr [esp+edx+9B8BA0DD]
0040401F 6E outs dx, byte ptr es:[edi]
00404020 F605 6B6A8A6E 80 test byte ptr [6E8A6A6B], 80
00404027 3246 52 xor al, byte ptr [esi+52]
0040402A 4D dec ebp
0040402B 55 push ebp
0040402C 55 push ebp
0040402D 1D 94803BB9 sbb eax, B93B8094
00404032 5D pop ebp
00404033 F693 7570A7FE not byte ptr [ebx+FEA77075]
00404039 F5 cmc
0040403A 04 4E add al, 4E
0040403C 2130 and dword ptr [eax], esi
0040403E 8C2459 mov word ptr [ecx+ebx*2], fs
00404041 0F23C1 mov dr0, ecx //这里看起来想一个伪指令的开始呢
00404044 E9 8C0B0000 jmp 00404BD5
```

那么 `00404000--00404041` 之间的就是 `CONTEXT` 结构，才 `41h` 大小呀？？远远比一个完整的 `CONTEXT` 结构要小很多呢！！

现在按我的理解这个 `CONTEXT` 结构指向的是一个内存地址，不是真实的寄存器。既然是内存的一块，要么是作者定义的一个对象，要么是申请的一块内存，

这里更象前者。那这样看来，论坛里讨论虚拟机帖子里给出的那些 `CONTEXT` 结构中的 `EAX`，`EDX`，`EIP`，`EFLAGS` 都不是真实的寄存器，只是一个变量，一个符号而已！？

接下来我也来到了 `VM` 指令解释器：

`VM` 指令解释器：

```
00404BD5 8A0E mov cl, byte ptr [esi]
00404BD7 00D9 add cl, bl
00404BD9 FEC1 inc cl
00404BDB F6D1 not cl
00404BDD 80E9 47 sub cl, 47 //对 esi 做一系列变换
00404BE0 C0C1 07 rol cl, 7
00404BE3 46 inc esi //esi++
00404BE4 80F1 C5 xor cl, 0C5
00404BE7 80E9 CD sub cl, 0CD
00404BEA C0C1 05 rol cl, 5
00404BED 00CB add bl, cl
00404BEF 0FB6C1 movzx eax, cl //eax 在这里作为指针
00404BF2 8D1485 48424000 lea edx, dword ptr [eax*4+404248] //00404248 就是那张大的表的起始地址
00404BF9 FF22 jmp dword ptr [edx] //jmp VM_Op_Code
```

`ESI` 指向 `PCODE`，地址位于 `VMP` 区段，这些 `PCODE` 是怎么来的呢？有没有什么规律？我现在也不清楚，但是对比了一下程序，我至少知道了它不是从原始程序中来的，

是作者自己安排的。原来的代码还好好的在那躺着呢！（刚开始的时候我认为虚拟机是把原始代指令化后保存在文件中，执行的时候再把这些指令还原，并跑到“虚拟机”中执行指令）

----“上面就是 `VM` 的主循环.”

~！#￥%￥%

前面突然停电了，来电后我用 `VMP` 再一次 `VM` 了程序，原来的就被覆盖掉了，真不好意思，下面的是新的程序，所以地址跟某些代码有些变化，但大致结构还是差不多的。`VMP` 我还不大会用，每次加的程序运行总会弹出个非法对话框。

接下来这段代码执行了很多次了，就是把堆栈顶端的数据弹到`[EDI+EAX*4]`这个地址中去。今天我突然觉得这段代码应该是设置 `CONTEXT` 结构的，原来我以为这里也是一个伪指令。

```
00404076 AC lods byte ptr [esi]
00404077 00D8 add al, bl
00404079 34 F9 xor al, 0F9
0040407B 2C 70 sub al, 70
0040407D C0C0 05 rol al, 5
00404080 2C 0C sub al, 0C
00404082 C0C0 06 rol al, 6
00404085 00C3 add bl, al
00404087 8F0487 pop dword ptr [edi+eax*4]
0040408A ^ E9 87FFFFFF jmp 00404016
```

`EDI` 指向 `CONTEXT` 结构的呢，`eax` 做为指针。代码把 0，`edi`，`esi`，`ebp`---`eflags` 弹到了 `CONTEXT` 结构这个地址，这样就设置了一个 `CONTEXT` 结构。例如把 `EAX` 弹到了 `XXXX` 这个地址，就可以把这个地址看成是 `EAX`，应该是这样吧。

接下来应该是执行伪指令了吧？？？？我见大虾们都给它们起了一个很好听的名字．

```
004045B5 58 pop eax
004045B6 ^ E9 5BFAFFFF jmp 00404016
```

把这里命名为 `p_popeax` 怎么样？？？

```
004042E3 AC lods byte ptr [esi]
004042E4 00D8 add al, bl
004042E6 34 F9 xor al, 0F9
004042E8 2C 70 sub al, 70
004042EA C0C0 05 rol al, 5
004042ED 2C 0C sub al, 0C
004042EF C0C0 06 rol al, 6
004042F2 00C3 add bl, al
004042F4 FF3487 push dword ptr [edi+eax*4]
004042F7 ^ E9 1AFDFFFF jmp 00404016

00404228 AD lods dword ptr [esi]
00404229 01D8 add eax, ebx
0040422B 05 972C1742 add eax, 42172C97
00404230 35 C476D6BE xor eax, BED676C4
00404235 05 43E5412D add eax, 2D41E543
0040423A C1C0 10 rol eax, 10
0040423D 01C3 add ebx, eax
0040423F 50 push eax
00404240 ^ E9 D1FDFFFF jmp 00404016
```

跟到这里就懒的跟下去拉。。。。如果说上面的都是伪指令，却没有看到修改 `EIP` 的，我只看到了 `VM` 解释器里的 `inc esi`（有些是 `lods byte ptr [esi]`).

意思是说我只看到程序在跑自己的 `PCODE`，跑完了就到原始的程序接着跑，当然我可没有跟到原始程序，我心疼我的 `F7` 跟 `F8`。

目前我对虚拟机的理解就到这里了，也不知道对不对。

前些天我问了一个外星人：

“我觉得 `VM` 过的程序只是多了一个 `VMP` 区段，把这个区段删掉，再修复 `PE` 头里的 `section`，`sizeofimage`，不就可以了吗？”

“还要修复 `VM`！”

“什么是修复 `VM` 呀？”

“就是还原成 `80X86` 指令！”

听到这样的答复我立即跑了，太深奥了嘛~！

今天我又拿程序出来对比了下，发现除了多出这个 `VMP` 区段以外，还有有一处不一样：

原始程序的：

```
00401166 >/$ E8 1EFFFFFF call 00401089
0040116B |. 6A 00 push 0 ; /ExitCode = 0
0040116D \\. E8 5A000000 call <jmp.&kernel32.ExitProcess> ; \\ExitProcess
00401172 $- FF25 48204000 jmp dword ptr [<&user32.BeginPaint>] ; user32.BeginPaint
```

`VM` 过的：

```
00401166 > $ 68 514E4000 push 00404E51 //压入 PCODE 首地址
0040116B .- E9 4F3A0000 jmp 00404BBF
```

我这里只有一处不一样，我不知道 `VM` 过的程序是否有多处，我把原始程序的二进制代码替换掉 `VM` 过的，保存后不要 `VMP` 区段也可以运行。难道这样就是修复 `VM`？？

如果是的话那可就麻烦大了，如何得知原来的代码是什么呢，`CTRL+B` 搜索不到，疑惑中。。。。。

总结一下今天的理解:

①: `VM` 里有自己的 `context` 结构,按我的理解我没有把这个结构与真实的 `context` 结构联系起来
②: `pcode` 是作者自己定义的,目前我还没有发现这些 `pcode` 跟伪指令有什么联系
③: 程序在跑自己的 `pcode`，似乎什么都没有做!

`2008-2-29` 下午 `1:32:06`

小娃崽
