
# MailScan V3.5.1.2

> 原文：[`www.pediy.com/kssd/pediy03/forum1283.htm`](https://www.pediy.com/kssd/pediy03/forum1283.htm)

软件名称：`MailScan` 电子邮件地址过滤器

软件下载：

`http://www.newhua.com.cn/down/mailscan.zip`

现在已经有 `V4.00 Bata 2.3` 测试版，功能更加强大，软件下载：

`http://heraldsoft.yeah.net`

`http://mailscan.yeah.net`

这个软件可以从文本文件中过滤出电子邮件地址，这里所指的文本文件不仅仅指 `*.txt` 文件，还包括 `*.htm` 文件、`*.html` 文件、`*.csv`、`*.eml`、`*.mbx` 等文本格式的文件。它可以在这些文件中搜寻可能包含邮件地址的文本行，然后对这些内容进行处理，过滤掉重复的和非法的地址，然后按照服务器地址自动排序，你存盘后就得到了一个 E-Mail 地址表。然后，你就可以轻松使用这些地址来进行你的宣传活动了。

### 1. 用 W32Dasm 反汇编：

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0048899F(C)

|

:00488A02 A14CC24800              mov eax, dword ptr [0048C24C]
:00488A07 8B00                    mov eax, dword ptr [eax]
:00488A09 8B10                    mov edx, dword ptr [eax]
:00488A0B FF92D8000000            call dword ptr [edx+000000D8]
:00488A11 48                       dec eax
:00488A12 0F85B6000000            jne 00488ACE
:00488A18 8D55FC                  lea edx, dword ptr [ebp-04]
:00488A1B A14CC24800              mov eax, dword ptr [0048C24C]
:00488A20 8B00                    mov eax, dword ptr [eax]
:00488A22 8B80DC020000            mov eax, dword ptr [eax+000002DC]
:00488A28 E83B43FAFF              call 0042CD68
:00488A2D 8D55F8                  lea edx, dword ptr [ebp-08]
:00488A30 A14CC24800              mov eax, dword ptr [0048C24C]
:00488A35 8B00                    mov eax, dword ptr [eax]
:00488A37 8B80E0020000            mov eax, dword ptr [eax+000002E0]
:00488A3D E82643FAFF              call 0042CD68
:00488A42 8B4DF8                  mov ecx, dword ptr [ebp-08]
:00488A45 8B55FC                  mov edx, dword ptr [ebp-04]
:00488A48 8B834C030000            mov eax, dword ptr [ebx+0000034C]
:00488A4E E8DDACFFFF              call 00483730    <======== 关键 call，用 F8 追入
:00488A53 84C0                    test al, al
:00488A55 745F                    je 00488AB6     <======== 关键跳转
:00488A57 6A00                    push 00000000

* Reference To: user32.MessageBeep, Ord:0000h

                                  |
:00488A59 E836F1F7FF              Call 00407B94

* Possible StringData Ref from Code Obj ->"E-Mail 地址捕捉器"

                                  |
:00488A5E BA008B4800              mov edx, 00488B00
:00488A63 8BC3                    mov eax, ebx
:00488A65 E82E43FAFF              call 0042CD98
:00488A6A 33D2                    xor edx, edx
:00488A6C 8B8310030000            mov eax, dword ptr [ebx+00000310]
:00488A72 8B08                    mov ecx, dword ptr [eax]
:00488A74 FF515C                  call [ecx+5C]
:00488A77 8B834C030000            mov eax, dword ptr [ebx+0000034C]
:00488A7D E81AA9FFFF              call 0048339C
:00488A82 B201                    mov dl, 01
:00488A84 8B83FC020000            mov eax, dword ptr [ebx+000002FC]
:00488A8A 8B08                    mov ecx, dword ptr [eax]
:00488A8C FF515C                  call [ecx+5C]
:00488A8F B201                    mov dl, 01
:00488A91 8B8300030000            mov eax, dword ptr [ebx+00000300]
:00488A97 8B08                    mov ecx, dword ptr [eax]
:00488A99 FF515C                  call [ecx+5C]
:00488A9C 6A40                    push 00000040

* Possible StringData Ref from Code Obj ->"注册成功"

                                  |
:00488A9E B9148B4800              mov ecx, 00488B14

* Possible StringData Ref from Code Obj ->"祝贺你,你现在可以长期使用这个软件了.
作者感谢"
                                ->"您对国产软件的尊重和支持."

                                  |
:00488AA3 BA208B4800              mov edx, 00488B20
:00488AA8 A188C04800              mov eax, dword ptr [0048C088]
:00488AAD 8B00                    mov eax, dword ptr [eax]
:00488AAF E8E421FCFF              call 0044AC98
:00488AB4 EB18                    jmp 00488ACE

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00488A55(C)                     <======== 从“关键跳转”跳过来
|
:00488AB6 6A40                    push 00000040

* Possible StringData Ref from Code Obj ->"警告"

                                  |
:00488AB8 B9688B4800              mov ecx, 00488B68

* Possible StringData Ref from Code Obj ->"注册码不对!"     <======== 注册错误信息

                                  |
:00488ABD BA708B4800              mov edx, 00488B70
:00488AC2 A188C04800              mov eax, dword ptr [0048C088]
:00488AC7 8B00                    mov eax, dword ptr [eax]
:00488AC9 E8CA21FCFF              call 0044AC98
```

### 2. 用 TRW2000 在 00488A4E 处下断点并用 F8 追入：

```
0167:00483751  CALL     0040407C
0167:00483756  XOR      EAX,EAX
0167:00483758  PUSH     EBP
0167:00483759  PUSH     DWORD 004837F6
0167:0048375E  PUSH     DWORD [FS:EAX]
0167:00483761  MOV      [FS:EAX],ESP
0167:00483764  LEA      ECX,[EBP-0C]
0167:00483767  MOV      EDX,[EBP-04]
0167:0048376A  MOV      EAX,ESI
0167:0048376C  CALL     00483820
0167:00483771  MOV      EAX,[EBP-0C]
0167:00483774  MOV      EDX,[EBP-08]
0167:00483777  CALL     00403FD8    <======== 关键 call
0167:0048377C  JNZ      004837D9
0167:0048377E  MOV      BL,01
```

在 `483777` 处找到：

```
d eax=1829191098
d edx=87654321
```

用户号：`06901-OEM-0088401-20426`

注册码：`1829191098`

换一台计算机安装：

用户号：`26099-OEM-0044402-16090`

注册码：`-2120266114` (出现负数似乎是错的，但仍然注册成功) `4dd466d0e1e0cc123afbe504da2462ca`
