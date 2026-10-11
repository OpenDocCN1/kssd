
`004068ED  MOV EAX,DWORD PTR SS:[EBP+8]`
`004068F0  PUSH EAX`
`004068F1  CALL 是男人就.00405A93` # 我们从这里出来
`004068F6  ADD ESP,4` # 停在这里
`004068F9  MOV EAX,DWORD PTR SS:[EBP+8]`
`004068FC  CMP DWORD PTR DS:[EAX+1338],0`
`00406903  JE 是男人就.00406924:[EA`
`00406909  MOV EAX,DWORD PTR SS:[EBP+8]`
`0040690C  MOV EAX,DWORD PTR DS:[EAX+11A8]`
`00406912  ADD EAX,8E` # 计算声音资源的索引
`00406917  PUSH EAX`
`00406918  MOV EAX,DWORD PTR SS:[EBP+8]`
`0040691B  PUSH EAX`
`0040691C  CALL 是男人就.00406E3D` # 这就是播放声音的那个 call
`00406921  ADD ESP,8`
`00406924  JMP 是男人就.00406A12`
`00406929  MOV EAX,DWORD PTR SS:[EBP+8]`
`0040692C  CMP DWORD PTR DS:[EAX+11AC],1`

我们回到了 `C` 区的代码，注意到这个事实非常重要，这有利于我们后面对代码的修改。

最后一个疑问：`E` 区代码对弹跳力的读取是干什么的呢？我想是程序开始用牛顿定律开始计算小人能跳多高了吧——这正是我们所希望的。那么 `A` 区和 `F` 区呢，它们对弹跳力值赋 `0`，我想可能与小人从一块跳板掉到（或者跳到）另一块跳板时弹跳力要重新计数有关吧，当然如果你觉得这个规则不利于你“做男人”，也可以发挥一下，`nop` 掉这句代码。无论如何，这些同我们所要实现的弹跳力无限功能已经没什么关系了：）

现在我们已经完全明白程序的处理了，如何修改呢？要想通过修改代码（而不是添加代码）来增加功能，我们必须做一件事：对原来的代码进行优化，籍此腾出空间增加我们自己的代码。当然这两步可以同时实现，就看各位各显神通了。我的思路是这样的：`11A8` 处还是必须存放无限增加的弹跳力值（因为后面要用它计算弹跳高度），但是在索引位图和声音资源时来个“偷天换日”，把有效的索引值赋给相应的参数。

幸运的是，程序是用 `VC4` 编译的，我们要修改的那部分代码似乎效率颇低，有些寄存器根本就没有使用，还有不少重复的工作量：

`004068C3  MOV EAX,DWORD PTR SS:[EBP+8]`
`004068C6  MOV DWORD PTR DS:[EAX+11AC],1`
`004068D0  MOV EAX,DWORD PTR SS:[EBP+8]` # 一遍又一遍地读取基址，而且每回只用一次
`004068D3  MOV ECX,0C` # 这是除数，不能省略的，但不一定要在这里赋值
`004068D8  MOV EAX,DWORD PTR DS:[EAX+11A8]`
`004068DE  CDQ`
`004068DF  IDIV ECX`
`004068E1  LEA EAX,DWORD PTR DS:[EDX+1]`
`004068E4  MOV ECX,DWORD PTR SS:[EBP+8]` # 一遍又一遍地读取基址，而且每回只用一次
`004068E7  MOV DWORD PTR DS:[ECX+11A8],EAX`
`004068ED  MOV EAX,DWORD PTR SS:[EBP+8]` # 一遍又一遍地读取基址，而且每回只用一次
`004068F0  PUSH EAX` # 这里传入基址值——早就可以做了！
`004068F1  CALL 是男人就.00405A93`
`004068F6  ADD ESP,4`

程序始终没有用到 `ebx`、`esi`、`edi`，我经过确认以后决定用 `ebx` 来保存反复回零的弹跳力值，作为索引位图块和声音资源只用。`esi` 和 `edi` 保存了两个貌似地址的东东，还是不动为妙。

上面这段代码修改如下：

