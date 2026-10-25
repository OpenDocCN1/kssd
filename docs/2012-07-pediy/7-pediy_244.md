
# ArtCursors 3.03

> 原文：[`www.pediy.com/kssd/pediy06/pediy6809.htm`](https://www.pediy.com/kssd/pediy06/pediy6809.htm)

首先声明：本文写的是脱壳后的修整，而不是脱壳。用`Stripper`脱`aspr`的壳还是挺方便的:P

目标软件：鼠标光标编辑软件`ArtCursors 3.03`，可制作 32 位色的`cur`、`ani`文件，并可以导入、导出多种图象格式

相关网址：[`www.aha-soft.com/artcursors/index.htm`](http://www.aha-soft.com/artcursors/index.htm)

编写语言：`Delphi`编写，`Aspr1.22`壳。

版本：`3.03`

工具：`Stripper`，`DEDE`，`OllyDBg`，`Trw2000`，`superBPM`，`TOPO`，`LordPE`，`ResHack`，`WinHex`

---------------------------------------------------------------------------------------------

脱完壳，软件一运行就退出，用`trw2000+SuperBPM`跟踪版比较，发现

```
005363E0   . FF15 F0CB5300  CALL DWORD PTR DS:[53CBF0]
```

有问题，原版`call`进去后来到`535dc0`，而脱壳版`call`进去后来到`5363d1`。试着把这一句改成

```
005363E0   . E8 DBF9FFFF    CALL UNPACKED\_.00535DC0
005363E5   . 90             NOP
```

正常运行，但为未注册版，运行前弹出`NAG`，说 30 天期限已经用了 1 天。。。

不对啊，我明明用了不止一天啊?莫非。。。?我把时间调后了一年，运行，他还是说只用了一天。。。

原来，时间限制已经解除!只要去掉`NAG`，改掉"未注册"的字样，不就搞定了?然而，事情没那么简单。。。

先用`DEDE`分析，点击`File -> load symbo file`载入调试符号`vcl60.dsf`、`vclx60.dsf`、`rtl60.dsf`

(可以用`dumpers -> dsf Builder`来创建，但必须先安装`delphi`),

在主窗口的`create`事件里发现

```
* Reference to: dmu.Proc_00534C60
00517B40   E81BD10100         call    00534C60
00517B45   837DF400          cmp     dword ptr [ebp-$0C], +$00
00517B49   7505             jnz     00517B50  <----关键跳

* Reference to: about.Proc_005084E4
00517B4B   E89409FFFF        call    005084E4  <---显示 NAG

* Reference to: kernel32.GetTickCount()
00517B50   E8F3F0EEFF        call    00406C48
```

用原版跟踪，发现`517B49`在原版跳，在脱壳版不跳，于是把脱壳版中的改成强行跳，

`NAG`没了，但是主窗口竟不能完整显示，整天异常。。。。。。

天啊，难道他`NAG`里对主窗口进行了一些初始化的工作(猜测)?怎么办。。。。连个`NAG`都搞不定?

把我这只菜鸟惹急了，我什么土办法都做的出来的。。。

我决定，用最土的办法干掉`NAG`：用`TOPO`插入代码，在合适的时机给`NAG`发一个`WM_CLOSE`消息。

首先要找到相关`API`的调用方法：

用`LordPE`的`PE Editor`打开脱壳版程序，点击`Directories`，再点击`Import Table`一栏的"..."按钮，

在上面的列表里找到`User32.dll`(好几个)，在下面的列表里找到`PostMessageA`，

看看他的`ThunkRVA`的值(最左边)，是`13f6b4`，那么`PostMessageA`的调用方法就是`call [53f6b4]`。

(`53f6b4=13f6b4+400000`)

然后`Topo`插入大概 200 字节空间，记下空间首地址：`544019`，然后在在`DEDE`里，找到`NAG`所在的单元：

`TfmAbout`，看到有`FormShow`事件，恩，好，就在这里动手脚!

接下来，要找到得到`NAG`的窗口句柄的办法，在`DEDE`里，我看到

```
00508BD5   8B00             mov     eax, [eax]

* Reference to: Controls.TWinControl.GetHandle()  <---DSF 文件的妙用
00508BD7   E8B42DF3FF      call    0043B990  <---可以得到窗口句柄
00508BDC   50             push    eax       <---返回值,窗口句柄
```

`Controls.TWinControl.GetHandle()`是得到特定窗口句柄的方法，

也就是说，在`TfmAbout`的窗口事件里，只要调用`call 0043B990`，

`EAX`里返回的就是`TfmAbout`的句柄!哈哈哈!这就是`DSF`文件的妙用，他可以认出不少`VCL`类的方法!

开始做手脚!

`FormShow`的首地址是`508ad4`，先记下开始的 5 个字节的指令(其对应的机器码是`55`，`8b`，`ec`，`6a`，`00`)：

```
00508AD4   55             push    ebp        <-- 记下
00508AD5   8BEC          mov     ebp, esp   <-- 这三个
00508AD7   6A00          push    $00        <-- 指令
00508AD9   6A00          push    $00        <-- 手脚做完从这里执行
```

然后打开`OllyDBG`，(我不习惯用`HIEW`)载入脱壳版程序，把`508AD4`处代码改成

```
00508AD4   E940B50300    jmp     00544019 <---跳到我们插入的代码空间
```

然后按`Ctrl-G`来到`544019`处，开始写入做手脚的代码：

```
                          _________________
00544019   55           PUSH EBP      |  这里先执行 508AD4 处 |
0054401A   8BEC         MOV EBP,ESP   |  被改掉的代码        |
0054401C   6A 00        PUSH 0        |_______________________|
0054401E   50           PUSH EAX  <---保护 EAX 寄存器
0054401F   E8 6C79EFFF  CALL UNPACKED\_.0043B990   <----Controls.TWinControl.GetHandle()
00544024   6A 00        PUSH 0      <---lParam
00544026   6A 00        PUSH 0      <---wParam
00544028   6A 10        PUSH 10     <---WM_CLOSE
0054402A   50           PUSH EAX    <---NAG 的窗口句柄
0054402B   FF15 B4F65300 CALL DWORD PTR DS:[53F6B4] <---PostMessageA
00544031   58           POP EAX    <---恢复 EAX 寄存器
00544032   E9 A24AFCFF  JMP UNPACKED\_.00508AD9    <---手脚做完,回到原处
```

然后选中被修改过的指令，单击右键，选"Copy to executable file"(别忘了`508AD4`处)，

再单击右键，选"Save File"，保存成新文件，运行，`NAG`窗口一闪而过，成功了。

虽然成功了，但又出来了新的麻烦：没办法显示"关于"对话筐了，因为那正是`NAG`窗口!

怎么办?把我这只菜鸟逼急了。。。(省略)

我酝酿了一个更土的计划：用`ResHack`插入一个对话筐资源，并插入代码来显示他!

我找了个用`Vc`写的有着不错的"关于"对话筐的程序，用`ResHack`打开，把"关于"对话筐另存为`.res`文件，

在用`ResHack`打开这个`.res`文件，把这个对话筐的资源名称改个象样的名字，叫什么好呢?

既然原来的"关于"对话筐的`RCDate`的名称叫`TfmAbout`，就把新对话筐的资源名称也改成"TFMABOUT"吧:P

在用`ResHack`打开动过手脚后的程序，插入`res`文件里的对话筐，保存，接下来就是写代码显示他了。

要显示对话筐，需要`DialogBoxParamA`、`EndDialog`、`GetModuleHandleA`三个`API`函数，

但是`DialogBoxParamA`、`EndDialog`两个是原程序里没有的，

不过不要紧，用`LordPE`可以在输入表里增加`API`函数。用`LordPE`打开动过手脚的程序，

点击`Directories`，再点击`Import Table`一栏的"..."按钮，在上面的列表里单击右键，选"add import"，

在`Dll`文本筐里写入`User32.dll`，在`API`文本筐里写入`EndDialog`，单击"+"号，

在`API`文本筐里再写入`DialogBoxParamA`，再单击"+"号，这样就加入了这两个`API`函数。

记下他们的`ThunkRVA`：

`EndDialog` : `1A800B`

`DialogBoxParamA` : `1A8017`

`GetModuleHandleA` : `13F210`

那么，

`EndDialog`的调用方法就是`call [5A800B]`

`DialogBoxParamA`的调用方法就是`call [5A8017]`

同理，

`GetModuleHandleA`的调用方法就是`call [53F210]`

记得保存修改。

显示"关于"对话筐的事件是主窗体`TfmMain`的`aHelpAboutExecute`事件，这是一个`TAction`对象的事件，

不能象以前那样用`call 43B990`得到主窗口的句柄了，只能另寻他法。

在`DEDE`里，`TfmAbout`的`FormActive`事件里我找到了这么一段代码：

```
* Reference to : TfmMain._PROC_0051A9BC()  <---在 NAG 里调用主窗口的过程
00508BC4   E8F31D0100        call    0051A9BC
00508BC9   E856AB0200        call    00533724
00508BCE   6AFF            push    $FF
00508BD0   A108CF5300       mov     eax, dword ptr [$0053CF08]
00508BD5   8B00            mov     eax, [eax]

* Reference to: Controls.TWinControl.GetHandle()   
00508BD7   E8B42DF3FF      call    0043B990  <--取主窗口句柄?
00508BDC   50             push    eax       <--主窗口句柄?

* Reference to : TApplication._PROC_00455A04()
00508BDD   E822CEF4FF      call    00455A04  <--改成 jmp 0054403B
00508BE2   5F             pop     edi
00508BE3   5E             pop     esi
00508BE4   5B             pop     ebx
00508BE5   5D             pop     ebp
00508BE6   C3             ret
```

猜测与实践证明，在`508BDC`一处，`eax`保存的是主窗口的句柄，我们干脆在这里也动点手脚，

让他把主窗口句柄保存起来算了，就保存在`504437`处吧，那里处于用`topo`增加的空间。

用`OllyDBG`载入修改后的程序

先把`508BDD`一处改成`jmp 0054403B`，

然后到`54403b`处写入如下代码，保存主窗口的句柄：

```
00544037   0000         ADD BYTE PTR DS:[EAX],AL
00544039   0000         ADD BYTE PTR DS:[EAX],AL
;------------- 544037 ~ 544039 的空间是用来保存主窗口句柄的---------------
0054403B   A3 37405400  MOV DWORD PTR DS:[544037],EAX <--保存住窗口句柄
00544040   E8 BF19F1FF  CALL 00455A04             <--执行 508bdd 处原来的代码
00544045   E9 984BFCFF  JMP UNPACKED1\_.00508BE2       <--手脚做完,返回原处
```

现在可以写入代码显示新的对话筐了，嘿嘿。。。

以下是主窗体`TfmMain`的`aHelpAboutExecute`事件的代码：

```
005163E8   E8F720FFFF      call    005084E4 <--改成 jmp 00544053
005163ED   C3             ret
```

把`5163E8`处改成`jmp 00544053`以后，来到`54404a`处，写入以下代码：

```
0054404a   <--此处写入 "TFMABOUT" 的 ASCII 码,是新插入的对话筐的资源名称
00544053   E8 8C44FCFF   CALL UNPACKED\_.005084E4  <---执行 5163e8 处原有代码
00544058   50           PUSH EAX               <--保护 EAX 寄存器
00544059   6A 00        PUSH 0               <---dwInitParam
0054405B   68 7C405400  PUSH UNPACKED\_.0054407C  <---对话筐过程
00544060   FF35 37405400 PUSH DWORD PTR DS:[544037]  <--父窗口句柄
00544066   68 4A405400  PUSH UNPACKED\_.0054404A <--对话筐资源名称"TFMABOUT"
0054406B   6A 00        PUSH 0
0054406D   FF15 10F25300 CALL DWORD PTR DS:[53F210] <--GetModuleHandleA
00544073   50           PUSH EAX              <--进程实例句柄 hInstance
00544074   FF15 2D805A00 CALL DWORD PTR DS:[5A802D] <--DialogBoxParamA
0054407A   58           POP EAX               <--恢复 EAX 值
0054407B   C3           RETN                  <--直接返回,不必 jmp 回原处了
```

紧接着写对话筐过程，直接写有点困难，我是用`MASM32`写了一个小程序，反汇编以后照抄的：

```
0054407C   55           PUSH EBP
0054407D   8BEC         MOV EBP,ESP
```



```
0054407F   8B45 0C          MOV EAX,DWORD PTR SS:[EBP+C]  <--把消息保存在 EAX  
00544082   3D 11010000      CMP EAX,111                     <--WM_COMMAND  
00544087   74 0B            JE SHORT UNPACKED_.00544094  

00544089   83F8 10          CMP EAX,10                      <--WM_CLOSE  
0054408C   74 06            JE SHORT UNPACKED_.00544094  

0054408E   33C0             XOR EAX,EAX  

00544090   C9               LEAVE  

00544091   C2 1000          RETN 10  

00544094   6A 00            PUSH 0  

00544096   FF75 08          PUSH DWORD PTR SS:[EBP+8]  

00544099   FF15 29805A00    CALL DWORD PTR DS:[5A8029] <--EndDialog  

0054409F   33C0             XOR EAX,EAX  

005440A1   C9               LEAVE  

005440A2   C2 1000          RETN 10  
```

这是我写的对话框过程的原代码:

```
DlgProc proc hWnd:HWND, uMsg:UINT, wParam:WPARAM, lParam:LPARAM  

mov eax,uMsg  

cmp eax,WM_COMMAND  

jz quit  

cmp eax,WM_CLOSE  

jz quit  

xor eax,eax  

ret  

quit:  

invoke EndDialog, hWnd,NULL  

xor eax,eax  

ret  

DlgProc endp  
```

别忘了保存修改结果。

试试看?点击 `help -> About`，啊，成功了，显示的是我们自己做的 About 对话框!!!

还有个恼人的地方：主窗口标题上有个可恨的“`EXPIRED!`”字样，在资源里找不到。

于是动用 `winHex` 寻找该字串，竟找不到！天啊，难道又要逼我用土办法。。。？

还是他把字母拆开了？绝望中，我在 `winHex` 里寻找“`D!`”字串，还真找到了！

在文件 `F565c` 处，有个“`D!PRDIXE`”字串，果然，他把“`EXPIRED!`”这个字串进行了变换（加密？），实在晕菜~

这下简单了，改成空格好了。

在工具栏上有个红红的“`REGISTER`”大字，很恶心，不过很好办，在 `RCDate` 资源里找的到，把大小改成 0 就好了。

大功告成，再修改资源，把 `NAG` 窗口的大小改成 0，把插入的关于对话框改的好看一点，哈哈，天衣无缝的说~~~~~~~~~~~~

`4dd466d0e1e0cc123afbe504da2462ca`
