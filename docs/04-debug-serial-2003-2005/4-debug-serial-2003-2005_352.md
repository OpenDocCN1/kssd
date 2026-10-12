
# 鼎丰图书光盘东东租售系统 V5.6

> 原文：[`www.pediy.com/kssd/pediy06/pediy6111.htm`](https://www.pediy.com/kssd/pediy06/pediy6111.htm)

作者：newlaos[CCG][DFCG]

软件大小：2491KB

软件语言：简体中文

软件类别：国产软件/共享版/仓储租借

运行环境：Win9x/Me/NT/2000/XP

加入时间：2003-6-8 10:35:01

下载地址：`http://www.zzshow.com/bookVcd.exe`

**软件介绍**：本软件是一款适合对图书、光盘、软件等出音像制品进行出租与销售管理使用的软件。软件功能强大，界面美观，容易上手。系统管理主要包括销售商品资料管理，租凭商品资料管理和会员资料管理。基本操作为：销售、零租、会员租、零租归还、会员归还。功能强大的查询功能：可对销售商品资料、租赁商品资料、会员资料、会员租赁、会员归还、商品销售等进行查询，同时支持模糊查询。对查询结果可按照任意条件进行排序，支持报表打印，导入 Excel。今日盘点可统计出今日销售情况，今日销售商品进货，今日租凭商品进货等。统计排名可分别按照条形、圆饼、三维条形显示租凭商品的前十名。资金统计可显示当天，当月，上月，当年，去年和任意时段内的资金使用情况。另外，为了保证系统的安全使用，设置了管理员功能。管理员分为高级管理员和普通管理员，高级管理员可设置其它管理员的权限。普通管理员只能修改自己的密码。同时内建强大的数据备份功能和数据清理功能。软件在手，定会另您事半功倍。祝您生意兴隆，恭喜发财！

加密方式：注册码+ 反跟踪 +反汇编

功能限制：功能限制

PJ 工具：`W32Dasm8.93` 黄金版，`FI2.5`，`OLLYDBG1.09` 中文修正版

PJ 日期：2003-04-20

作者 newlaos 申明：只是学习，请不用于商业用途或是将本文方法制作的注册机任意传播，造成后果，本人一概不负。

## 分析过程

1、先用 `FI2.5` 看一下主文件“bookVcd.exe”，没有加壳，程序是用 DELPHI 编写的。

2、用 `W32Dasm8.93` 黄金版对 `bookVcd.exe` 进行静态反汇编，再用串式数据参考，找到“注册码错误，请与作者联系”，双击来到下面代码段。

3、该软件对 `TRW20001.23` 进行反调试，也就是说不能用 TRW 调试，一调试就出错。只是能用 `OLLYDBG1.09` 进行动态调试，下断 `BPX 00575C56`(通常在注册成功与否的前面一些下断，这样，才能找到关键部分)。

输入假码: `78787878`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00575BE6(C)

|

```
:00575C56 6A00                  push 00000000   <===你会发现在这里设才能断下来。作者为了防破解在上面的代码段动了些手脚，如果不知道，就很容易进行死胡同。
:00575C58 49                    dec ecx
:00575C59 75F9                  jne 00575C54
:00575C5B 53                    push ebx
:00575C5C 8BD8                  mov ebx, eax
:00575C5E 33C0                  xor eax, eax
:00575C60 55                    push ebp
:00575C61 68B35D5700            push 00575DB3
:00575C66 64FF30                push dword ptr fs:[eax]
:00575C69 648920                mov dword ptr fs:[eax], esp
:00575C6C 8D55F8                lea edx, dword ptr [ebp-08]
:00575C6F 8B8300030000          mov eax, dword ptr [ebx+00000300]
:00575C75 E8363DEDFF            call 004499B0    <===算出注册码的长度
:00575C7A 8B45F8                mov eax, dword ptr [ebp-08] <===EAX=78787878
:00575C7D 8D55FC                lea edx, dword ptr [ebp-04]
:00575C80 E84F35E9FF            call 004091D4
:00575C85 8B45FC                mov eax, dword ptr [ebp-04] <===EAX=78787878
:00575C88 8B15845E5900          mov edx, dword ptr [00595E84]
:00575C8E 8B12                  mov edx, dword ptr [edx]    <===EDX=3702619562
:00575C90 E847EFE8FF            call 00404BDC
:00575C95 7436                  je 00575CCD   <===关键跳转，了也就是说，上面的 EAX 和 EDX 必须相等了
:00575C97 6A00                  push 00000000

\* Possible StringData Ref from Code Obj ->"error"
                                 |
:00575C99 B9C05D5700            mov ecx, 00575DC0

\* Possible StringData Ref from Code Obj ->"注册码错误，请与作者联系."
                                 |
:00575C9E BAC85D5700            mov edx, 00575DC8
:00575CA3 A1185D5900            mov eax, dword ptr [00595D18]
:00575CA8 8B00                  mov eax, dword ptr [eax]
:00575CAA E83942EFFF            call 00469EE8
:00575CAF 8B8300030000          mov eax, dword ptr [ebx+00000300]
:00575CB5 E8E217ECFF            call 0043749C
:00575CBA 8B8300030000          mov eax, dword ptr [ebx+00000300]
:00575CC0 8B10                  mov edx, dword ptr [eax]
:00575CC2 FF92C0000000          call dword ptr [edx+000000C0]
:00575CC8 E9B5000000            jmp 00575D82

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00575C95(C)         <===跳到这里就说明注册成功了，向上看
|
:00575CCD 8B830C030000          mov eax, dword ptr [ebx+0000030C]
:00575CD3 E8F443F4FF            call 004BA0CC
:00575CD8 8B830C030000          mov eax, dword ptr [ebx+0000030C]
.......
.......(此处删除一段与算法无关的代码)

\* Possible StringData Ref from Code Obj ->"information"
                                 |
:00575D65 B9345E5700            mov ecx, 00575E34

\* Possible StringData Ref from Code Obj ->"恭喜您已经注册成功，请重新运行本软件！"
                                 |
:00575D6A BA405E5700            mov edx, 00575E40
:00575D6F A1185D5900            mov eax, dword ptr [00595D18]
:00575D74 8B00                  mov eax, dword ptr [eax]
:00575D76 E86D41EFFF            call 00469EE8
:00575D7B 8BC3                  mov eax, ebx
:00575D7D E89A08EFFF            call 0046661C

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00575CC8(U)
|
:00575D82 33C0                  xor eax, eax
:00575D84 5A                    pop edx
:00575D85 59                    pop ecx
:00575D86 59                    pop ecx
.......
.......
```

4、很快就你就会发现，关键算法段不在这里。怎么办？再用串式数据参考找啊找，又发现“您已经注册成为了正版用户”，这其实就是当你点击菜单里的注册项时，程序就会判断，如果你已经注册就会显示这个，如果你没有注册就会出现要注册的提示框。呵呵，注册算法部分找到了:-)

