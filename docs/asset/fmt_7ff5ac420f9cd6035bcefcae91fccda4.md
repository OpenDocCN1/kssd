
进入`/path/to/xxx`目录，找到`xxx.json`。

在表达式`cvar = avar + bvar`中，加法运算符（`+`）将`avar`与`bvar`相加，得到它们的和`cvar`

在`List.of(arg0, arg1, arg2)`中，`List`接口的工厂方法`of()`接受一系列的元素，返回包含它们的只读列表。

之后我们这样调用`cmd`命令：`cmd arg0 arg1 arg2`。

```
if (condVar > someVal) {console.log("xxx")}
```

```
0040137E  /$  8B7424 04     mov     esi, dword ptr [esp+4]
00401382  |.  56            push    esi                              ;  ESI="lichsword"
00401383  |>  8A06          /mov     al, byte ptr [esi]              ;  取一个字符到 a[i]
00401385  |.  84C0          |test    al, al                          ;  判断 a[i]是否为 0，即字符串是否处理完
00401387  |.  74 13         |je      short serial.0040139C           ;  相等，即处理完毕，就跳到函数结束
00401389  |.  3C 41         |cmp     al, 41
0040138B  |.  72 1F         |jb      short serial.004013AC           ;  如果 a[i]<'A'成立就跳，一跳就完蛋，说明必须是字母，不能为数字或其它字符
0040138D  |.  3C 5A         |cmp     al, 5A
0040138F  |.  73 03         |jnb     short serial.00401394           ;  如果 a[i]>'Z'成立就跳到执行 00401383 函数
00401391  |.  46            |inc     esi
00401392  |.^ EB EF         |jmp     short serial.00401383
00401394  |>  E8 39000000   |call    serial.004013D2                  ;  小写字母转成大写字母
00401399  |.  46            |inc     esi                             ;  指向下一个字符
0040139A  |.^ EB E7         \\jmp     short serial.00401383           ;  下一循环
0040139C  |>  5E            pop     esi                              ;  EDI="LICHSWORD"
0040139D  |.  E8 20000000   call    serial.004013C2                  ;  功能：字符串各字符累加和送 EDI
004013A2  |.  81F7 78560000 xor     edi, 5678                        ;  用户名各字符累加和再与常数 5678 异或
004013A8  |.  8BC7          mov     eax, edi                         ;  结果送 EAX
004013AA  |.  EB 15         jmp     short serial.004013C1            ;  跳转到函数返回
004013AC  |>  5E            pop     esi
004013AD  |.  6A 30         push    30                               ; /Style = MB_OK|MB_ICONEXCLAMATION|MB_APPLMODAL
004013AF  |.  68 60214000   push    serial.00402160                  ; |Title = "Error!  "
004013B4  |.  68 69214000   push    serial.00402169                  ; |Text = "Incorrect!,Try Again"
004013B9  |.  FF75 08       push    dword ptr [ebp+8]                ; |hOwner
004013BC  |.  E8 79000000   call    <jmp.&USER32.MessageBoxA>        ; \\MessageBoxA
004013C1  \\>  C3            ret
```

其中 `00401394  |>  E8 39000000   |call    serial.004013D2                  ;  小写字母转成大写字母          ----内部实现代码如下：`

代码:

```
004013D2  /$  2C 20         sub     al, 20                           ;  a[i]-20h,即小写字母-20h=大写字母
004013D4  |.  8806          mov     byte ptr [esi], al               ;  覆盖原字符
004013D6  \\.  C3            ret
```

代码:

```
int F1(用户名){
    char name[]="lichsword";
    int i,n=0;
    for(i=0;a[i]!='\0';i++)
    {
        if(a[i]>'Z')
            a[i]-=0x20;
        n+=a[i]
    }
    n=n^0x5678;
    return n;
}
```

用户名的处理已经分析完了，下面开始分析序列号的算法

代码:

```
004013D8  /$  33C0          xor     eax, eax
004013DA  |.  33FF          xor     edi, edi
004013DC  |.  33DB          xor     ebx, ebx
004013DE  |.  8B7424 04     mov     esi, dword ptr [esp+4]           ;  ESI="123456"
004013E2  |>  B0 0A         /mov     al, 0A                          ;  j=10
004013E4  |.  8A1E          |mov     bl, byte ptr [esi]              ;  取一个字符送到 a[i]
004013E6  |.  84DB          |test    bl, bl
004013E8  |.  74 0B         |je      short serial.004013F5           ;  如果 a[i]=='\0'，即到了字串尾，就结束循环
004013EA  |.  80EB 30       |sub     bl, 30                          ;  a[i]-30h，有点像字符转换成数字
004013ED  |.  0FAFF8        |imul    edi, eax                        ;  n=n*10
004013F0  |.  03FB          |add     edi, ebx                        ;  n=n+a[i]
004013F2  |.  46            |inc     esi                             ;  指向下一个字符
004013F3  |.^ EB ED         \\jmp     short serial.004013E2           ;  下一循环
004013F5  |>  81F7 34120000 xor     edi, 1234                        ;  n 与 1234h 异或
004013FB  |.  8BDF          mov     ebx, edi                         ;  把结果 n 送 EBX，返回
004013FD  \\.  C3            ret
```

还原成 C 就是；

代码:

```
int F2(序列号){
    char sn[]="123456";
    int i,n=0;
    for(i=0;a[i]!='\0';i++)
    {
        a[i]-=0x30;
        n=n*10+a[i];
    }
    n=n^0x1234;
    return n;
}
```

注册机就不写了，这个例子是归类于 对等函数保护机制，有点意思。分析完毕！晚安。

