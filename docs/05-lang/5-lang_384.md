
# 飞雪桌面日历 V0.7 注册算法

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-402.htm`](https://www.pediy.com/kssd/pediy07/pediy7-402.htm)

飞雪桌面日历 V0.7

下载地址：`http://www.shareware.cn/SoftDown.asp?ID=49846`

在该网站还有留言：

没有注册机 \[xxo00 9/9\] 来自：`16.72.63.*`

好用！ \[xiaoshitou 9/1\] 来自：`106.205.135.*`

真的很有味道，想在这个网站找到注册机！！！！

本机机器码：`779910`

`Upx` 加壳

`VB6` 制作

注册名：`wofan`

注册码：`123456`

真注册码：`171580319`

它有一个严重的缺陷：

右击它的托盘图标，会显示：`FXRL.exe` 遇到问题需要关闭，……

填好注册信息，按确定，在一个 `Label` 中提示：注册码成功写入，下次动行后生效。

写在那里呢？

在目录下找到：`FXSYS.INI`

发现里面是注册信息。

在 `OD` 中查找：

```
004505F5    .   68 A8554100     push FXRL.004155A8    ;  UNICODE "FXSYS\\FXSYS.INI"  就是它，在这里 F2 下断
……
00450745    .   FF15 64054E00                   call dword ptr ds:[<&MSVBVM50.__vba>;  MSVBVM50.__vbaStrVarVal
0045074B    .   50                          push eax－－－－－－－－－－－－机器码：779910
0045074C    .   FF15 98064E00                   call dword ptr ds:[<&MSVBVM50.#581>>;  MSVBVM50.rtcR8ValFromBstr
00450752    .   DC1D 00194000                   fcomp qword ptr ds:[401900]
……
00450875    .   8D55 80                     lea edx,dword ptr ss:[ebp-80]
00450878    .   52                          push edx
00450879    .   FF15 64054E00                   call dword ptr ds:[<&MSVBVM50.__vba>;  MSVBVM50.__vbaStrVarVal
0045087F    .   50                          push eax－－－－－－－－－－－－－－－注册名：wofan
00450880    .   FF15 74044E00                   call dword ptr ds:[<&MSVBVM50.#631>>;  MSVBVM50.rtcMidCharBstr
00450886    .   8BD0                       mov edx,eax
00450888    .   8D8D 7CFFFFFF                 lea ecx,dword ptr ss:[ebp-84]
0045088E    .   FF15 3C064E00                   call dword ptr ds:[<&MSVBVM50.__vba>;  MSVBVM50.__vbaStrMove
00450894    .   50                          push eax
00450895    .   FF15 B4034E00                   call dword ptr ds:[<&MSVBVM50.#516>>;  MSVBVM50.rtcAnsiValueBstr--取 ASCII 码
0045089B    .   66:8985 20FFFFFF              mov word ptr ss:[ebp-E0],ax－－－－－－AX=77（w）
004508A2    .   C785 18FFFFFF 02000000        mov dword ptr ss:[ebp-E8],2
004508AC    .   8D45 B0                     lea eax,dword ptr ss:[ebp-50]
004508AF    .   50                          push eax
004508B0    .   8D8D 18FFFFFF                 lea ecx,dword ptr ss:[ebp-E8]
004508B6    .   51                          push ecx
004508B7    .   8D95 58FFFFFF                 lea edx,dword ptr ss:[ebp-A8]
004508BD    .   52                          push edx
004508BE    .   FF15 68034E00                   call dword ptr ds:[<&MSVBVM50.__vba>;  MSVBVM50.__vbaVarSub-----Sub
**********************************************************************

1E240(即注册码 123456 的十六进制形式）－77（注册名的第一个字符的 ASCII 码）＝0x1E1C9

**********************************************************************

004508C4    .   8BD0                       mov edx,eax
004508C6    .   8D4D B0                     lea ecx,dword ptr ss:[ebp-50]
004508C9    .   FF15 7C034E00                   call dword ptr ds:[<&MSVBVM50.__vba>;  MSVBVM50.__vbaVarMove
004508CF    .   8D85 7CFFFFFF                 lea eax,dword ptr ss:[ebp-84]
004508D5    .   50                          push eax
004508D6    .   8D4D 80                     lea ecx,dword ptr ss:[ebp-80]
004508D9    .   51                          push ecx
004508DA    .   6A 02                        push 2
004508DC    .   FF15 C0054E00                   call dword ptr ds:[<&MSVBVM50.__vba>;  MSVBVM50.__vbaFreeStrList
004508E2    .   83C4 0C                     add esp,0C
004508E5    .   8D8D 68FFFFFF                 lea ecx,dword ptr ss:[ebp-98]
004508EB    .   FF15 84034E00                   call dword ptr ds:[<&MSVBVM50.__vba>;  MSVBVM50.__vbaFreeVar
004508F1    .   C745 FC 11000000              mov dword ptr ss:[ebp-4],11
004508F8    .   66:C785 F8FEFFFF 0200        mov word ptr ss:[ebp-108],2
00450901    .   66:C785 FCFEFFFF FFFF        mov word ptr ss:[ebp-104],0FFFF
0045090A    .   66:8B55 94                    mov dx,word ptr ss:[ebp-6C]
0045090E    .   66:8955 C0                    mov word ptr ss:[ebp-40],dx
00450912    .   EB 0F                        jmp short FXRL.00450923
00450914    >  66:8B45 C0                    mov ax,word ptr ss:[ebp-40]－－－－长跳到这里，word ptr ss:[ebp-40]＝5
是注册名长度
00450918    .   66:0385 FCFEFFFF              add ax,word ptr ss:[ebp-104]－－－word ptr ss:[ebp-104]＝FFFF （也就是
－1）
0045091F    .   66:8945 C0                    mov word ptr ss:[ebp-40],ax
00450923    >  66:8B4D C0                    mov cx,word ptr ss:[ebp-40]－－－－－－word ptr ss:[ebp-40]是注册名长
度 5
00450927    .   66:3B8D F8FEFFFF              cmp cx,word ptr ss:[ebp-108]－－word ptr ss:[ebp-108]＝2（倒数取注册名
ASCII，直到前 1 位）我这里注册名长度为 5 位， 那么就要循环四次。
0045092E    .   0F8C F4000000                 jl FXRL.00450A28
00450934    .   C745 FC 12000000              mov dword ptr ss:[ebp-4],12
0045093B    .   C785 70FFFFFF 01000000        mov dword ptr ss:[ebp-90],1
00450945    .   C785 68FFFFFF 02000000        mov dword ptr ss:[ebp-98],2
0045094F    .   8D95 68FFFFFF                 lea edx,dword ptr ss:[ebp-98]
00450955    .   52                          push edx
00450956    .   0FBF45 C0                    movsx eax,word ptr ss:[ebp-40]
0045095A    .   50                          push eax
0045095B    .   8D4D D0                     lea ecx,dword ptr ss:[ebp-30]
0045095E    .   51                          push ecx
0045095F    .   8D95 7CFFFFFF                 lea edx,dword ptr ss:[ebp-84]
00450965    .   52                          push edx
00450966    .   FF15 64054E00                   call dword ptr ds:[<&MSVBVM50.__vba>;  MSVBVM50.__vbaStrVarVal
0045096C    .   50                          push eax
0045096D    .   FF15 74044E00                   call dword ptr ds:[<&MSVBVM50.#631>>;  MSVBVM50.rtcMidCharBstr
00450973    .   8BD0                       mov edx,eax
00450975    .   8D8D 78FFFFFF                 lea ecx,dword ptr ss:[ebp-88]
0045097B    .   FF15 3C064E00                   call dword ptr ds:[<&MSVBVM50.__vba>;  MSVBVM50.__vbaStrMove
00450981    .   50                          push eax
00450982    .   FF15 B4034E00                   call dword ptr ds:[<&MSVBVM50.#516>>;  MSVBVM50.rtcAnsiValueBstr
00450988    .   66:8985 04FFFFFF              mov word ptr ss:[ebp-FC],ax－－－－－－－－－－－ax=6E(n)注册名的末位
0045098F    .   8D45 84                     lea eax,dword ptr ss:[ebp-7C]
00450992    .   50                          push eax
00450993    .   8D4D 80                     lea ecx,dword ptr ss:[ebp-80]
00450996    .   51                          push ecx
00450997    .   FF15 64054E00                   call dword ptr ds:[<&MSVBVM50.__vba>;  MSVBVM50.__vbaStrVarVal
0045099D    .   50                          push eax－－－－－－－－－－－－－－－机器码，PUSH
0045099E    .   FF15 98064E00                   call dword ptr ds:[<&MSVBVM50.#581>>;  MSVBVM50.rtcR8ValFromBstr
004509A4    .   66:8B95 04FFFFFF              mov dx,word ptr ss:[ebp-FC]
004509AB    .   66:83EA 32                    sub dx,32－－－－－－－－－－－－－－－6E-32=3C
004509AF    .   0FBFC2                       movsx eax,dx
004509B2    .   8985 D8FEFFFF                 mov dword ptr ss:[ebp-128],eax
004509B8    .   DB85 D8FEFFFF                 fild dword ptr ss:[ebp-128]－－－－－3C----->60,装整数
```