依然是用 `OLLYDBG1.09` 进行动态调试，设断点 `005761B5`，当你点击菜单里的注册项时，立即断下来。

```
\------------------------\************************\-----------------------------

:005761B4 56                  push esi

\* Reference To: kernel32.GetComputerNameA, Ord:0000h
                                 |
:005761B5 E8C60FE9FF            Call 00407180 <===获取计算名称函数
:005761BA 83F801                cmp eax, 00000001
:005761BD 1BC0                  sbb eax, eax
:005761BF 40                    inc eax
:005761C0 84C0                  test al, al
:005761C2 750C                  jne 005761D0 <===如果获取到就从这里跳走了
:005761C4 8BC6                  mov eax, esi
:005761C6 E875C6E8FF            call 00402840
:005761CB E929010000            jmp 005762F9

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:005761C2(C)
|
:005761D0 8D45F8                lea eax, dword ptr [ebp-08]
:005761D3 8BD6                  mov edx, esi <===EDX=POWERLAOS
:005761D5 E8F6E7E8FF            call 004049D0
:005761DA 8B55F8                mov edx, dword ptr [ebp-08] <===EDX=POWERLAOS
:005761DD B888805900            mov eax, 00598088

\* Possible StringData Ref from Code Obj ->"zjhloveling"
                                 |
:005761E2 B93C635700            mov ecx, 0057633C  <===作者内定的字符
:005761E7 E8F8E8E8FF            call 00404AE4 <===将计算名和内定的字符串连起来
:005761EC 8D55F4                lea edx, dword ptr [ebp-0C]
:005761EF A188805900            mov eax, dword ptr [00598088] <===EAX="POWERLAOSzjhloveling"
:005761F4 E8DBFEFFFF            call 005760D4  <===这就是算法软件机器码的 CALL
:005761F9 8B55F4                mov edx, dword ptr [ebp-0C] <===EDX="981727556"这就是我的机器码了
:005761FC B888805900            mov eax, 00598088
:00576201 E82EE6E8FF            call 00404834
:00576206 8BC6                  mov eax, esi
:00576208 E833C6E8FF            call 00402840
:0057620D 8D55F0                lea edx, dword ptr [ebp-10]
:00576210 A188805900            mov eax, dword ptr [00598088]
:00576215 E8BAFEFFFF            call 005760D4  <===关键的算法 CALL，F8 跟进(晕，和生成机器码的 CALL 是一样的)
:0057621A 8B55F0                mov edx, dword ptr [ebp-10] <===EDX="3702619562"就是注册码了
:0057621D B88C805900            mov eax, 0059808C
:00576222 E80DE6E8FF            call 00404834
:00576227 8B83F0020000          mov eax, dword ptr [ebx+000002F0]
.......
.......(省去一段与算法无关的代码)

:005762AA 8D45B0                lea eax, dword ptr [ebp-50]

\* Possible StringData Ref from Code Obj ->"zjhloveling"
                                 |
:005762AD BA3C635700            mov edx, 0057633C
:005762B2 E8B1F5E8FF            call 00405868
:005762B7 8D55B0                lea edx, dword ptr [ebp-50]
:005762BA 58                    pop eax
:005762BB E8F8F5E8FF            call 004058B8
:005762C0 7521                  jne 005762E3  <===这里是一个关键跳转，如果跳走就要出现注册对话框，如果不跳就显示已经注册
```



#### 反汇编代码分析

`005762C2 6A00` `push 00000000`

\* 可能的字符串引用 -> `"提示"`

`005762C4 B984635700` `mov ecx, 00576384`

\* 可能的字符串引用 -> `"您已经注册成为了正版用户"`

`005762C9 BA8C635700` `mov edx, 0057638C`

`005762CE A1185D5900` `mov eax, dword ptr [00595D18]`

`005762D3 8B00` `mov eax, dword ptr [eax]`

`005762D5 E80E3CEFFF` `call 00469EE8`

`005762DA 8BC3` `mov eax, ebx`

`005762DC E83B03EFFF` `call 0046661C`

`005762E1 EB16` `jmp 005762F9`

\* 被无条件或条件跳转引用：

来自地址 `:00576291(C), :005762C0(C)`

`005762E3 A1745E5900` `mov eax, dword ptr [00595E74]`

......

......

-------------------------------------------------- `:00576215 call 005760D4` (算法 CALL，F8 跟进) --------------------------------------------------

......

...... (省去一段与算法无关的代码)

`0057610A 8B45F4` `mov eax, dword ptr [ebp-0C]` <=== EAX="981727556"

`0057610D E886E9E8FF` `call 00404A98` <=== 算出机器码的长度

`00576112 8BC8` `mov ecx, eax`

\* 被无条件或条件跳转引用：

来自地址 `:005760A5(C)`

`00576114 85C9` `test ecx, ecx`

`00576116 762A` `jbe 00576142` <=== 如果机器码的长度为 0，就跳走。这里不跳

`00576118 BE01000000` `mov esi, 00000001`

\* 被无条件或条件跳转引用：

来自地址 `:00576140(C)` ***最关键的循环结构***

`0057611D 8B45FC` `mov eax, dword ptr [ebp-04]` <=== EAX="981727556"

`00576120 0FB64430FF` `movzx eax, byte ptr [eax+esi-01]` <=== 依次取"981727556"字符的 ASC 值

