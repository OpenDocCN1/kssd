
# EZ Extract Resource v1.3

> 原文：[`www.pediy.com/kssd/pediy04/kanxue338.htm`](https://www.pediy.com/kssd/pediy04/kanxue338.htm)

**软件介绍：**

懒的写了，反正是个资源提取软件！

**下载地址：** `www.seamoontech.com`

废话少说，下载安装后，运行程序，弹出注册框，输入用户名：`Turkey`，注册码：`987654321`，确定。软件告诉你要重新运行软件来检查注册成功与否。好，关闭软件，我门来找它把注册信息放的哪里里了，呵呵。运气好，注册码存放在注册表里面，而且是明文形式的，好说。

启动 `trw`，`load` 软件，在入口处被 `trw` 断下，我们下断点：

```
bpx RegQueryValueExa do "db \*(esp+8)"
```

先解释一下这个命令的作用，当软件去读取注册表中某一个键的值的时候，就会被 `trw` 中断下来，且自动显示`\*(esp+8)`的内容，内容就是那个键的名称！双引号中就是被断下来后执行的命令，这样我们可以省很多操作，不要问我为什么`\*(esp+8)`中就是那个键的名称，我也不知道，我只是依葫芦画瓢！

接下来按 `F5` 让软件继续运行，在中断 N 次后，在 `trw` 的数据窗口的右上角区域，终于看到了存放用户名的那个键的名称“`regusername`”，再按几下 `F5`，看到了存放注册码的那个键的名称“`regcode`”，再安一下 `F5`，也就是当你第二次看到“`regcode`”时（为什么实在第二次？因为我试过啊，再按 `F5` 你看到的就不是 `regcode` 啦，所以是第二次啦），下 `bd *`，然后 `pmodule` 回到程序领空，这时候改用 `F10` 开始慢慢的，小心的跟踪，不久你就来到如下地方：

```
|:0040CE56 

|

:0040D5B0 6AFF                          push FFFFFFFF
:0040D5B2 68B0DB4500                    push 0045DBB0
:0040D5B7 64A100000000                  mov eax, dword ptr fs:\[00000000\]
:0040D5BD 50                            push eax
:0040D5BE 64892500000000                mov dword ptr fs:\[00000000\], esp
:0040D5C5 81ECD0000000                  sub esp, 000000D0
:0040D5CB C78424D800000000000000        mov dword ptr \[esp+000000D8\], 00000000
:0040D5D6 8B8424E0000000                mov eax, dword ptr \[esp+000000E0\]//用户名送 eax
:0040D5DD 6874CC4700                    push 0047CC74
:0040D5E2 50                            push eax
:0040D5E3 E8D1A20100                    call 004278B9
:0040D5E8 83C408                        add esp, 00000008
:0040D5EB 85C0                          test eax, eax//测试用户名是否为空
:0040D5ED 0F8432010000                  je 0040D725//空就跳走

:0040D5F3 8B8C24E4000000                mov ecx, dword ptr \[esp+000000E4\]//注册码送 ecx
:0040D5FA 6874CC4700                    push 0047CC74
:0040D5FF 51                            push ecx
:0040D600 E8B4A20100                    call 004278B9
:0040D605 83C408                        add esp, 00000008
:0040D608 85C0                          test eax, eax//测试注册码是否为空
:0040D60A 0F8415010000                  je 0040D725//空就跳走

:0040D610 8B9424E0000000                mov edx, dword ptr \[esp+000000E0\]//用户名送 edx
:0040D617 53                            push ebx
:0040D618 56                            push esi
:0040D619 33C9                          xor ecx, ecx
:0040D61B 8B72F8                        mov esi, dword ptr \[edx\-08\]//用户名长度送 esi
:0040D61E C644240868                    mov \[esp+08\], 68－－
:0040D623 85F6                          test esi, esi       |
:0040D625 C644240979                    mov \[esp+09\], 79    |
:0040D62A C644240A64                    mov \[esp+0A\], 64      |
:0040D62F C644240B73                    mov \[esp+0B\], 73 这一段代码是在从\[esp+08\]处放入 hydsoft 字符串
:0040D634 C644240C6F                    mov \[esp+0C\], 6F      |
:0040D639 C644240D66                    mov \[esp+0D\], 66      |
:0040D63E C644240E74                    mov \[esp+0E\], 74    |
:0040D643 C644240F00                    mov \[esp+0F\], 00－－
:0040D648 7E3F                          jle 0040D689
:0040D64A 55                            push ebp
:0040D64B 57                            push edi
:0040D64C 8D7C3417                      lea edi, dword ptr \[esp+esi+17\]

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040D685(C)
|

:0040D650 8B8424F0000000                mov eax, dword ptr \[esp+000000F0\]//用户名送 eax
:0040D657 BD07000000                    mov ebp, 00000007//7 送 ebp
:0040D65C 8A1C01                        mov bl, byte ptr \[ecx+eax\]//\[ecx+eax\]送 bl，eax 中放的是用户名，第一次经过时 ecx=0
:0040D65F 8BC1                          mov eax, ecx//ecx->eax
:0040D661 99                            cdq//edx 清零
:0040D662 F7FD                          idiv ebp//（eax idiv ebp）->dl(指余数，下同)
:0040D664 0FBEC3                        movsx eax, bl//bl->eax(带符号扩展，下同)
:0040D667 8BD9                          mov ebx, ecx//ecx->ebx
:0040D669 0FBE541410                    movsx edx, byte ptr \[esp+edx+10\]//\[esp+edx+10\]送 edx，\[esp+10\]处放着字符串 hydsoft
:0040D66E 03DA                          add ebx, edx//edx+ebx->ebx
:0040D670 03C3                          add eax, ebx//ebx+eax->eax
:0040D672 BB09000000                    mov ebx, 00000009//9 送 ebx
:0040D677 03C6                          add eax, esi//eax+esi->eax，esi 是用户名长度
:0040D679 99                            cdq//edx 清零
:0040D67A F7FB                          idiv ebx//(eax idiv ebx)->dl
:0040D67C 80C230                        add dl, 30//dl+30->dl，实际上是将结果转换成阿拉伯数字
:0040D67F 41                            inc ecx//ecx 加 1
:0040D680 8817                          mov byte ptr \[edi\], dl//dl->\[edi\]
:0040D682 4F                            dec edi//edi 减 1
:0040D683 3BCE                          cmp ecx, esi//比较用户名是否取完
:0040D685 7CC9                          jl 0040D650//没有取完，跳上去继续循环
:0040D687 5F                            pop edi
:0040D688 5D                            pop ebp

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040D648(C)
|

:0040D689 8D464D                        lea eax, dword ptr \[esi+4D\]//\[esi+4D\]送 eax，实际上是用户名长度加上 4D 的结果送 eax
:0040D68C B909000000                    mov ecx, 00000009//9 送 ecx
:0040D691 99                            cdq//edx 清零
:0040D692 F7F9                          idiv ecx//(eax idiv ecx)->dl
:0040D694 8B8424EC000000                mov eax, dword ptr \[esp+000000EC\]//用户输入的假注册码送 eax
:0040D69B 80C230                        add dl, 30//dl+30->dl，作用和上面一样
:0040D69E 88543410                      mov byte ptr \[esp+esi+10\], dl//dl->\[esp+esi+10\]
:0040D6A2 C644341100                    mov \[esp+esi+11\], 00//00->\[esp+esi+11\]，作为注册码的结束标志
:0040D6A7 8D742410                      lea esi, dword ptr \[esp+10\]//真正的注册码送 esi

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040D6CD(C)
|

:0040D6AB 8A10                          mov dl, byte ptr \[eax\]//这里向下是将真假注册码一位以为的进行比较，一个经典的组合！！
:0040D6AD 8A1E                          mov bl, byte ptr \[esi\]
:0040D6AF 8ACA                          mov cl, dl
:0040D6B1 3AD3                          cmp dl, bl
:0040D6B3 751E                          jne 0040D6D3//不正确就死
:0040D6B5 84C9                          test cl, cl
:0040D6B7 7416                          je 0040D6CF
:0040D6B9 8A5001                        mov dl, byte ptr \[eax+01\]
:0040D6BC 8A5E01                        mov bl, byte ptr \[esi+01\]
:0040D6BF 8ACA                          mov cl, dl
:0040D6C1 3AD3                          cmp dl, bl
:0040D6C3 750E                          jne 0040D6D3//不正确就死
:0040D6C5 83C002                        add eax, 00000002
:0040D6C8 83C602                        add esi, 00000002
:0040D6CB 84C9                          test cl, cl//取完了吗？
:0040D6CD 75DC                          jne 0040D6AB//没有取完就跳上去继续循环
```

好了，关键代码就这些了，在 `0040D6AB` 处 `d eax` 看到自己输入的假码，`d esi` 看到正确的注册码！！！算法就不总结了，我懒啊，呵呵，简单，一看就懂！

还是总结一下我的注册信息：

- 用户名：`Turkey`
- 注册码：`0666252`

**事后的一点废话：**

在注册表的这个地方：

`[HKEY_CURRENT_USER\Software\SeaMoonTech\EZ Extract Resource\V1.3]`

存放这你的注册信息，将它改动或删除就又可以注册了！！

**转载请保持其完整性！**

`Cracker:Turkey/灭害灵`

`2002.1.11@22:36`

```vb
Private Sub Command1_Click(Index As Integer)
    Dim lngLen As String
    Dim strSN As String
    Dim strTmp As String
    Dim strsum As String
    Dim str5 As String
    Dim mb(0 To 6) As Long
    Dim i, j, k, n As Long
    Dim byteAry() As Byte
    Dim regname, xz, ys As Long

    strSN = ""
    strTmp = Text2.Text
    byteAry = StrConv(strTmp, vbFromUnicode)
    For i = LBound(byteAry) To UBound(byteAry)
        str5 = str5 & ChrW(byteAry(i))
    Next i
    strTmp = str5
    lngLen = Len(strTmp)
    If Len(strTmp) <= 0 Then
        Form1.Hide
        MsgBox "用户名不能为空！", 0, "错误提示"
        Form1.Show
        Exit Sub
    End If
    ReDim code(0 To lngLen) As Long
    For j = 1 To Len("hydsoft")
        mb(j - 1) = Asc(Mid("hydsoft", j, 1))
        'mb(7) = 0
    Next j
    ys = 0
    For k = 1 To lngLen
```



```
    xz = Int(k - 1 / 7)

    regname = Val(byteAry(k - 1))

    If regname > 127 Then

        regname = (255 - regname + 1) * (-1)

    End If

    xz = mb(ys) + xz + regname + lngLen

    xz = xz Mod 9

code(lngLen - k) = xz + 48

ys = ys + 1

If ys > 6 Then

    ys = 0

End If

Next k

code(lngLen) = (lngLen + 77) Mod 9 + 48

For n = 0 To lngLen

    strSN = strSN & Chr(code(n))

Next n

Text3.Text = strSN

End Sub

Private Sub Command2_Click(Index As Integer)

    Text2.Text = ""

    Text3.Text = ""

End Sub
```

`keygen` for `ez`:

用`keymake14`

1. 中断地址：`0040ce56`

   中断次数：1

   首字节：`e8`

   命令长度：5

2. 中断地址：`0040d6ad`

   中断次数：1

   首字节：`8a`

   命令长度：2

3. 选择内存方式，在选择`esi`

4. 输入个人信息，确定生成`keygen`! `4dd466d0e1e0cc123afbe504da2462ca`
