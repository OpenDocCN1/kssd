
# [原创]vbsedit去除注册对话框

- 【文章标题】: vbsedit去除注册对话框
- 【文章作者】: MPL
- 【软件名称】: Vbsedit 5.2.4
- 【软件大小】: 8.5M
- 【下载地址】: ae8K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4k6T1M7$3g2V1K9i4c8Q4x3X3g2U0L8$3#2Q4x3V1k6@1M7W2)9#2k6X3c8G2N6$3&6D9L8$3q4V1i4K6u0W2j5i4y4H3
- 【加壳方式】: 无壳
- 【保护方式】: 不懂
- 【编写语言】: Microsoft Visual Studio .NET 2005 -- 2008 -> Microsoft Corporation [Overlay]
- 【使用工具】: OD PEID SPY++
- 【操作平台】: Windows XP
- 【软件介绍】: 编写vbs脚本的IDE，具有语法高亮、代码自动提示等功能，可调试脚本代码。另外，内附大量的模板代码，可直接套用。
- 【作者声明】: 本人非专业逆向人员，只是兴趣所致，想把原有版本的限制去掉。连这开头的几个“字段”都是抄的别人的:P

---

非注册版本有两个限制，一是软件启动的时候，会弹出要求注册的窗体A；二是在点击Start按钮时，会弹出另外一个不同的要求注册的窗体B。

首先打开利器OD，加载`vbsedit.exe`。RUn之后，要两次Shift+F9跳过异常（我也不知道是软件设置的反调试导致的，菜阿。。。）。

ps:之前我没有检查软件加壳了没有，因为菜到极致的我，并不知道加没加壳对调试有什么影响。。。

## 1. 寻找register按钮单击事件代码

我一开始想找到点击窗体A的Register按钮所执行的代码的地方，看能不能把关键的跳转改掉，进而随便输个注册码让注册成功。

### (1) 对`GetWindowTextA`、`GetWindowTextW`、`GetDlgItemTextA`、`GetDlgItemTextW`、`SendDlgItemMessageA`和`SendDlgItemMessagW`下断点

对`SendMessageW`下`WM_GETTEXT`的条件断点。

实际发现原作者都使用了unicode版本的函数。结果是在注册窗体A输入注册码的时候，会断下来；而在点击register按钮的时候却断不下来。我没仔细看代码，猜测是这样的：在输入的时候实时保存文本框的内容到全局字符串，然后点击register按钮的时候直接取全局字符串。

### (2) 对`SendMessageW`下`WM_COMMAND`的条件断点

我在OD的左下角命令行处输入`bp SendMessageW`，另外需对其设置条件，就是何时使这个断点激活。

当然是按Register按钮的时候，于是在ALT+B调出断点窗体后，右键编辑其条件：`[ESP+8]==WM_COMMAND && [ESP+10]==01234567`，其中，`01234567`是用SPY++查看到的Register按钮的句柄。

整个条件的意思就是，`SendMessageW`的第二个参数为`WM_COMMAND`,并且第4个参数是Register按钮的句柄(其实严格的说还要第三个参数是`MAKEWPARAM(按钮id, BN_CLICKED)`)。

因为这个条件就表示当按钮Register被单击的时候，Regiser按钮向其父控件（即窗体A）发送`WM_COMMAND`消息。

当时我也不知道参数怎么表示，我看了OD的帮助，里面的表达式的例子中有类似的，我就拿来用了。后来想明白了：

当执行到`SendMessageW`内部的时候，`[ESP]`即栈顶存储着由调用它的函数的`call`压入的`SendMessageW`的下一条语句的`EIP`，而`[ESP+4]`当然是`SendMessageW`的第一个参数（因为`SendMessageW`是`stdcall`调用约定，参数从右向左依次压入栈），进而`[ESP+8]`、`[ESP+0C]`、`[ESP+10]`分别存储第2、3、4个参数。

后来果然在点击register按钮之后断了下来，接着我就在OD右下角的窗口（一直不知道叫什么窗口，汗...）点击鼠标右键，选择“在汇编窗口跟随”，这样就来到了调用`SendMessageW`的函数所在的Context，居然不是vbsedit的环境，而是系统的另一个dll环境`COMCTL32`。

