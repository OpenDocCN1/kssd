
# 基于明码的典型 diy 注册机

> 原文：[www.pediy.com/kssd/pediy09/pediy09-205.htm](https://www.pediy.com/kssd/pediy09/pediy09-205.htm)

> 很久没写文章了，最近也没干什么，就整理了一些经验。

> 看题目，也就是说程序算法比较必须是明码，我们要把源程序部份改成注册机。当然，CrackMe 和普通的软件就不太一样！！我用一个明码 CrackMe 来说明，其他的举一反三就行了。

> 另外，我打包的这个 CrackMe 有多层壳吧，把图标都压没了！是 Delphi 写的程序。

> 用 `DEDE` 找到注册按钮事件代码，在 `00457344` 处下断：(一路按 `F8`，直到看到真码)

```
00457344  /.  55            push    ebp
00457345  |.  8BEC          mov     ebp, esp
00457347  |.  83C4 F0       add     esp, -10
0045734A  |.  53            push    ebx
0045734B  |.  56            push    esi
0045734C  |.  57            push    edi
0045734D  |.  33C9          xor     ecx, ecx
0045734F  |.  894D F4       mov     dword ptr [ebp-C], ecx
00457352  |.  894D F0       mov     dword ptr [ebp-10], ecx
00457355  |.  8945 FC       mov     dword ptr [ebp-4], eax
00457358  |.  33C0          xor     eax, eax
0045735A  |.  55            push    ebp
0045735B  |.  68 B4744500   push    004574B4
00457360  |.  64:FF30       push    dword ptr fs:[eax]
00457363  |.  64:8920       mov     dword ptr fs:[eax], esp
00457366  |.  E8 15FBFFFF   call    00456E80
0045736B  |.  8B45 FC       mov     eax, dword ptr [ebp-4]
0045736E  |.  83B8 FC020000>cmp     dword ptr [eax+2FC], 1
00457375  |.  0F84 1B010000 je      00457496
0045737B  |.  BF 12020000   mov     edi, 212
00457380  |>  47            /inc    edi
00457381  |.  83EF 02       |sub    edi, 2
00457384  |.  81FB 94000000 |cmp     ebx, 94
0045738A  |.  7F 09         |jg      short 00457395
0045738C  |>  43            |/inc    ebx
0045738D  |.  81FB 94000000 ||cmp     ebx, 94
00457393  |.^ 7E F7         |\jle     short 0045738C
00457395  |>  81FF F0000000 |cmp     edi, 0F0
0045739B  |.^ 7F E3         \jg      short 00457380
0045739D  |.  8D55 F4       lea     edx, dword ptr [ebp-C]
004573A0  |.  8B45 FC       mov     eax, dword ptr [ebp-4]
004573A3  |.  8B80 D0020000 mov     eax, dword ptr [eax+2D0]
004573A9  |.  E8 76D6FCFF   call    00424A24
004573AE  |.  837D F4 00    cmp     dword ptr [ebp-C], 0
004573B2  |.  75 11         jnz     short 004573C5
004573B4  |.  A1 3C964500   mov     eax, dword ptr [45963C]
004573B9  |.  8B00          mov     eax, dword ptr [eax]
004573BB  |.  E8 0864FEFF   call    0043D7C8
004573C0  |.  E9 D1000000   jmp     00457496
004573C5  |>  8D55 F4       lea     edx, dword ptr [ebp-C]
004573C8  |.  8B45 FC       mov     eax, dword ptr [ebp-4]
004573CB  |.  8B80 C4020000 mov     eax, dword ptr [eax+2C4]
004573D1  |.  E8 4ED6FCFF   call    00424A24
004573D6  |.  837D F4 00    cmp     dword ptr [ebp-C], 0
004573DA  |.  0F84 B6000000 je      00457496
004573E0  |.  C745 F8 02000>mov     dword ptr [ebp-8], 2
004573E7  |.  8B75 F8       mov     esi, dword ptr [ebp-8]
004573EA  |>  03DB          /add    ebx, ebx
004573EC  |.  81FB EE000000 |cmp     ebx, 0EE
004573F2  |.  7C 1B         |jl      short 0045740F
004573F4  |.  81FF 26020000 |cmp     edi, 226
004573FA  |.  7F 13         |jg      short 0045740F
004573FC  |>  83C7 14       |/add    edi, 14
004573FF  |.  43            ||inc    ebx
00457400  |.  83C7 1E       ||add    edi, 1E
00457403  |.  4B            ||dec    ebx
00457404  |.  83EF 31       ||sub    edi, 31
00457407  |.  81FF 26020000 ||cmp     edi, 226
0045740D  |.^ 7E ED        |\jle     short 004573FC
0045740F  |>  03DF          |add    ebx, edi
00457411  |.  8D55 F0       |lea     edx, dword ptr [ebp-10]
00457414  |.  8BC3          |mov     eax, ebx
00457416  |.  E8 2D04FBFF   |call    00407848
0045741B  |.  8B45 F0       |mov     eax, dword ptr [ebp-10]
0045741E  |.  E8 8DC6FAFF   |call    00403AB0
00457423  |.  8345 F8 03    |add     dword ptr [ebp-8], 3
00457427  |.  33DB          |xor     ebx, ebx
00457429  |.  8BFB          |mov     edi, ebx
0045742B  |.  836D F8 02    |sub     dword ptr [ebp-8], 2
0045742F  |.  81FE D30D0000 |cmp     esi, 0DD3
00457435  |.  7C 0D         |jl      short 00457444
00457437  |>  BE 01000000   |/mov    esi, 1
0045743C  |.  81FE D30D0000 ||cmp     esi, 0DD3
00457442  |.^ 7D F3         |\jge     short 00457437
00457444  |>  817D F8 D0070>|cmp     dword ptr [ebp-8], 7D0
0045744B  |.^ 7C 9D        \jl      short 004573EA
0045744D  |.  E8 AEFAFFFF   call    00456F00
00457452  |.  8D55 F4       lea     edx, dword ptr [ebp-C]
00457455  |.  8B45 FC       mov     eax, dword ptr [ebp-4]
00457458  |.  8B80 C8020000 mov     eax, dword ptr [eax+2C8]
0045745E  |.  E8 C1D5FCFF   call    00424A24
00457463  |.  8B45 F4       mov     eax, dword ptr [ebp-C]
00457466  |.  8B55 FC       mov     edx, dword ptr [ebp-4]
00457469  |.  8B92 F8020000 mov     edx, dword ptr [edx+2F8]<--这里可以看到真码，说明是明码，把明码的指针传到 edx,那么[edx+2f8]就是字符串的首地址了，看一下 OD 的信息窗口，如下
```

![image](img/38fba5fe8e48401d014ae58f35c67a0e.jpg)

可以看出 `00d6aeec` 是真码地址

```
0045746F  |.  E8 4CC7FAFF   call    00403BC0      //进去比较
00457474  |.  74 0E         je      short 00457484 //是否注册成功
```

记下地址；

接下来我们要把这串东西显示出来，当然，我们要 diy 一个似模似样的注册机，如果以 `MessageBox` 形式输入，太没趣，而且不能复制！

我们看一下程序的界面：

![image](img/465b00a06aeed0a6c22f24d6f2c7a328.jpg)

我想我们最好是把真码显示到 Serial 框，哈哈，这样才像真正的注册机！

要在对话框的编辑框上显示字符串，当然是用 `SetWindowText`，或者 `SetDlgItemText`

至于用哪个，我们看一下它们的参数：

```
BOOL SetWindowText(
  HWND hWnd,             // handle to window or control
  LPCTSTR lpString       // title or text
);
```

一个窗口句柄和要显示的字符串

```
BOOL SetDlgItemText(
  HWND hDlg,             // handle to dialog box
  int nIDDlgItem,       // control identifier
  LPCTSTR lpString      // text to set
);
```

一个对话框句，位于对话框的控件 ID，和要显示的字符串。

当然，我们就选用 `SetWindowText` 这个函数。

OK，看程序界面，初始化时我们已经可以看到两个对话框中已经有内容了，所以姑且判断程序有调用 `SetWindowText` 这个函数，我们用现成的，就不用再添加一个了，至于程序如果没调用此函数，我们又该怎么办！这个问题，后续再讲。

`Ctrl+f2` 重新在 `OD` 中载入程序，`Ctrl+n` 打开程序函数列表，找一下 `SetWindowText` 函数，发现有一个，在上面回车来查找参考，发现一个都没有，汗！！

我们用 `PE Load` 用打程序：

![image](img/4f87fe56a4493cb715c22fb8ef0ea961.jpg)



# 逆向分析与注册机补丁制作

> 点击“目录”按钮，我们可以看到有导出表和导入表，也就是导入函数的意思。程序使用函数都是导入后的函数，关于导入表知识，大家可以参看相关 PE 教程。

> 我们点击导入表后面的那个“。。”按钮，如下图：

![](img/f67bc703a5cce9853e3787436ba802b1.jpg)

在 `dll` 名称中找到 `user32.dll`，下面列表框就会出现程序调用 `user32.dll` 里的函数列表，我们找到 `SetWindowTextA`！

![](img/46b054f30e510796c71ddb4439f739a8.jpg)

`ThunkRVA` 值就是我们要的，关于 `RVA` 这个关键字的概念，大家可以去看 `CCDebuger` 的那篇《浅淡脱壳后的优化》，里面有介绍！

我们就可以 `call dword ptr[0045b42c]` 这样来调用 `SetWindowTextA` 这个函数了！

好了，知道怎么调用，也知道程序原有的 `SetWindowText` 函数了，现在要着手的是该函数的参数。我们知道该函数有两个参数，第二个字符串我们用真码地址，已经有了；第一个是窗口的句柄，就是我们要把该字符串设置到哪一个窗口的，该窗口的句柄！

找窗口的句柄有很多种方法，我们知道每一个按钮、每一个编辑框都是一个窗口，在我们屏幕上看到的都是一个个大小形态不同的窗口！创建窗口，无论你在编程时是画上一个按钮还是什么，系统都会调用函数去创建，比如 `CreateWindow` 或 `CreateWindowEx`，程序当然也会用到！！

我们就 `Ctrl+F2` 重新载入，清除所有断点，`Ctrl+N` 打开程序函数列表，看到有一个 `CreateWindowExA`。呵，在命令行上下 `bp CreateWindowExA`，`F9` 运行会多次中断，这是当然，因为程序的窗口控件不只一个。

我们中断之后看堆栈，因为是断在函数开头，参数还未被使用，所以在进入 `call` 前所压入栈内的参数还在，所以我们在堆栈中看到的是最准确的值。

## 第一次中断（堆栈）

![](img/7fbd0c27cc051184cbc14144c687098e.jpg)

这些都是该函数的参数，我们要关注的就是 `Class` 窗口类别、`WindowName` 窗口名，和其返回值。

`Class`：知道编辑框类别为 `Edit`，我们就 `F9` 直到 `Class` 参数为 `Edit` 关键字为止。

`F9` 中断 7 次后看到 `Class` 为 `TEdit` 类别，不过 `WindowName` 却是 `2Sweeet`，显然是第一个编辑框，不管它，继续 `F9`。

第 8 次终于看到我们想要的了，`Class` 为 `TEdit`，`WindowName` 为“Serial”，接下来我们 `Alt+F9`，返回调用语句的下一条语句！

来到这里：

```
00427D44  8986 40010000   mov  dword ptr [esi+140], eax  ; 句柄传[esi+140]
```

我前面说过要关注该函数的返回值的，该函数创建窗口成功之后会返回窗口的句柄，而汇编中函数的返回值都会先放在 `eax` 里，所以这里就是把句柄传到 `[esi+140]` 这个内存地址中。

看一下信息窗口：

![](img/075d717821fdad6860fb70536ddd4f1b.jpg)

得知句柄为 `0x80322`，当然我们不能直接使用 `0x80322`，句柄是系统分配的，每次运行程序都会不一样，所以，记住地址才是最有用的。记下 `esi+140=00d65654`。

## 寻找全 0 空间

要补丁的话还是用全 0 的地方比较安全，下面找全 0：

我们用 `PEid` 找到全 0 处代码，好打入我们的补丁。`Peid` 载入，点击 `EP` 区段右边的“>”按钮，出现如下图：

![](img/98a6343c621429ca4cd99e041b2311df.jpg)

在区段查看器上右键，搜索全 0，出现下图：

![](img/aff4a65cf881a00af242122cd91410d5.jpg)

呃，`upx1` 这个区都有，汗，这就是脱壳后没优化的结果，再次推荐看一下 `CCDebuger` 的那篇文章哈！！好了，我选择“SuckMe!!”吧，看起来实在点！^-^ 记住偏移 `00070cef` 加上基址 `400000`！

嘿嘿，找到全 0 处了！

## 条件汇总

我们所具备的条件，就是这些：

1.  真码地址 `00d6aeec`
2.  编辑框句柄地址 `00d65654`
3.  全 0 地址 `00470cef`

OK，所要的条件已具备，下面开始打注册机补丁！

`Ctrl+F2` 重新载入，`Ctrl+G` 来到 `457344` 处，也就是注册按钮点击事件代码：

```
00457344  /.  55            push   ebp   //断在这里

…………………....

省略 N 行代码

………………

0045744D  |.  E8 AEFAFFFF  call   00456F00  //f8 到这里时，注册码已经算出来了

………………

00457469  |.  8B92 F8020000 mov    edx, dword ptr [edx+2F8] 
// [edx+2f8]就是真码地址，这里把真码字符串指针传到 edx

Ps：这里可以利用一下，我们直接传参数为 edx 就行了！地址就不用记了！况且地址中 esi 会变动！— |||
```

```
0045746F  |.  E8 4CC7FAFF  call   00403BC0  
//比较的 call,为了不把上面的真码指针 edx 破坏掉，这里我们 nop★
```

```
00457474  /74 0E           je     short 00457484
//是个决定注册是否成功的跳转

Ps：这里也利用起来，就跳转到我们的全 0 处，打补丁去吧！

改为 jmp 00470cef，记住把“用 nop 填补”给选上！
```

`00470cef` 处，我们写入如下代码：

```
00470CEF   90            nop   //留个空吧
00470CF0   90            nop   //留个空吧
00470CF1   60            pushad  //保护现场
00470CF2   90            nop     //补个空
00470CF3   52            push    edx  //函数的第二个参数，把真码指针压进栈
00470CF4   FF35 5456D600  push    dword ptr [D65654] //第一个参数，把句柄压进栈
00470CFA   FF15 2CB44500 call dword ptr [<&USER32.SetWindowTex>; //调用 SetWindowText 函数,.前面说了怎么调用的，只要 call dword ptr[0045b42c]这样就行了
```

```
00470D00   61            popad  //恢复现场
00470D01   E9 7367FEFF    jmp     00457479  //跳回原来的代码执行
```

跳回到这里：

```
00457479   90           nop  //这是补空用的
0045747A   90           nop  //这是补空用的
0045747B   .  8B00      mov     eax, dword ptr [eax]
0045747D   E8 4663FEFF  call  0043D7C8 //这个不 nop 的话会异常，所以，nop★
```

OK，补丁完毕，输入 `key` 的那个框变灰不能用，这还不完美，搞得只能通过一个 `key` 来算注册码，我们还要再改一下：

直接下 `bp EnableWindow` 函数断点，断不下来，看来创建该窗口时已经决定变灰了！我们干脆自己调用 `EnableWindow` 函数算了！

看一下该函数的定义：

**BOOL** `EnableWindow(`
**  HWND*** `hWnd`***,     **// handle to window
  **BOOL** *bEnable*   // enable or disable input
**);**