`004068C3  MOV EAX,DWORD PTR SS:[EBP+8]`
`004068C6  MOV DWORD PTR DS:[EAX+11AC],1`
`004068D0  MOV EAX,DWORD PTR SS:[EBP+8]`
`004068D3  PUSH EAX` # 这里就把基址作为参数传入
`004068D4  INC DWORD PTR DS:[EAX+11A8]` # 弹跳力直接递增
`004068DA  MOV EAX,DWORD PTR DS:[EAX+11A8]` # 读取弹跳力的值
`004068E0  DEC EAX`
`004068E1  MOV ECX,0C` # 除数
`004068E6  CDQ`
`004068E7  IDIV ECX` # 求余，余数送 `edx`
`004068E9  INC EDX` # 余数加 `1`
`004068EA  MOV EBX,EDX` # 送 `ebx` 保存
`004068EC  NOP`
`004068ED  NOP`
`004068EE  NOP`
`004068EF  NOP`
`004068F0  NOP` # 还空出 `5` 个字节
`004068F1  CALL 是男人就.00405A93`
`004068F6  ADD ESP,4`

实现了新功能，也把有效的索引值保存下来了。当然下面的地方也要做修改：

`004068F9  MOV EAX,DWORD PTR SS:[EBP+8]`
`004068FC  CMP DWORD PTR DS:[EAX+1338],0`
`00406903  JE 是男人就.00406924:[EA`
`00406909  MOV EAX,DWORD PTR SS:[EBP+8]`
`0040690C  MOV EAX,EBX` # 声音的索引值从 `ebx` 取得
`0040690E  NOP`
`0040690F  NOP`
`00406910  NOP`
`00406911  NOP`
`00406912  ADD EAX,8E`
`00406917  PUSH EAX`
`00406918  MOV EAX,DWORD PTR SS:[EBP+8]`
`0040691B  PUSH EAX`
`0040691C  CALL 是男人就.00406E3D` # 播放声音
`00406921  ADD ESP,8`

另一处：

`00405AB2  MOV DWORD PTR SS:[EBP-4],EAX`
`00405AB5  PUSH 0CC0020` ; /ROP = SRCCOPY
`00405ABA  MOV EAX,DWORD PTR SS:[EBP+8]` ; |
`00405ABD  MOV EAX,EBX` # 位图块的索引值从 `ebx` 取得
`00405ABF  NOP`
`00405AC0  NOP`
`00405AC1  NOP`
`00405AC2  NOP`
`00405AC3  SHL EAX,4` ; |
`00405AC6  ADD EAX,0A0` ; |
`00405ACB  PUSH EAX` ; |YSrc
`00405ACC  PUSH 0C0` ; |XSrc = C0 (192.)
`00405AD1  MOV EAX,DWORD PTR SS:[EBP+8]` ; |
`00405AD4  MOV EAX,DWORD PTR DS:[EAX+13A4]` ; |
`00405ADA  PUSH EAX` ; |hSrcDC
`00405ADB  PUSH 10` ; |Height = 10 (16.)
`00405ADD  PUSH 60` ; |Width = 60 (96.)
`00405ADF  PUSH 20` ; |YDest = 20 (32.)
`00405AE1  PUSH 58` ; |XDest = 58 (88.)
`00405AE3  MOV EAX,DWORD PTR SS:[EBP+8]` ; |
`00405AE6  MOV EAX,DWORD PTR DS:[EAX+4]` ; |
`00405AE9  PUSH EAX` ; |hDestDC
`00405AEA  CALL DWORD PTR DS:[<&GDI32.BitBlt>]` ; \BitBlt # 显示相应的位图块
`00405AF0  PUSH 0` ; /ForceBackground = FALSE

经过上面的修改，我们的 bug 就被修复了，在实际弹跳力不断增加的同时，仍然用有效的索引取得位图和声音资源，画面的显示和声音的播放与原来完全一致^O^

又是一个不眠之夜……

`Author:cyclotron[BCG][DFCG][FCG][OCN]`

`2004.1`

`4dd466d0e1e0cc123afbe504da2462ca`
