
```asm
seg008:005DAA73                 sub     ecx, edx  

seg008:005DAA75                 lea     edi, [edx+edi+20h]  

seg008:005DAA79                 mov     edx, eax  

seg008:005DAA7B                 sub     eax, [ebp+var_App_hModule]  

seg008:005DAA81                 and     ax, 0F000h  

seg008:005DAA85                 push    [ebp+var_hLocal]  

seg008:005DAA8B                 push    [ebp+var_9B4]  

seg008:005DAA91                 call    Getjmpaddress                 ; 父进程异常后  

，写入正确的内容  

seg008:005DAA91                                                 ; 也就是动态写  

入跳转表  

seg008:005DAA96                 test    eax, eax                   ; 操作成功  

后 eax 返回跳转地址  

seg008:005DAA98                 jz      short loc_5DAAE7  

seg008:005DAA9A                 dec     edx  

seg008:005DAA9B                 push    edx  

seg008:005DAA9C                 mov     [ebp+var_ExcepAddress], eax  

seg008:005DAAA2                 push    eax  

seg008:005DAAA3                 push    [ebp+var_hProcess]  

seg008:005DAAA9                 push    [ebp+var_918]  

seg008:005DAAAF                 mov     al, 0E9h  

seg008:005DAAB1                 call    WritePresentMem              ; 写入代码，改  

写成 jmp address  

seg008:005DAAB1                                                 ; 其中 address  

就是前面 eax 的值  

seg008:005DAAB6                 lea     eax, [ebp+var_pContext]  

seg008:005DAABC                 push    eax  

seg008:005DAABD                 push    [ebp+var_hThread]  

seg008:005DAAC3                 call    [ebp+var_SetThreadContext]  

seg008:005DAAC9  

seg008:005DAAC9 loc_5DAAC9:                                           ; CODE XREF:  

Decrypt_Code+874j  

seg008:005DAAC9                                                 ;  

Decrypt_Code+A04j  

seg008:005DAAC9                 push    10002h  

seg008:005DAACE                 push    [ebp+var_ThreadId]  

seg008:005DAAD4                 push    [ebp+var_ProcessId]  

seg008:005DAADA                 call    [ebp+var_ContinueDebugEvent]  

seg008:005DAAE0                 pop     esi  

seg008:005DAAE1                 pop     edi  

seg008:005DAAE2                 jmp     loc_5DA864  

seg008:005DAAE7 ; ------------------------------------------------------------------  

----------  

seg008:005DAAE7  

seg008:005DAAE7 loc_5DAAE7:                                           ; CODE XREF:  

Decrypt_Code+8DEj  

seg008:005DAAE7                                                 ;  

Decrypt_Code+9D4j  

seg008:005DAAE7                 jmp     loc_5DAE36                 ;  

ContinueStatus  

seg008:005DAAEC ; ------------------------------------------------------------------  

----------  

seg008:005DAAEC  

seg008:005DAAEC loc_5DAAEC:                                           ; CODE XREF:  

Decrypt_Code+7DCj  

seg008:005DAAEC                 cmp     [ebp+var_9CC], 0  

seg008:005DAAF3                 jnz     short loc_5DAB1C  

seg008:005DAAF5  

seg008:005DAAF5 loc_5DAAF5:                                           ; CODE XREF:  

Decrypt_Code+A0Bj  

seg008:005DAAF5                 inc     [ebp+var_9CC]  

seg008:005DAAFB                 push    10002h  

seg008:005DAB00                 push    [ebp+var_ThreadId]  

seg008:005DAB06                 push    [ebp+var_ProcessId]  

seg008:005DAB0C                 call    [ebp+var_ContinueDebugEvent]  

seg008:005DAB12                 jmp     loc_5DA864  

seg008:005DAB17 ; ------------------------------------------------------------------  

----------  

seg008:005DAB17                 jmp     loc_5DABCA  

seg008:005DAB1C ; ------------------------------------------------------------------  

----------  

seg008:005DAB1C  

seg008:005DAB1C loc_5DAB1C:                                           ; CODE XREF:  

Decrypt_Code+939j  

seg008:005DAB1C                 push    edi  

seg008:005DAB1D                 push    esi  

seg008:005DAB1E                 mov     edi, [ebp+var_hMemLocal]  

seg008:005DAB24                 mov     ecx, [edi]  

seg008:005DAB26                 add     edi, 4  

seg008:005DAB29                 shl     ecx, 1  

seg008:005DAB2B                 mov     eax, [ebp+var_ThreadId]  

seg008:005DAB31                 repne scasd  

seg008:005DAB33                 jnz     loc_5DABC3  

seg008:005DAB39                 mov     esi, [edi]  

seg008:005DAB3B                 mov     [ebp+var_pContext], 10007h  

seg008:005DAB45                 lea     eax, [ebp+var_pContext]  

seg008:005DAB4B                 push    eax  

seg008:005DAB4C                 push    esi  

seg008:005DAB4D                 call    [ebp+var_GetThreadContext]  

seg008:005DAB53                 test    eax, eax  

seg008:005DAB55                 jz      short loc_5DABC3  

seg008:005DAB57                 mov     eax, [ebp+var_ExcepAddress]  

seg008:005DAB5D                 mov     edi, [ebp+var_9B4]  

seg008:005DAB63                 mov     edx, [edi+10h]  

seg008:005DAB66                 mov     ecx, [edi+14h]  

seg008:005DAB69                 sub     ecx, edx  

seg008:005DAB6B                 lea     edi, [edx+edi+20h]  

seg008:005DAB6F                 mov     edx, eax  

seg008:005DAB71                 sub     eax, [ebp+var_App_hModule]  

seg008:005DAB77                 and     ax, 0F000h  

seg008:005DAB7B                 push    [ebp+var_hLocal]  

seg008:005DAB81                 push    [ebp+var_9B4]  

seg008:005DAB87                 call    Getjmpaddress  

seg008:005DAB8C                 test    eax, eax  

seg008:005DAB8E                 jz      loc_5DAAE7  

seg008:005DAB94                 dec     edx  

seg008:005DAB95                 push    edx  

seg008:005DAB96                 mov     [ebp+var_ExcepAddress], eax  

seg008:005DAB9C                 push    eax  

seg008:005DAB9D                 push    [ebp+var_hProcess]  

seg008:005DABA3                 push    [ebp+var_918]  

seg008:005DABA9                 mov     al, 0E9h  

seg008:005DABAB                 call    WritePresentMem  

seg008:005DABB0                 lea     eax, [ebp+var_pContext]  

seg008:005DABB6                 push    eax  

seg008:005DABB7                 push    esi  

seg008:005DABB8                 call    [ebp+var_SetThreadContext]  

seg008:005DABBE                 jmp     loc_5DAAC9  

seg008:005DABC3 ; ------------------------------------------------------------------  

----------  

seg008:005DABC3  

seg008:005DABC3 loc_5DABC3:                                           ; CODE XREF:  

Decrypt_Code+979j  

seg008:005DABC3                                                 ;  

Decrypt_Code+99Bj  

seg008:005DABC3                 pop     esi  

seg008:005DABC4                 pop     edi  

seg008:005DABC5                 jmp     loc_5DAAF5  

seg008:005DABCA ; ------------------------------------------------------------------  

----------  

seg008:005DABCA  

seg008:005DABCA loc_5DABCA:                                           ; CODE XREF:  

Decrypt_Code+95Dj  

seg008:005DABCA                 jmp     loc_5DAE36                 ;  

ContinueStatus  

seg008:005DABCF ; ------------------------------------------------------------------  

----------  

seg008:005DABCF  

seg008:005DABCF loc_5DABCF:                                           ; CODE XREF:  

Decrypt_Code+7CAj  

seg008:005DABCF                 cmp     [ebp+var_Exception_Code], 0C0000005h ;  

ACCESS_VIOLATION  

seg008:005DABD9                 jnz     loc_5DAE36                 ;  

ContinueStatus  

seg008:005DABDF                 push    edi  

seg008:005DABE0                 mov     edi, [ebp+var_hMemLocal]  

seg008:005DABE6                 mov     ecx, [edi]  

seg008:005DABE8                 mov     eax, [ebp+var_ThreadId]  
```
