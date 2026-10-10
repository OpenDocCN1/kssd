

# Fengyue's DLL-Game.exe 加壳流程简单分析

> 原文：[`www.pediy.com/kssd/pediy10/89706.html`](https://www.pediy.com/kssd/pediy10/89706.html)

作者：Nisy/PYG  
声明：YPFX  
  
随便找一个文件加壳处理 BP CreateFileA 断在这里：  
  
004017C5  |.  6A 00         PUSH 0                                   ; /hTemplateFile = NULL  
004017C7  |.  68 80000000   PUSH 80                                  ; |Attributes = NORMAL  
004017CC  |.  6A 03         PUSH 3                                   ; |Mode = OPEN\_EXISTING  
004017CE  |.  6A 00         PUSH 0                                   ; |pSecurity = NULL  
004017D0  |.  6A 03         PUSH 3                                   ; |ShareMode = FILE\_SHARE\_READ|FILE\_SHARE\_WRITE  
004017D2  |.  68 00000080   PUSH 80000000                            ; |Access = GENERIC\_READ  
004017D7  |.  68 E8DA4000   PUSH Fengyue'.0040DAE8                   ; |FileName = "C:\\Documents and Settings\\Nisy\\桌面\\灰钮.exe"  
004017DC  |.  FFD3          CALL EBX                                 ; \\CreateFileA  
004017DE  |.  8BF0          MOV ESI,EAX                              ;  打开文件  
  
00401842  |> \\57            PUSH EDI                                 ;  获取文件大小  
00401843  |.  6A 00         PUSH 0                                   ; /pFileSizeHigh = NULL  
00401845  |.  56            PUSH ESI                                 ; |hFile  
00401846  |.  FF15 08A04000 CALL DWORD PTR DS:\[<&KERNEL32.GetFileSiz>; \\GetFileSize  
0040184C  |.  6A 04         PUSH 4                                   ; /Protect = PAGE\_READWRITE  
0040184E  |.  68 00100000   PUSH 1000                                ; |AllocationType = MEM\_COMMIT  
00401853  |.  50            PUSH EAX                                 ; |Size  
00401854  |.  6A 00         PUSH 0                                   ; |Address = NULL  
00401856  |.  A3 30DC4000   MOV DWORD PTR DS:\[40DC30\],EAX            ; |  
0040185B  |.  FF15 28A04000 CALL DWORD PTR DS:\[<&KERNEL32.VirtualAll>; \\VirtualAlloc  
00401861  |.  8B15 30DC4000 MOV EDX,DWORD PTR DS:\[40DC30\]            ;  申请 PE 文件大小的空间  
00401867  |.  6A 00         PUSH 0                                   ; /pOverlapped = NULL  
00401869  |.  8D4C24 10     LEA ECX,DWORD PTR SS:\[ESP+10\]            ; |  
0040186D  |.  51            PUSH ECX                                 ; |pBytesRead  
0040186E  |.  52            PUSH EDX                                 ; |BytesToRead => 13600 (79360.)  
0040186F  |.  50            PUSH EAX                                 ; |Buffer  
00401870  |.  56            PUSH ESI                                 ; |hFile  
00401871  |.  A3 38DC4000   MOV DWORD PTR DS:\[40DC38\],EAX            ; |将文件读取到申请的空间中 地址 01020000  
00401876  |.  FF15 24A04000 CALL DWORD PTR DS:\[<&KERNEL32.ReadFile>\] ; \\ReadFile  
0040187C  |.  8B3D 34A04000 MOV EDI,DWORD PTR DS:\[<&KERNEL32.CloseHa>;  kernel32.CloseHandle  
00401882  |.  56            PUSH ESI                                 ; /hObject  
00401883  |.  FFD7          CALL EDI                                 ; \\CloseHandle  
00401885  |.  8B0D 38DC4000 MOV ECX,DWORD PTR DS:\[40DC38\]            ;  关闭文件句柄  
  
  
  
00401842  |> \\57            PUSH EDI                                 ;  获取文件大小  
00401843  |.  6A 00         PUSH 0                                   ; /pFileSizeHigh = NULL  
00401845  |.  56            PUSH ESI                                 ; |hFile  
00401846  |.  FF15 08A04000 CALL DWORD PTR DS:\[<&KERNEL32.GetFileSiz>; \\GetFileSize  
0040184C  |.  6A 04         PUSH 4                                   ; /Protect = PAGE\_READWRITE  
0040184E  |.  68 00100000   PUSH 1000                                ; |AllocationType = MEM\_COMMIT  
00401853  |.  50            PUSH EAX                                 ; |Size  
00401854  |.  6A 00         PUSH 0                                   ; |Address = NULL  
00401856  |.  A3 30DC4000   MOV DWORD PTR DS:\[40DC30\],EAX            ; |  
0040185B  |.  FF15 28A04000 CALL DWORD PTR DS:\[<&KERNEL32.VirtualAll>; \\VirtualAlloc  
00401861  |.  8B15 30DC4000 MOV EDX,DWORD PTR DS:\[40DC30\]            ;  申请 PE 文件大小的空间  
00401867  |.  6A 00         PUSH 0                                   ; /pOverlapped = NULL  
00401869  |.  8D4C24 10     LEA ECX,DWORD PTR SS:\[ESP+10\]            ; |  
0040186D  |.  51            PUSH ECX                                 ; |pBytesRead  
0040186E  |.  52            PUSH EDX                                 ; |BytesToRead => 13600 (79360.)  
0040186F  |.  50            PUSH EAX                                 ; |Buffer  
00401870  |.  56            PUSH ESI                                 ; |hFile  
00401871  |.  A3 38DC4000   MOV DWORD PTR DS:\[40DC38\],EAX            ; |将文件读取到申请的空间中 地址 01020000  
00401876  |.  FF15 24A04000 CALL DWORD PTR DS:\[<&KERNEL32.ReadFile>\] ; \\ReadFile  
0040187C  |.  8B3D 34A04000 MOV EDI,DWORD PTR DS:\[<&KERNEL32.CloseHa>;  kernel32.CloseHandle  
00401882  |.  56            PUSH ESI                                 ; /hObject  
00401883  |.  FFD7          CALL EDI                                 ; \\CloseHandle  
00401885  |.  8B0D 38DC4000 MOV ECX,DWORD PTR DS:\[40DC38\]            ;  关闭文件句柄  
0040188B  |.  8B41 3C       MOV EAX,DWORD PTR DS:\[ECX+3C\]            ;  将 DOS 头部存放 PE 标识符数据的地址送 EAX  
0040188E  |.  0FB75408 14   MOVZX EDX,WORD PTR DS:\[EAX+ECX+14\]       ;  取 PE 头 IMAGE\_OPTIONAL\_HEADER32 的长度 = E0  
00401893  |.  03C1          ADD EAX,ECX                              ;  求出 PE 头的偏移值  
00401895  |.  66:8139 4D5A  CMP WORD PTR DS:\[ECX\],5A4D               ;  查看一下 PE 文件前两个字节是否为 MZ  
0040189A  |.  8D5402 18     LEA EDX,DWORD PTR DS:\[EDX+EAX+18\]        ;  求出 PE 头节表的偏移值 装入 EDX  
0040189E  |.  890D 20DC4000 MOV DWORD PTR DS:\[40DC20\],ECX            ;  申请空间的地址=PE 的地址  
004018A4  |.  A3 F8DB4000   MOV DWORD PTR DS:\[40DBF8\],EAX            ;  "PE"的地址  
004018A9  |.  8915 18DC4000 MOV DWORD PTR DS:\[40DC18\],EDX            ;  "text"节表的地址  
004018AF  |.  75 18         JNZ SHORT Fengyue'.004018C9              ;  若不是 PE 则跳  
004018B1  |.  8138 50450000 CMP DWORD PTR DS:\[EAX\],4550              ;  检查是否为 PE  
004018B7  |.  75 10         JNZ SHORT Fengyue'.004018C9  
004018B9  |.  66:8178 04 4C>CMP WORD PTR DS:\[EAX+4\],14C              ;  PE 运行平台是否为 386  
004018BF  |.  75 08         JNZ SHORT Fengyue'.004018C9  
004018C1  |.  66:8178 18 0B>CMP WORD PTR DS:\[EAX+18\],10B             ;  10B 为 EXE 格式  
004018C7  |.  74 4B         JE SHORT Fengyue'.00401914               ;  上方为检测是否为 PE 文件 OK 则跳走  
004018C7  |. /74 4B         JE SHORT Fengyue'.00401914               ;  上方为检测是否为 PE 文件  PE 则跳走  
004018C9  |> |E8 12FEFFFF   CALL Fengyue'.004016E0  
004018CE  |. |A1 A8B34000   MOV EAX,DWORD PTR DS:\[40B3A8\]  
004018D3  |. |8B0D ACB34000 MOV ECX,DWORD PTR DS:\[40B3AC\]  
004018D9  |. |8B15 B0B34000 MOV EDX,DWORD PTR DS:\[40B3B0\]  
004018DF  |. |A3 50CF4000   MOV DWORD PTR DS:\[40CF50\],EAX  
004018E4  |. |A1 B4B34000   MOV EAX,DWORD PTR DS:\[40B3B4\]  
004018E9  |. |5F            POP EDI  
004018EA  |. |5E            POP ESI  
004018EB  |. |A3 5CCF4000   MOV DWORD PTR DS:\[40CF5C\],EAX  
004018F0  |. |890D 54CF4000 MOV DWORD PTR DS:\[40CF54\],ECX  
004018F6  |. |8915 58CF4000 MOV DWORD PTR DS:\[40CF58\],EDX  
004018FC  |. |33C0          XOR EAX,EAX  
004018FE  |. |5B            POP EBX  
004018FF  |. |8B8C24 080100>MOV ECX,DWORD PTR SS:\[ESP+108\]  
00401906  |. |33CC          XOR ECX,ESP  
00401908  |. |E8 F3020000   CALL Fengyue'.00401C00  
0040190D  |. |81C4 0C010000 ADD ESP,10C  
00401913  |. |C3            RETN  
00401914  |> \\E8 E7F6FFFF   CALL Fengyue'.00401000                         ;  Jmp Here // 循环处理各区段的 VirtualSize 为 0X1000 的整数倍  
00401919  |.  E8 72F7FFFF   CALL Fengyue'.00401090                         ;  把磁盘 PE 文件 映射到 内存中  
0040191E  |.  E8 5DF8FFFF   CALL Fengyue'.00401180                         ;  把壳的代码段资源释放到一个空间  
00401923  |.  E8 48FAFFFF   CALL Fengyue'.00401370                         ;  这个函数 为 PE 添加新区段并将内存中的 PE 文件处理为磁盘 PE 文件格式  
00401928  |.  33C0          XOR EAX,EAX  
0040192A  |.  8D9B 00000000 LEA EBX,DWORD PTR DS:\[EBX\]  
00401930  |>  8A88 E8DA4000 /MOV CL,BYTE PTR DS:\[EAX+40DAE8\]               ;  取一下原文件名  
00401936  |.  884C04 10     |MOV BYTE PTR SS:\[ESP+EAX+10\],CL  
0040193A  |.  83C0 01       |ADD EAX,1  
0040193D  |.  84C9          |TEST CL,CL  
0040193F  |.^ 75 EF         \\JNZ SHORT Fengyue'.00401930  
00401941  |.  8D4424 10     LEA EAX,DWORD PTR SS:\[ESP+10\]  
00401945  |.  83C0 FF       ADD EAX,-1  
00401948  |>  8A48 01       /MOV CL,BYTE PTR DS:\[EAX+1\]                    ;  .bak  
0040194B  |.  83C0 01       |ADD EAX,1  
0040194E  |.  84C9          |TEST CL,CL  
00401950  |.^ 75 F6         \\JNZ SHORT Fengyue'.00401948  
00401952  |.  8B0D B8B34000 MOV ECX,DWORD PTR DS:\[40B3B8\]  
00401958  |.  8A15 BCB34000 MOV DL,BYTE PTR DS:\[40B3BC\]  
0040195E  |.  8908          MOV DWORD PTR DS:\[EAX\],ECX  
00401960  |.  8850 04       MOV BYTE PTR DS:\[EAX+4\],DL                     ;  备份一下文件  
00401963  |.  6A 00         PUSH 0                                         ; /FailIfExists = FALSE  
00401965  |.  8D4424 14     LEA EAX,DWORD PTR SS:\[ESP+14\]                  ; |  
00401969  |.  50            PUSH EAX                                       ; |NewFileName  
0040196A  |.  68 E8DA4000   PUSH Fengyue'.0040DAE8                         ; |ExistingFileName = "C:\\Documents and Settings\\Nisy\\桌面\\Test.exe"  
0040196F  |.  FF15 2CA04000 CALL DWORD PTR DS:\[<&KERNEL32.CopyFileA>\]      ; \\CopyFileA  
00401975  |.  6A 00         PUSH 0  
00401977  |.  68 80000000   PUSH 80  
0040197C  |.  6A 02         PUSH 2  
0040197E  |.  6A 00         PUSH 0  
00401980  |.  6A 02         PUSH 2  
00401982  |.  68 00000040   PUSH 40000000  
00401987  |.  68 E8DA4000   PUSH Fengyue'.0040DAE8  
0040198C  |.  FFD3          CALL EBX                                       ;  (kernel32.CreateFileA) 用原文件名创建一个文件  
0040198E  |.  8B15 28DC4000 MOV EDX,DWORD PTR DS:\[40DC28\]  
00401994  |.  6A 00         PUSH 0                                         ; /pOverlapped = NULL  
00401996  |.  8D4C24 10     LEA ECX,DWORD PTR SS:\[ESP+10\]                  ; |  
0040199A  |.  51            PUSH ECX                                       ; |pBytesWritten  
0040199B  |.  8BF0          MOV ESI,EAX                                    ; |  
0040199D  |.  A1 00DC4000   MOV EAX,DWORD PTR DS:\[40DC00\]                  ; |  
004019A2  |.  52            PUSH EDX                                       ; |nBytesToWrite => 6E00 (28160.)  
004019A3  |.  50            PUSH EAX                                       ; |Buffer => 010C0000  
004019A4  |.  56            PUSH ESI                                       ; |hFile  
004019A5  |.  FF15 1CA04000 CALL DWORD PTR DS:\[<&KERNEL32.WriteFile>\]      ; \\保存一下弄好的文件  
004019AB  |.  56            PUSH ESI  
004019AC  |.  FFD7          CALL EDI                                       ;  关闭文件句柄  
004019AE  |.  68 E8DA4000   PUSH Fengyue'.0040DAE8  
004019B3  |.  68 C0B34000   PUSH Fengyue'.0040B3C0                         ;  %s Save Success!  
004019B8  |.  68 50CF4000   PUSH Fengyue'.0040CF50                         ;  123123  
  
  
\--------------------------------------------  
  
这四个函数是处理 PE 文件的核心 我们关注一下：  
  
00401914  |> \\E8 E7F6FFFF   CALL Fengyue'.00401000                         ;  Jmp Here // 循环处理各区段的 VirtualSize 为 0X1000 的整数倍  
00401919  |.  E8 72F7FFFF   CALL Fengyue'.00401090                         ;  把磁盘 PE 文件 映射到 内存中  
0040191E  |.  E8 5DF8FFFF   CALL Fengyue'.00401180                         ;  把壳的非 PE 头段资源释放到一个空间并对代码段数据进行第一次的重定位调整  
00401923  |.  E8 48FAFFFF   CALL Fengyue'.00401370                         ;  这个函数 为 PE 添加新区段并将内存中的 PE 文件处理为磁盘 PE 文件格式  
  
\--------------------------------------------  
  
\>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  
  
第一个函数：CALL Fengyue'.00401000 : 循环处理各区段的 VirtualSize  
  
00401000  /$  A1 F8DB4000   MOV EAX,DWORD PTR DS:\[40DBF8\]            ;  EAX = F100C0 磁盘文件 PE 标识地址  
00401005  |.  8B48 50       MOV ECX,DWORD PTR DS:\[EAX+50\]            ;  EAX+50 = SizeOfImage 整个内存镜像的大小  
00401008  |.  0FB750 14     MOVZX EDX,WORD PTR DS:\[EAX+14\]           ;  IMAGE\_OPTIONAL\_HEADER32 结构的长度 = E0  
0040100C  |.  890D E4DA4000 MOV DWORD PTR DS:\[40DAE4\],ECX            ;  Size = 9000  
00401012  |.  0FB748 06     MOVZX ECX,WORD PTR DS:\[EAX+6\]            ;  PE 的区段数放 ECX  
00401016  |.  53            PUSH EBX                                 ;  EBX=CreateFileA  
00401017  |.  8BD9          MOV EBX,ECX  
00401019  |.  8D0C89        LEA ECX,DWORD PTR DS:\[ECX+ECX\*4\]  
0040101C  |.  56            PUSH ESI  
0040101D  |.  8B70 38       MOV ESI,DWORD PTR DS:\[EAX+38\]            ;  内存节 对齐粒度  
00401020  |.  8D14CA        LEA EDX,DWORD PTR DS:\[EDX+ECX\*8\]         ;  EDX = NT\_HEADER 长度 + 区段节长度  
00401023  |.  2B15 38DC4000 SUB EDX,DWORD PTR DS:\[40DC38\]            ;  申请空间地址 == PE 文件地址  
00401029  |.  85DB          TEST EBX,EBX                             ;  检测区段是否为 0  
0040102B  |.  57            PUSH EDI                                 ;  EDI=7C809B77 (kernel32.CloseHandle)  
0040102C  |.  8B78 3C       MOV EDI,DWORD PTR DS:\[EAX+3C\]            ;  BaseOfDate 数据段的 RAV  
0040102F  |.  8D4402 18     LEA EAX,DWORD PTR DS:\[EDX+EAX+18\]        ;  EAX = (PE 偏移)C0 + IMAGE\_FILE\_HEADER + IMAGE\_OPTIONAL\_HEADER32 + 区段表长度  
00401033  |.  8935 44DC4000 MOV DWORD PTR DS:\[40DC44\],ESI            ;  内存节对齐粒度 == 1000  
00401039  |.  893D 10DC4000 MOV DWORD PTR DS:\[40DC10\],EDI            ;  200  
0040103F  |.  891D 34DC4000 MOV DWORD PTR DS:\[40DC34\],EBX            ;  内存节数目  
00401045  |.  A3 D8DA4000   MOV DWORD PTR DS:\[40DAD8\],EAX            ;  258 整个 PE 头的长度  
0040104A  |.  76 33         JBE SHORT Fengyue'.0040107F  
0040104C  |.  8B0D 18DC4000 MOV ECX,DWORD PTR DS:\[40DC18\]            ;  test 节表的地址  
00401052  |.  83C1 08       ADD ECX,8  
00401055  |>  8B51 08       /MOV EDX,DWORD PTR DS:\[ECX+8\]            ;  SizeOfRawData 在文件中的对齐尺寸  
00401058  |.  8D443A FF     |LEA EAX,DWORD PTR DS:\[EDX+EDI-1\]        ;    
0040105C  |.  33D2          |XOR EDX,EDX  
0040105E  |.  F7F7          |DIV EDI  
00401060  |.  33D2          |XOR EDX,EDX  
00401062  |.  83C1 28       |ADD ECX,28  
00401065  |.  0FAFC7        |IMUL EAX,EDI  
00401068  |.  8941 E0       |MOV DWORD PTR DS:\[ECX-20\],EAX           ;  循环处理各区段 SizeOfRawData  
0040106B  |.  8B41 D8       |MOV EAX,DWORD PTR DS:\[ECX-28\]  
0040106E  |.  8D4430 FF     |LEA EAX,DWORD PTR DS:\[EAX+ESI-1\]        ;  ESI=1000  BaseOfCode  
00401072  |.  F7F6          |DIV ESI  
00401074  |.  0FAFC6        |IMUL EAX,ESI  
00401077  |.  83EB 01       |SUB EBX,1  
0040107A  |.  8941 D8       |MOV DWORD PTR DS:\[ECX-28\],EAX           ;  循环的处理各区段 VirtualSize 并以 0x1000 整数倍进行调整  
0040107D  |.^ 75 D6         \\JNZ SHORT Fengyue'.00401055  
0040107F  |>  5F            POP EDI  
00401080  |.  5E            POP ESI  
00401081  |.  5B            POP EBX  
00401082  \\.  C3            RETN  
  
\>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  
  
第二个函数 Call 401090 ： 把磁盘 PE 文件 映射到 内存中  
  
  
00401090  /$  51            PUSH ECX  
00401091  |.  A1 E4DA4000   MOV EAX,DWORD PTR DS:\[40DAE4\]            ;  程序的总大小  
00401096  |.  55            PUSH EBP  
00401097  |.  57            PUSH EDI  
00401098  |.  6A 04         PUSH 4                                   ; /Protect = PAGE\_READWRITE  
0040109A  |.  68 00100000   PUSH 1000                                ; |AllocationType = MEM\_COMMIT  
0040109F  |.  50            PUSH EAX                                 ; |Size => 9000 (36864.)  
004010A0  |.  6A 00         PUSH 0                                   ; |Address = NULL  
004010A2  |.  FF15 28A04000 CALL DWORD PTR DS:\[<&KERNEL32.VirtualAll>; \\ (kernel32.VirtualAlloc)申请新空间 SizeOfImage = 9000  01020000  
004010A8  |.  8B0D D8DA4000 MOV ECX,DWORD PTR DS:\[40DAD8\]            ;  PE 头部结束 // 258  
004010AE  |.  8B2D 38DC4000 MOV EBP,DWORD PTR DS:\[40DC38\]            ;  磁盘文件所属空间地址 00F10000  
004010B4  |.  81C1 FF010000 ADD ECX,1FF  
004010BA  |.  C1E9 09       SHR ECX,9  
004010BD  |.  C1E1 09       SHL ECX,9  
004010C0  |.  51            PUSH ECX                                 ;  400  // 把磁盘文件的 PE 头 Copy 过去  
004010C1  |.  8BF8          MOV EDI,EAX  
004010C3  |.  55            PUSH EBP  
004010C4  |.  57            PUSH EDI  
004010C5  |.  893D ECDB4000 MOV DWORD PTR DS:\[40DBEC\],EDI            ;  new alloc  
004010CB  |.  E8 10570000   CALL Fengyue'.004067E0                   ;  该函数为数据 Copy 函数   
004010D0  |.  8B47 3C       MOV EAX,DWORD PTR DS:\[EDI+3C\]  
004010D3  |.  0FB75438 14   MOVZX EDX,WORD PTR DS:\[EAX+EDI+14\]  
004010D8  |.  8B0D 34DC4000 MOV ECX,DWORD PTR DS:\[40DC34\]            ;  区段数  
004010DE  |.  03C7          ADD EAX,EDI  
004010E0  |.  A3 F8DB4000   MOV DWORD PTR DS:\[40DBF8\],EAX            ;  new alloc PE  
004010E5  |.  83C4 0C       ADD ESP,0C  
004010E8  |.  85C9          TEST ECX,ECX                             ;  把所有区段都 Copy 到新空间  
004010EA  |.  8D4402 18     LEA EAX,DWORD PTR DS:\[EDX+EAX+18\]        ;  区段名装入 EAX  
004010EE  |.  893D 20DC4000 MOV DWORD PTR DS:\[40DC20\],EDI  
004010F4  |.  A3 18DC4000   MOV DWORD PTR DS:\[40DC18\],EAX  
004010F9  |.  76 77         JBE SHORT Fengyue'.00401172  
004010FB  |.  53            PUSH EBX  
004010FC  |.  8B1D 10DC4000 MOV EBX,DWORD PTR DS:\[40DC10\]            ;  文件的对齐粒度  
00401102  |.  56            PUSH ESI  
00401103  |.  8D70 0C       LEA ESI,DWORD PTR DS:\[EAX+C\]             ;  VirtualAddress  ESI 中存放 VA  
00401106  |.  894C24 10     MOV DWORD PTR SS:\[ESP+10\],ECX            ;  保存 区段数   
0040110A  |.  8D9B 00000000 LEA EBX,DWORD PTR DS:\[EBX\]  
00401110  |> /8B4E 08       /MOV ECX,DWORD PTR DS:\[ESI+8\]            ;  PTRD 取该区段在磁盘文件中的偏移  
00401113  |.  33D2          |XOR EDX,EDX  
00401115  |.  8BC1          |MOV EAX,ECX  
00401117  |.  F7F3          |DIV EBX  
00401119  |.  85D2          |TEST EDX,EDX  
0040111B  |.  75 04         |JNZ SHORT Fengyue'.00401121  
0040111D  |.  8BC1          |MOV EAX,ECX  
0040111F  |.  EB 03         |JMP SHORT Fengyue'.00401124  
00401121  |>  0FAFC3        |IMUL EAX,EBX  
00401124  |> \\8B4E 04       |MOV ECX,DWORD PTR DS:\[ESI+4\]            ;  SizeOfRawData 文件的该区段的大小  
00401127  |.  8D1428        |LEA EDX,DWORD PTR DS:\[EAX+EBP\]          ;  new alloc  
0040112A  |.  8B06          |MOV EAX,DWORD PTR DS:\[ESI\]              ;  ESI 内存映射后 该区段的 VA  
0040112C  |.  51            |PUSH ECX                                ;  长度  
0040112D  |.  52            |PUSH EDX                                ;  源地址  
0040112E  |.  03C7          |ADD EAX,EDI  
00401130  |.  50            |PUSH EAX                                ;  目的地址  
00401131  |.  E8 AA560000   |CALL Fengyue'.004067E0                  ;  memcpy 函数  
00401136  |.  8B0E          |MOV ECX,DWORD PTR DS:\[ESI\]  
00401138  |.  8B56 FC       |MOV EDX,DWORD PTR DS:\[ESI-4\]  
0040113B  |.  03D1          |ADD EDX,ECX  
0040113D  |.  8D443A FF     |LEA EAX,DWORD PTR DS:\[EDX+EDI-1\]  
00401141  |.  83C4 0C       |ADD ESP,0C  
00401144  |.  8038 00       |CMP BYTE PTR DS:\[EAX\],0  
00401147  |.  75 13         |JNZ SHORT Fengyue'.0040115C  
00401149  |.  8D1439        |LEA EDX,DWORD PTR DS:\[ECX+EDI\]  
0040114C  |.  8D6424 00     |LEA ESP,DWORD PTR SS:\[ESP\]  
00401150  |>  3BC2          |/CMP EAX,EDX  
00401152  |.  76 08         ||JBE SHORT Fengyue'.0040115C  
00401154  |.  83E8 01       ||SUB EAX,1  
00401157  |.  8038 00       ||CMP BYTE PTR DS:\[EAX\],0  
0040115A  |.^ 74 F4         |\\JE SHORT Fengyue'.00401150  
0040115C  |>  2BC1          |SUB EAX,ECX  
0040115E  |.  2BC7          |SUB EAX,EDI  
00401160  |.  83C0 01       |ADD EAX,1  
00401163  |.  8946 04       |MOV DWORD PTR DS:\[ESI+4\],EAX            ;  求出该区段的 Size  并恢复该区段的真实尺寸  
00401166  |.  83C6 28       |ADD ESI,28  
00401169  |.  836C24 10 01  |SUB DWORD PTR SS:\[ESP+10\],1  
0040116E  |.^ 75 A0         \\JNZ SHORT Fengyue'.00401110  
00401170  |.  5E            POP ESI  
00401171  |.  5B            POP EBX  
00401172  |>  5F            POP EDI  
00401173  |.  5D            POP EBP  
00401174  |.  59            POP ECX  
00401175  \\.  C3            RETN  
  
  
<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<  
  
看一下上文中提到的 Memcpy 函数  
  
0040908F  /$  55            PUSH EBP  
  
F5 看看：  
  
int \_\_cdecl unknown\_libname\_62(int a1, int a2, int a3)  
{  
  int result; // eax@1  
  int v4; // ecx@1  
  int v5; // ecx@2  
  int v6; // ecx@7  
  int v7; // \[sp+4h\] \[bp-18h\]@2  
  int v8; // \[sp+0h\] \[bp-1Ch\]@7  
  
  v4 = a2 % -16;  
  result = a1;  
  if ( a1 % -16 | a2 % -16 )  
  {  
    if ( v4 == a1 % -16 )  
    {  
      v6 = -v4;  
      v8 = v6 + 16;  
      memcpy((void \*)a1, (const void \*)a2, v6 + 16);  
      unknown\_libname\_62(v8 + a1, v8 + a2, a3 - v8);  
      result = a1;  
    }  
    else  
    {  
      memcpy((void \*)a1, (const void \*)a2, a3);  
      result = a1;  
    }  
  }  
  else  
  {  
    v5 = a3 & 0x7F;  
    v7 = a3 & 0x7F;  
    if ( a3 != (a3 & 0x7F) )  
    {  
      fastcopy\_I(a1, a2, a3 - v5);  
      result = a1;  
      v5 = v7;  
    }  
    if ( v5 )  
    {  
      memcpy((void \*)(result + a3 - v5), (const void \*)(a3 + a2 - v5), v7);  
      result = a1;  
    }  
  }  
  return result;  
}  
  
<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<  
  
<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<  
  
函数三：CALL 00401180  把壳的非 PE 头段资源释放到一个空间并对代码段数据进行第一次的重定位调整  
  
00401180   $  83EC 10       SUB ESP,10  
00401183   .  A1 14DC4000   MOV EAX,DWORD PTR DS:\[40DC14\]  
00401188   .  53            PUSH EBX  
00401189   .  55            PUSH EBP  
0040118A   .  56            PUSH ESI  
0040118B   .  57            PUSH EDI  
0040118C   .  68 4CB34000   PUSH Fengyue'.0040B34C                   ; /RT\_RCDATA  
00401191   .  6A 67         PUSH 67                                  ; |ResourceName = 67  
00401193   .  50            PUSH EAX                                 ; |hModule => 00400000 (Fengyue')  
00401194   .  FF15 0CA04000 CALL DWORD PTR DS:\[<&KERNEL32.FindResour>; \\ (kernel32.FindResourceA) 获取资源句柄  
0040119A   .  8B0D 14DC4000 MOV ECX,DWORD PTR DS:\[40DC14\]            ;  Fengyue'.00400000  
004011A0   .  8BF0          MOV ESI,EAX  
004011A2   .  56            PUSH ESI                                 ; /hResource  
004011A3   .  51            PUSH ECX                                 ; |hModule => 00400000 (Fengyue')  
004011A4   .  897424 24     MOV DWORD PTR SS:\[ESP+24\],ESI            ; |  
004011A8   .  FF15 14A04000 CALL DWORD PTR DS:\[<&KERNEL32.LoadResour>; \\LoadResource  
004011AE   .  8B15 14DC4000 MOV EDX,DWORD PTR DS:\[40DC14\]            ;  Fengyue'.00400000  
004011B4   .  56            PUSH ESI                                 ; /hResource  
004011B5   .  52            PUSH EDX                                 ; |hModule => 00400000 (Fengyue')  
004011B6   .  8BD8          MOV EBX,EAX                              ; |  
004011B8   .  FF15 20A04000 CALL DWORD PTR DS:\[<&KERNEL32.SizeofReso>; \\SizeofResource  
004011BE   .  8B73 3C       MOV ESI,DWORD PTR DS:\[EBX+3C\]            ;  资源大小 1C00  
004011C1   .  8B4C1E 50     MOV ECX,DWORD PTR DS:\[ESI+EBX+50\]  
004011C5   .  6A 04         PUSH 4                                   ; /Protect = PAGE\_READWRITE  
004011C7   .  03F3          ADD ESI,EBX                              ; |  
004011C9   .  68 00100000   PUSH 1000                                ; |AllocationType = MEM\_COMMIT  
004011CE   .  A3 F4DB4000   MOV DWORD PTR DS:\[40DBF4\],EAX            ; |  
004011D3   .  0FB746 14     MOVZX EAX,WORD PTR DS:\[ESI+14\]           ; |  
004011D7   .  51            PUSH ECX                                 ; |Size  
004011D8   .  6A 00         PUSH 0                                   ; |Address = NULL  
004011DA   .  8D7C30 18     LEA EDI,DWORD PTR DS:\[EAX+ESI+18\]        ; |EDI 指向 HookStubt 字符串 // 该资源 PE 特殊的一个区段名称  
004011DE   .  FF15 28A04000 CALL DWORD PTR DS:\[<&KERNEL32.VirtualAll>; \\VirtualAlloc  
004011E4   .  8B96 80000000 MOV EDX,DWORD PTR DS:\[ESI+80\]            ;  输入表地址  
004011EA   .  8B8E 84000000 MOV ECX,DWORD PTR DS:\[ESI+84\]            ;  输入表长度  
004011F0   .  A3 3CDC4000   MOV DWORD PTR DS:\[40DC3C\],EAX            ;  刚申请的 size = 2000 指针  
004011F5   .  8B47 0C       MOV EAX,DWORD PTR DS:\[EDI+C\]             ;  HookStubt 区段的 VA 地址  
004011F8   .  2BD0          SUB EDX,EAX  
004011FA   .  8915 0CDC4000 MOV DWORD PTR DS:\[40DC0C\],EDX  
00401200   .  8B56 38       MOV EDX,DWORD PTR DS:\[ESI+38\]            ;  内存节对齐粒度 = 200  // 这个 PE 格式比较特殊 O(∩\_∩)O~  
00401203   .  890D DCDA4000 MOV DWORD PTR DS:\[40DADC\],ECX            ;  输入表长度送\[40DADC\]  
00401209   .  8B4F 08       MOV ECX,DWORD PTR DS:\[EDI+8\]             ;  HookStubt 区段的 Size  
0040120C   .  8D6F 08       LEA EBP,DWORD PTR DS:\[EDI+8\]  
0040120F   .  895424 18     MOV DWORD PTR SS:\[ESP+18\],EDX            ;  存放一下 内存对齐粒度  
00401213   .  8B56 28       MOV EDX,DWORD PTR DS:\[ESI+28\]            ;  OEP  
00401216   .  83E9 14       SUB ECX,14  
00401219   .  2BD0          SUB EDX,EAX  
0040121B   .  A3 2CDC4000   MOV DWORD PTR DS:\[40DC2C\],EAX            ;  HookStub 的 VA  
00401220   .  890D 04DC4000 MOV DWORD PTR DS:\[40DC04\],ECX            ;  74-14=60  
00401226   .  8915 40DC4000 MOV DWORD PTR DS:\[40DC40\],EDX            ;  AFE = EFE- 400  
0040122C   .  C74424 10 000>MOV DWORD PTR SS:\[ESP+10\],0  
00401234   .  C74424 14 040>MOV DWORD PTR SS:\[ESP+14\],4              ;  该资源 PE 的区段数  
0040123C   .  8D6424 00     LEA ESP,DWORD PTR SS:\[ESP\]  
00401240   >  8B45 00       MOV EAX,DWORD PTR SS:\[EBP\]  
00401243   .  8B4D 0C       MOV ECX,DWORD PTR SS:\[EBP+C\]             ;  该区段 在文件的偏移  
00401246   .  8B15 3CDC4000 MOV EDX,DWORD PTR DS:\[40DC3C\]            ;  为之申请的 空间  
0040124C   .  50            PUSH EAX                                 ;  SIZE  
0040124D   .  8B4424 14     MOV EAX,DWORD PTR SS:\[ESP+14\]            ;  该资源 PE 的区段数  
00401251   .  03CB          ADD ECX,EBX  
00401253   .  51            PUSH ECX                                 ;  源地址  
00401254   .  03D0          ADD EDX,EAX  
00401256   .  52            PUSH EDX                                 ;  循环 COPY 函数  
00401257   .  E8 84550000   CALL Fengyue'.004067E0                   ;  memcpy 函数  
0040125C   .  8B55 00       MOV EDX,DWORD PTR SS:\[EBP\]  
0040125F   .  8B4C24 24     MOV ECX,DWORD PTR SS:\[ESP+24\]            ;  对齐粒度  
00401263   .  8D440A FF     LEA EAX,DWORD PTR DS:\[EDX+ECX-1\]  
00401267   .  33D2          XOR EDX,EDX  
00401269   .  F7F1          DIV ECX  
0040126B   .  83C4 0C       ADD ESP,0C  
0040126E   .  83C5 28       ADD EBP,28                               ;  下一个区段的 size  
00401271   .  0FAFC1        IMUL EAX,ECX  
00401274   .  014424 10     ADD DWORD PTR SS:\[ESP+10\],EAX            ;  累加器  
00401278   .  836C24 14 01  SUB DWORD PTR SS:\[ESP+14\],1              ;  该资源 PE 的区段数  
0040127D   .^ 75 C1         JNZ SHORT Fengyue'.00401240  
0040127F   .  8B4424 10     MOV EAX,DWORD PTR SS:\[ESP+10\]  
00401283   .  6A 04         PUSH 4                                   ; /Protect = PAGE\_READWRITE  
00401285   .  A3 F4DB4000   MOV DWORD PTR DS:\[40DBF4\],EAX            ; |  
0040128A   .  8B86 A4000000 MOV EAX,DWORD PTR DS:\[ESI+A4\]            ; |  
00401290   .  68 00100000   PUSH 1000                                ; |AllocationType = MEM\_COMMIT  
00401295   .  50            PUSH EAX                                 ; |Size = 64  
00401296   .  6A 00         PUSH 0                                   ; |Address = NULL  
00401298   .  A3 1CDC4000   MOV DWORD PTR DS:\[40DC1C\],EAX            ; |  
0040129D   .  FF15 28A04000 CALL DWORD PTR DS:\[<&KERNEL32.VirtualAll>; \\VirtualAlloc  
004012A3   .  8B0D 1CDC4000 MOV ECX,DWORD PTR DS:\[40DC1C\]  
004012A9   .  8B96 A0000000 MOV EDX,DWORD PTR DS:\[ESI+A0\]            ;  Relocation Table address 重定位表  1C00  
004012AF   .  51            PUSH ECX  
004012B0   .  8BE8          MOV EBP,EAX  
004012B2   .  52            PUSH EDX  
004012B3   .  8BCB          MOV ECX,EBX  
004012B5   .  892D FCDB4000 MOV DWORD PTR DS:\[40DBFC\],EBP  
004012BB   .  E8 50080000   CALL Fengyue'.00401B10                   ; 该函数校对各节表的数据  
004012C0   .  83C4 04       ADD ESP,4  
004012C3   .  03C3          ADD EAX,EBX  
004012C5   .  50            PUSH EAX  
004012C6   .  55            PUSH EBP  
004012C7   .  E8 14550000   CALL Fengyue'.004067E0                   ;  memcpy 函数  
004012CC   .  8BCD          MOV ECX,EBP  
004012CE   .  83C4 0C       ADD ESP,0C  
004012D1   .  8379 04 00    CMP DWORD PTR DS:\[ECX+4\],0               ;  长度  
004012D5   .  74 7F         JE SHORT Fengyue'.00401356  
004012D7   .  EB 07         JMP SHORT Fengyue'.004012E0  
004012D9   .  8DA424 000000>LEA ESP,DWORD PTR SS:\[ESP\]  
004012E0   >  8B47 0C       MOV EAX,DWORD PTR DS:\[EDI+C\]             ;  第 1 个区段的 VA  
004012E3   .  8B59 04       MOV EBX,DWORD PTR DS:\[ECX+4\]             ;  size  
004012E6   .  25 00F0FFFF   AND EAX,FFFFF000  
004012EB   .  2901          SUB DWORD PTR DS:\[ECX\],EAX  
004012ED   .  83EB 08       SUB EBX,8                                ;  -8  
004012F0   .  D1EB          SHR EBX,1                                ;  / 2  word 型的去处理  
004012F2   .  BA 00000000   MOV EDX,0  
004012F7   .  74 54         JE SHORT Fengyue'.0040134D  
004012F9   .  8DA424 000000>LEA ESP,DWORD PTR SS:\[ESP\]  
00401300   >  0FB74451 08   MOVZX EAX,WORD PTR DS:\[ECX+EDX\*2+8\]      ;  按 WORD 型循环处理 数据  
00401305   .  8BE8          MOV EBP,EAX  
00401307   .  81E5 00F00000 AND EBP,0F000  
0040130D   .  81FD 00300000 CMP EBP,3000  
00401313   .  75 31         JNZ SHORT Fengyue'.00401346  
00401315   .  66:8B6F 0C    MOV BP,WORD PTR DS:\[EDI+C\]               ;  VA  
00401319   .  25 FF0F0000   AND EAX,0FFF                             ;  保留低 4\*3 位  
0040131E   .  66:81E5 FF0F  AND BP,0FFF  
00401323   .  66:2BC5       SUB AX,BP  
00401326   .  8B2D 3CDC4000 MOV EBP,DWORD PTR DS:\[40DC3C\]            ;  空间 基址  
0040132C   .  66:0D 0030    OR AX,3000                               ;  将数据处理后的结果  
00401330   .  66:894451 08  MOV WORD PTR DS:\[ECX+EDX\*2+8\],AX         ;  解码后的数据替换原数据 进行保存  
00401335   .  25 FF0F0000   AND EAX,0FFF  
0040133A   .  0301          ADD EAX,DWORD PTR DS:\[ECX\]  
0040133C   .  03C5          ADD EAX,EBP                              ;  计算出 空间中地址 进行重定位处理  
0040133E   .  8B6E 34       MOV EBP,DWORD PTR DS:\[ESI+34\]            ;  ImageBase  
00401341   .  036F 0C       ADD EBP,DWORD PTR DS:\[EDI+C\]             ;  VA  
00401344   .  2928          SUB DWORD PTR DS:\[EAX\],EBP               ;  进行重定位调整  
00401346   >  83C2 01       ADD EDX,1  
00401349   .  3BD3          CMP EDX,EBX  
0040134B   .^ 72 B3         JB SHORT Fengyue'.00401300  
0040134D   >  0349 04       ADD ECX,DWORD PTR DS:\[ECX+4\]  
00401350   .  8379 04 00    CMP DWORD PTR DS:\[ECX+4\],0  
00401354   .^ 75 8A         JNZ SHORT Fengyue'.004012E0  
00401356   >  8B4C24 1C     MOV ECX,DWORD PTR SS:\[ESP+1C\]  
0040135A   .  51            PUSH ECX                                 ; /hResource  
0040135B   .  FF15 10A04000 CALL DWORD PTR DS:\[<&KERNEL32.FreeResour>; \\FreeResource  
00401361   .  5F            POP EDI  
00401362   .  5E            POP ESI                                  ;  Fengyue'.0040F8A0  
00401363   .  5D            POP EBP  
00401364   .  5B            POP EBX  
00401365   .  83C4 10       ADD ESP,10  
00401368   .  C3            RETN  
  
  
  
  
\>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  
  
跟进 CALL Fengyue'.00401B10  看一下 该函数校对各节表的数据   
  
00401B10  /$  8B41 3C       MOV EAX,DWORD PTR DS:\[ECX+3C\]            ;  PE 的偏移  
00401B13  |.  8B5424 04     MOV EDX,DWORD PTR SS:\[ESP+4\]             ;  参数 II 1C00  
00401B17  |.  03C1          ADD EAX,ECX  
00401B19  |.  0FB748 14     MOVZX ECX,WORD PTR DS:\[EAX+14\]           ;  IMAGE\_OPTIONAL\_HEADER32 SIZE = E0  
00401B1D  |.  83EC 10       SUB ESP,10  
00401B20  |.  3B5401 24     CMP EDX,DWORD PTR DS:\[ECX+EAX+24\]        ;  第一个区段的 VA  
00401B24  |.  53            PUSH EBX                                 ;  资源 AV  
00401B25  |.  8B58 38       MOV EBX,DWORD PTR DS:\[EAX+38\]            ;  内存对齐粒度  
00401B28  |.  8D4C01 18     LEA ECX,DWORD PTR DS:\[ECX+EAX+18\]        ;  第一个区段的 NAME  
00401B2C  |.  56            PUSH ESI                                 ;  PE AV  
00401B2D  |.  8B70 3C       MOV ESI,DWORD PTR DS:\[EAX+3C\]            ;  磁盘文件 对齐粒度  
00401B30  |.  73 0F         JNB SHORT Fengyue'.00401B41              ;  上方 其实是用了一个结构体  其中 EAX=PE 的 VA 作为基址来寻址  
00401B32  |.  3950 54       CMP DWORD PTR DS:\[EAX+54\],EDX  
00401B35  |.  5E            POP ESI  
00401B36  |.  1BC0          SBB EAX,EAX  
00401B38  |.  F7D0          NOT EAX  
00401B3A  |.  23C2          AND EAX,EDX  
00401B3C  |.  5B            POP EBX  
00401B3D  |.  83C4 10       ADD ESP,10  
00401B40  |.  C3            RETN  
00401B41  |>  55            PUSH EBP  
00401B42  |.  57            PUSH EDI  
00401B43  |.  0FB778 06     MOVZX EDI,WORD PTR DS:\[EAX+6\]            ;  NumberOfSections = 5  
00401B47  |.  85FF          TEST EDI,EDI  
00401B49  |.  C74424 14 000>MOV DWORD PTR SS:\[ESP+14\],0              ;  做计数器  
00401B51  |.  897C24 1C     MOV DWORD PTR SS:\[ESP+1C\],EDI  
00401B55  |.  0F8E 9B000000 JLE Fengyue'.00401BF6  
00401B5B  |.  8D51 08       LEA EDX,DWORD PTR DS:\[ECX+8\]             ;  该区段的 VA  
00401B5E  |.  895424 10     MOV DWORD PTR SS:\[ESP+10\],EDX  
00401B62  |>  8B4A 0C       /MOV ECX,DWORD PTR DS:\[EDX+C\]            ;  文件偏移  
00401B65  |.  85C9          |TEST ECX,ECX  
00401B67  |.  8B02          |MOV EAX,DWORD PTR DS:\[EDX\]              ;  Size  
00401B69  |.  8B6A 08       |MOV EBP,DWORD PTR DS:\[EDX+8\]            ;  文件中的长度  
00401B6C  |.  74 6E         |JE SHORT Fengyue'.00401BDC  
00401B6E  |.  33D2          |XOR EDX,EDX  
00401B70  |.  F7F3          |DIV EBX  
00401B72  |.  8BF8          |MOV EDI,EAX  
00401B74  |.  33C0          |XOR EAX,EAX  
00401B76  |.  85D2          |TEST EDX,EDX  
00401B78  |.  0F95C0        |SETNE AL  
00401B7B  |.  33D2          |XOR EDX,EDX  
00401B7D  |.  03C7          |ADD EAX,EDI  
00401B7F  |.  0FAFC3        |IMUL EAX,EBX                            ;  计算一下 该区段 size 的对齐尺寸  
00401B82  |.  894424 18     |MOV DWORD PTR SS:\[ESP+18\],EAX  
00401B86  |.  8BC5          |MOV EAX,EBP  
00401B88  |.  F7F6          |DIV ESI  
00401B8A  |.  8BF8          |MOV EDI,EAX  
00401B8C  |.  33C0          |XOR EAX,EAX  
00401B8E  |.  85D2          |TEST EDX,EDX  
00401B90  |.  0F95C0        |SETNE AL  
00401B93  |.  33D2          |XOR EDX,EDX  
00401B95  |.  03C7          |ADD EAX,EDI  
00401B97  |.  0FAFC6        |IMUL EAX,ESI  
00401B9A  |.  8BF8          |MOV EDI,EAX  
00401B9C  |.  8BC1          |MOV EAX,ECX  
00401B9E  |.  F7F6          |DIV ESI  
00401BA0  |.  85D2          |TEST EDX,EDX  
00401BA2  |.  74 05         |JE SHORT Fengyue'.00401BA9  
00401BA4  |.  0FAFC6        |IMUL EAX,ESI  
00401BA7  |.  8BC8          |MOV ECX,EAX  
00401BA9  |>  8B5424 10     |MOV EDX,DWORD PTR SS:\[ESP+10\]  
00401BAD  |.  8B52 04       |MOV EDX,DWORD PTR DS:\[EDX+4\]            ;  VA  
00401BB0  |.  395424 24     |CMP DWORD PTR SS:\[ESP+24\],EDX           ;  文件的 size  
00401BB4  |.  72 1E         |JB SHORT Fengyue'.00401BD4  
00401BB6  |.  8B4424 18     |MOV EAX,DWORD PTR SS:\[ESP+18\]  
00401BBA  |.  03C2          |ADD EAX,EDX  
00401BBC  |.  394424 24     |CMP DWORD PTR SS:\[ESP+24\],EAX  
00401BC0  |.  73 12         |JNB SHORT Fengyue'.00401BD4  
00401BC2  |.  8BC1          |MOV EAX,ECX  
00401BC4  |.  2BC2          |SUB EAX,EDX  
00401BC6  |.  034424 24     |ADD EAX,DWORD PTR SS:\[ESP+24\]  
00401BCA  |.  3BC1          |CMP EAX,ECX  
00401BCC  |.  72 06         |JB SHORT Fengyue'.00401BD4  
00401BCE  |.  03F9          |ADD EDI,ECX  
00401BD0  |.  3BC7          |CMP EAX,EDI  
00401BD2  |.  72 24         |JB SHORT Fengyue'.00401BF8  
00401BD4  |>  8B7C24 1C     |MOV EDI,DWORD PTR SS:\[ESP+1C\]  
00401BD8  |.  8B5424 10     |MOV EDX,DWORD PTR SS:\[ESP+10\]  
00401BDC  |>  8B4424 14     |MOV EAX,DWORD PTR SS:\[ESP+14\]  
00401BE0  |.  83C0 01       |ADD EAX,1  
00401BE3  |.  83C2 28       |ADD EDX,28  
00401BE6  |.  3BC7          |CMP EAX,EDI                             ;  下一个区段的 size  
00401BE8  |.  894424 14     |MOV DWORD PTR SS:\[ESP+14\],EAX  
00401BEC  |.  895424 10     |MOV DWORD PTR SS:\[ESP+10\],EDX  
00401BF0  |.^ 0F8C 6CFFFFFF \\JL Fengyue'.00401B62                    ;  循环处理各区段表中前四项的数据 校对  
00401BF6  |>  33C0          XOR EAX,EAX  
00401BF8  |>  5F            POP EDI  
00401BF9  |.  5D            POP EBP  
00401BFA  |.  5E            POP ESI  
00401BFB  |.  5B            POP EBX  
00401BFC  |.  83C4 10       ADD ESP,10  
00401BFF  \\.  C3            RETN  
  
  
<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<  
  
\>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>  
  
第四个函数：CALL 00401370    这个函数 为 PE 添加新区段并将内存中的 PE 文件处理为磁盘 PE 文件格式  
  
  
00401370  /$  A1 F4DB4000   MOV EAX,DWORD PTR DS:\[40DBF4\]  
00401375  |.  8B0D F8DB4000 MOV ECX,DWORD PTR DS:\[40DBF8\]  
0040137B  |.  8B49 50       MOV ECX,DWORD PTR DS:\[ECX+50\]  
0040137E  |.  83EC 08       SUB ESP,8  
00401381  |.  53            PUSH EBX  
00401382  |.  55            PUSH EBP  
00401383  |.  56            PUSH ESI  
00401384  |.  57            PUSH EDI  
00401385  |.  05 FF010000   ADD EAX,1FF  
0040138A  |.  C1E8 09       SHR EAX,9  
0040138D  |.  6A 04         PUSH 4                                   ; /Protect = PAGE\_READWRITE  
0040138F  |.  C1E0 09       SHL EAX,9                                ; |  
00401392  |.  03C8          ADD ECX,EAX                              ; |  
00401394  |.  68 00100000   PUSH 1000                                ; |AllocationType = MEM\_COMMIT  
00401399  |.  51            PUSH ECX                                 ; |Size  
0040139A  |.  6A 00         PUSH 0                                   ; |Address = NULL  
0040139C  |.  A3 F4DB4000   MOV DWORD PTR DS:\[40DBF4\],EAX            ; |  
004013A1  |.  890D F0DB4000 MOV DWORD PTR DS:\[40DBF0\],ECX            ; |  
004013A7  |.  FF15 28A04000 CALL DWORD PTR DS:\[<&KERNEL32.VirtualAll>; \\VirtualAlloc  
004013AD  |.  8B15 F8DB4000 MOV EDX,DWORD PTR DS:\[40DBF8\]            ;  申请了新文件的空间  
004013B3  |.  8B0D ECDB4000 MOV ECX,DWORD PTR DS:\[40DBEC\]  
004013B9  |.  8BD8          MOV EBX,EAX  
004013BB  |.  8B42 50       MOV EAX,DWORD PTR DS:\[EDX+50\]  
004013BE  |.  50            PUSH EAX  
004013BF  |.  51            PUSH ECX  
004013C0  |.  53            PUSH EBX  
004013C1  |.  891D 24DC4000 MOV DWORD PTR DS:\[40DC24\],EBX  
004013C7  |.  E8 14540000   CALL Fengyue'.004067E0                   ;  将原文件全部 COPY 到新空间  
004013CC  |.  8B73 3C       MOV ESI,DWORD PTR DS:\[EBX+3C\]  
004013CF  |.  0FB7441E 06   MOVZX EAX,WORD PTR DS:\[ESI+EBX+6\]  
004013D4  |.  0FB7541E 14   MOVZX EDX,WORD PTR DS:\[ESI+EBX+14\]  
004013D9  |.  03F3          ADD ESI,EBX  
004013DB  |.  8D4C32 18     LEA ECX,DWORD PTR DS:\[EDX+ESI+18\]  
004013DF  |.  83E8 01       SUB EAX,1  
004013E2  |.  8D0480        LEA EAX,DWORD PTR DS:\[EAX+EAX\*4\]  
004013E5  |.  8B54C1 08     MOV EDX,DWORD PTR DS:\[ECX+EAX\*8+8\]  
004013E9  |.  8D04C1        LEA EAX,DWORD PTR DS:\[ECX+EAX\*8\]  
004013EC  |.  890D 18DC4000 MOV DWORD PTR DS:\[40DC18\],ECX  
004013F2  |.  8B48 0C       MOV ECX,DWORD PTR DS:\[EAX+C\]  
004013F5  |.  A1 F4DB4000   MOV EAX,DWORD PTR DS:\[40DBF4\]  
004013FA  |.  8DBC11 FF0F00>LEA EDI,DWORD PTR DS:\[ECX+EDX+FFF\]  
00401401  |.  8B0D 3CDC4000 MOV ECX,DWORD PTR DS:\[40DC3C\]  
00401407  |.  C1EF 0C       SHR EDI,0C  
0040140A  |.  C1E7 0C       SHL EDI,0C  
0040140D  |.  50            PUSH EAX  
0040140E  |.  8BEF          MOV EBP,EDI  
00401410  |.  2B2D 2CDC4000 SUB EBP,DWORD PTR DS:\[40DC2C\]  
00401416  |.  51            PUSH ECX  
00401417  |.  8D143B        LEA EDX,DWORD PTR DS:\[EBX+EDI\]  
0040141A  |.  52            PUSH EDX  
0040141B  |.  891D 20DC4000 MOV DWORD PTR DS:\[40DC20\],EBX  
00401421  |.  8935 F8DB4000 MOV DWORD PTR DS:\[40DBF8\],ESI  
00401427  |.  892D 2CDC4000 MOV DWORD PTR DS:\[40DC2C\],EBP  
0040142D  |.  E8 AE530000   CALL Fengyue'.004067E0                   ;  将 壳 区段 代码 连接到 PE 文件  
00401432  |.  A1 FCDB4000   MOV EAX,DWORD PTR DS:\[40DBFC\]  
00401437  |.  8B48 04       MOV ECX,DWORD PTR DS:\[EAX+4\]  
0040143A  |.  33D2          XOR EDX,EDX  
0040143C  |.  83C4 18       ADD ESP,18  
0040143F  |.  3BCA          CMP ECX,EDX  
00401441  |.  74 51         JE SHORT Fengyue'.00401494               ;  处理一下 壳区段部分代码的偏移值 第二次的重定位调整  
00401443  |>  83C1 F8       /ADD ECX,-8  
00401446  |.  D1E9          |SHR ECX,1  
00401448  |.  894C24 10     |MOV DWORD PTR SS:\[ESP+10\],ECX  
0040144C  |.  74 3C         |JE SHORT Fengyue'.0040148A  
0040144E  |.  8BFF          |MOV EDI,EDI  
00401450  |>  66:8B4C50 08  |/MOV CX,WORD PTR DS:\[EAX+EDX\*2+8\]       ;  每次取两个字节 相对于壳区段的偏移值  
00401455  |.  66:81E1 00F0  ||AND CX,0F000  
0040145A  |.  66:81F9 0030  ||CMP CX,3000  
0040145F  |.  75 1E         ||JNZ SHORT Fengyue'.0040147F  
00401461  |.  0FB74C50 08   ||MOVZX ECX,WORD PTR DS:\[EAX+EDX\*2+8\]  
00401466  |.  8B6E 34       ||MOV EBP,DWORD PTR DS:\[ESI+34\]  
00401469  |.  81E1 FF0F0000 ||AND ECX,0FFF  
0040146F  |.  0308          ||ADD ECX,DWORD PTR DS:\[EAX\]  
00401471  |.  03EF          ||ADD EBP,EDI                            ;  壳区段的 VA  
00401473  |.  03CB          ||ADD ECX,EBX                            ;  new alloc 中壳区段 RVA  
00401475  |.  03CF          ||ADD ECX,EDI                            ;  壳区段代码的 RVA  
00401477  |.  0129          ||ADD DWORD PTR DS:\[ECX\],EBP             ;  重定位调整 统一加上壳差值= 00409000  
00401479  |.  8B2D 2CDC4000 ||MOV EBP,DWORD PTR DS:\[40DC2C\]  
0040147F  |>  83C2 01       ||ADD EDX,1  
00401482  |.  3B5424 10     ||CMP EDX,DWORD PTR SS:\[ESP+10\]          ;  循环 2E 次  
00401486  |.^ 72 C8         |\\JB SHORT Fengyue'.00401450  
00401488  |.  33D2          |XOR EDX,EDX  
0040148A  |>  0340 04       |ADD EAX,DWORD PTR DS:\[EAX+4\]  
0040148D  |.  8B48 04       |MOV ECX,DWORD PTR DS:\[EAX+4\]  
00401490  |.  3BCA          |CMP ECX,EDX  
00401492  |.^ 75 AF         \\JNZ SHORT Fengyue'.00401443  
00401494  |>  A1 04DC4000   MOV EAX,DWORD PTR DS:\[40DC04\]            ;  ---------  
00401499  |.  03C3          ADD EAX,EBX  
0040149B  |.  03C7          ADD EAX,EDI  
0040149D  |.  A3 08DC4000   MOV DWORD PTR DS:\[40DC08\],EAX            ;  // 以下代码 将 PE 头中输入表信息 处理到 壳区段  
004014A2  |.  8B8E 80000000 MOV ECX,DWORD PTR DS:\[ESI+80\]            ;  输入表地址  
004014A8  |.  8948 04       MOV DWORD PTR DS:\[EAX+4\],ECX  
004014AB  |.  8B0D 08DC4000 MOV ECX,DWORD PTR DS:\[40DC08\]            ;  这一块要建立一个输入表数据  
004014B1  |.  8B86 84000000 MOV EAX,DWORD PTR DS:\[ESI+84\]            ;  输入表长度  
004014B7  |.  8941 08       MOV DWORD PTR DS:\[ECX+8\],EAX  
004014BA  |.  8B0D 08DC4000 MOV ECX,DWORD PTR DS:\[40DC08\]  
004014C0  |.  8B46 28       MOV EAX,DWORD PTR DS:\[ESI+28\]            ;  OEP  
004014C3  |.  8901          MOV DWORD PTR DS:\[ECX\],EAX  
004014C5  |.  A1 40DC4000   MOV EAX,DWORD PTR DS:\[40DC40\]            ;  长度  
004014CA  |.  8B0D 0CDC4000 MOV ECX,DWORD PTR DS:\[40DC0C\]  
004014D0  |.  03C7          ADD EAX,EDI                              ;  计算出 VA  
004014D2  |.  8946 28       MOV DWORD PTR DS:\[ESI+28\],EAX            ;  计算出 OEP （RVA）  
004014D5  |.  8D0439        LEA EAX,DWORD PTR DS:\[ECX+EDI\]  
004014D8  |.  8B0D DCDA4000 MOV ECX,DWORD PTR DS:\[40DADC\]  
004014DE  |.  898E 84000000 MOV DWORD PTR DS:\[ESI+84\],ECX            ;  修改输入表长度  
004014E4  |.  8B0D F4DB4000 MOV ECX,DWORD PTR DS:\[40DBF4\]  
004014EA  |.  81C1 FF0F0000 ADD ECX,0FFF  
004014F0  |.  C1E9 0C       SHR ECX,0C  
004014F3  |.  C1E1 0C       SHL ECX,0C  
004014F6  |.  014E 50       ADD DWORD PTR DS:\[ESI+50\],ECX            ;  增加 内存 镜像 大小  
004014F9  |.  8986 80000000 MOV DWORD PTR DS:\[ESI+80\],EAX            ;  修改导入表地址  
004014FF  |.  03C3          ADD EAX,EBX  
00401501  |.  8996 D0000000 MOV DWORD PTR DS:\[ESI+D0\],EDX            ;  清空 Bound Import Table address  
00401507  |.  8996 D4000000 MOV DWORD PTR DS:\[ESI+D4\],EDX            ;  清空 Bound Import Table size  
0040150D  |.  8996 D8000000 MOV DWORD PTR DS:\[ESI+D8\],EDX            ;  清空 Import Address Table address  
00401513  |.  8996 DC000000 MOV DWORD PTR DS:\[ESI+DC\],EDX            ;  清空 Import Address Table size  
00401519  |.  8996 A8000000 MOV DWORD PTR DS:\[ESI+A8\],EDX            ;  清空 Import Table address  
0040151F  |.  8996 AC000000 MOV DWORD PTR DS:\[ESI+AC\],EDX            ;  清空 Import Table size  
00401525  |.  3950 10       CMP DWORD PTR DS:\[EAX+10\],EDX  
00401528  |.  894C24 14     MOV DWORD PTR SS:\[ESP+14\],ECX            ;  保存一下区段大小  
0040152C  |.  74 29         JE SHORT Fengyue'.00401557  
0040152E  |.  EB 02         JMP SHORT Fengyue'.00401532  
00401530  |>  33D2          /XOR EDX,EDX  
00401532  |>  0168 10        ADD DWORD PTR DS:\[EAX+10\],EBP           ;  开始处理壳段 IAT 部分 100+偏移值(EBP=8C00)  
00401535  |.  8B48 10       |MOV ECX,DWORD PTR DS:\[EAX+10\]  
00401538  |.  0168 0C       |ADD DWORD PTR DS:\[EAX+C\],EBP  
0040153B  |.  03CB          |ADD ECX,EBX                             ;  壳区段 IAT 开始地址  
0040153D  |.  8910          |MOV DWORD PTR DS:\[EAX\],EDX  
0040153F  |.  8339 00       |CMP DWORD PTR DS:\[ECX\],0  
00401542  |.  74 0A         |JE SHORT Fengyue'.0040154E  
00401544  |>  0129          |/ADD DWORD PTR DS:\[ECX\],EBP             ;  循环处理壳段 IAT 的偏移地址  // 重定位的处理  
00401546  |.  83C1 04       ||ADD ECX,4  
00401549  |.  8339 00       ||CMP DWORD PTR DS:\[ECX\],0  
0040154C  |.^ 75 F6         |\\JNZ SHORT Fengyue'.00401544  
0040154E  |>  83C0 14       |ADD EAX,14  
00401551  |.  8378 10 00    |CMP DWORD PTR DS:\[EAX+10\],0  
00401555  |.^ 75 D9         \\JNZ SHORT Fengyue'.00401530             ;  以上循环为处理 IAT 的偏移  
00401557  |>  66:8346 06 01 ADD WORD PTR DS:\[ESI+6\],1                ;  区段+1  
0040155C  |.  0FB746 06     MOVZX EAX,WORD PTR DS:\[ESI+6\]  
00401560  |.  0FB7C8        MOVZX ECX,AX  
00401563  |.  0FB746 14     MOVZX EAX,WORD PTR DS:\[ESI+14\]  
00401567  |.  8D1489        LEA EDX,DWORD PTR DS:\[ECX+ECX\*4\]  
0040156A  |.  8D14D0        LEA EDX,DWORD PTR DS:\[EAX+EDX\*8\]  
0040156D  |.  2BD3          SUB EDX,EBX  
0040156F  |.  8D9432 170200>LEA EDX,DWORD PTR DS:\[EDX+ESI+217\]  
00401576  |.  C1EA 09       SHR EDX,9  
00401579  |.  C1E2 09       SHL EDX,9  
0040157C  |.  85C9          TEST ECX,ECX  
0040157E  |.  8915 D8DA4000 MOV DWORD PTR DS:\[40DAD8\],EDX            ;  400  
00401584  |.  8BEA          MOV EBP,EDX  
00401586  |.  C74424 10 000>MOV DWORD PTR SS:\[ESP+10\],0              ;  计数器  
0040158E  |.  7E 36         JLE SHORT Fengyue'.004015C6  
00401590  |.  8B1D 18DC4000 MOV EBX,DWORD PTR DS:\[40DC18\]  
00401596  |.  83C3 10       ADD EBX,10  
00401599  |.  8DA424 000000>LEA ESP,DWORD PTR SS:\[ESP\]  
004015A0  |>  8B03          /MOV EAX,DWORD PTR DS:\[EBX\]  
004015A2  |.  834424 10 01  |ADD DWORD PTR SS:\[ESP+10\],1  
004015A7  |.  05 FF010000   |ADD EAX,1FF  
004015AC  |.  C1E8 09       |SHR EAX,9  
004015AF  |.  C1E0 09       |SHL EAX,9  
004015B2  |.  896B 04       |MOV DWORD PTR DS:\[EBX+4\],EBP  
004015B5  |.  8903          |MOV DWORD PTR DS:\[EBX\],EAX              ;  该区段文件大小 处理成 200 整数倍  
004015B7  |.  03E8          |ADD EBP,EAX  
004015B9  |.  0FB746 06     |MOVZX EAX,WORD PTR DS:\[ESI+6\]  
004015BD  |.  83C3 28       |ADD EBX,28  
004015C0  |.  394424 10     |CMP DWORD PTR SS:\[ESP+10\],EAX  
004015C4  |.^ 7C DA         \\JL SHORT Fengyue'.004015A0  
004015C6  |>  8956 54       MOV DWORD PTR DS:\[ESI+54\],EDX  
004015C9  |.  8B15 18DC4000 MOV EDX,DWORD PTR DS:\[40DC18\]  
004015CF  |.  8D4C89 FB     LEA ECX,DWORD PTR DS:\[ECX+ECX\*4-5\]  
004015D3  |.  8D04CA        LEA EAX,DWORD PTR DS:\[EDX+ECX\*8\]         ;  最后一个节表的偏移值  
004015D6  |.  8B4C24 14     MOV ECX,DWORD PTR SS:\[ESP+14\]  
004015DA  |.  8B50 E8       MOV EDX,DWORD PTR DS:\[EAX-18\]  
004015DD  |.  0350 EC       ADD EDX,DWORD PTR DS:\[EAX-14\]  
004015E0  |.  8948 08       MOV DWORD PTR DS:\[EAX+8\],ECX             ;  FengYue VirtualSize = 2000  
004015E3  |.  8B0D F4DB4000 MOV ECX,DWORD PTR DS:\[40DBF4\]  
004015E9  |.  81C1 FF010000 ADD ECX,1FF  
004015EF  |.  C1E9 09       SHR ECX,9  
004015F2  |.  C740 24 E0000>MOV DWORD PTR DS:\[EAX+24\],E00000E0       ;  设置区段属性 Characteristics =   
  
CODE|INITIALIZED\_DATA|UNINITIALIZED\_DATA|EXECUTE|READ|WRITE  
004015F9  |.  8978 0C       MOV DWORD PTR DS:\[EAX+C\],EDI             ;  VirtualAddress = 9000  
004015FC  |.  8950 14       MOV DWORD PTR DS:\[EAX+14\],EDX            ;  PointerToRawData  = 5600  
004015FF  |.  C1E1 09       SHL ECX,9  
00401602  |.  8948 10       MOV DWORD PTR DS:\[EAX+10\],ECX            ;  SizeOfRawData = 1800  
00401605  |.  8B15 58B34000 MOV EDX,DWORD PTR DS:\[40B358\]  
0040160B  |.  8910          MOV DWORD PTR DS:\[EAX\],EDX  
0040160D  |.  8B15 5CB34000 MOV EDX,DWORD PTR DS:\[40B35C\]  
00401613  |.  6A 04         PUSH 4                                   ; /Protect = PAGE\_READWRITE  
00401615  |.  8950 04       MOV DWORD PTR DS:\[EAX+4\],EDX             ; |添加区段名 Fengyue  
00401618  |.  8A15 60B34000 MOV DL,BYTE PTR DS:\[40B360\]              ; |  
0040161E  |.  03E9          ADD EBP,ECX                              ; |  
00401620  |.  68 00100000   PUSH 1000                                ; |AllocationType = MEM\_COMMIT  
00401625  |.  55            PUSH EBP                                 ; |Size = 6E00  
00401626  |.  8850 08       MOV BYTE PTR DS:\[EAX+8\],DL               ; |  
00401629  |.  6A 00         PUSH 0                                   ; |Address = NULL  
0040162B  |.  892D 28DC4000 MOV DWORD PTR DS:\[40DC28\],EBP            ; |  
00401631  |.  C746 3C 00020>MOV DWORD PTR DS:\[ESI+3C\],200            ; |  
00401638  |.  FF15 28A04000 CALL DWORD PTR DS:\[<&KERNEL32.VirtualAll>; \\VirtualAlloc  
0040163E  |.  8B1D F8DB4000 MOV EBX,DWORD PTR DS:\[40DBF8\]            ;  申请了文件大小的空间  
00401644  |.  8B4B 28       MOV ECX,DWORD PTR DS:\[EBX+28\]            ;  OEP  
00401647  |.  8BE8          MOV EBP,EAX                              ;  新空间的指针 010C0000  
00401649  |.  A1 24DC4000   MOV EAX,DWORD PTR DS:\[40DC24\]            ;  连接好的 PE 文件 010B0000  
0040164E  |.  8D7C01 09     LEA EDI,DWORD PTR DS:\[ECX+EAX+9\]  
00401652  |.  B9 0A000000   MOV ECX,0A  
00401657  |.  BE 64B34000   MOV ESI,Fengyue'.0040B364  
0040165C  |.  F3:A5         REP MOVS DWORD PTR ES:\[EDI\],DWORD PTR DS>;  cOPY 了一个壳的标识  
0040165E  |.  66:A5         MOVS WORD PTR ES:\[EDI\],WORD PTR DS:\[ESI\]  
00401660  |.  8B53 54       MOV EDX,DWORD PTR DS:\[EBX+54\]  
00401663  |.  52            PUSH EDX  
00401664  |.  50            PUSH EAX  
00401665  |.  55            PUSH EBP  
00401666  |.  892D 00DC4000 MOV DWORD PTR DS:\[40DC00\],EBP  
0040166C  |.  E8 6F510000   CALL Fengyue'.004067E0                   ;  先 Copy 了 PE 头 Fengyue'sDll-Game v0.22 by 海月影\[CUG\]  
00401671  |.  8B73 54       MOV ESI,DWORD PTR DS:\[EBX+54\]  
00401674  |.  83C4 0C       ADD ESP,0C  
00401677  |.  66:837B 06 00 CMP WORD PTR DS:\[EBX+6\],0  
0040167C  |.  8935 28DC4000 MOV DWORD PTR DS:\[40DC28\],ESI  
00401682  |.  C74424 10 000>MOV DWORD PTR SS:\[ESP+10\],0  
0040168A  |.  76 40         JBE SHORT Fengyue'.004016CC  
0040168C  |.  8B3D 18DC4000 MOV EDI,DWORD PTR DS:\[40DC18\]  
00401692  |.  83C7 10       ADD EDI,10  
00401695  |>  8B4F FC       /MOV ECX,DWORD PTR DS:\[EDI-4\]  
00401698  |.  8B07          |MOV EAX,DWORD PTR DS:\[EDI\]  
0040169A  |.  030D 24DC4000 |ADD ECX,DWORD PTR DS:\[40DC24\]  
004016A0  |.  50            |PUSH EAX  
004016A1  |.  51            |PUSH ECX  
004016A2  |.  8D142E        |LEA EDX,DWORD PTR DS:\[ESI+EBP\]  
004016A5  |.  52            |PUSH EDX  
004016A6  |.  E8 35510000   |CALL Fengyue'.004067E0  
004016AB  |.  8B4424 1C     |MOV EAX,DWORD PTR SS:\[ESP+1C\]  
004016AF  |.  0FB74B 06     |MOVZX ECX,WORD PTR DS:\[EBX+6\]  
004016B3  |.  0337          |ADD ESI,DWORD PTR DS:\[EDI\]  
004016B5  |.  83C0 01       |ADD EAX,1  
004016B8  |.  83C4 0C       |ADD ESP,0C  
004016BB  |.  83C7 28       |ADD EDI,28  
004016BE  |.  3BC1          |CMP EAX,ECX  
004016C0  |.  894424 10     |MOV DWORD PTR SS:\[ESP+10\],EAX  
004016C4  |.^ 7C CF         \\JL SHORT Fengyue'.00401695              ;  这里就是 Copy 5 个内存区段的数据格式 保存成 磁盘文件格式  
004016C6  |.  8935 28DC4000 MOV DWORD PTR DS:\[40DC28\],ESI  
004016CC  |>  5F            POP EDI  
004016CD  |.  5E            POP ESI  
004016CE  |.  5D            POP EBP  
004016CF  |.  5B            POP EBX  
004016D0  |.  83C4 08       ADD ESP,8  
004016D3  \\.  C3            RETN  
  
  
<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<<  
  
  
    高手飘过，如对处理 PE 的流程感兴趣，不要取看文章，随便加载一个 PE 文件，然后查看 PE 头各数据的信息，然去直接动态跟踪该流程。百看不如一练，写东西比直接调试麻烦  
  
的多~~分析的有点乱，有出错之处请以自己的调试为准。先读取硬盘的磁盘文件，再根据 PE 头各区段 VirtualAddress、VirtualAddress 和 VirtualSize 来将磁盘文件映射到内存，处理  
  
映射到内存的 PE 文件，包括重定位处理、添加区段等操作。处理结束后再根据 PE 头区段的 PointerToRawData、VirtualSize 和 SizeOfRawData 来存储为磁盘文件格式并保存。  
    通过动态调试把壳处理 PE 文件：处理 PE 头（OEP 等）、节表、添加区段、增加壳代码并重定位输入表、将 PE 文件映射内存处理后并保存为磁盘文件的过程有了一个更清晰的认识。文中有些 E 文写的可能不是很对，大家动态跟踪的时候自己修正一下吧。