## 汇编指令分析与逻辑推导

### 1. 核心运算逻辑片段

```
004509BE . DEC9 fmulp st(1),st －－－－－－－－浮点乘法：779910×60=46794600

004509C0 . DD9D 20FFFFFF fstp qword ptr ss:[ebp-E0]－－浮点之上托出栈，保存在 qword ptr ss:[ebp-E0]

004509C6 . C785 18FFFFFF 05000000 mov dword ptr ss:[ebp-E8],5

004509D0 . 8D4D B0 lea ecx,dword ptr ss:[ebp-50]
004509D3 . 51 push ecx
004509D4 . 8D95 18FFFFFF lea edx,dword ptr ss:[ebp-E8]
004509DA . 52 push edx
004509DB . 8D85 58FFFFFF lea eax,dword ptr ss:[ebp-A8]
004509E1 . 50 push eax

004509E2 . FF15 68034E00 call dword ptr ds:[<&MSVBVM50.__vba>; MSVBVM50.__vbaVarSub----Sub(被减数的初值是：123337（0x1E1C9）,记住：0x1E1C9=123456-0x77（注册码－注册名的第一个 ASCII 码）

004509E8 . 8BD0 mov edx,eax
004509EA . 8D4D B0 lea ecx,dword ptr ss:[ebp-50]
004509ED . FF15 7C034E00 call dword ptr ds:[<&MSVBVM50.__vba>; MSVBVM50.__vbaVarMove

004509F3 . 8D8D 78FFFFFF lea ecx,dword ptr ss:[ebp-88]
004509F9 . 51 push ecx
004509FA . 8D95 7CFFFFFF lea edx,dword ptr ss:[ebp-84]
00450A00 . 52 push edx
00450A01 . 8D45 80 lea eax,dword ptr ss:[ebp-80]
00450A04 . 50 push eax
00450A05 . 6A 03 push 3
00450A07 . FF15 C0054E00 call dword ptr ds:[<&MSVBVM50.__vba>; MSVBVM50.__vbaFreeStrList

00450A0D . 83C4 10 add esp,10
00450A10 . 8D8D 68FFFFFF lea ecx,dword ptr ss:[ebp-98]
00450A16 . FF15 84034E00 call dword ptr ds:[<&MSVBVM50.__vba>; MSVBVM50.__vbaFreeVar
00450A1C . C745 FC 13000000 mov dword ptr ss:[ebp-4],13
00450A23 .^ E9 ECFEFFFF jmp FXRL.00450914－－－－－－－－－－－－一个长跳
```

