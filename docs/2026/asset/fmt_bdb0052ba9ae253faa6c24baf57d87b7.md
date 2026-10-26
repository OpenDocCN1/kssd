
# [原创]Turbo Photo V4.8 的注册

【破文作者】 rdsnow[BCG][PYG][D.4s]

【作者主页】 [9e9K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4u0V1M7$3&6G2N6#2)9J5k6i4W2K6x3e0j5^5i4K6u0W2j5$3!0E0](https://bbs.kanxue.com/elink@891K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4u0V1M7$3&6G2N6#2)9J5k6i4W2K6x3e0j5^5i4K6u0W2j5$3!0E0)

【 E-mail 】 [email]rdsnow@163.com[/email]

【 作者QQ 】 83757177

【文章题目】 Turbo Photo V4.8 的注册

【软件名称】 Turbo Photo

【软件版本】 V4.8

【下载地址】 [cfcK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3&6B7i4K6u0W2L8$3&6D9K9h3&6W2k6r3!0%4L8W2)9J5k6h3&6W2N6q4)9J5c8Y4y4G2k6Y4c8Q4x3V1j5#2y4U0t1H3i4K6u0W2K9s2c8E0](https://bbs.kanxue.com/elink@b30K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3&6B7i4K6u0W2L8$3&6D9K9h3&6W2k6r3!0%4L8W2)9J5k6h3&6W2N6q4)9J5c8Y4y4G2k6Y4c8Q4x3V1j5#2y4U0t1H3i4K6u0W2K9s2c8E0)

----------------------------------------------------------------------------------------

【加密方式】 序列号

【破解工具】 `ODbyDYK` v1.10[05.09]

【软件限制】 时间限制

【破解平台】 Microsoft Windows XP Professional

【平台版本】 5.1.2600 Service Pack 2 内部版本号 2600

----------------------------------------------------------------------------------------

## 【软件简介】

Turbo Photo是一个以数码影像为背景，面向数码相机普通用户和准专业用户而设计的一套集图片管理，浏览，处理，输出为一身的软件系统。她包括两个部分:`Turbo Photo` 相册和`Turbo Photo` 编辑器。

`Turbo Photo` 相册是一个照片管理、浏览与批处理为一体的软件。

`Turbo Photo`编辑器的所有功能均围绕如何让您的照片更出色这样一个主题而设计。每个功能都针对了数码相机本身的特点和最常见的问题。

## 【文章简介】

网友给的一个程序，算法不是太难，不过程序中的随机验证的思路倒是可以让我们学习一下。

----------------------------------------------------------------------------------------

## 【破解过程】

注册码跟用户名无关，输入`98765-54321-123456-56789` 下断：`BP` `MessageBoxA` 很容易找到解锁按钮的响应代码：

