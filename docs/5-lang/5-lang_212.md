
# MFC 的逆向工程

> 原文：[`www.pediy.com/kssd/pediy06/pediy6538.htm`](https://www.pediy.com/kssd/pediy06/pediy6538.htm)

例子 `test.exe`（向导生成的做了 6 个按钮,必须有 `mfc42.dll` 才能够运行）

原以为 MFC 的逆向工程和 SDK 一样简单，但是跟踪之后就发现自己想错了。

在程序种根本找不到消息的处理过程，也就是说没有办法在像在 SDK 中那样增加功能了（修改功能比较容易的，替换原来的处理过程即可）。

郁闷之间去查类库了，发现一个很有用的成员函数 `CWnd.DefWindowProc`，于是在这个函数调用处下断点。果然，由这个函数转入 `mfc42.dll` 中的默认函数。

嘿既然不能够修改 `mfc42.dll` 那么就修改这个函数的调用。

使用资源黑客打开 `test.exe` 添加一个按钮 id 为 1006，其他的随意。

```
004016DE   .- FF25 D8204000 JMP     DWORD PTR DS:[<&MFC42.#2385_?Def>; <-------修改这里 MFC42.#2385_?DefWindowProcA@CWnd@@MAEJIIJ@Z
```

因为 `CWnd.DefWindowProc` 是三个参数，所以在调用这个函数的时候在堆栈里由 3 个参数，分别为消息代码，`WPARAM`、`LPARAM`

```
004016DE   . /E9 7D030000   JMP     test.00401A60             修改为,跳转到自己的处理过程。
```

－－－－－－－－－－－添加的过程－－－－－－－－－－－－－－－－

```
00401A60   > \55           PUSH    EBP       <-----------保存 ebp
00401A61   .  8BEC         MOV     EBP, ESP
00401A63   .  60           PUSHAD          <-----保存所有的寄存器，非常重要
00401A64   .  90           NOP
00401A65   .  8B45 08      MOV     EAX, DWORD PTR SS:[EBP+8]  <-----------获得消息代码
00401A68   .  3D 11010000  CMP     EAX, 111     <------------比较消息是否为 WM_COMMAND
00401A6D   .  75 61        JNZ     SHORT test.00401AD0  <------------如果不是则跳到默认的处理过程
00401A6F   .  8B45 0C      MOV     EAX, DWORD PTR SS:[EBP+C]  <------------获得按钮 ID
00401A72   .  3D EE030000  CMP     EAX, 3EE              <-------比较是否为添加的 id 1006 的十六进制
00401A77   .  75 57        JNZ     SHORT test.00401AD0  <-------------如果不是则跳到默认的处理过程
```

```
--------------------添加自定义的功能部分开始----------
00401A79   .  6A 00        PUSH    0
00401A7B   .  6A 00        PUSH    0
00401A7D   .  6A 00        PUSH    0
00401A7F   .  E8 14FDFFFF  CALL    <JMP.&MFC42.#4224_?MessageBoxA@C>  <------调用成员函数
00401AD0   > \90           NOP
00401AD1   .  90           NOP
--------------------添加自定义的功能部分结束----------
00401ADC   .  61           POPAD           <---------------恢复所有寄存器，非常重要
00401ADD   .  90           NOP
00401ADE   .  5D           POP     EBP     <-----------------恢复 ebp
00401ADF   .  90           NOP
00401AE0   .- FF25 D8204000 JMP     DWORD PTR DS:<&MFC42.#2385_?Def>;  MFC42.#2385_?DefWindowProcA@CWnd@@MAEJIIJ@Z        <----------跳入到 mfc42.dll 的默认处理过程
```

保存运行，哈～ 添加功能成功。

总结一下，修改 `CWnd::DefWindowProc` 是重点，该函数有 3 个参数（api 函数 `DefWindowProc` 有 4 个参数）

`CWnd:: DefWindowProc(unit message, WPARAM wParam, LPARAM lPraram)`

分别是消息，附加信息，附加信息。也就是所有的消息处理过程都先通过这个函数，只要在前面加上自己添加功能的消息的判断，进入自己的消息处理，然后在条回到 `CWnd:: DefWindowProc` 这里即可。

另外一个重点就是在自己的处理过程之前要保存所有的寄存器，处理过后要恢复所有的寄存器。

我最先就是因为没有保存寄存器而郁闷了好久。（原本以为 api 函数不会更改寄存器的）

其实只是添加按钮的话还是修改 `CWnd:: OnCommand` 这里比较的好，因为修改 `CWnd:: DefWindowProc` 的话会有很大的资源开销，所有的消息都会先判断是否自己的消息，然后在进入默认的消息循环。

修改 `CWnd:: OnCommand` 的话就只在 `WM_COMMAND` 事件中判断是否为自己的消息，而且只需要获得按钮 id 即可。

打算再去研究一下 DELPHI 的逆向工程，添加按钮功能（修改功能比较容易，修改相应的处理过程即可）。

我还没有一点思路，对于 DELPHI 的消息处理也不怎么懂。如果能够的到高手指导，将不尽感激。

我的 QQ:1030092

或者 E-MAIL: `ogata@eyou.com`

`4dd466d0e1e0cc123afbe504da2462ca`
