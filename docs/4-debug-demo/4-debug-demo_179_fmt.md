
# 警告（NAG）窗口

> 原文：[`www.pediy.com/kssd/tutorial/chap6-1-21.htm`](https://www.pediy.com/kssd/tutorial/chap6-1-21.htm)

**警告（NAG）窗口** 【习题】

**1、习题一** `chap6-1-2-01` 157K NAG 易
**2、习题二** `chap6-1-2-02` 2K NAG 易
**3、习题三** `chap6-1-2-03` 9K NAG 中
**4、习题四** `chap6-1-2-04` 23K NAG 中（需 `cw3220.dll`）
**5、习题五** `chap6-1-2-05` 193K NAG 中

## 1、习题一 答案

此程序在开始运行有 NAG 出现，退出时也会出现。

在 `SOFTICE` 下命令：

```
bpx messageboxa
```

开始运行程序，立即将中断，此时按 `F11` 将看到 NAG。（不要按 `F5`, 不然你点 OK 将不会中断），`F11` 的含义：执行到返回地址（`^G @ss:sp;`）然后按 OK, 将再次中断返回 `SOFTICE`。如下：

```
\* Reference To: user32.MessageBoxA, Ord:0000h

:0043EFDE E87572FCFF           Call 00406258
:0043EFE3 33D2                 xor edx, edx---你将落在此
:0043EFE5 8B83C8020000         mov eax, dword ptr [ebx+000002C8]
:0043EFEB E8A82EFEFF          call 00421E98
:0043EFF0 5B                   pop ebx
:0043EFF1 C3                   ret
```

按 `F10`, 走出此 CALL, 会来到：

```
:00439721 8D4000               lea eax, dword ptr [eax+00]
:00439724 53                   push ebx
:00439725 6683B86602000000     cmp word ptr [eax+00000266], 0000
:0043972D 7410                 je 0043973F-------此处可跳出 NAg 的 call
:0043972F 8BD8                 mov ebx, eax
:00439731 8BD0                 mov edx, eax
:00439733 8B8368020000         mov eax, dword ptr [ebx+00000268]
:00439739 FF9364020000         call dword ptr [ebx+00000264] ----产生 NAg 的窗口
:0043973F 5B                   pop ebx<---你来到此
:00439740 C3                   ret
```

好，现在找到 1 个地方，此时按 `F5`，程序运行。最后，关闭程序，将再次中断。（你不要禁止开始的断点）

按 `f11`, NAG 出现，点击 Ok, 再次中断，按 `F10` 来到：

```
:0043709B 8945FC               mov dword ptr [ebp-04], eax
:0043709E 8B45FC               mov eax, dword ptr [ebp-04]
:004370A1 6683B8B602000000     cmp word ptr [eax+000002B6], 0000
:004370A9 7441                 je 004370EC-------此处可跳出 NAg 的 call
:004370AB 33C0                 xor eax, eax
:004370AD 55                   push ebp
:004370AE 68D5704300           push 004370D5
:004370B3 64FF30               push dword ptr fs:[eax]
:004370B6 648920               mov dword ptr fs:[eax], esp
:004370B9 8B5DFC               mov ebx, dword ptr [ebp-04]
:004370BC 8B55FC               mov edx, dword ptr [ebp-04]
:004370BF 8B83B8020000         mov eax, dword ptr [ebx+000002B8]
:004370C5 FF93B4020000         call dword ptr [ebx+000002B4]----产生 NAg 的窗口
:004370CB 33C0                 xor eax, eax---你将会来到此
:004370CD 5A                   pop edx
:004370CE 59                   pop ecx
:004370CF 59                   pop ecx
```

好了，在十六进制工具修改 `0043972D` 和 `004370A9` 处，将两处的 `je` 改 `jne` (`74 to 75`)

## 2、习题二 答案

在 `SOFTICE` 下命令：

```
bpx MessageBoxA
```

然后再运行程序，将会中断，按 `f11` 后 NAG 将出现，点击 OK, 将再次被中断，具体如下：

```
:0040100C  6A30                PUSH    30                       <-- 这四个 PUSH 是
:0040100E  6879204000          PUSH    00402079                 <-- 把参数传给
:00401013  688D204000          PUSH    0040208D                 <-- 下面的 MessageBoxA 函数
:00401018  FF3548204000        PUSH    DWORD PTR [00402048]     <--
:0040101E  E8DA010000          CALL    USER32!MessageBoxA ------产生 NAG 的窗口
:00401023  C7050020400003400000MOV     DWORD PTR [00402000],00004003
:0040102D  C705042040003D114000MOV     DWORD PTR [00402004],0040113D
:00401037  C7050820400000000000MOV     DWORD PTR [00402008],00000000
:00401041  C7050C20400000000000MOV     DWORD PTR [0040200C],00000000
:0040104B  A144204000          MOV     EAX,[00402044]
:00401050  A310204000          MOV     [00402010],EAX
```

向上看也没代码跳过 `:0040101E` 的 call，有什么好办法能跳过此 call?

**方法一：**

这方法我比较推荐，在 `0040100C` 处加一跳转指令，路过此 call。

也就是改成：`jmp 00401023`

在 `SOFTICE` 下，在 `:0040100C` 这一行，下 `A` 命令，改成 `jmp 00401023`，记下机器码的变化。

机器码的结果是：`EB 15`（15 是 401023-40100E 的值）

**方法二：**

向上看也没代码跳过 `:0040101E` 的 call，因此我们干脆将此处 `NOP` (就是无操作 No Operation), 该指令不执行任何操作，其机器码占有一个字节。

因此：`E8DA010000` 改 `9090909090` 后 NAG 将不出现。（指令 `nop` 的机器码是 `90`）

## 3、习题三 答案

这程序显示 NAG 是用了另一种方法，不是用 `messagebox`.

让我们先运行程序，开始的 NAG 窗口有 2 个按钮，按第一个给出一小的信息窗口(`Don't be lame ... blah blah blah`) ，第 2 个将带你进入程序。

想想我们该如何去掉 nag? 第一件事是用 `W32Dasm` 看看...

装载程序后，点击 `W32Dasm` 的串式数据参考（String Data References），列出相关字符串，双击 `[Don't be lame ...]` ，这时你将来到：

`* Referenced by a (U)nconditional or (C)onditional Jump at Address:`

```
:0040105F(C)

:0040109E 6840100000 push 00001040* Possible StringData Ref from Data Obj ->"NO!"

:004010A3 6808204000 push 00402008* Possible StringData Ref from Data Obj ->"Don't be lame, crack the program."

:004010A8 680C204000 push 0040200C
:004010AD FF3500204000 push dword ptr [00402000]* Reference To: USER32.MessageBoxA, Ord:0000h

:004010B3 E89B000000 Call 00401153
:004010B8 C9 leave
:004010B9 C21000 ret 0010
```

第一行 `* Referenced by...` 是告诉你这个信息框调用来自于 `[0040105F]`（小 c 意思是条件转移）

因此我们用转到代码位置命令（`shift+F12`）跳到 `0040105F`，来到：

`* Referenced by a (U)nconditional or (C)onditional Jump at Address:`

```
:00401032(C)

:0040105B 837D1001 cmp dword ptr [ebp+10], 00000001
:0040105F 743D je 0040109E
:00401061 837D1002 cmp dword ptr [ebp+10], 00000002
:00401065 7404 je 0040106B
:00401067 C9 leave
:00401068 C21000 ret 0010
```

这里是程序判断你的按钮，如你按第一个按钮，`0040105B` 测试，然后跳出一对话框 `"Don't be lame.."`，如你按第二个按钮，将在 `:00401061` 测试，.... 现在我们需要做的是，不管你按第一还是第二个按钮，都应带我们进入程序，这样做可防止我们改别的东西程序运行出错。我们将 `[je 0040109E]` 改成 `[je 0040106B]`

我们现在 `HVIEW` 里改：

打开程序，按 `F4` 选择模式，有 3 个，在这里先 `DECODE`, 将反汇编程序，按 `F7` 查找机器码 `837D1001`，将来到：

```
0000065B: 837D1001 cmp d,[ebp][00010],001 ;""
0000065F: 743D je 00000069E
00000661: 837D1002 cmp d,[ebp][00010],002 ;""
00000665: 7404 je 00000066B
```

大家注意了在 `HVIEW` 看到的地址和我们在 `W32DASM` 看到不同，`HVIEW` 是显示的是文件的偏移地址（File offset)，而 `W32DASM` 和 `SOFTICE` 下显示的地址完全一样，是内存地址（memory offset）或称虚拟地址。它们之间的换算有多种方法：