我迷茫了，然后我又在OD提供的堆栈窗口中（第一次知道有这个窗口，瞎弄...），发现了`WM_LBUTTONUP`常量，这不是鼠标左键弹起的消息吗？也就是点击按钮之后一定会产生的消息！然后我又稀里糊涂从堆栈窗口跟随到汇编窗口，当时好像是接着对`DispatchMessageW`下了条件断点，进入到了程序领空（其实后来发现在`WM_COMMAND`断下来后，在堆栈窗口可以右键选择"Show call"就来到程序领空了），但是一步一步跟着跟着，我就无所适从了，还是不知道到底register按钮的点击事件的代码在哪里。。。

### (3) 对`ShellExecuteW`下断点

依然想找到register按钮的点击事件代码。

我在窗体A不输任何东西，直接点击注册按钮，会打开注册的网页。

我猜测作者是调用了`ShellExecute`，而由之前的`SendMessageW`推知应当还是调用的Unicode版本的`ShellExecuteW`。

果然不错，断了下来，但是我在OD的右下角窗口中右键跟随到汇编窗口的时候傻眼了，这一堆的`db`啊！按Ctrl+A再分析一下依然没变啊！这样只能再次放弃了！（但是在整理这篇文档的时候，我又试了一次，发现又是正常的代码了，悲剧阿！是不是OD版本的原因阿？）

ps:其实后来我还以为软件是使用`GetProcAddress`动态加载`ShellExecuteW`的地址的。但是后来设置了条件断点，没有断到！！！

## 2. 寻找显示注册窗体A的时机

我一开始以为是`ShowWindow`，就简单的`bp ShowWindow`，但是殊不知很多控件如组合框、工具栏什么的都会使用`ShowWindow`。而且不仅仅是在他们创建之后才`ShowWindow`，好像有个时钟或是在消息循环里调用`ShowWindow`，如果改掉`ShowWindow`的调用以隐藏注册窗体A而不使其他控件受影响，应当会很麻烦。

## 3. 寻找创建注册窗体A的时机

对`CreateWindowW`（不知道OD为什么提示未知的标志符）、`CreateWindowExW`、`CreateDialogIndirectParamW`等下断点。

创建对话框的API比较多，我使用`CreateDialogIndirectParamW`成功断了下来，跟随到汇编窗口：