`00576125 03D8` `add ebx, eax`

`00576127 69C3D8600B00` `imul eax, ebx, 000B60D8`

`0057612D BB7B000000` `mov ebx, 0000007B`

`00576132 33D2` `xor edx, edx` <=== EDX 清 0

`00576134 F7F3` `div ebx` <=== 除以 7B

`00576136 69C039300000` `imul eax, 00003039` <=== 得到的商再乘以 0x3039

`0057613C 8BD8` `mov ebx, eax` <=== 最后的 EBX 就是我们要的结果

`0057613E 46` `inc esi`

`0057613F 49` `dec ecx`

`00576140 75DB` `jne 0057611D` <=== 向上跳构成一个循环结构

\* 被无条件或条件跳转引用：

来自地址 `:00576116(C)`

`00576142 8BC3` `mov eax, ebx` <=== 最后 EBX 就是我们要的注册码了，但还只是 16 进制的形式 DCB17DAA

`00576144 33D2` `xor edx, edx`

`00576146 52` `push edx`

`00576147 50` `push eax`

`00576148 8B45F8` `mov eax, dword ptr [ebp-08]`

`0057614B E8D033E9FF` `call 00409520`

`00576150 33C0` `xor eax, eax`

`00576152 5A` `pop edx`

`00576153 59` `pop ecx`

`00576154 59` `pop ecx`

`00576155 648910` `mov dword ptr fs:[eax], edx`

`00576158 6875615700` `push 00576175`

\* 被无条件或条件跳转引用：

来自地址 `:00576173(U)`

`0057615D 8D45F4` `lea eax, dword ptr [ebp-0C]`

`00576160 E87BE6E8FF` `call 004047E0`

`00576165 8D45FC` `lea eax, dword ptr [ebp-04]`

`00576168 E873E6E8FF` `call 004047E0`

`0057616D C3` `ret`

......

......

----------------------------------------------------------------------------

### 5、注册机源程序

================================================ VB6.0 源程序，在 WIN98 下测试通过 ================================

```vb
Private Sub Text1_Change()
    astr1 = Text1.Text
    charlen = Len(astr1)
    e = 1
    For i = 1 To charlen
        sumtmp = Asc(Mid(astr1, i, 1))
        If sumtmp < 48 Or sumtmp > 57 Then
            e = 2
        End If
    Next
    If charlen = 0 Then e = 3

    If e = 1 Then
        ebx = "0"
        For i = 1 To charlen
            sumtmp = bigadd(ebx, CStr(Asc(Mid(astr1, i, 1))))
            strtmp = oct2hex(bigmul(sumtmp, "745688"))
            strtmp = hex2oct(Right(strtmp, 8))
            strtmp = bigdiv(strtmp, "123")
            ebx = hex2oct(Right(oct2hex(bigmul(strtmp, "12345")), 8))
        Next
        laststr = CStr(ebx)
    Else
        laststr = "你输入的机器码有误"
    End If

    Text2.Text = laststr
End Sub

'***********************************上面部分就是真正的实现部分***下面只是大数运算的函数而已***
```