小结一下，逆向的关键之处为：

第一要点：找到关键代码。如果连关键代码都找不到，那从哪下断点，从哪开始分析算法呢，逆向更是空谈。

第二要点：读懂汇编代码，理清程序算法流程。这个时候已经可以在心中有个高级语言的算法雏形了。

第三要点：做些合适的处理。如，爆破、SMC、DIY、等等。

### 5.2 警告(Nag)窗口

首先，我思索着，去掉这个窗口的方法：

1、把这段代码从程序中“去掉”，即，用 `NOP` 指令覆盖全部与警告窗口相关的代码。

2、不调用显示警告窗口的子程序。相关于 `JMP OVER` 跳过。

第一种方法，不是很好，不知道为啥，好像高手们都没这么做，也许代码之间相关性很强，牵一发而动全身，因此目前我先用第二种方法。

第二种方法，要点在于，跳转，要跳得合适，既完成了去警告窗口的目的又不改变其它流程。

好，看完书后，我也自己手动分析一番。

首先当然是运行一下程序。运行之前，我先用 `PEID` 查一下壳：

![](img/36603fa68c115f991d4038b1b33acd43.jpg)![](img/2b8e04ef11a6bcfadeb60a4b81da17a7.jpg)

晕，如图所示，未知壳类型。再用 `pe-scan` 一查，也是无法识别类型。

汗，好吧，先运行看看。

![](img/e472ada1446bf56212e89189c5e2454e.jpg)

首先就出现了警告窗口，然后我们点 `OK`，主窗口显示。

![](img/b881a6a8b5537caad9298a98523a86f9.jpg)

说明警告窗口是在主窗口之前显示的，那么我们的任务就是在

警告窗口显示的代码中下断。

好，问题变成了，如何定位警告窗口的显示代码。

显示窗口的 `API` 有

`MessageBoxA(W)`---显示消息框

`DialogBoxParamA(W)`---显示对话框

`ShowWindow`---显示窗口

`CreateWindowExA(W)`---创建窗口

我们刚看到的警告窗口中，明显有 `Static` 静态文本、按钮等控件，这说明它不是消息框，而是资源定义的

对话框。

好了，我先试下 `DialogBoxParamA(W)`下断点。

![](img/f2066a71ae8de339c5c7c95d0e810198.jpg)

`Ctrl+N`,看到了 `DialogBoxParamA(W)`导入函数，选择“在每个参考上设置断点”中断。

这种情况会在 `DialogBoxParamA`

![](img/bfdf7f40672174564a99ed5f01527301.jpg)

`(W)`调用前断下，不会进入内部。

或者用 `Ctrl+G` 输入跟随的表达式 `DialogBoxParamA(W)`,不过这种方法不好，因为只有 `DialogBoxParamA(W)`函数已经被调

用后，我们才能引发中断，而这时是在系统进程中。有点走偏了的意味。

因为我们的目的是要到警告框显示之前。

代码:

```
0040104D  /$  8B4424 04     mov     eax, dword ptr [esp+4]
00401051      6A 00         push    0
00401053      68 C4104000   push    Nag.004010C4                     ;  NEG 对话框处理函数指针
00401058  |.  6A 00         push    0                                ; |hOwner = NULL
0040105A      6A 79         push    79                               ;  79 资源 ID，即 NEG
0040105C  |.  50            push    eax                              ; |hInst
0040105D  |.  A3 9C114000   mov     dword ptr [40119C], eax           ; |
00401062  |.  FF15 10104000 call    dword ptr <&USER32.DialogBoxPar>; \\DialogBoxParamA  断在此处
00401068  |.  33C0          xor     eax, eax
0040106A  \\.  C2 1000       ret     10
```

好了，现在断是断下了，但是我们怎么知道这个对话框是我们的警告框呢？

这时应该想到每个对话框都有自己的资源 `ID`，所以查看 `ID` 是否匹配就可以知道啦！

那么这个警告框的 `ID` 是什么呢？

用 `eXeScope.exe` 来查看 `ID`，如下图可知是 `79H=121`

![  

下面再看 `OD` 中我们断下来的 `ID`，不错哟，push 79 不就是这个吗，呵呵！

好了，现在的问题就是如何改跳转了，首先 `DialogBoxParamA` 的参数不能入栈，否则不就堆栈不平衡了？所以从第一个 `push 0` 改为跳转，

那么这个跳转跳到哪里去呢？

如果只是跳到最近的 `ret`。好，先试试，我改为如下：

`[CODE]00401051 /EB 17 jmp short Nag_done.0040106A`

`00401053 |. |68 C4104000 push Nag_done.004010C4 ; |DlgProc = Nag_done.004010C4`

`00401058 |. |6A 00 push 0 ; |hOwner = NULL`

`0040105A |. |6A 79 push 79 ; |pTemplate = 79`

`0040105C |. |50 push eax ; |hInst`

`0040105D |. |A3 9C114000 mov dword ptr [40119C], eax ; |`

`00401062 |. |FF15 10104000 call dword ptr [<&USER32.DialogBoxPar>; \\DialogBoxParamA`

`00401068 |. |33C0 xor eax, eax`

`0040106A \\. \\C2 1000 ret 10 ; |跳到此处。[/CODE]`

保存后，运行，结果对话框一个都没出来。

现在分析一下原因：

好，地址 `004010C4` 是警告框的处理函数起始地址。

我们查看一下这里的相关代码，在 `OD` 输入 `at 004010C4` 指令：

![](img/2c43956fb72d1b23614848da8878a9fc.jpg) 就会跳到 `004010C4` 处，查看代码。

查看代码如下：

代码:
