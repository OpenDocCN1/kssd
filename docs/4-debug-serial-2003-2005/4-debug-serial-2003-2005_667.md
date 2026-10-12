
# 软件管理专家 v1.22a

> 原文：[`www.pediy.com/kssd/pediy06/pediy6541.htm`](https://www.pediy.com/kssd/pediy06/pediy6541.htm)

**【软件名称】**: `软件管理专家 v1.22a`

**【软件限制】**: 功能限制

**【保护方式】**: 无

**【破解工具】**: `OllyDbg`

**【编程语言】**: `Delphi 6.0 - 7.0`

**【下载地址】**: [`www3.skycn.com/soft/4880.html`](http://www3.skycn.com/soft/4880.html)

**【软件简介】**: 下载文件的管理有时候是一件挺麻烦的事,特别是象笔者这样的网虫,每天都要下载大量的文件,而且一股脑的放在 `download` 目录下,但是时间一长的话,看着这些压缩包,自己也搞不清那个文件是干什么用的了。本软件就是用来管理这些下载文件的,你只要选中一个软件,将它拖拽到 `FlashSoft` 的窗口中,填写一些备忘的信息,就把这个下载软件分门别类地放到文件盒中了,下次需要的话,不用去找,直接在软件中调用就 `ok` 了,是不是挺方便的?

—————————————————————————————————

## 【破解过程】

软件目录下有个 `dfbox.ini` 文件,里面有：

```
UserName=aqtata
UserCode=123456
```

实时注册的地方有点不一样,那就从启动时的验证入手,`OD` 加载。

在 `GetPrivateProfileStringA` 断,注意堆栈窗口,直到出现：

```
0012F554   00488851  /CALL 到 GetPrivateProfileStringA 来自 DfBox.0048884C
0012F558   004913B4  |Section = "UserInfo"
0012F55C   004913A0  |Key = "UserName"
0012F560   0049138C  |Default = "No register"
0012F564   0012F57C  |ReturnBuffer = 0012F57C
0012F568   00000800  |BufSize = 800 (2048.)
0012F56C   00DA5370  \IniFileName = "D:\Flash Soft Manager\dfbox.ini"
```

好了,回到程序领空：

```
0048884B  .  50             PUSH EAX                 ; |Section
0048884C  .  E8 0BEBF7FF   CALL <JMP.&kernel32.GetPrivateProfileStr>; \GetPrivateProfileStringA
00488851  .  8BC8          MOV ECX,EAX   ====>停在这
```

一直 `F8`。

```
004EB28E  .  8B45 D4        MOV EAX,DWORD PTR SS:[EBP-2C]
004EB291  .  5A             POP EDX
004EB292  .  E8 655AFAFF    CALL DfBox.00490CFC    ====>这里跟进去
004EB297  .  84C0           TEST AL,AL
004EB299  .  74 07          JE SHORT DfBox.004EB2A2
004EB29B  .  C683 2C050000 >MOV BYTE PTR DS:[EBX+52C],1
004EB2A2  |> 80BB 2C050000 >CMP BYTE PTR DS:[EBX+52C],1
```

跟进 `00490CFC`：

```
00490CFC  /$ 55             PUSH EBP
00490CFD  |. 8BEC          MOV EBP,ESP
00490CFF  |. 83C4 F4       ADD ESP,-0C
00490D02  |. 53            PUSH EBX
00490D03  |. 33C9          XOR ECX,ECX
00490D05  |. 894D F4       MOV DWORD PTR SS:[EBP-C],ECX
00490D08  |. 8955 F8       MOV DWORD PTR SS:[EBP-8],EDX
00490D0B  |. 8945 FC       MOV DWORD PTR SS:[EBP-4],EAX
00490D0E  |. 8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]
00490D11  |. E8 5241F7FF   CALL DfBox.00404E68    ====>测试是否有用户名
00490D16  |. 8B45 F8       MOV EAX,DWORD PTR SS:[EBP-8]
00490D19  |. E8 4A41F7FF   CALL DfBox.00404E68    ====>测试是否有注册码
00490D1E  |. 33C0          XOR EAX,EAX
00490D20  |. 55            PUSH EBP
00490D21  |. 68 780D4900   PUSH DfBox.00490D78
00490D26  |. 64:FF30       PUSH DWORD PTR FS:[EAX]
00490D29  |. 64:8920       MOV DWORD PTR FS:[EAX],ESP
00490D2C  |. 8D55 F4       LEA EDX,DWORD PTR SS:[EBP-C]
00490D2F  |. 8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]
00490D32  |. E8 61000000   CALL DfBox.00490D98     ====>算法 call
00490D37  |. 8B45 F4       MOV EAX,DWORD PTR SS:[EBP-C]    ====>真码
00490D3A  |. 8B55 F8       MOV EDX,DWORD PTR SS:[EBP-8]    ====>假码
00490D3D  |. E8 8240F7FF   CALL DfBox.00404DC4          ====>比较
00490D42  |. 75 04         JNZ SHORT DfBox.00490D48     ====>不对则跳,爆破点
00490D44  |. B3 01         MOV BL,1
00490D46  |. EB 02         JMP SHORT DfBox.00490D4A
00490D48  |> 33DB         XOR EBX,EBX
00490D4A  |> 8B55 FC       MOV EDX,DWORD PTR SS:[EBP-4]
00490D4D  |. B8 900D4900   MOV EAX,DfBox.00490D90         ;  ASCII "ttdown"
00490D52  |. E8 6542F7FF   CALL DfBox.00404FBC
00490D57  |. 85C0          TEST EAX,EAX
00490D59  |. 74 02         JE SHORT DfBox.00490D5D
00490D5B  |. 33DB          XOR EBX,EBX
00490D5D  |> 33C0          XOR EAX,EAX
00490D5F  |. 5A            POP EDX
00490D60  |. 59            POP ECX
00490D61  |. 59            POP ECX
00490D62  |. 64:8910       MOV DWORD PTR FS:[EAX],EDX
00490D65  |. 68 7F0D4900   PUSH DfBox.00490D7F
00490D6A  |> 8D45 F4       LEA EAX,DWORD PTR SS:[EBP-C]
00490D6D  |. BA 03000000   MOV EDX,3
00490D72  |. E8 753CF7FF   CALL DfBox.004049EC
00490D77  \\. C3           RETN
```

没事看了看算法,发现很简单,跟进算法 `call`：

```
00490D98  /$ 55             PUSH EBP
00490D99  |. 8BEC          MOV EBP,ESP
00490D9B  |. 83C4 F8       ADD ESP,-8
00490D9E  |. 53            PUSH EBX
00490D9F  |. 56            PUSH ESI
00490DA0  |. 57            PUSH EDI
00490DA1  |. 8955 F8       MOV DWORD PTR SS:[EBP-8],EDX
00490DA4  |. 8945 FC       MOV DWORD PTR SS:[EBP-4],EAX
00490DA7  |. 8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]
00490DAA  |. E8 B940F7FF   CALL DfBox.00404E68
00490DAF  |. 33C0          XOR EAX,EAX
00490DB1  |. 55            PUSH EBP
00490DB2  |. 68 1D0E4900   PUSH DfBox.00490E1D
00490DB7  |. 64:FF30       PUSH DWORD PTR FS:[EAX]
00490DBA  |. 64:8920       MOV DWORD PTR FS:[EAX],ESP
00490DBD  |. BB 01000000   MOV EBX,1
00490DC2  |. 8B45 FC       MOV EAX,DWORD PTR SS:[EBP-4]
00490DC5  |. E8 B63EF7FF   CALL DfBox.00404C80   ====>用户名长度
00490DCA  |. 8BC8          MOV ECX,EAX    ====>送到 ecx
00490DCC  |. 85C9          TEST ECX,ECX   ====>测试长度
00490DCE  |. 7E 1C         JLE SHORT DfBox.00490DEC   ====>跳了就挂了
00490DD0  |. BE 01000000   MOV ESI,1
00490DD5  |> 8B45 FC       /MOV EAX,DWORD PTR SS:[EBP-4]   ====>用户名
00490DD8  |. 0FB67C30 FF   |MOVZX EDI,BYTE PTR DS:[EAX+ESI-1]   ====>依次取每个字符 ascii 值
00490DDD  |. 8BC3          |MOV EAX,EBX   ====>上次得出的结果送 eax
00490DDF  |. 99            |CDQ
00490DE0  |. 33C2          |XOR EAX,EDX
00490DE2  |. 2BC2          |SUB EAX,EDX
00490DE4  |. F7EF          |IMUL EDI   ====>ascii 值乘到 eax
00490DE6  |. 8BD8          |MOV EBX,EAX   ====>结果送到 ebx
00490DE8  |. 46            |INC ESI
00490DE9  |. 49            |DEC ECX
00490DEA  |. 75 E9         \JNZ SHORT DfBox.00490DD5
00490DEC  |> 8BC3          MOV EAX,EBX   ====>最后结果送到 eax
00490DEE  |. 99            CDQ
00490DEF  |. 33C2          XOR EAX,EDX
00490DF1  |. 2BC2          SUB EAX,EDX
00490DF3  |. 83C0 64       ADD EAX,64    ====>结果再加上 100
00490DF6  |. 99            CDQ
00490DF7  |. 33C2          XOR EAX,EDX
00490DF9  |. 2BC2          SUB EAX,EDX
00490DFB  |. 8BD8          MOV EBX,EAX
00490DFD  |. 8B55 F8       MOV EDX,DWORD PTR SS:[EBP-8]
00490E00  |. 8BC3          MOV EAX,EBX
00490E02  |. E8 AD86F7FF   CALL DfBox.004094B4
00490E07  |. 33C0          XOR EAX,EAX
00490E09  |. 5A            POP EDX
00490E0A  |. 59            POP ECX
00490E0B  |. 59            POP ECX
00490E0C  |. 64:8910       MOV DWORD PTR FS:[EAX],EDX
00490E0F  |. 68 240E4900   PUSH DfBox.00490E24
00490E14  |> 8D45 FC       LEA EAX,DWORD PTR SS:[EBP-4]
00490E17  |. E8 AC3BF7FF   CALL DfBox.004049C8
00490E1C  \\. C3           RETN
00490E1D   .^E9 EE34F7FF    JMP DfBox.00404310
00490E22   .^EB F0          JMP SHORT DfBox.00490E14
00490E24   .  5F           POP EDI
00490E25   .  5E           POP ESI
00490E26   .  5B           POP EBX
00490E27   .  59           POP ECX
00490E28   .  59           POP ECX
00490E29   .  5D           POP EBP
00490E2A   .  C3           RETN
```

这软件注册的方式不太一样,花了不少时间,我老是在实时比较注册码的地方兜圈子。

—————————————————————————————————

## 【爆破】

```
00490D42  .  75 04          JNZ SHORT DfBox.00490D48     ====>改成 nop(0x90142h)
```

—————————————————————————————————

## 【注册码】

`Name: aqtata`
`Code: 470414836`

—————————————————————————————————

## 【注册机】

算法简单了,难的也搞不了,呵呵,下面是 `vb.net` 注册机(控制台程序)：

```vb.net
Sub Main()
    System.Console.WriteLine("软件管理专家 v1.22a 算号器")
    Dim yhm As String
    System.Console.WriteLine("请输入用户名:")
    yhm = Console.ReadLine
    Dim i As Integer
    Dim a, b, d As Double
    b = 1
    For i = 1 To Len(yhm)
        d = Asc(Mid(yhm, i, 1))
        If Len(Hex(b)) > 8 Then b = "&H" & Right(Hex(b), 8)
        a = b
```



```
a = a \* d

If Len(Hex(a)) > 8 Then a = "&H" & Right(Hex(a), 8)

b = a

Next

System.Console.WriteLine("您的注册码是:" & b + 100)

Console.ReadLine()

End Sub
```

—————————————————————————————————

`4dd466d0e1e0cc123afbe504da2462ca`
