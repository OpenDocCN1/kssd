
# [原创]简单小木马脱壳

一个小木马脱壳，请虚拟机下调试。相关文件见附件。未知壳相当简单但是典型，有stolen code和overlay。只是练习脱壳，所以没有分析木马。

初次实战，一路F8。

载入：

```asm
00410336 >  57            PUSH EDI
00410337   .  8BFC          MOV EDI,ESP
00410339   .  83EC 26       SUB ESP,26
0041033C   .  C74424 0E 000>MOV DWORD PTR SS:[ESP+E],0
00410344   >  C74424 06 FB6>MOV DWORD PTR SS:[ESP+6],DC8A65FB       ;解密key
0041034C   .  C74424 02 000>MOV DWORD PTR SS:[ESP+2],0
00410354   .  BE D5034100   MOV ESI,1.004103D5          ;解密起始地址
00410359   .  897424 12     MOV DWORD PTR SS:[ESP+12],ESI
0041035D   >  EB 00         JMP SHORT 1.0041035F
0041035F   >  BE 7A210000   MOV ESI,217A
00410364   .  03F7          ADD ESI,EDI
00410366   .  2BF4          SUB ESI,ESP                             ;esi + esp + (esp-26h) = esi + 26h
00410368   .  397424 0E     CMP DWORD PTR SS:[ESP+E],ESI            ;[esp+e]计数器
0041036C   .  74 02         JE SHORT 1.00410370
0041036E   .  EB 0C         JMP SHORT 1.0041037C
00410370   >  8B7424 12     MOV ESI,DWORD PTR SS:[ESP+12]           ;[esp+e] == esi时 才开始解密
00410374   .  8A6424 06     MOV AH,BYTE PTR SS:[ESP+6]
00410378   .  2826          SUB BYTE PTR DS:[ESI],AH
0041037A   .  EB 00         JMP SHORT 1.0041037C
0041037C   >  834424 12 01  ADD DWORD PTR SS:[ESP+12],1
00410381   .  C16C24 06 02  SHR DWORD PTR SS:[ESP+6],2
00410386   .  C16C24 06 02  SHR DWORD PTR SS:[ESP+6],2
0041038B   .  C16C24 06 02  SHR DWORD PTR SS:[ESP+6],2
00410390   .  C16C24 06 02  SHR DWORD PTR SS:[ESP+6],2              ;右移8位  即右移一个字节
00410395   .  834424 02 01  ADD DWORD PTR SS:[ESP+2],1
0041039A   .  837C24 02 04  CMP DWORD PTR SS:[ESP+2],4
0041039F   .  75 10         JNZ SHORT 1.004103B1
004103A1   .  C74424 06 FB6>MOV DWORD PTR SS:[ESP+6],DC8A65FB
004103A9   .  C74424 02 000>MOV DWORD PTR SS:[ESP+2],0
004103B1   >  BE 98064100   MOV ESI,1.00410698                      ;解密终止地址
004103B6   .  397424 12     CMP DWORD PTR SS:[ESP+12],ESI
004103BA   .^ 72 A1         JB SHORT 1.0041035D
004103BC   .  FF4424 0E     INC DWORD PTR SS:[ESP+E]
004103C0   .  817C24 0E 332>CMP DWORD PTR SS:[ESP+E],2333
004103C8   .^ 0F86 76FFFFFF JBE 1.00410344
//整个解密过程 简单但是干扰性极大 [esp+e]为21A0时解密才开始  解密过程的key为DC8A65FB  变换过程 循环右移一个字节
00410407  |.  33C0          XOR EAX,EAX
00410409  |.  8B0424        MOV EAX,DWORD PTR SS:[ESP]               ;kernel里的一个地址送eax
0041040C  |.  66:33C0       XOR AX,AX
0041040F  |.  68 FFFF0000   PUSH 0FFFF                               ;函数名HASH值入栈
00410414  |.  68 B674755D   PUSH 5D7574B6                            ;GetProcAddress
00410419  |.  68 2207E471   PUSH 71E40722                            ;LoadLibraryA
0041041E  |.  68 80EFF815   PUSH 15F8EF80                            ;VirtualProtect
00410423  |.  68 EC5863D6   PUSH D66358EC                            ;ExitProcess
00410428  |.  8BFC          MOV EDI,ESP                              ;EDI 用来定位  堆栈
0041042A  |>  66:8138 4D5A  /CMP WORD PTR DS:[EAX],5A4D              ;PE
0041042F  |.  75 13         |JNZ SHORT 1.00410444
00410431  |.  8B50 3C       |MOV EDX,DWORD PTR DS:[EAX+3C]
00410434  |.  81FA 00100000 |CMP EDX,1000
0041043A  |.  77 08         |JA SHORT 1.00410444
0041043C  |.  66:813C10 504>|CMP WORD PTR DS:[EAX+EDX],4550          ;MZ
00410442  |.  74 07         |JE SHORT 1.0041044B
00410444  |>  2D 00000100   |SUB EAX,10000                           ;  UNICODE "ALLUSERSPROFILE=C:\Documents and Settings\All Users"
00410449  |.^ EB DF         \JMP SHORT 1.0041042A
//寻找kernel 基址
0041044B  |> \50            PUSH EAX                                 ;  kernel32.7C800000
0041044C  |.  8B7410 78     MOV ESI,DWORD PTR DS:[EAX+EDX+78]        ;  定位到DataDirectory[0].VirtualAddress
00410450  |.  03F0          ADD ESI,EAX                              ;  定位到导出表
00410452  |.  83C6 18       ADD ESI,18                               ;IMAGE\_EXPORT\_DIRECTORY.NumberOfNames
00410455  |.  93            XCHG EAX,EBX
00410456  |.  AD            LODS DWORD PTR DS:[ESI]
00410457  |.  50            PUSH EAX                                 ;IMAGE\_EXPORT\_DIRECTORY.NumberOfNames入栈
00410458  |.  AD            LODS DWORD PTR DS:[ESI]
00410459  |.  50            PUSH EAX                                 ;IMAGE\_EXPORT\_DIRECTORY.AddressOfFunctions入栈
0041045A  |.  AD            LODS DWORD PTR DS:[ESI]
0041045B  |.  50            PUSH EAX                                 ;IMAGE\_EXPORT\_DIRECTORY.AddressOfNames入栈
0041045C  |.  03C3          ADD EAX,EBX
0041045E  |.  50            PUSH EAX             ;IMAGE\_EXPORT\_DIRECTORY.AddressOfNames+ImageBase入栈
0041045F  |.  AD            LODS DWORD PTR DS:[ESI]
00410460  |.  50            PUSH EAX             ;IMAGE\_EXPORT\_DIRECTORY.AddressOfNameOrdinals入栈
00410461  |.  8BEC          MOV EBP,ESP                              ;EBP指向IMAGE\_EXPORT\_DIRECTORY.AddressOfNameOrdinals
00410463  |>  8B4D 10       /MOV ECX,DWORD PTR SS:[EBP+10]           ;IMAGE\_EXPORT\_DIRECTORY.NumberOfNames
00410466  |.  33D2          |XOR EDX,EDX
00410468  |>  8B75 04       |/MOV ESI,DWORD PTR SS:[EBP+4]           ;IMAGE\_EXPORT\_DIRECTORY.AddressOfNames+ImageBase
0041046B  |.  8B36          ||MOV ESI,DWORD PTR DS:[ESI]             ;ESI 为 函数名的起始地址RVA
0041046D  |.  03F3          ||ADD ESI,EBX                            ;得到函数名地址VA
0041046F  |.  33C0          ||XOR EAX,EAX
00410471  |.  50            ||PUSH EAX
00410472  |>  C1C8 07       ||/ROR EAX,7
00410475  |.  310424        |||XOR DWORD PTR SS:[ESP],EAX
00410478  |.  AC            |||LODS BYTE PTR DS:[ESI]
00410479  |.  84C0          |||TEST AL,AL
0041047B  |.^ 75 F5         ||\JNZ SHORT 1.00410472                  ;函数名HASH
0041047D  |.  58            ||POP EAX
0041047E  |.  57            ||PUSH EDI
0041047F  |>  813F FFFF0000 ||/CMP DWORD PTR DS:[EDI],0FFFF          ;函数是否 地址都找到  找到就跳
00410485  |.  74 09         |||JE SHORT 1.00410490
00410487  |.  3B07          |||CMP EAX,DWORD PTR DS:[EDI]            ;比较函数名HASH值
00410489  |.  74 0F         |||JE SHORT 1.0041049A
0041048B  |.  83C7 04       |||ADD EDI,4
0041048E  |.^ EB EF         ||\JMP SHORT 1.0041047F
00410490  |>  5F            ||POP EDI
00410491  |.  8345 04 04    ||ADD DWORD PTR SS:[EBP+4],4             ;移到下一个函数名起始地址
00410495  |.  42            ||INC EDX
00410496  |.^ E2 D0         |\LOOPD SHORT 1.00410468
00410498  |.  EB 2C         |JMP SHORT 1.004104C6
0041049A  |>  D1E2          |SHL EDX,1
0041049C  |.  8B4D 00       |MOV ECX,DWORD PTR SS:[EBP]              ;IMAGE\_EXPORT\_DIRECTORY.AddressOfNameOrdinals
0041049F  |.  03CB          |ADD ECX,EBX                             ;IMAGE\_EXPORT\_DIRECTORY.AddressOfNameOrdinals+IMAGEBASE
004104A1  |.  03CA          |ADD ECX,EDX                             ;得到特定函数 序号地址
004104A3  |.  8B09          |MOV ECX,DWORD PTR DS:[ECX]              ;得到函数序号
004104A5  |.  81E1 FFFF0000 |AND ECX,0FFFF
004104AB  |.  8B55 0C       |MOV EDX,DWORD PTR SS:[EBP+C]            ;IMAGE\_EXPORT\_DIRECTORY.AddressOfFunctions
004104AE  |.  03D3          |ADD EDX,EBX                             ;IMAGE\_EXPORT\_DIRECTORY.AddressOfFunctions+IAMGEBASE
004104B0  |.  C1E1 02       |SHL ECX,2
004104B3  |.  03D1          |ADD EDX,ECX
004104B5  |.  8B12          |MOV EDX,DWORD PTR DS:[EDX]
004104B7  |.  03D3          |ADD EDX,EBX                             ;得到函数VA
004104B9  |.  8917          |MOV DWORD PTR DS:[EDI],EDX              ;存放函数地址
004104BB  |.  8B4D 08       |MOV ECX,DWORD PTR SS:[EBP+8]
004104BE  |.  03CB          |ADD ECX,EBX
004104C0  |.  894D 04       |MOV DWORD PTR SS:[EBP+4],ECX
004104C3  |.  5F            |POP EDI
004104C4  |.^ EB 9D         \JMP SHORT 1.00410463
//想起failwest的shellcode。。。。。
004104C6  |> \83EF 04       SUB EDI,4
004104C9  |.  8BE7          MOV ESP,EDI
004104CB  |.  8B5C24 08     MOV EBX,DWORD PTR SS:[ESP+8]
004104CF  |.  8BFC          MOV EDI,ESP
004104D1  |.  83EF 04       SUB EDI,4
004104D4  |.  57            PUSH EDI
```
