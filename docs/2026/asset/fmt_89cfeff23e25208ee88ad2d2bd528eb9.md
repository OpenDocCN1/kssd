
```asm
    ;---- GET HEADER WRITE ACCESS -----

    mov edi,[ebp+dwImageBase]

    add edi,[edi+03Ch]        ;PE头

    assume edi : ptr IMAGE\_NT\_HEADERS            ; edi -> pointer to PE header

    mov esi,[ebp+dwImageBase]

    mov ecx,[edi].OptionalHeader.SizeOfHeaders        ;ECX = SizeOfHeaders

    assume edi : nothing

    ; 修改文件头的属性为PAGE\_READWRITE

    lea eax,[ebp+Buff]        ;保存现在的页属性

    push eax

    push PAGE\_READWRITE

    push ecx                        ;大小为SizeOfHeaders

    push [ebp+dwImageBase]

    call [ebp+\_VirtualProtect]

    ;是否要进行CRC校验

    test [ebp+PROTECTION\_FLAGS],CHECK\_HEADER\_CRC

    jz DontCheckCRC

    ;获取自身的文件名

    push MAX\_PATH

    lea edi,[ebp+Buff]        ;临时保存文件名

    push edi

    push 0

    call [ebp+\_GetModuleFileName]

    ;以只读方式打开自身

    push 0

    push FILE\_ATTRIBUTE\_NORMAL

    push OPEN\_EXISTING

    push NULL

    push FILE\_SHARE\_READ

    push GENERIC\_READ

    push edi

    call [ebp+\_CreateFile]

    .IF eax == INVALID\_HANDLE\_VALUE

       xor eax,eax

       jmp SkipChecksumCalc

    .ENDIF

    mov edi,eax

    ;获取文件大小

    push NULL

    push edi

    call [ebp+\_GetFileSize]

    sub eax,CHECKSUM\_SKIP\_SIZE

    xchg eax,esi        ;ESI = 文件大小

    ;申请内存空间

    push esi

    push GMEM\_FIXED+GMEM\_ZEROINIT

    call [ebp+\_GlobalAlloc]

    .IF eax == NULL

       jmp SkipChecksumCalcAndCleanUp

    .ENDIF

    xchg eax,ebx            ; EBX = 申请到的内存空间

    ;把文件读到申请的内存空间中

    push NULL

    lea eax,[ebp+Buff]        ;参数NumberOfBytesToRead

    push eax

    push esi                        ;大小

    push ebx                        ;申请到的内存空间

    push edi                        ;文件句柄

    call [ebp+\_ReadFile]

    ; 计算校验和

    mov eax,ebx

    mov ecx,esi

    PUSH EBX    ; [ESP] -> hMem

    PUSH EDI        ; EDI = hFile

    CALL GetChecksum

    mov [ebp+dwCalcedCRC],eax        ;保存校验和

    POP  EDI

    POP  EBX

    ; the calculated CRC will be compared at the start of the InitIT function >:-)

    LEA  EAX, [EBP+OFFSET AfterCRCCalcContinue]

    PUSH EAX            ;变形JMP

    RET

    ;-> Start of GetChecksum

   GetChecksum:            ;计算代码校验和

    ; eax = file image base

    ; ecx = filesize

    mov edi,eax                        ; edi -> data pointer

    xor eax,eax                        ; eax -> current bytes

    xor ebx,ebx                        ; ebx -> current checksum

    xor edx,edx                        ; edx -> Position (zero based)

    ; start calculation

   CheckSumLoop:

        mov al,byte ptr [edi]

        mul edx

        add ebx,eax

        inc edx

       inc edi

       loop CheckSumLoop

       xchg eax,ebx        ; eax -> checksum

       RET            ;返回值 EAX ＝校验和

       ;-> End of GetChecksum

   AfterCRCCalcContinue:

    ; clean up

    PUSH EBX

    call [ebp+\_GlobalFree]        ;释放空间

    xchg esi,eax

  SkipChecksumCalcAndCleanUp:

    push eax

    push edi

    call [ebp+\_CloseHandle]        ;关闭文件句柄

    pop eax

   SkipChecksumCalc:

   DontCheckCRC:

    ;解密

    mov eax,[ebp+dwImageBase]

    mov ebx,1

    CALL CryptPE

    LEA EAX, [EBP+OFFSET AfterDeCryptionContinue]

    PUSH EAX        ;JMP

    RET

;**************************************************************************

    ; eax = pointer to file memory

    ; ebx: 0 - 加密文件中的代码

    ;      1 - 解密内存中的代码

    CryptPE:

    mov edi,eax

    add edi,[edi+3Ch]                ;EDI = IMAGE\_NT\_HEADERS

    assume edi : ptr IMAGE\_NT\_HEADERS

    mov esi,edi

    add esi,0F8h                    ;ESI = IMAGE\_SECTION\_HEADER

    assume esi : ptr IMAGE\_SECTION\_HEADER

    xor edx,edx

    .REPEAT

       ;跳过下面这些区段，不进行加密

       .IF dword ptr [esi].Name1 == ('crsr')        ;.rsrc 资源

          jmp @@LoopEnd

       .ENDIF

       .IF dword ptr [esi].Name1 == ('rsr.')        ;.rsr  资源

          jmp @@LoopEnd

       .ENDIF

       .IF dword ptr [esi].Name1 == ('oler')        ;.relo 重定位

          jmp @@LoopEnd

       .ENDIF

       .IF dword ptr [esi].Name1 == ('ler.')        ;.rel 重定位

          jmp @@LoopEnd

       .ENDIF

       .IF dword ptr [esi].Name1 == ('Cy')            ;.yC 自己的段

          jmp @@LoopEnd

       .ENDIF

       .IF dword ptr [esi].Name1 == ('ade.')        ;.eda    ??

          jmp @@LoopEnd

       .ENDIF

       ;跳过空数据段

       .IF [esi].PointerToRawData == 0 || [esi].SizeOfRawData == 0

          jmp @@LoopEnd

       .ENDIF

       ;-> en-/decrypt it

           pushad

       mov ecx,[esi].SizeOfRawData

       .IF ebx == 0                ; 加密文件中的代码？

          mov esi,[esi].PointerToRawData

          ADD ESI, EAX            ;加上基址

          CALL EncryptSec

       .ELSE

          mov  esi,[esi].VirtualAddress

          add  esi,eax

          CALL DecryptSec

       .ENDIF

       JMP SecDecryptContinue1

    ; esi = CryptStart

    ; ecx = CryptSize

    DecryptSec:

        mov edi,esi

    SecDecryptLoop:

        LODSB

        SecDecryptBuff DB SEC\_PER\_SIZE DUP (0)            ;Make PER 函数生成的解密代码放在这里，进行解密

        STOSB            ;解密后的代码重新写回去

        LOOP SecDecryptLoop

        RET

    SecDecryptContinue1:

       popad

       @@LoopEnd:

       add esi,SIZEOF IMAGE\_SECTION\_HEADER                ;ESI 指向下一个节

       INC EDX

    .UNTIL dx == [edi].FileHeader.NumberOfSections            ;循环

    assume esi : nothing

    assume edi : nothing

    ret

;******************************************************************************************

   AfterDeCryptionContinue:

      ;------ PREPARE THE OEP JUMP EXCEPTION :) ------

    MOV  EBX, [EBP+dwImageBase]

    ADD  EBX, [EBP+dwOrgEntryPoint]

    ROR  EBX, 7

    MOV  [ESP+010h], EBX

    LEA  EBX, [EBP+OFFSET SehHandler\_OEP\_Jump]

    MOV  [ESP+01Ch], EBX

    ;

;下面代码将TLS索引清0

;**************************************************************************

    mov edi,[ebp+dwImageBase]

    add edi,dword ptr [edi+03Ch]            ;EDI = PE 头

    assume edi : ptr IMAGE\_NT\_HEADERS

    mov ebx,[edi].OptionalHeader.DataDirectory[SIZEOF IMAGE\_DATA\_DIRECTORY \* 9].VirtualAddress        ;Tls TABLE

    assume edi : nothing

    cmp ebx,0                     ; 存在TLS?

    jz SkipTlsFix

    add ebx,[ebp+dwImageBase]            ; ebx -> pointer to tls table

    assume ebx : ptr IMAGE\_TLS\_DIRECTORY32

    mov eax,[ebx].AddressOfIndex

    mov dword ptr [eax],0        ;TLS索引置0

    assume ebx : nothing

;**************************************************************************

  SkipTlsFix:

    ;校验和是否相等

    mov eax,[ebp+dwCalcedCRC]

    .IF eax != 0

       .IF eax != [ebp+dwOrgChecksum]

          jmp SkipInitIt        ;不相等就跳过初始化输入表

       .ENDIF

    .ENDIF

    ;----- INIT IMPORT TABLE -----

    ; 0 - 失败

    ; 1 - 成功

    LEA ESI, [EBP+OFFSET IIDInfo]                         ; ESI -> pointer to the current IID

    ASSUME ESI : PTR sItInfo

    ;------ PREPARE API REDIRECTION ------

    TEST [EBP+PROTECTION\_FLAGS], API\_REDIRECT\_FLAG

    .IF !ZERO?

       PUSH ESI

       LEA  EDI, [EBP+OFFSET Buff]

       ASSUME EDI : PTR sReThunkInfo

       XOR  ECX, ECX

;下面的循环，获取API函数的个数

       .WHILE [ESI].FirstThunk

              MOV  EDX, [ESI].FirstThunk

          ADD  EDX, [EBP+dwImageBase]

          .WHILE DWORD PTR [EDX]            ;获取一个IID的函数的个娄

             INC  ECX

             ADD  EDX, 4

          .ENDW

          ADD  ESI, SIZEOF sItInfo        ;指向下一个IID

       .ENDW

; 申请内存

       XOR  EDX, EDX

       MOV  EAX, SIZEOF sApiStub

       MUL  ECX

       PUSH EAX

       PUSH GMEM\_FIXED

       CALL [EBP+\_GlobalAlloc]

       .IF !EAX        ; fatal exit

              ADD  ESP, 4

          POPAD

          RET

       .ENDIF

       MOV  [EDI].ApiStubMemAddr, EAX        ;EAX = 申请到的内存地址

       MOV  [EDI].pNextStub, EAX

       ASSUME EDI : NOTHING

       POP  ESI

    .ENDIF
```
