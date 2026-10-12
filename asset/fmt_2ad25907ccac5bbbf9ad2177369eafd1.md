
## 反汇编代码分析

```
:0040A468 E801FDFFFF    call 0040A16E    ====> 进行计算，计算结果返回到 eax 中

上句可以看作是对某个参数进行变换的函数，如 ChangeCode(edx)，计算过程：如果 edx(要计算的字符的 ascii 值)在 30h-39h，则返回结果为 0-9，否则返回结果为 edx+"A"-0ah=edx\-37h

:0040A46D 83C404          add esp, 00000004

:0040A470 8945FC          mov dword ptr \[ebp\-04\], eax  ====> 计算结果放到\[ebp\-04\]中，为说明方便，设值为 X18

:0040A473 8B4508          mov eax, dword ptr \[ebp+08\]  ====> 变换码的地址

:0040A476 8B4D10          mov ecx, dword ptr \[ebp+10\]

:0040A479 8A11            mov dl, byte ptr \[ecx\]   ====> 取注册码的第 1 个字符

:0040A47B 8810            mov byte ptr \[eax\], dl   ====> 放到变换码中，即注册码的第 1 个字符也是变换码的第 1 个字符

:0040A47D C745F000000000  mov \[ebp\-10\], 00000000   ====> 设初值

:0040A484 EB09            jmp 0040A48F

\* Referenced by a (U)nconditional or (C)onditional Jump at Address: |:0040A51D(U) |

:0040A486 8B45F0          mov eax, dword ptr \[ebp\-10\]  ====> 取出变换次数值

:0040A489 83C001          add eax, 00000001        ====> 准备取下一个字符

:0040A48C 8945F0          mov dword ptr \[ebp\-10\], eax

\* Referenced by a (U)nconditional or (C)onditional Jump at Address: |:0040A484(U) |

:0040A48F 8B4DF0          mov ecx, dword ptr \[ebp\-10\]

:0040A492 3B4DF8          cmp ecx, dword ptr \[ebp\-08\]  ====> 是否变换完毕

:0040A495 0F8D87000000    jnl 0040A522            ====> 没有，则继续

:0040A49B 8B45F0          mov eax, dword ptr \[ebp\-10\]  ====> 取出变换次数值

:0040A49E 99              cdq

:0040A49F 2BC2            sub eax, edx

:0040A4A1 D1F8            sar eax, 1              ====> 除以 2

:0040A4A3 8945E4          mov dword ptr \[ebp\-1C\], eax

:0040A4A6 8B55FC          mov edx, dword ptr \[ebp\-04\]  ====> 第 18 个注册码的计算值 X18

:0040A4A9 F7DA            neg edx                 ====> 取负数，即-X18

:0040A4AB 2B55F0          sub edx, dword ptr \[ebp\-10\]  ====> -X18-变换次数

:0040A4AE 8955DC          mov dword ptr \[ebp\-24\], edx  ====> 放到\[ebp\-24\]中

:0040A4B1 837DE400        cmp dword ptr \[ebp\-1C\], 00000000  ====> 要变换的是否是第 2、3 个的注册码

:0040A4B5 7409            je 0040A4C0             ====> 是则转

:0040A4B7 8B45FC          mov eax, dword ptr \[ebp\-04\]  ====> 第 18 个注册码的计算值 X18

:0040A4BA 0345F0          add eax, dword ptr \[ebp\-10\]  ====> X18+变换次数

:0040A4BD 8945DC          mov dword ptr \[ebp\-24\], eax  ====> 放到\[ebp\-24\]中

40A486-40A4BD 的计算：i=1~19 表示注册码字符所在的位置，如果变换的是第 2 和 3 个的注册码，则\[ebp\-24\]=-(X18+i-2)，否则=(X18+i-2)

\* Referenced by a (U)nconditional or (C)onditional Jump at Address: |:0040A4B5(C) |

:0040A4C0 6A24            push 00000024          \\
:0040A4C2 8B4DDC          mov ecx, dword ptr \[ebp\-24\]  \\
:0040A4C5 F7D9            neg ecx                 |
:0040A4C7 51              push ecx                ====> 这段跟注册码变换无关
:0040A4C8 E829FEFFFF      call 0040A2F6           |
:0040A4CD 83C408          add esp, 00000008       /
:0040A4D0 8945E8          mov dword ptr \[ebp\-18\], eax /

:0040A4D3 8B5510          mov edx, dword ptr \[ebp+10\]  ====> 注册码地址

:0040A4D6 0355F0          add edx, dword ptr \[ebp\-10\]

:0040A4D9 8A4201          mov al, byte ptr \[edx+01\]    ====> 取出相应的注册码

:0040A4DC 8845E0          mov byte ptr \[ebp\-20\], al

:0040A4DF 8A4DE0          mov cl, byte ptr \[ebp\-20\]

:0040A4E2 51              push ecx

:0040A4E3 E886FCFFFF      call 0040A16E           ====> 进行变换

:0040A4E8 83C404          add esp, 00000004

:0040A4EB 8945EC          mov dword ptr \[ebp\-14\], eax  ====> 结果放入到\[ebp\-14\]中

:0040A4EE 8B55EC          mov edx, dword ptr \[ebp\-14\]

:0040A4F1 2B55DC          sub edx, dword ptr \[ebp\-24\]  ====> -\[ebp\-24\]，因此跟注册码所在的位置有关

:0040A4F4 8955EC          mov dword ptr \[ebp\-14\], edx

:0040A4F7 6A24            push 00000024           ====> 为什么是 24h？因为字符 0~9 加上 A~Z 正好共 36 个(24h)

:0040A4F9 8B45EC          mov eax, dword ptr \[ebp\-14\]

:0040A4FC 50              push eax

:0040A4FD E8F4FDFFFF      call 0040A2F6           ====> 进行变换

变换过程：如果\[ebp\-14\]为负数，则\[ebp\-14\]+24h，直到为非负值且小于 24h；如果\[ebp\-14\]为正数且大于等于 24h，则\[ebp\-14\]-24h 直到为非负值且小于 24h

:0040A502 83C408          add esp, 00000008

:0040A505 8945EC          mov dword ptr \[ebp\-14\], eax

:0040A508 8B4DEC          mov ecx, dword ptr \[ebp\-14\]

:0040A50B 51              push ecx

:0040A50C E8A9FBFFFF      call 0040A0BA           ====> 根据上面相应的计算结果确定变换代码

确定过程：如果\ebp\-14\的值在 0-9，则转为相应的数字，即 eax=ecx+30h，否则 eax=ecx+"A"-0ah=ecx+37h

:0040A511 83C404          add esp, 00000004

:0040A514 8B5508          mov edx, dword ptr \[ebp+08\]

:0040A517 0355F0          add edx, dword ptr \[ebp\-10\]

:0040A51A 884201          mov byte ptr \[edx+01\], al    ====> 变换结果放入到变换代码地址中

:0040A51D E964FFFFFF      jmp 0040A486

\* Referenced by a (U)nconditional or (C)onditional Jump at Address: |:0040A495(C) |

:0040A522 8BE5            mov esp, ebp

:0040A524 5D              pop ebp

:0040A525 C3              ret
```