```
0045B590   .  53            PUSH EBX
0045B591   .  55            PUSH EBP
0045B592   .  56            PUSH ESI
0045B593   .  57            PUSH EDI
0045B594   .  8BD9          MOV EBX,ECX
0045B596   .  6A 01         PUSH 1
0045B598   .  E8 45E20300   CALL <JMP.&MFC42.#6334\_CWnd::UpdateData>    ; UpdateData(true)
0045B59D   .  8DAB A0000000 LEA EBP,DWORD PTR [EBX+A0]
0045B5A3   .  8DBB A8000000 LEA EDI,DWORD PTR [EBX+A8]
0045B5A9   .  8BCD          MOV ECX,EBP
0045B5AB   .  8DB3 B4000000 LEA ESI,DWORD PTR [EBX+B4]
0045B5B1   .  E8 F2E50300   CALL <JMP.&MFC42.#4171\_CString::LockBuffer> ; S3
0045B5B6   .  50            PUSH EAX
0045B5B7   .  8BCF          MOV ECX,EDI
0045B5B9   .  E8 EAE50300   CALL <JMP.&MFC42.#4171\_CString::LockBuffer> ; S2
0045B5BE   .  50            PUSH EAX
0045B5BF   .  8D8B B0000000 LEA ECX,DWORD PTR [EBX+B0]
0045B5C5   .  E8 DEE50300   CALL <JMP.&MFC42.#4171\_CString::LockBuffer> ; S1
0045B5CA   .  50            PUSH EAX
0045B5CB   .  8BCE          MOV ECX,ESI
0045B5CD   .  E8 D6E50300   CALL <JMP.&MFC42.#4171\_CString::LockBuffer> ; S0
0045B5D2   .  50            PUSH EAX
0045B5D3   .  B9 8CA84C00   MOV ECX,TPhoto.004CA88C
0045B5D8   .  E8 53050000   CALL TPhoto.0045BB30                        ; 关键Call，要跟进
0045B5DD   .  85C0          TEST EAX,EAX
0045B5DF   .  75 1B         JNZ SHORT TPhoto.0045B5FC                   ; 关键跳转
0045B5E1   .  50            PUSH EAX
0045B5E2   .  50            PUSH EAX
0045B5E3   .  6A 47         PUSH 47
0045B5E5   .  B9 689A4C00   MOV ECX,TPhoto.004C9A68
0045B5EA   .  E8 C17D0000   CALL TPhoto.004633B0
0045B5EF   .  50            PUSH EAX
0045B5F0   .  8BCB          MOV ECX,EBX
0045B5F2   .  E8 4DE30300   CALL <JMP.&MFC42.#4224\_CWnd::MessageBoxA>   ; 错误的序列号
0045B5F7   .  5F            POP EDI
0045B5F8   .  5E            POP ESI
0045B5F9   .  5D            POP EBP
0045B5FA   .  5B            POP EBX
0045B5FB   .  C3            RETN
```

跟进后，走过一个 `S0` 跟 "`DDGTM`" 比较的循环后来到这里

```
0045BC39  |.  50            PUSH EAX                           ; /Arg4
0045BC3A  |.  51            PUSH ECX                           ; |Arg3
0045BC3B  |.  52            PUSH EDX                           ; |Arg2
0045BC3C  |.  55            PUSH EBP                           ; |Arg1
0045BC3D  |.  8D8B 04020000 LEA ECX,DWORD PTR [EBX+204]        ; |
0045BC43  |.  E8 08D40300   CALL TPhoto.00499050               ; \TPhoto.00499050
0045BC48  |.  85C0          TEST EAX,EAX
0045BC4A  |.  75 07         JNZ SHORT TPhoto.0045BC53          ; 把这里作为爆破点
0045BC4C  |.  5F            POP EDI
0045BC4D  |.  5E            POP ESI
0045BC4E  |.  5D            POP EBP
0045BC4F  |.  5B            POP EBX
0045BC50  |.  C2 1000       RETN 10
```

继续跟进 `0045BC43 CALL TPhoto.00499050` 来到：