```vb
Function hex2oct(sum16)
    '专门将大的 16 进制转为 10 进制的函数(目前只支持 56 位转换)
    '函数定义是 sum16 为 56 位以下的 16 进制数的字符串表示形式

    astr1 = UCase(sum16)
    nlen = Len(astr1)

    For i = 1 To nlen   '检查 16 进制的合法性
        SUMSUM = Asc(Mid(astr1, i, 1))
        If (SUMSUM >= 48 And SUMSUM <= 57) Or (SUMSUM >= 65 And SUMSUM <= 70) Then
            e = 1
        Else
            e = 2
        End If
    Next i

    If nlen = 0 Then
        e = 3
    End If

    Select Case e
        Case 1
            '在这里填入 16 进制转为 10 进制的代码段
            bigsum = "0"
            For i = 1 To nlen
                sesum = CStr(CInt("&h" + Mid(astr1, nlen - i + 1, 1)))
                For J = 1 To i - 1
                    sesum = bigmul(sesum, "16")
                Next J
                bigsum = bigadd(bigsum, sesum)
            Next i
            laststr = bigsum
            hex2oct = laststr

        Case 2
            hex2oct = "你的输入非法!"

        Case 3
            If Option1.value = True Then
                hex2oct = "16 进制实时显示"
            Else
                hex2oct = "10 进制实时显示"
            End If
    End Select
End Function

Function oct2hex(sum10)
    '专门将大的 10 进制转为 16 进制的函数(目前只支持 56 位转换)
    '函数定义是 sum10 为 56 位以下的 10 进制数的字符串表示形式

    astr1 = sum10
    e = 1 '输入操作数正确标志

    For i = 1 To Len(astr1)   '检查 10 进制的合法性
        SUMSUM = Asc(Mid(astr1, i, 1))
        If SUMSUM < 48 Or SUMSUM > 57 Then
            e = 2
        End If
    Next i

    If Len(astr1) = 0 Then
        e = 3
    End If

    Select Case e
        Case 1
            x = Array("", "", "", "", "", "", "", "", "", "", "", "", "", "")  '对数组进行定义,每个单元代表 4 位数，这个数组可以代表 56 位长度的 10 进制数
            '********************************************************
            Do
                nlen = Len(astr1) '得出 10 进制的长度
                p = Int((nlen - 1) / 4) '得出数字占数组的个数(减去 1 代表数组单元的下标)，每 4 位数占一个数组单元
                i = 0
                k = 1
                Do
                    alen = Len(astr1)
                    If alen <= 4 Then
                        k = 2
                        x(i) = astr1
                    Else
                        x(i) = Right(astr1, 4)          '每次从低位取 4 位
                        astr1 = Mid(astr1, 1, alen - 4)
                    End If
                    i = i + 1
                Loop While k = 1           '完成数据向数组的装入，每个单元为字符串，是从低位向高位的顺序

                strnext = ""
                strtmp = ""
                modtmp = ""
                r = 1      '前导去 0 标志
                '********************************************************
                For z = p To 0 Step -1      '从最后一位，也是 10 进制数的最高位开始
                    modtmp1 = CStr(CLng(modtmp + x(z)) Mod 16) '模作为与下一个数组单元相接合，再求模，直到这后的模
                    strtmp = CStr(Int(CLng(modtmp + x(z)) / 16))

                    If Len(strtmp) < 4 And r <> 1 Then '补 0 也是为了下次大循环的运算,如果是第一个值就不用补 0 了
                        For i = 1 To 4 - Len(strtmp)
                            strtmp = "0" + strtmp
                        Next i
                    End If

                    strnext = strnext + strtmp  '得到除以 16 的商，准备用下一次大循环的运算初始值
                    modtmp = modtmp1
                    r = 2
                Next z
                '********************************************************
                lastmod = Hex(Int(modtmp)) + lastmod '其实这就是 16 进制数了，不用再反向了
                astr1 = strnext
                i = 1
                Do      '前导去 0 循环
                    nlen = Len(astr1)
                    If Left(astr1, 1) = "0" Then
                        astr1 = Right(astr1, nlen - 1)
                    Else
                        Exit Do
                    End If
                    i = i + 1
                Loop While i <= Len(astr1)

                If Len(astr1) <= 2 Then    '因为前已经加成 4 个 0 了，对循环退出的判断
                    If Len(astr1) = 0 Then  '防止到下面来个 CInt(astr1)类型不符
                        Exit Do
                    End If
                    If CInt(astr1) < 16 Then
                        Exit Do
                    End If
                End If
            Loop
            '********************************************************
            If Len(astr1) = 0 Then
                laststr = lastmod
            Else
                laststr = Hex(CInt(astr1)) + lastmod
            End If

            oct2hex = laststr

        Case 2
            oct2hex = "你的输入非法!"

        Case 3
            If Option1.value = True Then
                oct2hex = "16 进制实时显示"
            Else
                oct2hex = "10 进制实时显示"
            End If
    End Select
End Function
```



## Function `bigadd(add1, add2)`

大数相加函数，参数定义`add1`,`add2`均为 10 进制数的字符串表示形式，限定为 28 位长度。

