

# System Cleaner. 4.91d 

> 原文：[`www.pediy.com/kssd/pediy06/pediy6390.htm`](https://www.pediy.com/kssd/pediy06/pediy6390.htm)

这个是论坛上别人转的一篇帖子，看看里面手工修复 IAT 的部分写的挺详细，所以大概翻译了一下，希望能给和我一样的新手一些帮助。   
\----by  ikki   
  
用 OLLDBG 脱 ASPR1.23   
(ASPR 1.23 Unpacking "Step-By-Step")                                     
\=========================================================              
目标连接：              : www.systemcleaner.com                        
程序：                  : System Cleaner. 4.91d                        
作者：                  : LaBBa                                        
                                                     
\========================                                               
在我开始尝试脱这个程序和别的新版本的 aspr 压缩的程序的时候，我 dump 下来的文件都是错误的，   
因为我在错误的地方 dump 程序，并因此常常导致脱壳后的程序崩溃。所以，这个教程将说明这个   
错误的原因以及这样对这个版本的 aspr 文件脱壳。   
                                   
需要的工具:                                                    
\-----------------------                                                
1) 你的头脑                                   
2) OllyDbg 1.09d                
3) Olly 的插件 : Cmdline and Plug108 (http://home.t-online.de/home/Ollydbg/)   
4) ProcDump 或者 Lord-Pe                                                 
5) Hview 或者 Hex Editor                                                 
6) ImpRec 1.6 Final Public (Mackt/UCF)                                 
\* 以及一些 pe 文件结构的知识 ..                          
\===============================                                        
脱壳步骤：                                           
\===============================                                        
1) 如何绕过 Anti-Debugger 保护                              
2) 如何找到合适的位置来 dump 文件                     
3) 如何找到程序真正的 OEP                                                
4) 关于 stolen bytes                                           
5) 如何用 ImpRec 修复 IAT                                          
6) 如何修复 OEP 并补上 Stolen bytes                         
7) 致谢                                                             
\=======================================                                
如何绕过 Anti-Debugger 保护：                                 
\=======================================                                
这个版本的 ASPR 含有 Atni-Debugger 代码                          
在 Cmdline 中简单的设置 BP IsDebuggerPresents 然后按 Shift+F9 我们将会到达这里:                                                
77E72E92 > 64:A1 18000000   MOV EAX,DWORD PTR FS:\[18\]                  
77E72E98   8B40 30          MOV EAX,DWORD PTR DS:\[EAX+30\]              
77E72E9B   0FB640 02        MOVZX EAX,BYTE PTR DS:\[EAX+2\]              
77E72E9F   C3               RETN                                       
在: MOVZX EAX,BYTE PTR DS:\[EAX+2\]处                                     
我们看到: BYTE PTR DS:\[EAX+2\] ==7FFDF002 == 01                     
01 - 表示发现调试器 .. 所以每次都要把这个值修改为 00 以便我们能够运行程序。按 Ctrl+F2 重新运行程序，运行到上面那个地方，然后在 dump 窗口中右键单击选择：转到-->表达式(GoTo->Expression),输入地址：7FFDF002，把内存中这个地址的值修改为:00。现在我们可以开始脱壳了.   
  