以上循环综述如下：

- 注册名不支持中文，取注册名的 ASCII - `0x32` 参加运算，倒着取（直到只剩下第一个字母为止）。
- 这里注册名是：`wofan`，它的长度是 5，那么就在这里循环 4 次。

ASCII 码计算过程：

| 字符 | ASCII |
| :--- | :--- |
| w | 76 (`0x4E`) |
| o | 6F (`0x6F`) |
| f | 66 (`0x66`) |
| a | 61 (`0x61`) |
| n | 6E (`0x6E`) |

机器码：`779910`

运算推导：
1. `(6E-32)*779910 = 46794600` -> `123337（0x1E1C9）－46794600 ＝－46671263`
2. `(61-32)*779910 = 36655770` -> `－46671263-36655770 = -83327033`
3. `(66-32)*779910 = 40555320` -> `-83327033-40555320 = -123882353`
4. `(6F-32)*779910 = 47574510` -> `-123853-47574510 = -171456863`

最后得到：`-171456863`

### 2. 后续汇编指令

```
00450A28 > C745 FC 14000000 mov dword ptr ss:[ebp-4],14
00450A2F . C785 30FFFFFF 01000000 mov dword ptr ss:[ebp-D0],1
00450A39 . C785 28FFFFFF 02000000 mov dword ptr ss:[ebp-D8],2
00450A43 . 8D4D B0 lea ecx,dword ptr ss:[ebp-50]
00450A46 . 51 push ecx
00450A47 . 8D95 28FFFFFF lea edx,dword ptr ss:[ebp-D8]
00450A4D . 52 push edx
00450A4E . 8D85 68FFFFFF lea eax,dword ptr ss:[ebp-98]
00450A54 . 50 push eax
00450A55 . FF15 68034E00 call dword ptr ds:[<&MSVBVM50.__vba>; MSVBVM50.__vbaVarSub
```

