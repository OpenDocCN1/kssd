
```asm
;*****************************************************************************  
              inc      dwCountBP   
               .if    dwCountBP==1   
                   invoke   ReadProcessMemory, pi.hProcess, dwOrgOEP, addr oldbyte, 1, 0   ;在 dwOrgOEP 中读出一个字节   
                   invoke   WriteProcessMemory, pi.hProcess, dwOrgOEP, addr int3, 1, 0     ;写入 INT3 断点                  

;*****************************************************************************  

; 第二次中断，中断在起先设置的原始入口点，恢复代码，在机器码处设置硬件断点  

;*****************************************************************************   

               .elseif dwCountBP==2                      

                   mov  context.ContextFlags, CONTEXT_CONTROL

                   invoke   GetThreadContext, pi.hThread, addr context 

                   dec  context.regEip 

                   invoke   WriteProcessMemory, pi.hProcess, dwOrgOEP, addr oldbyte, 1, 0  ;恢复入口代码

                   invoke   SetThreadContext, pi.hThread, addr context 

                   mov     context.ContextFlags, CONTEXT_DEBUG_REGISTERS 

                   invoke   GetThreadContext, pi.hThread, addr context 

                   mov    context.iDr0, BREAK_POINT1                                     ;设置硬件断点       

                   mov    context.iDr7, LOCAL_EXACT_BPM_ENABLED + DR0_ENABLED 

                   invoke   SetThreadContext, pi.hThread, addr context                      

               .endif 

                   invoke   ContinueDebugEvent, DBEvent.dwProcessId, DBEvent.dwThreadId, DBG_CONTINUE 

                   .continue        

     .elseif DBEvent.u.Exception.pExceptionRecord.ExceptionCode==EXCEPTION_SINGLE_STEP    ;单步运行模式     

;*****************************************************************************  

; 第三次中断，中断在机器码处，清除断点，在注册码处设置硬件断点  

;*****************************************************************************   

          inc     dwCountSS           

     .IF dwCountSS == 1 

                   mov     context.ContextFlags, CONTEXT_FULL 

                   invoke   GetThreadContext, pi.hThread, addr context 

                   mov eax,context.regEax                                            ;机器码在 EAX 中 

                   invoke   WriteProcessMemory,pi.hProcess,eax,addr UserID,sizeof UserID,NULL   ;写入机器码

                   invoke   SetThreadContext, pi.hThread, addr context 

                  ; invoke ReadProcessMemory,pi.hProcess,context.regEax,addr buffer,0ch,NULL    ;用于检测修改的机器码是否已经生效 

                  ; invoke   MessageBox,0, addr buffer,CTXT("输入的新机器码为："), MB_OK+MB_ICONINFORMATION   

                   mov     context.ContextFlags, CONTEXT_DEBUG_REGISTERS 

                   invoke   GetThreadContext, pi.hThread, addr context 

                   mov     context.iDr0, BREAK_POINT2 

                   mov     context.iDr7, LOCAL_EXACT_BPM_ENABLED + DR0_ENABLED 

                   invoke   SetThreadContext, pi.hThread, addr context   

                   invoke   ContinueDebugEvent, DBEvent.dwProcessId, DBEvent.dwThreadId, DBG_CONTINUE 

                   .continue          

;*****************************************************************************  

; 第四次中断，中断在注册码处，读出注册码，清除硬件断点，恢复线程  

;*****************************************************************************        

          .elseif dwCountSS == 2 

                   mov     context.ContextFlags, CONTEXT_FULL 

                   invoke   GetThreadContext, pi.hThread, addr context 

                   mov context.ContextFlags, CONTEXT_FULL 

                   mov EAX,context.regEax                             ;暗码在 EAX 中 

                   add EAX,0A98AC7h 

                   xor EAX,4ECAEDDh 

                   sub EAX,0FFF379A6h  

                invoke wsprintf,addr value,addr szFormat,eax    

                invoke SetDlgItemText,hDlg,IDC_CODE,addr value                   

                   mov     context.iDr0, 0 

                   mov     context.iDr7, 0   

                   invoke   SetThreadContext,pi.hThread, addr context     

                   invoke TerminateProcess,pi.hProcess,uExitCode               ;强行退出   

                   .break              

          .endif 

          invoke   ContinueDebugEvent, DBEvent.dwProcessId,DBEvent.dwThreadId, DBG_CONTINUE 

     .endif 

.endif 

   invoke   ContinueDebugEvent, DBEvent.dwProcessId,DBEvent.dwThreadId, DBG_EXCEPTION_NOT_HANDLED 

.endw 

;*****************************************************************************  

; 结束线程  

;*****************************************************************************   

       invoke   CloseHandle, pi.hThread 

       invoke   CloseHandle, pi.hProcess 

   popad 

   ret   

GetKey endp 

end start 
```

附件下载 `4dd466d0e1e0cc123afbe504da2462ca`
