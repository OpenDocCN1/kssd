
```
        PUSH EDI  
        PUSH ESI  
        LEA EDI, [EBP+Buff]  
        ASSUME EDI : PTR sReThunkInfo  
        MOV ESI, [EDI].pNextStub  
        MOV [EDX], ESI        ; make the thunk point to stub mem  
        SUB EAX, ESI  
        SUB EAX, 5            ; eax - esi -5 得到 JMP xxxxx 后面的偏移  
        MOV BYTE PTR [ESI], 0E9h        ;写入 E9--JMP  
        MOV DWORD PTR [ESI+1], EAX    ;将函数地址写进去  
        ADD [EDI].pNextStub, SIZEOF sApiStub        ;指向一个 sApiStub  
        ASSUME EDI : NOTHING  
        POP ESI  
        POP EDI  
    SkipThunkRed:  
       .ENDIF  
    add ecx,4            ;下一个 IMAGE_THUNK_DATA  
    add edx,4            ;下一个 IMAGE-IMPORT_BY_NAME  
   .ENDW      
   add esi,SIZEOF sItInfo                    ; ESI = 下一个 IID  
.ENDW  
assume esi:nothing  
xor eax,eax  
inc eax  
SkipInitIt:  
    .IF eax != TRUE  
       ; exit  
       popad  
       ret  
    .ENDIF  
     ;----- ERASE PE HEADER ------  
     test [ebp+PROTECTION_FLAGS],ERASE_HEADER_FLAG  
     jz SkipEraseHeader
;下面是销毁 PE 头。好猥琐!  
   mov edi,[ebp+dwImageBase]  
   add edi,[edi+03Ch]                                                     ;PE 头  
   assume edi : ptr IMAGE_NT_HEADERS      
   mov ecx,[edi].OptionalHeader.SizeOfHeaders        ;大小  
   mov esi,[ebp+dwImageBase]                                ;ESI= 镜像基址  
   assume edi : nothing  
  ZeroMemLoop:                  ;用 0 填充掉  
     mov byte ptr [esi],0  
     inc esi  
     loop ZeroMemLoop  
  SkipEraseHeader:  
     ;------ CHECK AGAIN LOADER CRC & COMPARE ------  
     LEA EAX, [EBP+DepackerCode]  
     MOV ECX, LOADER_CRC_CHECK_SIZE  
     JMP SM10  
     DB 0E9h                ;花指令  
     SM10:  
     CALL GetChecksum        ;校验和  
     JMP SM11  
     DB 0C7h                ;花指令  
     SM11:  
     MOV EBX, [EBP+dwLoaderCRC]  
     XOR EAX, EBX            ;检验校验和是否相等  
     .IF !ZERO?             ;不等返回  
       JMP SM12  
       DB 02Ch  
       SM12:  
       POPAD  
       JMP SM13  
       DB 0E8h  
       SM13:  
       RET  
     .ENDIF  
;解密 OEP_JUMP_CODE  
     LEA EDI, [EBP+OFFSET OEP_JUMP_CODE_START]  
     MOV ESI, EDI  
     MOV ECX, CRYPT_OEP_JUMP_SIZE  
   XOR EBX, EBX  
  OepJumpDecryptLoop:        ;下面是解密算法  
     LODSB  
     XOR AL, OEP_JUMP_ENCRYPT_NUM  
     SUB AL, BL  
     ROL AL, 2  
     STOSB  
     INC EBX  
     LOOP OepJumpDecryptLoop  
;跳到 OEP  
OEP_JUMP_CODE_START:  
   ;----- CHECK FOR DEBUG API's -----  
   LEA EAX, [EBP+OFFSET szIsDebuggerPresent]  
   PUSH EAX  
   PUSH [EBP+dwKernelBase]  
   CALL [EBP+_GetProcAddress]        ;获取 IsDebuggerPresent 函数地址  
   OR EAX, EAX        ; 95 系统下不支持这个函数  
   .IF !ZERO?  
     CALL EAX        ;调用 IsDebugPresent 检测调试器  
     OR EAX, EAX  
     .IF !ZERO?      ;存在调试器，直接返回  
       POPAD  
      RET  
     .ENDIF  
   .ENDIF  
   ;------ SECOND SI CHECK ------  
   ; doesn't work on NT  
   ; install SEH frame  
   TEST [EBP+PROTECTION_FLAGS], CHECK_SI_FLAG  
   JZ SkipSICheck2  
   LEA ESI,[EBP+SEH]  
   ASSUME ESI : PTR sSEH  
   LEA EAX, [EBP+OFFSET SICheck2_SP]  
   MOV [ESI].SaveEip, EAX  
     ASSUME ESI : NOTHING  
     XOR EBX, EBX  
   LEA EAX, [EBP+OFFSET SehHandler2]  
   PUSH EAX  
   PUSH FS:[EBX]  
   MOV FS:[EBX], ESP  
   MOV EDI, EBP  
   MOV EAX, 4400h  
   JMP SM4  
   DB 0C7h  
  SM4:  
   INT 68h  
  SICheck2_SP:     
     XOR EBX, EBX  
   POP FS:[EBX]  
   ADD ESP, 4  
   .IF DI == 01297h || DI == 01277h || DI == 01330h  
     JMP SM5  
     DB 0FFh  
      SM5:        
     POPAD  
     JMP SM6  
     DB 0E8h  
     SM6:  
     RET  
   .ENDIF  
  SkipSICheck2:  
  LEA EAX, [EBP+OFFSET OepJumpCodeCont]  
  PUSH EAX  
  RET     
 ;**************************************************************************************************************************************
  ;在这个异常处理会跳到 OEP 去  
   SehHandler_OEP_Jump PROC C pExcept:DWORD,pFrame:DWORD,pContext:DWORD,pDispatch:DWORD  
     PUSH EDI  
     MOV EAX,pContext  
     ASSUME EAX : PTR CONTEXT  
     ; 恢复原来的 SEH 链  
     MOV EDI, [EAX].regEsp  
     PUSH [EDI]  
     XOR EDI, EDI  
     POP FS:[EDI]  
     ;恢复原来的 SEH 链  
     ADD [EAX].regEsp, 8  
     MOV EDI, [EAX].regEbx    ; EDI ＝ 加密的 OEP  
     ROL EDI, 7             ;解密得到真正的 OEP  
     MOV [EAX].regEip, EDI    ;设置 EIP 到 OEP  
     mov EAX,ExceptionContinueExecution        ;继续执行，会跳到 OEP  
     ASSUME EAX : NOTHING  
     POP EDI  
     RET  
   SehHandler_OEP_Jump ENDP  
;**************************************************************************************************************************************
  
  OepJumpCodeCont:  
   ;用 0 填充掉 DepackerCode 到 SehHandler_OEP_Jump 的代码  
   XOR AL,AL  
   LEA EDI, [EBP+OFFSET DepackerCode]  
   MOV ECX, (OFFSET SehHandler_OEP_Jump - OFFSET DepackerCode)  
  LoaderZeroLoop:  
   STOSB  
   LOOP LoaderZeroLoop  
   ;用 0 填充掉从 LOADER_CRYPT_END 到 OEP_JUMP_CODE_END 的代码  
   LEA EDI, [EBP+OFFSET OEP_JUMP_CODE_END]  
   MOV ECX, (OFFSET LOADER_CRYPT_END - OFFSET OEP_JUMP_CODE_END)  
  LoaderVarZeroLoop:  
   STOSB  
   LOOP LoaderVarZeroLoop  
   POPAD    ; RESTORE STARTUP REGS  
     ; After this POPAD:  
     ; EAX - OEP Seh handler  
     ; EBX - OEP (rored)  
    ;安装 SEH,在 SehHandler 中跳到 OEP---    
   PUSH EAX  
   XOR EAX, EAX  
   PUSH FS:[EAX]  
   MOV FS:[EAX], ESP  
   JMP SM3  
   DB 087H        ;花指令  
  SM3:         ; 跳到这里执行，因为下面的代码已经用 0 填充，所以会异常。中到  
OEP_JUMP_CODE_END:  
; EAX = ASCII string address  
KillString:  
   .WHILE byte ptr [eax] != 0  
     mov byte ptr [eax],0  
     inc eax  
   .ENDW  
   ret  
SehHandler1 PROC C pExcept:DWORD,pFrame:DWORD,pContext:DWORD,pDispatch:DWORD  
   PUSH EDI  
   MOV EAX,pContext  
   ASSUME EAX : PTR CONTEXT  
   MOV EDI, [EAX].regEdi         
   push [EDI+SEH.SaveEip]       ;SaveEip 压栈  
   pop [eax].regEip             ;弹出到 regEip。更改 EIP 的地址，程序会转到 SaveEip 去执行  
   MOV [eax].regEbp, EDI        ;还原 EBP  
   MOV [EAX].regEax, 4         ; SI NOT detected !  
   mov EAX,ExceptionContinueExecution  ;返回到 SaveEip 去  
   ASSUME EAX : NOTHING  
   POP EDI  
   RET  
SehHandler1 ENDP  
SehHandler2 PROC C pExcept:DWORD,pFrame:DWORD,pContext:DWORD,pDispatch:DWORD  
   PUSH EDI  
   MOV EAX,pContext  
   ASSUME EAX : PTR CONTEXT  
   MOV EDI, [EAX].regEdi  
   push [EDI+SEH.SaveEip]  
   pop [eax].regEip  
   MOV [eax].regEbp, EDI  
   MOV [EAX].regEdi, 0         ; SI NOT detected !  
   mov EAX,ExceptionContinueExecution  
   ASSUME EAX : NOTHING  
   POP EDI  
   RET     
SehHandler2 ENDP  
;----- LOADER STRUCTS -----  
sItInfo STRUCT  
   DllNameRVA       dd ?  
   FirstThunk       dd ?  
   OrgFirstThunk    dd ?  
sItInfo ENDS  
sSEH STRUCT  
   OrgEsp           dd ?  
   OrgEbp           dd ?  
   SaveEip          dd ?  
sSEH ENDS  
sReThunkInfo STRUCT  
   ApiStubMemAddr   DD ?  
   pNextStub        DD ?  
sReThunkInfo ENDS  
sApiStub STRUCT                  ; UNUSED !  
   JumpOpc          DB ?  
   JumpAddr         DD ?  
sApiStub ENDS  
;----- LOADER VARIABLES -----  
dwImageBase         dd 0  
```