```
0047E22F   > \53            push    ebx                              ; /lParam
0047E230   .  68 D1D94700   push    0047D9D1                         ; |pDlgProc = vbsedit.0047D9D1
0047E235   .  50            push    eax                              ; |hOwner
0047E236   .  57            push    edi                              ; |pTemplate
0047E237   .  FF75 10       push    dword ptr [ebp+10]               ; |hInst
0047E23A   .  FF15 8CD96700 call    dword ptr [<&USER32.CreateDialog>; \CreateDialogIndirectParamW
0047E240   .  8B4D E5       mov     ecx, dword ptr [ebp-1C]
0047E243   .  83C1 F0       add     ecx, -10
0047E246   .  8BF8          mov     edi, eax
0047E248   .  E8 3330F8FF   call    00401280
0047E24D   .  834D FC FF    or      dword ptr [ebp-4], FFFFFFFF
0047E251   .  EB 24         jmp     short 0047E277
0047E253   .  8B4D D4       mov     ecx, dword ptr [ebp-2C]
0047E256   .  85C9          test    ecx, ecx
0047E258   .  74 05         je      short 0047E25F
0047E25A   .  E8 C4F3FFFF   call    0047D623
0047E25F   >  8B45 E0       mov     eax, dword ptr [ebp-20]
0047E262   .  8348 60 FF    or      dword ptr [eax+60], FFFFFFFF
0047E266   .  834D FC FF    or      dword ptr [ebp-4], FFFFFFFF
0047E26A   .  B8 70E24700   mov     eax, 0047E270
0047E26F   .  C3            retn
0047E270   .  8B75 E0       mov     esi, dword ptr [ebp-20]
0047E273   .  33DB          xor     ebx, ebx
0047E275   .  8BFB          mov     edi, ebx
0047E277   >  8B4D D8       mov     ecx, dword ptr [ebp-28]
0047E27A   .  3BCB          cmp     ecx, ebx
0047E27C   .  74 18         je      short 0047E296
0047E27E   .  3BFB          cmp     edi, ebx
0047E280   .  74 14         je      short 0047E296
0047E282   .  8B01          mov     eax, dword ptr [ecx]
0047E284   .  8D55 B8       lea     edx, dword ptr [ebp-48]
0047E287   .  52            push    edx
0047E288   .  FF50 18       call    dword ptr [eax+18]
0047E28B   .  8B06          mov     eax, dword ptr [esi]
0047E28D   .  53            push    ebx
0047E28E   .  8BCE          mov     ecx, esi
0047E290   .  FF90 58010000 call    dword ptr [eax+158]
0047E296   >  E8 97490000   call    00482C32
0047E29B   .  85C0          test    eax, eax
0047E29D   .  75 0A         jnz     short 0047E2A9
0047E29F   .  8B06          mov     eax, dword ptr [esi]
0047E2A1   .  8BCE          mov     ecx, esi
0047E2A3   .  FF90 20010000 call    dword ptr [eax+120]
0047E2A9   >  3BFB          cmp     edi, ebx
0047E2AB   .  74 0F         je      short 0047E2BC
0047E2AD   .  F646 58 10    test    byte ptr [esi+58], 10
0047E2B1   .  75 09         jnz     short 0047E2BC
0047E2B3   .  57            push    edi                              ; /hWnd
0047E2B4   .  FF15 A4D96700 call    dword ptr [<&USER32.DestroyWindo>; \DestroyWindow
0047E2BA   .  33FF          xor     edi, edi
0047E2BC   >  395D EC       cmp     dword ptr [ebp-14], ebx
0047E2BF   .  74 12         je      short 0047E2D3
0047E2C1   .  FF75 EC       push    dword ptr [ebp-14]               ; /hMem
0047E2C4   .  FF15 F0D36700 call    dword ptr [<&KERNEL32.GlobalUnlo>; \GlobalUnlock
0047E2CA   .  FF75 EC       push    dword ptr [ebp-14]               ; /hMem
0047E2CD   .  FF15 50D56700 call    dword ptr [<&KERNEL32.GlobalFree>; \GlobalFree
0047E2D3   >  33C0          xor     eax, eax
0047E2D5   .  3BFB          cmp     edi, ebx
0047E2D7   .  0F95C0        setne   al
0047E2DA   >  E8 4E551A00   call    0062382D
0047E2DF   .  C2 0C00       retn    0C;返回到下面

;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;

0047E3EA   .  33FF          xor     edi, edi                         ;  vbsedit.00400000
0047E3EC   .  3BC7          cmp     eax, edi
0047E3EE   .  74 3A         je      short 0047E42A
0047E3F0   .  F646 58 10    test    byte ptr [esi+58], 10
0047E3F4   .  74 1E         je      short 0047E414
0047E3F6   .  6A 04         push    4
0047E3F8   .  5F            pop     edi
0047E3F9   .  8BCE          mov     ecx, esi
0047E3FB   .  E8 62910000   call    00487562
0047E400   .  A9 00010000   test    eax, 100
0047E405   .  74 03         je      short 0047E40A
0047E407   .  6A 05         push    5
0047E409   .  5F            pop     edi
0047E40A   >  57            push    edi;push进的参数（nop掉！）
0047E40B   .  8BCE          mov     ecx, esi
0047E40D   .  E8 38400000   call    0048244A;显示窗体的函数，跟进去发现，不断的在PeekMessage。（nop掉！）
0047E412   .  33FF          xor     edi, edi
0047E414   > \397E 20       cmp     dword ptr [esi+20], edi
0047E417   .  74 11         je      short 0047E42A
0047E419   .  68 97000000   push    97
0047E41E   .  57            push    edi
0047E41F   .  57            push    edi
0047E420   .  57            push    edi
0047E421   .  57            push    edi
0047E422   .  57            push    edi
0047E423   .  8BCE          mov     ecx, esi
0047E425   .  E8 E6950000   call    00487A10;上面的显示窗体的函数的返回值eax实际上在这个call里面没有用到，所以不用构造其返回值；实际上这个call里面是在释放资源前的隐藏窗体的函数
```

注意：`nop`掉显示窗体的函数的同时，要把压入的参数那里也`nop`掉，否则堆栈不平衡！

后来证明`nop`掉的显示窗体的函数不仅对注册窗体A和注册窗体B起作用，