```
00499050  /$  83EC 64       SUB ESP,64
00499053  |.  53            PUSH EBX
00499054  |.  55            PUSH EBP
00499055  |.  56            PUSH ESI
00499056  |.  57            PUSH EDI
00499057  |.  8BF1          MOV ESI,ECX
00499059  |.  E8 62040000   CALL TPhoto.004994C0               ;  (Sum(用户帐户名)+1)%10
0049905E  |.  8BCE          MOV ECX,ESI
00499060  |.  8BE8          MOV EBP,EAX
00499062  |.  E8 59040000   CALL TPhoto.004994C0               ;  (Sum(用户帐户名)+1)%10
00499067  |.  8AC8          MOV CL,AL
00499069  |.  8D7C24 10     LEA EDI,DWORD PTR [ESP+10]
0049906D  |.  8AE9          MOV CH,CL
0049906F  |.  8D546D 00     LEA EDX,DWORD PTR [EBP+EBP*2]
00499073  |.  8BC1          MOV EAX,ECX
00499075  |.  C1E0 10       SHL EAX,10
00499078  |.  66:8BC1       MOV AX,CX
0049907B  |.  B9 19000000   MOV ECX,19
00499080  |.  F3:AB         REP STOS DWORD PTR ES:[EDI]
00499082  |.  33DB          XOR EBX,EBX
00499084  |.  8D7C96 40     LEA EDI,DWORD PTR [ESI+EDX*4+40]   ;  根据 (Sum(用户帐户名)+1)%10 取得验证地址的偏移表
00499088  |>  FF15 D0684A00 /CALL DWORD PTR [<&MSVCRT.rand>]   ; [rand
0049908E  |.  99            |CDQ
0049908F  |.  B9 64000000   |MOV ECX,64
00499094  |.  F7F9          |IDIV ECX                          ;  rand() % 100
00499096  |.  0FBE5414 10   |MOVSX EDX,BYTE PTR [ESP+EDX+10]
0049909B  |.  3BEA          |CMP EBP,EDX
0049909D  |.  74 09         |JE SHORT TPhoto.004990A8
0049909F  |.  8B07          |MOV EAX,DWORD PTR [EDI]
004990A1  |.  C70486 000000>|MOV DWORD PTR [ESI+EAX*4],0
004990A8  |>  8BCE          |MOV ECX,ESI
004990AA  |.  E8 31FFFFFF   |CALL TPhoto.00498FE0              ;  第一次 GetTickCount
004990AF  |.  8B9424 840000>|MOV EDX,DWORD PTR [ESP+84]
004990B6  |.  8B8424 800000>|MOV EAX,DWORD PTR [ESP+80]
004990BD  |.  8B0F          |MOV ECX,DWORD PTR [EDI]
004990BF  |.  52            |PUSH EDX
004990C0  |.  8B9424 800000>|MOV EDX,DWORD PTR [ESP+80]
004990C7  |.  50            |PUSH EAX
004990C8  |.  8B8424 800000>|MOV EAX,DWORD PTR [ESP+80]
004990CF  |.  52            |PUSH EDX
004990D0  |.  50            |PUSH EAX
004990D1  |.  FF148E        |CALL DWORD PTR [ESI+ECX*4]        ;  [ESI+ECX*4]得到验证地址
004990D4  |.  83C4 10       |ADD ESP,10
004990D7  |.  85C0          |TEST EAX,EAX
004990D9  |.  74 1F         |JE SHORT TPhoto.004990FA
004990DB  |.  8BCE          |MOV ECX,ESI
004990DD  |.  E8 1EFFFFFF   |CALL TPhoto.00499000              ;  第二次 GetTickCount，两次结果相减判断程序有没有被单步跟踪
004990E2  |.  43            |INC EBX
004990E3  |.  83C7 04       |ADD EDI,4
004990E6  |.  83FB 03       |CMP EBX,3
004990E9  |.^ 7C 9D         \JL SHORT TPhoto.00499088
004990EB  |.  5F            POP EDI
004990EC  |.  5E            POP ESI
004990ED  |.  5D            POP EBP
004990EE  |.  B8 01000000   MOV EAX,1
004990F3  |.  5B            POP EBX
004990F4  |.  83C4 64       ADD ESP,64
004990F7  |.  C2 1000       RETN 10
```

这一段代码中放了 2 个暗桩：

1、第一个暗桩

两次 `GetTickCount` ，其中第二次将返回值相减，得到结果如果太大会清空验证地址表，若要顺利调试，要跟进 `CALL TPhoto.00499000`

```
00499000  /$  56            PUSH ESI                           ;  <TPhoto.验证地址>
00499001  |.  8BF1          MOV ESI,ECX
00499003  |.  8B86 5C010000 MOV EAX,DWORD PTR [ESI+15C]
00499009  |.  85C0          TEST EAX,EAX
0049900B  |.  74 28         JE SHORT TPhoto.00499035
0049900D  |.  FF15 6C614A00 CALL DWORD PTR [<&KERNEL32.GetTick>; [GetTickCount(第二次)
00499013  |.  2B86 58010000 SUB EAX,DWORD PTR [ESI+158]
00499019  |.  3D F4010000   CMP EAX,1F4
0049901E  |.  76 15         JBE SHORT TPhoto.00499035          ; 这里改为 JMP，跳过清空
00499020  |.  57            PUSH EDI
00499021  |.  B9 10000000   MOV ECX,10
00499026  |.  33C0          XOR EAX,EAX
```
