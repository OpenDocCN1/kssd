
# 流星网络电视 V1.9.2.0 的注册

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-367.htm`](https://www.pediy.com/kssd/pediy07/pediy7-367.htm)

**【破文作者】** `rdsnow[BCG][PYG][D.4s]`

**【作者主页】** `http://rdsnow.ys168.com`

**【 E-mail 】** `rdsnow@163.com`

**【 作者 QQ 】** `83757177`

**【文章题目】** 流星网络电视 V1.9.2.0 的注册

**【软件名称】** 流星网络电视 1.9.2.0

**【下载地址】** `http://www.jesen.cn`

**【文章简介】**

这个程序原先版本加了 `Asprotect 2.11` 的加密外壳的，一直没有能够脱壳，不知道什么原因，新版本的程序换成 `Aspack 2.12` 的压缩壳，手脱或用工具都可以轻松搞定。

程序采用了现在流行的重启验证的方式。

加密过程中主要使用到了两个加密算法：`MD5` 和 `DES`，都没有变形。其中 `DES` 有两个函数，`DES_EN( )` 和 `DES_DE( )`，分别用来加密信息和解密信息，使用同一个密钥。

---

**【破解过程】**

启动程序，输入注册名和假码，来到这里：(注册码的长度是 16、20、24 或 32)