\===================================================                    
如何找到合适的位置来 dump 文件                        
\===================================================                    
一次次的按 Shift+F9，一直到我们到达这儿:   
00FF3A2C   3100             XOR DWORD PTR DS:\[EAX\],EAX                 
00FF3A2E   64:8F05 00000000 POP DWORD PTR FS:\[0\]                       
00FF3A35   58               POP EAX                                    
00FF3A36   833D B07EFF00 00 CMP DWORD PTR DS:\[FF7EB0\],0                
00FF3A3D   74 14            JE SHORT 00FF3A53                          
00FF3A3F   6A 0C            PUSH 0C                                    
00FF3A41   B9 B07EFF00      MOV ECX,0FF7EB0                            
00FF3A46   8D45 F8          LEA EAX,DWORD PTR SS:\[EBP\-8\]               
00FF3A49   BA 04000000      MOV EDX,4                                  
00FF3A4E   E8 EDD0FFFF      CALL 00FF0B40                              
00FF3A53   FF75 FC          PUSH DWORD PTR SS:\[EBP\-4\]                  
00FF3A56   FF75 F8          PUSH DWORD PTR SS:\[EBP\-8\]                  
00FF3A59   8B45 F4          MOV EAX,DWORD PTR SS:\[EBP\-C\]               
00FF3A5C   8338 00          CMP DWORD PTR DS:\[EAX\],0                   
00FF3A5F   74 02            JE SHORT 00FF3A63                          
00FF3A61   FF30             PUSH DWORD PTR DS:\[EAX\]                    
00FF3A63   FF75 F0          PUSH DWORD PTR SS:\[EBP\-10\]                 
00FF3A66   FF75 EC          PUSH DWORD PTR SS:\[EBP\-14\]                 
00FF3A69   C3               RETN                                       
按 F2 在 RET 那里设置一个断点，然后再次按 Shift+F9 ..                                                                
我们会在停在 RET 这一行..                                     
现在是设置跟踪命令(Trace Command)的时候了..                                  
我们知道，大多数 pe 文件的开始地址是 : 400000 -- FFFFFFFF                                         
但是小程序并不是这样 .. 小程序通常结束于 9FFFFF   
我们现在停在的地址是 : 00FF3A69                                    
所以我们可以通过命令行插件来跟踪这个程序直到它返回 OEP,就象这样:                                    
按 Alt+F1 再命令行插件的输入框中输入: TC EIP<900000 然后回车。   
这样，当程序运行到小于 900000 的地址时，将会象我们希望的那样自动中断下来..                                          
我们停在这里 :                                                   
00407278  -FF25 2C235700    JMP DWORD PTR DS:\[57232C\] <<--这里        
0040727E   8BC0             MOV EAX,EAX                                
00407280  -FF25 28235700    JMP DWORD PTR DS:\[572328\]                  
00407286   8BC0             MOV EAX,EAX                                
00407288  -FF25 24235700    JMP DWORD PTR DS:\[572324\]                  
0040728E   8BC0             MOV EAX,EAX                                
00407290  -FF25 20235700    JMP DWORD PTR DS:\[572320\]                  
00407296   8BC0             MOV EAX,EAX                                
00407298   50               PUSH EAX                                   
00407299   6A 40            PUSH 40                                    
0040729B   E8 E0FFFFFF      CALL SystemCl.00407280                     
004072A0   C3               RETN                                       
如果我们按一次 F8 我们将回到壳的代码中，       
所以我们按一次 F8 并再次执行跟踪命令(Trace Command)          
这一次，我们来到这里:                                        
0040734D   A3 68E65600      MOV DWORD PTR DS:\[56E668\],EAX            ; SystemCl.00400000   
00407352   A1 68E65600      MOV EAX,DWORD PTR DS:\[56E668\]              
00407357   A3 D8505600      MOV DWORD PTR DS:\[5650D8\],EAX              
0040735C   33C0             XOR EAX,EAX                                
0040735E   A3 DC505600      MOV DWORD PTR DS:\[5650DC\],EAX              
00407363   33C0             XOR EAX,EAX                                
00407365   A3 E0505600      MOV DWORD PTR DS:\[5650E0\],EAX              
  
  
0040736A   E8 C1FFFFFF      CALL SystemCl.00407330                     
0040736F   BA D4505600      MOV EDX,SystemCl.005650D4                  
00407374   8BC3             MOV EAX,EBX                                
00407376   E8 75D8FFFF      CALL SystemCl.00404BF0                     
0040737B   5B               POP EBX                                    
0040737C   C3               RETN                                       
这些是什么东西??                                                
这个问题花了我一些时间但在调试了更多的别的程序之后，我认为这是对 GetModuleHandleA 的调用代码的一部分，看起来应该象这样的:                                  
CALL <JMP.&kernel32.GetModuleHandleA>                                  
MOV DWORD PTR DS:\[584668\],EAX            ; SystemCl.00400000           
MOV EAX,DWORD PTR DS:\[584668\]                                          
MOV DWORD PTR DS:\[5780D8\],EAX                                          
XOR EAX,EAX                                                            
MOV DWORD PTR DS:\[5780DC\],EAX                                          
XOR EAX,EAX                                                            
MOV DWORD PTR DS:\[5780E0\],EAX                                          
CALL SystemCl.00407450                                                 
MOV EDX,SystemCl.005780D4                                              
MOV EAX,EBX                                                            
CALL SystemCl.00404BFC                                                 
POP EBX                                                                
RETN                                                                   
因为这是我们需要 Dump 的文件的一部分，所以我们应该在第二次跟踪到达这里的时候 dump: 0040734D..   
打开 ProcDump / Lord-PE 选择 dump(full)来 dump 进程                     
\=======================                                                
如何找到程序真正的 OEP                                                   
\=======================                                                
地址 0040734D 并不是 OEP，我们按 F8 单步执行，直到 RET，返回(ret)后我们来到这里:                                              
00564BEC   FF15 A4D15600    CALL DWORD PTR DS:\[56D1A4\]               ; SystemCl.00564460   
00564BF2   E8 C101EAFF      CALL SystemCl.00404DB8                     
00564BF7   90               NOP                                        
00564BF8   0000             ADD BYTE PTR DS:\[EAX\],AL                   
00564BFA   0000             ADD BYTE PTR DS:\[EAX\],AL                   
00564BFC   0000             ADD BYTE PTR DS:\[EAX\],AL                   
好了.. 现在我们可以用 ProcDump/Pe-Tool 来 dump 进程了，然后得到一个 dump 文件 ..                                                     
但那里才是 OEP ?                                                   
是这儿吗: 00564BEC ??                                                    
NO!                                                                    
如果你往上看，你会发现这样的代码:                                
00564BD3   0000             ADD BYTE PTR DS:\[EAX\],AL                   
00564BD5   0000             ADD BYTE PTR DS:\[EAX\],AL                   
00564BD7   00A444 56000000  ADD BYTE PTR SS:\[ESP+EAX\*2+56\],AH          
00564BDE   0000             ADD BYTE PTR DS:\[EAX\],AL                   
00564BE0   0000             ADD BYTE PTR DS:\[EAX\],AL                   
00564BE2   0000             ADD BYTE PTR DS:\[EAX\],AL                   
00564BE4   0000             ADD BYTE PTR DS:\[EAX\],AL                   
00564BE6   00E8             ADD AL,CH                                  
真正的 OEP 是 :                                                      
00564BDC                                                        
  