一个窗口句柄，和一个 `bool` 值，真假变量，为 0 就为假，非 0 为真，再看下面那句：

*bEnable* 

\[in\] Specifies whether to enable or disable the window. If this parameter is TRUE, the window is enabled. If the parameter is FALSE, the window is disabled. 

如果这个参数为真，这个窗口就激活，如果为假，就禁止！

我们只要传递一个非 0 值就行了！

### 打补丁

下断，`bp ShowWindow`，在程序显示的时候断下：

```
77D1D8A4 >  B8 2B120000    mov      eax, 122B  //断在这里
77D1D8A9   BA 0003FE7F    mov      edx, 7FFE0300
77D1D8AE   FF12         call    dword ptr [edx]
77D1D8B0   C2 0800      retn    8
```

`Alt+F9` 返回程序领空：

```
0043D3D1   50           push    eax //ShowWindow 函数的参数
0043D3D2   E8 ED90FCFF   call    <jmp.&USER32.ShowWindow>//调用 ShowWindow
0043D3D7   E9 05010000    jmp     0043D4E1 //无条件跳转，Ps：这里利用一下，先记下原跳转的目标地址，也就是 0043d4e1！然后我们通过用 peid 找全 0 位置，我找的是 00470d0f！这里改为 jmp 00470d0f★
```

