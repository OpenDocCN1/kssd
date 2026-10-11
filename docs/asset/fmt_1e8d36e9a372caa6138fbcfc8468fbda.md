
## 汇编代码片段

```
00435130   .  51                push    ecx  
00435131   .  6A 02             push    2  
00435133   .  FF15 00744400 call    [< &msvbvm50.__vbaFreeStrList>]   ;  msvbvm50.__vbaFreeStrList  
00435139   .  83C4 0C           add     esp, 0C  
0043513C   .  68 DC944000      push    004094DC  
00435141   .  68 B4944000      push    004094B4  
00435146   .  FFD7            call    edi  
00435148   .  8BD0            mov     edx, eax                            ;(UNICODE "del in.bat")  
0043514A   .  8D4D C8         lea     ecx, [ebp-38]  
0043514D   .  FFD6            call    esi  
0043514F   .  50              push    eax  
00435150   .  6A 01            push    1  
00435152   .  68 D8754000      push    004075D8  
00435157   .  FFD3            call    ebx                                   ;msvbvm50.__vbaPrintFile  
00435159   .  83C4 0C          add     esp, 0C  
0043515C   .  8D4D C8          lea     ecx, [ebp-38]  
0043515F   .  FF15 78744400  call    [< &msvbvm50.__vbaFreeStr>]         ;   
00435165   .  6A 01            push    1  
00435167   .  FF15 20734400   call    [< &msvbvm50.__vbaFileClose>]     ;  msvbvm50.__vbaFileClose  
0043516D   .  8D55 E4         lea     edx, [ebp-1C]  
00435170   .  8D45 80         lea     eax, [ebp-80]  
00435173   .  6A 00            push    0  
00435175   .  50              push    eax  
00435176   .  8955 88         mov     [ebp-78], edx  
00435179   .  C745 80 08400> mov     dword ptr [ebp-80], 4008  
00435180   .  FF15 70734400   call    [< &msvbvm50.rtcShell>]           ;  msvbvm50.rtcShell  
                                                                          shell 执行文件 In.bat  
00435186   .  DDD8            fstp    st  
00435188   .  FF15 90724400   call    [< &msvbvm50.__vbaEnd>]           ;  msvbvm50.__vbaEnd  结束  
```

### 9）`In.bat` 文件内容

（注：我调试时把文件名改为了 `1234.exe`）

```batch
:START  
del 1234.EXE  
if exist 1234.EXE GOTO START  
del in.bat  
```

这个文件有时删除不了那个可执行文件，尤其是文件名中带括号的，原因不知，可能是全角和半角的事吧。

### 10）注册机算法：`VB` 做得