你可能会问"为什么 ?"...                                             
ASPR 使用了一种叫做 "Steal-Bytes"的技术：                              
asrp 隐藏了 PE 文件 OEP 开始处的字节，并把这些字节从原始 PE 文件中擦除..   
\=====================                                                  
关于 stolen bytes                                                       
\=====================                                                  
稍微说明一下:                                                      
每一种编译工具例如 : VC++ , Delphi , Borland , etc..                  
在 OEP 有一个唯一的/相同的 PE 头                                
其中的一些是这样的:                                               
Push EBP                                                               
MOV Ebp,Esp                                                            
Add ESP , -010                                                         
Mov EAX, SOME\_VALUE                                                    
(共 11bytes)                                                     
或者:                                                                   
Push EBP                                                               
MOV Ebp,Esp                                                            
Add ESP , -010                                                         
Push EBX                                                               
Push ESi                                                               
Push EDi                                                               
Mov EAX, SOME\_VALUE                                                    
(共 14 bytes)                                                     
我不能确定 ADD ESP 的数值，但在这个程序中是：add esp,-10，当然，不是所有的程序都一样的。    
在这个程序中，我们看看:                                               
00564BD7   00A444 56000000  ADD BYTE PTR SS:\[ESP+EAX\*2+56\],AH          
00564BDE   0000             ADD BYTE PTR DS:\[EAX\],AL                   
00564BE0   0000             ADD BYTE PTR DS:\[EAX\],AL                   
00564BE2   0000             ADD BYTE PTR DS:\[EAX\],AL                   
00564BE4   0000             ADD BYTE PTR DS:\[EAX\],AL                   
00564BE6   00E8             ADD AL,CH                                  
从地址: 00564BE6 开始到我们找到的真正的 OEP:00564BDC,总共是 11 字节               
所以我们需要补上这 11 字节，现在的问题就是：EAX 的值是多少?                           
我发现这其实很容易找到..            
看看这段代码:                                                   
0040734D   A3 68E65600      MOV DWORD PTR DS:\[56E668\],EAX            ; SystemCl.00400000   
00407352   A1 68E65600      MOV EAX,DWORD PTR DS:\[56E668\]                                           
00407357   A3 D8505600      MOV DWORD PTR DS:\[5650D8\],EAX                                           
0040735C   33C0             XOR EAX,EAX                                                             
0040735E   A3 DC505600      MOV DWORD PTR DS:\[5650DC\],EAX                                           
00407363   33C0             XOR EAX,EAX                                                             
00407365   A3 E0505600      MOV DWORD PTR DS:\[5650E0\],EAX                                           
0040736A   E8 C1FFFFFF      CALL SystemCl.00407330                                                  
0040736F   BA D4505600      MOV EDX,SystemCl.005650D4                                               
00407374   8BC3             MOV EAX,EBX                                                             
00407376   E8 75D8FFFF      CALL SystemCl.00404BF0                                                  
0040737B   5B               POP EBX                                                                 
0040737C   C3               RETN                                                                    
注意这里:                                                                                         
00407374   8BC3             MOV EAX,EBX                                                             
我们将得到我们需要的 EAX 的正确值                                                             
在这个例子是 : 5644CC                                                                           
ok,现在剩下的工作就是修复 IAT 了。                                                           
\=================                                                                                   
如何 IAT                                                                                      
\=================                                                                                   
在 improved ImpRec 1.6 Final 中选取进程，然后        
点击: IAT AutoSearch                                                                   
修改 RVA 的大小(Size of the RVA )为 1000                                                              
然后点击获取输入表(GetImports)                                                                   点击显示无效地址(Show Invalid)                                                                   在无效地址列表(invalid list)中点击右键选择"Trace Level 1"                                        这一步骤完成之后，把那些太大的需要删除的地址清除掉(右键 , Cut Thunks)                                                                    
例如:                                                                          
1           00172278                kernel32.dll            019D        GetStartupInfoA             
0           0017227C                ?           0000        00FF17E4                <<-- don't cut   
0           00172280                ?           0000        00FF1CA4                            <<-- don't cut   
1           00172284                kernel32.dll            0166        GetModuleFileNameA          
1           00172288                kernel32.dll            015E        GetLocaleInfoA              
1           0017228C                kernel32.dll            015B        GetLastError                
1           00172290                kernel32.dll            012E        GetCurrentDirectoryA        
0           00172294                ?           0000        00FF1D18                            <<-- don't cut   
1           00172298                kernel32.dll            00E6        FreeLibrary                 
1           0017229C                kernel32.dll            00C6        FindFirstFileA              
1           001722A0                kernel32.dll            00C2        FindClose                   
1           001722A4                kernel32.dll            00AC        ExitProcess                 
1           001722A8                kernel32.dll            00AD        ExitThread                  
1           001722AC                kernel32.dll            0066        CreateThread                
1           001722B0                kernel32.dll            0377        WriteFile                   
1           001722B4                kernel32.dll            0343        UnhandledExceptionFilter    
1           001722B8                kernel32.dll            02F2        SetFilePointer              
1           001722BC                kernel32.dll            02E9        SetEndOfFile                
1           001722C0                kernel32.dll            02B2        RtlUnwind                   
1           001722C4                kernel32.dll            0291        ReadFile                    
1           001722C8                kernel32.dll            0284        RaiseException              
1           001722CC                kernel32.dll            024B        MoveFileA                   
1           001722D0                kernel32.dll            019F        GetStdHandle                
1           001722D4                kernel32.dll            014E        GetFileSize                 
1           001722D8                kernel32.dll            0151        GetFileType                 
1           001722DC                kernel32.dll            0079        DeleteFileA                 
1           001722E0                kernel32.dll            004B        CreateFileA                 
1           001722E4                kernel32.dll            002D        CloseHandle                 
0           001722E8                ?           0000        CCB36727                            <<-- CUT !   
1           001722EC                user32.dll              0128        GetKeyboardType            
  