返回值也是 10 进制的字符串形式。当然还有一个错误返回值是`"inputerror"`

每一个 0 代表 4 位数，第一个 0 代表 1-4 位，第 2 个 0 代表 5-8 位，第 3 个 0 代表 9-12 们，类推，第 7 个 0 代表 25-28 位(这是从最低位开始数哟)

```vb
astr1 = add1
astr2 = add2
e = 1 '输入操作数正确标志
For i = 1 To Len(astr1)
    '检查第一输入的合法性
    SUMSUM = Asc(Mid(astr1, i, 1))
    If SUMSUM < 48 Or SUMSUM > 57 Then
        e = 2
    End If
Next i
For i = 1 To Len(astr2)
    '检查第二个输入的合法性
    SUMSUM = Asc(Mid(astr2, i, 1))
    If SUMSUM < 48 Or SUMSUM > 57 Then
        e = 2
    End If
Next i
If Len(astr1) = 0 Or Len(astr2) = 0 Then '其中任意一个操作数不能为空
    e = 2
End If

If e = 1 Then
    A = Array(0, 0, 0, 0, 0, 0, 0)
    i = 0
    k = 1
    Do
        alen = Len(astr1)
        If alen <= 4 Then
            k = 2
            A(i) = CLng(astr1)
        Else
            A(i) = CLng(Right(astr1, 4))
            '每次从低位取 4 位
            astr1 = Mid(astr1, 1, alen - 4)
        End If
        i = i + 1
    Loop While k = 1
    '到这里完成将 10 进制数第 4 位放入数组 a
    wa = i
    '这里代表 a 数组里有几个数

    B = Array(0, 0, 0, 0, 0, 0, 0)
    i = 0
    k = 1
    Do
        alen = Len(astr2)
        If alen <= 4 Then
            k = 2
            B(i) = CLng(astr2)
        Else
            B(i) = CLng(Right(astr2, 4))
            '每次取 4 位
            astr2 = Mid(astr2, 1, alen - 4)
        End If
        i = i + 1
    Loop While k = 1
    '到这里完成将 10 进制数第 4 位放入数组 b
    wb = i
    '这里代表 b 数组里有几个数

    C = Array(0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0)
    '定义最终结果数组，每一个 0 代表 4 位数

    '---------------------------------相加代码段--------------------------------
    If wa > wb Then
        w = wa - 1
    Else
        w = wb - 1
    End If
    i = 0
    tmp2 = 0
    Do While i <= w
        '以数组最多的数决定循环次数
        tmp = A(i) + B(i) + tmp2
        C(i) = (tmp Mod 10000)
        '只留 4 位
        tmp2 = Int(tmp / 10000)
        '商自动进入下一位
        i = i + 1
        If i = 7 Then
            Exit Do
        End If
    Loop
    C(i) = tmp2
    '最后一个进位值

    k = 1
    For i = 13 To 0 Step -1
        If C(i) <> 0 Or k <> 1 Then
            ttmp = CStr(C(i))
            If Len(ttmp) < 4 And k <> 1 Then
                '当出现某个数组中的数为 0890 的情况，如果不补 0，就会出错了。当然也要考第一个数防止出现数字最前面出现多余的 0 的情况
                For J = 1 To 4 - Len(ttmp)
                    ttmp = "0" + ttmp
                Next
            End If
            laststr3 = laststr3 + ttmp
            k = 2
        End If
    Next i
    If k = 1 Then
        '两个 0 两相加，就可以一次都不经过
        bigadd = "0"
    Else
        bigadd = laststr3
    End If
Else
    bigadd = "inputerror"
    '说明输入错误了
End If
'---------------------------------相加代码段结束--------------------------------
```

## Function `bigmul(mul1, mul2)`

大数相乘函数，参数定义`mul1`,`mul2`均为 10 进制数的字符串表示形式，限定为 28 位长度。

返回值也是 10 进制的字符串形式。当然还有一个错误返回值是`"inputerror"`

每一个 0 代表 4 位数，第一个 0 代表 1-4 位，第 2 个 0 代表 5-8 位，第 3 个 0 代表 9-12 们，类推，第 7 个 0 代表 25-28 位(这是从最低位开始数哟)

