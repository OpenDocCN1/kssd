
# MFC 的逆向工程 - 给 SPYXX 添加新功能

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-428.htm`](https://www.pediy.com/kssd/pediy09/pediy09-428.htm)

> 【文章标题】: MFC 的逆向工程 - 给 SPYXX.exe 添加新功能

> 【文章作者】: Suyana

> 【作者邮箱】: Suyasha@163.com

> 【作者 QQ 号】: 517949855(请注明来自看雪论坛)

> 【软件名称】: VC++6.0 自带(`SPYXX.exe`)

> 【编写语言】: `VC++`, `MFC`

> 【使用工具】: `OllyDBG`, `ResHacker`

> 【操作平台】: `WinXP`

> 【软件介绍】: VC++6.0 自带的工具

> 【作者声明】: 本文原创于 Suyana, 转载请注明作者并保持文章的完整, 谢谢!

> ----------------------------------------------------------------------------

> 【详细过程】

> 1. 用 `VC++6.0` 或 `ResHacker` 修改程序资源(对话框->200)，添加四个按钮。偶的四个按钮的 ID 是 1000,1002,1003,1004。程序界面比较挤，偶把它汉化后修改了大小，重新调整位置。如图 1

> ![](img/1d1412a02a93981ca3779b5ee4121522.jpg)

> 2. 分析一下，实现这四个功能需要两条函数：`EnableWindow`、`ShowWindow`。程序里都有，就不用手工添加了，还要获得要操作的窗口的句柄。用资源修改器查看一下程序显示窗口句柄的控件的 ID 是 201(0c9h)，所以在 `OD` 中"查找"->"所有命令"，输入`push 0c9`。来到：

> 代码:

```
    0042BC88        mov     edx, [edi+220]          ; 这里保存的就是目标窗口的句柄因地址是变化的，所以要保存起来    
    0042BC8E        push    edx                     ; 修改成 jmp 0044B0B8    
    0042BC8F        call    00427810    
    0042BC94        add     esp, 4    
    0042BC97        mov     ecx, esi    
    0042BC99        push    eax    
    0042BC9A        push    0C9    
    0042BC9F        call    <jmp.&MFC42.#5953_CWnd::SetDlgItemTex>  
  在 0044B0B8 处添加偶门的代码：    
    0044B0B8        mov     [46CFF0], edx          ; 保存到这个地方    
    0044B0BE        push    edx                     ; 被覆盖的代码    
    0044B0BF        call    00427810                ; 被覆盖的代码    
    0044B0C4        jmp     0042BC94                ; 继续执行
```

> 3. `MFC` 工程中要到消息的处理过程是很麻烦的，有兴趣的话去一下看雪吧，那里有讲。偶是替换转到 `mfc42.dll` 的跳转实现的。

> 方法 A: 用 `MFC SPY` 找到 `OnCommand` 的地址。偶也不懂怎么说，看图 2

> ![](img/adcd24865f79e6c75376b22c21d2f028.jpg)

> 方法 B: 用 `OllyDbg`，用导入库处理，重新载入，这样 `mfc` 函数名才会显示出来。随便找到 1 个 `mfc` 函数，按回车，就来到末尾一大片 `jmp ....` 的地方，找到 `OnCommand`

> 按上面的方法，偶找到的是：

> `00447F0A        jmp     [<&MFC42.#4441_CWnd::OnCommand>]      ;MFC42.#4441_CWnd::OnCommand`

> 在程序末尾找到空白的地方 0044B03E，把上面代码改成 `jmp 0044B03E`

> `CWnd::OnCommand` 只有一个参数，就是控件的 ID，只要判断是不是偶的按钮的 ID 就可以了

> 4. 在 0044B03E 处添加偶门的代码:

> 代码:

```
    0044B03E        push    ebp    
    0044B03F        mov     ebp, esp    
    0044B041        pushad    
    0044B042        mov     eax, [ebp+8]            ; 取第一个参数    
    0044B045        cmp     eax, 3E8                ; 是不是 ID=1000 的按钮 Enable    
    0044B04A        jnz     short 0044B054    
    0044B04C        push    1                       ; 可用    
    0044B04E        pop     ebx    
    0044B04F        call    0044B090                ; 为了省代码,偶写了一个函数,用来使窗口可用或禁用,ebx=1 为可用,0 为禁用    
    0044B054        cmp     eax, 3EA                ; 是不是 ID=1002 的按钮 Disable    
    0044B059        jnz     short 0044B063    
    0044B05B        push    0                       ; 禁用    
    0044B05D        pop     ebx    
    0044B05E        call    0044B090    
    0044B063        cmp     eax, 3EB                ; 是不是 ID=1003 的按钮 Show    
    0044B068        jnz     short 0044B072    
    0044B06A        push    5                       ; 显示    
    0044B06C        pop     ebx    
    0044B06D        call    0044B0A3                ; 自己写的函数，用来显示或隐藏窗口    
    0044B072        cmp     eax, 3EC                ; 是不是 ID=1004 的按钮 Hide    
    0044B077        jnz     short 0044B081    
    0044B079        push    0                       ; 隐藏    
    0044B07B        pop     ebx    
    0044B07C        call    0044B0A3    
    0044B081        popad    
    0044B082        nop    
    0044B083        pop     ebp    
    0044B084        jmp     [44C664]                ; 跳到 MFC42 库的 OnCommand
```

> 在 0044B090 写入偶的第一个函数：

> 代码:

```
    0044B090        push    ebx                     ; /Enable    
    0044B091        mov     eax, [46CFF0]           ; |    
    0044B096        push    eax                     ; |hWnd => NULL    
    0044B097        call    [44C9AC]                ; \EnableWindow    
    0044B09D        retn                            ; 返回
```

> 在 0044B0A3 写入偶的第二个函数：

> 代码:

```
    0044B0A3        push    ebx             ; /ShowState    
    0044B0A4        mov     eax, [46CFF0]           ; |    
    0044B0A9        push    eax                     ; |hWnd => NULL    
    0044B0AA        call    [44C8F0]               ; \ShowWindow    
    0044B0B0        retn
```

> 保存就可以了，试一下，是不是可以把窗口隐藏、显示、禁用、使之可用了呢!

> ----------------------------------------------------------------------------

> 【经验总结】

> 主要是 `MFC` 的消息处理，有很多种方法。可以看《看雪论坛精华》，我这种方法应该算是比较简单(偷懒??)的方法... `T_T`

> ----------------------------------------------------------------------------

> 【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

> ------------------------------------------------------------------

> 文章写于 2007-08-05

`4dd466d0e1e0cc123afbe504da2462ca`
