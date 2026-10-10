

# Hying's Armor v0.7x Code Injection

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-570.htm`](https://www.pediy.com/kssd/pediy07/pediy7-570.htm)

【目     标】:商业类的东西不便说出名字  
【工     具】:Olydbg1.1(diy 版)、LORDPE、ImportREC1.6F   
【目　　的】: 通过注入代码，在程序运行前显示一个提示框  
【操作平台】:Windows 2003 server   
【作     者】: LOVEBOOM\[DFCG\]\[FCG\]\[US\]  
【简要说明】： 相信会脱这个壳的人已经有很多很多了，在你想明白怎么注入代码之前，你得有一点基础知识，对这个壳比较了解，这篇文章是  
讲如何注入，并非脱壳分析，如果你想知道如何脱壳，可以看我以前的文章和别的网友的文章，如果那些文章你都看懂了，而你有一定的基础，相  
信这篇文章对你来说是非常简单的:-)。  
【详细过程】:  
正式开始前先讲一些：这个壳经常伪装成别的壳来欺骗检测工具的检测，骗过对壳没有了解的朋友。这次我找的目标也同样伪装了，这次伪装  
成\*Neolite 2.0 -> Neoworx Inc.\*的了，如果你按那个壳的方法去操作肯定是没有结果的。  
  这个壳的反跟踪比较多，有精彩代码值得我们学习的。用 OD 跟踪比用 SICE 跟踪更为方便，异常太多了。花指令也是做的比较有个性的:-)。  
用 OD 跟踪的的话，只有一个 anti-debug 可以防止我们跟踪。  
也许你会说，直接在壳代码处就改自己的代码不就行了吗?是的，没错。我选择在程序代码解压出来之后再出现信息框是想说，代码已经解压出来  
了，你可以修改程序的流程了，可以暴力破解或找修改壳的流程，去除全部的 anti-debug 让壳和 upx 一样等等，只有想不到的，没有做不到的。  
要在壳运行过程中添加提示信息的话，先得找入手点，找入手点当然比须对壳比较了解，然后考虑我们的写注入的东西里要用到哪些东西，这些东  
西又从哪里获取，是自己添加还是直接借用壳的。Patch 代码又存放到哪里，是否有足够的空间。等等。  
我选择的是在壳解压出程序的代码后，壳 CRC 检测时用到的 UnmapViewOfFile 作为入口点，我做的方式是在壳入口处就开始 hook 掉该进程的 UnmapViewOfFile,  
让壳执行这个程序之前先显示我想显示的对话框。Hook 的过程我们要修改进程空间的代码，所以我们就得先获取相关权限，然后修改内存页，让相关空  
间代码能够修改，这个过程我们就得知道，我这里偷了下懒，作为完整的写法应该是修改完后，还原程序代码，然后把相关权限也写回去的，我没有写还  
原代码的。获取和修改权限就得用到 VirtualQuery 和 VirtualProtect，用 VirtualQuery 获取原有的权限后应该保存一个地方呀，这个地方我选择动态申请，动  
态申请用取了 VirtualAlloc 和 VirtualFree 这两个 API。添加信息框要用到 MessageBoxA 这个 Api,这个 API 在 USER32.DLL 中，壳开始的时候并没有加载这个 dll,所  
以还得用到 LoadLibraryA,载入了 DLL 之后，还得取 api 的地址呀，所以就用到了 GetProcAddress,用 GetProcAddress 中要用到 DLL 的句柄，获取句柄的话就得用  
到 GetModuleHandleA。还好壳里已经有 GetModuleHandleA 这个 api。总结一下用到了这么些 API：  
  GETMODULEHANDLEA  
  **GETPROCADDRESS**  
  LOADLIBRARYA  
  **VIRTUALALLOC**  
  **VIRTUALFREE**  
  **VIRTUALQUERY**  
  **VIRTUALPROTECT**  
  **UnmapViewOfFile**  
  MESSAGEBOXA  
要用的 API 已经得到，要提示信息还得到在空地上写上想提示的信息，还得把要动态获取地址的 API 的名字写上去，还得把那两个 DLL 的名字都写上去。  
OK，自己想了这么些情况，然后开始动手：  
用 peid 看了一下报告成伪壳，随便在段后面的空间里写的点东西，再运行，发现程序不让运行，这说明程序有检测。用 LordPE 看看只有少数几个 api 函数：  
\->Import Table  
   1. ImageImportDescriptor:  
    OriginalFirstThunk:  0x0007E079  
    TimeDateStamp:       0x00000000  (GMT: Thu Jan 01 00:00:00 1970)  
    ForwarderChain:      0x00000000  
    Name:                0x0007E089  ("KERNEL32.dll")  
    FirstThunk:          0x0007E079  
  
    Ordinal/Hint API name  
    ------------ ---------------------------------------  
    0x0000       "GetProcAddress"  
    0x0000       "LoadLibraryA"  
    0x0000       "GetModuleHandleA"  
看到这些输入表信息我们就能达到目标了（如果输入表为空的话，能不能达到我的目的呢?答案是肯定的。因为壳自己也要用到 API，壳用什么方法获取，我们也模拟它来完成使命。）  
上面分析后代码量可能比较大，所以还得找找有没有足够的空间。所以用 lordpe 看看程序的”空地”有多少，壳是加密后，通过在原程序的基础上增加一个段来保存解壳代码，所  
以我们看看最后一个段有没有足够的空间。最后一个段的信息：  
item:  
    Name:                    
    VirtualSize:           0x00014000  
    VirtualAddress:        0x0007E000  
    SizeOfRawData:         0x00014000  
    PointerToRawData:      0x00029A00  
    PointerToRelocations:  0x00000000  
    PointerToLinenumbers:  0x00000000  
    NumberOfRelocations:   0x0000  
    NumberOfLinenumbers:   0x0000  
    Characteristics:       0xC0000040  
    (INITIALIZED\_DATA, READ, WRITE)  
看了下空间不是很多，所以自己修改下，把空间改大点，添加了&H1000 的空间，这样可以放足够的代码了。修改后：  
    Name:                    
    VirtualSize:           0x00015000       \*\*\*\*\*\*\*\*\*\*\*\*  
    VirtualAddress:        0x0007E000  
    SizeOfRawData:         0x00015000      \*\*\*\*\*\*\*\*\*\*\*\*  
    PointerToRawData:      0x00029A00  
    PointerToRelocations:  0x00000000  
    PointerToLinenumbers:  0x00000000  
    NumberOfRelocations:   0x0000  
    NumberOfLinenumbers:   0x0000  
    Characteristics:       0xC0000040  
    (INITIALIZED\_DATA, READ, WRITE)  
修改好后用 OD 载入目标,现在来获取 LoadLibraryA、GetModuleHandleA、GetProcAddress ,  
00491DAD > /E9 00000000     JMP 00491DB2                             ; EP,伪装别的壳的入口  
00491DB2   \\60              PUSHAD  
00491DB3    E8 14000000     CALL 00491DCC  
00491DB8    5D              POP EBP  
载入后，下命令 g GetModuleHandleA，断下后，通过堆栈看看 GetModuleHandleA 载入地址：  
0012FF9C   0047E117  /CALL to GetModuleHandleA from cjbo.0047E114  
0012FFA0   0047E089  \\pModule = "KERNEL32.dll"  
……  
0047E114    FF53 08         CALL DWORD PTR DS:\[EBX+8\]                ; 这里调用 GetModuleHandleA  
0047E117    8BF8            MOV EDI,EAX                              ; 返回到这里  
计算一下得知 GetModuleHandleA 保存在 0047E081 处。因为 IAT 地址一般是连续的，这样的话，直接 DD 47e081 看看数据窗口就知道其它两个 API 保存的位置了：  
0047E079 >77E12DFB  kernel32.GetProcAddress  
0047E07D >77E1850D  kernel32.LoadLibraryA  
0047E081 >77E12CD1  kernel32.GetModuleHandleA  
  
因为关系到重定位，所以为了防止意外我用到相对地址的方式来获取实际地址。  
第一步已经完成了，第二步看看 UnmapViewOfFile 在哪里调用，看它的原因是去找 CRC 检测数据。重来, 这次要去除 ZwSetInformationThread 这个 anti-ring3’debug,否  
则壳会脱离 Debugger,让调试器不能跟踪。去除 ZwSetInformation 时要注意不能直接改成 RET　10 之类的，壳会检测的：  
00382CB2    8A06            MOV AL,BYTE PTR DS:\[ESI\]  
00382CB4    3C B8           CMP AL,0B8                               ; 开始检测了，判断是否为 mov eax,xxxxxx 的方式，如果不是则跳  
00382CB6    75 04           JNZ SHORT 00382CBC  
00382CB8    A4              MOVS BYTE PTR ES:\[EDI\],BYTE PTR DS:\[ESI\]  
00382CB9    A5              MOVS DWORD PTR ES:\[EDI\],DWORD PTR DS:\[ES\>  
00382CBA    EB 33           JMP SHORT 00382CEF  
00382CBC    3C 8D           CMP AL,8D  
00382CBE    75 03           JNZ SHORT 00382CC3                       ; 判断是否为 LEA 的方式  
00382CC0    A5              MOVS DWORD PTR ES:\[EDI\],DWORD PTR DS:\[ES\>  
00382CC1    EB 2C           JMP SHORT 00382CEF  
00382CC3    3C CD           CMP AL,0CD                               ; 判断是否为 Int xx 的入口  
00382CC5    75 04           JNZ SHORT 00382CCB  
00382CC7    66:A5           MOVS WORD PTR ES:\[EDI\],WORD PTR DS:\[ESI\]  
00382CC9    EB 24           JMP SHORT 00382CEF  
00382CCB    3C BA           CMP AL,0BA                               ; 判断是否为 mov edx,address 的方式  
00382CCD    75 04           JNZ SHORT 00382CD3  
00382CCF    A4              MOVS BYTE PTR ES:\[EDI\],BYTE PTR DS:\[ESI\]  
00382CD0    A5              MOVS DWORD PTR ES:\[EDI\],DWORD PTR DS:\[ES\>  
00382CD1    EB 1C           JMP SHORT 00382CEF  
00382CD3    3C FF           CMP AL,0FF                               ; 判断是否为 push \[address\]的方式  
00382CD5    75 04           JNZ SHORT 00382CDB  
00382CD7    66:A5           MOVS WORD PTR ES:\[EDI\],WORD PTR DS:\[ESI\]  
00382CD9    EB 14           JMP SHORT 00382CEF  
00382CDB    3C C2           CMP AL,0C2                               ; 判断是否为 RET val 的方式  
00382CDD    75 0A           JNZ SHORT 00382CE9  
00382CDF    A4              MOVS BYTE PTR ES:\[EDI\],BYTE PTR DS:\[ESI\]  
00382CE0    66:A5           MOVS WORD PTR ES:\[EDI\],WORD PTR DS:\[ESI\]  
00382CE2    61              POPAD  
00382CE3    33C0            XOR EAX,EAX  
00382CE5    40              INC EAX  
00382CE6    C2 0C00         RETN 0C  
00382CE9    61              POPAD  
00382CEA    33C0            XOR EAX,EAX  
00382CEC    C2 0C00         RETN 0C  
当然如果你的目标不是新版本的不会检测的。  
去除这个 anti-Debug 后，follow **UnmapViewOfFile**,在 RET 4 处下断。运行中断后，取消断点，返回壳代码：  
0037DAB9    FFB5 64174100   PUSH DWORD PTR SS:\[EBP+411764\]           ; 返回到这里  
0037DABF    7C 05           JL SHORT 0037DAC6  
0037DAC1    EB 05           JMP SHORT 0037DAC8  
0037DAC3    0FCD            BSWAP EBP  
0037DAC5    2074F9 8D       AND BYTE PTR DS:\[ECX+EDI\*8-73\],DH  
0037DAC9    859D C34000EB   TEST DWORD PTR SS:\[EBP+EB0040C3\],EBX  
0037DACF    020F            ADD CL,BYTE PTR DS:\[EDI\]  
0037DAD1    0F50E8          MOVMSKPS EBP,XMM0  
0037DAD4    04 00           ADD AL,0  
0037DAD6    0000            ADD BYTE PTR DS:\[EAX\],AL  
0037DAD8    CD 20           INT 20  
0037DADA  - E9 6883C404     JMP 04FC5E47  
0037DADF    8B85 A2184100   MOV EAX,DWORD PTR SS:\[EBP+4118A2\]        ; CloseHandle  
0037DAE5    EB 04           JMP SHORT 0037DAEB  
  
没跟多久看会看到，CRC 的比较了。  
0037DC7E    3BF7            CMP ESI,EDI                              ; 到这里进行 CRC 检测  
0037DC80    0F85 F65E0000   JNZ 00383B7C                             ; 不等则 over  
其中 ESI 是计算后的结果，EDI 就是正确的 CRC 值，这里就有三种去除方式了，第一种在把 crc 值保存在文件中，第二种 mov esi,edi,的方式，第三种直接改成 nop。  
用 LordPE 看一下正确的 CRC 值正是保存在 PE header-4 处。  
004000F0  00 00 00 00 00 00 00 00 00 00 00 00 61 95 C1 32  ............a 暳 2  
00400100  50 45 00 00 4C 01 03 00 19 5E 42 2A 00 00 00 00  PE..L  . ^B\*....  
原程序没修改时的正确 CRC 值为 32c19561,这个值是换算后的。换算方法是：  
KEY==CRC xor Key(048F7032)  
key==KEY ROR 3  
如果你现在直接在 od 里是看不到程序解码的，因为我并没有处理 DRx 部分。这篇文章是讲代码注入的，所以我不讲那些东西。  
现在我们的 CRC 值的信息已经找到，也知道是哪里用 UnmapViewOfFile 这个函数，现在只需在壳入口处写上自己的代码，然后在上面比较 CRC 值是否正确的地方，  
把正确的检测值，保存到 pe header-4 处。现在只差写代码，我在这里开始写：  
  
00491DCC    5D              POP EBP                                  ; 我从这里开始修改,我通过相当地址来计算实际地址的,EBP== 00491DB8  
过了这里后，记下 EBP 的值，然后和上面获取的地址运算后得到实际地址  
00491DCD    E8 0E000000     CALL 00491DE0                            ; 先取 Kernel32.dll 的句柄  
整理了一下代码：  
  start:  
    call @F  
      
    @@:  
      pop ebp          ;获取 EBP 的值  
      call @F  
      db 'Kernel32.dll',0,0  
    @@:  
      CALL DWORD PTR SS:\[EBP+0FFFEC2C9h\]        ;获取 Kernel32.dll 的句柄  
      MOV DWORD PTR SS:\[EBP+500h\],EAX           ;保存句柄  
      call @F  
      db 'VirtualAlloc',0  
    @@:  
      PUSH DWORD PTR SS:\[EBP+500h\]              ; push hModule  
      CALL DWORD PTR SS:\[EBP+0FFFEC2C1h\]        ; GetProcAddress  
      MOV DWORD PTR SS:\[EBP+504h\],EAX           ;保存 VirtualAlloc 的地址  
      call @F  
      db 'VirtualQuery',0  
    @@:  
      PUSH DWORD PTR SS:\[EBP+500h\]              ; push hModule        
      CALL DWORD PTR SS:\[EBP+0FFFEC2C1h\]        ; GetProcAddress  
      MOV DWORD PTR SS:\[EBP+508h\],EAX           ;保存 VirtualQuery 的地址  
      CALL @F  
      DB 'VirtualProtect',0,0                   ;因为我自己是用 od 直接写的代码，所以我现在整理把那些多余的 0 都写进来  
    @@:    
      PUSH DWORD PTR SS:\[EBP+500h\]              ; push hModule  
      CALL DWORD PTR SS:\[EBP+0FFFEC2C1h\]        ; GetProcAddress  
      MOV DWORD PTR SS:\[EBP+50Ch\],EAX           ;Save address  
      CALL @F  
      db 'VirtualFree',0,0  
    @@:  
      PUSH DWORD PTR SS:\[EBP+500h\]              ; push hModule  
      CALL DWORD PTR SS:\[EBP+0FFFEC2C1h\]        ; GetProcAddress        
      MOV DWORD PTR SS:\[EBP+510h\],EAX           ;SaveAddress  
      CALL @F  
      db 'User32.dll',0,0  
    @@:  
      CALL DWORD PTR SS:\[EBP+0FFFEC2C5h\]        ;Load Library user32.dll  
      CALL @F  
      db 'MessageBoxA',0  
    @@:  
      push eax                  ;push hModule  
      db 5 dup (90h)  
      CALL DWORD PTR SS:\[EBP+0FFFEC2C1h\]         ; GetProcAddress            
      MOV DWORD PTR SS:\[EBP+514h\],EAX            ;save address  
      call @F  
      db 'UnmapViewOfFile',0,0  
    @@:  
      PUSH DWORD PTR SS:\[EBP+500h\]               ; push hModule  
      CALL DWORD PTR SS:\[EBP+0FFFEC2C1h\]         ; GetProcAddress        
      MOV DWORD PTR SS:\[EBP+518\],EAX             ;save address  
      PUSH 4h                                    ; /Protect = PAGE\_READWRITE  
      PUSH 1000h                                 ; |AllocationType = MEM\_COMMIT  
      PUSH 28h                                   ; |Size = 28 (40.)  
      PUSH 0                                     ; |Address = NULL  
      CALL DWORD PTR SS:\[EBP+504h\]               ; \\VirtualAlloc  
      MOV DWORD PTR SS:\[EBP+52Ch\],EAX            ; 保存 hMeM  
      MOV ESI,EAX                                ; 获取权限  
      PUSH 28h                                   ; /BufSize = 28 (40.)  
      PUSH ESI                                   ; |Buffer = 003A0000  
      PUSH EDI                                   ; |Address = kernel32.UnmapViewOfFile  
      CALL DWORD PTR SS:\[EBP+508h\]               ; \\VirtualQuery  
      LEA EAX,DWORD PTR DS:\[ESI+14h\]             ; 准备修改权限  
      PUSH EAX                                   ; /pOldProtect = 003A0014  
      PUSH 40h                                   ; |NewProtect = PAGE\_EXECUTE\_READWRITE  
      LEA EAX,DWORD PTR DS:\[ESI+0Ch\]             ; |  
      PUSH DWORD PTR DS:\[EAX\]                    ; |Size = 78000 (491520.)  
      PUSH DWORD PTR DS:\[ESI\]                    ; |Address = kernel32.77E16000  
      CALL DWORD PTR SS:\[EBP+50Ch\]               ; \\VirtualProtect  
      PUSH EBX                                   ; 修改 API 入口代码  
      MOV EAX,DWORD PTR SS:\[EBP+518h\]  
      INC EAX  
      LEA EBX,DWORD PTR SS:\[EBP+196h\]  
      SUB EBX,EAX  
      SUB EBX,5h  
      MOV BYTE PTR DS:\[EAX\],0E8h  
      MOV DWORD PTR DS:\[EAX+1\],EBX  
      POP EBX                                    ; 7FFDF000  
      POPAD  
      ;JMP 0047E0D6                              ; 跳去执行原壳的流程  
  end start  
修改 API 入口入＋1 的代码为:  
77E1667C >  53              PUSH EBX             ;API 入口  
77E1667D    E8 CCB86788     CALL cjb.00491F4E  
然后在 491F4E 处写上相关的代码就可以了：  
00491F4E    60              PUSHAD  
00491F4F    E8 00000000     CALL 00491F54  
00491F54    5D              POP EBP  
00491F55    8B85 7C030000   MOV EAX,DWORD PTR SS:\[EBP+37C\]  
00491F5B    C740 01 8B5C240>MOV DWORD PTR DS:\[EAX+1\],8245C8B         ; 还原 API 的代码  
00491F62    C640 05 56      MOV BYTE PTR DS:\[EAX+5\],56  
00491F66    8D40 01         LEA EAX,DWORD PTR DS:\[EAX+1\]  
00491F69    894424 20       MOV DWORD PTR SS:\[ESP+20\],EAX  
00491F6D    9C              PUSHFD  
00491F6E    6A 40           PUSH 40                                  ; /Style = MB\_OK|MB\_ICONASTERISK|MB\_APPLMODAL  
00491F70    8D85 AC000000   LEA EAX,DWORD PTR SS:\[EBP+AC\]            ; |  
00491F76    50              PUSH EAX                                 ; |Title = "Demo"  
00491F77    8D85 B2000000   LEA EAX,DWORD PTR SS:\[EBP+B2\]            ; |  
00491F7D    50              PUSH EAX                                 ; |Text = "Hying's Armor v0.7x Code Injection Demo  
00491F7E    6A 00           PUSH 0                                   ; |hOwner = NULL  
00491F80    FF95 78030000   CALL DWORD PTR SS:\[EBP+378\]              ; \\MessageBoxA  
00491F86    9D              POPFD  
00491F87    61              POPAD  
00491F88    C3              RETN  
代码已经写好了，保存一下，然后通过上面找 CRC 的方式找出实际的正确值。写在 PE HEADER-4 处：  
004000F0  00 00 00 00 00 00 00 00 00 00 00 00 07 61 90 D6  ............a 愔  
00400100  50 45 00 00 4C 01 03 00 19 5E 42 2A 5B 4C 6F 72  PE..L  . ^B\*\[Lor  
全部修改完，保存为文件，试试就可以成功了。再说一下，因为是商业性的东西，所以我不以具体怎么破解之类的来写。  
Greetz:  
Fly.Jingulong,yock,tDasm.David.hexer,hmimys,ahao.UFO(brother).alan(sister).all of my friends and you!  
  
                                                    By loveboom\[DFCG\]\[FCG\]\[US\]  
                                                    http://blog.csdn.net/bmd2chen  
                                                    Email:loveboom#163.com  
                                                    Date:2005-6-6 17:59  
'好色'了一下![](img/83bdd63c29485f7768335d7ba60e67a5.jpg)  

 4dd466d0e1e0cc123afbe504da2462ca