第一、用我刚才方法，查找机器码来确定其位置。

第二、是借助些这方面的工具软件来计算，在主页的工具下载中第 2 个链接站点有这方面的工具。

第三、此种方法更简单：你在 `W32DASM` 中光标定位需要一行，看看 `W32Dasm` 的最底端，将会看到类似：

```
Line:298 Pg 4 of 12 Code Data @:0040113E @Offset 0000073Eh in File:????.exe
```

其中 `Offset 0000073Eh` 就是 `HVIEW` 中的位置。

我们按 `F3` 进入编辑状态，按 `TAB` 键或回车键，将 `[je 00000069E]` 改成 `[je 00000066B]`，按 `F9` 存盘。当然这时你的 `W32DASM` 不能调用此文件，不然是不能存盘的。

Ok, 完成第一步

看看，上一段代码是何处被调用？是 `00401032(C)` 处，因此我们跳转此处：

`* Reference To: USER32.DialogBoxParamA, Ord:0000h`

```
:0040101D E82B010000 Call 0040114D
:00401022 E911010000 jmp 00401138
:00401027 C8000000 enter 0000, 00
:0040102B 817D0C11010000 cmp dword ptr [ebp+0C], 00000111
:00401032 7427 je 0040105B
:00401034 817D0C10010000 cmp dword ptr [ebp+0C], 00000110
:0040103B 7410 je 0040104D
:0040103D 837D0C10 cmp dword ptr [ebp+0C], 00000010
:00401041 0F84F1000000 je 00401138
:00401047 33C0 xor eax, eax
:00401049 C9 leave
:0040104A C21000 ret 0010
```

