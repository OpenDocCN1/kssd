

# jingulong's unpackme NO.2 外壳分析

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-592.htm`](https://www.pediy.com/kssd/pediy07/pediy7-592.htm)

JGL's UnpackMe v2 外壳分析  
  
【目    标】:JGL's UnpackMe 2  
【工    具】:Olydbg1.1(diy 版)、LORDPE、ImportREC1.6F IDA4.7  
【任    务】:分析外壳    
【操作平台】:Windows 2003 server   
【作     者】: LOVEBOOM\[DFCG\]\[FCG\]\[US\]  
【相关链接】:[`www.pediy.com/bbs/pediy7/pediy7-592.rar`](http://www.pediy.com/bbs/pediy7/pediy7-592.rar)  
【简要说明】: 近日心神不定，昨夜恶梦连连，今晨头仍晕晕。随便分析下外壳让脑袋醒醒。  
【详细过程】:  
    一般外壳分析的文章，我就不多讲什么几个 F7 之类的，如果你没有那么多时间或兴趣，就休  
  
息下看别的东西。  
我也是边脱边写，所以呢，当中可能会有很多自己水平不足造成的错误。如你发现欢迎指正:-)。  
od 载入:  
005D90C5 >  E8 02000000     CALL 005D90CC                            ; EP  
005D90CA    98              CWDE  
005D90CB    4C              DEC ESP  
......  
经过两次解压后执行部分代码:  
  
sag008:005D96C5 ; ------------------------------------------------------------------  
  
\----------  
seg008:005D96C5                 push    ebp  
seg008:005D96C6                 mov     eax, 0FFBFFFFFh  
seg008:005D96CB                 mov     ecx, 400h  
seg008:005D96D0                 xor     eax, 0FFFFFFFFh                ; 计算  
  
出 ImageBase,和很多外壳有点不同的方式:-)  
seg008:005D96D5                 or      edx, 0FFFFFFFFh  
seg008:005D96D8  
seg008:005D96D8 loc\_5D96D8:                                            ; CODE XREF:   
  
seg008:005D96DEj  
seg008:005D96D8                 xor     dl, \[eax+ecx\-1\]                ; 计算 PeHeader  
  
的值:1C780A22,这个值是关键 Key 来的  
seg008:005D96DC                 rol     edx, cl  
seg008:005D96DE                 loop    loc\_5D96D8                     ; 计算 PeHeader  
  
的值:1C780A22,这个值是关键 Key 来的  
seg008:005D96E0                 push    edx  
seg008:005D96E1                 call    GETAPIADDR  
这里面执行 SetUnhandledExceptionFilter 之前先 patch 一下，然后记下传入地  
  
址 pTopLevelFilter = JGL's\_Un.005D9BA1  
否则后面的 INT 3 无法通过的。      
seg008:005D96E6                 push    edx                            ; Push szKey  
seg008:005D96E7                 push    0  
seg008:005D96E9                 call    DeCode                         ; 这里进去处理  
  
部分 NT 函数和解压代码  
解压后:  
seg008:005D96EE                 cld  
seg008:005D96EF                 mov     ebp, offset off\_5D9028 ; CODE XREF:   
  
seg008:005D9753j  
seg008:005D96F4                 push    4               ; Protect = PAGE\_READWRITE  
seg008:005D96F6                 push    3000h           ; AllocationType =   
  
MEM\_COMMIT|MEM\_RESERVE  
seg008:005D96FB                 push    4000h           ; Size = 4000 (16384.)  
seg008:005D9700                 push    ecx             ; Address = NULL  
seg008:005D9701                 mov     edx, \[ebp+4\]  
seg008:005D9704                 cmp     byte ptr \[edx\], 0CCh ; 一个比较有新意的判断有  
seg008:005D9704                                         ; 没有在相关 api 下 CC 断的方法  
seg008:005D9707                 setz    cl  
seg008:005D970A                 add     edx, ecx        ; CODE XREF:   
  
seg008:005D974Fj  
seg008:005D970A                                         ; 如果下了 CC 断 cl 为 1 则跳去 over  
  
了  
seg008:005D970C                 call    edx             ; VirtualAlloc 申请空间  
seg008:005D970E                 lea     edx, \[ebp+4088h\] ; 5DD0B0  
seg008:005D9714                 push    eax  
seg008:005D9715                 push    edx  
seg008:005D9716                 push    eax  
seg008:005D9717                 call    $+5  
seg008:005D971C                 add     dword ptr \[esp\], 8  
seg008:005D9720                 retn  
seg008:005D9720 ; ------------------------------------------------------------------  
  
\----------  
seg008:005D9721                 db 0B8h ; ?  
  
seg008:005D9722 ; ------------------------------------------------------------------  
  
\----------  
seg008:005D9722                 jmp     short loc\_5D9726  
seg008:005D9722 ; ------------------------------------------------------------------  
  
\----------  
seg008:005D9724                 db 0CCh ; ?            ; 如果没有 Patch 会在这个 Int3 处  
  
无法绕过,  
seg008:005D9725                 db 0CCh ; ?  
  
seg008:005D9726 ; ------------------------------------------------------------------  
  
\----------  
如果前面 patch 了，直接在 5D9BA1 处下断。  
seg008:005D9BA1 ; ------------------------------------------------------------------  
  
\----------  
seg008:005D9BA1                 mov     edx, \[esp+4\]  
seg008:005D9BA5                 mov     edx, \[edx+4\]  
seg008:005D9BA8                 mov     eax, \[edx+0B8h\]           ; 获取异常地址  
seg008:005D9BAE                 call    $+5  
seg008:005D9BB3                 pop     ecx  
seg008:005D9BB4                 and     cx, 0F000h  
seg008:005D9BB9                 add     ecx, 66h  
seg008:005D9BBF                 mov     \[ecx\], eax                ; 保存异常地址  
seg008:005D9BC1                 inc     eax  
seg008:005D9BC2                 inc     eax  
seg008:005D9BC3                 mov     \[edx+0B8h\], eax           ; 异常地址处有两  
  
个 int 3,改为异常地址+2。  
seg008:005D9BC9                 call    Decrypt\_Code      ; 这里进去关键，外  
  
壳的解密部分  
seg008:005D9BCE                 xor     eax, eax  
seg008:005D9BD0                 dec     eax  
seg008:005D9BD1                 retn    4  
seg008:005D9BD1 ; ------------------------------------------------------------------  
  
\----------  
进去后有两个地方会动态改变，第一次就是解密后的获取 api 的地址后，会把相关名字给清除掉，  
  
第二处就是壳通过  
逽名管道，把子进程建立后，通过同步事件把关键代码写入子进程，然后子进程执行关键代码。  
第一处：  
005DA22C    E8 3C020000     CALL 005DA46D  
005DA231    3215 120F121A   XOR DL,BYTE PTR DS:\[1A120F12\]  
005DA237    17              POP SS                                   ; Modification   
  
of segment register  
通过后面的分析写出解密代码:  
  PUSHAD  
  MOV ESI,005DA231  
  MOV EDI,ESI  
L003:  
  LODS BYTE PTR DS:\[ESI\]  
  CMP AL,0  
  JE L009  
  XOR AL,7B  
  STOS BYTE PTR ES:\[EDI\]  
  JMP L003  
L009:  
  CMP BYTE PTR DS:\[ESI\],0  
  NOP  
  JE L017  
  NOP  
  INC EDI  
  JMP L003  
  NOP  
  NOP  
L017:  
  POPAD  
  
第二处:  
005DA747    E8 0D000000     CALL 005DA759  
005DA74C    41              INC ECX  
005DA74D    44              INC ESP  
005DA74E    56              PUSH ESI  
  
第二处的分析结果如下：  
  
  第一次解密地址:5DA7E9  size:0F0   KEY:06  
  
  第二次解密地址:5DA8D9  size:0F0   KEY:05  
  
  第三次解密地址:5DA9C9  size:0F0   KEY:04  
  
  第三次解密地址:5DAAB9  size:0F0   KEY:03  
  
  第四次解密地址:5DABA9  size:0F0   KEY:02  
  
  第五次解密地址:5DAC99  size:0F0   KEY:01  
  
  第六次解密地址:5DAD89  size:0C9   KEY:C9  
  
    根据分析再写一段解密代码，写这两段代码都是为了方便用 IDA 分析外壳，  
如果只是为了一般的脱壳，没有必要这样做,解密代码：  
  PUSHAD  
  MOV EBX,6  
  MOV ECX,0F0  
  MOV ESI,005DA7E9  
  MOV EDI,ESI  
L005:  
  LODS BYTE PTR DS:\[ESI\]  
  XOR AL,BL  
  STOS BYTE PTR ES:\[EDI\]  
  LOOPD L005  
  CMP EBX,1  
  JE L014  
  DEC EBX  
  MOV ECX,0F0  
  JMP L005  
L014:  
  MOV EBX,0C9  
  MOV ECX,EBX  
L016:  
  LODS BYTE PTR DS:\[ESI\]  
  XOR AL,BL  
  STOS BYTE PTR ES:\[EDI\]  
  LOOPD L016  
  POPAD  
  
解密出来了，用 IDA 分析就好很多了,内容比较长，慢慢看:-)：  
  
seg008:005DA1BA Decrypt\_Code    proc far                               ; CODE XREF:   
  
seg008:005D9BC9p  
seg008:005DA1BA  
seg008:005DA1BA var\_16B0        = dword ptr -16B0h  
seg008:005DA1BA var\_pSecurity   = dword ptr -0A04h  
seg008:005DA1BA var\_A00         = dword ptr -0A00h  
seg008:005DA1BA var\_9FC         = dword ptr -9FCh  
seg008:005DA1BA var\_KERBASE     = dword ptr -9F8h  
seg008:005DA1BA var\_hUser32     = dword ptr -9F4h  
seg008:005DA1BA LookupPrivilegeValueA= dword ptr -9F0h  
seg008:005DA1BA var\_hGDI32      = dword ptr -9ECh  
seg008:005DA1BA var\_9E8         = dword ptr -9E8h  
seg008:005DA1BA var\_9E4         = dword ptr -9E4h  
seg008:005DA1BA var\_9E0         = dword ptr -9E0h  
seg008:005DA1BA var\_PEB         = dword ptr -9DCh  
seg008:005DA1BA var\_PEB1        = dword ptr -9D8h  
seg008:005DA1BA var\_9D0         = dword ptr -9D0h  
seg008:005DA1BA var\_9CC         = dword ptr -9CCh  
seg008:005DA1BA var\_App\_hModule = dword ptr -9C8h  
seg008:005DA1BA var\_9C4         = dword ptr -9C4h  
seg008:005DA1BA var\_9C0         = dword ptr -9C0h  
seg008:005DA1BA var\_9BC         = dword ptr -9BCh  
seg008:005DA1BA var\_9B8         = dword ptr -9B8h  
seg008:005DA1BA var\_9B4         = dword ptr -9B4h  
seg008:005DA1BA var\_GetProcAddress= dword ptr -9A4h  
seg008:005DA1BA var\_hLocal      = dword ptr -998h  
seg008:005DA1BA var\_994         = dword ptr -994h  
seg008:005DA1BA var\_990         = dword ptr -990h  
seg008:005DA1BA var\_LoadLibraryA= dword ptr -98Ch  
seg008:005DA1BA var\_988         = dword ptr -988h  
seg008:005DA1BA var\_hMemLocal   = dword ptr -984h  
seg008:005DA1BA var\_CmdLen      = dword ptr -980h  
seg008:005DA1BA var\_97C         = dword ptr -97Ch  
seg008:005DA1BA var\_CurrentPID  = dword ptr -978h  
seg008:005DA1BA var\_hEvent      = dword ptr -974h  
seg008:005DA1BA var\_hMap        = dword ptr -970h  
seg008:005DA1BA var\_hMem        = dword ptr -96Ch  
seg008:005DA1BA var\_CodeBase    = dword ptr -968h  
seg008:005DA1BA var\_964         = dword ptr -964h  
seg008:005DA1BA var\_hWrite2     = dword ptr -960h  
seg008:005DA1BA var\_hRead2      = dword ptr -95Ch  
seg008:005DA1BA var\_hWrite1     = dword ptr -958h  
seg008:005DA1BA var\_hRead1      = dword ptr -954h  
seg008:005DA1BA \_InitCriticalSection= dword ptr -950h  
seg008:005DA1BA var\_GetCurrentThread= dword ptr -93Ch  
seg008:005DA1BA var\_LocalAlloc  = dword ptr -934h  
seg008:005DA1BA var\_ContinueDebugEvent= dword ptr -92Ch  
seg008:005DA1BA var\_WaitForDebugEvent= dword ptr -928h  
seg008:005DA1BA var\_GetThreadContext= dword ptr -924h  
seg008:005DA1BA var\_SetThreadContext= dword ptr -920h  
seg008:005DA1BA var\_ReadProcMemory= dword ptr -91Ch  
seg008:005DA1BA var\_918         = dword ptr -918h  
seg008:005DA1BA var\_DebugActiveProcess= dword ptr -914h  
seg008:005DA1BA var\_910         = dword ptr -910h  
seg008:005DA1BA var\_SetEvent    = dword ptr -90Ch  
seg008:005DA1BA var\_908         = dword ptr -908h  
seg008:005DA1BA var\_Sleep       = dword ptr -904h  
seg008:005DA1BA var\_WaitForsingleObject= dword ptr -900h  
seg008:005DA1BA var\_GetStartupInfoA= dword ptr -8F8h  
seg008:005DA1BA var\_CreateProcessA= dword ptr -8F4h  
seg008:005DA1BA var\_CloseHandle = dword ptr -8F0h  
seg008:005DA1BA var\_CreatePipe  = dword ptr -8ECh  
seg008:005DA1BA var\_CreateEventA= dword ptr -8E8h  
seg008:005DA1BA var\_GetCurrentProcessId= dword ptr -8E4h  
seg008:005DA1BA GetCurrentProc  = dword ptr -8E0h  
seg008:005DA1BA var\_MapViewOfFile= dword ptr -8DCh  
seg008:005DA1BA var\_GetWin32LastError= dword ptr -8D8h  
seg008:005DA1BA var\_CreateFileMappingA= dword ptr -8D4h  
seg008:005DA1BA var\_RtlZeroMemory= dword ptr -8D0h  
seg008:005DA1BA var\_GetModuleFileNameA= dword ptr -8CCh  
seg008:005DA1BA var\_GetModuleHandleA= dword ptr -8C8h  
seg008:005DA1BA var\_GetCommandLineA= dword ptr -8C4h  
seg008:005DA1BA var\_ReSetEvent  = dword ptr -8C0h  
seg008:005DA1BA var\_pContext    = dword ptr -8BCh  
seg008:005DA1BA var\_80C         = dword ptr -80Ch  
seg008:005DA1BA var\_ExcepAddress= dword ptr -804h  
seg008:005DA1BA var\_pBaseAddress= dword ptr -7F8h  
seg008:005DA1BA var\_DebugEvent  = dword ptr -5E8h  
seg008:005DA1BA var\_ProcessId   = dword ptr -5E4h  
seg008:005DA1BA var\_ThreadId    = dword ptr -5E0h  
seg008:005DA1BA var\_Exception\_Code= dword ptr -5DCh  
seg008:005DA1BA var\_5D8         = dword ptr -5D8h  
seg008:005DA1BA var\_pThread     = dword ptr -5D4h  
seg008:005DA1BA var\_hGDI321     = dword ptr -588h  
seg008:005DA1BA var\_RepPath     = dword ptr -578h  
seg008:005DA1BA var\_Cstr(PID)   = dword ptr -478h  
seg008:005DA1BA var\_hProcess    = dword ptr -470h  
seg008:005DA1BA var\_hThread     = dword ptr -46Ch  
seg008:005DA1BA var\_460         = dword ptr -460h  
seg008:005DA1BA var\_410         = dword ptr -410h  
seg008:005DA1BA var\_RlZMemDestination= dword ptr -400h  
seg008:005DA1BA arg\_40          = qword ptr  4Ch  
seg008:005DA1BA arg\_36FFF738    = dword ptr  36FFF744h  
seg008:005DA1BA arg\_3FA5743E    = byte ptr  3FA5744Ah  
seg008:005DA1BA  
seg008:005DA1BA                 push    ebp  
seg008:005DA1BB                 mov     ebp, esp  
seg008:005DA1BD                 add     esp, 0FFFFF5FCh  
seg008:005DA1C3                 push    esi  
seg008:005DA1C4                 push    edi  
seg008:005DA1C5                 push    ebx  
seg008:005DA1C6                 push    edx  
seg008:005DA1C7                 xor     eax, eax  
seg008:005DA1C9                 mov     \[ebp+var\_hGDI321\], eax         ; 初始化相关变  
  
量  
seg008:005DA1CF                 mov     \[ebp+var\_990\], eax  
seg008:005DA1D5                 mov     \[ebp+var\_9CC\], eax  
seg008:005DA1DB                 mov     \[ebp+var\_RepPath\], eax  
seg008:005DA1E1                 mov     \[ebp+var\_994\], eax  
seg008:005DA1E7                 call    GetCodeBase  
seg008:005DA1EC                 jmp     short loc\_5DA1F3  
seg008:005DA1EE ; ------------------------------------------------------------------  
  
\----------  
seg008:005DA1EE  
seg008:005DA1EE loc\_5DA1EE:                                            ; CODE XREF:   
  
Decrypt\_Code+3Cj  
seg008:005DA1EE                 sub     eax, 1000h  
seg008:005DA1F3  
seg008:005DA1F3 loc\_5DA1F3:                                            ; CODE XREF:   
  
Decrypt\_Code+32j  
seg008:005DA1F3                 cmp     byte ptr \[eax\], 28h  
seg008:005DA1F6                 jnz     short loc\_5DA1EE  
seg008:005DA1F8                 mov     esi, eax  
seg008:005DA1FA                 mov     edi, eax  
seg008:005DA1FC                 mov     \[ebp+var\_CodeBase\], eax  
seg008:005DA202                 push    dword ptr \[eax+28h\]  
seg008:005DA205                 pop     \[ebp+var\_LoadLibraryA\]  
seg008:005DA20B                 add     edi, 62h  
seg008:005DA211                 push    dword ptr \[edi\]  
seg008:005DA213                 pop     \[ebp+var\_KERBASE\]  
seg008:005DA219                 mov     eax, \[ebp+var\_CodeBase\]  
seg008:005DA21F                 add     eax, 5Eh  
seg008:005DA224                 push    dword ptr \[eax\]  
seg008:005DA226                 pop     \[ebp+var\_PEB\]  
seg008:005DA22C                 call    @F  
seg008:005DA22C ; ------------------------------------------------------------------  
  
\----------  
seg008:005DA231 aInitializecrit db 'InitializeCriticalSection',0       ; 这些字符串是  
  
解密后的  
seg008:005DA24B aWritefile      db 'WriteFile',0  
seg008:005DA255 aLeavecriticals db 'LeaveCriticalSection',0  
seg008:005DA26A aEntercriticals db 'EnterCriticalSection',0  
seg008:005DA27F aReadfileo      db 'ReadFile',0  
seg008:005DA288 aGetcurrentthre db 'GetCurrentThread',0  
seg008:005DA299 aLocalfree      db 'LocalFree',0  
seg008:005DA2A3 aLocalalloc     db 'LocalAlloc',0  
seg008:005DA2AE aOpenthread     db 'OpenThread',0  
seg008:005DA2B9 aContinuedebuge db 'ContinueDebugEvent',0  
seg008:005DA2CC aWaitfordebugev db 'WaitForDebugEvent',0  
seg008:005DA2DE aGetthreadconte db 'GetThreadContext',0  
seg008:005DA2EF aSetthreadconte db 'SetThreadContext',0  
seg008:005DA300 aReadprocessmem db 'ReadProcessMemory',0  
seg008:005DA312 aWriteprocessme db 'WriteProcessMemory',0  
seg008:005DA325 aDebugactivepro db 'DebugActiveProcess',0  
seg008:005DA338 aOpenprocess    db 'OpenProcess',0  
seg008:005DA344 aSetevent       db 'SetEvent',0  
seg008:005DA34D aDuplicatehandl db 'DuplicateHandle',0  
seg008:005DA35D aSleep          db 'Sleep',0  
seg008:005DA363 aWaitforsingleo db 'WaitForSingleObject',0  
seg008:005DA377 aUnmapviewoffil db 'UnmapViewOfFile',0  
seg008:005DA387 aGetstartupinfo db 'GetStartupInfoA',0  
seg008:005DA397 aCreateprocessa db 'CreateProcessA',0  
seg008:005DA3A6 aClosehandle    db 'CloseHandle',0  
seg008:005DA3B2 aCreatepipe     db 'CreatePipe',0  
seg008:005DA3BD aCreateeventa   db 'CreateEventA',0  
seg008:005DA3CA aGetcurrentproc db 'GetCurrentProcessId',0  
seg008:005DA3DE aGetcurrentpr\_0 db 'GetCurrentProcess',0  
seg008:005DA3F0 aMapviewoffile  db 'MapViewOfFile',0  
seg008:005DA3FE aGetlasterror   db 'GetLastError',0  
seg008:005DA40B aCreatefilemapp db 'CreateFileMappingA',0  
seg008:005DA41E aRtlzeromemory  db 'RtlZeroMemory',0  
seg008:005DA42C aGetmodulefilen db 'GetModuleFileNameA',0  
seg008:005DA43F aGetmodulehandl db 'GetModuleHandleA',0  
seg008:005DA450 aGetcommandline db 'GetCommandLineA',0  
seg008:005DA460 aResetevent     db 'ResetEvent',0  
seg008:005DA46B                 db    0  
seg008:005DA46C                 db    0  
seg008:005DA46D ; ------------------------------------------------------------------  
  
\----------  
seg008:005DA46D  
seg008:005DA46D @F:                                                    ; CODE XREF:   
  
Decrypt\_Code+72p  
seg008:005DA46D                 mov     edx, \[edi\]  
seg008:005DA46F                 pop     edi  
seg008:005DA470                 push    dword ptr \[esi+24h\]  
seg008:005DA473                 pop     \[ebp+var\_GetProcAddress\]  
seg008:005DA479                 lea     esi, \[ebp+\_InitCriticalSection\]  
seg008:005DA47F  
seg008:005DA47F loc\_5DA47F:                                            ; CODE XREF:   
  
Decrypt\_Code+309j  
seg008:005DA47F                 push    edx                            ; 这里循环解密  
  
出正确的字符串,Key=7B  
seg008:005DA480                 xor     ecx, ecx  
seg008:005DA482  
seg008:005DA482 loc\_5DA482:                                            ; CODE XREF:   
  
Decrypt\_Code+2D1j  
seg008:005DA482                 xor     byte ptr \[ecx+edi\], '{'  
seg008:005DA486                 inc     ecx  
seg008:005DA487                 cmp     byte ptr \[ecx+edi\], 0  
seg008:005DA48B                 jnz     short loc\_5DA482  
seg008:005DA48D                 push    edi                            ; push apiName  
seg008:005DA48E                 push    edx                            ; push hModule  
seg008:005DA48F  
seg008:005DA48F loc\_5DA48F:                                            ;   
  
**GetProcAddress** 获取 api 的地址  
seg008:005DA48F                 call    \[ebp+var\_GetProcAddress\]  
seg008:005DA495                 mov     \[esi\], eax                     ; 保存 api 地址  
seg008:005DA497                 push    eax  
seg008:005DA498                 push    eax  
seg008:005DA499                 call    sub\_5DB6B3  
seg008:005DA49E                 cmp     eax, 0FFFFFFFFh  
seg008:005DA4A1                 jz      loc\_5DA81A  
seg008:005DA4A7                 lodsd  
seg008:005DA4A8                 xchg    esi, \[ebp+var\_PEB\]  
seg008:005DA4AE  
seg008:005DA4AE loc\_5DA4AE:                                            ; CODE XREF:   
  
Decrypt\_Code+2F9j  
seg008:005DA4AE                 lodsb  
seg008:005DA4AF                 stosb  
seg008:005DA4B0                 cmp     byte ptr \[edi\], 0  
seg008:005DA4B3                 jnz     short loc\_5DA4AE               ; 获取了函数的  
  
API 后，相关地址保存 PEB 的内容  
seg008:005DA4B5                 lodsb  
seg008:005DA4B6                 stosb  
seg008:005DA4B7                 xchg    esi, \[ebp+var\_PEB\]  
seg008:005DA4BD                 pop     edx  
seg008:005DA4BE                 cmp     byte ptr \[edi\], 0  
seg008:005DA4C1                 jz      short loc\_5DA4C5               ; 如果相关 api  
  
的地址获取完毕则跳去下一步  
seg008:005DA4C3                 jmp     short loc\_5DA47F               ; 这里循环解密  
  
出正确的字符串,Key=7B  
seg008:005DA4C5 ; ------------------------------------------------------------------  
  
\----------  
seg008:005DA4C5  
seg008:005DA4C5 loc\_5DA4C5:                                            ; CODE XREF:   
  
Decrypt\_Code+307j  
seg008:005DA4C5                 push    esi  
seg008:005DA4C6                 push    edi  
seg008:005DA4C7                 push    400h  
seg008:005DA4CC                 lea     esi, \[ebp+var\_RlZMemDestination\]  
seg008:005DA4D2                 push    esi  
seg008:005DA4D3                 call    \[ebp+var\_RtlZeroMemory\]        ;   
  
RtlZeroMemory　初始化内存空间  
seg008:005DA4D3                                                        ;  
seg008:005DA4D9                 push    400h  
seg008:005DA4DE                 push    esi  
seg008:005DA4DF                 push    0  
seg008:005DA4E1                 call    \[ebp+var\_GetModuleHandleA\]     ; 获取程序  
  
的 ImageBase  
seg008:005DA4E7                 mov     \[ebp+var\_App\_hModule\], eax  
seg008:005DA4ED                 push    eax  
seg008:005DA4EE                 call    \[ebp+var\_GetModuleFileNameA\]  
seg008:005DA4F4                 lea     edi, \[ebp+var\_RepPath\]  
seg008:005DA4FA                 lodsb  
seg008:005DA4FB  
seg008:005DA4FB loc\_5DA4FB:                                            ; CODE XREF:   
  
Decrypt\_Code+34Bj  
seg008:005DA4FB                 cmp     al, ''                        ; 把完整路径名  
  
中的\\替换为\_.  
seg008:005DA4FD                 jnz     short loc\_5DA501  
seg008:005DA4FF                 mov     al, '\_'  
seg008:005DA501  
seg008:005DA501 loc\_5DA501:                                            ; CODE XREF:   
  
Decrypt\_Code+343j  
seg008:005DA501                 stosb  
seg008:005DA502                 lodsb  
seg008:005DA503                 or      al, al  
seg008:005DA505                 jnz     short loc\_5DA4FB               ; 如果没有替换  
  
完则继续  
seg008:005DA507                 stosb  
seg008:005DA508                 call    \[ebp+var\_GetCurrentProcessId\]  
seg008:005DA50E                 mov     \[ebp+var\_CurrentPID\], eax      ; 保存当前进程  
  
ID  
seg008:005DA514                 call    \[ebp+var\_GetWin32LastError\]  
seg008:005DA51A                 mov     edi, eax  
seg008:005DA51C                 lea     esi, \[ebp+var\_Cstr(PID)\]  
seg008:005DA522                 push    esi  
seg008:005DA523                 push    \[ebp+var\_CurrentPID\]  
seg008:005DA529                 call    Cstr\_PID\_                      ; 把当前进程 ID  
  
转为字符串  
seg008:005DA529                                                        ; 类似 wsprintf  
  
把十六进制转为字符串  
seg008:005DA52E                 call    \[ebp+var\_GetCommandLineA\]  
seg008:005DA534                 xor     ecx, ecx  
seg008:005DA536                 mov     esi, eax  
seg008:005DA538                 call    \[ebp+var\_GetWin32LastError\]  
seg008:005DA53E                 mov     edi, eax  
seg008:005DA540  
seg008:005DA540 loc\_5DA540:                                            ; CODE XREF:   
  
Decrypt\_Code+392j  
seg008:005DA540                 lodsb                                  ; 循环计算命令  
  
行长度,去除前后两个"号  
seg008:005DA541                 cmp     al, 22h  
seg008:005DA543                 jnz     short loc\_5DA549  
seg008:005DA545                 or      al, al  
seg008:005DA547                 jz      short loc\_5DA54A  
seg008:005DA549  
seg008:005DA549 loc\_5DA549:                                            ; CODE XREF:   
  
Decrypt\_Code+389j  
seg008:005DA549                 inc     ecx  
seg008:005DA54A  
seg008:005DA54A loc\_5DA54A:                                            ; CODE XREF:   
  
Decrypt\_Code+38Dj  
seg008:005DA54A                 or      al, al  
seg008:005DA54C                 jnz     short loc\_5DA540               ; 循环计算命令  
  
行长度,去除前后两个"号  
seg008:005DA54E                 mov     \[ebp+var\_CmdLen\], ecx  
seg008:005DA554                 lea     esi, \[ebp+var\_RepPath\]  
seg008:005DA55A                 push    esi                            ; /MapName =   
  
"D:\_Documents and Settings\_  
seg008:005DA55B                 push    400h                           ;   
  
|MaximumSizeLow = 400  
seg008:005DA560                 push    0                              ;   
  
|MaximumSizeHigh = 0  
seg008:005DA562                 push    4                              ; |Protection   
  
\= PAGE\_READWRITE  
seg008:005DA564                 push    0                              ; |pSecurity =   
  
NULL  
seg008:005DA566                 push    0FFFFFFFFh                     ; |hFile =   
  
FFFFFFFF  
seg008:005DA568                 call    \[ebp+var\_CreateFileMappingA\]   ;   
  
\\CreateFileMappingA  
seg008:005DA56E                 mov     \[ebp+var\_hMem\], eax  
seg008:005DA574                 call    \[ebp+var\_GetWin32LastError\]    ; 如果是子进程  
  
则返回 0B7  
seg008:005DA57A                 mov     edi, eax  
seg008:005DA57C                 push    0                              ; /MapSize = 0  
seg008:005DA57E                 push    0                              ; |OffsetLow =   
  
0  
seg008:005DA580                 push    0                              ; |OffsetHigh   
  
\= 0  
seg008:005DA582                 push    0F001Fh                        ; |AccessMode   
  
\= F001F  
seg008:005DA587                 push    \[ebp+var\_hMem\]                 ; |hMapObject   
  
\= 0000001C  
seg008:005DA58D                 call    \[ebp+var\_MapViewOfFile\]        ;   
  
\\MapViewOfFile  
seg008:005DA593                 push    eax  
seg008:005DA594                 mov     eax, \[ebp+var\_CodeBase\]  
seg008:005DA59A                 add     eax, 8Eh  
seg008:005DA59F                 pop     dword ptr \[eax\]  
seg008:005DA5A1                 push    dword ptr \[eax\]  
seg008:005DA5A3                 pop     \[ebp+var\_hMap\]  
seg008:005DA5A9                 cmp     edi, 0B7h  
seg008:005DA5AF                 jnz     loc\_5DAE58                     ; 这里跳就去父  
  
进程处理部分  
seg008:005DA5B5                 mov     esi, \[ebp+var\_hMap\]  
seg008:005DA5BB                 add     esi, 100h  
seg008:005DA5C1                 lodsd  
seg008:005DA5C2                 lea     esi, \[ebp+var\_Cstr(PID)\]  
seg008:005DA5C8                 push    esi  
seg008:005DA5C9                 push    eax  
seg008:005DA5CA                 call    Cstr\_PID\_  
seg008:005DA5CF                 push    edi  
seg008:005DA5D0                 call    loc\_5DA5E6  
seg008:005DA5D0 ; ------------------------------------------------------------------  
  
\----------  
seg008:005DA5D5 aPelock\_event\_1 db 'PeLock\_Event\_123',0  
seg008:005DA5E6 ; ------------------------------------------------------------------  
  
\----------  
seg008:005DA5E6  
seg008:005DA5E6 loc\_5DA5E6:                                            ; CODE XREF:   
  
Decrypt\_Code+416p  
seg008:005DA5E6                 pop     edi  
seg008:005DA5E7                 push    edi  
seg008:005DA5E8                 xor     eax, eax  
seg008:005DA5EA                 push    0FFFFFFFFh  
seg008:005DA5EC                 pop     ecx  
seg008:005DA5ED                 repne scasb  
seg008:005DA5EF                 sub     edi, 4  
seg008:005DA5F2                 lea     esi, \[ebp+var\_Cstr(PID)\]  
seg008:005DA5F8  
seg008:005DA5F8 loc\_5DA5F8:                                            ; CODE XREF:   
  
Decrypt\_Code+442j  
seg008:005DA5F8                 lodsb  
seg008:005DA5F9                 stosb  
seg008:005DA5FA                 or      al, al  
seg008:005DA5FC                 jnz     short loc\_5DA5F8  
seg008:005DA5FE                 call    \[ebp+GetCurrentProc\]  
seg008:005DA604                 mov     edi, eax  
seg008:005DA606                 push    1  
seg008:005DA608                 push    1  
seg008:005DA60A                 push    0  
seg008:005DA60C                 call    \[ebp+var\_CreateEventA\]  
seg008:005DA612                 mov     \[ebp+var\_hEvent\], eax  
seg008:005DA618                 push    0  
seg008:005DA61A                 push    \[ebp+var\_hEvent\]  
seg008:005DA620                 call    \[ebp+var\_WaitForsingleObject\]  
seg008:005DA626                 cmp     eax, 102h  
seg008:005DA62B                 jz      short loc\_5DA632  
seg008:005DA62D                 call    loc\_5D9B1E  
seg008:005DA632  
seg008:005DA632 loc\_5DA632:                                            ; CODE XREF:   
  
Decrypt\_Code+471j  
seg008:005DA632                 call    \[ebp+var\_GetWin32LastError\]  
seg008:005DA638                 mov     \[ebp+var\_9D0\], eax  
seg008:005DA63E  
seg008:005DA63E loc\_5DA63E:  
seg008:005DA63E                 mov     esi, \[ebp+var\_hMap\]  
seg008:005DA644                 add     esi, 100h  
seg008:005DA64A                 lodsd  
seg008:005DA64B                 mov     \[ebp+var\_97C\], eax  
seg008:005DA651                 push    eax  
seg008:005DA652                 push    0  
seg008:005DA654                 push    1F0FFFh  
seg008:005DA659                 call    \[ebp+var\_910\]  
seg008:005DA65F                 mov     \[ebp+var\_964\], eax  
seg008:005DA665                 push    2  
seg008:005DA667                 push    1  
seg008:005DA669                 push    0  
seg008:005DA66B                 lea     ecx, \[ebp+var\_hRead2\]  
seg008:005DA671                 push    ecx  
seg008:005DA672                 push    edi  
seg008:005DA673                 add     esi, 7Ch  
seg008:005DA676                 lodsd  
seg008:005DA677                 push    eax  
seg008:005DA678                 push    \[ebp+var\_964\]  
seg008:005DA67E                 call    \[ebp+var\_908\]  
seg008:005DA684                 push    \[ebp+var\_hRead2\]  
seg008:005DA68A                 pop     \[ebp+var\_hRead1\]  
seg008:005DA690                 push    2  
seg008:005DA692                 push    1  
seg008:005DA694                 push    0  
seg008:005DA696                 lea     ecx, \[ebp+var\_hWrite1\]  
seg008:005DA69C                 push    ecx  
seg008:005DA69D                 push    edi  
seg008:005DA69E                 lodsd  
seg008:005DA69F                 push    eax  
seg008:005DA6A0                 push    \[ebp+var\_964\]  
seg008:005DA6A6                 call    \[ebp+var\_908\]  
seg008:005DA6AC                 push    \[ebp+var\_hWrite1\]  
seg008:005DA6B2                 pop     \[ebp+var\_hWrite2\]  
seg008:005DA6B8                 push    \[ebp+var\_964\]  
seg008:005DA6BE                 call    \[ebp+var\_CloseHandle\]  
seg008:005DA6C4                 mov     eax, \[ebp+var\_CodeBase\]  
seg008:005DA6CA                 add     eax, 96h  
seg008:005DA6CF                 push    eax  
seg008:005DA6D0                 call    \[ebp+\_InitCriticalSection\]  
seg008:005DA6D6                 push    \[ebp+var\_hEvent\]  
seg008:005DA6DC                 call    \[ebp+var\_SetEvent\]  
seg008:005DA6E2                 mov     edi, \[ebp+var\_CodeBase\]  
seg008:005DA6E8                 add     edi, 17E9h  
seg008:005DA6EE                 mov     eax, 669h  
seg008:005DA6F3                 xor     edx, edx  
seg008:005DA6F5                 mov     ecx, 0F0h  
seg008:005DA6FA                 mov     \[ebp+var\_9E0\], 102h  
seg008:005DA704                 mov     \[ebp+var\_9E4\], ecx  
seg008:005DA70A                 div     ecx  
seg008:005DA70C                 mov     ecx, eax  
seg008:005DA70E                 push    edx  
seg008:005DA70F  
seg008:005DA70F loc\_5DA70F:                                            ; CODE XREF:   
  
Decrypt\_Code+576j  
seg008:005DA70F                 push    ecx  
seg008:005DA710                 lea     eax, \[ebp+var\_hWrite2\]  
seg008:005DA716                 push    \[ebp+var\_9E0\]  
seg008:005DA71C                 push    \[ebp+var\_9E4\]  
seg008:005DA722                 push    edi  
seg008:005DA723                 push    eax  
seg008:005DA724                 call    Write\_File  
seg008:005DA729                 add     edi, \[ebp+var\_9E4\]  
seg008:005DA72F                 pop     ecx  
seg008:005DA730                 loop    loc\_5DA70F  
seg008:005DA732                 pop     edx  
seg008:005DA733                 lea     eax, \[ebp+var\_hWrite2\]  
seg008:005DA739                 push    \[ebp+var\_9E0\]  
seg008:005DA73F                 push    edx  
seg008:005DA740                 push    edi  
seg008:005DA741                 push    eax  
seg008:005DA742                 call    Write\_File  
seg008:005DA747                 call    loc\_5DA759  
seg008:005DA747 ; ------------------------------------------------------------------  
  
\----------  
seg008:005DA74C aAdvapi32\_dll   db 'ADVAPI32.dll',0  
seg008:005DA759 ; ------------------------------------------------------------------  
  
\----------  
seg008:005DA759  
seg008:005DA759 loc\_5DA759:                                            ; CODE XREF:   
  
Decrypt\_Code+58Dp  
seg008:005DA759                 call    \[ebp+var\_LoadLibraryA\]  
seg008:005DA75F                 mov     \[ebp+LookupPrivilegeValueA\], eax  
seg008:005DA765                 push    \[ebp+var\_CloseHandle\]  
seg008:005DA76B                 call    loc\_5DA786  
seg008:005DA76B ; ------------------------------------------------------------------  
  
\----------  
seg008:005DA770 aAdjusttokenpri db 'AdjustTokenPrivileges',0  
seg008:005DA786 ; ------------------------------------------------------------------  
  
\----------  
seg008:005DA786  
seg008:005DA786 loc\_5DA786:                                            ; CODE XREF:   
  
Decrypt\_Code+5B1p  
seg008:005DA786                 push    \[ebp+LookupPrivilegeValueA\]  
seg008:005DA78C                 call    \[ebp+var\_GetProcAddress\]  
seg008:005DA792                 push    eax  
seg008:005DA793                 call    loc\_5DA7AE  
seg008:005DA793 ; ------------------------------------------------------------------  
  
\----------  
seg008:005DA798 aLookupprivileg db 'LookupPrivilegeValueA',0  
seg008:005DA7AE ; ------------------------------------------------------------------  
  
\----------  
seg008:005DA7AE  
seg008:005DA7AE loc\_5DA7AE:                                            ; CODE XREF:   
  
Decrypt\_Code+5D9p  
seg008:005DA7AE                 push    \[ebp+LookupPrivilegeValueA\]  
seg008:005DA7B4                 call    \[ebp+var\_GetProcAddress\]  
seg008:005DA7BA                 push    eax  
seg008:005DA7BB                 call    loc\_5DA7D1                     ;   
  
**AdjustTokenPrivileges**  
seg008:005DA7BB ; ------------------------------------------------------------------  
  
\----------  
seg008:005DA7C0 aOpenprocesstok db 'OpenProcessToken',0  
seg008:005DA7D1 ; ------------------------------------------------------------------  
  
\----------  
seg008:005DA7D1  
seg008:005DA7D1 loc\_5DA7D1:                                            ; CODE XREF:   
  
Decrypt\_Code+601p  
seg008:005DA7D1                 push    \[ebp+LookupPrivilegeValueA\]    ;   
  
LookupPrivilegeValueA  
seg008:005DA7D7                 call    \[ebp+var\_GetProcAddress\]  
seg008:005DA7DD                 push    eax                            ;   
  
OpenProcessToke  
seg008:005DA7DE                 push    \[ebp+GetCurrentProc\]           ; 分别传入:  
seg008:005DA7DE                                                        ;   
  
**GetCurrentProcess**  
seg008:005DA7DE                                                        ;   
  
**OpenProcessToken**  
seg008:005DA7DE                                                        ;   
  
LookupPrivilegeValueA  
seg008:005DA7DE                                                        ;   
  
**AdjustTokenPrivileges**  
seg008:005DA7DE                                                        ; CloseHandle  
seg008:005DA7E4                 call    Get\_Privileges  
seg008:005DA7E9                 nop               ;如果没解密之  
  
前这里全部是"垃圾"来的:-)  
seg008:005DA7EA                 push    0  
seg008:005DA7EC                 push    0  
seg008:005DA7EE                 push    11h  
seg008:005DA7F0                 call    \[ebp+var\_GetCurrentThread\]  
seg008:005DA7F6                 push    eax  
seg008:005DA7F7                 call    GetCodeBase  
seg008:005DA7FC                 add     eax, 76h  
seg008:005DA801                 mov     eax, \[eax\]  
seg008:005DA803                 ror     eax, 2  
seg008:005DA806                 call    eax                            ;   
  
ZwSetInformationThread  
seg008:005DA808                 push    \[ebp+var\_97C\]  
seg008:005DA80E                 call    \[ebp+var\_DebugActiveProcess\]  
seg008:005DA814                 push    eax  
seg008:005DA815                 call    $+5  
seg008:005DA81A  
seg008:005DA81A loc\_5DA81A:                                            ; CODE XREF:   
  
Decrypt\_Code+2E7j  
seg008:005DA81A                 pop     eax  
seg008:005DA81B                 add     eax, 89Dh  
seg008:005DA820                 mov     byte ptr \[eax\], 90h  
seg008:005DA823                 pop     eax  
seg008:005DA824                 or      eax, eax  
seg008:005DA826                 jz      Over                           ; 如  
  
果 DebugActiveProcess 失败则结束程序  
seg008:005DA82C                 push    800h  
seg008:005DA831                 push    0  
seg008:005DA833                 call    \[ebp+var\_LocalAlloc\]  
seg008:005DA839                 xor     ecx, ecx  
seg008:005DA83B                 mov     \[eax\], ecx  
seg008:005DA83D                 mov     \[ebp+var\_hMemLocal\], eax  
seg008:005DA843                 mov     \[ebp+var\_hGDI321\], 0  
seg008:005DA84D                 push    \[ebp+var\_hEvent\]  
seg008:005DA853                 push    0A0h  
seg008:005DA858                 call    \[ebp+var\_Sleep\]  
seg008:005DA85E                 call    \[ebp+var\_SetEvent\]  
seg008:005DA864  
seg008:005DA864 loc\_5DA864:                                            ; CODE XREF:   
  
Decrypt\_Code+928j  
seg008:005DA864                                                        ;   
  
Decrypt\_Code+958j ...  
seg008:005DA864                 push    0FFFFFFFFh  
seg008:005DA866                 lea     eax, \[ebp+var\_DebugEvent\]  
seg008:005DA86C                 push    eax  
seg008:005DA86D                 call    \[ebp+var\_WaitForDebugEvent\]  
seg008:005DA873                 cmp     \[ebp+var\_DebugEvent\], 5        ; case   
  
EXIT\_PROCESS\_DEBUG\_EVENT  
seg008:005DA87A                 jnz     short loc\_5DA89D               ; case   
  
CREATE\_PROCESS\_DEBUG\_EVENT  
seg008:005DA87C                 push    80010001h  
seg008:005DA881                 push    \[ebp+var\_ThreadId\]  
seg008:005DA887                 push    \[ebp+var\_ProcessId\]  
seg008:005DA88D                 call    \[ebp+var\_ContinueDebugEvent\]  
seg008:005DA893                 jmp     loc\_5DAE52  
seg008:005DA898 ; ------------------------------------------------------------------  
  
\----------  
seg008:005DA898                 jmp     loc\_5DAE36                     ;   
  
ContinueStatus  
seg008:005DA89D ; ------------------------------------------------------------------  
  
\----------  
seg008:005DA89D  
seg008:005DA89D loc\_5DA89D:                                            ; CODE XREF:   
  
Decrypt\_Code+6C0j  
seg008:005DA89D                 cmp     \[ebp+var\_DebugEvent\], 3        ; case   
  
CREATE\_PROCESS\_DEBUG\_EVENT  
seg008:005DA8A4                 jnz     short loc\_5DA900               ; case   
  
CREATE\_THREAD\_DEBUG\_EVENT  
seg008:005DA8A6                 mov     edx, \[ebp+var\_hMemLocal\]  
seg008:005DA8AC                 inc     dword ptr \[edx\]  
seg008:005DA8AE                 mov     eax, \[ebp+var\_ThreadId\]  
seg008:005DA8B4                 mov     \[edx+4\], eax  
seg008:005DA8B7                 mov     \[ebp+var\_988\], eax  
seg008:005DA8BD                 mov     eax, \[ebp+var\_pThread\]  
seg008:005DA8C3                 mov     \[edx+8\], eax  
seg008:005DA8C6                 mov     \[ebp+var\_hThread\], eax  
seg008:005DA8CC                 push    \[ebp+var\_5D8\]  
seg008:005DA8D2                 pop     \[ebp+var\_hProcess\]  
seg008:005DA8D8                 mov     \[ebp+var\_pContext\], 10001h  
seg008:005DA8E2                 lea     eax, \[ebp+var\_pContext\]  
seg008:005DA8E8                 push    eax  
seg008:005DA8E9                 push    \[ebp+var\_pThread\]  
seg008:005DA8EF                 call    \[ebp+var\_GetThreadContext\]  
seg008:005DA8F5                 mov     eax, \[ebp+var\_ExcepAddress\]  
seg008:005DA8FB                 jmp     loc\_5DAE36                     ;   
  
ContinueStatus  
seg008:005DA900 ; ------------------------------------------------------------------  
  
\----------  
seg008:005DA900  
seg008:005DA900 loc\_5DA900:                                            ; CODE XREF:   
  
Decrypt\_Code+6EAj  
seg008:005DA900                 cmp     \[ebp+var\_DebugEvent\], 2        ; case   
  
CREATE\_THREAD\_DEBUG\_EVENT  
seg008:005DA907                 jnz     short loc\_5DA92C               ; case   
  
EXIT\_THREAD\_DEBUG\_EVENT  
seg008:005DA909                 mov     edx, \[ebp+var\_hMemLocal\]  
seg008:005DA90F                 mov     ecx, \[edx\]  
seg008:005DA911                 mov     eax, \[ebp+var\_ThreadId\]  
seg008:005DA917                 mov     \[edx+ecx\*8+4\], eax  
seg008:005DA91B                 mov     eax, \[ebp+var\_Exception\_Code\]  
seg008:005DA921                 mov     \[edx+ecx\*8+8\], eax  
seg008:005DA925                 inc     dword ptr \[edx\]  
seg008:005DA927                 jmp     loc\_5DAE36                     ;   
  
ContinueStatus  
seg008:005DA92C ; ------------------------------------------------------------------  
  
\----------  
seg008:005DA92C  
seg008:005DA92C loc\_5DA92C:                                            ; CODE XREF:   
  
Decrypt\_Code+74Dj  
seg008:005DA92C                 cmp     \[ebp+var\_DebugEvent\], 4        ; case   
  
EXIT\_THREAD\_DEBUG\_EVENT  
seg008:005DA933                 jnz     short loc\_5DA96D               ; case   
  
EXCEPTION\_DEBUG\_EVENT  
seg008:005DA935                 push    edi  
seg008:005DA936                 mov     edi, \[ebp+var\_hMemLocal\]  
seg008:005DA93C                 mov     ecx, \[edi\]  
seg008:005DA93E                 add     edi, 4  
seg008:005DA941                 shl     ecx, 1  
seg008:005DA943                 mov     eax, \[ebp+var\_ThreadId\]  
seg008:005DA949                 repne scasd  
seg008:005DA94B                 jnz     short loc\_5DA967  
seg008:005DA94D                 shr     ecx, 1  
seg008:005DA94F                 push    esi  
seg008:005DA950                 sub     edi, 4  
seg008:005DA953                 lea     esi, \[edi+8\]  
seg008:005DA956                 or      ecx, ecx  
seg008:005DA958                 jz      short loc\_5DA966  
seg008:005DA95A  
seg008:005DA95A loc\_5DA95A:                                            ; CODE XREF:   
  
Decrypt\_Code+7A2j  
seg008:005DA95A                 lodsd  
seg008:005DA95B                 stosd  
seg008:005DA95C                 loop    loc\_5DA95A  
seg008:005DA95E                 mov     edi, \[ebp+var\_hMemLocal\]  
seg008:005DA964                 dec     dword ptr \[edi\]  
seg008:005DA966  
seg008:005DA966 loc\_5DA966:                                            ; CODE XREF:   
  
Decrypt\_Code+79Ej  
seg008:005DA966                 pop     esi  
seg008:005DA967  
seg008:005DA967 loc\_5DA967:                                            ; CODE XREF:   
  
Decrypt\_Code+791j  
seg008:005DA967                 pop     edi  
seg008:005DA968                 jmp     loc\_5DAE36                     ;   
  
ContinueStatus  
seg008:005DA96D ; ------------------------------------------------------------------  
  
\----------  
seg008:005DA96D  
seg008:005DA96D loc\_5DA96D:                                            ; CODE XREF:   
  
Decrypt\_Code+779j  
seg008:005DA96D                 cmp     \[ebp+var\_DebugEvent\], 1        ; case   
  
EXCEPTION\_DEBUG\_EVENT  
seg008:005DA974                 jnz     loc\_5DAE36                     ;   
  
ContinueStatus  
seg008:005DA97A                 cmp     \[ebp+var\_Exception\_Code\], 80000003h ; 断点异  
  
常  
seg008:005DA984                 jnz     loc\_5DABCF                     ;   
  
ACCESS\_VIOLATION  
seg008:005DA98A                 mov     eax, \[ebp+var\_988\]  
seg008:005DA990                 cmp     eax, \[ebp+var\_ThreadId\]  
seg008:005DA996                 jnz     loc\_5DAAEC  
seg008:005DA99C                 cmp     \[ebp+var\_hGDI321\], 0  
seg008:005DA9A3                 jnz     loc\_5DAA33  
seg008:005DA9A9                 call    loc\_5DA9B9  
seg008:005DA9A9 ; ------------------------------------------------------------------  
  
\----------  
seg008:005DA9AE aUser32\_dll     db 'User32.dll',0  
seg008:005DA9B9 ; ------------------------------------------------------------------  
  
\----------  
seg008:005DA9B9  
seg008:005DA9B9 loc\_5DA9B9:                                            ; CODE XREF:   
  
Decrypt\_Code+7EFp  
seg008:005DA9B9                 call    \[ebp+var\_LoadLibraryA\]  
seg008:005DA9BF                 mov     \[ebp+var\_hUser32\], eax  
seg008:005DA9C5                 call    loc\_5DA9D4  
seg008:005DA9C5 ; ------------------------------------------------------------------  
  
\----------  
seg008:005DA9CA aGdi32\_dll      db 'GDI32.dll',0  
seg008:005DA9D4 ; ------------------------------------------------------------------  
  
\----------  
seg008:005DA9D4  
seg008:005DA9D4 loc\_5DA9D4:                                            ; CODE XREF:   
  
Decrypt\_Code+80Bp  
seg008:005DA9D4                 call    \[ebp+var\_LoadLibraryA\]  
seg008:005DA9DA                 mov     \[ebp+var\_hGDI32\], eax  
seg008:005DA9E0                 mov     \[ebp+var\_hGDI321\], eax  
seg008:005DA9E6                 push    edi  
seg008:005DA9E7                 push    esi  
seg008:005DA9E8                 mov     edi, \[ebp+var\_hMemLocal\]  
seg008:005DA9EE                 push    dword ptr \[edi+8\]  
seg008:005DA9F1                 pop     \[ebp+var\_hThread\]  
seg008:005DA9F7                 mov     \[ebp+var\_pContext\], 10007h  
seg008:005DAA01                 lea     eax, \[ebp+var\_pContext\]  
seg008:005DAA07                 push    eax  
seg008:005DAA08                 push    \[ebp+var\_hThread\]  
seg008:005DAA0E                 call    \[ebp+var\_GetThreadContext\]  
seg008:005DAA14  
seg008:005DAA14 loc\_5DAA14:  
seg008:005DAA14                 add     \[ebp+var\_ExcepAddress\], 6  
seg008:005DAA1B                 lea     eax, \[ebp+var\_pContext\]  
seg008:005DAA21                 push    eax  
seg008:005DAA22                 push    \[ebp+var\_hThread\]  
seg008:005DAA28                 call    \[ebp+var\_SetThreadContext\]  
seg008:005DAA2E                 jmp     loc\_5DAAC9  
seg008:005DAA33 ; ------------------------------------------------------------------  
  
\----------  
seg008:005DAA33  
seg008:005DAA33 loc\_5DAA33:                                            ; CODE XREF:   
  
Decrypt\_Code+7E9j  
seg008:005DAA33                 push    edi  
seg008:005DAA34                 push    esi  
seg008:005DAA35                 mov     esi, \[ebp+var\_hMemLocal\]  
seg008:005DAA3B                 push    dword ptr \[esi+8\]  
seg008:005DAA3E                 pop     \[ebp+var\_hThread\]  
seg008:005DAA44                 mov     \[ebp+var\_pContext\], 10007h  
seg008:005DAA4E                 lea     eax, \[ebp+var\_pContext\]  
seg008:005DAA54                 push    eax  
seg008:005DAA55                 push    \[ebp+var\_hThread\]  
seg008:005DAA5B                 call    \[ebp+var\_GetThreadContext\]  
seg008:005DAA61                 mov     eax, \[ebp+var\_ExcepAddress\]    ; 获取异常地址  
seg008:005DAA67                 mov     edi, \[ebp+var\_9B4\]  
seg008:005DAA6D                 mov     edx, \[edi+10h\]  
seg008:005DAA70                 mov     ecx, \[edi+14h\]  
seg008:005DAA73                 sub     ecx, edx  
seg008:005DAA75                 lea     edi, \[edx+edi+20h\]  
seg008:005DAA79                 mov     edx, eax  
seg008:005DAA7B                 sub     eax, \[ebp+var\_App\_hModule\]  
seg008:005DAA81                 and     ax, 0F000h  
seg008:005DAA85                 push    \[ebp+var\_hLocal\]  
seg008:005DAA8B                 push    \[ebp+var\_9B4\]  
seg008:005DAA91                 call    Getjmpaddress                  ; 父进程异常后  
  
，写入正确的内容  
seg008:005DAA91                                                        ; 也就是动态写  
  
入跳转表  
seg008:005DAA96                 test    eax, eax                       ; 操作成功  
  
后 eax 返回跳转地址  
seg008:005DAA98                 jz      short loc\_5DAAE7  
seg008:005DAA9A                 dec     edx  
seg008:005DAA9B                 push    edx  
seg008:005DAA9C                 mov     \[ebp+var\_ExcepAddress\], eax  
seg008:005DAAA2                 push    eax  
seg008:005DAAA3                 push    \[ebp+var\_hProcess\]  
seg008:005DAAA9                 push    \[ebp+var\_918\]  
seg008:005DAAAF                 mov     al, 0E9h  
seg008:005DAAB1                 call    WritePresentMem                ; 写入代码，改  
  
写成 jmp address  
seg008:005DAAB1                                                        ; 其中 address  
  
就是前面 eax 的值  
seg008:005DAAB6                 lea     eax, \[ebp+var\_pContext\]  
seg008:005DAABC                 push    eax  
seg008:005DAABD                 push    \[ebp+var\_hThread\]  
seg008:005DAAC3                 call    \[ebp+var\_SetThreadContext\]  
seg008:005DAAC9  
seg008:005DAAC9 loc\_5DAAC9:                                            ; CODE XREF:   
  
Decrypt\_Code+874j  
seg008:005DAAC9                                                        ;   
  
Decrypt\_Code+A04j  
seg008:005DAAC9                 push    10002h  
seg008:005DAACE                 push    \[ebp+var\_ThreadId\]  
seg008:005DAAD4                 push    \[ebp+var\_ProcessId\]  
seg008:005DAADA                 call    \[ebp+var\_ContinueDebugEvent\]  
seg008:005DAAE0                 pop     esi  
seg008:005DAAE1                 pop     edi  
seg008:005DAAE2                 jmp     loc\_5DA864  
seg008:005DAAE7 ; ------------------------------------------------------------------  
  
\----------  
seg008:005DAAE7  
seg008:005DAAE7 loc\_5DAAE7:                                            ; CODE XREF:   
  
Decrypt\_Code+8DEj  
seg008:005DAAE7                                                        ;   
  
Decrypt\_Code+9D4j  
seg008:005DAAE7                 jmp     loc\_5DAE36                     ;   
  
ContinueStatus  
seg008:005DAAEC ; ------------------------------------------------------------------  
  
\----------  
seg008:005DAAEC  
seg008:005DAAEC loc\_5DAAEC:                                            ; CODE XREF:   
  
Decrypt\_Code+7DCj  
seg008:005DAAEC                 cmp     \[ebp+var\_9CC\], 0  
seg008:005DAAF3                 jnz     short loc\_5DAB1C  
seg008:005DAAF5  
seg008:005DAAF5 loc\_5DAAF5:                                            ; CODE XREF:   
  
Decrypt\_Code+A0Bj  
seg008:005DAAF5                 inc     \[ebp+var\_9CC\]  
seg008:005DAAFB                 push    10002h  
seg008:005DAB00                 push    \[ebp+var\_ThreadId\]  
seg008:005DAB06                 push    \[ebp+var\_ProcessId\]  
seg008:005DAB0C                 call    \[ebp+var\_ContinueDebugEvent\]  
seg008:005DAB12                 jmp     loc\_5DA864  
seg008:005DAB17 ; ------------------------------------------------------------------  
  
\----------  
seg008:005DAB17                 jmp     loc\_5DABCA  
seg008:005DAB1C ; ------------------------------------------------------------------  
  
\----------  
seg008:005DAB1C  
seg008:005DAB1C loc\_5DAB1C:                                            ; CODE XREF:   
  
Decrypt\_Code+939j  
seg008:005DAB1C                 push    edi  
seg008:005DAB1D                 push    esi  
seg008:005DAB1E                 mov     edi, \[ebp+var\_hMemLocal\]  
seg008:005DAB24                 mov     ecx, \[edi\]  
seg008:005DAB26                 add     edi, 4  
seg008:005DAB29                 shl     ecx, 1  
seg008:005DAB2B                 mov     eax, \[ebp+var\_ThreadId\]  
seg008:005DAB31                 repne scasd  
seg008:005DAB33                 jnz     loc\_5DABC3  
seg008:005DAB39                 mov     esi, \[edi\]  
seg008:005DAB3B                 mov     \[ebp+var\_pContext\], 10007h  
seg008:005DAB45                 lea     eax, \[ebp+var\_pContext\]  
seg008:005DAB4B                 push    eax  
seg008:005DAB4C                 push    esi  
seg008:005DAB4D                 call    \[ebp+var\_GetThreadContext\]  
seg008:005DAB53                 test    eax, eax  
seg008:005DAB55                 jz      short loc\_5DABC3  
seg008:005DAB57                 mov     eax, \[ebp+var\_ExcepAddress\]  
seg008:005DAB5D                 mov     edi, \[ebp+var\_9B4\]  
seg008:005DAB63                 mov     edx, \[edi+10h\]  
seg008:005DAB66                 mov     ecx, \[edi+14h\]  
seg008:005DAB69                 sub     ecx, edx  
seg008:005DAB6B                 lea     edi, \[edx+edi+20h\]  
seg008:005DAB6F                 mov     edx, eax  
seg008:005DAB71                 sub     eax, \[ebp+var\_App\_hModule\]  
seg008:005DAB77                 and     ax, 0F000h  
seg008:005DAB7B                 push    \[ebp+var\_hLocal\]  
seg008:005DAB81                 push    \[ebp+var\_9B4\]  
seg008:005DAB87                 call    Getjmpaddress  
seg008:005DAB8C                 test    eax, eax  
seg008:005DAB8E                 jz      loc\_5DAAE7  
seg008:005DAB94                 dec     edx  
seg008:005DAB95                 push    edx  
seg008:005DAB96                 mov     \[ebp+var\_ExcepAddress\], eax  
seg008:005DAB9C                 push    eax  
seg008:005DAB9D                 push    \[ebp+var\_hProcess\]  
seg008:005DABA3                 push    \[ebp+var\_918\]  
seg008:005DABA9                 mov     al, 0E9h  
seg008:005DABAB                 call    WritePresentMem  
seg008:005DABB0                 lea     eax, \[ebp+var\_pContext\]  
seg008:005DABB6                 push    eax  
seg008:005DABB7                 push    esi  
seg008:005DABB8                 call    \[ebp+var\_SetThreadContext\]  
seg008:005DABBE                 jmp     loc\_5DAAC9  
seg008:005DABC3 ; ------------------------------------------------------------------  
  
\----------  
seg008:005DABC3  
seg008:005DABC3 loc\_5DABC3:                                            ; CODE XREF:   
  
Decrypt\_Code+979j  
seg008:005DABC3                                                        ;   
  
Decrypt\_Code+99Bj  
seg008:005DABC3                 pop     esi  
seg008:005DABC4                 pop     edi  
seg008:005DABC5                 jmp     loc\_5DAAF5  
seg008:005DABCA ; ------------------------------------------------------------------  
  
\----------  
seg008:005DABCA  
seg008:005DABCA loc\_5DABCA:                                            ; CODE XREF:   
  
Decrypt\_Code+95Dj  
seg008:005DABCA                 jmp     loc\_5DAE36                     ;   
  
ContinueStatus  
seg008:005DABCF ; ------------------------------------------------------------------  
  
\----------  
seg008:005DABCF  
seg008:005DABCF loc\_5DABCF:                                            ; CODE XREF:   
  
Decrypt\_Code+7CAj  
seg008:005DABCF                 cmp     \[ebp+var\_Exception\_Code\], 0C0000005h ;   
  
ACCESS\_VIOLATION  
seg008:005DABD9                 jnz     loc\_5DAE36                     ;   
  
ContinueStatus  
seg008:005DABDF                 push    edi  
seg008:005DABE0                 mov     edi, \[ebp+var\_hMemLocal\]  
seg008:005DABE6                 mov     ecx, \[edi\]  
seg008:005DABE8                 mov     eax, \[ebp+var\_ThreadId\]  
seg008:005DABEE                 add     edi, 4  
seg008:005DABF1                 shl     ecx, 1  
seg008:005DABF3                 repne scasd  
seg008:005DABF5                 jnz     loc\_5DAE35  
seg008:005DABFB                 push    dword ptr \[edi\]  
seg008:005DABFD                 pop     \[ebp+var\_hThread\]  
seg008:005DAC03                 mov     \[ebp+var\_pContext\], 10007h  
seg008:005DAC0D                 lea     eax, \[ebp+var\_pContext\]  
seg008:005DAC13                 push    eax  
seg008:005DAC14                 push    \[ebp+var\_hThread\]  
seg008:005DAC1A                 call    \[ebp+var\_GetThreadContext\]  
seg008:005DAC20                 mov     eax, \[ebp+var\_ExcepAddress\]  
seg008:005DAC26                 cmp     eax, 0FADE68B1h                ; 关键  
seg008:005DAC2B                 jnz     loc\_5DAD07  
seg008:005DAC31                 mov     \[ebp+var\_994\], eax  
seg008:005DAC37                 inc     \[ebp+var\_990\]  
seg008:005DAC3D                 push    esi  
seg008:005DAC3E                 lea     eax, \[ebp+var\_410\]  
seg008:005DAC44                 push    eax  
seg008:005DAC45                 push    30h  
seg008:005DAC47                 mov     edi, \[ebp+var\_hMemLocal\]  
seg008:005DAC4D                 add     edi, 10h  
seg008:005DAC50                 push    edi  
seg008:005DAC51                 push    \[ebp+var\_pBaseAddress\]  
seg008:005DAC57                 push    \[ebp+var\_hProcess\]  
seg008:005DAC5D                 call    \[ebp+var\_ReadProcMemory\]  
seg008:005DAC63                 cmp     \[ebp+var\_410\], 30h  
seg008:005DAC6A                 jnz     short loc\_5DACD1  
seg008:005DAC6C                 add     esp, 0FFFFFFECh  
seg008:005DAC6F                 lea     esi, \[edi+4\]  
seg008:005DAC72                 mov     edi, esp  
seg008:005DAC74                 mov     ecx, 4  
seg008:005DAC79                 rep movsd  
seg008:005DAC7B                 mov     \[edi\], esi  
seg008:005DAC7D                 cmp     \[ebp+var\_990\], 1  
seg008:005DAC84                 jnz     short loc\_5DACA4  
seg008:005DAC86                 mov     edi, esp  
seg008:005DAC88                 lea     eax, \[ebp+var\_410\]  
seg008:005DAC8E                 push    eax  
seg008:005DAC8F                 push    100h  
seg008:005DAC94                 push    dword ptr \[edi\]  
seg008:005DAC96                 push    dword ptr \[edi\]  
seg008:005DAC98                 push    \[ebp+var\_hProcess\]  
seg008:005DAC9E                 call    \[ebp+var\_ReadProcMemory\]  
seg008:005DACA4  
seg008:005DACA4 loc\_5DACA4:                                            ; CODE XREF:   
  
Decrypt\_Code+ACAj  
seg008:005DACA4                 push    esi  
seg008:005DACA5                 call    \[ebp+var\_LoadLibraryA\]  
seg008:005DACAB                 call    GetAPI  
seg008:005DACB0                 add     esp, 4  
seg008:005DACB3                 mov     \[ebp+var\_80C\], eax  
seg008:005DACB9                 mov     eax, \[ebp+var\_hMemLocal\]  
seg008:005DACBF                 add     eax, 10h  
seg008:005DACC2                 push    dword ptr \[eax\]  
seg008:005DACC4                 pop     \[ebp+var\_ExcepAddress\]  
seg008:005DACCA                 add     \[ebp+var\_pBaseAddress\], 30h  
seg008:005DACD1  
seg008:005DACD1 loc\_5DACD1:                                            ; CODE XREF:   
  
Decrypt\_Code+AB0j  
seg008:005DACD1                 pop     esi  
seg008:005DACD2                 lea     eax, \[ebp+var\_pContext\]  
seg008:005DACD8                 push    eax  
seg008:005DACD9                 push    \[ebp+var\_hThread\]  
seg008:005DACDF                 call    \[ebp+var\_SetThreadContext\]  
seg008:005DACE5                 push    10002h  
seg008:005DACEA                 push    \[ebp+var\_ThreadId\]  
seg008:005DACF0                 push    \[ebp+var\_ProcessId\]  
seg008:005DACF6                 call    \[ebp+var\_ContinueDebugEvent\]  
seg008:005DACFC                 pop     edi  
seg008:005DACFD                 jmp     loc\_5DA864  
seg008:005DAD02 ; ------------------------------------------------------------------  
  
\----------  
seg008:005DAD02                 jmp     loc\_5DAE35  
seg008:005DAD07 ; ------------------------------------------------------------------  
  
\----------  
seg008:005DAD07  
seg008:005DAD07 loc\_5DAD07:                                            ; CODE XREF:   
  
Decrypt\_Code+A71j  
seg008:005DAD07                 cmp     eax, 70000000h  
seg008:005DAD0C                 jbe     loc\_5DADB6  
seg008:005DAD12                 lea     eax, \[ebp+var\_410\]  
seg008:005DAD18                 push    eax  
seg008:005DAD19                 push    30h  
seg008:005DAD1B                 mov     edi, \[ebp+var\_hMemLocal\]  
seg008:005DAD21                 add     edi, 10h  
seg008:005DAD24                 push    edi  
seg008:005DAD25                 push    \[ebp+var\_pBaseAddress\]  
seg008:005DAD2B                 push    \[ebp+var\_hProcess\]  
seg008:005DAD31                 call    \[ebp+var\_ReadProcMemory\]  
seg008:005DAD37                 mov     eax, \[edi\]  
seg008:005DAD39                 mov     edx, eax  
seg008:005DAD3B                 sub     eax, \[ebp+var\_App\_hModule\]  
seg008:005DAD41                 and     ax, 0F000h  
seg008:005DAD45                 mov     edi, \[ebp+var\_9B4\]  
seg008:005DAD4B                 mov     ecx, \[edi+10h\]  
seg008:005DAD4E                 add     edi, 20h  
seg008:005DAD51                 push    \[ebp+var\_hLocal\]  
seg008:005DAD57                 push    \[ebp+var\_9B4\]  
seg008:005DAD5D                 call    Getjmpaddress  
seg008:005DAD62                 test    eax, eax  
seg008:005DAD64                 jz      short loc\_5DADB4  
seg008:005DAD66                 sub     edx, 5  
seg008:005DAD69                 push    edx  
seg008:005DAD6A                 mov     \[ebp+var\_ExcepAddress\], eax  
seg008:005DAD70                 push    eax  
seg008:005DAD71                 push    \[ebp+var\_hProcess\]  
seg008:005DAD77  
seg008:005DAD77 loc\_5DAD77:  
seg008:005DAD77                 push    \[ebp+var\_918\]  
seg008:005DAD7D                 mov     al, 0E8h  
seg008:005DAD7F                 call    WritePresentMem  
seg008:005DAD84                 lea     eax, \[ebp+var\_pContext\]  
seg008:005DAD8A                 push    eax  
seg008:005DAD8B                 push    \[ebp+var\_hThread\]  
seg008:005DAD91                 call    \[ebp+var\_SetThreadContext\]  
seg008:005DAD97                 push    10002h  
seg008:005DAD9C                 push    \[ebp+var\_ThreadId\]  
seg008:005DADA2                 push    \[ebp+var\_ProcessId\]  
seg008:005DADA8                 call    \[ebp+var\_ContinueDebugEvent\]  
seg008:005DADAE                 pop     edi  
seg008:005DADAF                 jmp     loc\_5DA864  
seg008:005DADB4 ; ------------------------------------------------------------------  
  
\----------  
seg008:005DADB4  
seg008:005DADB4 loc\_5DADB4:                                            ; CODE XREF:   
  
Decrypt\_Code+BAAj  
seg008:005DADB4                 jmp     short loc\_5DAE35  
seg008:005DADB6 ; ------------------------------------------------------------------  
  
\----------  
seg008:005DADB6  
seg008:005DADB6 loc\_5DADB6:                                            ; CODE XREF:   
  
Decrypt\_Code+B52j  
seg008:005DADB6                 cmp     \[ebp+var\_994\], 0  
seg008:005DADBD                 jz      short loc\_5DAE35  
seg008:005DADBF                 push    1000h  
seg008:005DADC4                 push    40h  
seg008:005DADC6                 call    \[ebp+var\_LocalAlloc\]  
seg008:005DADCC                 mov     \[ebp+var\_hLocal\], eax  
seg008:005DADD2                 lea     eax, \[ebp+var\_410\]  
seg008:005DADD8                 push    eax  
seg008:005DADD9                 mov     eax, \[ebp+var\_CodeBase\]  
seg008:005DADDF                 add     eax, 2AC4h  
seg008:005DADE4                 mov     \[ebp+var\_9B4\], eax  
seg008:005DADEA                 push    dword ptr \[eax+0Ch\]  
seg008:005DADED                 pop     \[ebp+var\_9C0\]  
seg008:005DADF3                 push    dword ptr \[eax+8\]  
seg008:005DADF6                 pop     \[ebp+var\_9BC\]  
seg008:005DADFC                 push    dword ptr \[eax+4\]  
seg008:005DADFF                 pop     \[ebp+var\_9C4\]  
seg008:005DAE05                 add     esp, 0FFFFFFFCh  
seg008:005DAE08                 push    \[ebp+var\_hLocal\]  
seg008:005DAE0E                 mov     eax, \[eax\]  
seg008:005DAE10                 push    eax  
seg008:005DAE11                 sub     eax, \[ebp+var\_hLocal\]  
seg008:005DAE17                 neg     eax  
seg008:005DAE19                 mov     \[ebp+var\_9B8\], eax  
seg008:005DAE1F                 push    \[ebp+var\_hProcess\]  
seg008:005DAE25                 call    \[ebp+var\_ReadProcMemory\]  
seg008:005DAE2B                 mov     \[ebp+var\_994\], 0  
seg008:005DAE35  
seg008:005DAE35 loc\_5DAE35:                                            ; CODE XREF:   
  
Decrypt\_Code+A3Bj  
seg008:005DAE35                                                        ;   
  
Decrypt\_Code+B48j ...  
seg008:005DAE35                 pop     edi  
seg008:005DAE36  
seg008:005DAE36 loc\_5DAE36:                                            ; CODE XREF:   
  
Decrypt\_Code+6DEj  
seg008:005DAE36                                                        ;   
  
Decrypt\_Code+741j ...  
seg008:005DAE36                 push    80010001h                      ;   
  
ContinueStatus  
seg008:005DAE3B                 push    \[ebp+var\_ThreadId\]  
seg008:005DAE41                 push    \[ebp+var\_ProcessId\]  
seg008:005DAE47                 call    \[ebp+var\_ContinueDebugEvent\]  
seg008:005DAE4D                 jmp     loc\_5DA864  
seg008:005DAE52 ; ------------------------------------------------------------------  
  
\----------  
seg008:005DAE52  
seg008:005DAE52 loc\_5DAE52:                                            ; CODE XREF:   
  
Decrypt\_Code+6D9j  
seg008:005DAE52                 pop     edi  
seg008:005DAE53                 jmp     Over  
seg008:005DAE58 ; ------------------------------------------------------------------  
  
\----------  
seg008:005DAE58  
seg008:005DAE58 loc\_5DAE58:                                            ; CODE XREF:   
  
Decrypt\_Code+3F5j  
seg008:005DAE58                 call    \[ebp+var\_GetCurrentProcessId\]  
seg008:005DAE5E  
seg008:005DAE5E loc\_5DAE5E:  
seg008:005DAE5E                 mov     edi, \[ebp+var\_hMap\]  
seg008:005DAE64                 add     edi, 100h  
seg008:005DAE6A                 stosd                                  ; hMap+100h 处  
  
保存当前进程 ID  
seg008:005DAE6B                 push    edi                            ; EventName =   
  
"PeLock\_Event\_C40"  
seg008:005DAE6C                 call    loc\_5DAE82  
seg008:005DAE6C ; ------------------------------------------------------------------  
  
\----------  
seg008:005DAE71 aPelock\_eMzC    db 'PeLock\_Event\_123',0  
seg008:005DAE82 ; ------------------------------------------------------------------  
  
\----------  
seg008:005DAE82  
seg008:005DAE82 loc\_5DAE82:                                            ; CODE XREF:   
  
Decrypt\_Code+CB2p  
seg008:005DAE82                 pop     edi  
seg008:005DAE83                 push    edi  
seg008:005DAE84                 xor     eax, eax  
seg008:005DAE86                 push    0FFFFFFFFh  
seg008:005DAE88                 pop     ecx  
seg008:005DAE89                 repne scasb                            ; 定位  
  
到 Pelock\_Event\_123 的结束处  
seg008:005DAE8B                 sub     edi, 4  
seg008:005DAE8E                 lea     esi, \[ebp+var\_Cstr(PID)\]  
seg008:005DAE94  
seg008:005DAE94 loc\_5DAE94:                                            ; CODE XREF:   
  
Decrypt\_Code+CDEj  
seg008:005DAE94                 lodsb  
seg008:005DAE95                 stosb  
seg008:005DAE96                 or      al, al  
seg008:005DAE98                 jnz     short loc\_5DAE94               ; 替  
  
换 PeLock\_Event\_123 为  
seg008:005DAE98                                                        ;   
  
PeLock\_Event\_xxx(CSTR(PID))  
seg008:005DAE9A                 push    0                              ;   
  
|InitiallySignaled = FALSE  
seg008:005DAE9C                 push    1                              ; |ManualReset   
  
\= TRUE  
seg008:005DAE9E                 push    0                              ; |pSecurity =   
  
NULL  
seg008:005DAEA0                 call    \[ebp+var\_CreateEventA\]         ;   
  
\\CreateEventA  
seg008:005DAEA6                 mov     \[ebp+var\_hEvent\], eax  
seg008:005DAEAC                 pop     edi  
seg008:005DAEAD                 stosd  
seg008:005DAEAE                 push    100h                           ; /BufSize =   
  
100 (256.)  
seg008:005DAEB3                 lea     eax, \[ebp+var\_pSecurity\]  
seg008:005DAEB9                 push    eax                            ; |pSecurity =   
  
0012EC8C  
seg008:005DAEBA                 mov     \[ebp+var\_pSecurity\], 0Ch  
seg008:005DAEC4                 xor     eax, eax  
seg008:005DAEC6                 mov     \[ebp+var\_A00\], eax  
seg008:005DAECC                 inc     eax  
seg008:005DAECD                 mov     \[ebp+var\_9FC\], eax  
seg008:005DAED3                 lea     eax, \[ebp+var\_hWrite1\]  
seg008:005DAED9                 push    eax                            ;   
  
|pWriteHandle = 0012ED38  
seg008:005DAEDA                 lea     eax, \[ebp+var\_hRead1\]  
seg008:005DAEE0                 push    eax                            ; |pReadHandle   
  
\= 0012ED3C  
seg008:005DAEE1                 call    \[ebp+var\_CreatePipe\]           ; \\CreatePipe  
seg008:005DAEE7                 push    100h  
seg008:005DAEEC                 lea     eax, \[ebp+var\_pSecurity\]       ; /pSecurity  
seg008:005DAEF2                 push    eax                            ; |  
seg008:005DAEF3                 lea     eax, \[ebp+var\_hWrite2\]         ;   
  
|pWriteHandle = 0012ED30  
seg008:005DAEF9                 push    eax                            ; |  
seg008:005DAEFA                 lea     eax, \[ebp+var\_hRead2\]          ; |pReadHandle   
  
\= 0012ED34  
seg008:005DAF00                 push    eax                            ; |  
seg008:005DAF01                 call    \[ebp+var\_CreatePipe\]           ; \\CreatePipe  
seg008:005DAF07                 mov     eax, \[ebp+var\_hRead2\]  
seg008:005DAF0D                 add     edi, 78h  
seg008:005DAF10                 stosd  
seg008:005DAF11                 mov     eax, \[ebp+var\_hWrite1\]  
seg008:005DAF17                 stosd  
seg008:005DAF18                 mov     eax, \[ebp+var\_CodeBase\]  
seg008:005DAF1E                 push    eax  
seg008:005DAF1F                 add     eax, 5Eh                       ; Trap to   
  
Debugger  
seg008:005DAF24                 mov     eax, \[eax\]  
seg008:005DAF26                 xchg    eax, \[esp+16B0h+var\_16B0\]  
seg008:005DAF29                 pop     \[ebp+var\_PEB1\]  
seg008:005DAF2F                 add     eax, 8Ah  
seg008:005DAF34                 push    dword ptr \[eax\]  
seg008:005DAF36                 call    dword ptr \[eax\-4\]              ;   
  
**SetUnhandledExceptionFilter**  
seg008:005DAF39                 lea     eax, \[ebp+var\_hProcess\]  
seg008:005DAF3F                 push    eax  
seg008:005DAF40                 push    10h  
seg008:005DAF42                 push    eax  
seg008:005DAF43                 call    \[ebp+var\_RtlZeroMemory\]  
seg008:005DAF49                 lea     eax, \[ebp+var\_460\]  
seg008:005DAF4F                 push    eax  
seg008:005DAF50                 push    eax  
seg008:005DAF51                 push    44h  
seg008:005DAF53                 push    eax  
seg008:005DAF54                 call    \[ebp+var\_RtlZeroMemory\]  
seg008:005DAF5A                 call    \[ebp+var\_GetStartupInfoA\]  
seg008:005DAF60                 push    0  
seg008:005DAF62                 push    0  
seg008:005DAF64                 push    0  
seg008:005DAF66                 push    1  
seg008:005DAF68                 push    0  
seg008:005DAF6A                 push    0  
seg008:005DAF6C                 lea     eax, \[ebp+var\_Cstr(PID)\]  
seg008:005DAF72                 push    eax                            ; CommandLine   
  
\= "C40"  
seg008:005DAF73                 lea     eax, \[ebp+var\_RlZMemDestination\]  
seg008:005DAF79                 push    eax  
seg008:005DAF7A                 call    \[ebp+var\_CreateProcessA\]  
seg008:005DAF80                 test    eax, eax  
seg008:005DAF82                 push    11170h  
seg008:005DAF87                 push    \[ebp+var\_hEvent\]  
seg008:005DAF8D                 call    \[ebp+var\_WaitForsingleObject\]  
seg008:005DAF93                 cmp     eax, 102h  
seg008:005DAF98                 jnz     short loc\_5DAFA4  
seg008:005DAF9A                 call    loc\_5D9B1E  
seg008:005DAF9F                 jmp     Over  
seg008:005DAFA4 ; ------------------------------------------------------------------  
  
\----------  
seg008:005DAFA4  
seg008:005DAFA4 loc\_5DAFA4:                                            ; CODE XREF:   
  
Decrypt\_Code+DDEj  
seg008:005DAFA4                 push    \[ebp+var\_hWrite2\]  
seg008:005DAFAA                 pop     \[ebp+var\_hWrite1\]  
seg008:005DAFB0                 call    GetCodeBase  
seg008:005DAFB5                 add     eax, 96h  
seg008:005DAFBA                 push    eax  
seg008:005DAFBB                 call    \[ebp+\_InitCriticalSection\]  
seg008:005DAFC1                 mov     \[ebp+var\_9E0\], 28h  
seg008:005DAFCB                 mov     eax, 669h  
seg008:005DAFD0                 xor     edx, edx  
seg008:005DAFD2                 mov     \[ebp+var\_9E8\], edx  
seg008:005DAFD8                 mov     ecx, 0F0h  
seg008:005DAFDD                 mov     \[ebp+var\_9E4\], ecx  
seg008:005DAFE3                 div     ecx  
seg008:005DAFE5                 mov     esi, eax  
seg008:005DAFE7                 mov     ecx, eax  
seg008:005DAFE9                 push    edx  
seg008:005DAFEA  
seg008:005DAFEA loc\_5DAFEA:                                            ; CODE XREF:   
  
Decrypt\_Code+E5Aj  
seg008:005DAFEA                 push    ecx  
seg008:005DAFEB                 movzx   eax, cl  
seg008:005DAFEE                 ror     eax, 8  
seg008:005DAFF1                 or      eax, \[ebp+var\_9E0\]  
seg008:005DAFF7                 ror     eax, 4  
seg008:005DAFFA                 push    eax  
seg008:005DAFFB                 lea     eax, \[ebp+var\_hWrite2\]  
seg008:005DB001                 push    \[ebp+var\_9E4\]  
seg008:005DB007                 push    \[ebp+var\_9E8\]  
seg008:005DB00D                 push    eax  
seg008:005DB00E                 call    Write\_File                     ; 解密代码  
seg008:005DB013                 pop     ecx  
seg008:005DB014                 loop    loc\_5DAFEA  
seg008:005DB016                 pop     edx  
seg008:005DB017                 lea     ecx, \[ebp+var\_hWrite2\]  
seg008:005DB01D                 mov     eax, edx  
seg008:005DB01F                 ror     edx, 8  
seg008:005DB022                 or      edx, \[ebp+var\_9E0\]  
seg008:005DB028                 ror     edx, 4  
seg008:005DB02B                 push    edx  
seg008:005DB02C                 push    eax  
seg008:005DB02D                 push    \[ebp+var\_9E8\]  
seg008:005DB033                 push    ecx  
seg008:005DB034                 call    Write\_File  
seg008:005DB039                 push    \[ebp+var\_hEvent\]  
seg008:005DB03F                 call    \[ebp+var\_ReSetEvent\]  
seg008:005DB045                 push    80h  
seg008:005DB04A                 call    \[ebp+var\_Sleep\]  
seg008:005DB050                 push    11170h  
seg008:005DB055                 push    \[ebp+var\_hEvent\]  
seg008:005DB05B                 call    \[ebp+var\_WaitForsingleObject\]  
seg008:005DB061                 mov     eax, \[ebp+var\_PEB1\]  
seg008:005DB067                 xor     cl, cl  
seg008:005DB069                 mov     \[eax+2\], cl  
seg008:005DB06C  
seg008:005DB06C Over:                                                  ; CODE XREF:   
  
Decrypt\_Code+66Cj  
seg008:005DB06C                                                        ;   
  
Decrypt\_Code+C99j ...  
seg008:005DB06C                 mov     edi, \[ebp+var\_CodeBase\]  
seg008:005DB072                 add     edi, 1351h  
seg008:005DB078                 lea     esi, \[ebp+var\_ReSetEvent\]  
seg008:005DB07E                 mov     ecx, 30h  
seg008:005DB083  
seg008:005DB083 loc\_5DB083:                                            ; CODE XREF:   
  
Decrypt\_Code+ECDj  
seg008:005DB083                 std  
seg008:005DB084                 lodsd  
seg008:005DB085                 cld  
seg008:005DB086                 stosd  
seg008:005DB087                 loop    loc\_5DB083  
seg008:005DB089                 call    GetCodeBase  
seg008:005DB08E                 mov     edx, eax  
seg008:005DB090                 add     eax, 726h  
seg008:005DB095                 add     edx, 1231h  
seg008:005DB09B                 movzx   edx, byte ptr \[edx+68h\]  
seg008:005DB09F                 lea     esi, \[eax+edx\]  
seg008:005DB0A2                 mov     edi, eax  
seg008:005DB0A4                 mov     ecx, 29Ch  
seg008:005DB0A9  
seg008:005DB0A9 loc\_5DB0A9:                                            ; CODE XREF:   
  
Decrypt\_Code+EF3j  
seg008:005DB0A9                 lodsb  
seg008:005DB0AA                 xor     al, \[esi\]  
seg008:005DB0AC                 stosb  
seg008:005DB0AD                 loop    loc\_5DB0A9  
seg008:005DB0AF                 pop     edi  
seg008:005DB0B0                 pop     esi  
seg008:005DB0B1                 mov     eax, \[ebp+var\_CodeBase\]  
seg008:005DB0B7                 int     3                              ; Trap to   
  
Debugger  
seg008:005DB0B8                 nop  
seg008:005DB0B9                 push    0  
seg008:005DB0BB                 call    dword ptr \[eax+4\]              ; ExitProcess  
seg008:005DB0BE                 pop     edx  
seg008:005DB0BF                 pop     ebx  
seg008:005DB0C0                 pop     edi  
seg008:005DB0C1                 pop     esi  
seg008:005DB0C2                 leave  
seg008:005DB0C3                 retn  
seg008:005DB0C3 Decrypt\_Code    endp  
seg008:005DB0C3  
......  
这么长看了晕不晕?晕呀，我也一样哦:-),休息去,下次再继续,嘿嘿!!!!!!!!!!!  
  
各模块清单:  
  
Cstr\_PID\_       proc near                              ; CODE XREF:   
  
Decrypt\_Code+36Fp  
seg008:005DA142                                                        ;   
  
Decrypt\_Code+410p  
seg008:005DA142  
seg008:005DA142 arg\_0           = dword ptr  8  
seg008:005DA142 arg\_4           = dword ptr  0Ch  
seg008:005DA142  
seg008:005DA142                 push    ebp  
seg008:005DA143                 mov     ebp, esp  
seg008:005DA145                 push    esi  
seg008:005DA146                 push    ecx  
seg008:005DA147                 push    edi  
seg008:005DA148                 push    edx  
seg008:005DA149                 mov     edi, \[ebp+arg\_4\]  
seg008:005DA14C                 mov     edx, \[ebp+arg\_0\]  
seg008:005DA14F                 xor     ecx, ecx  
seg008:005DA151                 rol     edx, 4  
seg008:005DA154                 inc     ecx  
seg008:005DA155  
seg008:005DA155 loc\_5DA155:                                            ; CODE XREF:   
  
Cstr\_PID\_+19j  
seg008:005DA155                 rol     edx, 4  
seg008:005DA158                 inc     ecx  
seg008:005DA159                 or      dl, dl  
seg008:005DA15B                 jz      short loc\_5DA155  
seg008:005DA15D                 mov     al, dl  
seg008:005DA15F                 jmp     short loc\_5DA16F  
seg008:005DA161 ; ------------------------------------------------------------------  
  
\----------  
seg008:005DA161  
seg008:005DA161 loc\_5DA161:                                            ; CODE XREF:   
  
Cstr\_PID\_+30j  
seg008:005DA161                 and     al, 0Fh  
seg008:005DA163                 cmp     al, 0Ah  
seg008:005DA165                 sbb     al, 69h  
seg008:005DA167                 das  
seg008:005DA168                 stosb  
seg008:005DA169                 rol     edx, 4  
seg008:005DA16C                 inc     ecx  
seg008:005DA16D                 mov     al, dl  
seg008:005DA16F  
seg008:005DA16F loc\_5DA16F:                                            ; CODE XREF:   
  
Cstr\_PID\_+1Dj  
seg008:005DA16F                 cmp     ecx, 8  
seg008:005DA172                 jbe     short loc\_5DA161  
seg008:005DA174                 xor     al, al  
seg008:005DA176                 stosb  
seg008:005DA177                 pop     edx  
seg008:005DA178                 pop     edi  
seg008:005DA179                 pop     ecx  
seg008:005DA17A                 pop     esi  
seg008:005DA17B                 leave  
seg008:005DA17C                 retn    8  
seg008:005DA17C Cstr\_PID\_       endp  
seg008:005DA17C  
......  
  
seg008:005DB0F8 ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E   
  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
seg008:005DB0F8  
seg008:005DB0F8 ; Attributes: bp-based frame  
seg008:005DB0F8  
seg008:005DB0F8 Write\_File      proc near                              ; CODE XREF:   
  
Decrypt\_Code+56Ap  
seg008:005DB0F8                                                        ;   
  
Decrypt\_Code+588p ...  
seg008:005DB0F8  
seg008:005DB0F8 var\_12C         = dword ptr -12Ch  
seg008:005DB0F8 var\_28          = dword ptr -28h  
seg008:005DB0F8 var\_24          = dword ptr -24h  
seg008:005DB0F8 var\_1C          = dword ptr -1Ch  
seg008:005DB0F8 var\_18          = dword ptr -18h  
seg008:005DB0F8 var\_WriteFile   = dword ptr -10h  
seg008:005DB0F8 var\_C           = dword ptr -0Ch  
seg008:005DB0F8 var\_8           = dword ptr -8  
seg008:005DB0F8 var\_ReadFile    = dword ptr -4  
seg008:005DB0F8 arg\_0           = dword ptr  8  
seg008:005DB0F8 arg\_4           = dword ptr  0Ch  
seg008:005DB0F8 arg\_DeCryptSize = dword ptr  10h  
seg008:005DB0F8 arg\_C           = dword ptr  14h  
seg008:005DB0F8  
seg008:005DB0F8                 push    ebp  
seg008:005DB0F9                 mov     ebp, esp  
seg008:005DB0FB                 add     esp, 0FFFFFED4h  
seg008:005DB101                 push    ebx  
seg008:005DB102                 push    esi  
seg008:005DB103                 push    edi  
seg008:005DB104                 lea     edi, \[ebp+var\_24\]  
seg008:005DB107                 mov     esi, \[ebp+arg\_0\]  
seg008:005DB10A                 mov     ecx, 9  
seg008:005DB10F                 rep movsd  
seg008:005DB111                 lea     ebx, \[ebp+var\_12C\]  
seg008:005DB117                 cmp     \[ebp+arg\_4\], 0  
seg008:005DB11B                 jz      short loc\_5DB120  
seg008:005DB11D                 mov     ebx, \[ebp+arg\_4\]  
seg008:005DB120  
seg008:005DB120 loc\_5DB120:                                            ; CODE XREF:   
  
Write\_File+23j  
seg008:005DB120                 lea     esi, \[ebp+var\_28\]  
seg008:005DB123                 call    GetCodeBase  
seg008:005DB128                 add     eax, 96h  
seg008:005DB12D                 mov     edi, eax  
seg008:005DB12F                 mov     eax, \[ebp+arg\_C\]  
seg008:005DB132                 shld    ecx, eax, 18h  
seg008:005DB136                 test    ecx, 1  
seg008:005DB13C                 jz      short loc\_5DB150  
seg008:005DB13E                 mov     ecx, \[ebp+var\_ReadFile\]  
seg008:005DB141                 xchg    ecx, \[ebp+var\_WriteFile\]  
seg008:005DB144                 mov     \[ebp+var\_ReadFile\], ecx  
seg008:005DB147                 mov     ecx, \[ebp+var\_18\]  
seg008:005DB14A                 xchg    ecx, \[ebp+var\_1C\]  
seg008:005DB14D                 mov     \[ebp+var\_18\], ecx  
seg008:005DB150  
seg008:005DB150 loc\_5DB150:                                            ; CODE XREF:   
  
Write\_File+44j  
seg008:005DB150                 test    al, 1  
seg008:005DB152                 jnz     short loc\_5DB16C  
seg008:005DB154                 push    0  
seg008:005DB156                 push    esi  
seg008:005DB157                 push    \[ebp+arg\_DeCryptSize\]  
seg008:005DB15A                 push    ebx  
seg008:005DB15B                 push    \[ebp+var\_18\]  
seg008:005DB15E                 call    \[ebp+var\_ReadFile\]  
seg008:005DB161                 test    eax, eax  
seg008:005DB163                 jz      short \_ReadFail  
seg008:005DB165                 mov     eax, \[ebp+arg\_DeCryptSize\]  
seg008:005DB168                 cmp     \[esi\], eax                     ; \[esi\]和 eax 保  
  
存着解密代码大小  
seg008:005DB16A                 jnz     short \_ReadFail  
seg008:005DB16C  
seg008:005DB16C loc\_5DB16C:                                            ; CODE XREF:   
  
Write\_File+5Aj  
seg008:005DB16C                 push    edi  
seg008:005DB16D                 call    \[ebp+var\_8\]  
seg008:005DB170                 mov     eax, \[ebp+arg\_C\]  
seg008:005DB173                 rol     eax, 1  
seg008:005DB175                 test    al, 1  
seg008:005DB177                 jz      short loc\_5DB184  
seg008:005DB179                 mov     ecx, \[esi\]  
seg008:005DB17B                 rol     eax, 0Bh  
seg008:005DB17E  
seg008:005DB17E loc\_5DB17E:                                            ; CODE XREF:   
  
Write\_File+8Aj  
seg008:005DB17E                 xor     \[ecx+ebx\-1\], al  
seg008:005DB182                 loop    loc\_5DB17E  
seg008:005DB184  
seg008:005DB184 loc\_5DB184:                                            ; CODE XREF:   
  
Write\_File+7Fj  
seg008:005DB184                 push    edi  
seg008:005DB185                 call    \[ebp+var\_C\]  
seg008:005DB188                 mov     eax, \[ebp+arg\_C\]  
seg008:005DB18B                 test    eax, 3  
seg008:005DB190                 jz      short \_ReadFail  
seg008:005DB192                 push    0  
seg008:005DB194                 push    esi  
seg008:005DB195                 push    \[ebp+arg\_DeCryptSize\]  
seg008:005DB198                 lea     ecx, \[ebp+var\_12C\]  
seg008:005DB19E                 cmp     \[ebp+arg\_4\], 0  
seg008:005DB1A2                 jz      short loc\_5DB1A7  
seg008:005DB1A4                 mov     ecx, \[ebp+arg\_4\]  
seg008:005DB1A7  
seg008:005DB1A7 loc\_5DB1A7:                                            ; CODE XREF:   
  
Write\_File+AAj  
seg008:005DB1A7                 push    ecx  
seg008:005DB1A8                 push    \[ebp+var\_1C\]  
seg008:005DB1AB                 call    \[ebp+var\_WriteFile\]  
seg008:005DB1AE  
seg008:005DB1AE \_ReadFail:                                             ; CODE XREF:   
  
Write\_File+6Bj  
seg008:005DB1AE                                                        ;   
  
Write\_File+72j ...  
seg008:005DB1AE                 pop     edi  
seg008:005DB1AF                 pop     esi  
seg008:005DB1B0                 pop     ebx  
seg008:005DB1B1                 leave  
seg008:005DB1B2                 retn    10h  
seg008:005DB1B2 Write\_File      endp  
seg008:005DB1B2  
......  
seg008:005DB1B5 ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E   
  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
seg008:005DB1B5  
seg008:005DB1B5 ; Attributes: bp-based frame  
seg008:005DB1B5  
seg008:005DB1B5 ; int \_\_stdcall Get\_Privileges(int GetCurrentProc,int   
  
OpenProcessToke,int LookupPrivilegeValueA,int **AdjustTokenPrivileges**,int **CloseHandle**)  
seg008:005DB1B5 Get\_Privileges  proc near                              ; CODE XREF:   
  
Decrypt\_Code+62Ap  
seg008:005DB1B5  
seg008:005DB1B5 var\_handle      = dword ptr -34h  
seg008:005DB1B5 var\_pLocalId    = dword ptr -24h  
seg008:005DB1B5 var\_1C          = dword ptr -1Ch  
seg008:005DB1B5 var\_18          = dword ptr -18h  
seg008:005DB1B5 var\_14          = dword ptr -14h  
seg008:005DB1B5 var\_10          = dword ptr -10h  
seg008:005DB1B5 var\_C           = dword ptr -0Ch  
seg008:005DB1B5 var\_8           = dword ptr -8  
seg008:005DB1B5 var\_4           = dword ptr -4  
seg008:005DB1B5 GetCurrentProc  = dword ptr  8  
seg008:005DB1B5 OpenProcessToke = dword ptr  0Ch  
seg008:005DB1B5 LookupPrivilegeValueA= dword ptr  10h  
seg008:005DB1B5 **AdjustTokenPrivileges**\= dword ptr  14h  
seg008:005DB1B5 **CloseHandle**     = dword ptr  18h  
seg008:005DB1B5  
seg008:005DB1B5                 push    ebp  
seg008:005DB1B6                 mov     ebp, esp  
seg008:005DB1B8                 sub     esp, 1Ch  
seg008:005DB1BB                 lea     eax, \[esp+1Ch+var\_1C\]  
seg008:005DB1BE                 push    esi  
seg008:005DB1BF                 push    eax  
seg008:005DB1C0                 push    28h  
seg008:005DB1C2                 xor     esi, esi  
seg008:005DB1C4                 call    \[ebp+GetCurrentProc\]  
seg008:005DB1C7                 push    eax  
seg008:005DB1C8                 call    \[ebp+OpenProcessToke\]  
seg008:005DB1CB                 test    eax, eax  
seg008:005DB1CD                 jnz     short loc\_5DB1D1  
seg008:005DB1CF                 jmp     short loc\_5DB236  
seg008:005DB1D1 ; ------------------------------------------------------------------  
  
\----------  
seg008:005DB1D1  
seg008:005DB1D1 loc\_5DB1D1:                                            ; CODE XREF:   
  
Get\_Privileges+18j  
seg008:005DB1D1                 lea     ecx, \[esp+2Ch+var\_pLocalId\]  
seg008:005DB1D5                 push    ecx  
seg008:005DB1D6                 call    loc\_5DB1EC  
seg008:005DB1D6 ; ------------------------------------------------------------------  
  
\----------  
seg008:005DB1DB aSedebugprivile db 'SeDebugPrivilege',0  
seg008:005DB1EC ; ------------------------------------------------------------------  
  
\----------  
seg008:005DB1EC  
seg008:005DB1EC loc\_5DB1EC:                                            ; CODE XREF:   
  
Get\_Privileges+21p  
seg008:005DB1EC                 push    0  
seg008:005DB1EE                 call    \[ebp+LookupPrivilegeValueA\]  
seg008:005DB1F1                 test    eax, eax  
seg008:005DB1F3                 jz      short loc\_5DB22C  
seg008:005DB1F5                 mov     edx, \[esp+20h+var\_18\]  
seg008:005DB1F9                 mov     eax, \[esp+20h+var\_14\]  
seg008:005DB1FD                 push    0  
seg008:005DB1FF                 push    0  
seg008:005DB201                 lea     ecx, \[esp+28h+var\_10\]  
seg008:005DB205                 mov     \[esp+28h+var\_C\], edx  
seg008:005DB209                 mov     edx, \[esp+28h+var\_1C\]  
seg008:005DB20D                 push    10h  
seg008:005DB20F                 push    ecx  
seg008:005DB210                 push    0  
seg008:005DB212                 push    edx  
seg008:005DB213                 mov     \[esp+38h+var\_10\], 1  
seg008:005DB21B                 mov     \[esp+38h+var\_4\], 2  
seg008:005DB223                 mov     \[esp+38h+var\_8\], eax  
seg008:005DB227                 call    \[ebp+**AdjustTokenPrivileges**\]  
seg008:005DB22A                 mov     esi, eax  
seg008:005DB22C  
seg008:005DB22C loc\_5DB22C:                                            ; CODE XREF:   
  
Get\_Privileges+3Ej  
seg008:005DB22C                 mov     eax, \[esp+38h+var\_handle\]  
seg008:005DB230                 push    eax  
seg008:005DB231                 call    \[ebp+**CloseHandle**\]  
seg008:005DB234                 mov     eax, esi  
seg008:005DB236  
seg008:005DB236 loc\_5DB236:                                            ; CODE XREF:   
  
Get\_Privileges+1Aj  
seg008:005DB236                 pop     esi  
seg008:005DB237                 add     esp, 1Ch  
seg008:005DB23A                 leave  
seg008:005DB23B                 retn    14h  
seg008:005DB23B Get\_Privileges  endp  
seg008:005DB23B  
seg008:005DB23E ; ------------------------------------------------------------------  
  
\----------  
......  
seg008:005DB0C4 ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E   
  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
seg008:005DB0C4  
seg008:005DB0C4 ; Attributes: bp-based frame  
seg008:005DB0C4  
seg008:005DB0C4 Getjmpaddress   proc near                              ; CODE XREF:   
  
Decrypt\_Code+8D7p  
seg008:005DB0C4                                                        ;   
  
Decrypt\_Code+9CDp ...  
seg008:005DB0C4  
seg008:005DB0C4 arg\_4           = dword ptr  0Ch  
seg008:005DB0C4  
seg008:005DB0C4                 push    ebp  
seg008:005DB0C5                 mov     ebp, esp  
seg008:005DB0C7                 shr     ecx, 2  
seg008:005DB0CA                 repne scasd  
seg008:005DB0CC                 jnz     short loc\_5DB0F2  
seg008:005DB0CE                 mov     ecx, \[edi\]                     ; 关键表 1  
seg008:005DB0D0                 sub     ecx, 8  
seg008:005DB0D3                 add     edi, 4  
seg008:005DB0D6                 shr     ecx, 1  
seg008:005DB0D8                 mov     eax, edx  
seg008:005DB0DA                 and     eax, 0FFFh  
seg008:005DB0DF                 repne scasw  
seg008:005DB0E2                 jnz     short loc\_5DB0F2  
seg008:005DB0E4                 xor     eax, eax  
seg008:005DB0E6                 mov     ax, \[edi\]  
seg008:005DB0E9                 add     eax, \[ebp+arg\_4\]               ; 关键表 2  
seg008:005DB0EC                 mov     eax, \[eax\]  
seg008:005DB0EE                 leave  
seg008:005DB0EF                 retn    8  
seg008:005DB0F2 ; ------------------------------------------------------------------  
  
\----------  
......  
seg008:005DA17F ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E   
  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
seg008:005DA17F  
seg008:005DA17F ; Attributes: bp-based frame  
seg008:005DA17F  
seg008:005DA17F WritePresentMem proc near                              ; CODE XREF:   
  
Decrypt\_Code+8F7p  
seg008:005DA17F                                                        ;   
  
Decrypt\_Code+9F1p ...  
seg008:005DA17F  
seg008:005DA17F var\_10          = dword ptr -10h  
seg008:005DA17F var\_4           = dword ptr -4  
seg008:005DA17F arg\_WriteProcessMem= dword ptr  8  
seg008:005DA17F arg\_4           = dword ptr  0Ch  
seg008:005DA17F arg\_8           = dword ptr  10h  
seg008:005DA17F arg\_C           = dword ptr  14h  
seg008:005DA17F  
seg008:005DA17F                 push    ebp  
seg008:005DA180                 mov     ebp, esp  
seg008:005DA182                 add     esp, 0FFFFFFF0h  
seg008:005DA185                 push    esi  
seg008:005DA186                 push    edi  
seg008:005DA187                 push    ebx  
seg008:005DA188                 push    edx  
seg008:005DA189                 lea     edi, \[ebp+var\_10\]  
seg008:005DA18C                 mov     \[edi\], al  
seg008:005DA18E                 mov     eax, \[ebp+arg\_8\]  
seg008:005DA191                 sub     eax, \[ebp+arg\_C\]  
seg008:005DA194                 sub     eax, 5  
seg008:005DA197                 mov     \[edi+1\], eax  
seg008:005DA19A                 mov     \[ebp+var\_4\], 5  
seg008:005DA1A1                 lea     eax, \[ebp+var\_4\]  
seg008:005DA1A4                 push    eax  
seg008:005DA1A5                 push    \[ebp+var\_4\]  
seg008:005DA1A8                 push    edi  
seg008:005DA1A9                 push    \[ebp+arg\_C\]  
seg008:005DA1AC                 push    \[ebp+arg\_4\]  
seg008:005DA1AF                 call    \[ebp+arg\_WriteProcessMem\]  
seg008:005DA1B2                 pop     edx  
seg008:005DA1B3                 pop     ebx  
seg008:005DA1B4                 pop     edi  
seg008:005DA1B5                 pop     esi  
seg008:005DA1B6                 leave  
seg008:005DA1B7                 retn    10h  
seg008:005DA1B7 WritePresentMem endp  
seg008:005DA1B7  
......  
  
g008:005D99D4 ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E   
  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
seg008:005D99D4  
seg008:005D99D4 ; Attributes: bp-based frame  
seg008:005D99D4  
seg008:005D99D4 GETAPIADDR      proc near                              ; CODE XREF:   
  
seg008:005D96E1p  
seg008:005D99D4  
seg008:005D99D4 var\_130         = dword ptr -130h  
seg008:005D99D4 var\_128         = dword ptr -128h  
seg008:005D99D4 var\_GetProcAddresVA= dword ptr -118h  
seg008:005D99D4 var\_GetProcAddressRVA= dword ptr -114h  
seg008:005D99D4 var\_KerBASE     = dword ptr -108h  
seg008:005D99D4 var\_CodeBase    = dword ptr -104h  
seg008:005D99D4 arg\_0           = dword ptr  8  
seg008:005D99D4  
seg008:005D99D4                 push    ebp  
seg008:005D99D5                 mov     ebp, esp  
seg008:005D99D7                 add     esp, 0FFFFFEE8h  
seg008:005D99DD                 push    esi  
seg008:005D99DE                 push    edi  
seg008:005D99DF                 call    GetCodeBase                    ; 获取段的起始  
  
地址  
seg008:005D99E4                 mov     \[ebp+var\_CodeBase\], eax  
seg008:005D99EA                 mov     al, 2Ch  
seg008:005D99EC                 mov     \[ebp+var\_GetProcAddressRVA\], eax  
seg008:005D99F2                 mov     esi, eax  
seg008:005D99F4                 lodsd  
seg008:005D99F5                 mov     \[ebp+var\_GetProcAddresVA\], eax  
seg008:005D99FB                 xor     ax, ax  
seg008:005D99FE                 mov     dx, 'ZM'                       ; 准备定  
  
位 KERBASE  
seg008:005D9A02                 jmp     short @1  
seg008:005D9A04 ; ------------------------------------------------------------------  
  
\----------  
seg008:005D9A04  
seg008:005D9A04 @B:                                                    ; CODE XREF:   
  
GETAPIADDR+37j  
seg008:005D9A04                 dec     eax  
seg008:005D9A05                 xor     ax, ax  
seg008:005D9A08  
seg008:005D9A08 @1:                                                    ; CODE XREF:   
  
GETAPIADDR+2Ej  
seg008:005D9A08                 cmp     \[eax\], dx  
seg008:005D9A0B                 jnz     short @B                       ; 循环的方式获  
  
取 DOS HEADER  
seg008:005D9A0D                 mov     edi, \[ebp+var\_CodeBase\]  
seg008:005D9A13                 add     edi, 62h  
seg008:005D9A19                 stosd  
seg008:005D9A1A                 mov     \[ebp+var\_KerBASE\], eax         ; 保  
  
存 kernel32.dll 的句柄  
seg008:005D9A20                 call    @2                             ; Kernel base  
seg008:005D9A20 ; ------------------------------------------------------------------  
  
\----------  
seg008:005D9A25 aExitprocess    db 'ExitProcess',0  
seg008:005D9A31 ; ------------------------------------------------------------------  
  
\----------  
seg008:005D9A31  
seg008:005D9A31 @2:                                                    ; CODE XREF:   
  
GETAPIADDR+4Cp  
seg008:005D9A31                 push    eax                            ; Kernel base  
seg008:005D9A32                 call    \[ebp+var\_GetProcAddresVA\]  
seg008:005D9A38                 mov     edx, \[ebp+var\_CodeBase\]  
seg008:005D9A3E                 mov     \[edx+4\], eax  
seg008:005D9A41                 call    @3  
seg008:005D9A41 ; ------------------------------------------------------------------  
  
\----------  
seg008:005D9A46 aUnhandledexceptionfi db 'UnhandledExceptionFilter',0  
seg008:005D9A5F ; ------------------------------------------------------------------  
  
\----------  
seg008:005D9A5F  
seg008:005D9A5F @3:                                                    ; CODE XREF:   
  
GETAPIADDR+6Dp  
seg008:005D9A5F                 push    \[ebp+var\_KerBASE\]  
seg008:005D9A65                 call    \[ebp+var\_GetProcAddresVA\]  
seg008:005D9A6B                 rol     eax, 2  
seg008:005D9A6E                 push    eax  
seg008:005D9A6F                 mov     eax, \[ebp+var\_CodeBase\]  
seg008:005D9A75                 add     eax, 0BDh  
seg008:005D9A7A                 pop     dword ptr \[eax\]  
seg008:005D9A7C                 call    @4  
seg008:005D9A7C ; ------------------------------------------------------------------  
  
\----------  
seg008:005D9A81 aVirtualalloc   db 'VirtualAlloc',0  
seg008:005D9A8E ; ------------------------------------------------------------------  
  
\----------  
seg008:005D9A8E  
seg008:005D9A8E @4:                                                    ; CODE XREF:   
  
GETAPIADDR+A8p  
seg008:005D9A8E                 push    \[ebp+var\_KerBASE\]  
seg008:005D9A94                 call    \[ebp+var\_GetProcAddresVA\]  
seg008:005D9A9A  
seg008:005D9A9A @5:  
seg008:005D9A9A                 or      eax, eax  
seg008:005D9A9C                 jz      short Exit\_Proc  
seg008:005D9A9E                 push    eax  
seg008:005D9A9F                 sub     esi, 4  
seg008:005D9AA2                 mov     edi, esi  
seg008:005D9AA4                 lodsd  
seg008:005D9AA5                 xchg    eax, \[esp+128h+var\_128\]  
seg008:005D9AA8                 stosd  
seg008:005D9AA9                 pop     eax  
seg008:005D9AAA                 sub     edi, 0Ch  
seg008:005D9AAD                 stosd  
seg008:005D9AAE                 mov     eax, 0BA1h  
seg008:005D9AB3  
seg008:005D9AB3 @6:  
seg008:005D9AB3                 add     eax, \[ebp+var\_CodeBase\]  
seg008:005D9AB9                 push    eax  
seg008:005D9ABA                 call    @7  
seg008:005D9ABA ; ------------------------------------------------------------------  
  
\----------  
seg008:005D9ABF aSetunhandledexceptio db 'SetUnhandledExceptionFilter',0  
seg008:005D9ADB ; ------------------------------------------------------------------  
  
\----------  
seg008:005D9ADB  
seg008:005D9ADB @7:                                                    ; CODE XREF:   
  
GETAPIADDR+E6p  
seg008:005D9ADB                 push    \[ebp+var\_KerBASE\]  
seg008:005D9AE1                 call    \[ebp+var\_GetProcAddresVA\]  
seg008:005D9AE7                 mov     edx, \[ebp+var\_CodeBase\]  
seg008:005D9AED                 add     edx, 86h  
seg008:005D9AF3                 mov     \[edx\], eax  
seg008:005D9AF5                 call    eax  
seg008:005D9AF7                 push    eax  
seg008:005D9AF8                 push    \[ebp+var\_CodeBase\]  
seg008:005D9AFE                 add     \[esp+130h+var\_130\], 8Ah  
seg008:005D9B05                 pop     eax  
seg008:005D9B06                 pop     dword ptr \[eax\]  
seg008:005D9B08                 jmp     short ok\_way  
seg008:005D9B0A ; ------------------------------------------------------------------  
  
\----------  
seg008:005D9B0A  
seg008:005D9B0A Exit\_Proc:                                             ; CODE XREF:   
  
GETAPIADDR+C8j  
seg008:005D9B0A                 mov     eax, \[ebp+var\_CodeBase\]  
seg008:005D9B10                 push    0FFFFFFFFh  
seg008:005D9B12                 call    dword ptr \[eax+4\]  
seg008:005D9B15  
seg008:005D9B15 ok\_way:                                                ; CODE XREF:   
  
GETAPIADDR+134j  
seg008:005D9B15                 mov     edx, \[ebp+arg\_0\]  
seg008:005D9B18                 pop     edi  
seg008:005D9B19                 pop     esi  
seg008:005D9B1A                 leave  
seg008:005D9B1B                 retn    4  
seg008:005D9B1B GETAPIADDR      endp  
seg008:005D9B1B  
seg008:005D9B1E ; ------------------------------------------------------------------  
  
\----------  
  
  
  
  
g008:005D99C2 ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E   
  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
seg008:005D99C2  
seg008:005D99C2 ; Attributes: bp-based frame  
seg008:005D99C2  
seg008:005D99C2 UnPackCode\_5D96EE proc near                            ; CODE XREF:   
  
DeCode+2D8p  
seg008:005D99C2                 push    ebp  
seg008:005D99C3                 mov     ebp, esp  
seg008:005D99C5                 mov     eax, \[ebp+8\]  
seg008:005D99C8  
seg008:005D99C8 loc\_5D99C8:                                            ; CODE XREF:   
  
UnPackCode\_5D96EE+Cj  
seg008:005D99C8                 sub     \[eax+ecx\-1\], dl  
seg008:005D99CC                 rol     edx, cl  
seg008:005D99CE                 loop    loc\_5D99C8  
seg008:005D99D0                 leave  
seg008:005D99D1                 retn    4  
seg008:005D99D1 UnPackCode\_5D96EE endp  
seg008:005D99D1; -------------------------------------------------------------------  
  
\---------  
  
  
g008:005DB23E ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E   
  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
seg008:005DB23E  
seg008:005DB23E ; Attributes: bp-based frame  
seg008:005DB23E  
seg008:005DB23E DeCode          proc near                              ; CODE XREF:   
  
seg008:005D96E9p  
seg008:005DB23E  
seg008:005DB23E var\_1994        = dword ptr -1994h  
seg008:005DB23E var\_1988        = dword ptr -1988h  
seg008:005DB23E var\_1984        = dword ptr -1984h  
seg008:005DB23E var\_1980        = dword ptr -1980h  
seg008:005DB23E var\_1978        = dword ptr -1978h  
seg008:005DB23E var\_1974        = dword ptr -1974h  
seg008:005DB23E var\_hMem1       = dword ptr -1970h  
seg008:005DB23E \_ZwSetInfoThread= dword ptr -11A0h  
seg008:005DB23E \_ZwQueryInfoThread= dword ptr -119Ch  
seg008:005DB23E var\_CloseHandle = dword ptr -1198h  
seg008:005DB23E var\_OpenProcess = dword ptr -1194h  
seg008:005DB23E var\_GlobalFree  = dword ptr -1190h  
seg008:005DB23E var\_GlobalAlloc = dword ptr -118Ch  
seg008:005DB23E var\_CodeBase    = dword ptr -1188h  
seg008:005DB23E var\_1184        = dword ptr -1184h  
seg008:005DB23E var\_1174        = dword ptr -1174h  
seg008:005DB23E var\_1154        = dword ptr -1154h  
seg008:005DB23E var\_1150        = dword ptr -1150h  
seg008:005DB23E var\_1148        = dword ptr -1148h  
seg008:005DB23E var\_1140        = dword ptr -1140h  
seg008:005DB23E var\_113C        = dword ptr -113Ch  
seg008:005DB23E var\_112C        = dword ptr -112Ch  
seg008:005DB23E var\_28          = dword ptr -28h  
seg008:005DB23E var\_24          = dword ptr -24h  
seg008:005DB23E var\_20          = dword ptr -20h  
seg008:005DB23E var\_1C          = dword ptr -1Ch  
seg008:005DB23E \_ZwQueryObject  = dword ptr -18h  
seg008:005DB23E \_ZwDupObject    = dword ptr -14h  
seg008:005DB23E \_ZwQueryInfoProc= dword ptr -10h  
seg008:005DB23E \_ZwQuerysysInfo = dword ptr -0Ch  
seg008:005DB23E var\_NTdllHandle = dword ptr -8  
seg008:005DB23E var\_hMem        = dword ptr -4  
seg008:005DB23E arg\_PID         = dword ptr  8  
seg008:005DB23E arg\_PeHeaderCValue= dword ptr  0Ch  
seg008:005DB23E  
seg008:005DB23E                 push    ebp  
seg008:005DB23F                 mov     ebp, esp  
seg008:005DB241                 add     esp, 0FFFFEE60h  
seg008:005DB247                 push    esi  
seg008:005DB248                 push    edi  
seg008:005DB249                 push    ebx  
seg008:005DB24A                 xor     eax, eax  
seg008:005DB24C  
seg008:005DB24C loc\_5DB24C:                                            ; CODE XREF:   
  
seg008:005DB1E3j  
seg008:005DB24C                                                        ;   
  
seg008:005DB1E5j  
seg008:005DB24C                 mov     \[ebp+var\_1150\], eax  
seg008:005DB252                 mov     \[ebp+var\_113C\], eax  
seg008:005DB258                 call    GetCodeBase  
seg008:005DB25D                 mov     esi, eax  
seg008:005DB25F                 mov     \[ebp+var\_CodeBase\], eax  
seg008:005DB265                 add     eax, 62h  
seg008:005DB26A                 call    loc\_5DB27A  
seg008:005DB26A ; ------------------------------------------------------------------  
  
\----------  
seg008:005DB26F aGlobalfree     db 'GlobalFree',0  
seg008:005DB27A ; ------------------------------------------------------------------  
  
\----------  
seg008:005DB27A  
seg008:005DB27A loc\_5DB27A:                                            ; CODE XREF:   
  
DeCode+2Cp  
seg008:005DB27A                 push    dword ptr \[eax\]  
seg008:005DB27C                 call    loc\_5DB28D  
seg008:005DB27C ; ------------------------------------------------------------------  
  
\----------  
seg008:005DB281 aClosehandle    db 'CloseHandle',0  
seg008:005DB28D ; ------------------------------------------------------------------  
  
\----------  
seg008:005DB28D  
seg008:005DB28D loc\_5DB28D:                                            ; CODE XREF:   
  
DeCode+3Ep  
seg008:005DB28D                 push    dword ptr \[eax\]  
seg008:005DB28F                 call    loc\_5DB2A0  
seg008:005DB28F ; ------------------------------------------------------------------  
  
\----------  
seg008:005DB294 aOpenprocess    db 'OpenProcess',0  
seg008:005DB2A0 ; ------------------------------------------------------------------  
  
\----------  
seg008:005DB2A0  
seg008:005DB2A0 loc\_5DB2A0:                                            ; CODE XREF:   
  
DeCode+51p  
seg008:005DB2A0                 push    dword ptr \[eax\]  
seg008:005DB2A2                 call    loc\_5DB2B3  
seg008:005DB2A2 ; ------------------------------------------------------------------  
  
\----------  
seg008:005DB2A7 aGlobalalloc    db 'GlobalAlloc',0  
seg008:005DB2B3 ; ------------------------------------------------------------------  
  
\----------  
seg008:005DB2B3  
seg008:005DB2B3 loc\_5DB2B3:                                            ; CODE XREF:   
  
DeCode+64p  
seg008:005DB2B3                 push    dword ptr \[eax\]  
seg008:005DB2B5                 call    loc\_5DB2CE  
seg008:005DB2B5 ; ------------------------------------------------------------------  
  
\----------  
seg008:005DB2BA aGetcurrentprocessid db 'GetCurrentProcessId',0  
seg008:005DB2CE ; ------------------------------------------------------------------  
  
\----------  
seg008:005DB2CE  
seg008:005DB2CE loc\_5DB2CE:                                            ; CODE XREF:   
  
DeCode+77p  
seg008:005DB2CE                 push    dword ptr \[eax\]  
seg008:005DB2D0                 mov     edx, large fs:18h              ; 准备检测调试  
  
器?  
seg008:005DB2D7                 mov     edx, \[edx+30h\]  
seg008:005DB2DA                 mov     \[eax\-4\], edx  
seg008:005DB2DD                 mov     eax, \[ebp+var\_CodeBase\]  
seg008:005DB2E3                 movzx   edi, byte ptr \[eax\]  
seg008:005DB2E6                 call    dword ptr \[edi+esi\-4\]          ; 获  
  
取 GetCurrentProcessId 的实际地址  
seg008:005DB2EA                 call    eax                            ;   
  
**GetCurrentProcessID**  
seg008:005DB2EC                 mov     \[ebp+arg\_PID\], eax  
seg008:005DB2EF                 call    dword ptr \[edi+esi\-4\]          ; 获  
  
取 GlobalAlloc 地址  
seg008:005DB2F3                 mov     \[ebp+var\_GlobalAlloc\], eax  
seg008:005DB2F9                 call    dword ptr \[edi+esi\-4\]          ;   
  
**GetProcAddress**  
seg008:005DB2FD                 mov     \[ebp+var\_OpenProcess\], eax  
seg008:005DB303                 call    dword ptr \[edi+esi\-4\]          ;   
  
**GetProcAddress**  
seg008:005DB307                 mov     \[ebp+var\_CloseHandle\], eax  
seg008:005DB30D                 call    dword ptr \[edi+esi\-4\]          ;   
  
**GetProcAddress**  
seg008:005DB311                 mov     \[ebp+var\_GlobalFree\], eax  
seg008:005DB317                 call    loc\_5DB326                     ; LoadLibraryA  
  
载入 Ntdll.dll  
seg008:005DB317 ; ------------------------------------------------------------------  
  
\----------  
seg008:005DB31C aNtdll\_dll      db 'ntdll.dll',0  
seg008:005DB326 ; ------------------------------------------------------------------  
  
\----------  
seg008:005DB326  
seg008:005DB326 loc\_5DB326:                                            ; CODE XREF:   
  
DeCode+D9p  
seg008:005DB326                 call    dword ptr \[edi+esi\]            ; LoadLibraryA  
  
载入 Ntdll.dll  
seg008:005DB329                 test    eax, eax  
seg008:005DB32B                 jz      Over                           ; 如果载入失败  
  
不处理 WinNt 函数,这里不能跳  
seg008:005DB32B                                                        ; 跳了就 over 了  
  
,不知道这样算不算壳不能在  
seg008:005DB32B                                                        ; Win9X 下运行  
  
呢  
seg008:005DB331                 mov     \[ebp+var\_NTdllHandle\], eax     ; 保  
  
存 NTDLL.dll 的句柄  
seg008:005DB334                 call    loc\_5DB352  
seg008:005DB334 ; ------------------------------------------------------------------  
  
\----------  
seg008:005DB339 aZwquerysysteminforma db 'ZwQuerySystemInformation',0  
seg008:005DB352 ; ------------------------------------------------------------------  
  
\----------  
seg008:005DB352  
seg008:005DB352 loc\_5DB352:                                            ; CODE XREF:   
  
DeCode+F6p  
seg008:005DB352                 push    \[ebp+var\_NTdllHandle\]  
seg008:005DB355                 call    dword ptr \[edi+esi\-4\]          ; 获取 Nt 函数中  
  
的地址了  
seg008:005DB359                 test    eax, eax                       ; 如果获取失败  
  
则 over  
seg008:005DB35B                 jz      Over  
seg008:005DB361                 mov     \[ebp+\_ZwQuerysysInfo\], eax  
seg008:005DB364                 rol     eax, 2  
seg008:005DB367                 push    eax  
seg008:005DB368                 call    GetCodeBase  
seg008:005DB36D                 add     eax, 72h  
seg008:005DB372                 pop     dword ptr \[eax\]                ; ROL   
  
\_ZwQuerysysInfo,2 的值,不明白  
这里 ZwQuerySystemInformation 起了什么作用  
seg008:005DB372                                                        ; 保存  
  
在\[CODEBASE+72H\]=\[005D9072\]处  
seg008:005DB374                 add     eax, 4  
seg008:005DB377                 push    eax  
seg008:005DB378                 call    loc\_5DB394  
seg008:005DB378 ; ------------------------------------------------------------------  
  
\----------  
seg008:005DB37D aZwsetinformationthre db 'ZwSetInformationThread',0  
seg008:005DB394 ; ------------------------------------------------------------------  
  
\----------  
seg008:005DB394  
seg008:005DB394 loc\_5DB394:                                            ; CODE XREF:   
  
DeCode+13Ap  
seg008:005DB394                 push    \[ebp+var\_NTdllHandle\]  
seg008:005DB397                 call    dword ptr \[edi+esi\-4\]          ; 获  
  
取 ZwSetInformationThread 的地址  
seg008:005DB39B                 test    eax, eax  
seg008:005DB39D                 jz      Over  
seg008:005DB3A3                 mov     \[ebp+\_ZwSetInfoThread\], eax  
seg008:005DB3A9                 rol     eax, 2  
seg008:005DB3AC                 xchg    eax, \[esp+118Ch+var\_GlobalAlloc\]  
seg008:005DB3AF                 pop     dword ptr \[eax\]                ; 保存加密后的  
  
地址  
seg008:005DB3B1                 add     eax, 4  
seg008:005DB3B4                 push    eax  
seg008:005DB3B5                 call    loc\_5DB3D3  
seg008:005DB3B5 ; ------------------------------------------------------------------  
  
\----------  
seg008:005DB3BA aZwqueryinformationth db 'ZwQueryInformationThread',0  
seg008:005DB3D3 ; ------------------------------------------------------------------  
  
\----------  
seg008:005DB3D3  
seg008:005DB3D3 loc\_5DB3D3:                                            ; CODE XREF:   
  
DeCode+177p  
seg008:005DB3D3                 push    \[ebp+var\_NTdllHandle\]  
seg008:005DB3D6                 call    dword ptr \[edi+esi\-4\]          ;   
  
**GetProcAddress**  
seg008:005DB3DA                 test    eax, eax  
seg008:005DB3DC                 jz      Over  
seg008:005DB3E2                 mov     \[ebp+\_ZwQueryInfoThread\], eax  
seg008:005DB3E8                 rol     eax, 2  
seg008:005DB3EB                 xchg    eax, \[esp+1188h+var\_CodeBase\]  
seg008:005DB3EE                 pop     dword ptr \[eax\]  
seg008:005DB3F0                 call    loc\_5DB40F  
seg008:005DB3F0 ; ------------------------------------------------------------------  
  
\----------  
seg008:005DB3F5 aZwqueryinformationpr db 'ZwQueryInformationProcess',0  
seg008:005DB40F ; ------------------------------------------------------------------  
  
\----------  
seg008:005DB40F  
seg008:005DB40F loc\_5DB40F:                                            ; CODE XREF:   
  
DeCode+1B2p  
seg008:005DB40F                 push    \[ebp+var\_NTdllHandle\]  
seg008:005DB412                 call    dword ptr \[edi+esi\-4\]  
seg008:005DB416                 test    eax, eax  
seg008:005DB418                 jz      Over  
seg008:005DB41E                 mov     \[ebp+\_ZwQueryInfoProc\], eax  
seg008:005DB421                 call    loc\_5DB438  
seg008:005DB421 ; ------------------------------------------------------------------  
  
\----------  
seg008:005DB426 aZwduplicateobject db 'ZwDuplicateObject',0  
seg008:005DB438 ; ------------------------------------------------------------------  
  
\----------  
seg008:005DB438  
seg008:005DB438 loc\_5DB438:                                            ; CODE XREF:   
  
DeCode+1E3p  
seg008:005DB438                 push    \[ebp+var\_NTdllHandle\]  
seg008:005DB43B                 call    dword ptr \[edi+esi\-4\]  
seg008:005DB43F                 test    eax, eax  
seg008:005DB441                 mov     \[ebp+\_ZwDupObject\], eax        ; 这里和前面的  
  
有点不同，先保存后判断  
seg008:005DB444                 jz      Over  
seg008:005DB44A                 call    loc\_5DB45D  
seg008:005DB44A ; ------------------------------------------------------------------  
  
\----------  
seg008:005DB44F aZwqueryobject  db 'ZwQueryObject',0  
seg008:005DB45D ; ------------------------------------------------------------------  
  
\----------  
seg008:005DB45D  
seg008:005DB45D loc\_5DB45D:                                            ; CODE XREF:   
  
DeCode+20Cp  
seg008:005DB45D                 push    \[ebp+var\_NTdllHandle\]  
seg008:005DB460                 call    dword ptr \[edi+esi\-4\]          ;   
  
**GetProcAddress**  
seg008:005DB464                 test    eax, eax  
seg008:005DB466                 mov     \[ebp+\_ZwQueryObject\], eax  
seg008:005DB469                 jz      Over  
seg008:005DB46F                 push    esi  
seg008:005DB470                 push    edi  
seg008:005DB471                 mov     eax, \[ebp+var\_CodeBase\]  
seg008:005DB477                 add     eax, 5Eh  
seg008:005DB47C                 mov     edx, \[eax\]  
seg008:005DB47E                 mov     al, \[edx+2\]                    ; 果然没错检测  
  
ring3 级调试器  
seg008:005DB481                 test    al, al  
seg008:005DB483                 jnz     loc\_5DB521  
seg008:005DB489                 mov     eax, \[ebp+var\_CodeBase\]  
seg008:005DB48F                 add     eax, 0BDh  
seg008:005DB494                 mov     edi, \[eax\]  
seg008:005DB496                 mov     ax, 15FFh  
seg008:005DB49A                 ror     edi, 2  
seg008:005DB49D                 add     edi, 10h  
seg008:005DB4A0                 mov     ecx, 100h  
seg008:005DB4A5                 jmp     short loc\_5DB51D  
seg008:005DB4A7 ; ------------------------------------------------------------------  
  
\----------  
seg008:005DB4A7  
seg008:005DB4A7 loc\_5DB4A7:                                            ; CODE XREF:   
  
DeCode+2E1j  
seg008:005DB4A7                 repne scasb  
seg008:005DB4A9                 cmp     \[edi\], ah  
seg008:005DB4AB                 jnz     short loc\_5DB51D  
seg008:005DB4AD                 mov     edx, \[edi+1\]  
seg008:005DB4B0                 mov     edx, \[edx\]  
seg008:005DB4B2                 cmp     edx, \[ebp+\_ZwQueryInfoProc\]  
seg008:005DB4B5                 jnz     short loc\_5DB51D  
seg008:005DB4B7                 cmp     byte ptr \[edi+7\], 0Fh  
seg008:005DB4BB                 jnz     short loc\_5DB4C8  
seg008:005DB4BD                 cmp     byte ptr \[edi+0Dh\], 39h  
seg008:005DB4C1                 jnz     short loc\_5DB4C8  
seg008:005DB4C3                 add     edi, 0Eh  
seg008:005DB4C6                 jmp     short loc\_5DB4DD  
seg008:005DB4C8 ; ------------------------------------------------------------------  
  
\----------  
seg008:005DB4C8  
seg008:005DB4C8 loc\_5DB4C8:                                            ; CODE XREF:   
  
DeCode+27Dj  
seg008:005DB4C8                                                        ; DeCode+283j  
seg008:005DB4C8                 cmp     byte ptr \[edi+7\], 7Fh  
seg008:005DB4CC                 jbe     short loc\_5DB4D4  
seg008:005DB4CE  
seg008:005DB4CE loc\_5DB4CE:  
seg008:005DB4CE                 cmp     byte ptr \[edi+7\], 70h  
seg008:005DB4D2                 jb      short loc\_5DB4DD  
seg008:005DB4D4  
seg008:005DB4D4 loc\_5DB4D4:                                            ; CODE XREF:   
  
DeCode+28Ej  
seg008:005DB4D4                 cmp     byte ptr \[edi+9\], 39h  
seg008:005DB4D8                 jnz     short loc\_5DB4DD  
seg008:005DB4DA                 add     edi, 0Ah  
seg008:005DB4DD  
seg008:005DB4DD loc\_5DB4DD:                                            ; CODE XREF:   
  
DeCode+288j  
seg008:005DB4DD                                                        ; DeCode+294j   
  
...  
seg008:005DB4DD                 cmp     byte ptr \[edi\], 7Fh  
seg008:005DB4E0                 jnb     short loc\_5DB4EA  
seg008:005DB4E2                 cmp     byte ptr \[edi+2\], 0Fh  
seg008:005DB4E6                 jnz     short loc\_5DB4EA  
seg008:005DB4E8                 jmp     short loc\_5DB503  
seg008:005DB4EA ; ------------------------------------------------------------------  
  
\----------  
seg008:005DB4EA  
seg008:005DB4EA loc\_5DB4EA:                                            ; CODE XREF:   
  
DeCode+2A2j  
seg008:005DB4EA                                                        ; DeCode+2A8j  
seg008:005DB4EA                 cmp     byte ptr \[edi\], 7Fh  
seg008:005DB4ED                 jbe     short loc\_5DB4F7  
seg008:005DB4EF                 cmp     byte ptr \[edi+5\], 0Fh  
seg008:005DB4F3                 jnz     short loc\_5DB4F7  
seg008:005DB4F5                 jmp     short loc\_5DB503  
seg008:005DB4F7 ; ------------------------------------------------------------------  
  
\----------  
seg008:005DB4F7  
seg008:005DB4F7 loc\_5DB4F7:                                            ; CODE XREF:   
  
DeCode+2AFj  
seg008:005DB4F7                                                        ; DeCode+2B5j  
seg008:005DB4F7                 mov     eax, \[ebp+var\_CodeBase\]  
seg008:005DB4FD                 mov     \[eax+3000h\], edi  
seg008:005DB503  
seg008:005DB503 loc\_5DB503:                                            ; CODE XREF:   
  
DeCode+2AAj  
seg008:005DB503                                                        ; DeCode+2B7j  
seg008:005DB503                 call    GetCodeBase  
seg008:005DB508                 add     eax, 6EEh  
seg008:005DB50D                 mov     edx, \[ebp+arg\_PeHeaderCValue\]  ; EDX 当 Key 来进  
  
行解压运算  
seg008:005DB510                 mov     ecx, 2D4h                      ; 解压大小  
seg008:005DB515                 push    eax                            ; 解压的起始地  
  
址 005D96EE  
seg008:005DB516                 call    UnPackCode\_5D96EE              ; 解压代码  
seg008:005DB51B                 jmp     short loc\_5DB521  
seg008:005DB51D ; ------------------------------------------------------------------  
  
\----------  
seg008:005DB51D  
seg008:005DB51D loc\_5DB51D:                                            ; CODE XREF:   
  
DeCode+267j  
seg008:005DB51D                                                        ; DeCode+26Dj   
  
...  
seg008:005DB51D                 or      eax, eax  
seg008:005DB51F                 jnz     short loc\_5DB4A7  
seg008:005DB521  
seg008:005DB521 loc\_5DB521:                                            ; CODE XREF:   
  
DeCode+245j  
seg008:005DB521                                                        ; DeCode+2DDj  
seg008:005DB521                 pop     edi  
seg008:005DB522                 pop     esi  
seg008:005DB523  
seg008:005DB523 loc\_5DB523:                                            ; CODE XREF:   
  
DeCode+466j  
seg008:005DB523                 add     esp, 0FFFFF800h  
seg008:005DB529                 mov     ebx, 2000h  
seg008:005DB52E                 xor     eax, eax  
seg008:005DB530                 mov     \[esp+1978h+var\_1978\], eax  
seg008:005DB533  
seg008:005DB533 loc\_5DB533:                                            ; CODE XREF:   
  
DeCode+34Bj  
seg008:005DB533                 push    ebx  
seg008:005DB534                 push    0  
seg008:005DB536                 call    \[ebp+var\_GlobalAlloc\]  
seg008:005DB53C                 mov     \[ebp+var\_hMem\], eax  
seg008:005DB53F                 mov     \[esp+1980h+var\_hMem1\], eax  
seg008:005DB543                 cmp     \[esp+1980h+var\_hMem1\], 0       ; 判断申请空间  
  
是否成功  
seg008:005DB548                 jz      short loc\_5DB58B  
seg008:005DB54A                 lea     edx, \[esp+1980h+var\_1974\]  
seg008:005DB54E                 push    edx  
seg008:005DB54F                 push    ebx  
seg008:005DB550                 mov     ecx, \[esp+1988h+var\_hMem1\]  
seg008:005DB554                 push    ecx  
seg008:005DB555                 push    10h  
seg008:005DB557                 call    \[ebp+\_ZwQuerysysInfo\]  
seg008:005DB55A                 test    eax, eax  
seg008:005DB55C                 jz      short loc\_5DB58B  
seg008:005DB55E                 mov     eax, \[esp+1990h+var\_1980\]  
seg008:005DB562                 push    eax  
seg008:005DB563                 call    \[ebp+var\_GlobalFree\]  
seg008:005DB569                 xor     edx, edx  
seg008:005DB56B                 mov     \[esp+1994h+var\_1984\], edx  
seg008:005DB56F                 cmp     \[esp+1994h+var\_1988\], 0  
seg008:005DB574                 jnz     short loc\_5DB57E  
seg008:005DB576                 mov     ecx, ebx  
seg008:005DB578                 add     ecx, ecx  
seg008:005DB57A                 mov     ebx, ecx  
seg008:005DB57C                 jmp     short loc\_5DB582  
seg008:005DB57E ; ------------------------------------------------------------------  
  
\----------  
seg008:005DB57E  
seg008:005DB57E loc\_5DB57E:                                            ; CODE XREF:   
  
DeCode+336j  
seg008:005DB57E                 mov     ebx, \[esp+1994h+var\_1988\]  
seg008:005DB582  
seg008:005DB582 loc\_5DB582:                                            ; CODE XREF:   
  
DeCode+33Ej  
seg008:005DB582                 inc     \[esp+1994h+var\_1994\]  
seg008:005DB585                 cmp     \[esp+1994h+var\_1994\], 0Ah  
seg008:005DB589                 jl      short loc\_5DB533  
seg008:005DB58B  
seg008:005DB58B loc\_5DB58B:                                            ; CODE XREF:   
  
DeCode+30Aj  
seg008:005DB58B                                                        ; DeCode+31Ej  
seg008:005DB58B                 cmp     \[esp+1994h+var\_1984\], 0  
seg008:005DB590                 jnz     short loc\_5DB599  
seg008:005DB592                 xor     eax, eax  
seg008:005DB594                 jmp     loc\_5DB692  
seg008:005DB599 ; ------------------------------------------------------------------  
  
\----------  
seg008:005DB599  
seg008:005DB599 loc\_5DB599:                                            ; CODE XREF:   
  
DeCode+352j  
seg008:005DB599                 mov     eax, \[ebp+var\_hMem\]  
seg008:005DB59C                 push    dword ptr \[eax\]  
seg008:005DB59E                 pop     \[ebp+var\_1C\]  
seg008:005DB5A1                 add     eax, 4  
seg008:005DB5A4                 mov     \[ebp+var\_24\], eax  
seg008:005DB5A7                 mov     esi, \[eax\]  
seg008:005DB5A9                 mov     esi, eax  
seg008:005DB5AB                 xor     eax, eax  
seg008:005DB5AD                 mov     \[ebp+var\_20\], eax  
seg008:005DB5B0                 mov     eax, \[ebp+var\_OpenProcess\]  
seg008:005DB5B6                 rol     eax, 2  
seg008:005DB5B9                 mov     \[ebp+var\_1148\], eax  
seg008:005DB5BF  
seg008:005DB5BF loc\_5DB5BF:                                            ; CODE XREF:   
  
DeCode+445j  
seg008:005DB5BF                 movzx   eax, word ptr \[esi+4\]  
seg008:005DB5C3                 cmp     eax, 5  
seg008:005DB5C6                 jnz     loc\_5DB679  
seg008:005DB5CC                 push    dword ptr \[esi\]  
seg008:005DB5CE                 push    0  
seg008:005DB5D0                 push    1F0FFFh  
seg008:005DB5D5                 call    \[ebp+var\_OpenProcess\]  
seg008:005DB5DB                 or      eax, eax  
seg008:005DB5DD                 jz      loc\_5DB679  
seg008:005DB5E3                 mov     \[ebp+var\_1140\], eax  
seg008:005DB5E9                 xor     eax, eax  
seg008:005DB5EB                 mov     \[ebp+var\_28\], eax  
seg008:005DB5EE                 push    2  
seg008:005DB5F0                 push    eax  
seg008:005DB5F1                 push    eax  
seg008:005DB5F2                 lea     ecx, \[ebp+var\_28\]  
seg008:005DB5F5                 push    ecx  
seg008:005DB5F6                 push    0FFFFFFFFh  
seg008:005DB5F8                 movzx   edx, word ptr \[esi+6\]  
seg008:005DB5FC                 mov     \[ebp+var\_112C\], edx  
seg008:005DB602                 push    edx  
seg008:005DB603                 push    \[ebp+var\_1140\]  
seg008:005DB609                 call    \[ebp+\_ZwDupObject\]  
seg008:005DB60C                 cmp     \[ebp+var\_28\], 0  
seg008:005DB610                 jz      short loc\_5DB66D  
seg008:005DB612                 lea     eax, \[ebp+var\_1154\]  
seg008:005DB618                 push    eax  
seg008:005DB619                 push    18h  
seg008:005DB61B                 lea     eax, \[ebp+var\_1184\]  
seg008:005DB621                 push    eax  
seg008:005DB622                 push    0  
seg008:005DB624                 push    \[ebp+var\_28\]  
seg008:005DB627                 call    \[ebp+\_ZwQueryInfoProc\]  
seg008:005DB62A                 test    eax, eax  
seg008:005DB62C                 or      eax, eax  
seg008:005DB62E                 jnz     short loc\_5DB664  
seg008:005DB630                 mov     eax, \[ebp+var\_1174\]  
seg008:005DB636                 cmp     eax, \[ebp+arg\_PID\]  
seg008:005DB639                 jnz     short loc\_5DB664  
seg008:005DB63B                 push    dword ptr \[esi\]  
seg008:005DB63D                 pop     \[ebp+var\_1150\]  
seg008:005DB643                 call    GetCodeBase  
seg008:005DB648                 push    eax  
seg008:005DB649                 add     eax, 0C1h  
seg008:005DB64E                 push    \[ebp+var\_1150\]  
seg008:005DB654                 pop     dword ptr \[eax\]  
seg008:005DB656                 pop     eax  
seg008:005DB657                 add     eax, 82h  
seg008:005DB65C                 push    \[ebp+var\_1148\]  
seg008:005DB662                 pop     dword ptr \[eax\]  
seg008:005DB664  
seg008:005DB664 loc\_5DB664:                                            ; CODE XREF:   
  
DeCode+3F0j  
seg008:005DB664                                                        ; DeCode+3FBj  
seg008:005DB664                 push    \[ebp+var\_28\]  
seg008:005DB667                 call    \[ebp+var\_CloseHandle\]  
seg008:005DB66D  
seg008:005DB66D loc\_5DB66D:                                            ; CODE XREF:   
  
DeCode+3D2j  
seg008:005DB66D                 push    \[ebp+var\_1140\]  
seg008:005DB673                 call    \[ebp+var\_CloseHandle\]  
seg008:005DB679  
seg008:005DB679 loc\_5DB679:                                            ; CODE XREF:   
  
DeCode+388j  
seg008:005DB679                                                        ; DeCode+39Fj  
seg008:005DB679                 add     esi, 10h  
seg008:005DB67C                 dec     \[ebp+var\_1C\]  
seg008:005DB67F                 cmp     \[ebp+var\_1C\], 0  
seg008:005DB683                 jnz     loc\_5DB5BF  
seg008:005DB689                 push    \[ebp+var\_hMem\]  
seg008:005DB68C                 call    \[ebp+var\_GlobalFree\]  
seg008:005DB692  
seg008:005DB692 loc\_5DB692:                                            ; CODE XREF:   
  
DeCode+356j  
seg008:005DB692                 add     esp, 800h  
seg008:005DB698  
seg008:005DB698 Over:                                                  ; CODE XREF:   
  
DeCode+EDj  
seg008:005DB698                                                        ; DeCode+11Dj   
  
...  
seg008:005DB698                 mov     eax, \[ebp+var\_1150\]  
seg008:005DB69E                 cmp     eax, \[ebp+var\_1148\]  
seg008:005DB6A4                 jz      loc\_5DB523  
seg008:005DB6AA                 xor     ecx, ecx  
seg008:005DB6AC                 pop     ebx  
seg008:005DB6AD                 pop     edi  
seg008:005DB6AE                 pop     esi  
seg008:005DB6AF                 leave  
seg008:005DB6B0                 retn    8  
seg008:005DB6B0 DeCode          endp  
seg008:005DB6B0 ; ------------------------------------------------------------------  
  
\----------  
.........  
  
sag008:005D90B2 ; \*\*\*\*\*\*\*\*\*\*\*\*\*\* S U B R O U T I N E   
  
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
seg008:005D90B2  
seg008:005D90B2  
seg008:005D90B2 GetCodeBase     proc near                              ; CODE XREF:   
  
GETAPIADDR+Bp  
seg008:005D90B2                                                        ;   
  
seg008:005D9DEFp ...  
seg008:005D90B2                 call    $+5  
seg008:005D90B7                 pop     eax  
seg008:005D90B8                 and     ax, 0F000h  
seg008:005D90BC                 retn  
seg008:005D90BC GetCodeBase     endp  
seg008:005D90BC  
seg008:005D90BC ; ------------------------------------------------------------------  
  
\----------  
  
  
  
\-----------------------------------------未完，待继!^\_^-----------------------------  
  
\-----------------  
  
BTW:做成 ubb 的就不知道如何设置像 html 里的:  
<a name="label"\>label name</a>  
在 link 处 <a href="label"\>text</a>  
  
知道的朋友说说:-p  
  
Greetz:  
 Fly.Jingulong,yock,tDasm.David.hexer,hmimys,ahao.UFO(brother).alan(sister).all of my   
  
friends and you!  
  
          By loveboom\[DFCG\]\[FCG\]\[US\]  
                      http://blog.csdn.net/bmd2chen  
                  
  
                                        Email:loveboom#163.com 

 4dd466d0e1e0cc123afbe504da2462ca