
# OllyDBG 入门系列（七）－汇编功能

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-415.htm`](https://www.pediy.com/kssd/pediy08/pediy8-415.htm)

今天我们的目标程序是 `MyUninstaller` 1.34 版。这是一个非常小的程序卸载工具，`VC6` 编写，大小只有 61K。我拿到的这个是上次闪电狼兄弟给我的，附带在里面的简体中文语言文件是由六芒星制作的。

附件下载：`myuninst.rar`

这个程序有个毛病：就是在列出的可卸载程序上双击查看属性时，弹出的属性窗口的字体非常难看，应该就是系统字体（`SYSTEM_FONT`）：

![](img/dfce7f2f2b4be47fe183d1d3cbd4adff.jpg)

我们今天的目标就是利用 `OllyDBG` 的汇编功能把上面显示的字体改成我们常见的 9 号（小五）宋体。首先我们用 `OllyDBG` 载入程序，按 `CTR+N` 组合键查找一下有哪些 API 函数，只发现一个和设置字体相关的 `CreateFontIndirectA`。现在我们按鼠标右键，选择“在每个参考上设置断点”，关掉名称对话框，`F9` 运行，程序已经运行起来了。我们在程序的列表框中随便找一项双击一下，很不幸，那个字体难看的界面又出现了，`OllyDBG` 没有任何动作。可见创建这个窗口的时候根本没调用 `CreateFontIndirectA`，问题现在就变得有点复杂了。先点确定把这个字体难看的对话框关闭，现在我们从另一个方面考虑：既然没有调用设置字体的函数，那我们来看看这个窗口是如何创建的，跟踪窗口创建过程可能会找到一些对我们有用的信息。现在我们再回到我们调试程序的领空，按 `CTR+N` 看一下，发现 `CreateWindowExA` 这个 API 函数比较可疑。我们在 `CreateWindowExA` 函数的每个参考上设上断点，在 `MyUninstaller` 的列表框中再随便找一项双击一下，被 `OllyDBG` 断下：

```
  00408F5E  |.  FF15 98B24000   |CALL DWORD PTR DS:[<&USER32.CreateWindowExA>]    ; \\断在这里
```

上下翻看一下代码：

```
  00408F3B  |.  50               |PUSH EAX                                     ; |hInst
  00408F3C  |.  8B45 C0          |MOV EAX,DWORD PTR SS:[EBP-40]                 ; |
  00408F3F  |.  6A 00            |PUSH 0                                       ; |hMenu = NULL
  00408F41  |.  03C6            |ADD EAX,ESI                                   ; |
  00408F43  |.  FF75 08          |PUSH DWORD PTR SS:[EBP+8]                    ; |hParent
  00408F46  |.  FF75 D0          |PUSH DWORD PTR SS:[EBP-30]                   ; |Height
  00408F49  |.  57               |PUSH EDI                                     ; |Width
  00408F4A  |.  50               |PUSH EAX                                     ; |Y
  00408F4B  |.  FF75 BC          |PUSH DWORD PTR SS:[EBP-44]                   ; |X
  00408F4E  |.  FF75 EC          |PUSH DWORD PTR SS:[EBP-14]                   ; |Style
  00408F51  |.  68 80DE4000      |PUSH myuninst.0040DE80                       ; |WindowName = ""
  00408F56  |.  68 DCD94000      |PUSH myuninst.0040D9DC                       ; |Class = "STATIC"
  00408F5B  |.  FF75 D4          |PUSH DWORD PTR SS:[EBP-2C]                   ; |ExtStyle
  00408F5E  |.  FF15 98B24000    |CALL DWORD PTR DS:[<&USER32.CreateWindowExA>]    ; \\断在这里
  00408F64  |   6A 00            |PUSH 0                                       ; 第一处要修改的地方
  00408F66  |   8945 F4          |MOV DWORD PTR SS:[EBP-C],EAX
  00408F69  |.  E8 A098FFFF      |CALL <myuninst.sub_40280E>
  00408F6E  |.  50               |PUSH EAX                                     ; |hInst
  00408F6F  |.  8B45 DC          |MOV EAX,DWORD PTR SS:[EBP-24]                 ; |
  00408F72  |.  6A 00            |PUSH 0                                       ; |hMenu = NULL
  00408F74  |.  03F0            |ADD ESI,EAX                                   ; |
  00408F76  |.  FF75 08          |PUSH DWORD PTR SS:[EBP+8]                    ; |hParent
  00408F79  |.  FF75 CC          |PUSH DWORD PTR SS:[EBP-34]                   ; |Height
  00408F7C  |.  53               |PUSH EBX                                     ; |Width
  00408F7D  |.  56               |PUSH ESI                                     ; |Y
  00408F7E  |.  FF75 D8          |PUSH DWORD PTR SS:[EBP-28]                   ; |X
  00408F81  |.  FF75 E8          |PUSH DWORD PTR SS:[EBP-18]                   ; |Style
  00408F84  |.  68 80DE4000      |PUSH myuninst.0040DE80                       ; |WindowName = ""
  00408F89  |.  68 D4D94000      |PUSH myuninst.0040D9D4                       ; |Class = "EDIT"
  00408F8E  |.  FF75 B8          |PUSH DWORD PTR SS:[EBP-48]                   ; |ExtStyle
  00408F91  |.  FF15 98B24000    |CALL DWORD PTR DS:[<&USER32.CreateWindowExA>]    ; \\CreateWindowExA
  00408F97  |   8945 F0          |MOV DWORD PTR SS:[EBP-10],EAX                ; 第二处要修改的地方
  00408F9A  |   8B45 F8          |MOV EAX,DWORD PTR SS:[EBP-8]
  00408F9D  |.  FF30            |PUSH DWORD PTR DS:[EAX]                      ; /<%s>
  00408F9F  |.  8D85 B0FEFFFF    |LEA EAX,DWORD PTR SS:[EBP-150]               ; |
  00408FA5  |.  68 D0D94000      |PUSH myuninst.0040D9D0                       ; |format = "%s:"
  00408FAA  |.  50               |PUSH EAX                                     ; |s
  00408FAB  |.  FF15 90B14000    |CALL DWORD PTR DS:[<&MSVCRT.sprintf>]        ; \\sprintf
  00408FB1  |.  8B35 84B24000    |MOV ESI,DWORD PTR DS:[<&USER32.SetWindowTextA>]  ;  USER32.SetWindowTextA
  00408FB7  |.  83C4 0C          |ADD ESP,0C
  00408FBA  |.  8D85 B0FEFFFF    |LEA EAX,DWORD PTR SS:[EBP-150]
  00408FC0  |.  50               |PUSH EAX                                     ; /Text
  00408FC1  |.  FF75 F4          |PUSH DWORD PTR SS:[EBP-C]                    ; |hWnd
  00408FC4  |.  FFD6            |CALL ESI                                     ; \\SetWindowTextA
  00408FC6  |.  8D85 ACFAFFFF    |LEA EAX,DWORD PTR SS:[EBP-554]
  00408FCC  |.  50               |PUSH EAX                                     ; /Arg3
  00408FCD  |.  FF75 FC          |PUSH DWORD PTR SS:[EBP-4]                    ; |Arg2
  00408FD0  |.  FF35 00EF4000    |PUSH DWORD PTR DS:[40EF00]                   ; |Arg1 = 00BEADCC
  00408FD6  |.  E8 1884FFFF      |CALL <myuninst.sub_4013F3>                   ; \\sub_4013F3
  00408FDB  |.  83C4 0C          |ADD ESP,0C
  00408FDE  |.  50               |PUSH EAX
  00408FDF  |.  FF75 F0          |PUSH DWORD PTR SS:[EBP-10]
  00408FE2  |.  FFD6            |CALL ESI
  00408FE4  |.  FF45 FC         |INC DWORD PTR SS:[EBP-4]
  00408FE7  |.  8345 F8 14       |ADD DWORD PTR SS:[EBP-8],14
  00408FEB  |.  837D FC 0F      |CMP DWORD PTR SS:[EBP-4],0F
  00408FEF  |.^ 0F8C 32FFFFFF    \\JL <myuninst.loc_408F27>
  00408FF5  |.  5F               POP EDI
  00408FF6  |.  5E               POP ESI
  00408FF7  |.  5B               POP EBX
  00408FF8  |.  C9               LEAVE
  00408FF9  \.  C3               RETN