看看 `[00401032]`，另一条件指令（一个检测你按了哪个按钮的指令），你不需了解它是如何比较的，因此我们让它直接跳转 `0040105B` 处，不让它在那里循环等待你按哪个键，因此我们简单将 `00401032` 处的 `JE` 改成 `JNE`，这样程序应被 cracked 了！

让我运行程序看看，天啊！跳出一警告窗口！！`ERROR...` 说什么程序被改了！因此这程序有 CRC 检测功能（如检测你修改程序将停止运行），好吧，再让我们把它干掉。

在 `W32Dasm` 串式数据参考（String Data References）中查找 `'ERROR'` 信息，双击来到：

`* Referenced by a (U)nconditional or (C)onditional Jump at Address:`

```
:004010E8(C)

:004010EE 6840100000 push 00001040* Possible StringData Ref from Data Obj ->"ERROR"

:004010F3 68BD204000 push 004020BD* Possible StringData Ref from Data Obj ->"ERROR: Program has detected tampering. "
->"Execution terminated"

:004010F8 6881204000 push 00402081
:004010FD FF3500204000 push dword ptr [00402000]
```

上述被 `[004010E8]` 调用，让我们来到那里：

`* Referenced by a (U)nconditional or (C)onditional Jump at Address:`

```
:004010D0(C)

:004010DE 813D04204000697A0000 cmp dword ptr [00402004], 00007A69
:004010E8 7504 jne 004010EE
:004010EA C9 leave
:004010EB C21000 ret 0010
```

