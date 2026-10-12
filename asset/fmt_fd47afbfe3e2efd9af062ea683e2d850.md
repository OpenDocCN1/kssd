
```asm
SecDecryptBuff DB SEC_PER_SIZE DUP (0) ; Make PER 函数生成的解密代码放在这里，进行解密
STOSB ; 解密后的代码重新写回去
LOOP SecDecryptLoop
RET

SecDecryptContinue1:
    popad
@@LoopEnd:
    add esi, SIZEOF IMAGE_SECTION_HEADER ; ESI 指向下一个节
    INC EDX
.UNTIL dx == [edi].FileHeader.NumberOfSections ; 循环
assume esi : nothing
assume edi : nothing
ret
```

```text
********************************************************************************
```

```asm
AfterDeCryptionContinue:
    ;------ PREPARE THE OEP JUMP EXCEPTION :) ------
    MOV  EBX, [EBP+dwImageBase]
    ADD  EBX, [EBP+dwOrgEntryPoint]
    ROR  EBX, 7
    MOV  [ESP+010h], EBX
    LEA  EBX, [EBP+OFFSET SehHandler_OEP_Jump]
    MOV  [ESP+01Ch], EBX
    ;

; 下面代码将 TLS 索引清 0
********************************************************************************
```

```asm
    mov edi,[ebp+dwImageBase]
    add edi,dword ptr [edi+03Ch] ; EDI = PE 头
    assume edi : ptr IMAGE_NT_HEADERS
    mov ebx,[edi].OptionalHeader.DataDirectory[SIZEOF IMAGE_DATA_DIRECTORY * 9].VirtualAddress ; Tls TABLE
    assume edi : nothing
    cmp ebx,0 ; 存在 TLS?
    jz SkipTlsFix
    add ebx,[ebp+dwImageBase] ; ebx -> pointer to tls table
    assume ebx : ptr IMAGE_TLS_DIRECTORY32
    mov eax,[ebx].AddressOfIndex
    mov dword ptr [eax],0 ; TLS 索引置 0
    assume ebx : nothing

********************************************************************************
```

```asm
SkipTlsFix:
    ; 校验和是否相等
    mov eax,[ebp+dwCalcedCRC]
    .IF eax != 0
        .IF eax != [ebp+dwOrgChecksum]
            jmp SkipInitIt ; 不相等就跳过初始化输入表
        .ENDIF
    .ENDIF
    ;----- INIT IMPORT TABLE -----
    ; 0 - 失败
    ; 1 - 成功
    LEA ESI, [EBP+OFFSET IIDInfo] ; ESI -> pointer to the current IID
    ASSUME ESI : PTR sItInfo
    ;------ PREPARE API REDIRECTION ------
    TEST [EBP+PROTECTION_FLAGS], API_REDIRECT_FLAG
    .IF !ZERO?
        PUSH ESI
        LEA  EDI, [EBP+OFFSET Buff]
        ASSUME EDI : PTR sReThunkInfo
        XOR  ECX, ECX

; 下面的循环，获取 API 函数的个数
        .WHILE [ESI].FirstThunk
            MOV  EDX, [ESI].FirstThunk
            ADD  EDX, [EBP+dwImageBase]
            .WHILE DWORD PTR [EDX] ; 获取一个 IID 的函数的个数
                INC  ECX
                ADD  EDX, 4
            .ENDW
            ADD  ESI, SIZEOF sItInfo ; 指向下一个 IID
        .ENDW

; 申请内存
        XOR  EDX, EDX
        MOV  EAX, SIZEOF sApiStub
        MUL  ECX
        PUSH EAX
        PUSH GMEM_FIXED
        CALL [EBP+_GlobalAlloc]
        .IF !EAX ; fatal exit
            ADD  ESP, 4
            POPAD
            RET
        .ENDIF
        MOV  [EDI].ApiStubMemAddr, EAX ; EAX = 申请到的内存地址
        MOV  [EDI].pNextStub, EAX
        ASSUME EDI : NOTHING
        POP  ESI
    .ENDIF

    ; start with the real routine
    ; 解密 DLL 名字
    .WHILE [esi].FirstThunk != 0
        mov ebx,[esi].DllNameRVA
        add ebx,[ebp+dwImageBase]
        MOV EAX,EBX
        CALL EnDeCryptString
        LEA EAX, [EBP+InitITContinue1] ; goto InitITContinue1
        PUSH EAX
        RET ; JMP InitITContinue1, 加载 DLL

********************************************************************************
```

