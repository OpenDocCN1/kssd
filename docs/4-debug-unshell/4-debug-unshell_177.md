

# ARM Protector 0.1

> 原文：[`www.pediy.com/kssd/pediy06/pediy7005.htm`](https://www.pediy.com/kssd/pediy06/pediy7005.htm)

【目标来源】 crakedes.de  
【下载地址】 点击下载  
  
Creator: nivel  
  
Objective: Unpack and stop the time trial. Make the program display always  
that it has 5 trials left.  
  
e-mail: nivel\_999@yahoo.com  
  
Hint: The program has anti-sice,anti-monitoring and other protections.  
      When you unpack it this protections will be removed.  
【破解目标】：作者说得，脱掉它，在 qj 她！  
【破解作者】：mejy【人称 IPB 脱盲】  
【作者声明】：练手的咚咚，不对之处敬请谅解！  
【简要介绍】：该壳虽然不是很难，在当今壳世界中只能算是小儿科，但是用了很多变形 call 和异常，  
还有 SMC 技术，监测 CC，检测调试器等技术。输入表的处理又比较弱，菜鸟研究起来也别有一番风味呀！  
【脱壳过程】：  
（1）查壳：拿出 peid－监测结果 ARM Protector 0.1 -> SMoKE 没见过的说不知是啥咚咚，fly 曰：exethield 的初级版  
（2）掏出脱衣利器:Ollydbg1.1  
（3）载入：程序停在  

> ```
> 代码:
> 00419000 > E8 04000000            CALL crackme4.00419009         //程序停在这里 入口点地址在壳段里
> 00419005   8360 EB 0C             AND DWORD PTR DS:\[EAX\-15\],0C
> 00419009   5D                     POP EBP
> 
> ```
> 
>   

  
怎么样，一件一件的脱啦：）  
切记切记不能心急，小心的用 F8，多用 F7 慢慢抚摸是没错的，程序有很多假的 call，一不小心你就跟飞了！  
俺飞了 n 次  
小心的 F7 来到下面  

> ```
> 代码:
> 004190A7   47                     INC EDI
> 004190A8   4A                     DEC EDX
> 004190A9  ⁷⁵ B4                  JNZ SHORT crackme4.0041905F  这段代码是解码
> 004190AB   EB 01                  JMP SHORT crackme4.004190AE 光标停在这里 F7
> 
> ```
> 
>   

  
004190C1   E8 7E0D0000            CALL crackme4.00419E44       这个一定要用 F7 走  
进入后发现下面的作用是检测 armp 区段，也就是壳区段是否有 int3 断点 ,后面多次用到  

> ```
> 代码:
> 00419E61   EB 01                  JMP SHORT crackme4.00419E64
> 00419E64   8A06                   MOV AL,BYTE PTR DS:\[ESI\]    取要检测的地址
> 00419E66   EB 02                  JMP SHORT crackme4.00419E6A 
> 00419E6A   FEC8                   DEC AL                      程序是为了避免自身出现 CC 所以先减 1
> 00419E6C   EB 01                  JMP SHORT crackme4.00419E6F
> 
> 00419E6F   3C CB                  CMP AL,0CB                这样比较
> 00419E71   EB 01                  JMP SHORT crackme4.00419E74
> 00419E74   75 0B                  JNZ SHORT crackme4.00419E81    跳到这里 如果相等你就玩玩了
> 00419E76   83C4 10                ADD ESP,10
> 00419E79   EB 01                  JMP SHORT crackme4.00419E7C
> 00419E7B   EB 61                  JMP SHORT crackme4.00419EDE
> 00419E7D   EB 01                  JMP SHORT crackme4.00419E80
> 00419E7F   C3                     RETN
> 00419E80   C3                     RETN
> 00419E81   EB 01                  JMP SHORT crackme4.00419E84
> 00419E84   46                     INC ESI                                         ; crackme4.00419002
> 00419E85   EB 01                  JMP SHORT crackme4.00419E88     指令会变化
> 00419E88  ^E2 D7                  LOOPD SHORT crackme4.00419E61   循环
> 00419E8A   58                     POP EAX
> 00419E8B   59                     POP ECX
> 00419E8C   EB 01                  JMP SHORT crackme4.00419E8F     如果比较完毕
> 00419E8E   60                     PUSHAD
> 00419E8F   5E                     POP ESI
> 00419E90   C3                     RETN        返回，注意不能在这里直接用 F2 设断，因为同样这里也有 CC 监测
>                                                所以在此处设内存断点，会中断两次，一次监测 cc 一次真正的执行指令
> 
> 004190C6   EB 01                  JMP SHORT crackme4.004190C9  上面返回到这里
> 
> 004190C6   EB 01                  JMP SHORT crackme4.004190C9
> 
> 004190D6   74 5D                  JE SHORT crackme4.00419135    必须跳走
> 004190D8   EB 01                  JMP SHORT crackme4.004190DB  否则跳到下面
> 
> 7C816D4F   50                     PUSH EAX      来到这里你也玩了
> 7C816D50   E8 545FFFFF            CALL kernel32.ExitThread
> 7C816D55   90                     NOP
> 7C816D56   90                     NOP
> 7C816D57   90                     NOP
> 
> 00419D03  ⁷⁵ DD                  JNZ SHORT crackme4.00419CE2
> 00419D05   EB 01                  JMP SHORT crackme4.00419D08   光标放在这里 F4
> 
> 00419D22   813C02 50450000        CMP DWORD PTR DS:\[EDX+EAX\],4550   PE 文件
> 00419D29  ⁷⁵ E4                  JNZ SHORT crackme4.00419D0F
> 
> 
> 
> 00419D2E   8985 13304000          MOV DWORD PTR SS:\[EBP+403013\],EAX   EAX＝7C800000
> 00419D34   C3                     RETN
> 走到这里会再次检测 CC  在 00419E90 处设内存断点，两次 F9 之后 F8 后面类似处理相同
> 00419EE7   E8 A6F4FFFF            CALL crackme4.00419392  这个里面对系统函数有一个处理
> 00419EEC   66:8B13                MOV DX,WORD PTR DS:\[EBX\]
> 00419EEF   0FB7D2                 MOVZX EDX,DX
> 00419EF2   EB 01                  JMP SHORT crackme4.00419EF5
> 
> 00419F0E   83C3 02                ADD EBX,2
> 00419F11   833C24 00              CMP DWORD PTR SS:\[ESP\],0
> 00419F15  ⁷⁷ C1                  JA SHORT crackme4.00419ED8
> 
> 
> 00419FD5   EB 01                  JMP SHORT crackme4.00419FD8         下面代码 计算壳所用的 api
> 00419FD8   8B85 13304000          MOV EAX,DWORD PTR SS:\[EBP+403013\]      ; kernel32.7C800000 很明显 kernel32 的地址
> 00419FDE   FF36                   PUSH DWORD PTR DS:\[ESI\]
> 00419FE0   50                     PUSH EAX
> 00419FE1   EB 01                  JMP SHORT crackme4.00419FE4
> 00419FE4   EB 01                  JMP SHORT crackme4.00419FE7
> 00419FE7   F785 17304000 40000000 TEST DWORD PTR SS:\[EBP+403017\],40
> 00419FF1   EB 01                  JMP SHORT crackme4.00419FF4
> 
> 00419FF4   74 09                  JE SHORT crackme4.00419FFF
> 00419FF6   EB 02                  JMP SHORT crackme4.00419FFA
> 00419FF8   E8 00E845FE            CALL FE8787FD
> 00419FFD   FFFF                   ???                                             ; 未知命令
> 
> 00419FFA   E8 45FEFFFF            CALL crackme4.00419E44     F7 走 中间会监测该 API 是否有断点
> 00419FFF   EB 01                  JMP SHORT crackme4.0041A002
> 
> 0041A002   E8 8AFEFFFF            CALL crackme4.00419E91        取 API API 地址保存在\[41A103\]
> 0041A007   EB 01                  JMP SHORT crackme4.0041A00A
> 中间会监测该 API 是否有断点
> 
> 0041A007   EB 01                  JMP SHORT crackme4.0041A00A
> 
> 
> 0041A024   8946 04                MOV DWORD PTR DS:\[ESI+4\],EAX                    ; kernel32.GetModuleHandleA 提取壳要用的 API，
> 0041A027   83C6 08                ADD ESI,8
> 0041A02A   EB 01                  JMP SHORT crackme4.0041A02D
> 0041A02D   FF8D D2304000          DEC DWORD PTR SS:\[EBP+4030D2\]          内存中的值为 0xB，即要提取的 API 数量
> 0041A033   83BD D2304000 00       CMP DWORD PTR SS:\[EBP+4030D2\],0        比较 11 个壳所用的 API 是否处理完毕
> 0041A03A  ⁷⁷ 99                  JA SHORT crackme4.00419FD5
> 
> 00419145   8D85 24314000          LEA EAX,DWORD PTR SS:\[EBP+403124\]  API 地址 这时内存中可以看见
> 0041914B   E8 04000000            CALL crackme4.00419154
> 中间会监测 CC
> 004191AF   EB 01                  JMP SHORT crackme4.004191B2
> 004191B2   8DB5 DA304000          LEA ESI,DWORD PTR SS:\[EBP+4030DA\] 地址=0041A196, (ASCII "\\\\.\\TRW")
> 004191B8   EB 02                  JMP SHORT crackme4.004191BC
> 内存中
> 【
> 0041A18F  00 00 00 00 00 00 00 5C 5C 2E 5C 54 52 57 00 5C  .......\\\\.\\TRW.\\
> 0041A19F  5C 2E 5C 53 49 43 45 00 5C 5C 2E 5C 4E 54 49 43  \\.\\SICE.\\\\.\\NTIC
> 0041A1AF  45 00 5C 5C 2E 5C 46 49 4C 45 56 58 44 00 5C 5C  E.\\\\.\\FILEVXD.\\\\
> 0041A1BF  2E 5C 46 49 4C 45 4D 4F 4E 00 5C 5C 2E 5C 52 45  .\\FILEMON.\\\\.\\RE
> 0041A1CF  47 56 58 44 00 5C 5C 2E 5C 52 45 47 4D 4F 4E 00  GVXD.\\\\.\\REGMON.
>  】
> 004191C7   803E 00                CMP BYTE PTR DS:\[ESI\],0
> 004191CA  ⁷⁵ F7                  JNZ SHORT crackme4.004191C3  
> 004191D0   803E 00                CMP BYTE PTR DS:\[ESI\],0
> 004191D3   EB 03                  JMP SHORT crackme4.004191D8
> 004191D8   75 05                  JNZ SHORT crackme4.004191DF  这里可以跳过检测调试器 
> 004191DA   EB 01                  JMP SHORT crackme4.004191DD
> 
> 
> 
> 004191E8   FF95 87304000          CALL DWORD PTR SS:\[EBP+403087\]                  ; kernel32.\_lopen
> 004191EE   40                     INC EAX               检测调试器了
> 004191EF   EB 02                  JMP SHORT crackme4.004191F3
> 004191F3  ⁷⁴ C9                  JE SHORT crackme4.004191BE  一定要跳
> 004191F5   61                     POPAD
> 004191F6   C3                     RETN
> 
> 004193FC   0F84 D8000000          JE crackme4.004194DA   小心这里，一定要跳
> 00419402   EB 01                  JMP SHORT crackme4.00419405
> 
> 
> 00419516   8BBD 1B304000          MOV EDI,DWORD PTR SS:\[EBP+40301B\]               ; crackme4.00400000
> 0041951C   037F 3C                ADD EDI,DWORD PTR DS:\[EDI+3C\]                  3C 处是 PE 格式标志
> 
> 00419522   8BF7                   MOV ESI,EDI                                区段信息
> 00419524   81C6 F8000000          ADD ESI,0F8
> 0041952A   EB 01                  JMP SHORT crackme4.0041952D
> 
> 004196EB   837E 14 00             CMP DWORD PTR DS:\[ESI+14\],0
> 004196EF   74 06                  JE SHORT crackme4.004196F7
> 004196F1   837E 10 00             CMP DWORD PTR DS:\[ESI+10\],0
> 004196F5   75 05                  JNZ SHORT crackme4.004196FC
> 004196F7   E9 A3000000            JMP crackme4.0041979F
> 004196FC   8B46 0C                MOV EAX,DWORD PTR DS:\[ESI+C\]
> 004196FF   EB 01                  JMP SHORT crackme4.00419702
> 
> 0041978F   E8 B0060000            CALL crackme4.00419E44
> 00419794   EB 02                  JMP SHORT crackme4.00419798
> 
> 00419798   E8 C0070000            CALL crackme4.00419F5D
> 
> 中间是区段解密
> 
> 
> 004197A6   3BD3                   CMP EDX,EBX       比较已经处理的区段数目和程序区段总数目
> 004197A8  ⁰F82 3DFFFFFF          JB crackme4.004196EB
> 004197AE   EB 01                  JMP SHORT crackme4.004197B1
> 
> 004197B1   66:C747 06 0000        MOV WORD PTR DS:\[EDI+6\],0
> 004197B7   EB 01                  JMP SHORT crackme4.004197BA
> 下面开始处理输入表了
> 004198B9   57                     PUSH EDI   userdll 等的地址
> 004198BA   EB 02                  JMP SHORT crackme4.004198BE
> 004198C8   8B85 7F304000          MOV EAX,DWORD PTR SS:\[EBP+40307F\]               ; kernel32.LoadLibraryA
> 004198CE   EB 01                  JMP SHORT crackme4.004198D1
> 
> 004198EF   FFD0                   CALL EAX                                    获取 API 的地址
> 004198F1   0BC0                   OR EAX,EAX                                      ; user32.77D10000
> 
> 00419984   8B56 10                MOV EDX,DWORD PTR DS:\[ESI+10\]
> 00419987   EB 01                  JMP SHORT crackme4.0041998A
> 004199B0   0395 1B304000          ADD EDX,DWORD PTR SS:\[EBP+40301B\]               ; crackme4.00400000
> 004199B6   EB 01                  JMP SHORT crackme4.004199B9       
> 
> 
> 00419A8E   8902                   MOV DWORD PTR DS:\[EDX\],EAX    写入 IAT
> 00419A90   EB 01                  JMP SHORT crackme4.00419A93
>  其中 EDX 中保存着 IAT 的地址，因为从 12000 开始写起的，
> 00419B20   3BCA                   CMP ECX,EDX    是否处理完毕  第二次 ecx 的值就是 IAT 起始
> 00419B22   74 18                  JE SHORT crackme4.00419B3C
> 
> 凌晨 4 点了，IAT 表的计算略处 到这里自己跟一下就看见了
> 继续走
> 00419B31   6A 01                  PUSH 1
> 00419B33   51                     PUSH ECX
> 00419B34   EB 01                  JMP SHORT crackme4.00419B37
> 00419B42   83C2 04                ADD EDX,4
> 00419B45   8339 00                CMP DWORD PTR DS:\[ECX\],0
> 00419B48  ⁰F85 7AFEFFFF          JNZ crackme4.004199C8
> 00419B4E   EB 01                  JMP SHORT crackme4.00419B51
> 
> 00419B6C   83C6 14                ADD ESI,14
> 00419B6F   33C0                   XOR EAX,EAX
> 00419B71   837E 10 00             CMP DWORD PTR DS:\[ESI+10\],0
> 00419B75  ⁰F85 C5FCFFFF          JNZ crackme4.00419840
> 00419B7B   EB 01                  JMP SHORT crackme4.00419B7E
> 
> 00419F50  ^E2 F7                  LOOPD SHORT crackme4.00419F49
> 00419F52   8D85 30314000          LEA EAX,DWORD PTR SS:\[EBP+403130\] F4
> 
> 
> 
> 0041A226  ^E2 F7                  LOOPD SHORT crackme4.0041A21F
> 0041A228   61                     POPAD   F4 到这
> 0041A22C   50                     PUSH EAX                                        ; crackme4.00419B86
> 下面有一个 0 地址异常，EAx 是异常的结束点，在 EAX 处 F2 设断，然后 SHIFT+F9
> 0041A22D   EB 01                  JMP SHORT crackme4.0041A230
> 0041A22F   EB 64                  JMP SHORT crackme4.0041A295
> 
> 来到
> 00419B86   EB 01                  JMP SHORT crackme4.00419B89  取消这里的断点
> 
> 来到
> 00419BC1   8986 B8000000          MOV DWORD PTR DS:\[ESI+B8\],EAX                   ; crackme4.00407C2E 
> EAX 中就是 OEP，呵呵用异常这样处理，我们在 OEP 处 F2，然后 SHIFT+F9 就道 OEP 了
> 00419BC7   B8 00000000            MOV EAX,0
> 00419BCC   EB 01                  JMP SHORT crackme4.00419BCF
> 
> ```
> 
>   

  
到这里完毕，打开 LordPe 修复一下，OEP＝7C2E 就可以用了，到达 OEP 的方法有点好玩。  
  
  
【脱壳方法总结】  
1 载入程序  
2 0041912A   74 09                  JE SHORT crackme4.00419135 设置硬件访问断点，改变这里的跳转使它跳  
  
3 004193FC   0F84 D8000000          JE crackme4.004194DA   设置硬件访问断点 使她跳  
00419A8E   8902                   MOV DWORD PTR DS:\[EDX\],EAX  这里也可以设硬件断点，可以看得 IAT  
DS:\[004120F8\]=17CE3053  
4 00419B86   EB 01                  JMP SHORT crackme4.00419B89 设置硬件断点 SHIFT+F9  
5 00407C2E   55                     PUSH EBP                  设置内存访问断点 SHIFT+F9  
停在这里后，就可以 dump 程序了。  
  
【破解】  
可以反汇编看看，这里直接跟了，猜它注册信息保存在注册表中，不信你可以用 OD 的搜索当前模块函数看看，然后设断  
小讨厌：你不防先运行一次程序，关掉在运行一次，呵呵，出错了吧！是脱壳没脱干净吗？非也，作者在这里放了个苍蝇  

> ```
> 代码:
> 00401300  |. 8D5424 00            LEA EDX,DWORD PTR SS:\[ESP\]
> 00401304  |. 51                   PUSH ECX                                        ; /pDisposition
> 00401305  |. 52                   PUSH EDX                                        ; |pHandle
> 00401306  |. 6A 00                PUSH 0                                          ; |pSecurity = NULL
> 00401308  |. 68 3F000F00          PUSH 0F003F                                     ; |Access = KEY\_ALL\_ACCESS
> 0040130D  |. 6A 00                PUSH 0                                          ; |Options = REG\_OPTION\_NON\_VOLATILE
> 0040130F  |. 6A 00                PUSH 0                                          ; |Class = NULL
> 00401311  |. 6A 00                PUSH 0                                          ; |Reserved = 0
> 00401313  |. 68 34524100          PUSH dumped\_.00415234                           ; |Subkey = "nv"
> 00401318  |. 50                   PUSH EAX                                        ; |hKey
> 00401319  |. FF15 0C204100        CALL DWORD PTR DS:\[<&advapi32.RegCreateKeyExA>\] ; \\RegCreateKeyExA
> 0040131F  |. 8B4424 08            MOV EAX,DWORD PTR SS:\[ESP+8\]
> 00401323  |. 83F8 01              CMP EAX,1
> 00401326     75 1D                JNZ SHORT dumped\_.00401345       呵呵，这里如果注册表中有值的话 这里跳走
> 00401328  |. 8B4C24 00            MOV ECX,DWORD PTR SS:\[ESP\]
> 上面会返回到这里，当然异常退出了
> 00400000   4D                     DEC EBP
> 00400001   5A                     POP EDX
> 00400002   90                     NOP
> 00400003   0003                   ADD BYTE PTR DS:\[EBX\],AL
> 00400005   0000                   ADD BYTE PTR DS:\[EAX\],AL
> 
> 我们把那个苍蝇 NOP 掉，就可以了 用 OD 保存，继续载入新的程序
> 0040172D  |. 51                   PUSH ECX                                        ; /pDisposition
> 0040172E  |. 52                   PUSH EDX                                        ; |pHandle
> 0040172F  |. 6A 00                PUSH 0                                          ; |pSecurity = NULL
> 00401731  |. 68 3F000F00          PUSH 0F003F                                     ; |Access = KEY\_ALL\_ACCESS
> 00401736  |. 6A 00                PUSH 0                                          ; |Options = REG\_OPTION\_NON\_VOLATILE
> 00401738  |. 68 9C524100          PUSH dumped1\_.0041529C                          ; |Class = "REG\_SZ"
> 0040173D  |. 6A 00                PUSH 0                                          ; |Reserved = 0
> 0040173F  |. 68 98524100          PUSH dumped1\_.00415298                          ; |Subkey = "niv"
> 00401744  |. 50                   PUSH EAX                                        ; |hKey
> 00401745  |. FF15 0C204100        CALL DWORD PTR DS:\[<&advapi32.RegCreateKeyExA>\] ; \\RegCreateKeyExA
> 0040174B  |. 8B4424 0C            MOV EAX,DWORD PTR SS:\[ESP+C\]
> 0040174F  |. 5F                   POP EDI
> 00401750  |. 83F8 01              CMP EAX,1
> 00401753     75 23                JNZ SHORT dumped1\_.00401778    这里比较是否已经存在该建值，有的话会进行操作
> 没有会给他赋个值，呵呵，既然这样我们就每次都给他赋新值就可以了吗 我们把它也 NOP 掉
> 00401755  |. 8B4C24 00            MOV ECX,DWORD PTR SS:\[ESP\]
> 00401759  |. 6A 03                PUSH 3                                          ; /Length = 3
> 0040175B  |. 68 94524100          PUSH dumped1\_.00415294                          ; |Value = "535"
> 00401760  |. 50                   PUSH EAX                                        ; |ValueType
> 00401761  |. 6A 00                PUSH 0                                          ; |Subkey = NULL
> 00401763  |. 51                   PUSH ECX                                        ; |hKey
> 
> 004014C7  |. 75 1B                JNZ SHORT dumped1\_.004014E4                   比较是否过期
> 004014C9  |. 6A 00                PUSH 0                                          ; /Style = MB\_OK|MB\_APPLMODAL
> 004014CB  |. 68 64524100          PUSH dumped1\_.00415264                          ; |Title = "Warning"
> 004014D0  |. 68 4C524100          PUSH dumped1\_.0041524C                          ; |Text = "Your trial has expired"
> 004014D5  |. 6A 00                PUSH 0                                          ; |hOwner = NULL
> 004014D7  |. FF15 00214100        CALL DWORD PTR DS:\[<&user32.MessageBoxA>\]       ; \\MessageBoxA
> 004014DD  |. 6A 01                PUSH 1
> 004014DF  |. E8 AB5B0000          CALL dumped1\_.0040708F
> 004014E4  \\> C3                   RETN
> 还有很多地方可以爆破这个程序
> 
> ```
> 
>   

  
  
【后记】好累呀！脱壳好累！  
        本 unpack 我等菜鸟练手实在不错  
        感谢 ipb 群一起灌水的大大们，陪我度过的美好时光，呵呵！

 4dd466d0e1e0cc123afbe504da2462ca