好了，我们不必再了解此段代码是何处引用了，足够了。这段程序是检测源代码是否修改，如果修改将给出出错信息，我们将 `:004010E8` 处的 `JNE` `NOP` 掉或改成 `JE.`。

Ok, 再运行，YE.。成功！



### 小结
本题主要是了解如何用 `W32DASM` 来静态反汇编调试程序，以及 `HVIEW` 的用法。

### 4. 习题四 答案

**作者：** Etenal Bliss

#### 使用 `W32Dasm` 分析导入表
用 `W32Dasm` 装载程序，在菜单处选择功能 → 导入（imports)，在弹出的菜单中你将看到几个 `"cw3220.\_\_XXX"` 函数，这些是该程序调用的 `cw3220.dll`。还有这些：

- `USER32.DialogboxParamA`
- `USER32.EndDialog`
- `USER32.MessageBoxA`
- `USER32.DialogboxParamA`
- `USER32.EndDialog`

其中 `MessageBoxA` 肯定没用在 NAG。（因为作者在程序已说明了）

因此很可能：函数 `DialogBoxParamA` 产生 NAG, `EndDialog` 结束 NAG。

双击 `SER32.DialogBoxParamA` 我们看看哪里代码调用此函数，你多双击几次，将会看到几处地方都引用了它：

`004010AF`, `0040114C`, `004014EE`

具体如下...

**USER32.DialogBoxParamA at 004010AF**

```
\=================================================================

\* Possible Reference to Dialog: DialogID\_0002

:00401098 6A02                   push 00000002
:0040109A FF7508                 push [ebp+08]

\* Reference To: USER32.EndDialog, Ord:0000h

:0040109D E858040000             Call 004014FA
:004010A2 6A00                   push 00000000
:004010A4 68DF104000             push 004010DF
:004010A9 6A00                   push 00000000

\* Possible Reference to Dialog: DialogID\_0001

:004010AB 6A01                   push 00000001
:004010AD 6A00                   push 00000000

\* Reference To: USER32.DialogBoxParamA, Ord:0000h

:004010AF E83A040000             Call 004014EE

\* Possible Reference to Dialog: DialogID\_0001

:004010B4 B801000000             mov eax, 00000001
:004010B9 EB20                   jmp 004010DB

\=================================================================
```

**USER32.DialogBoxParamA at 0040114C**

```
\=================================================================

:0040113B 55                     push ebp
:0040113C 8BEC                   mov ebp, esp
:0040113E 6A00                   push 00000000
:00401140 687C104000             push 0040107C
:00401145 6A00                   push 00000000

\* Possible Reference to Dialog: DialogID\_0002

:00401147 6A02                   push 00000002
:00401149 FF7508                 push [ebp+08]

\* Reference To: USER32.DialogBoxParamA, Ord:0000h

:0040114C E89D030000             Call 004014EE
:00401151 33C0                   xor eax, eax
:00401153 5D                     pop ebp
:00401154 C21000                 ret 0010

\=================================================================
```

最后一处的 `004014EE` 调用 `USER32.DialogBoxParamA` 和我们没什么关系。

看看这两段代码，你将看到 `"DialogID\_0001"` 和 `"DialodID\_0002"` 在 `DialogBoxParamA` 之前。因此让我们想想这函数有些什么参数？我们参考 W32 API 手册：

```
int DialogBoxParam(
    HINSTANCE hInstance,      // handle to application instance
    LPCTSTR lpTemplateName,   // identifies dialog box template
    HWND hWndParent,          // handle to owner window
    DLGPROC lpDialogFunc,     // pointer to dialog box procedure
    LPARAM dwInitParam        // initialization value
);
```

