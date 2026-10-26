
# [原创]History Cleaner v1.0算法分析

- 【文章标题】: History Cleaner v1.0算法分析
- 【文章作者】: daokers
- 【作者邮箱】: daokers@qq.com
- 【作者主页】: cb9K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3c8S2L8$3E0W2M7Y4y4Q4x3X3g2U0L8$3@1`.
- 【作者QQ号】: 94788078
- 【软件名称】: History Cleaner
- 【软件大小】: 600kb
- 【下载地址】: b4fK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3S2A6M7%4c8G2M7Y4W2U0L8r3g2S2L8X3g2J5i4K6u0W2j5$3!0E0i4K6u0r3K9r3y4$3x3g2)9J5k6h3g2^5k6b7`.`.
- 【加壳方式】: 无壳
- 【保护方式】: 注册码
- 【编写语言】: Microsoft Visual Basic  6.0
- 【使用工具】: OD
- 【操作平台】: windows xp sp2
- 【软件介绍】: History Cleaner是当前最为先进的网络残留和cookie
- 【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

## 详细过程

History Cleaner是一个系统和游览器历史清除工具，vb语言编写，注册码保护，根据机器码来生成注册码。比较函数难下手，这里从字符如手。

启动前，搜索‘Registered Copy’之后段首下断，好处是可以直接爆破，无需注册码，启动后从注册处下手追码相对容易些。断下后来到这里。

```
0044E190   > \55            push ebp                                   ;  段首，一切罪恶都是从这里开始
0044E191   .  8BEC          mov ebp,esp
0044E193   .  83EC 18       sub esp,18
0044E196   .  68 C6574000   push <jmp.&MSVBVM60.\_\_vbaExceptHandler>    ;  SE 处理程序安装
0044E19B   .  64:A1 0000000>mov eax,dword ptr fs:[0]
0044E1A1   .  50            push eax
0044E1A2   .  64:8925 00000>mov dword ptr fs:[0],esp
0044E1A9   .  B8 6C030000   mov eax,36C
0044E1AE   .  E8 0D76FBFF   call <jmp.&MSVBVM60.\_\_vbaChkstk>
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
0044E206   .  52            push edx                                   ;
0044E207   .  FF15 08114000 call dword ptr ds:[<&MSVBVM60.\_\_vbaObjSet>>;  MSVBVM60.\_\_vbaObjSet，调用组件
```

首先看机器码生成部分

```
0044E20D   .  8985 CCFEFFFF mov dword ptr ss:[ebp-134],eax
0044E213   .  68 D0C04300   push 11.0043C0D0                         ;  字符“c”压栈，求c盘的序列号
0044E218   .  E8 A3AF0300   call 11.004891C0                         ;  机器码函数，跟进看看
```

F7跟进0044E218

```
004891C0   $  55            push ebp
004891C1   .  8BEC          mov ebp,esp
004891C3   .  83EC 18       sub esp,18
004891C6   .  68 C6574000   push <jmp.&MSVBVM60.\_\_vbaExceptHandler>  ;  SE 处理程序安装
004891CB   .  64:A1 0000000>mov eax,dword ptr fs:[0]
004891D1   .  50            push eax
004891D2   .  64:8925 00000>mov dword ptr fs:[0],esp
004891D9   .  B8 50000000   mov eax,50
004891DE   .  E8 DDC5F7FF   call <jmp.&MSVBVM60.\_\_vbaChkstk>
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
0048920B   .  FF15 A0124000 call dword ptr ds:[<&MSVBVM60.\_\_vbaStrCo>;  MSVBVM60.\_\_vbaStrCopy
00489211   .  C745 FC 02000>mov dword ptr ss:[ebp-4],2
00489218   .  6A 01         push 1                                   ; /OnErrEvent = Goto Address
0048921A   .  FF15 10114000 call dword ptr ds:[<&MSVBVM60.\_\_vbaOnErr>; \\_\_vbaOnError
00489220   .  C745 FC 03000>mov dword ptr ss:[ebp-4],3
00489227   .  8B45 DC       mov eax,dword ptr ss:[ebp-24]
0048922A   .  50            push eax                                 ; /String    字符“c”压栈
0048922B   .  FF15 5C104000 call dword ptr ds:[<&MSVBVM60.\_\_vbaLenBs>; \\_\_vbaLenBstr 求字符长度
00489231   .  85C0          test eax,eax    检查eax是否为0
00489233   .  0F84 C9000000 je 11.00489302  为0就走人
00489239   .  C745 FC 04000>mov dword ptr ss:[ebp-4],4
00489240   .  6A 01         push 1
00489242   .  8B4D DC       mov ecx,dword ptr ss:[ebp-24]
00489245   .  51            push ecx
00489246   .  68 18124400   push 11.00441218                         ;  将11.00441218的数据‘\’压栈
0048924B   .  6A 00         push 0
0048924D   .  FF15 80124000 call dword ptr ds:[<&MSVBVM60.\_\_vbaInStr>;  MSVBVM60.\_\_vbaInStr  检查是否有‘\’
00489253   .  83F8 01       cmp eax,1 是否为空
00489256   .  75 6A         jnz short 11.004892C2 没有'\'，那么跳走
................................省略部分代码
004892C2   >  C745 FC 09000>mov dword ptr ss:[ebp-4],9  跳到这里
004892C9   .  6A 01         push 1
004892CB   .  8B55 DC       mov edx,dword ptr ss:[ebp-24]
004892CE   .  52            push edx
004892CF   .  FF15 08134000 call dword ptr ds:[<&MSVBVM60.#616>]     ;  MSVBVM60.rtcLeftCharBstr  取左边一个字符，即C
004892D5   .  8BD0          mov edx,eax
004892D7   .  8D4D D4       lea ecx,dword ptr ss:[ebp-2C]
004892DA   .  FF15 28134000 call dword ptr ds:[<&MSVBVM60.\_\_vbaStrMo>;  MSVBVM60.\_\_vbaStrMove
004892E0   .  50            push eax
004892E1   .  68 90EB4300   push 11.0043EB90                         ; /:    压":\"入栈
004892E6   .  FF15 AC104000 call dword ptr ds:[<&MSVBVM60.\_\_vbaStrCa>; \\_\_vbaStrCat  将c和:\连接起来，即c:\
004892EC   .  8BD0          mov edx,eax
004892EE   .  8D4D DC       lea ecx,dword ptr ss:[ebp-24]
004892F1   .  FF15 28134000 call dword ptr ds:[<&MSVBVM60.\_\_vbaStrMo>;  MSVBVM60.\_\_vbaStrMove 移动字串
004892F7   .  8D4D D4       lea ecx,dword ptr ss:[ebp-2C]
004892FA   .  FF15 68134000 call dword ptr ds:[<&MSVBVM60.\_\_vbaFreeS>;  MSVBVM60.\_\_vbaFreeStr 释放字串内存
00489300   >  EB 12         jmp short 11.00489314
00489302   >  C745 FC 0C000>mov dword ptr ss:[ebp-4],0C
00489309   .  33D2          xor edx,edx
0048930B   .  8D4D DC       lea ecx,dword ptr ss:[ebp-24]
0048930E   .  FF15 A0124000 call dword ptr ds:[<&MSVBVM60.\_\_vbaStrCo>;  MSVBVM60.\_\_vbaStrCopy
00489314   >  C745 FC 0E000>mov dword ptr ss:[ebp-4],0E
```

此前的大半部分都是在确认路径`c:\`是否正确，如果没有就添加，修正，

相当于`IIf(Right(App.Path, 1) = "\", App.Path, App.Path & "\") & App.EXEName & ".exe"`

```
0048931B   .  6A 00         push 0
0048931D   .  6A 00         push 0
0048931F   .  6A 00         push 0
00489321   .  6A 00         push 0
00489323   .  8D45 D8       lea eax,dword ptr ss:[ebp-28]
00489326   .  50            push eax
00489327   .  6A 00         push 0
00489329   .  6A 00         push 0
0048932B   .  8B4D DC       mov ecx,dword ptr ss:[ebp-24]
0048932E   .  51            push ecx
0048932F   .  8D55 D4       lea edx,dword ptr ss:[ebp-2C]
00489332   .  52            push edx
00489333   .  FF15 F0124000 call dword ptr ds:[<&MSVBVM60.\_\_vbaStrTo>;  MSVBVM60.\_\_vbaStrToAnsi    转换字串为Ansi码
00489339   .  50            push eax
```