```
005BD888   |.   55               PUSH EBP
005BD889   |.   68 F5DA5B00      PUSH unpacked.005BDAF5
005BD88E   |.   64:FF30          PUSH DWORD PTR FS:[EAX]
005BD891   |.   64:8920         MOV DWORD PTR FS:[EAX],ESP
005BD894   |.   8D55 EC         LEA EDX,DWORD PTR SS:[EBP-14]
005BD897   |.   8B83 14030000   MOV EAX,DWORD PTR DS:[EBX+314]
005BD89D   |.   E8 7275E9FF     CALL unpacked.00454E14            ;  取出输入的用户名
005BD8A2   |.   8B45 EC         MOV EAX,DWORD PTR SS:[EBP-14]
005BD8A5   |.   8D55 F0         LEA EDX,DWORD PTR SS:[EBP-10]
005BD8A8   |.   E8 CFB5E4FF     CALL unpacked.00408E7C
005BD8AD   |.   837D F0 00      CMP DWORD PTR SS:[EBP-10],0       ;  检查是否成功取出
005BD8B1   |.   0F84 E9010000   JE unpacked.005BDAA0
005BD8B7   |.   8D55 E4         LEA EDX,DWORD PTR SS:[EBP-1C]
005BD8BA   |.   8B83 18030000   MOV EAX,DWORD PTR DS:[EBX+318]
005BD8C0   |.   E8 4F75E9FF     CALL unpacked.00454E14            ;  取出输入的注册码
005BD8C5   |.   8B45 E4         MOV EAX,DWORD PTR SS:[EBP-1C]
005BD8C8   |.   8D55 E8         LEA EDX,DWORD PTR SS:[EBP-18]
005BD8CB   |.   E8 ACB5E4FF     CALL unpacked.00408E7C
005BD8D0   |.   837D E8 00      CMP DWORD PTR SS:[EBP-18],0       ;  检查是否成功取出
005BD8D4   |.   0F84 C6010000   JE unpacked.005BDAA0
005BD8DA   |.   8D55 E0         LEA EDX,DWORD PTR SS:[EBP-20]
005BD8DD   |.   8B83 14030000   MOV EAX,DWORD PTR DS:[EBX+314]
005BD8E3   |.   E8 2C75E9FF     CALL unpacked.00454E14            ;  计算用户名的长度
005BD8E8   |.   8B45 E0         MOV EAX,DWORD PTR SS:[EBP-20]
005BD8EB   |.   8D55 F8         LEA EDX,DWORD PTR SS:[EBP-8]
005BD8EE   |.   E8 89B5E4FF     CALL unpacked.00408E7C
005BD8F3   |.   8D55 D8         LEA EDX,DWORD PTR SS:[EBP-28]     ;  下面开始对注册码变换
005BD8F6   |.   8B83 18030000   MOV EAX,DWORD PTR DS:[EBX+318]
005BD8FC   |.   E8 1375E9FF     CALL unpacked.00454E14            ;  取出注册码的长度
005BD901   |.   8B45 D8         MOV EAX,DWORD PTR SS:[EBP-28]
005BD904   |.   8D55 DC         LEA EDX,DWORD PTR SS:[EBP-24]
005BD907   |.   E8 70B5E4FF     CALL unpacked.00408E7C
005BD90C   |.   8B45 DC         MOV EAX,DWORD PTR SS:[EBP-24]
005BD90F   |.   8D55 FC         LEA EDX,DWORD PTR SS:[EBP-4]
005BD912   |.   E8 11B3E4FF     CALL unpacked.00408C28            ;  注册码大写转为小写
005BD917   |.   8D55 D4         LEA EDX,DWORD PTR SS:[EBP-2C]
005BD91A   |.   8B45 FC         MOV EAX,DWORD PTR SS:[EBP-4]
005BD91D   |.   E8 52A5EBFF     CALL unpacked.00477E74            ;  对注册码采用 MD5 运算
005BD922   |.   8B45 D4         MOV EAX,DWORD PTR SS:[EBP-2C]
005BD925   |.   8D4D F4         LEA ECX,DWORD PTR SS:[EBP-C]
005BD928   |.   BA 0CDB5B00     MOV EDX,unpacked.005BDB0C         ;  ASCII "dream"
005BD92D   |.   E8 5697EBFF     CALL unpacked.00477088            ;  再经过 DES 加密，Key="dream"因为密钥至少要有 64 位，不够用 0x00 替代
005BD932   |.   8B45 FC         MOV EAX,DWORD PTR SS:[EBP-4]
005BD935   |.   E8 D66DE4FF     CALL unpacked.00404710            ;  计算输入注册码的长度
005BD93A   |.   83E8 10         SUB EAX,10                        ;  Switch (cases 10..20)
005BD93D   |.   74 11           JE SHORT unpacked.005BD950
005BD93F   |.   83E8 04         SUB EAX,4
005BD942   |.   74 26           JE SHORT unpacked.005BD96A
005BD944   |.   83E8 04         SUB EAX,4
005BD947   |.   74 3B           JE SHORT unpacked.005BD984
005BD949   |.   83E8 08         SUB EAX,8
005BD94C   |.   74 50           JE SHORT unpacked.005BD99E
005BD94E   |.   EB 68           JMP SHORT unpacked.005BD9B8
005BD950   |>  A1 586A5C00     MOV EAX,DWORD PTR DS:[5C6A58]      ;  Case 10 of switch 005BD93A
005BD955   |.   50              PUSH EAX
005BD956   |.   B9 0A000000     MOV ECX,0A
005BD95B   |.   BA 06000000     MOV EDX,6
005BD960   |.   8B45 F4         MOV EAX,DWORD PTR SS:[EBP-C]
005BD963   |.   E8 0870E4FF     CALL unpacked.00404970            ;  如果输入 16 位注册码，从 DES 结果的第 6 位起取 10 个字符
005BD968   |.   EB 6B           JMP SHORT unpacked.005BD9D5
005BD96A   |>  A1 586A5C00     MOV EAX,DWORD PTR DS:[5C6A58]      ;  Case 14 of switch 005BD93A
005BD96F   |.   50              PUSH EAX
005BD970   |.   B9 0E000000     MOV ECX,0E
005BD975   |.   BA 03000000     MOV EDX,3
005BD97A   |.   8B45 F4         MOV EAX,DWORD PTR SS:[EBP-C]
005BD97D   |.   E8 EE6FE4FF     CALL unpacked.00404970            ;  如果输入 20 位注册码，从 DES 结果的第 3 位起取 14 个字符
005BD982   |.   EB 51           JMP SHORT unpacked.005BD9D5
005BD984   |>  A1 586A5C00     MOV EAX,DWORD PTR DS:[5C6A58]      ;  Case 18 of switch 005BD93A
005BD989   |.   50              PUSH EAX
005BD98A   |.   B9 10000000     MOV ECX,10
005BD98F   |.   BA 05000000     MOV EDX,5
005BD994   |.   8B45 F4         MOV EAX,DWORD PTR SS:[EBP-C]
005BD997   |.   E8 D46FE4FF     CALL unpacked.00404970            ;  如果输入 24 位注册码，从 DES 结果的第 5 位起取 16 个字符
005BD99C   |.   EB 37           JMP SHORT unpacked.005BD9D5
005BD99E   |>  A1 586A5C00     MOV EAX,DWORD PTR DS:[5C6A58]      ;  Case 20 of switch 005BD93A
005BD9A3   |.   50              PUSH EAX
005BD9A4   |.   B9 0C000000     MOV ECX,0C
005BD9A9   |.   BA 0C000000     MOV EDX,0C
005BD9AE   |.   8B45 F4         MOV EAX,DWORD PTR SS:[EBP-C]
005BD9B1   |.   E8 BA6FE4FF     CALL unpacked.00404970            ;  如果输入 32 位注册码，从 DES 结果的第 12 位起取 32 个字符
005BD9B6   |.   EB 1D           JMP SHORT unpacked.005BD9D5       ;  以上已经完成了对注册码的处理，取出的字符等待存入注册表
005BD9B8   |>  6A 30           PUSH 30                             ;  Default case of switch 005BD93A
005BD9BA   |.   B9 14DB5B00     MOV ECX,unpacked.005BDB14         ;  如果输入的注册码不是以上长度，就到这里，OVER 了
005BD9BF   |.   BA 20DB5B00     MOV EDX,unpacked.005BDB20
005BD9C4   |.   A1 A46C5C00     MOV EAX,DWORD PTR DS:[5C6CA4]
005BD9C9   |.   8B00           MOV EAX,DWORD PTR DS:[EAX]
005BD9CB   |.   E8 A477EBFF     CALL unpacked.00475174
005BD9D0   |.   E9 CB000000     JMP unpacked.005BDAA0             ;  这个 JMP，跳向注册码错误的对话框
005BD9D5   |>  B2 01           MOV DL,1
005BD9D7   |.   A1 FCD34300     MOV EAX,DWORD PTR DS:[43D3FC]
005BD9DC   |.   E8 1BFBE7FF     CALL unpacked.0043D4FC
005BD9E1   |.   8BD8           MOV EBX,EAX
005BD9E3   |.   BA 02000080     MOV EDX,80000002
005BD9E8   |.   8BC3           MOV EAX,EBX
005BD9EA   |.   E8 ADFBE7FF     CALL unpacked.0043D59C
005BD9EF   |.   B1 01           MOV CL,1
005BD9F1   |.   BA 48DB5B00     MOV EDX,unpacked.005BDB48         ;  ASCII "\\SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\zlb"
005BD9F6   |.   8BC3           MOV EAX,EBX
005BD9F8   |.   E8 07FCE7FF     CALL unpacked.0043D604
005BD9FD   |.   84C0           TEST AL,AL
005BD9FF   |.   74 14           JE SHORT unpacked.005BDA15
005BDA01   |.   8B0D 586A5C00  MOV ECX,DWORD PTR DS:[5C6A58]     ;  unpacked.005C92E0
005BDA07   |.   8B09           MOV ECX,DWORD PTR DS:[ECX]
005BDA09   |.   BA 0CDB5B00     MOV EDX,unpacked.005BDB0C         ;  ASCII "dream"
005BDA0E   |.   8BC3           MOV EAX,EBX
005BDA10   |.   E8 3300E8FF     CALL unpacked.0043DA48            ;  下面代码是将注册码的处理结果存入注册表中
005BDA15   |>  8BC3           MOV EAX,EBX
005BDA17   |.   E8 50FBE7FF     CALL unpacked.0043D56C
005BDA1C   |.   8BC3           MOV EAX,EBX
005BDA1E   |.   E8 E15BE4FF     CALL unpacked.00403604
```

点击 `OK` 后重新启动程序，找到字符串 `dream` 或 `whole` 来到这儿：