```asm
; 参数; eax = 要解密的字符串的 VA
EnDeCryptString:
        PUSH ESI
        PUSH EDI
        MOV ESI,EAX
        MOV EDI,EAX
    DllCryptLoop:
        LODSB
        ROR AL,4
        STOSB
        CMP BYTE PTR [EDI],0
        JNZ DllCryptLoop
        POP EDI
        POP ESI
        RET

********************************************************************************
```

```asm
InitITContinue1:
       push ebx
       call [ebp+_LoadLibrary] ; 加载 DLL
       test eax,eax
       jz SkipInitIt

; 下面要销毁 DLL 名字
       PUSH EAX
       test [ebp+PROTECTION_FLAGS],DESTROY_IMPORT_FLAG
       jz DontKillDllName
       LEA EAX, [EBP+OFFSET DontKillDllName]
       PUSH EAX ; KillString 的返回地址
       MOV EAX, EBX
       JMP KillString ; 销毁 DLL 名字

   DontKillDllName:
          POP EBX ; EBX = DLL 的句柄
       ; process the (Original-)FirstThunk members
       mov ecx,[esi].OrgFirstThunk
       .IF ecx == 0
          mov ecx,[esi].FirstThunk
       .ENDIF
       add ecx,[ebp+dwImageBase] ; ECX 指向 IMAGE_THUNK_DATA
       mov edx,[esi].FirstThunk
       add edx,[ebp+dwImageBase] ; edx - IAT，IMAGE_IMPORT_BY_NAME
       .WHILE dword ptr [ecx] != 0
          test dword ptr [ecx],IMAGE_ORDINAL_FLAG32 ; 判断是否由序号引出
          jnz @@OrdinalImp
          ; process a name import
          mov dword ptr eax,[ecx] ; EAX = IMAGE_IMPORT_BY_NAME
          add eax,2 ; 跳过 Hint
          add eax,[ebp+dwImageBase] ; EAX = API NAME
          PUSH EAX
          CALL EnDeCryptString ; 解密 APINAME
          POP  EAX
          mov edi,eax ; 保存 API 名字指针，后面会用来销毁 API 名字
          push edx
          push ecx ; 保存 IMAGE_THUNK_DATA 指针
          push eax
          push ebx
          call [ebp+_GetProcAddress] ; 获取函数地址
          .IF eax == NULL
             pop ecx
             pop edx
             jmp SkipInitIt
          .ENDIF
          pop ecx
          pop edx
          ; 销毁 API 名字，这段很像 Acprotect 中销毁字符串的代码
            PUSHAD
          test [ebp+PROTECTION_FLAGS],DESTROY_IMPORT_FLAG
            JZ  DontKillApiName
            LEA EAX, [EBP+OFFSET DontKillApiName] ; 保存返回地址
            PUSH EAX
          MOV EAX, EDI
          JMP KillString ; 销毁

   DontKillApiName:
          POPAD
          ;-> paste API address
          mov dword ptr [edx],eax ; API 地址到填充 IAT
          jmp @@NextThunkPlease

   @@OrdinalImp:
            ; process an ordinal import
            push edx
            push ecx ; save the thunk pointers
            mov dword ptr eax,[ecx] ; 取得函数序号
            sub eax,080000000h ; 去掉高位
            push eax
            push ebx
            call [ebp+_GetProcAddress] ; 获取 API 函数地址
            test eax,eax
            jz SkipInitIt
            pop ecx
            pop edx
            mov dword ptr [edx],eax ; 写到 IAT

   ; 下面是 IAT 重定向的处理
   @@NextThunkPlease:
            ; eax = Current Api address
            ; ebx = dll base
            ; edx = non-org thunk pointer
            TEST [EBP+PROTECTION_FLAGS], API_REDIRECT_FLAG
            .IF !ZERO?
                .IF [EBP+bNT] ; 如果是 NT 下的非系统 DLL 则跳过
                    .IF EBX < 070000000h || EBX > 077FFFFFFh
                        JMP SkipThunkRed
                    .ENDIF
                .ELSE
                    .IF EBX < 080000000h ; 98 下的非系统 DLL 同样跳过
                        JMP SkipThunkRed
                    .ENDIF
                .ENDIF
