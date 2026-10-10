

# 分析了一下 ArmaFindprotected 原理

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-551.htm`](https://www.pediy.com/kssd/pediy08/pediy8-551.htm)

小小分析了一下 ArmaFindprotected 原理  
  
  
1.脱壳  
  
好像是 upack,载入出错,停在系统断点  
  
7C921230 >/$  CC            INT3  
7C921231      C3            RETN  
  
PEid 查看入口是 00001018，去 400000+1018 处 F2，F9 到达壳入口。  
  
00401018    BE B0114000     MOV     ESI, 004011B0  
0040101D    AD              LODS    DWORD PTR \[ESI\]  
0040101E    50              PUSH    EAX                              ; Armadill.00402025  
0040101F    FF76 34         PUSH    DWORD PTR \[ESI+34\]  
00401022    EB 7C           JMP     SHORT 004010A0  
  
本来多跟了几步，后来发现第一个 push eax 就是 OEP，Ctrl+G eax，F4 到 OEP。  
  
然后用 LordPE dump， ImpRec 修复，另外修复数据目录和 NumberOfDictionaries，不用每次都有错误提示了。  
  
\->DOS 头  
   e\_magic:     0x5A4D  
   e\_cblp:      0x454B  
   e\_cp:        0x4E52  
   e\_crlc:      0x4C45  
   e\_cparhdr:   0x3233  
   e\_minalloc:  0x442E  
   e\_maxalloc:  0x4C4C  
   e\_ss:        0x0000  
   e\_sp:        0x4550  
   e\_csum:      0x0000  
   e\_ip:        0x014C  
   e\_cs:        0x0004  
   e\_lfarlc:    0xB0BE  
   e\_ovno:      0x4011  
   e\_res:       0xAD00FF5034767CEB  
   e\_oemid:     0x00E0  
   e\_oeminfo:   0x010F  
   e\_res2:      0x010B6F4C6461694C726272614179000020250000  
   e\_lfanew:    0x00000010  
  
\->文件头  
   所需机器:              0x014C  (I386)  
   区段数目:              0x0004  
   时间日期戳:            0x4011B0BE  (GMT: Fri Jan 23 23:39:42 2004)  
   符号表指针:            0xFF50AD00  
   符号数:                0x7CEB3476  
   可选文件头大小:        0x00E0  
   特征:                  0x010F  
                          (去除重定位表)  
                          (文件可执行)  
                          (去除行号)  
                          (去除本地标志)  
                          (需 32 位机器)  
  
\->可选文件头  
   幻术:                         0x010B  (HDR32\_MAGIC)  
   连接器主版本:                 0x4C  
   连接器副版本:                 0x6F  -> 76.111  
   代码段大小:                   0x694C6461  
   已初始化数据大小:             0x72617262  
   未初始化数据大小:             0x00004179  
   程序执行入口点地址:           0x00002025  
   代码段基址:                   0x00000010  
   数据段基址:                   0x00005000  
   映像基址:                     0x00400000  
   内存区段对齐单位:             0x00001000  
   文件区段对齐单位:             0x00001000  
   所需操作系统主版本号:         0x0004  
   所需操作系统副版本号:         0x0000  -> 4.00  
   自定义主版本号:               0x0000  
   自定义副版本号:               0x0039  -> 0.57  
   所需子系统主版本号:           0x0004  
   所需子系统副版本号:           0x0000  -> 4.00  
   Win32 版本号值:                0x00000000  
   映像大小:                     0x00017000  
   文件头大小:                   0x00001000  
   CRC 校验和:                    0x00000000  
   子系统:                       0x0002  (WINDOWS\_GUI)  
   DLL 特征值:                    0x0000  
   栈初始保留区大小:             0x00100000  
   栈初始提交区大小:             0x00001000  
   堆初始保留区大小:             0x00100000  
   堆初始提交区大小:             0x00001000  
   加载标志:                     0x00000000  
   数据目录结构数组项数:         0x00000010  
  
   数据目录 (16)                 RVA        大小  
   -------------                 ---------- ----------  
   导出表                        0x00000000 0x00000000  
   导入表                        0x00016000 0x00000050  (".mackt")  
   资源                          0x0000A000 0x000009A4  (".rsrc")  
   例外表                        0x00000000 0x00000000  
   安全证书                      0x00000000 0x00000000  
   重定位表                      0x00000000 0x00000000  
   调试                          0x00000000 0x00000000  
   版权号                        0x00000000 0x00000000  
   全局指针                      0x00000000 0x00000000  
   TLS 表                        0x00000000 0x00000000  
   加载构造表                    0x00000000 0x00000000  
   绑定导入表                    0x00000000 0x00000000  
   IAT                           0x00000000 0x00000000  
   延迟导入表                    0x00000000 0x00000000  
   COM                           0x00000000 0x00000000  
   保留                          0x00000000 0x00000000  
  
  
  
2.分析  
  
从 GetOpenFileNameA 入手，很快得到：  
  
  
主线剧情  
  
  
mz      IMAGE\_DOS\_HEADER  <>  
pe      IMAGE\_NT\_HEADERS  <>  
  
call  CreateFileA, offset filename, GENERIC\_READ, FILE\_SHARE\_READ, NULL, OPEN\_EXISTING, NORMAL|SEQUENTIAL\_SCAN, NULL  
call  ReadFile, filehandle, offset mz, 40h, offset bytesread, NULL  
call  SetFilePointer, filehandle, mz.e\_lfanew, NULL, FILE\_BEGIN  
call  ReadFile, filehandle, offset pe, 0F8h, offset bytesread, NULL  
  
  
00401467    66:81BD EAF8FFF>CMP     WORD PTR \[EBP-716\], 5253//linker version  
00401470    74 1B           JE      SHORT 0040148D  
00401472    66:81BD F8FBFFF>CMP     WORD PTR \[EBP-408\], 5253  
0040147B    74 10           JE      SHORT 0040148D  
0040147D    68 20634000     PUSH    00406320                         ; ASCII "?- Not Armadillo protected",CR,LF  
00401482    E8 79FBFFFF     CALL    00401000  
00401487    59              POP     ECX  
00401488    E9 CD040000     JMP     0040195A  
  
call  CreateProcessA, offset filename, NULL, NULL, NULL, FALSE, DEBUG\_PROCESS|DEBUG\_ONLY\_THIS\_PROCESS, NULL, NULL, offset sbox, offset pbox  
call  WaitForDebugEvent, offset devent, 1000  
  
分支剧情  
  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\* 1 \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
  
0040188A    83BD 20FFFFFF 0>CMP     DWORD PTR \[EBP-E0\], 6                 ; LOAD\_DLL\_DEBUG\_EVENT  
00401891    75 2B           JNZ     SHORT 004018BE  
00401893    8B45 80         MOV     EAX, \[EBP-80\]                         ; ker  
00401896    3B85 30FFFFFF   CMP     EAX, \[EBP-D0\]                         ; ntd, wait kernel32  
0040189C    75 20           JNZ     SHORT 004018BE  
  
0012F3A0   004018B0  /CALL to WriteProcessMemory from dumped\_.004018AE  
0012F3A4   00000108  |hProcess = 00000108 (window)  
0012F3A8   7C812E03  |Address = 7C812E03  
0012F3AC   0012FA88  |Buffer = 0012FA88  
0012F3B0   0000000E  |BytesToWrite = E (14.)  
0012F3B4   00000000  \\pBytesWritten = NULL  
  
004018B0    8D45 FC         LEA     EAX, \[EBP-4\]  
004018B3    50              PUSH    EAX  
004018B4    FF75 90         PUSH    DWORD PTR \[EBP-70\]                    ; OpenMutexA  
004018B7    E8 31F8FFFF     CALL    <save\_set\_CC>  
  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\* 2 \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
  
004014EA    3985 20FFFFFF   CMP     \[EBP-E0\], EAX                         ; EXCEPTION\_DEBUG\_EVENT  
004014F0    0F85 94030000   JNZ     0040188A  
004014F6    81BD 2CFFFFFF 0>CMP     DWORD PTR \[EBP-D4\], 80000003          ; EXCEPTION\_BREAKPOINT  
00401500    0F85 B8030000   JNZ     004018BE  
00401506    8B8D 38FFFFFF   MOV     ECX, \[EBP-C8\]                         ; context.Eip  
0040150C    394D 90         CMP     \[EBP-70\], ECX                         ; is OpenMutexA?  
0040150F    0F85 FD000000   JNZ     00401612                              ; no  
00401515    53              PUSH    EBX  
  
  
主线剧情  
  
0012F3A0   00401526  /CALL to WriteProcessMemory from dumped\_.00401524//fix OpenMutexA  
0012F3A4   00000108  |hProcess = 00000108 (window)  
0012F3A8   7C80EC1B  |Address = 7C80EC1B  
0012F3AC   0012FAF0  |Buffer = 0012FAF0  
0012F3B0   00000001  |BytesToWrite = 1  
0012F3B4   00000000  \\pBytesWritten = NULL  
  
0012F3AC   00401543  /CALL to GetThreadContext from dumped\_.0040153D  
0012F3B0   00000114  |hThread = 00000114  
0012F3B4   0012F748  \\pContext = 0012F748  
  
00401543    FF8D 0CFDFFFF   DEC     DWORD PTR \[EBP-2F4\]                   ; eip--  
  
0012F3AC   0040155C  /CALL to SetThreadContext from dumped\_.00401556//restore EIP  
0012F3B0   00000114  |hThread = 00000114  
0012F3B4   0012F748  \\pContext = 0012F748  
  
0012F3A0   00401571  /CALL to ReadProcessMemory from dumped\_.0040156F//read Contex.Eip  
0012F3A4   00000108  |hProcess = 00000108 (window)  
0012F3A8   0012F56C  |pBaseAddress = 12F56C  
0012F3AC   0012FAD8  |Buffer = 0012FAD8  
0012F3B0   00000004  |BytesToRead = 4  
0012F3B4   00000000  \\pBytesRead = NULL  
  
/\*  
  
in debuggy  
  
00821C99    8D95 D4FEFFFF   LEA     EDX, \[EBP-12C\]  
00821C9F    52              PUSH    EDX  
00821CA0    6A 00           PUSH    0  
00821CA2    68 01001F00     PUSH    1F0001  
00821CA7    FF15 58C08400   CALL    \[<&KERNEL32.OpenMutexA>\]         ; kernel32.OpenMutexA  
00821CAD    85C0            TEST    EAX, EAX  
00821CAF    74 04           JE      SHORT 00821CB5  
00821CB1    C645 D8 00      MOV     BYTE PTR \[EBP-28\], 0  
00821CB5    8B45 D8         MOV     EAX, \[EBP-28\]  
00821CB8    25 FF000000     AND     EAX, 0FF  
00821CBD    85C0            TEST    EAX, EAX  
00821CBF    0F84 97010000   JE      00821E5C  
00821CC5    8B0D DCC28400   MOV     ECX, \[84C2DC\]//\[821CC7\] --> this dword 84C2DC(protection flags)  
00821CCB    330D F0C28400   XOR     ECX, \[84C2F0\]//key1  
00821CD1    330D A8C28400   XOR     ECX, \[84C2A8\]//key2  
00821CD7    83E1 20         AND     ECX, 20  
00821CDA    85C9            TEST    ECX, ECX  
00821CDC    0F84 7A010000   JE      00821E5C  
  
\*/  
  
0012F3A0   00401587  /CALL to ReadProcessMemory from dumped\_.00401585  
0012F3A4   00000108  |hProcess = 00000108 (window)  
0012F3A8   00821CC7  |pBaseAddress = 821CC7/////////when OpenMutexA, \[esp\]+1A=821cc7  
0012F3AC   0012FAE4  |Buffer = 0012FAE4/////////////////////////ptr to ProtectionFlags  
0012F3B0   00000004  |BytesToRead = 4  
0012F3B4   00000000  \\pBytesRead = NULL  
0012F3A0   00401599  /CALL to ReadProcessMemory from dumped\_.00401597//read flag  
0012F3A4   00000108  |hProcess = 00000108 (window)  
0012F3A8   0084C2DC  |pBaseAddress = 84C2DC  
0012F3AC   0012FAE0  |Buffer = 0012FAE0  
0012F3B0   00000004  |BytesToRead = 4  
0012F3B4   00000000  \\pBytesRead = NULL  
  
  
0012F3A0   004015AF  /CALL to ReadProcessMemory from dumped\_.004015AD  
0012F3A4   00000108  |hProcess = 00000108 (window)  
0012F3A8   00821CCD  |pBaseAddress = 821CCD//\[esp\]+20h ------->get key 1  
0012F3AC   0012FAE4  |Buffer = 0012FAE4  
0012F3B0   00000004  |BytesToRead = 4  
0012F3B4   00000000  \\pBytesRead = NULL  
  
\*好像漏了一个，无所谓  
  
0012F3A0   004015DD  /CALL to ReadProcessMemory from dumped\_.004015DB  
0012F3A4   00000108  |hProcess = 00000108 (window)  
0012F3A8   00821CD3  |pBaseAddress = 821CD3//\[esp\]+26h -------->key 2  
0012F3AC   0012FAE4  |Buffer = 0012FAE4  
0012F3B0   00000004  |BytesToRead = 4  
0012F3B4   00000000  \\pBytesRead = NULL  
0012F3A0   004015EF  /CALL to ReadProcessMemory from dumped\_.004015ED  
0012F3A4   00000108  |hProcess = 00000108 (window)  
0012F3A8   0084C2A8  |pBaseAddress = 84C2A8  
0012F3AC   0012FADC  |Buffer = 0012FADC  
0012F3B0   00000004  |BytesToRead = 4  
0012F3B4   00000000  \\pBytesRead = NULL  
  
004015EF    8B45 EC         MOV     EAX, \[EBP-14\]//eax = flag xor key1  
004015F2    3345 E8         XOR     EAX, \[EBP-18\]//xor key2  
  
x 高潮  
  
  
00401177    55              PUSH    EBP  
00401178    8BEC            MOV     EBP, ESP  
0040117A    53              PUSH    EBX  
0040117B    56              PUSH    ESI  
0040117C    57              PUSH    EDI  
0040117D    68 3C624000     PUSH    0040623C                              ; ASCII CR,LF,"<Protection Options>",CR,LF  
00401182    E8 79FEFFFF     CALL    00401000  
00401187    59              POP     ECX  
00401188    8B45 08         MOV     EAX, \[EBP+8\]  
0040118B    83E0 10         AND     EAX, 10  
0040118E    75 0D           JNZ     SHORT 0040119D  
00401190    68 10624000     PUSH    00406210                              ; ASCII "Standard protection or Minimum protection",CR,LF  
00401195    E8 66FEFFFF     CALL    00401000  
0040119A    59              POP     ECX  
0040119B    EB 0B           JMP     SHORT 004011A8  
0040119D    68 00624000     PUSH    00406200                              ; ASCII "Debug-Blocker",CR,LF  
004011A2    E8 59FEFFFF     CALL    00401000  
004011A7    59              POP     ECX  
004011A8    8B45 08         MOV     EAX, \[EBP+8\]  
004011AB    25 000000FF     AND     EAX, FF000000  
004011B0    74 0B           JE      SHORT 004011BD  
004011B2    68 F0614000     PUSH    004061F0                              ; ASCII "CopyMem-II",CR,LF  
004011B7    E8 44FEFFFF     CALL    00401000  
004011BC    59              POP     ECX  
004011BD    8B45 08         MOV     EAX, \[EBP+8\]  
004011C0    25 00004000     AND     EAX, 400000  
004011C5    74 0B           JE      SHORT 004011D2  
004011C7    68 CC614000     PUSH    004061CC                              ; ASCII "Enable Import Table Elimination",CR,LF  
004011CC    E8 2FFEFFFF     CALL    00401000  
004011D1    59              POP     ECX  
004011D2    8B45 08         MOV     EAX, \[EBP+8\]  
004011D5    25 00002000     AND     EAX, 200000  
004011DA    74 0B           JE      SHORT 004011E7  
004011DC    68 A8614000     PUSH    004061A8                              ; ASCII "Enable Strategic Code Splicing",CR,LF  
004011E1    E8 1AFEFFFF     CALL    00401000  
004011E6    59              POP     ECX  
004011E7    8B45 08         MOV     EAX, \[EBP+8\]  
004011EA    25 00020000     AND     EAX, 200  
004011EF    74 0B           JE      SHORT 004011FC  
004011F1    68 88614000     PUSH    00406188                              ; ASCII "Enable Nanomites Processing",CR,LF  
004011F6    E8 05FEFFFF     CALL    00401000  
004011FB    59              POP     ECX  
004011FC    8B45 08         MOV     EAX, \[EBP+8\]  
004011FF    83E0 08         AND     EAX, 8  
00401202    74 0B           JE      SHORT 0040120F  
00401204    68 60614000     PUSH    00406160                              ; ASCII "Enable Memory-Patching Protections",CR,LF  
00401209    E8 F2FDFFFF     CALL    00401000  
0040120E    59              POP     ECX  
0040120F    68 44614000     PUSH    00406144                              ; ASCII CR,LF,"<Backup Key Options>",CR,LF  
00401214    E8 E7FDFFFF     CALL    00401000  
00401219    59              POP     ECX  
0040121A    8B45 08         MOV     EAX, \[EBP+8\]  
0040121D    25 00C00000     AND     EAX, 0C000  
00401222    75 22           JNZ     SHORT 00401246  
00401224    8B45 08         MOV     EAX, \[EBP+8\]  
00401227    25 00000100     AND     EAX, 10000  
0040122C    75 0D           JNZ     SHORT 0040123B  
0040122E    68 28614000     PUSH    00406128                              ; ASCII "No Registry Keys at All",CR,LF  
00401233    E8 C8FDFFFF     CALL    00401000  
00401238    59              POP     ECX  
00401239    EB 0B           JMP     SHORT 00401246  
0040123B    68 08614000     PUSH    00406108                              ; ASCII "Main Key Only, No Backup Keys",CR,LF  
00401240    E8 BBFDFFFF     CALL    00401000  
00401245    59              POP     ECX  
00401246    8B45 08         MOV     EAX, \[EBP+8\]  
00401249    25 00800000     AND     EAX, 8000  
0040124E    74 0B           JE      SHORT 0040125B  
00401250    68 F0604000     PUSH    004060F0                              ; ASCII "Variable Backup Keys",CR,LF  
00401255    E8 A6FDFFFF     CALL    00401000  
0040125A    59              POP     ECX  
0040125B    8B45 08         MOV     EAX, \[EBP+8\]  
0040125E    25 00400000     AND     EAX, 4000  
00401263    74 0B           JE      SHORT 00401270  
00401265    68 DC604000     PUSH    004060DC                              ; ASCII "Fixed Backup Keys",CR,LF  
0040126A    E8 91FDFFFF     CALL    00401000  
0040126F    59              POP     ECX  
00401270    68 C0604000     PUSH    004060C0                              ; ASCII CR,LF,"<Compression Options>",CR,LF  
00401275    E8 86FDFFFF     CALL    00401000  
0040127A    59              POP     ECX  
0040127B    8B45 08         MOV     EAX, \[EBP+8\]  
0040127E    83E0 03         AND     EAX, 3  
00401281    75 0B           JNZ     SHORT 0040128E  
00401283    68 A0604000     PUSH    004060A0                              ; ASCII "Minimal/Fastest Compression ",CR,LF  
00401288    E8 73FDFFFF     CALL    00401000  
0040128D    59              POP     ECX  
0040128E    8B45 08         MOV     EAX, \[EBP+8\]  
00401291    83E0 01         AND     EAX, 1  
00401294    74 0B           JE      SHORT 004012A1  
00401296    68 84604000     PUSH    00406084                              ; ASCII "Better/Slower Compression",CR,LF  
0040129B    E8 60FDFFFF     CALL    00401000  
004012A0    59              POP     ECX  
004012A1    8B45 08         MOV     EAX, \[EBP+8\]  
004012A4    83E0 02         AND     EAX, 2  
004012A7    74 0B           JE      SHORT 004012B4  
004012A9    68 68604000     PUSH    00406068                              ; ASCII "Best/Slowest Compression",CR,LF  
004012AE    E8 4DFDFFFF     CALL    00401000  
004012B3    59              POP     ECX  
004012B4    5F              POP     EDI  
004012B5    5E              POP     ESI  
004012B6    5B              POP     EBX  
004012B7    5D              POP     EBP  
004012B8    C3              RETN  
  
004015FE    8D45 FF         LEA     EAX, \[EBP-1\]//restore CC  
00401601    50              PUSH    EAX  
00401602    FF75 B8         PUSH    DWORD PTR \[EBP-48\]  
00401605    E8 E3FAFFFF     CALL    <save\_set\_CC>  
  
0040160A    83C4 0C         ADD     ESP, 0C  
0040160D    E9 6F020000     JMP     00401881  
  
00401881    C745 88 0200010>MOV     DWORD PTR \[EBP-78\], 10002             ; UNICODE "::=::\\"  
00401888    EB 34           JMP     SHORT 004018BE  
  
后面就没什么了。  
  
by forgot/iPB  
06.05.02 

 4dd466d0e1e0cc123afbe504da2462ca