
# 进程执法官 V1.02 注册码的生成

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-296.htm`](https://www.pediy.com/kssd/pediy07/pediy7-296.htm)

【破文作者】 rdsnow\[BCG\]\[PYG\]\[D.4s\]

【作者主页】 [`rdsnow.ys168.com`](http://rdsnow.ys168.com)

【 E-mail 】 rdsnow@163.com

【 作者 QQ 】 83757177

【软件名称】 进程执法官 V1.02

【下载地址】 [`www5.skycn.com/soft/22666.html`](http://www5.skycn.com/soft/22666.html)

----------------------------------------------------------------------------------------------

【加密方式】 序列号

【软件限制】 功能限制

【文章简介】

重启验证，明码比较，可以很容易作内存注册注册机。让我们来跟一下注册码的形成过程，达到作出算法注册机的目标。

----------------------------------------------------------------------------------------------

【破解过程】

`peid0.93` 查，得到 `ASPack 2.12` -> `Alexey Solodovnikov`，`AspackDie1.41` 脱壳，无需修复，`Microsoft Visual C++ 6.0` 编写。

重新启动后，输入的假注册信息保存在注册表中：

`[HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet]`

`"Name"="rdsnow[BCG][PYG][D.4s]"`

`"Description"="987654321abcd"`

```
004133E0  /$  6A FF           PUSH -1
004133E2  |.  68 C01F4200     PUSH Unpacked.00421FC0                    ;  SE 句柄安装
…………
00413445  |.  8D4C24 20       LEA ECX,DWORD PTR SS:[ESP+20]
00413449  |.  50              PUSH EAX                                 ; /Arg2
0041344A  |.  68 4CF24200     PUSH Unpacked.0042F24C                    ; |Arg1 = 0042F24C  ASCII "SystemBiosVersion"
0041344F  |.  E8 3CBD0000     CALL Unpacked.0041F190                   ; \\取出系统 BIOS 的版本
00413454  |.  8B4C24 10       MOV ECX,DWORD PTR SS:[ESP+10]
00413458  |.  8D5424 08       LEA EDX,DWORD PTR SS:[ESP+8]
0041345C  |.  51              PUSH ECX
0041345D  |.  68 FCD44200     PUSH Unpacked.0042D4FC                    ;  ASCII "%s"
00413462  |.  52              PUSH EDX
00413463  |.  E8 8CBF0000     CALL <JMP.&MFC42.#2818_CString::Format>   ;  转换成字符串格式
00413468  |.  83C4 0C         ADD ESP,0C
0041346B  |.  EB 0E           JMP SHORT Unpacked.0041347B
0041346D  |>  68 44F24200     PUSH Unpacked.0042F244                    ;  ASCII "unkown"
00413472  |.  8D4C24 0C       LEA ECX,DWORD PTR SS:[ESP+C]
00413476  |.  E8 67BF0000     CALL <JMP.&MFC42.#860_CString::operator=>
0041347B  |>  8D4C24 0C       LEA ECX,DWORD PTR SS:[ESP+C]
0041347F  |.  E8 2EBF0000     CALL <JMP.&MFC42.#540_CString::CString>
00413484  |.  8D4424 1C       LEA EAX,DWORD PTR SS:[ESP+1C]
00413488  |.  C64424 30 03    MOV BYTE PTR SS:[ESP+30],3
0041348D  |.  50              PUSH EAX
0041348E  |.  E8 FDFeFFFF     CALL Unpacked.00413390                   ;  取得处理器的信息
00413493  |.  8B08            MOV ECX,DWORD PTR DS:[EAX]
00413495  |.  8B5424 0C       MOV EDX,DWORD PTR SS:[ESP+C]
00413499  |.  51              PUSH ECX
0041349A  |.  52              PUSH EDX
0041349B  |.  8D4424 18       LEA EAX,DWORD PTR SS:[ESP+18]
0041349F  |.  68 3CF24200     PUSH Unpacked.0042F23C                    ;  ASCII "%s-%s"
004134A4  |.  50              PUSH EAX
004134A5  |.  C64424 44 04    MOV BYTE PTR SS:[ESP+44],4
004134AA  |.  E8 45BF0000     CALL <JMP.&MFC42.#2818_CString::Format>   ;  按"BIOS 版本-CPU 信息"格式，生成程序的机器码
004134AF  |.  83C4 14         ADD ESP,14
004134B2  |.  8D4C24 1C       LEA ECX,DWORD PTR SS:[ESP+1C]
004134B6  |.  C64424 30 03    MOV BYTE PTR SS:[ESP+30],3
004134BB  |.  E8 E0BE0000     CALL <JMP.&MFC42.#800_CString::~CString>
004134C0  |.  8D4C24 18       LEA ECX,DWORD PTR SS:[ESP+18]
004134C4  |.  E8 57460000     CALL Unpacked.00417B20
004134C9  |.  51              PUSH ECX
004134CA  |.  C64424 34 05    MOV BYTE PTR SS:[ESP+34],5
004134CF  |.  8BCC            MOV ECX,ESP
004134D1  |.  896424 20       MOV DWORD PTR SS:[ESP+20],ESP
004134D5  |.  68 34F24200     PUSH Unpacked.0042F234                    ;  ASCII "Name"
004134DA  |.  E8 37BE0000     CALL <JMP.&MFC42.#537_CString::CString>
004134DF  |.  8D4C24 20       LEA ECX,DWORD PTR SS:[ESP+20]             ; |
004134E3  |.  81C6 34140000   ADD ESI,1434                              ; |
004134E9  |.  51              PUSH ECX                                  ; |Arg1
004134EA  |.  8BCE            MOV ECX,ESI                               ; |
004134EC  |.  E8 CF740000     CALL Unpacked.0041A9C0                   ; \\取出输入的用户名
004134F1  |.  8B10            MOV EDX,DWORD PTR DS:[EAX]
004134F3  |.  8B4424 0C       MOV EAX,DWORD PTR SS:[ESP+C]
004134F7  |.  52              PUSH EDX
004134F8  |.  50              PUSH EAX
004134F9  |.  8D4C24 10       LEA ECX,DWORD PTR SS:[ESP+10]
004134FD  |.  68 3CF24200     PUSH Unpacked.0042F23C                    ;  ASCII "%s-%s"
00413502  |.  51              PUSH ECX
00413503  |.  C64424 40 06    MOV BYTE PTR SS:[ESP+40],6
00413508  |.  E8 E7BE0000     CALL <JMP.&MFC42.#2818_CString::Format>   ;  生成字符串"机器码-用户名"
0041350D  |.  83C4 10         ADD ESP,10
00413510  |.  8D4C24 1C       LEA ECX,DWORD PTR SS:[ESP+1C]
00413514  |.  C64424 30 05    MOV BYTE PTR SS:[ESP+30],5
00413519  |.  E8 82BE0000     CALL <JMP.&MFC42.#800_CString::~CString>
0041351E  |.  51              PUSH ECX
0041351F  |.  8D5424 0C       LEA EDX,DWORD PTR SS:[ESP+C]
00413523  |.  8BCC            MOV ECX,ESP
00413525  |.  896424 20       MOV DWORD PTR SS:[ESP+20],ESP
00413529  |.  52              PUSH EDX
0041352A  |.  E8 BFBE0000     CALL <JMP.&MFC42.#535_CString::CString>
0041352F  |.  8D4424 18       LEA EAX,DWORD PTR SS:[ESP+18]
00413533  |.  50              PUSH EAX
00413534  |.  E8 27460000     CALL Unpacked.00417B60                   ;  生成真注册码
00413539  |.  83C4 04         ADD ESP,4
0041353C  |.  C64424 34 07    MOV BYTE PTR SS:[ESP+34],7
00413541  |.  8BCC            MOV ECX,ESP
00413543  |.  896424 20       MOV DWORD PTR SS:[ESP+20],ESP
00413547  |.  68 28F24200     PUSH Unpacked.0042F228                    ;  ASCII "Description"
0041354C  |.  E8 C5BD0000     CALL <JMP.&MFC42.#537_CString::CString>
00413551  |.  8D4C24 20       LEA ECX,DWORD PTR SS:[ESP+20]             ; |
00413555  |.  51              PUSH ECX                                  ; |Arg1
00413556  |.  8BCE            MOV ECX,ESI                               ; |
00413558  |.  E8 63740000     CALL Unpacked.0041A9C0                   ; \\从注册表中取出假注册码
0041355D  |.  8B00            MOV EAX,DWORD PTR DS:[EAX]
0041355F  |.  8B5424 14       MOV EDX,DWORD PTR SS:[ESP+14]
00413563  |.  50              PUSH EAX                                  ; /s2
00413564  |.  52              PUSH EDX                                  ; |s1
00413565  |.  FF15 30474200   CALL DWORD PTR DS:[<&MSVCRT._mbscmp>]     ; \\真假注册码进行比较
0041356B  |.  83C4 08         ADD ESP,8
0041356E  |.  8D4C24 1C       LEA ECX,DWORD PTR SS:[ESP+1C]
00413572  |.  85C0            TEST EAX,EAX
00413574  |.  0F94C3          SETE BL
00413577  |.  E8 24BE0000     CALL <JMP.&MFC42.#800_CString::~CString>
0041357C  |.  84DB            TEST BL,BL
0041357E  |.  C64424 30 05    MOV BYTE PTR SS:[ESP+30],5
00413583  |.  8D4C24 14       LEA ECX,DWORD PTR SS:[ESP+14]
00413587  |.  74 64           JE SHORT Unpacked.004135ED                ;  跳走就挂了
00413589  |.  E8 12BE0000     CALL <JMP.&MFC42.#800_CString::~CString>
```

想要作出算法注册机，就跟进 `00413534 CALL Unpacked.00417B60`：

下面为了表示方便：

将"机器码－用户名"这个字符串简称：`SA`

将"ProcessJudgerhaha1234567890ABCDEFGHIJKLMNOPQRSTUVWXYZ"这个字符串简称：`SB`

```
00417B60  /$  6A FF           PUSH -1
00417B62  |.  68 97264200     PUSH Unpacked.00422697                    ;  SE 句柄安装
00417B67  |.  64:A1 00000000  MOV EAX,DWORD PTR FS:[0]
00417B6D  |.  50              PUSH EAX
00417B6E  |.  64:8925 000000>MOV DWORD PTR FS:[0],ESP
00417B75  |.  83EC 0C         SUB ESP,0C
00417B78  |.  53              PUSH EBX
00417B79  |.  55              PUSH EBP
```
