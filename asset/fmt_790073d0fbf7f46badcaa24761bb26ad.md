
# Super Utilities Standard 9.6.11 算法分析 (MD5 算法)

> 原文：[`www.pediy.com/kssd/pediy10/93217.html`](https://www.pediy.com/kssd/pediy10/93217.html)

- **破文标题**：Super Utilities Standard 9.6.11 算法分析
- **破文作者**：tianxj
- **作者邮箱**：`tianxj_2007@126.com`
- **作者主页**：`WwW.ChiNaPYG.CoM`
- **破解工具**：`PEiD`, `OD`
- **破解平台**：`Windows XP`
- **软件名称**：`Super Utilities Standard 9.6.11`
- **软件大小**：4234KB
- **软件类别**：国外软件/系统辅助
- **软件授权**：共享版
- **软件语言**：英文
- **运行环境**：`Win9x/Me/NT/2000/XP/2003`
- **更新时间**：2009-7-9 8:00:02
- **原版下载**：`http://www.onlinedown.net/soft/16811.htm`
- **保护方式**：注册码
- **软件简介**：Super Utilities 一套强力系统工具。包含了磁盘清理，注册表清理，程序卸载，自启动管理，内存优化及目录文件夹保护，历史记录清除，进程管理、IE 保护等大量有用的功能
- **破解声明**：我是一只小菜鸟，偶得一点心得，愿与大家分享 :)

## 破解内容

### 一、运行程序，进行注册
输入错误的注册信息进行检测，无提示信息。

### 二、查壳
用 `PEiD` 对 `SuperUtil.exe` 查壳，为 `UPX 0.89.6 - 1.02 / 1.05 - 1.24 -> Markus & Laszlo`。

### 三、调试
打开 `OD`，带壳调试，附加进程。

代码：

```
未命名的窗口, 条目 9
 进程=00000318
 名称=SuperUtil
 窗口=Register
 路径=C:\Program Files\SuperLogix\Super Utilities\SuperUtil.exe
```

下万能断点，断到关键之处。

代码：

```
005F19AC    FF75 F4         PUSH    DWORD PTR SS:[EBP-C]           ; //第 1 组试炼码
005F19AF    68 6C1B5F00     PUSH    SuperUti.005F1B6C              ; //"-"
005F19B4    8D55 F0         LEA     EDX, DWORD PTR SS:[EBP-10]
005F19B7    8B03            MOV     EAX, DWORD PTR DS:[EBX]
005F19B9    8B80 1C030000   MOV     EAX, DWORD PTR DS:[EAX+31C]
005F19BF    E8 84F2EEFF     CALL    SuperUti.004E0C48
005F19C4    FF75 F0         PUSH    DWORD PTR SS:[EBP-10]          ; //第 2 组试炼码
005F19C7    68 6C1B5F00     PUSH    SuperUti.005F1B6C              ; //"-"
005F19CC    8D55 EC         LEA     EDX, DWORD PTR SS:[EBP-14]
005F19CF    8B03            MOV     EAX, DWORD PTR DS:[EBX]
005F19D1    8B80 20030000   MOV     EAX, DWORD PTR DS:[EAX+320]
005F19D7    E8 6CF2EEFF     CALL    SuperUti.004E0C48
005F19DC    FF75 EC         PUSH    DWORD PTR SS:[EBP-14]          ; //第 3 组试炼码
005F19DF    68 6C1B5F00     PUSH    SuperUti.005F1B6C              ; //"-"
005F19E4    8D55 E8         LEA     EDX, DWORD PTR SS:[EBP-18]
005F19E7    8B03            MOV     EAX, DWORD PTR DS:[EBX]
005F19E9    8B80 24030000   MOV     EAX, DWORD PTR DS:[EAX+324]
005F19EF    E8 54F2EEFF     CALL    SuperUti.004E0C48
005F19F4    FF75 E8         PUSH    DWORD PTR SS:[EBP-18]          ; //第 4 组试炼码
005F19F7    8D45 FC         LEA     EAX, DWORD PTR SS:[EBP-4]
005F19FA    BA 07000000     MOV     EDX, 7
005F19FF    E8 0838E1FF     CALL    SuperUti.0040520C             ; //7 组字符串相连,设为字符串 A
005F1A04    8B03            MOV     EAX, DWORD PTR DS:[EBX]
005F1A06    8B90 34030000   MOV     EDX, DWORD PTR DS:[EAX+334]     ; //用户名
005F1A0C    A1 40147000     MOV     EAX, DWORD PTR DS:[701440]
005F1A11    8B00            MOV     EAX, DWORD PTR DS:[EAX]
005F1A13    8B4D FC         MOV     ECX, DWORD PTR SS:[EBP-4]        ; //字符串 A
005F1A16    E8 653CFAFF     CALL    SuperUti.00595680             ; //关键 CALL
005F1A1B    84C0            TEST    AL, AL
005F1A1D    74 4B           JE      SHORT SuperUti.005F1A6A
005F1A1F    66:A1 C01A5F00  MOV     AX, WORD PTR DS:[5F1AC0]
005F1A25    50              PUSH    EAX
005F1A26    6A 00           PUSH    0
005F1A28    8D4D E4         LEA     ECX, DWORD PTR SS:[EBP-1C]
005F1A2B    BA 781B5F00     MOV     EDX, SuperUti.005F1B78         ; ASCII "C_FrmAbout_lbThanks"
005F1A30    B8 941B5F00     MOV     EAX, SuperUti.005F1B94         ; ASCII "We sincerely appreciate your support of SuperLogix. Now you become a legal user
of Super Utilities."
```

跟进关键 CALL。

代码：