Ok.在这函数前你需 5 个参数。

现在，这两个 `"DialogID\_0001"` 和 `"DialodID\_0002"` 是哪个部分调用的??让我们来到代码开始处：

+++++++++++++++++ DIALOG INFORMATION ++++++++++++++++++

Number of Dialogs = 2 (decimal)

Name: `DialogID\_0001`, # of Controls=009, Caption:"Crackme 2a - n0p3x", ClassName:""

  001 - ControlID:0002, Control Class:"BUTTON" Control Text:"E&xit"
  002 - ControlID:0009, Control Class:"BUTTON" Control Text:"A&bout"
  003 - ControlID:0065, Control Class:"EDIT" Control Text:"Nag Removal The previous programs have"
  004 - ControlID:0066, Control Class:"BUTTON" Control Text:"-=n0p3x=-"
  005 - ControlID:FFFF, Control Class:"STATIC" Control Text:"Coded By n0p3x. 10th May 1999."
  006 - ControlID:FFFF, Control Class:"STATIC" Control Text:"EMAIL: adminno1@yahoo.com"
  007 - ControlID:FFFF, Control Class:"STATIC" Control Text:"WEB: http://cod3r.cjb.net"
  008 - ControlID:FFFF, Control Class:"STATIC" Control Text:"If you suceed in killing this nag screen and write a tutorial on it then email"
  009 - ControlID:FFFF, Control Class:"STATIC" Control Text:"Frame2"

Name: `DialogID\_0002`, # of Controls=004, Caption:"The deadly NAG!", ClassName:""

  001 - ControlID:FFFF, Control Class:"STATIC" Control Text:"This is a demonstration version of this program."
  002 - ControlID:0065, Control Class:"BUTTON" Control Text:"Uhh, youv'e made me feel guilty now. Heres all my money."
  003 - ControlID:0066, Control Class:"BUTTON" Control Text:"Take the program for a test drive before paying."
  004 - ControlID:FFFF, Control Class:"STATIC" Control Text:"SOFTWARE PIRACY IS ILLEGAL"

\=================================================================

现在，如你运行程序，你将看到其 NAG 的标题是 `"The deadly NAG!"`。因此 NAG 是 `DialogID\_0002` 而主程序调用的是 `0001`.

还记得在导入（imports)处的 `"USER32.EndDialog"` ？它是根据 `DialogID` 的 `push` 参数来关掉相应的对话框的，OK,让我们来 crack 它：

**USER32.DialogBoxParamA at 004010AF**

```
\=================================================================

\* Possible Reference to Dialog: DialogID\_0002        <<Nag ID

:00401098 6A02                   push 00000002
:0040109A FF7508                 push [ebp+08]

\* Reference To: USER32.EndDialog, Ord:0000h

:0040109D E858040000             Call 004014FA          <<关闭 Nag!!
:004010A2 6A00                   push 00000000          <<主程序调用
:004010A4 68DF104000             push 004010DF           USER32.DialogBoxParamA 的第一参数
:004010A9 6A00                   push 00000000

\* Possible Reference to Dialog: DialogID\_0001         <<主程序的 ID

:004010AB 6A01                   push 00000001
:004010AD 6A00                   push 00000000

\* Reference To: USER32.DialogBoxParamA, Ord:0000h     <<显示程序

:004010AF E83A040000             Call 004014EE

\* Possible Reference to Dialog: DialogID\_0001

:004010B4 B801000000             mov eax, 00000001
:004010B9 EB20                   jmp 004010DB

\=================================================================
```

**USER32.DialogBoxParamA at 0040114C**

