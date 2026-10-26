
```asm
0040F48F  |>  8B8D 6CFFFFFF /mov     ecx, dword ptr [ebp-94]         ;  再次8位倒换

0040F495  |.  83C1 01       |add     ecx, 1

0040F498  |.  898D 6CFFFFFF |mov     dword ptr [ebp-94], ecx

0040F49E  |>  83BD 6CFFFFFF> cmp     dword ptr [ebp-94], 8

0040F4A5  |.  7D 3C         |jge     short 0040F4E3

0040F4A7  |.  8B95 6CFFFFFF |mov     edx, dword ptr [ebp-94]

0040F4AD  |.  8B8495 78FFFF>|mov     eax, dword ptr [ebp+edx\*4-88]

0040F4B4  |.  8985 1CFFFFFF |mov     dword ptr [ebp-E4], eax

0040F4BA  |.  8B8D 6CFFFFFF |mov     ecx, dword ptr [ebp-94]

0040F4C0  |.  8B95 6CFFFFFF |mov     edx, dword ptr [ebp-94]

0040F4C6  |.  8B4495 98     |mov     eax, dword ptr [ebp+edx\*4-68]

0040F4CA  |.  89848D 78FFFF>|mov     dword ptr [ebp+ecx\*4-88], eax

0040F4D1  |.  8B8D 6CFFFFFF |mov     ecx, dword ptr [ebp-94]

0040F4D7  |.  8B95 1CFFFFFF |mov     edx, dword ptr [ebp-E4]

0040F4DD  |.  89548D 98     |mov     dword ptr [ebp+ecx\*4-68], edx

0040F4E1  |.^ EB AC         \jmp     short 0040F48F

0040F4E3  |>  C785 6CFFFFFF>mov     dword ptr [ebp-94], 0

0040F4ED  |.  EB 0F         jmp     short 0040F4FE

0040F4EF  |>  8B85 6CFFFFFF /mov     eax, dword ptr [ebp-94]         ;  比较部分

0040F4F5  |.  83C0 01       |add     eax, 1

0040F4F8  |.  8985 6CFFFFFF |mov     dword ptr [ebp-94], eax

0040F4FE  |>  83BD 6CFFFFFF> cmp     dword ptr [ebp-94], 8

0040F505  |.  7D 27         |jge     short 0040F52E

0040F507  |.  B9 0F000000   |mov     ecx, 0F                         ;  0F

0040F50C  |.  2B8D 6CFFFFFF |sub     ecx, dword ptr [ebp-94]         ;  ecx-x

0040F512  |.  8B95 6CFFFFFF |mov     edx, dword ptr [ebp-94]

0040F518  |.  8B8495 78FFFF>|mov     eax, dword ptr [ebp+edx\*4-88]   ;  首尾比较，相等则注册成功

0040F51F  |.  3B848D 78FFFF>|cmp     eax, dword ptr [ebp+ecx\*4-88]

0040F526  |.  74 04         |je      short 0040F52C

0040F528  |.  32C0          |xor     al, al

0040F52A  |.  EB 04         |jmp     short 0040F530

0040F52C  |>^ EB C1         \jmp     short 0040F4EF

0040F52E  |>  B0 01         mov     al, 1
```

## 破解总结

```
00410961  |> \C605 45A54300>mov     byte ptr [43A545], 1
```

`[43A545]`为全局变量。。。

## 算法总结

算法有了，但不是明码比较。。。

第一步：取固定字符串前16位：Hard Listening a

大写-0x36,小写-0x3C,空格为0 得到S1

第二步：S1八位倒换得到S2

第二步：取注册码同样处理得到S3

第三步：S3-S2得到S4

第四步：S4 8位倒换，首尾比较，相等注册成功，不等注册失败

一组可用的注册码：

注册名：随意

注册码：GDCV16GHzrHsnlHF

注册机源代码：

```
Function Check(Number As Integer) As Integer

    If Number < 58 Then

        Check = Number - 47

    Else

        If Number >= 58 And Number <= 96 Then

            Check = Number - 54

        Else

            If Number > 96 Then

                Check = Number - 60

            End If

        End If

    End If

End Function

Private Sub Command1_Click()

    Dim Word As String

    Dim Char As String

    Dim Ref_String(16) As Integer

    Dim Test_Char As String

    Dim Serial(16) As Integer

    Dim Output(16) As String

    Dim Success As Boolean

    Dim First_Rnd_Char As Integer

    Dim Selected_Char As Integer

    Text1.Text = ""

    Word = "3829322D322B00251225362800162D37"

    Char = "0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz"

        For x = 1 To 16

            Ref_String(x) = Val("&h" & Mid(Word, 2 * x - 1, 2))

        Next

    For i = 1 To 8

        Do

            Do

                Randomize

                First_Rnd_Char = (122 - 48) * Rnd() + 48

            Loop Until (First_Rnd_Char > 47 And First_Rnd_Char < 58) Or (First_Rnd_Char > 64 And First_Rnd_Char < 91) Or (First_Rnd_Char > 96 And First_Rnd_Char < 123)

            Success = False

            Output(i) = Chr(First_Rnd_Char)

            First_Rnd_Char = Check(First_Rnd_Char)

            First_Rnd_Char = First_Rnd_Char - Ref_String(i + 8)

            For y = 1 To Len(Char)

                Selected_Char = Asc(Mid(Char, y, 1))

                Test_Char = Chr(Selected_Char)

                Selected_Char = Check(Selected_Char)

                Selected_Char = Selected_Char - Ref_String(8 - i + 1)

                If First_Rnd_Char = Selected_Char Then

                    Output(16 - i + 1) = Test_Char

                    Success = True

                    Exit For

                End If

            Next

        Loop Until Success = True

    Next

    For m = 1 To 16

        First_Rnd_Char_serial = First_Rnd_Char_serial & Output(m)

        Text1.Text = Mid(First_Rnd_Char_serial, 9, 8) & Mid(First_Rnd_Char_serial, 1, 8)

    Next

End Sub

Private Sub Command2_Click()

Unload Me

End Sub
```

![](https://bbs.kanxue.com/upload/attach/200911/278066_7d9bfi35qv58z0i.gif)

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

上传的附件：

- [Snap1.gif](javascript:void(0);)

  （6.41kb，74次下载）

- [Space Trip 3D keygen.rar](javascript:void(0);)

  （15.11kb，17次下载）
