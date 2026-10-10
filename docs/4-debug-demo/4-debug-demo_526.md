

# 浅谈 cool edit pro2.1 脱壳后解决自校验

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-579.htm`](https://www.pediy.com/kssd/pediy07/pediy7-579.htm)

声音编辑软件 cool edit Pro2.10 脱壳后自效验的解决：  
用 UPX Shell 脱壳后出现错误提示"Internal Error (13)" 然后就死在：  
  
005AA600  /$  8B4424 04     MOV EAX,DWORD PTR SS:\[ESP+4\]  
005AA604  |.  53            PUSH EBX  
005AA605  |.  55            PUSH EBP  
005AA606  |.  56            PUSH ESI  
005AA607  |.  57            PUSH EDI  
005AA608  |.  6A EB         PUSH -15                                 ; /Index = GWL\_USERDATA  
005AA60A  |.  50            PUSH EAX                                 ; |hWnd  
005AA60B  |.  FF15 38166900 CALL DWORD PTR DS:\[<&USER32.GetWindowLon>; \\GetWindowLongA  
005AA611  |.  8BF8          MOV EDI,EAX  
005AA613  |.  8B5C24 24     MOV EBX,DWORD PTR SS:\[ESP+24\]  
005AA617  |.  8B47 10       MOV EAX,DWORD PTR DS:\[EDI+10\]      ；无效的值  
  
DS:\[00000010\]=???  
EAX=00000000  
  
修改了原程序时间和原程序名，运行正常，说明是对主程序的 CRC 效验。  
  
但是这个程序的效验是基于消息参数的验证，不容易跟踪到验证的地方：  
  
005F0BFF   .  FFD6          CALL ESI                                 ; \\GetWindowLongA  
005F0C01   .  8BF8          MOV EDI,EAX  
005F0C03   .  A1 D04A6E00   MOV EAX,DWORD PTR DS:\[6E4AD0\]  
005F0C08   .  8B9C24 080300>MOV EBX,DWORD PTR SS:\[ESP+308\]  
005F0C0F   .  3BD8          CMP EBX,EAX  
  
看看这里的 EBX 的值和没有脱壳的 EBX 值的比较 log EBX：  
  
未脱壳时：  
  
005F0C0F  COND: 00000024  
005F0C0F  COND: 00000081  
005F0C0F  COND: 00000083  
005F0C0F  COND: 00000001  
77BA0000  模块 C:\\WINDOWS\\System32\\midimap.dll  
005F0C0F  COND: 0000000D  
005F0C0F  COND: 00000210  
005F0C0F  COND: 0000001F  
005F0C0F  COND: 0000000A  
005F0C0F  COND: 0000001C  
005F0C0F  COND: 00002A71                 //第 10 次的值  
005F0C0F  COND: 00000113  
005F0C0F  COND: 00000113  
005F0C0F  COND: 00000113  
005F0C0F  COND: 00000113  
005F0C0F  COND: 00000113  
005F0C0F  COND: 00000113  
60800000  模块 C:\\WINDOWS\\System32\\mslbui.dll  
005F0C0F  COND: 00000113  
005F0C0F  COND: 0000001C  
005F0C0F  COND: 0000000A  
005F0C0F  COND: 00000055  
005F0C0F  COND: 00000129  
005F0C0F  COND: 00000210  
77E5D295  ID 00000834 的新线程已经创建    //程序运行了  
  
  
脱壳后：  
  
005F0C0F  COND: 00000024  
005F0C0F  COND: 00000081  
005F0C0F  COND: 00000083  
005F0C0F  COND: 00000001  
77BA0000  模块 C:\\WINDOWS\\System32\\midimap.dll  
005F0C0F  COND: 0000000D  
005F0C0F  COND: 00000210  
005F0C0F  COND: 0000001F  
005F0C0F  COND: 0000000A  
005F0C0F  COND: 0000001C  
005F0C0F  COND: 00000010                   //看看第 10 次的值不同了  
005F0C0F  COND: 00000210                
10000000  模块 D:\\Program Files\\sina\\UC\\UCIdleHook.dll  
005F0C0F  COND: 00000113  
005F0C0F  COND: 00000113  
005F0C0F  COND: 00000113  
005F0C0F  COND: 0000001C  
005F0C0F  COND: 00000002  
005F0C0F  COND: 00000082  
005AA617  访问违反： 读取 \[00000010\]       //程序出错  
00400000  卸载 C:\\Program Files\\coolpro2\\coolpro20.exe    
  
原想跟踪这个值，看看什么地方验证的，一直找不到，花了不少的时间 :(  
  
后来考虑，既然是 CRC 验证一般会使用这个 CreateFileA 函数，试试：  
  
  
bp CreateFileA  几次后来到：  
  
00492570  /$  55            PUSH EBP  
00492571  |.  8BEC          MOV EBP,ESP  
00492573  |.  81EC 24010000 SUB ESP,124  
00492579  |.  53            PUSH EBX  
0049257A  |.  8B1D F8456E00 MOV EBX,DWORD PTR DS:\[6E45F8\]  
00492580  |.  57            PUSH EDI  
00492581  |.  8D85 DCFEFFFF LEA EAX,DWORD PTR SS:\[EBP-124\]  
00492587  |.  68 04010000   PUSH 104                                 ; /BufSize = 104 (260.)  
0049258C  |.  50            PUSH EAX                                 ; |PathBuffer  
0049258D  |.  6A 00         PUSH 0                                   ; |hModule = NULL  
0049258F  |.  895D EC       MOV DWORD PTR SS:\[EBP-14\],EBX            ; |  
00492592  |.  FF15 C4126900 CALL DWORD PTR DS:\[<&KERNEL32.GetModuleF>; \\GetModuleFileNameA  
00492598  |.  85C0          TEST EAX,EAX  
0049259A  |.  0F84 74010000 JE coolpro2.00492714  
004925A0  |.  6A 00         PUSH 0                                   ; /hTemplateFile = NULL  
004925A2  |.  68 80000008   PUSH 8000080                             ; |Attributes = NORMAL|SEQUENTIAL\_SCAN  
004925A7  |.  6A 03         PUSH 3                                   ; |Mode = OPEN\_EXISTING  
004925A9  |.  6A 00         PUSH 0                                   ; |pSecurity = NULL  
004925AB  |.  6A 01         PUSH 1                                   ; |ShareMode = FILE\_SHARE\_READ  
004925AD  |.  8D8D DCFEFFFF LEA ECX,DWORD PTR SS:\[EBP-124\]           ; |  
004925B3  |.  68 00000080   PUSH 80000000                            ; |Access = GENERIC\_READ  
004925B8  |.  51            PUSH ECX                                 ; |FileName  
004925B9  |.  FF15 10136900 CALL DWORD PTR DS:\[<&KERNEL32.CreateFile>; \\CreateFileA  
004925BF  |.  8BF8          MOV EDI,EAX  
004925C1  |.  83FF FF       CMP EDI,-1  
004925C4  |.  897D E8       MOV DWORD PTR SS:\[EBP-18\],EDI  
004925C7  |.  0F84 47010000 JE coolpro2.00492714  
004925CD  |.  56            PUSH ESI  
004925CE  |.  6A 00         PUSH 0                                   ; /pFileSizeHigh = NULL  
004925D0  |.  57            PUSH EDI                                 ; |hFile  
004925D1  |.  FF15 14136900 CALL DWORD PTR DS:\[<&KERNEL32.GetFileSiz>; \\GetFileSize  
004925D7  |.  8BF0          MOV ESI,EAX  
004925D9  |.  8975 F8       MOV DWORD PTR SS:\[EBP-8\],ESI  
004925DC  |.  FF15 9C126900 CALL DWORD PTR DS:\[<&KERNEL32.GetProcess>; \[GetProcessHeap  
004925E2  |.  56            PUSH ESI                                 ; /HeapSize  
004925E3  |.  6A 08         PUSH 8                                   ; |Flags = HEAP\_ZERO\_MEMORY  
004925E5  |.  50            PUSH EAX                                 ; |hHeap  
004925E6  |.  8945 E4       MOV DWORD PTR SS:\[EBP-1C\],EAX            ; |  
004925E9  |.  FF15 94126900 CALL DWORD PTR DS:\[<&KERNEL32.HeapAlloc>>; \\HeapAlloc  
004925EF  |.  8BF0          MOV ESI,EAX  
004925F1  |.  85F6          TEST ESI,ESI  
004925F3  |.  8975 E0       MOV DWORD PTR SS:\[EBP-20\],ESI  
004925F6  |.  0F84 10010000 JE coolpro2.0049270C  
004925FC  |.  8B45 F8       MOV EAX,DWORD PTR SS:\[EBP-8\]  
004925FF  |.  8D55 F4       LEA EDX,DWORD PTR SS:\[EBP-C\]  
00492602  |.  6A 00         PUSH 0                                   ; /pOverlapped = NULL  
00492604  |.  52            PUSH EDX                                 ; |pBytesRead  
00492605  |.  50            PUSH EAX                                 ; |BytesToRead  
00492606  |.  56            PUSH ESI                                 ; |Buffer  
00492607  |.  57            PUSH EDI                                 ; |hFile  
00492608  |.  FF15 4C136900 CALL DWORD PTR DS:\[<&KERNEL32.ReadFile>\] ; \\ReadFile  
0049260E  |.  85C0          TEST EAX,EAX  
00492610  |.  0F84 E9000000 JE coolpro2.004926FF  
00492616  |.  C745 FC 01000>MOV DWORD PTR SS:\[EBP-4\],1  
0049261D  |.  BB 03000000   MOV EBX,3  
00492622  |.  C745 F0 00000>MOV DWORD PTR SS:\[EBP-10\],0  
00492629  |.  C745 F8 06000>MOV DWORD PTR SS:\[EBP-8\],6  
00492630  |.  B9 02000000   MOV ECX,2  
00492635  |>  66:0FB63E     /MOVZX DI,BYTE PTR DS:\[ESI\]  
00492639  |.  8B55 F0       |MOV EDX,DWORD PTR SS:\[EBP-10\]  
0049263C  |.  46            |INC ESI  
0049263D  |.  8D0417        |LEA EAX,DWORD PTR DS:\[EDI+EDX\]  
00492640  |.  33D2          |XOR EDX,EDX  
00492642  |.  8A55 FD       |MOV DL,BYTE PTR SS:\[EBP-3\]  
00492645  |.  8D04B8        |LEA EAX,DWORD PTR DS:\[EAX+EDI\*4\]  
00492648  |.  8B7D F8       |MOV EDI,DWORD PTR SS:\[EBP-8\]  
0049264B  |.  03D0          |ADD EDX,EAX  
0049264D  |.  03D7          |ADD EDX,EDI  
0049264F  |.  03D3          |ADD EDX,EBX  
00492651  |.  33D1          |XOR EDX,ECX  
00492653  |.  8BC2          |MOV EAX,EDX  
00492655  |.  8B55 FC       |MOV EDX,DWORD PTR SS:\[EBP-4\]  
00492658  |.  F6C6 80       |TEST DH,80  
0049265B  |.  74 02         |JE SHORT coolpro2.0049265F  
0049265D  |.  0C 01         |OR AL,1  
0049265F  |>  8945 FC       |MOV DWORD PTR SS:\[EBP-4\],EAX  
00492662  |.  8B45 F4       |MOV EAX,DWORD PTR SS:\[EBP-C\]  
00492665  |.  48            |DEC EAX  
00492666  |.  8945 F4       |MOV DWORD PTR SS:\[EBP-C\],EAX  
00492669  |.  74 70         |JE SHORT coolpro2.004926DB  
0049266B  |.  33C0          |XOR EAX,EAX  
0049266D  |.  66:8B45 FC    |MOV AX,WORD PTR SS:\[EBP-4\]  
00492671  |.  BA F1F0F0F0   |MOV EDX,F0F0F0F1  
00492676  |.  F7E2          |MUL EDX  
00492678  |.  C1EA 04       |SHR EDX,4  
0049267B  |.  8955 F0       |MOV DWORD PTR SS:\[EBP-10\],EDX  
0049267E  |.  8B4D FC       |MOV ECX,DWORD PTR SS:\[EBP-4\]  
00492681  |.  66:0FB606     |MOVZX AX,BYTE PTR DS:\[ESI\]  
00492685  |.  81E1 FFFF0000 |AND ECX,0FFFF  
0049268B  |.  D1E0          |SHL EAX,1  
0049268D  |.  8D14CD 000000>|LEA EDX,DWORD PTR DS:\[ECX\*8\]  
00492694  |.  25 FFFF0000   |AND EAX,0FFFF  
00492699  |.  2BD1          |SUB EDX,ECX  
0049269B  |.  8B4D F0       |MOV ECX,DWORD PTR SS:\[EBP-10\]  
0049269E  |.  03D0          |ADD EDX,EAX  
004926A0  |.  33C0          |XOR EAX,EAX  
004926A2  |.  D1FA          |SAR EDX,1  
004926A4  |.  8AC7          |MOV AL,BH  
004926A6  |.  03CA          |ADD ECX,EDX  
004926A8  |.  8B55 F8       |MOV EDX,DWORD PTR SS:\[EBP-8\]  
004926AB  |.  03C1          |ADD EAX,ECX  
004926AD  |.  46            |INC ESI  
004926AE  |.  33C2          |XOR EAX,EDX  
004926B0  |.  F6C7 80       |TEST BH,80  
004926B3  |.  74 02         |JE SHORT coolpro2.004926B7  
004926B5  |.  0C 01         |OR AL,1  
004926B7  |>  8BD8          |MOV EBX,EAX  
004926B9  |.  895D F8       |MOV DWORD PTR SS:\[EBP-8\],EBX  
004926BC  |.  33C0          |XOR EAX,EAX  
004926BE  |.  66:8B45 F8    |MOV AX,WORD PTR SS:\[EBP-8\]  
004926C2  |.  BA 9ED8899D   |MOV EDX,9D89D89E  
004926C7  |.  F7E2          |MUL EDX  
004926C9  |.  C1EA 03       |SHR EDX,3  
004926CC  |.  8955 F0       |MOV DWORD PTR SS:\[EBP-10\],EDX  
004926CF  |.  8B45 FC       |MOV EAX,DWORD PTR SS:\[EBP-4\]  
004926D2  |.  8D141B        |LEA EDX,DWORD PTR DS:\[EBX+EBX\]  
004926D5  |.  8955 F8       |MOV DWORD PTR SS:\[EBP-8\],EDX  
004926D8  |.  8D0C00        |LEA ECX,DWORD PTR DS:\[EAX+EAX\]  
004926DB  |>  8B45 F4       |MOV EAX,DWORD PTR SS:\[EBP-C\]  
004926DE  |.  48            |DEC EAX  
004926DF  |.  8945 F4       |MOV DWORD PTR SS:\[EBP-C\],EAX  
004926E2  |.^ 0F85 4DFFFFFF \\JNZ coolpro2.00492635  
004926E8  |.  8B45 EC       MOV EAX,DWORD PTR SS:\[EBP-14\]  
004926EB  |.  33C9          XOR ECX,ECX  
004926ED  |.  8B75 E0       MOV ESI,DWORD PTR SS:\[EBP-20\]  
004926F0  |.  8B7D E8       MOV EDI,DWORD PTR SS:\[EBP-18\]  
004926F3  |.  8AEB          MOV CH,BL  
004926F5  |.  8A4D FC       MOV CL,BYTE PTR SS:\[EBP-4\]  
004926F8  |.  03C1          ADD EAX,ECX  
004926FA  |.  8945 EC       MOV DWORD PTR SS:\[EBP-14\],EAX  
004926FD  |.  8BD8          MOV EBX,EAX  
004926FF  |>  8B55 E4       MOV EDX,DWORD PTR SS:\[EBP-1C\]  
00492702  |.  56            PUSH ESI                                 ; /pMemory  
00492703  |.  6A 00         PUSH 0                                   ; |Flags = 0  
00492705  |.  52            PUSH EDX                                 ; |hHeap  
00492706  |.  FF15 A4126900 CALL DWORD PTR DS:\[<&KERNEL32.HeapFree>\] ; \\HeapFree  
0049270C  |>  57            PUSH EDI                                 ; /hObject  
0049270D  |.  FF15 08126900 CALL DWORD PTR DS:\[<&KERNEL32.CloseHandl>; \\CloseHandle  
00492713  |.  5E            POP ESI  
00492714  |>  A1 F0456E00   MOV EAX,DWORD PTR DS:\[6E45F0\]  
00492719  |.  891D 8C4C6E00 MOV DWORD PTR DS:\[6E4C8C\],EBX  
0049271F  |.  53            PUSH EBX                                 ; /lParam  
00492720  |.  2BD8          SUB EBX,EAX                              ; |  
00492722  |.  8B45 08       MOV EAX,DWORD PTR SS:\[EBP+8\]             ; |  
00492725  |.  83E3 07       AND EBX,7                                ; |  
00492728  |.  81C3 592A0000 ADD EBX,2A59                             ; |  
0049272E  |.  6A 00         PUSH 0                                   ; |wParam = 0  
00492730  |.  53            PUSH EBX                                 ; |Message  
00492731  |.  50            PUSH EAX                                 ; |hWnd  
00492732  |.  FF15 EC156900 CALL DWORD PTR DS:\[<&USER32.SendMessageA>; \\SendMessageA  
00492738  |.  8B0D 24516B00 MOV ECX,DWORD PTR DS:\[6B5124\]  
0049273E  |.  8B15 60496E00 MOV EDX,DWORD PTR DS:\[6E4960\]  
00492744  |.  6A 00         PUSH 0                                   ; /lParam = 0  
00492746  |.  81E1 FFFF0000 AND ECX,0FFFF                            ; |  
0049274C  |.  6A 00         PUSH 0                                   ; |wParam = 0  
0049274E  |.  51            PUSH ECX                                 ; |Message  
0049274F  |.  52            PUSH EDX                                 ; |hWnd => 1C02CA  
00492750  |.  FF15 D4156900 CALL DWORD PTR DS:\[<&USER32.PostMessageA>; \\PostMessageA  
00492756  |.  5F            POP EDI  
00492757  |.  B8 01000000   MOV EAX,1  
0049275C  |.  5B            POP EBX  
0049275D  |.  8BE5          MOV ESP,EBP  
0049275F  |.  5D            POP EBP  
00492760  \\.  C2 0400       RETN 4  
  
  
这是个明显的对程序的二进制验证：  
  
004925B9  |.  FF15 10136900 CALL DWORD PTR DS:\[<&KERNEL32.CreateFile>; \\CreateFileA  
  
0012EAB0   004925BF  /CALL 到 CreateFileA 来自 coolpro2.004925B9  
0012EAB4   0012EAD8  |FileName = "C:\\Program Files\\coolpro2\\coolpro20.exe"  
0012EAB8   80000000  |Access = GENERIC\_READ  
0012EABC   00000001  |ShareMode = FILE\_SHARE\_READ  
0012EAC0   00000000  |pSecurity = NULL  
0012EAC4   00000003  |Mode = OPEN\_EXISTING  
0012EAC8   08000080  |Attributes = NORMAL|SEQUENTIAL\_SCAN  
0012EACC   00000000  \\hTemplateFile = NULL  
  
获得 Hfile 句柄  
  
004925D1  |.  FF15 14136900 CALL DWORD PTR DS:\[<&KERNEL32.GetFileSiz>; \\GetFileSize  
  
0012EAC0   004925D7  /CALL 到 GetFileSize 来自 coolpro2.004925D1  
0012EAC4   00000138  |hFile = 00000138 (window)  
0012EAC8   00000000  \\pFileSizeHigh = NULL  
  
获得文件的 size  
  
004925E9  |.  FF15 94126900 CALL DWORD PTR DS:\[<&KERNEL32.HeapAlloc>>; \\HeapAlloc  
  
申请一个 size 大小的临时空间，存放程序数据  
  
  
00492608  |.  FF15 4C136900 CALL DWORD PTR DS:\[<&KERNEL32.ReadFile>\] ; \\ReadFile  
  
0012EAB4   0049260E  /CALL 到 ReadFile 来自 coolpro2.00492608  
0012EAB8   00000138  |hFile = 00000138 (window)  
0012EABC   01A90020  |Buffer = 01A90020  
0012EAC0   00903010  |BytesToRead = 903010 (9449488.)  
0012EAC4   0012EBF0  |pBytesRead = 0012EBF0  
0012EAC8   00000000  \\pOverlapped = NULL  
  
把程序数据全部读入 Buffer，以便后面验证：  
  
主验证数据获得的代码：  
  
00492635  /MOVZX DI,BYTE PTR DS:\[ESI\]    
  
DS:\[01A90020\]=4D ('M')    
DI=0138  
Jump from 004926E2  
  
//从第一个字节开始  
         
00492639  |MOV EDX,DWORD PTR SS:\[EBP-10\]  
0049263C  |INC ESI  
0049263D  |LEA EAX,DWORD PTR DS:\[EDI+EDX\]  
00492640  |XOR EDX,EDX  
00492642  |MOV DL,BYTE PTR SS:\[EBP-3\]  
00492645  |LEA EAX,DWORD PTR DS:\[EAX+EDI\*4\]  
00492648  |MOV EDI,DWORD PTR SS:\[EBP-8\]  
0049264B  |ADD EDX,EAX  
0049264D  |ADD EDX,EDI  
0049264F  |ADD EDX,EBX  
00492651  |XOR EDX,ECX  
00492653  |MOV EAX,EDX  
00492655  |MOV EDX,DWORD PTR SS:\[EBP-4\]  
00492658  |TEST DH,80  
0049265B  |JE SHORT coolpro2.0049265F  
0049265D  |OR AL,1  
0049265F  |MOV DWORD PTR SS:\[EBP-4\],EAX  
00492662  |MOV EAX,DWORD PTR SS:\[EBP-C\]  
00492665  |DEC EAX  
00492666  |MOV DWORD PTR SS:\[EBP-C\],EAX  
00492669  |JE SHORT coolpro2.004926DB  
0049266B  |XOR EAX,EAX  
0049266D  |MOV AX,WORD PTR SS:\[EBP-4\]  
00492671  |MOV EDX,F0F0F0F1  
00492676  |MUL EDX  
00492678  |SHR EDX,4  
0049267B  |MOV DWORD PTR SS:\[EBP-10\],EDX  
0049267E  |MOV ECX,DWORD PTR SS:\[EBP-4\]  
00492681  |MOVZX AX,BYTE PTR DS:\[ESI\]  
00492685  |AND ECX,0FFFF  
0049268B  |SHL EAX,1  
0049268D  |LEA EDX,DWORD PTR DS:\[ECX\*8\]  
00492694  |AND EAX,0FFFF  
00492699  |SUB EDX,ECX  
0049269B  |MOV ECX,DWORD PTR SS:\[EBP-10\]  
0049269E  |ADD EDX,EAX  
004926A0  |XOR EAX,EAX  
004926A2  |SAR EDX,1  
004926A4  |MOV AL,BH  
004926A6  |ADD ECX,EDX  
004926A8  |MOV EDX,DWORD PTR SS:\[EBP-8\]  
004926AB  |ADD EAX,ECX  
004926AD  |INC ESI  
004926AE  |XOR EAX,EDX  
004926B0  |TEST BH,80  
004926B3  |JE SHORT coolpro2.004926B7  
004926B5  |OR AL,1  
004926B7  |MOV EBX,EAX  
004926B9  |MOV DWORD PTR SS:\[EBP-8\],EBX  
004926BC  |XOR EAX,EAX  
004926BE  |MOV AX,WORD PTR SS:\[EBP-8\]  
004926C2  |MOV EDX,9D89D89E  
004926C7  |MUL EDX  
004926C9  |SHR EDX,3  
004926CC  |MOV DWORD PTR SS:\[EBP-10\],EDX  
004926CF  |MOV EAX,DWORD PTR SS:\[EBP-4\]  
004926D2  |LEA EDX,DWORD PTR DS:\[EBX+EBX\]  
004926D5  |MOV DWORD PTR SS:\[EBP-8\],EDX  
004926D8  |LEA ECX,DWORD PTR DS:\[EAX+EAX\]  
004926DB  |MOV EAX,DWORD PTR SS:\[EBP-C\]  
004926DE  |DEC EAX  
004926DF  |MOV DWORD PTR SS:\[EBP-C\],EAX  
004926E2  \\JNZ coolpro2.00492635  
004926E8  MOV EAX,DWORD PTR SS:\[EBP-14\]  
004926EB  XOR ECX,ECX  
004926ED  MOV ESI,DWORD PTR SS:\[EBP-20\]  
004926F0  MOV EDI,DWORD PTR SS:\[EBP-18\]  
004926F3  MOV CH,BL  
004926F5  MOV CL,BYTE PTR SS:\[EBP-4\]  
004926F8  ADD EAX,ECX  
004926FA  MOV DWORD PTR SS:\[EBP-14\],EAX  
004926FD  MOV EBX,EAX  
  
不管他的算法 最终得到的值有  
  
\[EBP-10\]  
\[EBP-8\]  
\[EBP-4\]  
  
和后面：004926F3  MOV CH,BL 要用的  
  
EBX  
  
现在跟踪原程序 得到这些值，然后修改这段代码，直接赋值给需要的地址  
修改后的代码是：  
  
004926B7  |> \\33C0          XOR EAX,EAX  
004926B9  |.  B9 CA8FFEB0   MOV ECX,B0FE8FCA  
004926BE  |.  BA CE58437C   MOV EDX,7C4358CE  
004926C3  |.  BB 67AC21BE   MOV EBX,BE21AC67  
004926C8  |.  BE 30FEF301   MOV ESI,1F3FE30  
004926CD  |.  BF 8C5320BE   MOV EDI,BE20538C  
004926D2  |.  C745 F0 430D0>MOV DWORD PTR SS:\[EBP-10\],0D43  
004926D9  |.  90            NOP  
004926DA  |.  90            NOP  
004926DB  |>  90            NOP  
004926DC  |.  90            NOP  
004926DD  |.  90            NOP  
004926DE  |.  90            NOP  
004926DF  |.  90            NOP  
004926E0  |.  894D FC       MOV DWORD PTR SS:\[EBP-4\],ECX  
004926E3  |.  90            NOP  
004926E4  |.  8955 F8       MOV DWORD PTR SS:\[EBP-8\],EDX  
004926E7  |.  90            NOP  
004926E8  |.  8B45 EC       MOV EAX,DWORD PTR SS:\[EBP-14\]  
  
保存修改的程序，再次运行看看，正常了:)  
  
  
                                 by fxyang  
             
                                 2005.6.10

强  
我分析到这里，最后却没有还原完数据。功亏一篑  
  
.text:00492570       push ebp  
.text:00492571       mov ebp, esp  
.text:00492573       sub esp, 124h  
.text:00492579       push ebx  
.text:0049257A       mov ebx, dword\_6E45F8  
.text:00492580       push edi  
.text:00492581       lea eax, \[ebp+FileName\]  
.text:00492587       push 104h              ; nSize  
.text:0049258C       push eax               ; lpFilename  
.text:0049258D       push 0                 ; hModule  
.text:0049258F       mov \[ebp+var\_14\], ebx  
.text:00492592       call ds:GetModuleFileNameA  
.text:00492598       test eax, eax  
.text:0049259A       jz loc\_492714  
.text:004925A0       push 0                 ; hTemplateFile  
.text:004925A2       push 8000080h          ; dwFlagsAndAttributes  
.text:004925A7       push 3                 ; dwCreationDisposition  
.text:004925A9       push 0                 ; lpSecurityAttributes  
.text:004925AB       push 1                 ; dwShareMode  
.text:004925AD       lea ecx, \[ebp+FileName\]  
.text:004925B3       push 80000000h         ; dwDesiredAccess  
.text:004925B8       push ecx               ; lpFileName  
.text:004925B9       call ds:CreateFileA  
.text:004925BF       mov edi, eax  
.text:004925C1       cmp edi, 0FFFFFFFFh  
.text:004925C4       mov \[ebp+var\_18\], edi  
.text:004925C7       jz loc\_492714  
.text:004925CD       push esi  
.text:004925CE       push 0                 ; lpFileSizeHigh  
.text:004925D0       push edi               ; hFile  
.text:004925D1       call ds:GetFileSize  
//获得文件 Size  
.text:004925D7       mov esi, eax  
.text:004925D9       mov \[ebp+nNumberOfBytesToRead\], esi  
.text:004925DC       call ds:GetProcessHeap  
.text:004925E2       push esi               ; dwBytes  
.text:004925E3       push 8                 ; dwFlags  
.text:004925E5       push eax               ; hHeap  
.text:004925E6       mov \[ebp+hHeap\], eax  
.text:004925E9       call ds:HeapAlloc  
//申请内存  
.text:004925EF       mov esi, eax  
.text:004925F1       test esi, esi  
.text:004925F3       mov \[ebp+lpMem\], esi  
.text:004925F6       jz loc\_49270C  
.text:004925FC       mov eax, \[ebp+nNumberOfBytesToRead\]  
.text:004925FF       lea edx, \[ebp+NumberOfBytesRead\]  
.text:00492602       push 0                 ; lpOverlapped  
.text:00492604       push edx               ; lpNumberOfBytesRead  
.text:00492605       push eax               ; nNumberOfBytesToRead  
.text:00492606       push esi               ; lpBuffer  
.text:00492607       push edi               ; hFile  
.text:00492608       call ds:ReadFile  
//读取原文件到新申请的内存  
.text:0049260E       test eax, eax  
.text:00492610       jz loc\_4926FF  
.text:00492616       mov \[ebp+var\_4\], 1  
.text:0049261D       mov ebx, 3  
.text:00492622       mov \[ebp+var\_10\], 0  
.text:00492629       mov \[ebp+nNumberOfBytesToRead\], 6  
.text:00492630       mov ecx, 2  
.text:00492635  
.text:00492635 loc\_492635:                  ; CODE XREF: sub\_492570+172j  
.text:00492635       movzx di, byte ptr \[esi\]  
.text:00492639       mov edx, \[ebp+var\_10\]  
.text:0049263C       inc esi  
.text:0049263D       lea eax, \[edi+edx\]  
.text:00492640       xor edx, edx  
.text:00492642       mov dl, byte ptr \[ebp+var\_4+1\]  
.text:00492645       lea eax, \[eax+edi\*4\]  
.text:00492648       mov edi, \[ebp+nNumberOfBytesToRead\]  
.text:0049264B       add edx, eax  
.text:0049264D       add edx, edi  
.text:0049264F       add edx, ebx  
.text:00492651       xor edx, ecx  
.text:00492653       mov eax, edx  
.text:00492655       mov edx, \[ebp+var\_4\]  
.text:00492658       test dh, 80h  
.text:0049265B       jz short loc\_49265F  
.text:0049265D       or al, 1  
.text:0049265F  
.text:0049265F loc\_49265F:                  ; CODE XREF: sub\_492570+EBj  
.text:0049265F       mov \[ebp+var\_4\], eax  
.text:00492662       mov eax, \[ebp+NumberOfBytesRead\]  
.text:00492665       dec eax  
.text:00492666       mov \[ebp+NumberOfBytesRead\], eax  
.text:00492669       jz short loc\_4926DB  
.text:0049266B       xor eax, eax  
.text:0049266D       mov ax, word ptr \[ebp+var\_4\]  
.text:00492671       mov edx, 0F0F0F0F1h  
.text:00492676       mul edx  
.text:00492678       shr edx, 4  
.text:0049267B       mov \[ebp+var\_10\], edx  
.text:0049267E       mov ecx, \[ebp+var\_4\]  
.text:00492681       movzx ax, byte ptr \[esi\]  
.text:00492685       and ecx, 0FFFFh  
.text:0049268B       shl eax, 1  
.text:0049268D       lea edx, ds:0\[ecx\*8\]  
.text:00492694       and eax, 0FFFFh  
.text:00492699       sub edx, ecx  
.text:0049269B       mov ecx, \[ebp+var\_10\]  
.text:0049269E       add edx, eax  
.text:004926A0       xor eax, eax  
.text:004926A2       sar edx, 1  
.text:004926A4       mov al, bh  
.text:004926A6       add ecx, edx  
.text:004926A8       mov edx, \[ebp+nNumberOfBytesToRead\]  
.text:004926AB       add eax, ecx  
.text:004926AD       inc esi  
.text:004926AE       xor eax, edx  
.text:004926B0       test bh, 80h  
.text:004926B3       jz short loc\_4926B7  
.text:004926B5       or al, 1  
.text:004926B7  
.text:004926B7 loc\_4926B7:                  ; CODE XREF: sub\_492570+143j  
.text:004926B7       mov ebx, eax  
.text:004926B9       mov \[ebp+nNumberOfBytesToRead\], ebx  
.text:004926BC       xor eax, eax  
.text:004926BE       mov ax, word ptr \[ebp+nNumberOfBytesToRead\]  
.text:004926C2       mov edx, 9D89D89Eh  
.text:004926C7       mul edx  
.text:004926C9       shr edx, 3  
.text:004926CC       mov \[ebp+var\_10\], edx  
.text:004926CF       mov eax, \[ebp+var\_4\]  
.text:004926D2       lea edx, \[ebx+ebx\]  
.text:004926D5       mov \[ebp+nNumberOfBytesToRead\], edx  
.text:004926D8       lea ecx, \[eax+eax\]  
.text:004926DB  
.text:004926DB loc\_4926DB:                  ; CODE XREF: sub\_492570+F9j  
.text:004926DB       mov eax, \[ebp+NumberOfBytesRead\]  
.text:004926DE       dec eax  
.text:004926DF       mov \[ebp+NumberOfBytesRead\], eax  
.text:004926E2       jnz loc\_492635  
//循环计算检验值  
.text:004926E8       mov eax, \[ebp+var\_14\]  
.text:004926EB       xor ecx, ecx  
.text:004926ED       mov esi, \[ebp+lpMem\]  
.text:004926F0       mov edi, \[ebp+var\_18\]  
.text:004926F3       mov ch, bl  
.text:004926F5       mov cl, byte ptr \[ebp+var\_4\]  
.text:004926F8       add eax, ecx  
.text:004926FA       mov \[ebp+var\_14\], eax  
.text:004926FD       mov ebx, eax 

使用 UPX-Ripper 自动脱壳后调试会有一个内存写入异常  
  
  
.text:00604EC7 loc\_604EC7:        ; CODE XREF: sub\_604540+A11j  
.text:00604EC7       xor edx, edx  
.text:00604EC9       mov eax, 80808081h  
.text:00604ECE       mov dl, \[ebx\]  
.text:00604ED0       imul edx, \[esp+228h+nHeight\]  
.text:00604ED5       mov \[esp+228h+var\_20C\], edx  
.text:00604ED9       imul edx  
.text:00604EDB       add edx, \[esp+228h+var\_20C\]  
.text:00604EDF       sar edx, 7  
.text:00604EE2       mov eax, edx  
.text:00604EE4       shr eax, 1Fh  
.text:00604EE7       add edx, eax  
.text:00604EE9       mov al, \[edi\]  
.text:00604EEB       add dl, al  
.text:00604EED       mov eax, 80808081h  
.text:00604EF2       mov \[ecx\], dl  
//ECX=00740060       内存写入异常    
.text:00604EF4       inc ecx  
.text:00604EF5       inc edi  
.text:00604EF6       inc ebx  
.text:00604EF7       xor edx, edx  
.text:00604EF9       mov dl, \[ebx\]  
.text:00604EFB       imul edx, \[esp+228h+var\_208\]  
.text:00604F00       mov \[esp+228h+var\_20C\], edx  
.text:00604F04       imul edx  
.text:00604F06       add edx, \[esp+228h+var\_20C\]  
.text:00604F0A       sar edx, 7  
.text:00604F0D       mov eax, edx  
.text:00604F0F       shr eax, 1Fh  
.text:00604F12       add edx, eax  
.text:00604F14       mov al, \[edi\]  
.text:00604F16       add dl, al  
.text:00604F18       mov eax, 80808081h  
.text:00604F1D       mov \[ecx\], dl  
.text:00604F1F       inc ecx  
.text:00604F20       inc edi  
.text:00604F21       inc ebx  
.text:00604F22       xor edx, edx  
.text:00604F24       mov dl, \[ebx\]  
.text:00604F26       imul edx, \[esp+228h+lpString\]  
.text:00604F2B       mov \[esp+228h+var\_20C\], edx  
.text:00604F2F       imul edx  
.text:00604F31       add edx, \[esp+228h+var\_20C\]  
.text:00604F35       sar edx, 7  
.text:00604F38       mov eax, edx  
.text:00604F3A       shr eax, 1Fh  
.text:00604F3D       add edx, eax  
.text:00604F3F       mov al, \[edi\]  
.text:00604F41       add dl, al  
.text:00604F43       mov eax, \[esp+228h+var\_200\]  
.text:00604F47       mov \[ecx\], dl  
.text:00604F49       inc ecx  
.text:00604F4A       inc edi  
.text:00604F4B       inc ebx  
.text:00604F4C       dec eax  
.text:00604F4D       mov \[esp+228h+var\_200\], eax  
.text:00604F51       jnz loc\_604EC7  
  
  
原来 UPX-Ripper 把脱壳文件的.rsrc 区段属性设置为 40000040，修改为 C0000040（可读可写）就行了。 

 4dd466d0e1e0cc123afbe504da2462ca