```

我想上面的代码我不需多做解释，`OllyDBG` 自动给出的注释已经够清楚的了。我们双击 `MyUninstaller` 列表框中的某项查看属性时，弹出的属性窗口上的 `STATIC` 控件和 `EDIT` 控件都是由 `CreateWindowExA` 函数创建的，然后再调用 `SetWindowTextA` 来设置文本，根本没考虑控件上字体显示的问题，所以我们看到的都是系统默认的字体。我们要设置控件上的字体，可以考虑在 `CreateWindowExA` 创建完控件后，在使用 `SetWindowTextA` 函数设置文本之前调用相关字体创建函数来选择字体，再调用 `SendMessageA` 函数发送 `WM_SETFONT` 消息来设置控件字体。思路定下来后，我们就开始来实施。首先我们看一下这个程序中的导入函数，`CreateFontIndirectA` 这个字体创建函数已经有了，再看看 `SendMessageA`，呵呵，不错，原程序也有这个函数。这样我们就省事了。有人可能要问，如果原来并没有这两个导入函数，那怎么办呢？其实这也很简单，我们可以直接用 `LordPE` 来在程序中添加我们需要的导入函数。我这里用个很小的 PE 工具 `zeroadd` 来示范一下，这个程序里面没有 `CreateFontIndirectA` 和 `SendMessageA` 函数（这里还有个问题说一下，其实我们编程时调用这两个函数时都是直接写 `CreateFontIndirect` 及 `SendMessage`，一般不需指定。但在程序中写补丁代码时我们要指定这是什么类型的函数。这里在函数后面加个“A”表示这是 ASCII 版本，同样 UNICODE 版本在后面加个“W”，如 `SendMessageW`。在 `Win9X` 下我们一般都用 ASCII 版本的函数，UNICODE 版本的函数很多在 `Win9X` 下是不能运行的。而 NT 系统如 `WinXP` 一般都是 UNICODE 版本的，但如果我们用了 ASCII 版本的函数，系统会自动转换调用 UNICODE 版本。这样我们写补丁代码的时候就可以直接指定为 ASCII 版本的函数，可以兼容各个系统）：我们用 `LordPE` 的 PE 编辑器载入 `zeroadd` 程序，选择“目录”，再在弹出的目录表对话框中选择输入表后面的那个“...”按钮，会弹出一个对话框：

![](img/bc27b7c5b10e55c2aa5df82e6f2effd8.jpg)

因为 `SendMessageA` 在 `USER32.dll` 中，我们在右键菜单中点击按钮“添加导入表”，来到下面：

![](img/4c1c7eea55635afd3259d09dba132a6f.jpg)

按上面的提示完成后点“确定”，我们回到原先的那个“输入表”对话框：

![](img/3a6a63758697b47d29a8fc33f73a660c.jpg)
