
# 逆向 Becky! Internet Mail version 2 密码加密算法

> 原文：[`www.pediy.com/kssd/pediy12/128082.html`](https://www.pediy.com/kssd/pediy12/128082.html)

【文章标题】:逆向 Becky! Internet Mail version 2 密码加密算法  
【文章作者】: `crosoli`  
【作者邮箱】: `crosoli@126.com`  
【软件名称】: Becky! Internet Mail  
【下载地址】: 自己搜索下载  
【编写语言】: `VC6.0`  
【软件介绍】: 邮件收取工具  
【作者声明】: 出于学习的目的  

---

## 详细过程

1. 上个月买了本段刚老师的加密解密 3，看完了第五章（只能下班的时间看），总感觉作者很了解我。这本书太适合我了。

2. 分析过程。

### 步骤

1. 首先打开 Becky! Internet Mail，配置好自己的邮件信息。
2. 然后 `OD` 载入 `B2.exe`，`F9` 运行后，右键点击邮件->属性，然后修改密码，此时对 `GetWindowTextA` 下断点。会发现配件的所有信息都会读入到内存。让发现密码明文读取到内存后，然后数据窗口跟随，找到密码的明文，并对它下内存断点。并移除先前的 `GetWindowTextA` 断点。

`F9` 运行后程序来到:

```
0051A8C5  |.  85DB          TEST EBX,EBX
0051A8C7  |.  75 3A         JNZ SHORT B2.0051A903
0051A8C9  |.  8B86 B8000000 MOV EAX,DWORD PTR DS:[ESI+B8]             ;   新密码
0051A8CF  |.  8BAF B8000000 MOV EBP,DWORD PTR DS:[EDI+B8]             ;   老密码
0051A8D5  |>  8A55 00        MOV DL,BYTE PTR SS:[EBP]                 ;  取老密码一位
0051A8D8  |.  8ACA          MOV CL,DL
0051A8DA  |.  3A10          CMP DL,BYTE PTR DS:[EAX]                  ;  取新密码一位
0051A8DC  |.  75 1C         JNZ SHORT B2.0051A8FA                     ;  密码不相同跳转
0051A8DE  |.  84C9          TEST CL,CL
0051A8E0  |.  74 14         JE SHORT B2.0051A8F6
0051A8E2  |.  8A55 01       MOV DL,BYTE PTR SS:[EBP+1]                ;  继续比较
0051A8E5  |.  8ACA          MOV CL,DL
0051A8E7  |.  3A50 01       CMP DL,BYTE PTR DS:[EAX+1]
0051A8EA  |.  75 0E         JNZ SHORT B2.0051A8FA
0051A8EC  |.  83C5 02       ADD EBP,2
0051A8EF  |.  83C0 02       ADD EAX,2
0051A8F2  |.  84C9          TEST CL,CL
0051A8F4  |.^ 75 DF         \JNZ SHORT B2.0051A8D5
0051A8F6  |>  33C0          XOR EAX,EAX
0051A8F8  |.  EB 05         JMP SHORT B2.0051A8FF
0051A8FA  |>  1BC0          SBB EAX,EAX                                ;   
0051A8FC  |.  83D8 FF       SBB EAX,-1
0051A8FF  |>  85C0          TEST EAX,EAX
0051A901  |.  74 5F         JE SHORT B2.0051A962                      ; 原密码和新密码相同时候跳过不写入
0051A903  |>  8DAE B8000000 LEA EBP,DWORD PTR DS:[ESI+B8]
0051A909  |.  8D8F B8000000 LEA ECX,DWORD PTR DS:[EDI+B8]
0051A90F  |.  55            PUSH EBP
0051A910  |.  E8 22280800   CALL B2.0059D137
0051A915  |.  8B6D 00       MOV EBP,DWORD PTR SS:[EBP]
0051A918  |.  A1 38685F00   MOV EAX,DWORD PTR DS:[5F6838]
0051A91D  |.  55            PUSH EBP
0051A91E  |.  68 00100000   PUSH 1000
0051A923  |.  8D4C24 24     LEA ECX,DWORD PTR SS:[ESP+24]
0051A927  |.  894424 24     MOV DWORD PTR SS:[ESP+24],EAX
0051A92B  |.  E8 FE2A0800   CALL B2.0059D42E
0051A930  |.  50            PUSH EAX
0051A931  |.  E8 4A00F7FF   CALL B2.0048A980                          ;对新密码进行加密处理。
0051A936  |.  6A FF         PUSH -1
0051A938  |.  8D4C24 20     LEA ECX,DWORD PTR SS:[ESP+20]
0051A93C  |.  E8 3C2B0800   CALL B2.0059D47D
0051A941  |.  8B0E          MOV ECX,DWORD PTR DS:[ESI]
0051A943  |.  8B5424 1C     MOV EDX,DWORD PTR SS:[ESP+1C]
0051A947  |.  51            PUSH ECX                                   ; /FileName
0051A948  |.  52            PUSH EDX                                   ; |String
0051A949  |.  68 B8B35E00   PUSH B2.005EB3B8                            ; |Key = "PassWd"
0051A94E  |.  68 E4E15E00   PUSH B2.005EE1E4                            ; |Section = "Account"
0051A953  |.  FF15 64225C00 CALL DWORD PTR DS:[<&KERNEL32.WritePriva>]; \WritePrivateProfileStringA    把加密后的新密码写入配置文件的"PassWd"字段
```

下面来分析 `0048A980` 这个函数

```
0051A931  |.  E8 4A00F7FF   CALL B2.0048A980                          ;对新密码进行加密处理。
```

```
0048A980  /$  B8 00800000   MOV EAX,8000
0048A985  |.  E8 268F0F00   CALL B2.005838B0
0048A98A  |.  8B8424 088000>MOV EAX,DWORD PTR SS:[ESP+8008]           ;EAX 新密码
0048A991  |.  56            PUSH ESI
0048A992  |.  8038 00       CMP BYTE PTR DS:[EAX],0                   ;判断新密码是不是为空
0048A995  |.  75 14         JNZ SHORT B2.0048A9AB
0048A997  |.  8B8424 088000>MOV EAX,DWORD PTR SS:[ESP+8008]
0048A99E  |.  5E            POP ESI
0048A99F  |.  C600 00       MOV BYTE PTR DS:[EAX],0
0048A9A2  |.  81C4 00800000 ADD ESP,8000
0048A9A8  |.  C2 0800       RETN 8
0048A9AB  |>  8D4C24 04     LEA ECX,DWORD PTR SS:[ESP+4]
0048A9AF  |.  50            PUSH EAX                                   ; /Arg2
0048A9B0  |.  51            PUSH ECX                                   ; |Arg1
0048A9B1  |.  E8 8AF5FFFF   CALL B2.00489F40                           ; \\B2.00489F40      第一次对新密码进行 00489F40 加密处理
0048A9B6  |.  8D5424 04     LEA EDX,DWORD PTR SS:[ESP+4]
0048A9BA  |.  8BB424 088000>MOV ESI,DWORD PTR SS:[ESP+8008]
0048A9C1  |.  52            PUSH EDX                                   ; /Arg2
0048A9C2  |.  56            PUSH ESI                                   ; |Arg1
0048A9C3  |.  E8 78F5FFFF   CALL B2.00489F40                           ; \\B2.00489F40      2 次加密
0048A9C8  |.  8A06          MOV AL,BYTE PTR DS:[ESI]
0048A9CA  |.  84C0          TEST AL,AL
0048A9CC  |.  74 0C         JE SHORT B2.0048A9DA
0048A9CE  |>  34 02         /XOR AL,2                                   ;把 2 次加密码后的结果和 2 进行 xor
0048A9D0  |.  8806          |MOV BYTE PTR DS:[ESI],AL
0048A9D2  |.  8A46 01       |MOV AL,BYTE PTR DS:[ESI+1]
0048A9D5  |.  46            |INC ESI
0048A9D6  |.  84C0          |TEST AL,AL
0048A9D8  |.^ 75 F4         \JNZ SHORT B2.0048A9CE                     ;循环 xor
0048A9DA  |>  5E            POP ESI
0048A9DB  |.  81C4 00800000 ADD ESP,8000
0048A9E1  \\.  C2 0800       RETN 8
```

下面来分析 `00489F40` 这个加密函数

```
00489F40  /$  83EC 0C       SUB ESP,0C
00489F43  |.  8B4424 10     MOV EAX,DWORD PTR SS:[ESP+10]
00489F47  |.  53            PUSH EBX
00489F48  |.  57            PUSH EDI
00489F49  |.  8B7C24 1C     MOV EDI,DWORD PTR SS:[ESP+1C]
00489F4D  |.  32DB          XOR BL,BL
00489F4F  |.  894424 08     MOV DWORD PTR SS:[ESP+8],EAX
00489F53  |.  8A0F          MOV CL,BYTE PTR DS:[EDI]
00489F55  |.  3ACB          CMP CL,BL
00489F57  |.  0F84 CF000000 JE B2.0048A02C
00489F5D  |.  55            PUSH EBP
00489F5E  |.  56            PUSH ESI
00489F5F  |>  47            /INC EDI
00489F60  |.  884C24 14     |MOV BYTE PTR SS:[ESP+14],CL              ;base64 加密。
00489F64  |.  885C24 16     |MOV BYTE PTR SS:[ESP+16],BL
00489F68  |.  885C24 15     |MOV BYTE PTR SS:[ESP+15],BL
00489F6C  |.  8A0F          |MOV CL,BYTE PTR DS:[EDI]
00489F6E  |.  BE 01000000   |MOV ESI,1
00489F73  |.  3ACB          |CMP CL,BL
00489F75  |.  74 1A         |JE SHORT B2.00489F91
```

由于篇幅的文件就不对 BASE64 进行分析了。

其实 `00489F40` 就是一个 BASE64 的加密。

综上所述 他的密码的保存方式就是 2 次 BASE64 后 `xor2`。

附上 Becky!的密码破解代码；

`break mail.rar`