```vb
astr1 = mul1
astr2 = mul2
e = 1 '输入操作数正确标志
For i = 1 To Len(astr1)
    '检查第一输入的合法性
    SUMSUM = Asc(Mid(astr1, i, 1))
    If SUMSUM < 48 Or SUMSUM > 57 Then
        e = 2
    End If
Next i
For i = 1 To Len(astr2)
    '检查第二个输入的合法性
    SUMSUM = Asc(Mid(astr2, i, 1))
    If SUMSUM < 48 Or SUMSUM > 57 Then
        e = 2
    End If
Next i
If Len(astr1) = 0 Or Len(astr2) = 0 Then '其中任意一个操作数不能为空
    e = 2
End If

If e = 1 Then
    A = Array(0, 0, 0, 0, 0, 0, 0)
    i = 0
    k = 1
    Do
        alen = Len(astr1)
        If alen <= 4 Then
            k = 2
            A(i) = CLng(astr1)
        Else
            A(i) = CLng(Right(astr1, 4))
            '每次从低位取 4 位
            astr1 = Mid(astr1, 1, alen - 4)
        End If
        i = i + 1
    Loop While k = 1
    '到这里完成将 10 进制数第 4 位放入数组 a
    wa = i
    '这里代表 a 数组里有几个数

    B = Array(0, 0, 0, 0, 0, 0, 0)
    i = 0
    k = 1
    Do
        alen = Len(astr2)
        If alen <= 4 Then
            k = 2
            B(i) = CLng(astr2)
        Else
            B(i) = CLng(Right(astr2, 4))
            '每次取 4 位
            astr2 = Mid(astr2, 1, alen - 4)
        End If
        i = i + 1
    Loop While k = 1
    '到这里完成将 10 进制数第 4 位放入数组 b
    wb = i
    '这里代表 b 数组里有几个数

    C = Array(0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0)
    '定义最终结果数组，每一个 0 代表 4 位数

    '---------------------------------相乘代码段开始(不考虑负数情况)--------------------------------
    For i = 0 To 13
        C(i) = 0
    Next i

    For i = 0 To 6
        tmp2 = 0
        For J = 0 To 6
            sumtmp = A(J) \ B(i) + tmp2 + C(i + J)
            C(i + J) = (sumtmp Mod 10000)
            tmp2 = Int(sumtmp / 10000)
        Next J
        C(i + J) = tmp2
    Next i

    k = 1
    For i = 13 To 0 Step -1
        If C(i) <> 0 Or k <> 1 Then
            ttmp = CStr(C(i))
            If Len(ttmp) < 4 And k <> 1 Then
                '当出现某个数组中的数为 0890 的情况，如果不补 0，就会出错了。当然也要考第一个数防止出现数字最前面出现多余的 0 的情况
                For J = 1 To 4 - Len(ttmp)
                    ttmp = "0" + ttmp
                Next
            End If
            laststr4 = laststr4 + ttmp
            k = 2
        End If
    Next i
    If k = 1 Then
        'k=1 说明一次都没经过，结果就为 0
        bigmul = "0"
    Else
        bigmul = laststr4
    End If
Else
    bigmul = "inputerror"
End If
```

## Function `bigdiv(mul1, mul2)`

大数相除函数，参数定义`mul1`,`mul2`均为 10 进制数的字符串表示形式

这里的被除数只能在`long`的范围内

```vb
astr1 = mul1
nlen = Len(astr1)
astr2 = mul2
lasturn = ""
laststr = ""
'最后的商
k = 1
For i = 1 To nlen
    strtmp = lasturn + Mid(astr1, i, 1)
    sumtmp = CLng(strtmp)
    If sumtmp < CLng(astr2) Then
        lasturn = CStr(strtmp)
        If k = 2 Then laststr = laststr & "0"
    Else
        laststr = laststr & CStr(Int(sumtmp / CLng(astr2)))
        lasturn = CStr(sumtmp Mod CLng(astr2))
        k = 2
    End If
Next
bigdiv = laststr
'lasturn 就是模了
```

`4dd466d0e1e0cc123afbe504da2462ca`