## 变换过程总结

1. 注册码必须是 19 个字符且字符范围在 `0~9` 和 `A~Z` 之间。
2. 设注册码用 `R` 来表示，则 `R19=((R1+R2+...+R18) mod 24h)+(30h 或 37h)`，如果前面的余数<=9 则+30h，否则+37h。
3. 设第 18 位注册码的变换码为 `X18`，则 `X18=R18-37h` (注册码在 `A~Z`) 或 `X18=R18-30h` (注册码在 `0~9`)，如果用笔计算一下就可知道，`R18` 肯定是 `A~Z` 之间的字符，因此，`X18=R18-37h`。
4. 用户名和 E-mail 地址名的计算值=(所有字符的 ascii 值相加 mod 1Ah)+37h，注意：用户名和 E-mail 地址名不能用中文字符。
5. 变换代码的计算过程(用 `y` 表示)
   (1) `y1=r1`
   (2) 如果 `i=2` 或 `3`，`yi=((X1-(-(X18+i-2))) mod 24h)+30h 或 37h=(X1+X18+i-2) mod 24h)+30h 或 37h`
   (3) 如果 `i=4` 到 `17`，`yi=((X1-(X18+i-2))±n*24h)+30h 或 37h=(X1-X18-i+2±n*24h)+30h 或 37h`，`n=0,1,2`
6. `y5` 与用户名的计算值相等，`y7` 与 E-mail 地址的计算值相等，因此，`y5` 和 `y7` 肯定是在 `A~Z` 之间。

## 注册器实现

根据上面的变换过程我们可以做出这个软件的注册器，第一种方法是先构造一个 19 位的注册码，然后利用第 5,7,18 位的注册码再相应构造用户名和 E-mail 地址，这种方法虽然简单，但用户名和 E-mail 地址可能很难看，因为我们还是喜欢用户名和 E-mail 地址易懂的注册码，因此采用第二种方法。

第二种方法是先构造用户名和密码，生成相应的计算值，再构造第 18 位的注册码，然后根据用户名和密码的计算值及第 18 位的注册码确定第 5 和第 7 位的注册码，而第 1~4，6，8~17 位的注册码随机生成，只要符合在 `0~9` 或 `A~Z` 之间即可，最后生成第 18 位的注册码，根据这个思路，注册机如下(VB 程序)：

```vb
Private Sub Command1_Click()
    Dim UserName, EmailAddress, RegCode, UserNameLong, EmailLong
    Dim CheckUserCode, CheckEmailCode, RandCode, CheckCode
    RegCode = "": CheckCode = ""
    UserName = Text1.Text
    EmailAddress = Text2.Text
    UserNameLong = Len(Text1.Text)
    EmailLong = Len(Text2.Text)
    If UserNameLong = 0 Then
        MsgBox "用户名不能为空！", vbOKOnly, "请输入用户名"
        Exit Sub
    Else
        For i = 1 To UserNameLong
            CheckUserCode = CheckUserCode + Asc(Mid(UserName, i, 1))
        Next
        CheckUserCode = (CheckUserCode Mod 26) + &H41
    End If
    If EmailLong = 0 Then
        MsgBox "Email 地址不能为空！", vbOKOnly, "请输入 Email 地址"
        Exit Sub
    Else
        For i = 1 To EmailLong
            CheckEmailCode = CheckEmailCode + Asc(Mid(EmailAddress, i, 1))
        Next
        CheckEmailCode = (CheckEmailCode Mod 26) + &H41
    End If

GetReg18:
    Reg18 = MakeRndCode()
    If Reg18 <= &H39 Then
        GoTo GetReg18
    Else
        Reg18Mod = Reg18 - &H37
    End If
    Reg5 = (Reg18Mod + 3 + CheckUserCode - &H37) Mod &H24
    If Reg5 <= 9 Then
        Reg5 = Reg5 + &H30
    Else
        Reg5 = Reg5 + &H37
    End If
    If Reg5 > &H5A Then GoTo GetReg18
End Sub
```
