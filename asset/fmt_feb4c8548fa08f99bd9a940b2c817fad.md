
```asm
    ; encrypt !  
    mov eax,pMem  
    mov ebx,0  
    call CryptPE           ;加密各个节  
    popad  
```

`CryptPE()`

```asm
;----- UPDATE PE HEADER -----  
mov edi,dwNTHeaderAddr  
assume edi : ptr IMAGE_NT_HEADERS        ; edi -> pointer to PE header  
push [esi].VirtualAddress             ;壳段的 RVA(刚开始的地方保存的就是壳的输入表)压栈  
pop [edi].OptionalHeader.DataDirectory[SIZEOF IMAGE_DATA_DIRECTORY].VirtualAddress        ;把 PE 头的的输入表指向壳的输入表  
; EntryPoint...  
mov eax,[esi].VirtualAddress          

add eax,IT_SIZE            ;壳段 RVA + 输入表的大小，后面的就是壳的代码  
mov [edi].OptionalHeader.AddressOfEntryPoint,eax         ;OEP 修改为壳的代码  
; SizeOfImage ...  
mov eax,[esi].VirtualAddress        ;基址  
add eax,[esi].Misc.VirtualSize    ;加上虚拟大小得到程序在内存的映像大小  
mov [edi].OptionalHeader.SizeOfImage,eax             ;修改 SizeOfImage  
; save protection flags...  
push dwProtFlags  
pop PROTECTION_FLAGS            ;保存加壳标志  
assume esi : nothing  
assume edi : nothing  
; ----- CALCULATE THE NEW EOF -----  
;计算文件加壳后大小  
mov eax,dwNewSectionRO  
add eax,IT_SIZE  
add eax,DEPACKER_CODE_SIZE  
mov dwNewFileEnd,eax          

; ----- COPY LOADER CODE TO FILE MEMORY & DO CHECKSUM STUFF ------ 
;复制壳代码到新节中  
mov edi,dwNewSectionRO  
add edi,IT_SIZE  
add edi,pMem  
mov esi,offset DepackerCode  
mov ecx,DEPACKER_CODE_SIZE  
rep movsb  
;----- ENCRYPT OEP JUMP CODE -----  
;下面的代码把跳到要 OEP 的这段代码加密  
MOV  EDI, pMem  
ADD  EDI, dwNewSectionRO  
ADD  EDI, IT_SIZE  
ADD  EDI, (OFFSET OEP_JUMP_CODE_START - OFFSET DepackerCode)  
MOV  ESI, EDI            ;ESI 指向 JUMP_CODE_START  
MOV  ECX, CRYPT_OEP_JUMP_SIZE  
XOR  EBX, EBX  

OepJumpEncryptLoop:        ;开始循环加密  
        LODSB  
       ROR  AL, 2  
       ADD  AL, BL  
       XOR  AL, OEP_JUMP_ENCRYPT_NUM     
       STOSB  
       INC EBX  
       LOOP OepJumpEncryptLoop     
;----- ENCRYPT LOADER -----  
; generate PER  
PUSH VAR_PER_SIZE  
MOV  EAX, pMem  
ADD  EAX, dwNewSectionRO  
ADD  EAX, IT_SIZE  
ADD  EAX, (OFFSET VarDecryptBuff - OFFSET DepackerCode)  
PUSH EAX            ;得到 VarDecryptBuff 的 VA  
PUSH OFFSET VarEncryptBuff          

CALL MakePER            ;产生加解密函数  
; encryption !  
MOV  EDI, pMem  
ADD  EDI, dwNewSectionRO  
ADD  EDI, IT_SIZE  
ADD  EDI, (OFFSET LOADER_CRYPT_START - OFFSET DepackerCode)        ;EDI = LOADER_CRYPT_START 的地址  
MOV  ECX, CRYPT_LOADER_SIZE_DB        ;大小  
MOV  ESI, EDI   

@@VarEncryptionLoop:        ;循环加密  
        LODSB  
VarEncryptBuff DB VAR_PER_SIZE DUP (0)            ;MakePER 产生的加密函数  
        STOSB  
        LOOP @@VarEncryptionLoop  
;----- CALCULATE CHECKSUM -----  
;计算文件校验和  
mov eax,pMem  
mov ecx,dwNewFileEnd  
sub ecx,CHECKSUM_SKIP_SIZE     
call GetChecksum  
mov dwOrgChecksum,eax        ;保存文件校验和  
;----- PASTE CHECKSUM ------  
MOV EAX, pMem  
ADD EAX, IT_SIZE  
ADD EAX, dwNewSectionRO  
ADD EAX, CHECKSUM_ADDR  
MOV EDX, dwOrgChecksum  
MOV DWORD PTR [EAX], EDX        ;保存校验和到壳段  
; ----- WRITE FILE MEMORY TO DISK -----  
;将加壳后的程序写到文件  
invoke SetFilePointer,hFile,0,NULL,FILE_BEGIN  
invoke WriteFile,hFile,pMem,dwOutPutSize,offset dwBytesWritten,NULL  
; ------ FORCE CALCULATED FILE SIZE ------  
;这是干什么的?????  
invoke SetFilePointer,hFile,dwNewFileEnd,NULL,FILE_BEGIN  
invoke SetEndOfFile,hFile  
invoke MessageBox,hDlg,offset szDone,offset szDoneCap,MB_ICONINFORMATION  
; ----- CLEAN UP -----  
push pMem  
call GlobalFree  
push hFile  
call CloseHandle  
@@Exit:  
    ret           
```

