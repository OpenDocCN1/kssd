
# 彩票快车 V1.07

> 原文：[`www.pediy.com/kssd/pediy03/forum659.htm`](https://www.pediy.com/kssd/pediy03/forum659.htm)

国产软件每日一破(1) (彩票快车 V1.07 破解记录)

软件名称：彩票快车 V1.07 [国产]

来源 2001-02-19 海阔天空 502k

软件授权：共享软件

注册费用：28 元人民币

软件下载 `ftp://ftp.bj.software.chinese.com/software/soft_others/cpkc.zip`

软件简介：

全国各地电脑福利电脑体育彩票最具实力的分析管理投注软件，及时快捷的数据更新！

适用于全国各省、市的各类电脑体育、福利彩票。电脑随机选号的范围可以自行指定。

用后感觉不错，推荐，祝中大奖。

程序不修改 `Windows` 注册表的任何内容，没有加壳。

## 一、暴力破解

`W32DASM` 金版 => 串式参考 "您的彩票快车已注册成功，希望多提宝贵意见。谢谢" =>

只有 1 处，找到后，向上找第一个跳转语句，即 (2) 处，此为关键跳转。(1) 处为关键比较。

```asm
:004A8BE0 C1E003           shl eax, 03
:004A8BE3 2BC2             sub eax, edx
:004A8BE5 3BD8             cmp ebx, eax  ************ (1)关键比较
:004A8BE7 0F858B010000     jne 004A8D78     (2)关键跳转
           ^^=>84(爆破)     ****************
:004A8BED A1947C4B00       mov eax, dword ptr [004B7C94]
:004A8BF2 8B15EC794B00     mov edx, dword ptr [004B79EC]
:004A8BF8 8B12             mov edx, dword ptr [edx]
:004A8BFA E8B9AFF5FF       call 00403BB8
:004A8BFF 6A01             push 00000001

* Possible StringData Ref from Code Obj ->"彩票快提示窗口" (丢掉车字,国产的"质量"!!!)

           |
:004A8C01 B9608E4A00       mov ecx, 004A8E60

* Possible StringData Ref from Code Obj ->"您的彩票快车已注册成功，希望多提宝贵意见。谢谢"
                             ->"！"
           |
:004A8C06 BA708E4A00       mov edx, 004A8E70
:004A8C0B A1147C4B00       mov eax, dword ptr [004B7C14]
:004A8C10 8B00             mov eax, dword ptr [eax]
```

`w32dasm` 中 (2) 处底部偏移地址为 `A7FE7`，用 `ultraedit` 修改 `0F85..` => `0F84..`。

改完后再启动，软件注册菜单消失。

## 二、追注册码

1. 软件注册菜单中填入姓名、电话、电子邮件后，点生成用户码，信息存在当前目录下 `lotteryrapido.ini` 中，我的用户码是 `16131363`。

   ```ini
   [彩票快车]
   姓名=风飘雪
   电话=68394831
   电子邮件=gd1@yeah.net
   用户码=16131363
   注册码=
   ```

2. 试填入注册码，发现注册码必须是 8 位（可用穷举法，haha!!）。

   输入 `11111111`。

3. 启动 `trw2000`，下中断：

   ```asm
   bpx 4a8be5
   go
   ```

   点注册，被拦。

   ```asm
   ? eax
   eax=77777777 (在哪里,在哪里见过你,你的笑容这麽熟悉)
   ```

   想起舒淇和天心 MM 的电影我爱 777

   ```asm
   ? ebx
   ebx=368098003
   ```

4. 琢磨 `368098003÷7=52585429`。

5. 填入 `52585429`，点注册，`ok, ok!!!`。

作者：风飘雪

主页：`http://duba.126.com`

`e-mail`：`gd1@yeah.net`

```asm
016F:004A8A80  CALL    00403DE4           取 name 长度
016F:004A8A85  ADD     EAX,BYTE +0B        加上 0B
016F:004A8A88  MOV     ECX,EAX
016F:004A8A8A  SHL     ECX,07
016F:004A8A8D  SUB     ECX,EAX            *7F 设为 L
016F:004A8A8F  MOV     EAX,[004B7CF0]
016F:004A8A94  MOV     EAX,[EAX]          取用户码
016F:004A8A96  MOV     EBX,7F
016F:004A8A9B  CDQ
016F:004A8A9C  IDIV    EBX
016F:004A8A9E  MOV     EDX,EAX            除以 7F 取余
016F:004A8AA0  SHL     EAX,04
016F:004A8AA3  ADD     EAX,EDX            *11
016F:004A8AA5  XOR     EAX,02EFB69C       XOR 02EFB69C
016F:004A8AAA  CDQ                       \\
016F:004A8AAB  XOR     EAX,EDX           > 取绝对值
016F:004A8AAD  SUB     EAX,EDX          /
016F:004A8AAF  ADD     ECX,EAX          与 L 相加
016F:004A8AB1  MOV     EAX,[004B7CF0]
016F:004A8AB6  MOV     [EAX],ECX          放回用户码处，设为 Y
016F:004A8AB8  MOV     EDX,[004B7C08]
016F:004A8ABE  MOV     EDX,[EDX]
016F:004A8AC0  MOV     EAX,[004B7AAC]
016F:004A8AC5  MOV     ECX,004A8E40
016F:004A8ACA  CALL    00403E30           将 name 与**&%#$@KLhd 串起来
016F:004A8ACF  MOV     EAX,[004B7AAC]      如：abc->abc**&%#$@KLhd 设为 name'
016F:004A8AD4  MOV     EAX,[EAX]
016F:004A8AD6  CALL    00403DE4           取 name'的长度
016F:004A8ADB  MOV     ECX,EAX
016F:004A8ADD  TEST    ECX,ECX
016F:004A8ADF  JNG     004A8B17
016F:004A8AE1  MOV     EBX,01             循环变量
016F:004A8AE6  MOV     EAX,[004B7AAC]      －－循环头－－
016F:004A8AEB  MOV     EAX,[EAX]
016F:004A8AED  MOVZX   EAX,BYTE [EAX+EBX-01] 取 name'的第 EBX 个字符
016F:004A8AF2  MOV     EDX,[004B7CF0]
016F:004A8AF8  IMUL    DWORD [EDX]        与 Y 相乘
016F:004A8AFA  ADD     EAX,9F             加上 9F
016F:004A8AFF  CDQ
016F:004A8B00  XOR     EAX,EDX
016F:004A8B02  SUB     EAX,EDX            取绝对值
016F:004A8B04  MOV     EDI,05F5E100
016F:004A8B09  CDQ
016F:004A8B0A  IDIV    EDI                除以 05F5E100
016F:004A8B0C  MOV     EAX,[004B7CF0]
016F:004A8B11  MOV     [EAX],EDX          放入 Y
016F:004A8B13  INC     EBX
016F:004A8B14  DEC     ECX                循环次数为 name'的长度
016F:004A8B15  JNZ     004A8AE6           －－循环尾－－
016F:004A8B17  MOV     EAX,[004B7CF0]
016F:004A8B1C  ADD     DWORD [EAX],016676E1 y+016676E1-->Y
016F:004A8B22  MOV     EAX,[004B7AAC]
016F:004A8B27  PUSH    EAX
016F:004A8B28  LEA     EDX,[EBP-10]
016F:004A8B2B  MOV     EAX,[004B7CF0]
016F:004A8B30  MOV     EAX,[EAX]
016F:004A8B32  CALL    00408684           将 Y 变成 10 进制的字符串，设为 Y'
016F:004A8B37  MOV     EAX,[EBP-10]
016F:004A8B3A  MOV     ECX,08
016F:004A8B3F  MOV     EDX,01
016F:004A8B44  CALL    00403FEC           取 Y'的前 8 位，设为 Y"
016F:004A8B49  LEA     EAX,[EBP-04]
016F:004A8B4C  MOV     EDX,004A8E54       这里有一串字符：35217864
016F:004A8B51  CALL    00403BFC
016F:004A8B56  MOV     EAX,[004B7D30]
016F:004A8B5B  CALL    00403B64
016F:004A8B60  MOV     EBX,01             循环变量
016F:004A8B65  LEA     EAX,[EBP-14]       －－循环头－－
016F:004A8B68  MOV     EDX,[EBP-04]
016F:004A8B6B  MOVZX   EDX,BYTE [EDX+EBX-01]
016F:004A8B70  MOV     ECX,[004B7AAC]
016F:004A8B76  MOV     ECX,[ECX]
016F:004A8B78  MOV     DL,[ECX+EDX-31]    这个循环的作用是：
016F:004A8B7C  CALL    00403D0C           将 Y"串作位置上的变换，
016F:004A8B81  MOV     EDX,[EBP-14]       如 Y"=12345678，则变换为 35217864
016F:004A8B84  MOV     EAX,[004B7D30]     哈哈，变换后的串就是注册码了！
016F:004A8B89  CALL    00403DEC
016F:004A8B8E  MOV     EAX,[004B7D30]
016F:004A8B93  INC     EBX
016F:004A8B94  CMP     EBX,BYTE +09       循环 8 次
016F:004A8B97  JNZ     004A8B65           －－循环尾－－
016F:004A8B99  LEA     EDX,[EBP-1C]
016F:004A8B9C  MOV     EAX,[ESI+02FC]
```

```vb
'《彩票飞车 1.07》算号器，请用 VB5 以上编译

'说明：Text1 为用户名（可用汉字），Text2 为用户码，Text3 为注册码

'作者：小明　2001.2.20

Dim x, y As Double

Private Sub Text1_Change()
  Text2_Change
End Sub

Private Sub Text2_Change()
  uname = Trim(Text1.Text)
  l = Len(uname)
  x = Val(Text2.Text)
  
  If l > 0 And Len(Trim(Text2.Text)) = 8 And x > 0 Then
    For i = 1 To l
      If Asc(Mid(uname, i, 1)) < 0 Then l = l + 1
    Next
    
    x = (x \ 127) * 17 Xor &H2EFB69C
    x = zh(x) + (l + 11) * 127
    
    uname = uname + "**&%#$@KLhd"
    
    For i = 1 To Len(uname)
      a = Asc(Mid(uname, i, 1))
      If a < 0 Then
        a = a + 65536
        x = js(x, a \ 256)
        x = js(x, a Mod 256)
      Else
        x = js(x, a)
      End If
    Next
    
    x = x + &H16676E1
    zcm = Trim(Str(x))
    zzcm = ""
    
    For i = 1 To 8
      zzcm = zzcm + Mid(zcm, Val(Mid("35217864", i, 1)), 1)
    Next
    
    Text3.Text = zzcm
  Else
    Text3.Text = ""
  End If
End Sub

Function zh(y)
  y = y - Int(y / 4294967296#) * 4294967296#
  If y >= 2147483648# Then y = 4294967296# - y
  zh = y
End Function

Function js(y, b)
  y = zh(y * b + &H9F)
  y = y - Int(y / &H5F5E100) * &H5F5E100
  js = y
End Function
```

`4dd466d0e1e0cc123afbe504da2462ca`