计算：`-171456863 - 1 = -171456864`

```
00450A5B . 50 push eax
00450A5C . FF15 D0054E00 call dword ptr ds:[<&MSVBVM50.__vba>; MSVBVM50.__vbaR8Var
00450A62 . DD5D C8 fstp qword ptr ss:[ebp-38]
```

此时 `st = -171456864.00000000000`

```
00450A65 > 9B wait
...
00450AE8 . C3 retn
```

### 3. 最终校验逻辑

```
004B0357 . DC0D B8314000 fmul qword ptr ds:[4031B8]－－－
```

状态：
- `st = -171456864.00000000000`
- `ds:[004031B8] = 205.0000000000000`

```
004B035D . DC1D C0314000 fcomp qword ptr ds:[4031C0]
```

状态：
- `st = -3.5148657120000000000e+10` (即 `-171456864 * 205 = -35148657120`)
- `ds:[004031C0] = -205.0000000000000` 
*(注：此处原文逻辑似乎隐含比较的是最终结果与目标值的匹配，原文标注 fcomp 比较的是 004031C0，通常注册码验证最后一步是判断计算结果是否等于特定值或0。根据下文 `fstsw ax` 和 `test ah,40`，这是浮点状态字检查。原文中 `fcomp` 后接 `fstsw` 通常用于比较结果。)*

```
004B0363 . DFE0 fstsw ax
004B0365 . F6C4 40 test ah,40
004B0368 . 74 07 je short FXRL.004B0371----------这里 Nop 掉，就完美爆破！！！
004B036A . C605 85304D00 01 mov byte ptr ds:[4D3085],1－－－－注册标志置 1
```

### 4. 注册机算法推导

**问题：** 如何写这个注册机？

**输入数据：**
- 注册名：`w o f a n`
- 相应的 ASCII 码：`76 6F 66 61 6E`

**计算公式推导：**

原始方程：
`((注册码-77)－(6E-32)*779910-(61-32)*779910-(66-32)*779910-(6F-32)*779910－1)*205 = -205`

简化算式：
`注册码 = 0x77 + (0x6E+0x61+0x66+0x6F-4*0x32)*779910`
*(注：这里的 `205` 在除以 `205` 后变成 `-1`，再移项抵消，或者原式中有隐含的除以205步骤，原文称“可以看出这个 205 其实是个无关紧要的数”)*

解方程得到：`171580319`

**结论：** 这就是我的注册码！！！

**签名：**
`13:23 2005-9-23`
`by wofan[OCN]`

### 5. VB6 注册机源码

```vb
Option Explicit 

'本软件不支中文名注册册，右击它的托盘图标，会出错！ 
'为不不出错，请设置软件启动，不在托盘上显示图标！ 
'wofan[OCN] 属于中国破解组织：网眼天下 

Private Sub Command1_Click() 
    Dim name As String 
    Dim i As Integer 
    Dim machine As Double 
    Dim regcode As Double 

    If Text1.Text = "" Then 
        MsgBox "please paste then machine code!", vbOKOnly, "wofan" 
        Text1.SetFocus 
        Exit Sub 
    End If 

    If Text2.Text = "" Then 
        MsgBox "please input you regname!", vbOKOnly, "wofan" 
        Text2.SetFocus 
        Exit Sub 
    End If 

    machine = CDbl(Trim(Text1.Text)) 
    name = Trim(Text2.Text) 

    For i = 1 To Len(name) 
        If i = 1 Then 
            regcode = Asc(Mid(name, i, 1)) 
        Else 
            regcode = regcode + (Asc(Mid(name, i, 1)) - &H32) * machine 
        End If 
    Next i 
    
    Text3.Text = CStr(regcode) 
End Sub
```

`4dd466d0e1e0cc123afbe504da2462ca`