```
\=================================================================

:0040113B 55                     push ebp
:0040113C 8BEC                   mov ebp, esp
:0040113E 6A00                   push 00000000          << Nag 调用函数
:00401140 687C104000             push 0040107C           USER32.DialogBoxParamA 第一参数
:00401145 6A00                   push 00000000

\* Possible Reference to Dialog: DialogID\_0002         <<Nag ID

:00401147 6A02                   push 00000002
:00401149 FF7508                 push [ebp+08]

\* Reference To: USER32.DialogBoxParamA, Ord:0000h     <<显示 Nag!!

:0040114C E89D030000             Call 004014EE
:00401151 33C0                   xor eax, eax
:00401153 5D                     pop ebp
:00401154 C21000                 ret 0010

\=================================================================
```

如果你将代码 `0040113E` 到 `0040114C` 中 6 行 `NOP` 掉，你将出错，程序运行将崩溃！

该如何打这个补丁？

OK,我们直接从 `0040113E` 跳转到 `004010A2`，为什么是 `004010A2`??

因从 `00401098` 到 `0040109D` 这段代码是 NAG 的结束，我们停止程序的 NAG 进程，不需要 End-Nag 这段代码。

现在我们需要发现用什么机器码补丁，程序跳到主程序时不经 NAG 的显示和 End-Nag。

你可用 `SOFTICE` 来做。当然你也可用 `W32Dasm` 来代替。

1) 在 `"Debug"`, 选择 `"Load Process"`
2) 点击 `"Load"`
3) 你将会看到三个窗口，不要害怕. 8P
4) 来到右边的窗口（那个显示`"Code Address ..."`）
5) 点击`"Goto Address"`
6) 填上地址 `0040113E` (你还记得我们将要从这跳走)
7) 你将看到来到那里，`PUSH...`
8) 点击 `"Patch Code"`



9. 在 `Enter New Instruction Below` 这行，键入 `jmp 004010A2`（这没有引号）。（记住我们将要跳到这儿）

10. 点击 `回车键`。你将看到 `jmp 004010A2...`

11. 机器码是 `E95FFFFFFF`

12. 现在，点击 `Clear Patch` 回答 `Yes` 然后点击 `Close`

13. 回到右窗口点击 `Terminate` 因为我们结束了。

返回主窗口，来到 `0040113E`，在 `W32Dasm` 的最底部，你将看到：

```
Line:298 Pg 4 of 12 Code Data @:0040113E @Offset 0000073Eh in File:crackme2a.exe
```

你将看到其偏移地址：`0000073E`

现在复制一份这 `crackme` 程序，用十六进制程序打开（不然你在 `W32Dasm` 下是不能修改文件的），来到地址 `0000073E`，你将看到字节：`6A 00 68 7C 10`（机器码）

同 `W32Dasm` 比较一下（它们是一样的）

改 `E9 5F FF FF FF` 并存盘

`crack` 结束。

小结：其实作者修改机器码不需这麻烦，可直接在 `hievw` 用汇编代码改。但大家可进一步了解 `W32DASM` 的动态调试功能。

### 5. 习题五 答案

用 `TRW LOAD` 需破解的 `cm_id11.exe`

按 `F10` 直到：

```
00447D90          CALL  0043EA00
```

跳出 NAG

尝试把这句 NOP 掉，可是出错。

那就按 `F8` 进入此 CALL

```
0043EA00          PUSH  EBP
...               ...
...               ...
```

按 `F10`，直到：

```
0043EA31          CALL  [EDI+2C]
```

跳出 NAG

尝试把这句 NOP 掉，可是出错。

那就按 `F8` 进入此 CALL

按 `F10`，直到：

```
00437E8B              CMP  BYTE PTR [EBP-05], 00
00437E8F  740f        JE   00437EAO
00437E91  e876b1fcff  CALL 0040300C  ----跳出 NAG
```

尝试修改 `00437E8F` 一句，出错。

再尝试把 `00437E91` 的 `CALL` NOP 掉，即将机器码 `e876b1fcff` 改成 `9090909090`，成功！

`ZXEM` `2000.3.26`

`4dd466d0e1e0cc123afbe504da2462ca`