1           001722F0                user32.dll              01C9        LoadStringA                 
1           001722F4                user32.dll              01DD        MessageBoxA                 
1           001722F8                user32.dll              002B        CharNextA                   
0           001722FC                ?           0000        2C24B7E9                            <<-- CUT !   
1           00172300                advapi32.dll            01EC        RegQueryValueExA            
1           00172304                advapi32.dll            01E2        RegOpenKeyExA               
1           00172308                advapi32.dll            01C9        RegCloseKey                 
0           0017230C                ?           0000        48E33A34                            <<-- CUT !   
1           00172310                oleaut32.dll            0006        SysFreeString               
1           00172314                oleaut32.dll            0005        SysReAllocStringLen         
1           00172318                oleaut32.dll            0004        SysAllocStringLen           
在清除完后，我们还有 8 个无效的项需要修复 :                                            
1) 00FF1CCC                                                                                         
2) 00FF17E4                                                                                         
3) 00FF1CA4                                                                                         
4) 00FF1D18                                                                                         
5) 00FF1D08                                                                                         
6) 00FF1D00                                                                                         
7) 00FF1CF8                                                                                         
8) 00FF1D30                                                                                         
点击 show invalid 并右键单击选择其中的一个，让我们来修复它 :                                                                                
00FF1D30 <- 右键单击选择 DeasmHexView 查看:                
push ebp                                                                                            
00FF1D31                                                                                            
mov ebp,esp                                                                                         
00FF1D33                                                                                            
mov eax,\[FF7E24\]    // DWORD value: 00152398                                                         
00FF1D39                                                                                            
pop ebp                                                                                             
00FF1D3A                                                                                            
retn 4                                                                                              
仔细瞧瞧 .. 这个函数在开始的时候 push EBP 然后在结束的时候 Pop EBP,所以这个函数什么都没执行，应该是 LockResource 或者 FreeResource                                                                     
我选择的是: FreeResource                                                                            
\----------------------------------------------------------------------------------------------                                                                                              
在新版本的 imprec 中有个 Deasmbler/HexView 的功能，你可以选择一个无效的 api 查看调用代码 ...                                                               
修复 8 个无效的地址:                                                                                  
1) 00FF1CCC                                                                                         
00FF1CCC   FF35 147EFF00    PUSH DWORD PTR DS:\[FF7E14\]                                              
00FF1CD2   58               POP EAX                                                                 
00FF1CD3   C3               RETN                                                                    
如果你执行这段代码你会看到:                                        
00407235   25 FF000000      AND EAX,0FF                                                             
在我这里，执行完这段代码后(OS WinXP Pro)EAX\=5                                                    这个是操作系统的值。每个操作系统都有一个唯一的标识                
所以这个是: GetVersion  <-- Good Imprec !                                                          
\----------------------------------------------------------------------------------                  
2) 00FF17E4                                                                                
  
