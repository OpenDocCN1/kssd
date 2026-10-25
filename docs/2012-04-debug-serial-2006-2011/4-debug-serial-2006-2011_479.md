
# History Cleaner v1.0 算法分析

> 原文：[`www.pediy.com/kssd/pediy10/92493.html`](https://www.pediy.com/kssd/pediy10/92493.html)

- 【文章标题】: History Cleaner v1.0 算法分析
- 【文章作者】: `daokers`
- 【作者邮箱】: `daokers@qq.com`
- 【作者主页】: `http://www.daokers.com`
- 【作者 QQ 号】: `94788078`
- 【软件名称】: History Cleaner
- 【软件大小】: `600kb`
- 【下载地址】: `http://www.historycleaner.com/hcv1.exe`
- 【加壳方式】: 无壳
- 【保护方式】: 注册码
- 【编写语言】: Microsoft Visual Basic 6.0
- 【使用工具】: `OD`
- 【操作平台】: `windows xp sp2`
- 【软件介绍】: History Cleaner 是当前最为先进的网络残留和 cookie
- 【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

---

## 详细过程

History Cleaner 是一个系统和游览器历史清除工具，`vb` 语言编写，注册码保护，根据机器码来生成注册码。比较函数难下手，这里从字符入手，

启动前，搜索`'Registered Copy'`之后段首下断，好处是可以直接爆破，无需注册码，启动后从注册处下手追码相对容易些。断下后来到这里。

```
0044E190   > \55             push ebp                              ;  段首，一切罪恶都是从这里开始
0044E191   .  8BEC           mov ebp,esp
0044E193   .  83EC 18        sub esp,18
0044E196   .  68 C6574000    push <jmp.&MSVBVM60.__vbaExceptHandler>    ;  SE 处理程序安装
0044E19B   .  64:A1 0000000>mov eax,dword ptr fs:[0]
0044E1A1   .  50            push eax
0044E1A2   .  64:8925 00000>mov dword ptr fs:[0],esp
0044E1A9   .  B8 6C030000   mov eax,36C
0044E1AE   .  E8 0D76FBFF   call <jmp.&MSVBVM60.__vbaChkstk>
0044E1B3   .  53            push ebx
0044E1B4   .  56            push esi
0044E1B5   .  57            push edi
0044E1B6   .  8965 E8       mov dword ptr ss:[ebp-18],esp
0044E1B9   .  C745 EC 581B4>mov dword ptr ss:[ebp-14],11.00401B58      ;  '
0044E1C0   .  8B45 08       mov eax,dword ptr ss:[ebp+8]
0044E1C3   .  83E0 01       and eax,1
0044E1C6   .  8945 F0       mov dword ptr ss:[ebp-10],eax
0044E1C9   .  8B4D 08       mov ecx,dword ptr ss:[ebp+8]
0044E1CC   .  83E1 FE       and ecx,FFFFFFFE
0044E1CF   .  894D 08       mov dword ptr ss:[ebp+8],ecx
0044E1D2   .  C745 F4 00000>mov dword ptr ss:[ebp-C],0
0044E1D9   .  8B55 08       mov edx,dword ptr ss:[ebp+8]
0044E1DC   .  8B02          mov eax,dword ptr ds:[edx]
0044E1DE   .  8B4D 08       mov ecx,dword ptr ss:[ebp+8]
0044E1E1   .  51            push ecx
0044E1E2   .  FF50 04       call dword ptr ds:[eax+4]
0044E1E5   .  C745 FC 01000>mov dword ptr ss:[ebp-4],1
0044E1EC   .  C745 FC 02000>mov dword ptr ss:[ebp-4],2
0044E1F3   .  8B55 08       mov edx,dword ptr ss:[ebp+8]
0044E1F6   .  8B02          mov eax,dword ptr ds:[edx]
0044E1F8   .  8B4D 08       mov ecx,dword ptr ss:[ebp+8]
0044E1FB   .  51            push ecx
0044E1FC   .  FF90 20030000 call dword ptr ds:[eax+320]
0044E202   .  50            push eax
0044E203   .  8D55 B8       lea edx,dword ptr ss:[ebp-48]
0044E206   .  52            push edx                              ;
0044E207   .  FF15 08114000 call dword ptr ds:[<&MSVBVM60.__vbaObjSet>>;  MSVBVM60.__vbaObjSet，调用组件
```

首先看机器码生成部分

```
0044E20D   .  8985 CCFEFFFF mov dword ptr ss:[ebp-134],eax
0044E213   .  68 D0C04300   push 11.0043C0D0                       ;  字符“c”压栈，求 c 盘的序列号
0044E218   .  E8 A3AF0300   call 11.004891C0                       ;  机器码函数，跟进看看
```

`F7` 跟进 `0044E218`

```
004891C0   $  55            push ebp
004891C1   .  8BEC          mov ebp,esp
004891C3   .  83EC 18       sub esp,18
004891C6   .  68 C6574000   push <jmp.&MSVBVM60.__vbaExceptHandler>  ;  SE 处理程序安装
004891CB   .  64:A1 0000000>mov eax,dword ptr fs:[0]
004891D1   .  50            push eax
004891D2   .  64:8925 00000>mov dword ptr fs:[0],esp
004891D9   .  B8 50000000   mov eax,50
004891DE   .  E8 DDC5F7FF   call <jmp.&MSVBVM60.__vbaChkstk>
004891E3   .  53            push ebx
004891E4   .  56            push esi
004891E5   .  57            push edi
004891E6   .  8965 E8       mov dword ptr ss:[ebp-18],esp
004891E9   .  C745 EC D84E4>mov dword ptr ss:[ebp-14],11.00404ED8    ;  6
004891F0   .  C745 F0 00000>mov dword ptr ss:[ebp-10],0
004891F7   .  C745 F4 00000>mov dword ptr ss:[ebp-C],0
004891FE   .  C745 FC 01000>mov dword ptr ss:[ebp-4],1
00489205   .  8B55 08       mov edx,dword ptr ss:[ebp+8]
00489208   .  8D4D DC       lea ecx,dword ptr ss:[ebp-24]
0048920B   .  FF15 A0124000 call dword ptr ds:[<&MSVBVM60.__vbaStrCo>;  MSVBVM60.__vbaStrCopy
00489211   .  C745 FC 02000>mov dword ptr ss:[ebp-4],2
00489218   .  6A 01         push 1                                ; /OnErrEvent = Goto Address
0048921A   .  FF15 10114000 call dword ptr ds:[<&MSVBVM60.__vbaOnErr>; \__vbaOnError
00489220   .  C745 FC 03000>mov dword ptr ss:[ebp-4],3
00489227   .  8B45 DC       mov eax,dword ptr ss:[ebp-24]
0048922A   .  50            push eax                              ; /String    字符“c”压栈
0048922B   .  FF15 5C104000 call dword ptr ds:[<&MSVBVM60.__vbaLenBs>; \__vbaLenBstr 求字符长度
00489231   .  85C0          test eax,eax    检查 eax 是否为 0
00489233   .  0F84 C9000000 je 11.00489302  为 0 就走人
00489239   .  C745 FC 04000>mov dword ptr ss:[ebp-4],4
00489240   .  6A 01         push 1
00489242   .  8B4D DC       mov ecx,dword ptr ss:[ebp-24]
00489245   .  51            push ecx
00489246   .  68 18124400   push 11.00441218                       ;  将 11.00441218 的数据‘\’压栈
0048924B   .  6A 00         push 0
0048924D   .  FF15 80124000 call dword ptr ds:[<&MSVBVM60.__vbaInStr>;  MSVBVM60.__vbaInStr  检查是否有‘\’
00489253   .  83F8 01       cmp eax,1 是否为空
00489256   .  75 6A         jnz short 11.004892C2 没有'\’，那么跳走
................................省略部分代码
004892C2   >  C745 FC 09000>mov dword ptr ss:[ebp-4],9  跳到这里
004892C9   .  6A 01        push 1
004892CB   .  8B55 DC       mov edx,dword ptr ss:[ebp-24]
004892CE   .  52            push edx
004892CF   .  FF15 08134000 call dword ptr ds:[<&MSVBVM60.#616616>]      ;  MSVBVM60.rtcLeftCharBstr  取左边一个字符，即 C
004892D5   .  8BD0          mov edx,eax
004892D7   .  8D4D D4       lea ecx,dword ptr ss:[ebp-2C]
004892DA   .  FF15 28134000 call dword ptr ds:[<&MSVBVM60.__vbaStrMo>;  MSVBVM60.__vbaStrMove
004892E0   .  50            push eax
004892E1   .  68 90EB4300   push 11.0043EB90                       ; /:    压":\”入栈
004892E6   .  FF15 AC104000 call dword ptr ds:[<&MSVBVM60.__vbaStrCa>; \__vbaStrCat  将 c 和:\连接起来，即 c:\
004892EC   .  8BD0          mov edx,eax
004892EE   .  8D4D DC       lea ecx,dword ptr ss:[ebp-24]
004892F1   .  FF15 28134000 call dword ptr ds:[<&MSVBVM60.__vbaStrMo>;  MSVBVM60.__vbaStrMove 移动字串
004892F7   .  8D4D D4       lea ecx,dword ptr ss:[ebp-2C]
004892FA   .  FF15 68134000 call dword ptr ds:[<&MSVBVM60.__vbaFreeS>;  MSVBVM60.__vbaFreeStr 释放字串内存
00489300   >  EB 12         jmp short 11.00489314
00489302   >  C745 FC 0C000>mov dword ptr ss:[ebp-4],0C
00489309   .  33D2         xor edx,edx
0048930B   .  8D4D DC       lea ecx,dword ptr ss:[ebp-24]
0048930E   .  FF15 A0124000 call dword ptr ds:[<&MSVBVM60.__vbaStrCo>;  MSVBVM60.__vbaStrCopy
00489314   >  C745 FC 0E000>mov dword ptr ss:[ebp-4],0E
```

此前的大半部分都是在确认路径`c:\`是否正确，如果没有就添加，修正，

相当于`IIf(Right(App.Path, 1) = "\", App.Path, App.Path & "\") & App.EXEName & ".exe"`

```
0048931B   .  6A 00        push 0
0048931D   .  6A 00        push 0
0048931F   .  6A 00        push 0
00489321   .  6A 00        push 0
00489323   .  8D45 D8       lea eax,dword ptr ss:[ebp-28]
00489326   .  50            push eax
00489327   .  6A 00        push 0
00489329   .  6A 00        push 0
0048932B   .  8B4D DC       mov ecx,dword ptr ss:[ebp-24]
```



`0048932E . 51 push ecx`

`0048932F . 8D55 D4 lea edx,dword ptr ss:[ebp-2C]`

`00489332 . 52 push edx`

`00489333 . FF15 F0124000 call dword ptr ds:[<&MSVBVM60.__vbaStrTo>; MSVBVM60.__vbaStrToAnsi 转换字串为 Ansi 码`

`00489339 . 50 push eax`

`0048933A . E8 A12AFBFF call 11.0043BDE0 ; 获取磁盘序列号函数`

再跟进此函数可知程序时根据此函数 `kernel32.GetVolumeInformationA` 来获取 `c` 盘的序列号。不再跟进。

`0044E21D . 8BC8 mov ecx,eax`

`0044E21F . FF15 04114000 call dword ptr ds:[<&MSVBVM60.__vbaI4Abs>; 求绝对值，磁盘序列号变正值`

`0044E225 . 50 push eax`

`0044E226 . FF15 3C104000 call dword ptr ds:[<&MSVBVM60.__vbaStrI4>; 转换成 10 进制字符串`

`0044E22C . 8BD0 mov edx,eax ; 得到机器码`

`0044E22E . 8D4D C4 lea ecx,dword ptr ss:[ebp-3C] 将机器码存入 ss:[0012FAD8]`

`.......................省略部分代码`

`0044E39D > C785 40FEFFFF mov dword ptr ss:[ebp-1C0],0 ;`

`0044E3A7 > 8B4D C4 mov ecx,dword ptr ss:[ebp-3C] ; 从 0012FAD8 拿回机器码`

`0044E3AA . 51 push ecx ; 压栈`

`0044E3AB . FF15 84124000 call dword ptr ds:[<&MSVBVM60.__vbaR8Str>] ; 转为双精度浮点型`

`0044E3B1 . D9E1 fabs ; 求绝对值`

`0044E3B3 . DFE0 fstsw ax ; 把值送给通用寄存器 AX`

`0044E3B5 . A8 0D test al,0D ; 测试 al 中余数是否为空`

`0044E3B7 . 0F85 D44C0000 jnz 11.00453091 ; 为空就玩完，跳走了`

`0044E3BD . 83EC 08 sub esp,8 ; 把 0012F77C 的值减 8，及 esp=0012F774`

`0044E3C0 . DD1C24 fstp qword ptr ss:[esp] ; 弹栈指令，将 st0 弹出`

`0044E3C3 . FF15 FC124000 call dword ptr ds:[<&MSVBVM60.#614>] ; 求平方根`

`0044E3C9 . DD9D D0FEFFFF fstp qword ptr ss:[ebp-130] ; 弹栈指令，将 st0 弹出`

`0044E3CF . DD05 30184000 fld qword ptr ds:[401830] ; 载入浮点数 4141`

`0044E3D5 . DC8D D0FEFFFF fmul qword ptr ss:[ebp-130] ; 乘以 4141`

`0044E3DB . DFE0 fstsw ax ; 把值送给通用寄存器 AX`

`0044E3DD . A8 0D test al,0D ; 测试 al 中余数是否为空`

`0044E3DF . 0F85 AC4C0000 jnz 11.00453091 ; 为空就玩完，跳走了`

`0044E3E5 . FF15 3C134000 call dword ptr ds:[<&MSVBVM60.__vbaR8IntI4>; 取整数，得出注册码`

`0044E3EB . 8945 D4 mov dword ptr ss:[ebp-2C],eax ; 存放注册码，算法结束，和下面对应`

`0044E3EE . 8D4D C4 lea ecx,dword ptr ss:[ebp-3C] ; 下面释放内存`

`0044E3F1 . FF15 68134000 call dword ptr ds:[<&MSVBVM60.__vbaFreeStr>; MSVBVM60.__vbaFreeStr`

`0044E3F7 . 8D4D B8 lea ecx,dword ptr ss:[ebp-48]`

`0044E3FA . FF15 64134000 call dword ptr ds:[<&MSVBVM60.__vbaFreeObj>; MSVBVM60.__vbaFreeObj`

`.......................省略部分代码`

`0044E66C . 894A 0C mov dword ptr ds:[edx+C],ecx ; 访问注册表`

`0044E66F . 68 40C64300 push 11.0043C640 ; /flag`

`0044E674 . 68 34C64300 push 11.0043C634 ; |REG`

`0044E679 . 8B55 C4 mov edx,dword ptr ss:[ebp-3C] ; |`

`0044E67C . 52 push edx ; |AppName`

`0044E67D . FF15 D4124000 call dword ptr ds:[<&MSVBVM60.#689>] ; \\rtcGetSetting`

`0044E683 . 8945 A8 mov dword ptr ss:[ebp-58],eax`

`0044E686 . C745 A0 08000>mov dword ptr ss:[ebp-60],8`

`0044E68D . 8D45 A0 lea eax,dword ptr ss:[ebp-60]`

`0044E690 . 50 push eax`

`0044E691 . 8D4D 90 lea ecx,dword ptr ss:[ebp-70]`

`0044E694 . 51 push ecx`

检查注册表中 `reg` 是否标记为 `true`

`.......................省略部分代码`

`0044E809 . 8950 0C mov dword ptr ds:[eax+C],edx ; 访问注册表，检查是否注册`

`0044E80C . 68 D0C64300 push 11.0043C6D0 ; /Info`

`0044E811 . 68 E0C64300 push 11.0043C6E0 ; |UnlockCode`

`0044E816 . 8B45 C4 mov eax,dword ptr ss:[ebp-3C] ; |`

`0044E819 . 50 push eax ; |AppName`

`0044E81A . FF15 D4124000 call dword ptr ds:[<&MSVBVM60.#689>] ; \\rtcGetSetting`

`0044E820 . 8945 A8 mov dword ptr ss:[ebp-58],eax`

`0044E823 . C745 A0 08000>mov dword ptr ss:[ebp-60],8`

`0044E82A . 8D4D A0 lea ecx,dword ptr ss:[ebp-60]`

`0044E82D . 51 push ecx`

`0044E82E . 8D55 90 lea edx,dword ptr ss:[ebp-70] 和上面雷同`

`0044E831 . 52 push edx`

检查注册表中是否已经存放好注册码，注册信息在 `HKEY_CURRENT_USER\Software\VB and VBA Program Settings\HistoryCleaner\`

`.......................省略部分代码`

`0044E85F . 8D4D 90 lea ecx,dword ptr ss:[ebp-70]`

`0044E862 . 51 push ecx 进栈`

`0044E863 . 8D55 A0 lea edx,dword ptr ss:[ebp-60]`

`0044E866 . 52 push edx 进栈，开始对上面的“侦查”结果进行判断了`

`0044E867 . 6A 02 push 2 2 进栈`

`0044E869 . FF15 68104000 call dword ptr ds:[<&MSVBVM60.__vbaFreeV>; MSVBVM60.__vbaFreeVarList`

`0044E86F . 83C4 0C add esp,0C`

`0044E872 . C745 FC 0C000>mov dword ptr ss:[ebp-4],0C`

`0044E879 . 66:833D D0424>cmp word ptr ds:[4942D0],0FFFF ; 进行比较`

`0044E881 0F85 4C090000 jnz 11.0044F1D3 ; 关键跳，爆破位置，标志位修改，`

`0044E887 . C745 FC 0D000>mov dword ptr ss:[ebp-4],0D`

`0044E88E . 8B45 CC mov eax,dword ptr ss:[ebp-34]`

`0044E891 . 50 push eax`

`0044E892 . 8B4D D4 mov ecx,dword ptr ss:[ebp-2C] ; 还记得上面算法分析时这个位置的值吗？`

`0044E895 . 51 push ecx`

`0044E896 . FF15 3C104000 call dword ptr ds:[<&MSVBVM60.__vbaStrI4>; MSVBVM60.__vbaStrI4 转换成 10 进制字符串`

`0044E89C . 8BD0 mov edx,eax ; 注册码`

`0044E89E . 8D4D C4 lea ecx,dword ptr ss:[ebp-3C]`

`0044E8A1 . FF15 28134000 call dword ptr ds:[<&MSVBVM60.__vbaStrMo>; MSVBVM60.__vbaStrMove`

`0044E8A7 . 50 push eax`

`0044E8A8 . FF15 80114000 call dword ptr ds:[<&MSVBVM60.__vbaStrCm>; MSVBVM60.__vbaStrCmp`

`0044E8AE . F7D8 neg eax`

`0044E8B0 . 1BC0 sbb eax,eax`

`0044E8B2 . F7D8 neg eax`

`0044E8B4 . F7D8 neg eax`

`0044E8B6 . 66:8985 CCFEF>mov word ptr ss:[ebp-134],ax`

`0044E8BD . 8D4D C4 lea ecx,dword ptr ss:[ebp-3C]`

`0044E8C0 . FF15 68134000 call dword ptr ds:[<&MSVBVM60.__vbaFreeS>; MSVBVM60.__vbaFreeStr`

`0044E8C6 . 0FBF95 CCFEFF>movsx edx,word ptr ss:[ebp-134]`

`0044E8CD . 85D2 test edx,edx`

`0044E8CF 0F84 25010000 je 11.0044E9FA ; 关键跳，跳向注册成功`

程序的算法比较简单，但是我们可以从多个方面来分析这个程序，就非常有意思。从程序启动后注册处来下手追码，随便输入假码，会弹出`"invalid unlock code. try again"`，

就可以从这里下手，从这里追码相对简单这里就不在详述。

这个程序没注册时，标题显示的时`“Unregistered Copy“`，但是从字符串中试搜索不到的，因为这个是它 `form` 的原始 `caption`，当判断注册后才会修改 `form.caption="registered Copy"`，

这样只能搜索到`”registered Copy“`这个字串。并且这样追码的难点是程序做了这样的设置，如果已经注册，就在注册表中将 `reg` 键值的属性写为 `ture`，如果没有就写为 `false`，启动时判断

这个值是否为 `true`，如果不是它就不进行下一步的注册码的比较了，直接跳走，那么我们也就追不到名码了。但是启动后从注册处下手，它会对输入的假码进行直接的比较，那么也就很容易追出来。

所以如果仅仅为了爆破还是从注册处追码较好。

至此，这个程序的算法就分析完了，用 `vb` 还原一下就是：`注册码=Int(Sqr(Abs(CDbl(机器码))) * 4141)`

`vb` 注册机源码：

```vb
Option Explicit
```



```vba
Private Declare Function GetVolumeInformation Lib "kernel32" Alias "GetVolumeInformationA" (ByVal lpRootPathName As String, ByVal lpVolumeNameBuffer As String, ByVal nVolumeNameSize As Long, lpVolumeSerialNumber As Long, lpMaximumComponentLength As Long, lpFileSystemFlags As Long, ByVal lpFileSystemNameBuffer As String, ByVal nFileSystemNameSize As Long) As Long  

Private Sub Drive\_Change()  

   Dim Volume As String, SysName As String  

   Dim SerialNum As Long, SysFlags As Long, ComponentLength As Long, Res As Long  

   Dim DrvName As String  

   Volume = String(256, 0)  

   SysName = String(256, 0)  

   DrvName = "c:\\"  

   Res = GetVolumeInformation(DrvName, Volume, 255, SerialNum, ComponentLength, SysFlags, SysName, 255)  

   If Res = 0 Then  
      Text1.Text = "不能得到磁盘信息"  
   Else  
      Text1.Text = Abs(SerialNum)  
      Text2.Text = Int(Sqr(Abs(CDbl(Text1.Text))) \* 4141)  
   End If  

End Sub  

Private Sub Form\_Load()  

   Call Drive\_Change  

End Sub
```

画 2 个文本框，贴上这个代码，就成了。

\--------------------------------------------------------------------------------

【经验总结】

凡事自己多琢磨，没有困难和挫折就没有动力奋力向前。从多方位考虑问题将使问题更加清晰