`Ctrl+G` 到达 `00470d0f`：

```
00470D0F   60           pushad //保护现场
00470D10   6A 01        push  1 //第二个参数，我们传到非 0 值，让窗口激活
00470D12   FF35 6C5DD600  push    dword ptr [D65D6C] //传进得到的窗口句柄
00470D18   FF15 C8B54500      call dword ptr [<&USER32.EnableWindow> //调用函数
00470D1E   61           popad  //恢复现场
00470D1F   E9 BDC7FCFF     jmp     0043D4E1 //恢复被我们破坏的代码,跳到我们刚才记住的地址上去执行，万无一失哈！
00470D24   90           nop   //补一个空\]
```

OK，打完补丁，右键->复制可执行文件->所有修改(如果没有所有修改的话，用鼠标先中上面我们打进的补丁代码，右键->复制可执行文件->选择->出现的新窗口再右键->保存文件，这样只是把上面的补丁码给保存了，程序压根就没运行到这里，所以，我们再用 OD 载入保存后的文件，把 `0043d3d7` 处的跳转再改为 `jmp 00470d0f`，跳到我们的代码去执行！再按上述方法保存一次)OK，运行程序，`key` 框已经可以输入了！

哟嘿，程序有注册机的内涵了，界面也改一下，用 `eXe` 或 `Res` 去改就行了，不过程序含有非标准资源，改的时候注册别改变原本资源的大小，比如我把 `name` 框的“2Sweeet”，总共 7 个字节，改为“大菜一号”的话，就不行了！改为“大菜 1 号”倒可以，哈！

因为那个机器码的框，在本机上注册的话要本机的机器码，所以可以不清空，注册码输入框，程序会调用 `SetWindowText` 把真码显示出来，所以也不用清空了，注册名框的话，为了宣传宣传自己，所以改为“大菜 1 号”啦``哈哈哈哈哈⁰^ ！！

下面请看两个程序的截图：

![](img/e2e25665b3ae562ba21edb484c7617b0.jpg)

最后请看胜利截图：

![](img/088b01201cf0543d6599204a43c53c0e.jpg)

呀哈哈哈哈，终于，，终于，完成啦！！！！，

想想第二篇要写什么哈？？？…….想到再说啦！！