```asm
MemErr:  
    mov eax,offset szNoMemErr  
    jmp ShowErr  
PEErr:  
    mov eax,offset szNoPEErr  
    jmp ShowErr  
FileErr:  
    mov eax,offset szFileErr  
    jmp ShowErr  
NoRoom4SectionErr:  
    mov eax,offset szNoRoom4SectionErr  
    jmp ShowErr     
FsizeErr:  
    mov eax,offset szFsizeErr  
    jmp ShowErr  
SecNumErr:  
    mov eax,offset szSecNumErr  
    jmp ShowErr  
IIDErr:  
    MOV  EAX, OFFSET szIIDErr  
    JMP  ShowErr  
ShowErr:  
    invoke MessageBox,hDlg,eax,offset szErr,MB_ICONERROR  
    jmp @@Exit  
CryptFile ENDP  
```

```asm
;--------- functions ----------------- 
; esi = CryptStart  
; ecx = CryptSize  
EncryptSec:        ;加密函数  
    mov edi,esi  
SecEncryptLoop:  
    LODSB  
    SecEncryptBuff DB SEC_PER_SIZE DUP (0)        ;MakePER 函数生成的加密函数代码放在这里，进行加密  
    STOSB            ;把加密后的代码写回去  
    LOOP SecEncryptLoop  
    RET  
```

`AddSection()`

```asm
;添加一个区段函数  
; 返回值:  
; 0 - 空间不足  
; 1 - 添加节失败，可能文件已经加壳  
; 否则: 返回一个指向新添加节的 IMAGE_SECTION_HEADER 的指针  

AddSection PROC USES edi esi ebx ecx edx, pMem_ : LPVOID  
    LOCAL dwSecNum    : DWORD  
    mov edi,pMem_  
    add DWORD PTR edi,[edi+03Ch]  
    assume edi : ptr IMAGE_NT_HEADERS        ; edi 指向 PE 头  
;检查是否有足够的空间来添加新节  
    xor eax,eax  
    mov ax,[edi].FileHeader.NumberOfSections        ;区段数  
    mov dwSecNum,eax  
    mov ecx,SIZEOF IMAGE_SECTION_HEADER  
    imul eax,eax,ecx                    ; eax = 所有 Section Header 的大小的和  
    add eax,SIZEOF IMAGE_SECTION_HEADER        ; 加上一个区段的大小  
    mov ecx,eax           ; ecx ＝  PE 头  
    sub ecx,pMem_           ; 减去 pMem 得到 DOS 头大小  
    add ecx,ecx           ; 加上新的总 SectionHeader 的大小  
    add ecx,0F8h           ; 加上 PE 头的大小得 ecx = sizeof (DOS 头 + Section Header +PE Header)  
    .IF ecx > [edi].OptionalHeader.SizeOfHeaders        ;如果 ecx < SizeOfHeaders  则有足够空间  
       xor eax,eax  
       jmp @@ExitProc_AS  
    .ENDIF  
    ; create a new section  
    mov esi,edi  
    add esi,0F8h        ;esi 加上 PE 头大小，此时 esi 指向 节表  
    assume esi : ptr IMAGE_SECTION_HEADER     
    mov edx,dwSecNum     
    sub edx,1        ;区段数减去 1，下面的 repeat 循环用到的计数器  
    ;下面循环把区段添加一个可写属性  
    .REPEAT  
       mov eax,[esi].Characteristics        ;得到区段的属性  
       or eax,080000000h             ;or 080000000，添加可写属性  
       mov [esi].Characteristics,eax        ;更新区段属性  
       add esi,SIZEOF IMAGE_SECTION_HEADER        ;指向下一个区段  
       dec edx  
    .UNTIL edx == 0        ;循环,循环结束里，ESI 指向最后一个区段  
    ; 开始添加区段  
    mov edx,esi        ;EDX 指向最后一个区段  
    add edx,SIZEOF IMAGE_SECTION_HEADER        ; EDX 指向新的区段  
    assume edx : ptr IMAGE_SECTION_HEADER  
    ; VirtualAddress...  
    mov eax,[esi].VirtualAddress        ;最后一个区段的 RVA  
```