```vb
'超级函数表达式运算器 v20 注册机  
'ccrlei  
'2006-2-8  

Private Sub Command2_Click()  
Unload Me  
End Sub  

Private Sub Command1_Click()  
Dim SnWord, SJqm, SJqm1, Sntemp, Sntemp1, Sntemp2, Sntemp3 As String  
Dim i, j As Integer  
Dim Dsn, SnLen1, JqmLen, JqmLenL, JqmLenR As Integer  
Dim Sq, Sh, Stemp As String  

SJqm = Text1.Text               '机器码变量  
JqmLen = Len(SJqm) - 1         '机器码长度  
JqmLenL = InStr(1, SJqm, "\_", vbBinaryCompare) - 1 '"\_"左边字符串长度  
JqmLenR = JqmLen - JqmLenL       '"\_"右边字符串长度  

SJqm = Mid(SJqm, 1, JqmLenL) + Mid(SJqm, JqmLenL + 2, JqmLenR) '去掉“\_”  
Sq = Mid(SJqm, 1, 8)    '程序退出时检验注册码要用机器码前八位  
Sh = Mid(SJqm, JqmLenL + 1, JqmLenR)  '程序退出时检验注册码要用机器码原“\_”后字符串  

If Len(Sh) > 9 Then                  '机器码后段位数大于 9 位时取 8 位，否则取 9 位  
    Sh = Mid(SJqm, JqmLen - 7, 8)  
End If  

Stemp = CDbl(Sq) And CDbl(Sh)    '前八位和后 8 或 9 位逻辑与  
SJqm1 = SJqm + Stemp   '新生成的机器码  

i = 0  
For i = 1 To Len(SJqm1) - 5        '机器码前 10 位每 2 位对应一个注册码  
    If i <= 5 Then  
        SnWord = Mid(SJqm1, i * 2 - 1, 2)  
    Else  
        SnWord = Mid(SJqm1, 5 + i, 1)  
    End If  
Dsn = Int(Val(SnWord) * 2.5 - 0.5 + 65 + 1)     '转换 详见上介绍  
Sntemp = Sntemp + Chr(Dsn)                     '连接  
Next i  

SnLen1 = Len(Sntemp)      '新注册码长度  

If SnLen1 / 2 <> Int(SnLen1 / 2) Then                 '根据注册码位数的奇偶性重新排序  
    i = 0  
    For i = 1 To SnLen1  
        If i / 2 <> Int(i / 2) Then  
            Sntemp1 = Sntemp1 + Mid(Sntemp, i, 1)  
        Else  
            Sntemp2 = Mid(Sntemp, i, 1) + Sntemp2  
        End If  
    Next i  
    Sntemp3 = Sntemp1 + Sntemp2  
    sntemp4 = Mid(Sntemp3, 1, Int(SnLen1 / 2) + 1) + "CJHBYS" + Mid(Sntemp3, Int(SnLen1 / 2) + 2 _  
    , SnLen1 - Int(SnLen1 / 2) - 1)  
Else  
    i = 0  
    For i = 1 To SnLen1  
        If i / 2 = Int(i / 2) Then  
            Sntemp1 = Sntemp1 + Mid(Sntemp, i, 1)  
        Else  
            Sntemp2 = Mid(Sntemp, i, 1) + Sntemp2  
        End If  
    Next i  
    Sntemp3 = Sntemp1 + Sntemp2  
    sntemp4 = Mid(Sntemp3, 1, SnLen1 / 2) + "CJHBYS" + Mid(Sntemp3, SnLen1 / 2 + 1, SnLen1 / 2)    
    '插入检验所需的“CJHBYS”  
End If  

Text3.Text = sntemp4  
End Sub  

Private Sub Form_Load()  
Text1.Text = "22222222222854720884_754026158"    '本机机器码  
'Text1.Text = "22222222222257244519_3438081953"  
Text3.Text = ""          '注册码  
End Sub  
```

### 11）其他

安装完软件，第一次运行后程序会写入：

1. 注册表 `Software\Microsoft\htoolse\Data`
   - `\w1` `24EC652` 由 `hex(安装日期*999-234)` 得来
   - `\w2` `20D2118`     `hex( 最后一次运行日期 * 888-456)`
   - `\w3` `20D2118`     `hex((最后一次运行日期*999 ) Xor (最后一次运行日期*888 ))`

写文件： `"C:\WINNT\system32\config\cdelisub.hld"`

`"24EC652","20D2118","4449756"`     各项内容同上

并且这些内容卸载不掉，这样的话再重新安装该软件，共享期不会因此而改为 30 天，而是该日期从你第一次运行该软件算起。

### 12）破解心得

- a、善用多种工具进行分析，诸如 `Regmon`、`filemonnet`、`SmartCheck`，本次破解以 `SmartCheck` 分析为主，其他为辅；
- b、该软件作者在加密方面的却下了一番功夫，很多人都会被注册时“恭喜您，注册成功！”所迷惑，以为已成功破解，其实不然，软件在退出程序时再校验一次注册码，并且以删除文件来阻止他人用破解的注册码注册，还好，作者还是比较有善心的，只删除自己的文件，如果他要是删除些系统文件什么的，我的机器还不得重装几回呀，表扬表扬作者。
- c、在注册码和文件中写入第一次安装日期、最后一次运行日期的办法值得学习一下，但是乱丢垃圾就不好了，呵呵！

[版权声明]: 本文纯属技术交流，转载请注明作者并保持文章的完整，谢谢!

`4dd466d0e1e0cc123afbe504da2462ca`
