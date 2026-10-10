

# Mp3 TagClinic V2.5 Build 7

> 原文：[`www.pediy.com/kssd/pediy05/pediy50210.htm`](https://www.pediy.com/kssd/pediy05/pediy50210.htm)

aspr 程序的脱壳修复体会   
作者：Fpc/CCG   
  
  工具：softice, fs0's aspr Loader, imprec 1.42, icedump, superbpm, fi2.45   
    
  我们的目标是.....呵呵，不要条件反射，是脱壳。Mp3 TagClinic V2.5 Build 7。跟过以前版本的注册码，现在是用了 aspr 的 Key 注册方式，注册码是不用想了:( 只能脱壳，暴破。   
    
  目标软件可在华军下到，8 月 1、2 号更新的，1.8M。用 fi245 检查是 Asprotect 1.2x 的壳，谁知道呢。限制方式是一定的使用次数，好象是 20 次，注册码验证集成在 aspr 中。   
    
  说说我明白的地方：   
  找入口点用 fs0 写的 loader，太牛了~~，dump 可用 softice + superbpm + icedump 或 trw2000 + superbpm，接下来是修复 ImportTable，用现成的工具 ImpRec 1.42+(Revirgin 不熟~) 这一版本的 aspr 无法全部修复，插件看来需要升级了。还有 8 个函数需要手工修复(不同机器上的偏移地址不同)：   
  
1.  0130E8DC   
2.  0130E930   
3.  0130E93C   
4.  0130E8F8   
5.  0130E94C   
6.  0130E95C   
7.  01321A0C   
8.  01321A1C   
  
  
  
  下面是修复过程，首先简单看一下 aspr 对一些函数的优待：   
  
019F:0131253B  PUSH      0131290D   
019F:01312540  PUSH      DWORD PTR FS:\[EAX\]   
019F:01312543  MOV      FS:\[EAX\],ESP   
019F:01312546  CALL      0130E70C        <- 特殊函数的处理   
  
\>>>>>>>>   
  
019F:0130E70C  PUSHAD   
019F:0130E70D  PUSH      01316716   
019F:0130E712  PUSH      00   
019F:0130E714  PUSH      0130E723   
019F:0130E719  MOV      EAX,\[013050B2\]   
019F:0130E71F  JMP      \[EAX\]   
019F:0130E721  JMP      0130E748   
019F:0130E723  PUSH      0130E72E   
019F:0130E728  JMP      0130E7B2   
  
... ....   
  
<<<<<<<<   
  
  
019F:0131254B  CALL      0131258A   
019F:01312550  PUSH      01312559   
019F:01312555  INC      DWORD PTR \[ESP\]   
019F:01312558  RET                                                                
019F:01312559  MOV      ESP,0C24448B   
019F:0131255E  JMP      01312561   
  
  上面这个 call 不长，里面有花指令，它对几个特殊的函数作处理。举例来说，对于向 GetVersion 这类函数来说，返回值为一个 DWORD，是在 eax 中。aspr 在壳中执行 GetVersion，将返回的 eax 放到内存的某个位置；而后程序再调用 GetVersion 时，由于 Import Address Table 作了处理，并不是再次执行这个函数，只是简单的从内存中赋值给 eax，返回，算是反脱壳的一种方式。对付此种加密方式，对内存保留返回值的地方下 bpm，倒追，就可以找到究竟是调用哪一个函数来的。   
  
(1)来看第一个 0130E8DC ：   
  
019F:0040700A  RET   
019F:0040700B  NOP   
019F:0040700C  PUSH      EAX   
019F:0040700D  PUSH      00                 <- 参数\*\*   
019F:0040700F  CALL      00406F0C            <- 指向 0130E8DC   
019F:00407014  MOV      EDX,005D6108   
019F:00407019  PUSH      EDX   
019F:0040701A  MOV      \[005DB4DC\],EAX    
  
  
call 的内容：   
  
019F:0130E8DC  PUSH      EBP   
019F:0130E8DD  MOV      EBP,ESP    
019F:0130E8DF  MOV      EAX,\[EBP+08\]            <- 参数\*\*   
019F:0130E8E2  TEST      EAX,EAX   
019F:0130E8E4  JNZ      0130E8ED             <- 跟踪可知，eax 不为 0 时，指向 dll 文件名   
019F:0130E8E6  MOV      EAX,\[01316624\]            <- 是基址：400000，就是程序的 Handle   
019F:0130E8EB  JMP      0130E8F3   
  
019F:0130E8ED  PUSH      EAX   
019F:0130E8EE  CALL      KERNEL32!GetModuleHandleA   
  
019F:0130E8F3  POP      EBP   
019F:0130E8F4  RET      0004   
  
\[0130E8DC\]=GetModuleHandleA   
  
  注意最后多弹出 4 字节的堆栈，原因是：无论是否执行到 0130E8EE，由于在前面 0040700D 处的参数入栈是多余的，必须要保持平衡。   
  
  根据上面程序看出，第一个函数是?GetModuleHandleA，下面是它的原型：   
  
The GetModuleHandle function returns a module handle for the specified module if the file has been mapped into the address space of the calling process.   
  
HMODULE GetModuleHandle(   
  
    LPCTSTR lpModuleName     // address of module name to return handle for    
  );       
  
Parameters   
  
lpModuleName   
  
Points to a null-terminated string that names a Win32 module (either a .DLL or .EXE file). If the filename extension is omitted, the default library extension .DLL is appended. The filename string can include a trailing point character (.) to indicate that the module name has no extension. The string does not have to specify a path. The name is compared (case independently) to the names of modules currently mapped into the address space of the calling process.   
If this parameter is NULL, GetModuleHandle returns a handle of the file used to create the calling process.   
  
  
Return Values   
  
If the function succeeds, the return value is a handle to the specified module.   
If the function fails, the return value is NULL. To get extended error information, call GetLastError. ?              ?                  ?                                      ?                                                                              ?                                                                                                                            019F:0130E930  CALL      KERNEL32!GetVersion    
  
  
(2)函数 2、3、4，aspr 的诱惑？？   
  
019F:0130E930  CALL      KERNEL32!GetVersion        <- do nothing...   
019F:0130E935  MOV      EAX,\[01316710\]         <- 倒追上面的 CALL 0130E70C，可知此处是 GetCurrentProcessID   
019F:0130E93A  RET    
019F:0130E93B  NOP   
  
\[0130E930\]=GetCurrentProcessID   
  
019F:0130E93C  PUSH      00   
019F:0130E93E  CALL      KERNEL32!GetModuleHandleA    <- do nothing...   
019F:0130E943  PUSH      DWORD PTR \[01316714\]        <- 倒追法，此处是 GetVersion   
019F:0130E949  POP      EAX   
019F:0130E94A  RET   
019F:0130E94B  RET   
  
\[0130E93C\]=GetVersion   
  
同样的方法可得到：   
\[0130E8F8\]=GetVersion   
  
(3)函数 5、6，什么都不作：   
    
019F:0130E94C  PUSH      EBP   
019F:0130E94D  MOV      EBP,ESP   
019F:0130E94F  CALL      KERNEL32!GetCurrentProcess   
019F:0130E954  MOV      EAX,\[EBP+08\]   
019F:0130E957  POP      EBP                                                      
019F:0130E958  RET      0004   
  
简化为：   
mov eax, dword ptr \[esp+4\]   
ret 04   
整个函数相当于弹出压栈的参数到 eax，即 pop eax。   
解决办法：在 ImpRec 中修复时选一个特殊一点的函数，比如 MakeCriticalSectionGlobal 之类的。修复完成后用 wdasm 反汇编，将对这个函数的调用改为 pop eax 和几个 nop 。   
  
\[0130E94C\]=pop eax   
  
下面这个也一样：   
                                                    
019F:0130E95C  PUSH      EBP                                                      
019F:0130E95D  MOV      EBP,ESP                                                  
019F:0130E95F  CALL      KERNEL32!GetVersion                                      
019F:0130E964  POP      EBP                                                      
019F:0130E965  RET      0004   
  
简化为：   
call GetVersion   
ret 04   
  
注意它与 GetVersion 并不等价，原因在于 ret 04 != ret。跟踪对此处的调用发现，返回的 eax 值立刻被 mov eax, esi 覆盖掉，所以这个也同样处理为 pop eax + Nop。   
  
\[0130E95C\]=pop eax   
  
(4)函数 7、8，aspr 的乾坤挪移？？   
  
我是第一次见到这种方式~~~先执行函数的几条指令，然后跳进函数体中继续~：   
  
019F:01321A0C  PUSH      EBP    
019F:01321A0D  MOV      EBP,ESP   
019F:01321A0F  PUSH      EBX   
019F:01321A10  JMP      BFF9369            <- 跳走   
  
跟下去会发现是到了这里：   
  
KERNEL32!MulDiv   
019F:BFF9368F  PUSH      EBP    
019F:BFF93690  MOV      EBP,ESP                                                  
019F:BFF93692  PUSH      EBX   
                                                    
019F:BFF93693  MOV      EBX,\[EBP+10\]            <- 在这里继续                                            
019F:BFF93696  MOV      ECX,EBX                                                  
019F:BFF93698  OR        EBX,EBX                                                  
019F:BFF9369A  JNS      BFF9369E                                                
019F:BFF9369C  NEG      EBX                                                      
019F:BFF9369E  MOV      EAX,\[EBP+08\]                                            
019F:BFF936A1  XOR      ECX,EAX                                                  
019F:BFF936A3  OR        EAX,EAX                                                  
019F:BFF936A5  JNS      BFF936A9                                                
019F:BFF936A7  NEG      EAX        
019F:BFF936A9  MOV      EDX,\[EBP+0C\]   
  
...   
  
\[01321A0C\]=MulDiv   
  
同样：   
  
019F:01321A1C  PUSH      EBP    
019F:01321A1D  MOV      EBP,ESP   
019F:01321A1F  PUSH      EBX                                                      
019F:01321A20  JMP      BFF93693   
  
\[01321A1C\]=MulDiv   
  
  这种方式修复起来没有任何难度，而且也降低了加壳后程序运行的兼容性，是作者作出的一种尝试。   
  
  上述修复完毕，脱后的文件变成未注册版，可正常运行，没有次数限制，但每次有 NAG，软件也不想用，懒得跟下去。谁知道使用次数是存在哪的，半天也找不到:(   
    
  希望能对大家水平提高有所帮助:) 

**编者注：此壳可以用 Import REConstructor1.42+修复 IAT。本文提供了如用 Import REConstructor1.42+不能发现正确的函数时，手动修复的方法。**

 4dd466d0e1e0cc123afbe504da2462ca