00FF17E4   55               PUSH EBP                                   
00FF17E5   8BEC             MOV EBP,ESP                                
00FF17E7   8B55 0C          MOV EDX,DWORD PTR SS:\[EBP+C\]               
00FF17EA   8B45 08          MOV EAX,DWORD PTR SS:\[EBP+8\]               
00FF17ED   8B0D 8464FF00    MOV ECX,DWORD PTR DS:\[FF6484\]              
00FF17F3   8B09             MOV ECX,DWORD PTR DS:\[ECX\]                 
00FF17F5   3BC8             CMP ECX,EAX                                
00FF17F7   75 09            JNZ SHORT 00FF1802                         
00FF17F9   8B0495 D863FF00  MOV EAX,DWORD PTR DS:\[EDX\*4+FF63D8\]        
00FF1800   EB 07            JMP SHORT 00FF1809                         
00FF1802   52               PUSH EDX                                   
00FF1803   50               PUSH EAX                                   
00FF1804   E8 B739FFFF      CALL 00FE51C0     ; JMP to kernel32.GetProcAddress   
00FF1809   5D               POP EBP                                    
00FF180A   C2 0800          RETN 8                                     
正如我们看到的那样: GetProcAddress  <-- Good Imprec !                
\-----------------------------------------------------------------------------------   
3) 00FF1CA4                                                            
00FF1CA4   55               PUSH EBP                                   
00FF1CA5   8BEC             MOV EBP,ESP                                
00FF1CA7   8B45 08          MOV EAX,DWORD PTR SS:\[EBP+8\]               
00FF1CAA   85C0             TEST EAX,EAX                               
00FF1CAC   75 13            JNZ SHORT 00FF1CC1                         
00FF1CAE   813D A47AFF00 00>CMP DWORD PTR DS:\[FF7AA4\],400000   ; ASCII "MZP"   
00FF1CB8   75 07            JNZ SHORT 00FF1CC1                         
00FF1CBA   A1 A47AFF00      MOV EAX,DWORD PTR DS:\[FF7AA4\]              
00FF1CBF   EB 06            JMP SHORT 00FF1CC7                         
00FF1CC1   50               PUSH EAX                                   
00FF1CC2   E8 F134FFFF      CALL 00FE51B8    ; JMP to kernel32.GetModuleHandleA   
00FF1CC7   5D               POP EBP                                    
00FF1CC8   C2 0400          RETN 4                                     
00FF1CCB   90               NOP                                        
00FF1CCC   FF35 147EFF00    PUSH DWORD PTR DS:\[FF7E14\]                 
00FF1CD2   58               POP EAX                                    
00FF1CD3   C3               RETN                                       
如上: GetModuleHandleA   <-- Good Imprec !             
\------------------------------------------------------------------------------------   
4) 00FF1D18                                                            
00FF1D18   6A 00            PUSH 0                                     
00FF1D1A   E8 9934FFFF      CALL 00FE51B8    ; JMP to kernel32.GetModuleHandleA   
00FF1D1F   FF35 147EFF00    PUSH DWORD PTR DS:\[FF7E14\]                 
00FF1D25   58               POP EAX                                    
00FF1D26   8B05 247EFF00    MOV EAX,DWORD PTR DS:\[FF7E24\]              
00FF1D2C   C3               RETN                                       
仔细看看，你会发现这个可不是 GetModuleHandleA   
为什么呢? 因为在 API GetModuleHandleA 的调用之后程序利用这段代码来获取某些信息存入 EAx 作为调用的返回值 ..                             
那么这个是是什么函数呢? 执行完这段代码之后你会发现 EAX 中是程序路径的 ascii 字符串。                 所以这个是: GetCommandLineA   <-- Good Imprec !                       
\------------------------------------------------------------------------------------   
5) 00FF1D08                                                            
00FF1D08   55               PUSH EBP                                   
00FF1D09   8BEC             MOV EBP,ESP                                
00FF1D0B   8B05 247EFF00    MOV EAX,DWORD PTR DS:\[FF7E24\]              
00FF1D11   8B45 08          MOV EAX,DWORD PTR SS:\[EBP+8\]               
00FF1D14   5D               POP EBP                                    
00FF1D15   C2 0400          RETN 4                                     
仔细瞧瞧 .. 这个函数在开始的时候 push EBP 然后在结束的时候 Pop EBP,所以这个函数什么都没执行，应该是 LockResource 或者 FreeResource                                                                     
我选择的是: FreeResource       <-- Good Imprec !                         
\------------------------------------------------------------------------------------   
6) 00FF1D00                                                            
00FF1D00   A1 207EFF00      MOV EAX,DWORD PTR DS:\[FF7E20\]              
00FF1D05   C3               RETN                                       
这个真有趣 .. 执行这段代码你会看到在 EAX 中有一个值， 这个值是进程的 PID!                           
打开 ImpRec 在进程列表中你会看到同样的 PID 值(PID - Process ID)                                           
所以这个是 : GetCurrentProcessId   <-- Good Imprec !                   
\------------------------------------------------------------------------------------   
7) 00FF1CF8                                                            
00FF1CF8   A1 187EFF00      MOV EAX,DWORD PTR DS:\[FF7E18\]              
00FF1CFD   C3               RETN                                       
在：00FF1CF8 和 00FF1D00 这两个邻近的地址中我只能在 00FF1D00 这个地址设置断点，发现是: GetCurrentProcessId     
我只能得出一个结论 .. 这个 api 一定是: GetCurrentProcess   
新版本的 imprec 也把这个地址修复为 GetCurrentProcessId...      
在新版本的 Imprec 的 News.txt 中说明了:                                     
"出色的 disassembler/hew-viewer 功能有助于你查看重定向代码. 你不再需要 ASProtect 修复插件，因为 ASProtect 修复插件并没有更新，只是做为一个例子放在那儿"                                           问题不是出在 ImpRec 而是那个插件，我说这些只是为了你能有一些了解。   
.. GetCurrentProcessId 也工作的很好...   
\------------------------------------------------------------------------------------   
8) 00FF1D30                                                            
00FF1D30   55               PUSH EBP                                   
00FF1D31   8BEC             MOV EBP,ESP                                
00FF1D33   8B05 247EFF00    MOV EAX,DWORD PTR DS:\[FF7E24\]              
00FF1D39   5D               POP EBP                                    
00FF1D3A   C2 0400          RETN 4                                     
如前所叙，我选的是   : FreeResource    <-- Good Imprec !                          
好了，IAT 修复完了 ..              
\=============================================                          
如何修复 OEP 并补上 Stolen bytes                            
\=============================================                          
用 Pe-Editor of ProcDump  Lord-PE 把 OEP 修改为:564BDC    
然后打开 Hex-Edit  Hview 把 OEP 处的"00"修改为   
Push EBP                                                          
Mov EBP,ESP                                                            
Add ESP,-010                                                           
Mov eax, 05644CC                                                       
到这里 aspr 的脱壳就完成了...                                         
致谢 :                                                              
\=============                                                          
to all the ppl that have helped me lern and beeing there for me ..     
Mackt , ^Daemon^ , VAG , EvilWT , dynm8 , urad0x , parabytes , Jb\_\_ , NeOXQuiCK , SexyGeek ,    
ThE-SAiNt                                                              
SAC , NchantA , Eternal Bliss , evaluator , Bengaly , Termin-X , jond .   
all the ppl at http://woodmann.cjb.net -  RCE Messageboard's           
and to all the ppl the i forgoted :D  
 4dd466d0e1e0cc123afbe504da2462ca