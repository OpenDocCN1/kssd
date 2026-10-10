

# Yoda Cryptor 1.2 的源码分析

> 原文：[`www.pediy.com/kssd/pediy11/113362.html`](https://www.pediy.com/kssd/pediy11/113362.html)

**前言**  
  
这两天分析了下 Yoda Cryptor 1.2 的源码和大家分享下  
本来想分析 Yoda's Protect 1.02 的，看了一下，代码是 C 语言的，不过那写界面的代码比用来实现主要功能的代码还要多，看得我晕了，就分析了 Yoda Cryptor 1.2 的 ASM 源码，比 VC 的那个简洁多了  
  
我主要分析了 CryptStuff.ASM 和 PER.ASM 这两个文件，主要功能是在 CryptFile 这个函数。  
分析过程可能会有错误，大家作为参考下就行了。  
有一些变量的定义在其它的两个文件大家可以在附件中下载原始的源码对着来看  
  
  
  
**CryptStuff.ASM**  
  
;------ MACROS -----  
PUPO MACRO pSrc, pDest  
    PUSH pSrc  
    POP  pDest  
ENDM  
  
;------ DEFINITIONS -------  
DEPACKER\_CODE\_SIZE      equ (offset DepackerCodeEnd - offset DepackerCode)  
CHECKSUM\_SKIP\_SIZE      equ 5    ; (don't include the saved checksum itself in the checksum calculation)  
TLS\_BACKUP\_ADDR         equ (offset TlsBackupLabel - offset DepackerCode)  
CHECKSUM\_ADDR           equ (OFFSET ChecksumLabel - OFFSET DepackerCode)  
CRYPT\_LOADER\_SIZE\_DB    EQU (OFFSET LOADER\_CRYPT\_END - OFFSET LOADER\_CRYPT\_START)  
CRYPT\_OEP\_JUMP\_SIZE     equ (OFFSET OEP\_JUMP\_CODE\_END - OFFSET OEP\_JUMP\_CODE\_START)  
IT\_SIZE                 equ 060h  
MAX\_SECTION\_NUM         equ 20  
MAX\_IID\_NUM             equ 30  
OEP\_JUMP\_ENCRYPT\_NUM    equ ('y')  
LOADER\_CRC\_CHECK\_SIZE   equ (OFFSET OEP\_JUMP\_CODE\_START - OFFSET DepackerCode)  
VAR\_PER\_SIZE            EQU 030h  
SEC\_PER\_SIZE            EQU 030h  
  
;------- CONST --------  
.const  
szDone                  db "File encrypted successfully !",0  
szDoneCap               db ":)",0  
szFileErr               db "File access error :(",0  
szNoPEErr               db "Invalid PE file !",0  
szNoMemErr              db "Not enough memory :(",0  
szFsizeErr              db "Files with a filesize of 0 aren't allowed !",0  
szNoRoom4SectionErr     db "There's no room for a new section :(",0  
szSecNumErr             db "Too many sections !",0  
szIIDErr                DB "Too much ImageImportDescriptors !",0  
  
ALIGN\_CORRECTION        dd 01000h        ; this big value is e.g. needed for WATCOM compiled files  
DEPACKER\_SECTION\_NAME   dd ('Cy')  
szKernel                db "KeRnEl32.dLl",0  
szLoadLibrary           db "LoadLibraryA",0  
szGetProcAddress        db "GetProcAddress",0  
  
;------- DATA ---------  
.data  
pMap                    dd 0  
dwBytesRead             dd 0  
dwBytesWritten          dd 0  
pMem                    dd 0  
dwFsize                 dd 0  
dwOutPutSize            dd 0  
dwNewFileEnd            dd 0  
dwNTHeaderAddr          dd 0  
dwSectionNum            dd 0  
dwNewSectionRO          dd 0  
dwOrgITRVA              dd 0  
hFile                   dd 0  
  
;------- CODE ---------  
.code  
CryptFile PROC szFname : LPSTR, hDlg : HWND,dwProtFlags : DWORD  
    assume fs : nothing  
      
    CALL InitRandom    ;这里初始化随机数种子主要利用了 GetTickCount  
      
    ;----- MAP THE FILE -----  
      
;;下面主要是分配内存空间并把程序读取到内存中,并且检测 PE 文件的有效性  
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
      
    invoke CreateFile,szFname,GENERIC\_WRITE + GENERIC\_READ,FILE\_SHARE\_WRITE + FILE\_SHARE\_READ,\\  
                       NULL,OPEN\_EXISTING,FILE\_ATTRIBUTE\_NORMAL,0  
    cmp eax,INVALID\_HANDLE\_VALUE  
    jz FileErr  
    mov hFile,eax  
    invoke GetFileSize,hFile,0        ;获取文件大小  
    .IF eax == 0  
       push hFile  
       call CloseHandle  
       jmp FsizeErr  
    .ENDIF  
    mov dwFsize,eax  
    mov eax,dwFsize  
    add eax,IT\_SIZE        ;IT\_SIZE 输入表大小  
    add eax,DEPACKER\_CODE\_SIZE        ;DEPACKER\_CODE\_SIZE 解码代码的大小  
    add eax,ALIGN\_CORRECTION  
    mov dwOutPutSize,eax        ;把上面的加起来得到要输出的文件大小  
    push eax  
    push GMEM\_FIXED + GMEM\_ZEROINIT  
    call GlobalAlloc        ;分配内存空间  
    .IF eax == NULL  
       push hFile  
       call CloseHandle  
       jmp MemErr  
    .ENDIF  
    mov pMem,eax  
    invoke ReadFile,hFile,pMem,dwFsize,offset dwBytesRead,NULL        ;把文件的内容读到内存中  
      
    ; ----- check the PE Signature and get some needed values -----  
    mov edi,pMem  
    .IF word ptr \[edi\] != 'ZM'        ;检验 IMAGE\_DOS\_SIGNATURE  
       push pMem  
       call GlobalFree  
       push hFile  
       call CloseHandle  
       jmp PEErr  
    .ENDIF  
    add edi,\[edi+3Ch\]  
    .IF word ptr \[edi\] != 'EP'        ;检验 IMAGE\_NT\_SIGNATURE  
       push pMem  
       call GlobalFree  
       push hFile  
       call CloseHandle  
       jmp PEErr  
    .ENDIF      
      
;;下面是保存程序的一些重要信息  
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
  
    mov dwNTHeaderAddr,edi  
    assume edi : ptr IMAGE\_NT\_HEADERS  
    push \[edi\].OptionalHeader.DataDirectory\[SIZEOF IMAGE\_DATA\_DIRECTORY\].VirtualAddress        ;获取程序的输入表 RVA  
    pop dwOrgITRVA  
    push word ptr \[edi\].FileHeader.NumberOfSections        ;获取程序的区段数  
    pop word ptr dwSectionNum  
    .IF dwSectionNum > MAX\_SECTION\_NUM            ;区段数不能大于 20  
       JMP SecNumErr  
    .ENDIF  
    push \[edi\].OptionalHeader.AddressOfEntryPoint        ;保存程序 OEP  
    pop dwOrgEntryPoint  
    push \[edi\].OptionalHeader.ImageBase            ;保存程序镜像基址  
    pop dwImageBase  
      
    ;----- DELETE Bound Import & IAT DIRECTORIES -----  
      
   
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
;下面销毁 IMAGE\_BOUND\_IMPORT\_DESCRIPTOR      
  
    XOR  EAX, EAX  
    MOV  ECX, 4  
    LEA  EDI, \[EDI\].OptionalHeader.DataDirectory\[11 \*SIZEOF IMAGE\_DATA\_DIRECTORY\].VirtualAddress        ;获取 IMAGE\_BOUND\_IMPORT\_DESCRIPTOR RVA  
    assume edi : nothing  
   DirDelLoop:  
        STOSD  
        LOOP DirDelLoop            ;用 0 填充掉 IMAGE\_BOUND\_IMPORT\_DESCRIPTOR  
  
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*      
;加密 DLL 和 API 名字 并且销毁 IID  
    PUSH dwOrgITRVA  
    PUSH pMem  
    CALL RVA2Offset     ;RVA2Offset 把 RVA 转成文件的偏移，这里得到原程序输入表的偏移  
    PUSH EAX  
    PUSH pMem  
  
    CALL ProcessOrgIT        ;加密原程序输入表  
    OR   EAX, EAX  
    .IF  ZERO?        ;是否加密成功  
       PUSH pMem  
       CALL GlobalFree  
       PUSH hFile  
       CALL CloseHandle  
       JMP  IIDErr  
    .ENDIF      
         
;;增加一个新节\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
    push pMem  
    call AddSection          
    .IF eax == 0  
       push pMem  
       call GlobalFree  
       push hFile  
       call CloseHandle  
       jmp NoRoom4SectionErr  
    .ENDIF  
      
;创建壳自己的输入表\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*88  
    xchg eax,esi                    ;ESI 指向新节的节头  
    assume esi : ptr IMAGE\_SECTION\_HEADER  
    mov eax,\[esi\].PointerToRawData  
    MOV  dwNewSectionRO, EAX  
    add eax,pMem  
    push \[esi\].VirtualAddress  
    push eax  
    call AssembleIT        ;开始创建壳的输入表  
      
  
;处理 Tls 表。把 Tls 复制到壳中\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
    push \[esi\].VirtualAddress  
    push pMem  
    call ProcessTlsTable  
      
;加密节表\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
    pushad  
    ; generate PER  
    ;这个 PER 是什么意思我也不知道，这里保留原注释  
    ;下面是产生加密和解密函数，同时会生成花指令。是动态生成的。每一次加壳生成的加解密函数都不相同  
    ;作者在文件中注释为此多态(PS:原来多态就是这样子的)。  
    ;生成的函数用来后面的加密节代码，和解密节代码、  
    PUSH SEC\_PER\_SIZE  
    PUSH OFFSET SecDecryptBuff            ;解密函数保存在这  
    PUSH OFFSET SecEncryptBuff            ;加密函数保存在这  
    CALL MakePER            ;生成加解官函数。  
      
    ; encrypt !  
    mov eax,pMem  
    mov ebx,0  
    call CryptPE            ;加密各个节  
    popad  
  
  
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*      
    ; ----- UPDATE PE HEADER -----  
    mov edi,dwNTHeaderAddr  
    assume edi : ptr IMAGE\_NT\_HEADERS        ; edi -> pointer to PE header  
      
    push \[esi\].VirtualAddress            ;壳段的 RVA(刚开始的地方保存的就是壳的输入表)压栈  
    pop \[edi\].OptionalHeader.DataDirectory\[SIZEOF IMAGE\_DATA\_DIRECTORY\].VirtualAddress            ;把 PE 头的的输入表指向壳的输入表  
    ; EntryPoint...  
    mov eax,\[esi\].VirtualAddress              
    add eax,IT\_SIZE            ;壳段 RVA + 输入表的大小，后面的就是壳的代码  
    mov \[edi\].OptionalHeader.AddressOfEntryPoint,eax        ;OEP 修改为壳的代码  
    ; SizeOfImage ...  
    mov eax,\[esi\].VirtualAddress        ;基址  
    add eax,\[esi\].Misc.VirtualSize    ;加上虚拟大小得到程序在内存的映像大小  
    mov \[edi\].OptionalHeader.SizeOfImage,eax            ;修改 SizeOfImage  
    ; save protection flags...  
    push dwProtFlags  
    pop PROTECTION\_FLAGS            ;保存加壳标志  
      
    assume esi : nothing  
    assume edi : nothing  
      
    ; ----- CALCULATE THE NEW EOF -----  
    ;计算文件加壳后大小  
    mov eax,dwNewSectionRO  
    add eax,IT\_SIZE  
    add eax,DEPACKER\_CODE\_SIZE  
    mov dwNewFileEnd,eax              
  
    ; ----- COPY LOADER CODE TO FILE MEMORY & DO CHECKSUM STUFF ------  
;复制壳代码到新节中  
    mov edi,dwNewSectionRO  
    add edi,IT\_SIZE  
    add edi,pMem  
    mov esi,offset DepackerCode  
    mov ecx,DEPACKER\_CODE\_SIZE  
    rep movsb  
      
    ;----- ENCRYPT OEP JUMP CODE -----  
    ;下面的代码把跳到要 OEP 的这段代码加密  
    MOV  EDI, pMem  
    ADD  EDI, dwNewSectionRO  
    ADD  EDI, IT\_SIZE  
    ADD  EDI, (OFFSET OEP\_JUMP\_CODE\_START - OFFSET DepackerCode)  
    MOV  ESI, EDI                ;ESI 指向 JUMP\_CODE\_START  
    MOV  ECX, CRYPT\_OEP\_JUMP\_SIZE  
    XOR  EBX, EBX  
   OepJumpEncryptLoop:        ;开始循环加密  
        LODSB  
       ROR  AL, 2  
       ADD  AL, BL  
       XOR  AL, OEP\_JUMP\_ENCRYPT\_NUM      
       STOSB  
       INC EBX  
       LOOP OepJumpEncryptLoop      
  
    ;----- ENCRYPT LOADER -----  
    ; generate PER  
    PUSH VAR\_PER\_SIZE  
    MOV  EAX, pMem  
    ADD  EAX, dwNewSectionRO  
    ADD  EAX, IT\_SIZE  
    ADD  EAX, (OFFSET VarDecryptBuff - OFFSET DepackerCode)  
    PUSH EAX                ;得到 VarDecryptBuff 的 VA  
    PUSH OFFSET VarEncryptBuff                  
    CALL MakePER                ;产生加解密函数  
      
    ; encryption !  
    MOV  EDI, pMem  
    ADD  EDI, dwNewSectionRO  
    ADD  EDI, IT\_SIZE  
    ADD  EDI, (OFFSET LOADER\_CRYPT\_START - OFFSET DepackerCode)            ;EDI = LOADER\_CRYPT\_START 的地址  
    MOV  ECX, CRYPT\_LOADER\_SIZE\_DB            ;大小  
    MOV  ESI, EDI      
   @@VarEncryptionLoop:            ;循环加密  
        LODSB  
    VarEncryptBuff DB VAR\_PER\_SIZE DUP (0)                ;MakePER 产生的加密函数  
        STOSB  
        LOOP @@VarEncryptionLoop  
  
    ;----- CALCULATE CHECKSUM -----  
    ;计算文件校验和  
    mov eax,pMem  
    mov ecx,dwNewFileEnd  
    sub ecx,CHECKSUM\_SKIP\_SIZE      
    call GetChecksum  
    mov dwOrgChecksum,eax        ;保存文件校验和  
  
    ;----- PASTE CHECKSUM ------  
    MOV EAX, pMem  
    ADD EAX, IT\_SIZE  
    ADD EAX, dwNewSectionRO  
    ADD EAX, CHECKSUM\_ADDR  
    MOV EDX, dwOrgChecksum  
    MOV DWORD PTR \[EAX\], EDX        ;保存校验和到壳段  
  
    ; ----- WRITE FILE MEMORY TO DISK -----  
    ;将加壳后的程序写到文件  
    invoke SetFilePointer,hFile,0,NULL,FILE\_BEGIN  
    invoke WriteFile,hFile,pMem,dwOutPutSize,offset dwBytesWritten,NULL  
      
    ; ------ FORCE CALCULATED FILE SIZE ------  
    ;这是干什么的?????  
    invoke SetFilePointer,hFile,dwNewFileEnd,NULL,FILE\_BEGIN  
    invoke SetEndOfFile,hFile  
      
    invoke MessageBox,hDlg,offset szDone,offset szDoneCap,MB\_ICONINFORMATION  
  
    ; ----- CLEAN UP -----  
    push pMem  
    call GlobalFree  
    push hFile  
    call CloseHandle  
@@Exit:  
    ret                  
;加密完成\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
  
  
  
;----- ERROR MESSAGES -----  
MemErr:  
    mov eax,offset szNoMemErr  
    jmp ShowErr  
PEErr:  
    mov eax,offset szNoPEErr  
    jmp ShowErr  
FileErr:  
    mov eax,offset szFileErr  
    jmp ShowErr  
      
NoRoom4SectionErr:  
    mov eax,offset szNoRoom4SectionErr  
    jmp ShowErr      
FsizeErr:  
    mov eax,offset szFsizeErr  
    jmp ShowErr  
      
SecNumErr:  
    mov eax,offset szSecNumErr  
    jmp ShowErr  
      
IIDErr:  
    MOV  EAX, OFFSET szIIDErr  
    JMP  ShowErr  
      
ShowErr:  
    invoke MessageBox,hDlg,eax,offset szErr,MB\_ICONERROR  
    jmp @@Exit  
CryptFile ENDP  
  
;--------- functions -----------------  
  
; esi = CryptStart  
; ecx = CryptSize  
EncryptSec:                ;加密函数  
    mov edi,esi  
SecEncryptLoop:  
    LODSB  
    SecEncryptBuff DB SEC\_PER\_SIZE DUP (0)            ;MakePER 函数生成的加密函数代码放在这里，进行加密  
    STOSB                        ;把加密后的代码写回去  
    LOOP SecEncryptLoop  
    RET  
  
  
  
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*      
;添加一个区段函数  
; 返回值:  
; 0 - 空间不足  
; 1 - 添加节失败，可能文件已经加壳  
; 否则: 返回一个指向新添加节的 IMAGE\_SECTION\_HEADER 的指针  
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
  
AddSection PROC USES edi esi ebx ecx edx, pMem\_ : LPVOID  
    LOCAL dwSecNum    : DWORD  
  
    mov edi,pMem\_  
    add DWORD PTR edi,\[edi+03Ch\]  
    assume edi : ptr IMAGE\_NT\_HEADERS            ; edi 指向 PE 头  
      
;检查是否有足够的空间来添加新节  
    xor eax,eax  
    mov ax,\[edi\].FileHeader.NumberOfSections            ;区段数  
    mov dwSecNum,eax  
    mov ecx,SIZEOF IMAGE\_SECTION\_HEADER  
    imul eax,ecx                                           ; eax = 所有 Section Header 的大小的和  
    add eax,SIZEOF IMAGE\_SECTION\_HEADER                    ; 加上一个区段的大小  
    mov ecx,edi                           ; ecx ＝  PE 头  
    sub ecx,pMem\_                           ; 减去 pMem 得到 DOS 头大小  
    add ecx,eax                           ; 加上新的总 SectionHeader 的大小  
    add ecx,0F8h                           ; 加上 PE 头的大小得 ecx = sizeof (DOS 头 + Section Header +PE Header)  
    .IF ecx > \[edi\].OptionalHeader.SizeOfHeaders            ;如果 ecx < SizeOfHeaders  则有足够空间  
       xor eax,eax  
       jmp @@ExitProc\_AS  
    .ENDIF  
      
    ; create a new section  
    mov esi,edi  
    add esi,0F8h        ;esi 加上 PE 头大小，此时 esi 指向 节表  
    assume esi : ptr IMAGE\_SECTION\_HEADER      
    mov edx,dwSecNum      
    sub edx,1        ;区段数减去 1，下面的 repeat 循环用到的计数器  
    ;下面循环把区段添加一个可写属性  
    .REPEAT  
       mov eax,\[esi\].Characteristics            ;得到区段的属性  
       or eax,080000000h                    ;or 080000000，添加可写属性  
       mov \[esi\].Characteristics,eax            ;更新区段属性  
  
       add esi,SIZEOF IMAGE\_SECTION\_HEADER            ;指向下一个区段  
       dec edx  
    .UNTIL edx == 0        ;循环,循环结束里，ESI 指向最后一个区段  
    ; 开始添加区段  
    mov edx,esi            ;EDX 指向最后一个区段  
    add edx,SIZEOF IMAGE\_SECTION\_HEADER        ; EDX 指向新的区段  
    assume edx : ptr IMAGE\_SECTION\_HEADER  
      
    ; VirtualAddress...  
    mov eax,\[esi\].VirtualAddress        ;最后一个区段的 RVA  
    add eax,\[esi\].Misc.VirtualSize    ;加上大小得到未对齐时区段末尾  
    push 01000h            ;对齐大小  
    push eax                ;  
    call PEAlign        ;对齐，得到对齐后区段末尾，就是新节的开始  
    mov \[edx\].VirtualAddress,eax        ;新节的开始  
      
    ; VirtualSize..  
    mov \[edx\].Misc.VirtualSize,02000h        ;新节的大小为 2000h  
      
    ; RawSize..  
    mov eax,IT\_SIZE        ;新的开始加上壳的输入表大小  
    add eax,DEPACKER\_CODE\_SIZE            ;加上壳代码的大小得到新区段的大小  
    mov \[edx\].SizeOfRawData,eax        ;写入 SizeOfRawData  
      
    ; Section name  
    lea  eax,\[edx\].Name1            ;写入区段名  
    push DEPACKER\_SECTION\_NAME  
    pop  \[eax\]  
    MOV  DWORD PTR \[EAX+4\],0  
      
    ; Characteristics  
    mov \[edx\].Characteristics,0E00000E0h            ;写入新区段属性  
      
    ; RawOffset  
    mov eax,\[esi\].PointerToRawData        ;最后一个区段的偏移  
    add eax,\[esi\].SizeOfRawData            ;加上大小，指向还没对齐的段末尾  
    push 0200h  
    push eax  
    call PEAlign            ;对齐，对到新节的偏移  
    mov \[edx\].PointerToRawData,eax            ;写入新节偏移  
    mov eax,edx                    ; eax =新节的偏移，作为返回值  
      
    ; update the PE header  
    inc \[edi\].FileHeader.NumberOfSections        ;区段数加 1  
      
    assume edx : nothing  
    assume esi : nothing  
    assume edi : nothing      
@@ExitProc\_AS:  
    ret  
AddSection ENDP  
  
  
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*8  
;函数功能:创建壳自己的输入表，里面包含壳要用到的函数,LoadLibraryA,GetProcAddressA  
;参数: pAdress4IT      壳输入表的 地址，  
;        dwNewSectionVA  壳段的 VA               
;反回值:  无  
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*8  
  
AssembleIT PROC USES ebx ecx edx esi edi, pAddress4IT : LPVOID, dwNewSectionVA : DWORD  
    mov esi,pAddress4IT                    ; esi -> base of the new IT          
  
    ; Zero the memory for the new IT  
    mov eax,pAddress4IT  
    mov ecx,IT\_SIZE  
   ZeroMem:            ;用 0 初始化   
    mov byte ptr \[eax\],0  
    inc eax  
    loop ZeroMem      
      
    ; build a new,nice ImportTable :)  
    mov ebx,esi  
    mov eax,SIZEOF IMAGE\_IMPORT\_DESCRIPTOR  
    xor edx,edx  
    mov ecx,2            ;壳的输入表中包含 2 个 IID  
    mul ecx  
    add ebx,eax            ;ebx 指向 IID 末尾          
    assume esi:ptr IMAGE\_IMPORT\_DESCRIPTOR  
    mov eax,ebx                                             
    sub eax,esi            ;eax 等于 2 个 IID 的大小 ,最后一个 IID 是全 0，代码 IID 结束  
    add eax,dwNewSectionVA     ;eax 等于 IID 末尾的 RVA  
    mov \[esi\].Name1,eax         ;笫一个 IID 的 DLL 名字指向了 IID 末尾  
    push esi  
    mov esi,offset szKernel        ;szKernel  指向 "Kernel32.dll"  
    mov edi,ebx  
    .REPEAT            ;把 "Kernel32.dll" 填充到 IID 末尾  
       lodsb  
       stosb  
    .UNTIL byte ptr \[esi\] == 0  
    pop esi  
    mov ebx,edi  
    inc ebx                ;在"Kernel32.dll" 末尾空出一字节,是 IMAGE\_THUNK\_DATA 数组  
    mov eax,ebx                              
    sub eax,esi  
    add eax,dwNewSectionVA        ;RVA,这里保存一个 IMAGE\_THUNK\_DATA  
    mov \[esi\].FirstThunk,eax                                 
    mov edx,ebx  
    add edx,10        ;edx = IMAGE\_IMPORT\_BY\_NAME      
    mov eax,edx  
    sub eax,esi  
    add eax,dwNewSectionVA        ;IMAGE\_IMPORT\_BY\_NAME VA  
    mov \[ebx\],eax  
    add edx,2                        ;Hint  
    push esi  
    mov esi,offset szLoadLibrary        ;"LoadLibraryA"  
    mov edi,edx  
    .REPEAT        ;"LoadLibratyA" 填充到 edx.Name  
       lodsb  
       stosb  
    .UNTIL byte ptr \[esi\] == 0  
    pop esi  
    mov edx,edi      
    add ebx,4            ;下一个 IMAGE\_THUNK\_DATA  
    mov eax,edx            ;"GetProcAddress"  
    sub eax,esi  
    add eax,dwNewSectionVA    ;VA  
    mov \[ebx\],eax      
    add edx,2            ;Hint  
    mov esi,offset szGetProcAddress  
    mov edi,edx  
    .REPEAT                ;填充"GetProcAddress"到 edx.name  
       lodsb  
       stosb  
    .UNTIL byte ptr \[esi\] == 0      
    assume esi : nothing  
    ret  
AssembleIT ENDP  
  
  
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
;处理 Tls 表。把原程序的 Tls 复制到壳的代码中。并把数据上录的 Tls 地址修正到壳的 Tls 地址  
;参数:  pFIleMem         文件的基址  
;            CryptSectionVA     壳节基址 VA  
;返回值: 无  
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
  
ProcessTlsTable PROC USES edi ebx esi ecx, pFileMem : LPVOID, CryptSectionVA : DWORD  
    LOCAL  pTlsDirAddr : LPVOID  
      
    mov edi,pFileMem  
    add edi,\[edi+03Ch\]                ;EDI = PE 头  
    assume edi : ptr IMAGE\_NT\_HEADERS  
    lea ebx,\[edi\].OptionalHeader.DataDirectory\[SIZEOF IMAGE\_DATA\_DIRECTORY \* 9\].VirtualAddress  ;ebx = IMAGE\_TLS\_DIRECTORY32  
    mov pTlsDirAddr,ebx  
    mov ebx,\[ebx\]                ;ebx 指向 Tls 数据起始地址  
    assume edi : nothing  
    cmp ebx,0                     ; Tls 是否存在  
    jz ExitTlsFixProc  
      
    ; get a RAW pointer to the tls table  
    push ebx  
    push pFileMem  
    call RVA2Offset        ;计算 Tls 的偏移  
    cmp eax,0  
    jz ExitTlsFixProc  
    mov esi,pFileMem  
    add esi,eax                    ; esi 指向 Tls 的地址  
      
    ;把 Tls 复制到 壳的代码中保存  
    mov edi,offset TlsBackup  
    mov ecx,sizeof IMAGE\_TLS\_DIRECTORY32  
    rep movsb  
      
    ;把 Tls 的地址指向壳的 Tls  
    mov eax,CryptSectionVA  
    add eax,IT\_SIZE  
    add eax,TLS\_BACKUP\_ADDR    ;基址加上输入表大小，再加上壳 Tls 基于解压代码的偏移得到 Tls 的地址   
    mov esi,pTlsDirAddr  
    mov \[esi\],eax            ;修正地址  
  ExitTlsFixProc:  
    ret  
ProcessTlsTable ENDP  
  
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
;;处理加密输入表,函数会把 DLL 的名字，API 的名字加密保存真情为，并销毁原来的 IID  
;参数  
;    pFileImage    基址  
;    pITBaseR0   输入表在文件中的偏移  
  
;函数返回值   
;            1:成功  
;            0:IID 结构太大  
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
  
ProcessOrgIT PROC USES edi esi edx, pFileImage : LPVOID, pITBaseRO : LPVOID  
    LOCAL dwIIDNum : DWORD  
  
    ; clear the IIDInfo array  
    XOR EAX,EAX  
    MOV EDI, OFFSET IIDInfo  
    MOV ECX, SIZEOF IIDInfo  
   ClearArrayLoop:  
        STOSB  
        LOOP ClearArrayLoop    ;用 0 初始化 IIDInfo 数组  
      
    ; get a random number        ;产生一个随机数，后面会用来填充掉 IID  
    INVOKE GetTickCount  
    XOR EAX, ("yoda")  
    MOV EDX,EAX                            ; EDX -> stupid number :)  
      
    ; start  
    MOV  dwIIDNum, 0  
    MOV  EDI,pITBaseRO  
    ADD  EDI,pFileImage          
    ASSUME EDI : PTR IMAGE\_IMPORT\_DESCRIPTOR            ;EDI 指向笫一个 IID 结构  
    MOV ESI,OFFSET IIDInfo  
    ASSUME ESI : PTR sItInfo                    ;ESI 指向 一个 sItInfo 数组,主要用来保存 IID 的一些重要信心  
    .WHILE \[EDI\].Name1        ;循环加密 DLL 名字  
       ; too much IID's ?  
       INC  dwIIDNum  
       .IF dwIIDNum == (MAX\_IID\_NUM)  
           XOR  EAX, EAX  
           JMP  POIT\_Exit  
       .ENDIF  
         
       ; save IID Infos        ;保存 IID 的重要信息  
       PUPO <\[EDI\].Name1>, <\[ESI\].DllNameRVA>  
       PUPO <\[EDI\].OriginalFirstThunk>, <\[ESI\].OrgFirstThunk>  
       PUPO <\[EDI\].FirstThunk>, <\[ESI\].FirstThunk>  
         
       ;-> get dll pointer  
       PUSH  \[EDI\].Name1  
       PUSH  pFileImage      
       CALL  RVA2Offset  
       ADD   EAX, pFileImage        ;得到 DLL 名字的地址  
       ;-> crypt string  
       CALL EnDeCryptString            ;加密 DLL 名字  
           
         ;--- CRYPT API name strings ---  
         PUSH ESI  
         MOV  ESI, \[EDI\].OriginalFirstThunk  
         .IF !ESI            ;如果 OriginalFirstThunk 不为空，则使用 OriginalFirstThunk。否则使用 FirstThunk  
            MOV ESI, \[EDI\].FirstThunk  
         .ENDIF  
         PUSH ESI  
         PUSH pFileImage  
         CALL RVA2Offset          
         MOV  ESI, EAX  
         ADD  ESI, pFileImage            ;得到 IMAGE\_THUNK\_DATA  
         .WHILE DWORD PTR \[ESI\]        ;得到 IMAGE\_IMPORT\_BY\_NAME   
            MOV  EAX, \[ESI\]  
             
          TEST EAX,IMAGE\_ORDINAL\_FLAG32        ;判断 Hint 高位是否为 1，为 1 则函数以序号引出，跳过此函数  
          JNZ  SkipApiString  
            PUSH EAX  
            PUSH pFileImage  
            CALL RVA2Offset  
            OR   EAX, EAX  
            JZ   SkipApiString  
            ADD  EAX, pFileImage          
            ADD  EAX, 2                    ;跳过 Hint  
            CALL EnDeCryptString        ;加密 API 名字  
   SkipApiString:              
            ADD  ESI, 4        ;指向下一个 IMAGE\_THUNK\_DATA  
         .ENDW                ;循环加密 API 名字  
         POP ESI  
           
         ; 用随机数填充 IID  
         MOV \[EDI\].Name1, EDX              
         MOV \[EDI\].OriginalFirstThunk, EDX  
         MOV \[EDI\].FirstThunk, EDX  
         MOV \[EDI\].TimeDateStamp, EDX  
         MOV \[EDI\].ForwarderChain, EDX  
            
       ;EDI 指向下一个 IID，ESI 指向下一个 sItInfo  
       ADD EDI,SIZEOF IMAGE\_IMPORT\_DESCRIPTOR  
       ADD ESI,SIZEOF sItInfo  
    .ENDW        ;循环  
    ASSUME ESI : NOTHING  
    ASSUME EDI : NOTHING  
    XOR  EAX, EAX  
    INC  EAX        ;返回 1 代表加密成功  
   POIT\_Exit:  
    RET  
ProcessOrgIT ENDP  
  
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
;对齐函数  
PEAlign PROC USES ecx edx, dwTarNum : DWORD, dwAlignTo : DWORD  
    mov ecx,dwAlignTo  
    mov eax,dwTarNum  
    xor edx,edx  
    div ecx  
    cmp edx,0  
    jz AlreadyAligned  
    inc eax      
   AlreadyAligned:  
       mul ecx  
    ret  
PEAlign ENDP  
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
  
  
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
; 把一个 RVA 值转成在文件的偏移值  
; Base    - 映射基址  
; dwITRVA - 要转换的 RVA  
; 失败返回 0，成功返回转换后的偏移  
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
RVA2Offset PROC USES ebx ecx edx, Base : DWORD,dwITRVA : DWORD  
  
    mov eax,Base  
    add eax,\[eax+03Ch\]        ;eax 指向 PE 头  
    invoke ImageRvaToSection,eax,Base,dwITRVA        ;得到 RVA 所在区段的 Section Header  
    test eax,eax        ;是否成功  
    jz @@ExitProc  
      
    xchg eax,ebx  
    assume ebx : ptr IMAGE\_SECTION\_HEADER  
    mov eax,dwITRVA  
    sub eax,\[ebx\].VirtualAddress            ;RVA - 区段虚拟地址  
    add eax,\[ebx\].PointerToRawData        ;加上区段在文件的偏移，得到 RVA 在文件的偏移  
    assume ebx : nothing  
@@ExitProc:  
    ret  
RVA2Offset ENDP  
  
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
;下面是壳的代码  
DepackerCode:  
    pushad  
  
    ; get base ebp  
    call CallMe            ;重定位  
  CallMe:  
    pop ebp  
    sub ebp,offset CallMe  
      
    ;----- DECRYPT LOADER VARIABLES -----  
    MOV ECX, CRYPT\_LOADER\_SIZE\_DB  
    LEA EDI, \[EBP+OFFSET LOADER\_CRYPT\_START\]  
    MOV ESI, EDI  
   VarDecryptionLoop:  
        LODSB  
    VarDecryptBuff DB VAR\_PER\_SIZE DUP (0)            ;MakePER 函数产生的解密函数。对下面的代码进行解密  
        STOSB  
        LOOP VarDecryptionLoop  
          
LOADER\_CRYPT\_START:  
    ;------ DETECT WinNT ------  
    MOV  EAX, \[ESP+020h\]            ;判断是不是 NT?什么原理?  
    INC  EAX  
    JS   NoNT  
    MOV  DWORD PTR \[EBP+bNT\], 1  
   NoNT:      
  
        ;------ Get CRC OF LOADER CODE ------  
        LEA  EAX, \[EBP+OFFSET DepackerCode\]        ;计算校验和  
        MOV  ECX, LOADER\_CRC\_CHECK\_SIZE  
        CALL GetChecksum  
        MOV  \[EBP+dwLoaderCRC\], EAX        ;保存校验和  
      
    ;----- SI Check 1 -----  
    MOV EAX, \[ebp+PROTECTION\_FLAGS\]  
    AND EAX, CHECK\_SI\_FLAG          
    jz SkipSICheck  
      
    ; install SEH frame  
    LEA  ESI,\[EBP+SEH\]      
    ASSUME ESI : PTR sSEH  
    LEA  EAX, \[EBP+OFFSET SICheck1\_SP\]  
    mov  \[ESI\].SaveEip, EAX  
        ASSUME ESI : NOTHING  
    MOV  EDI, EBP  
    LEA  EAX, \[EBP+OFFSET SehHandler1\]        ;取 SehHandler1 的偏移  
    XOR  EBX, EBX  
    push EAX                ;安装 SEH      
    push FS:\[EBX\]  
    mov  FS:\[EBX\], ESP  
      
    ; 0 - SI not found  
    ; 1 - SI found  
       mov     ebp, 04243484Bh            ;?????  
        mov     ax, 04h  
        JMP     SM1  
        DB 0FFh            ;花指令  
  SM1:  
          INT  3        ;异常。跳到 SehHandler1  
          
   SICheck1\_SP:            ;SaveEip  
    MOV  EBP, EDI  
    ; uninstall SEH frame  
    XOR  EBX, EBX        ;移除 SEH  
        POP  FS:\[EBX\]      
        ADD  ESP, 4        ;  
          
       .IF AL != 4          
           ; exit  
           JMP SM2  
           DB 0E9h    ;花指令  
   SM2:    popad  
       ret  
        .ENDIF  
  SkipSICheck:  
      
    ;----- GET BASE API ADDRESSES -----  
    ; find the ImageImportDescriptor and grab dll addresses  
    mov eax,\[ebp+dwImageBase\]        ;获取镜像基址  
    add eax,\[eax+03Ch\]                ;EAX = PE Header  
    add eax,080h                        ;EAX = DataDirectory\[IMAGE\_DIRECTORY\_ENTRY\_IMPORT\].VirtuallAddress  
    mov ecx,\[eax\]                 ;ECX = IID RVA  
    add ecx,\[ebp+dwImageBase\]        ;IID VA  
    add ecx,16                    ;IID.FirstThunk  
    mov eax,dword ptr \[ecx\]            ;EAX = IMAGE\_THUNK\_DATA RVA  
    add eax,\[ebp+dwImageBase\]        ;Va  
    mov ebx,dword ptr \[eax\]            ;ebx 指向 LoadLibraryA 的地址  
    mov \[ebp+\_LoadLibrary\],ebx        ;保存 LoadLIbraryA 地址  
    add eax,4                            ;下一个 IMAGE\_THUNK\_DATA\_VA  
    mov ebx,dword ptr \[eax\]            ;GetProcAddressA 地址  
    mov \[ebp+\_GetProcAddress\],ebx    ;保存  
      
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*      
;获取需要用的函数地址  
    ; get kernel base  
    lea eax,\[ebp+offset szKernel32\]    ;"Kernel32.dll"  
    push eax  
    call \[ebp+\_LoadLibrary\]                ;"加载 Kernel32.dll"  
    mov esi,eax                    ; esi = Kernel32 handle  
    MOV \[EBP+dwKernelBase\], EAX    ;保存 Kernel32 handle  
  
    ;-> GetModuleHandle  
    lea eax,\[ebp+szGetModuleHandle\]  
    call DoGetProcAddr            ;获取 GetModuleHandle 地址  
    mov \[ebp+\_GetModuleHandle\],eax    ;保存  
      
    ;-> VirtualProtect  
    lea eax,\[ebp+szVirtualProtect\]          
    call DoGetProcAddr                ;获取 VirtualProtect 地址  
    mov \[ebp+\_VirtualProtect\],eax      
      
    ;-> GetModuleFileName      
    lea eax,\[ebp+szGetModuleFileName\]  
    call DoGetProcAddr                    ;获取 GetModuleFileName 地址  
    mov \[ebp+\_GetModuleFileName\],eax  
      
    ;-> CreateFile  
    lea eax,\[ebp+szCreateFile\]  
    call DoGetProcAddr                    ;获取 CreateFileA 地址  
    mov \[ebp+\_CreateFile\],eax  
      
    ;-> GlobalAlloc  
    lea eax,\[ebp+szGlobalAlloc\]  
    call DoGetProcAddr                    ;获取 GlobalAlloc 地址  
    mov \[ebp+\_GlobalAlloc\],eax  
      
    ;-> GlobalFree  
    lea eax,\[ebp+szGlobalFree\]  
    call DoGetProcAddr                    ;获取 GlobalFree 地址  
    mov \[ebp+\_GlobalFree\],eax  
      
    ;-> ReadFile  
    lea eax,\[ebp+szReadFile\]  
    call DoGetProcAddr                    ;获取 ReadFileA 地址  
    mov \[ebp+\_ReadFile\],eax  
      
    ;-> GetFileSize  
    lea eax,\[ebp+szGetFileSize\]  
    call DoGetProcAddr                    ;获取 GetFileSize 地址  
    mov \[ebp+\_GetFileSize\],eax  
      
    ;-> CloseHandle  
    lea eax,\[ebp+szCloseHandle\]  
    call DoGetProcAddr                    ;获取 CloseHandle 地址  
    mov \[ebp+\_CloseHandle\],eax  
      
    ; FUNNY JUMP :)  
    LEA EAX, \[EBP+OFFSET LoaderContinue1\]  
    PUSH EAX                                    ;变形 JMP 到 LoaderContinue1  
    RET  
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
  
; it's in an own function to keep a the loader code small  
; eax = address of API string  
; esi = target dll base      
DoGetProcAddr:  
    push eax  
    push esi  
    call \[ebp+\_GetProcAddress\]  
    ret  
  
LoaderContinue1:  
    ;----- ANTI DUMP -----  
    test \[ebp+PROTECTION\_FLAGS\],ANTI\_DUMP\_FLAG            ;是否要 anti dump  
    jz LetDumpable  
      
        push    fs:\[30h\]            ;PEB  
        pop     eax  
        TEST    EAX, EAX          
        JS      fuapfdw\_is9x     ; detected Win 9x  
   fuapfdw\_isNT:  
        MOV     EAX, \[EAX+0Ch\]    ;\_PEB\_LDR\_DATA  
        MOV     EAX, \[EAX+0Ch\]    ;LDR\_MODULE  
        MOV     DWORD PTR \[EAX+20h\], 1000h ; 修改进程映像大小 为 1000h  
        JMP     fuapfdw\_finished  
   fuapfdw\_is9x:  
        PUSH    0  
        CALL    \[ebp+\_GetModuleHandle\]        ;获取自身句柄  
        TEST    EDX, EDX  
        JNS     fuapfdw\_finished      ; Most probably incompatible!!!  
        CMP     DWORD PTR \[EDX+8\], -1        ;??????      
        JNE     fuapfdw\_finished      ; Most probably incompatible!!!  
        MOV     EDX, \[EDX+4\]          ; get address of internaly used  
                                      ; PE header  
        MOV     DWORD PTR \[EDX+50h\], 1000h ; increase size variable  
   fuapfdw\_finished:  
  
   LetDumpable:  
      
    ;---- GET HEADER WRITE ACCESS -----  
    mov edi,\[ebp+dwImageBase\]  
    add edi,\[edi+03Ch\]        ;PE 头  
    assume edi : ptr IMAGE\_NT\_HEADERS            ; edi -> pointer to PE header  
    mov esi,\[ebp+dwImageBase\]  
    mov ecx,\[edi\].OptionalHeader.SizeOfHeaders        ;ECX = SizeOfHeaders  
    assume edi : nothing  
          
    ; 修改文件头的属性为 PAGE\_READWRITE  
    lea eax,\[ebp+Buff\]        ;保存现在的页属性  
    push eax  
    push PAGE\_READWRITE  
    push ecx                        ;大小为 SizeOfHeaders  
    push \[ebp+dwImageBase\]  
    call \[ebp+\_VirtualProtect\]              
      
      
    ;是否要进行 CRC 校验  
    test \[ebp+PROTECTION\_FLAGS\],CHECK\_HEADER\_CRC  
    jz DontCheckCRC          
  
    ;获取自身的文件名  
    push MAX\_PATH  
    lea edi,\[ebp+Buff\]        ;临时保存文件名  
    push edi              
    push 0  
    call \[ebp+\_GetModuleFileName\]  
      
    ;以只读方式打开自身  
    push 0  
    push FILE\_ATTRIBUTE\_NORMAL  
    push OPEN\_EXISTING  
    push NULL  
    push FILE\_SHARE\_READ  
    push GENERIC\_READ  
    push edi  
    call \[ebp+\_CreateFile\]          
    .IF eax == INVALID\_HANDLE\_VALUE  
       xor eax,eax  
       jmp SkipChecksumCalc  
    .ENDIF  
    mov edi,eax              
    ;获取文件大小  
    push NULL  
    push edi  
    call \[ebp+\_GetFileSize\]  
    sub eax,CHECKSUM\_SKIP\_SIZE  
    xchg eax,esi        ;ESI = 文件大小      
    ;申请内存空间  
    push esi  
    push GMEM\_FIXED+GMEM\_ZEROINIT  
    call \[ebp+\_GlobalAlloc\]  
    .IF eax == NULL  
       jmp SkipChecksumCalcAndCleanUp  
    .ENDIF  
    xchg eax,ebx            ; EBX = 申请到的内存空间  
    ;把文件读到申请的内存空间中  
    push NULL  
    lea eax,\[ebp+Buff\]        ;参数 NumberOfBytesToRead  
    push eax  
    push esi                        ;大小  
    push ebx                        ;申请到的内存空间  
    push edi                        ;文件句柄  
    call \[ebp+\_ReadFile\]  
      
    ; 计算校验和  
    mov eax,ebx  
    mov ecx,esi  
    PUSH EBX    ; \[ESP\] -> hMem  
    PUSH EDI        ; EDI = hFile  
      
    CALL GetChecksum  
    mov \[ebp+dwCalcedCRC\],eax        ;保存校验和  
      
    POP  EDI  
    POP  EBX  
    ; the calculated CRC will be compared at the start of the InitIT function >:-)  
    LEA  EAX, \[EBP+OFFSET AfterCRCCalcContinue\]  
    PUSH EAX            ;变形 JMP  
    RET  
  
    ;-> Start of GetChecksum  
   GetChecksum:            ;计算代码校验和  
    ; eax = file image base  
    ; ecx = filesize      
    mov edi,eax                        ; edi -> data pointer  
    xor eax,eax                        ; eax -> current bytes  
    xor ebx,ebx                        ; ebx -> current checksum  
    xor edx,edx                        ; edx -> Position (zero based)  
      
    ; start calculation  
   CheckSumLoop:  
        mov al,byte ptr \[edi\]  
        mul edx  
        add ebx,eax          
        inc edx  
       inc edi         
       loop CheckSumLoop  
       xchg eax,ebx        ; eax -> checksum  
       RET            ;返回值 EAX ＝校验和  
       ;-> End of GetChecksum  
  
   AfterCRCCalcContinue:  
      
    ; clean up  
    PUSH EBX  
    call \[ebp+\_GlobalFree\]        ;释放空间  
    xchg esi,eax  
  
  SkipChecksumCalcAndCleanUp:      
    push eax  
    push edi  
    call \[ebp+\_CloseHandle\]        ;关闭文件句柄  
    pop eax  
   SkipChecksumCalc:  
   DontCheckCRC:  
  
    ;解密  
    mov eax,\[ebp+dwImageBase\]          
    mov ebx,1  
    CALL CryptPE  
    LEA EAX, \[EBP+OFFSET AfterDeCryptionContinue\]  
    PUSH EAX        ;JMP   
    RET  
      
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*      
    ; eax = pointer to file memory  
    ; ebx: 0 - 加密文件中的代码  
    ;      1 - 解密内存中的代码  
    CryptPE:  
    mov edi,eax  
    add edi,\[edi+3Ch\]                ;EDI = IMAGE\_NT\_HEADERS  
    assume edi : ptr IMAGE\_NT\_HEADERS      
    mov esi,edi  
    add esi,0F8h                    ;ESI = IMAGE\_SECTION\_HEADER  
    assume esi : ptr IMAGE\_SECTION\_HEADER          
    xor edx,edx  
    .REPEAT  
         
       ;跳过下面这些区段，不进行加密  
       .IF dword ptr \[esi\].Name1 == ('crsr')        ;.rsrc 资源  
          jmp @@LoopEnd  
       .ENDIF      
       .IF dword ptr \[esi\].Name1 == ('rsr.')        ;.rsr  资源  
          jmp @@LoopEnd  
       .ENDIF  
       .IF dword ptr \[esi\].Name1 == ('oler')        ;.relo 重定位  
          jmp @@LoopEnd  
       .ENDIF  
       .IF dword ptr \[esi\].Name1 == ('ler.')        ;.rel 重定位  
          jmp @@LoopEnd  
       .ENDIF  
       .IF dword ptr \[esi\].Name1 == ('Cy')            ;.yC 自己的段  
          jmp @@LoopEnd  
       .ENDIF  
       .IF dword ptr \[esi\].Name1 == ('ade.')        ;.eda    ??  
          jmp @@LoopEnd  
       .ENDIF  
         
       ;跳过空数据段  
       .IF \[esi\].PointerToRawData == 0 || \[esi\].SizeOfRawData == 0  
          jmp @@LoopEnd  
       .ENDIF  
     
       ;-> en-/decrypt it  
           pushad  
       mov ecx,\[esi\].SizeOfRawData  
       .IF ebx == 0                ; 加密文件中的代码？  
          mov esi,\[esi\].PointerToRawData  
          ADD ESI, EAX            ;加上基址  
          CALL EncryptSec  
       .ELSE                              
          mov  esi,\[esi\].VirtualAddress  
          add  esi,eax  
          CALL DecryptSec  
       .ENDIF  
  
       JMP SecDecryptContinue1  
         
    ; esi = CryptStart  
    ; ecx = CryptSize  
    DecryptSec:  
        mov edi,esi  
    SecDecryptLoop:  
        LODSB  
        SecDecryptBuff DB SEC\_PER\_SIZE DUP (0)            ;Make PER 函数生成的解密代码放在这里，进行解密  
        STOSB            ;解密后的代码重新写回去  
        LOOP SecDecryptLoop  
        RET  
          
    SecDecryptContinue1:         
       popad  
         
       @@LoopEnd:     
       add esi,SIZEOF IMAGE\_SECTION\_HEADER                ;ESI 指向下一个节  
       INC EDX  
    .UNTIL dx == \[edi\].FileHeader.NumberOfSections            ;循环  
    assume esi : nothing  
    assume edi : nothing  
    ret  
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
  
   AfterDeCryptionContinue:  
     
      ;------ PREPARE THE OEP JUMP EXCEPTION :) ------  
    MOV  EBX, \[EBP+dwImageBase\]  
    ADD  EBX, \[EBP+dwOrgEntryPoint\]  
    ROR  EBX, 7  
    MOV  \[ESP+010h\], EBX  
    LEA  EBX, \[EBP+OFFSET SehHandler\_OEP\_Jump\]  
    MOV  \[ESP+01Ch\], EBX  
      
    ;  
;下面代码将 TLS 索引清 0  
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
    mov edi,\[ebp+dwImageBase\]  
    add edi,dword ptr \[edi+03Ch\]            ;EDI = PE 头  
    assume edi : ptr IMAGE\_NT\_HEADERS  
    mov ebx,\[edi\].OptionalHeader.DataDirectory\[SIZEOF IMAGE\_DATA\_DIRECTORY \* 9\].VirtualAddress        ;Tls TABLE  
    assume edi : nothing  
    cmp ebx,0                     ; 存在 TLS?  
    jz SkipTlsFix  
    add ebx,\[ebp+dwImageBase\]            ; ebx -> pointer to tls table  
    assume ebx : ptr IMAGE\_TLS\_DIRECTORY32  
    mov eax,\[ebx\].AddressOfIndex  
    mov dword ptr \[eax\],0        ;TLS 索引置 0  
    assume ebx : nothing  
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*      
      
  SkipTlsFix:  
  
    ;校验和是否相等  
    mov eax,\[ebp+dwCalcedCRC\]  
    .IF eax != 0  
       .IF eax != \[ebp+dwOrgChecksum\]  
          jmp SkipInitIt        ;不相等就跳过初始化输入表  
       .ENDIF  
    .ENDIF  
      
    ;----- INIT IMPORT TABLE -----  
    ; 0 - 失败  
    ; 1 - 成功  
    LEA ESI, \[EBP+OFFSET IIDInfo\]                         ; ESI -> pointer to the current IID  
    ASSUME ESI : PTR sItInfo  
      
    ;------ PREPARE API REDIRECTION ------  
    TEST \[EBP+PROTECTION\_FLAGS\], API\_REDIRECT\_FLAG  
    .IF !ZERO?  
       PUSH ESI  
       LEA  EDI, \[EBP+OFFSET Buff\]  
       ASSUME EDI : PTR sReThunkInfo  
       XOR  ECX, ECX  
         
;下面的循环，获取 API 函数的个数  
       .WHILE \[ESI\].FirstThunk  
              MOV  EDX, \[ESI\].FirstThunk  
          ADD  EDX, \[EBP+dwImageBase\]  
          .WHILE DWORD PTR \[EDX\]            ;获取一个 IID 的函数的个娄  
             INC  ECX  
             ADD  EDX, 4  
          .ENDW         
          ADD  ESI, SIZEOF sItInfo        ;指向下一个 IID  
       .ENDW  
         
; 申请内存  
       XOR  EDX, EDX  
       MOV  EAX, SIZEOF sApiStub      
       MUL  ECX  
       PUSH EAX  
       PUSH GMEM\_FIXED  
       CALL \[EBP+\_GlobalAlloc\]  
       .IF !EAX        ; fatal exit  
              ADD  ESP, 4  
          POPAD  
          RET  
       .ENDIF  
         
       MOV  \[EDI\].ApiStubMemAddr, EAX        ;EAX = 申请到的内存地址  
       MOV  \[EDI\].pNextStub, EAX      
       ASSUME EDI : NOTHING  
       POP  ESI  
    .ENDIF      
  
    ; start with the real routine      
    ;解密 DLL 名字  
    .WHILE \[esi\].FirstThunk != 0  
       mov ebx,\[esi\].DllNameRVA  
       add ebx,\[ebp+dwImageBase\]  
       MOV EAX,EBX  
       CALL EnDeCryptString  
       LEA EAX, \[EBP+InitITContinue1\]        ; goto InitITContinue1  
       PUSH EAX  
       RET  ;JMP InitITContinue1,加载 DLL  
         
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*         
    ; 参数;eax = 要解密的字符串的 VA  
   EnDeCryptString:  
         PUSH ESI  
         PUSH EDI             
        MOV ESI,EAX  
        MOV EDI,EAX  
    DllCryptLoop:        ;  
           LODSB  
           ROR AL,4  
           STOSB  
           CMP BYTE PTR \[EDI\],0  
           JNZ DllCryptLoop  
           POP EDI  
           POP ESI  
        RET         
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
  
   InitITContinue1:  
       push ebx  
       call \[ebp+\_LoadLibrary\]        ;加载 DLL  
       test eax,eax  
       jz SkipInitIt  
      
;下面要销毁 DLL 名字  
       PUSH EAX                        
       test \[ebp+PROTECTION\_FLAGS\],DESTROY\_IMPORT\_FLAG  
       jz DontKillDllName  
       LEA EAX, \[EBP+OFFSET DontKillDllName\]  
       PUSH EAX         ;KillString 的返回地址  
       MOV EAX, EBX  
       JMP KillString            ;销毁 DLL 名字  
   DontKillDllName:  
          POP EBX         ;EBX = DLL 的句柄  
      
       ; process the (Original-)FirstThunk members  
       mov ecx,\[esi\].OrgFirstThunk  
       .IF ecx == 0  
          mov ecx,\[esi\].FirstThunk  
       .ENDIF  
       add ecx,\[ebp+dwImageBase\]            ;ECX 指向 IMAGE\_THUNK\_DATA  
       mov edx,\[esi\].FirstThunk  
       add edx,\[ebp+dwImageBase\]            ; edx - IAT ，IMAGE\_IMPORT\_BY\_NAME  
       .WHILE dword ptr \[ecx\] != 0  
          test dword ptr \[ecx\],IMAGE\_ORDINAL\_FLAG32           ;判断是否由序号引出  
          jnz @@OrdinalImp  
        
          ; process a name import  
          mov dword ptr eax,\[ecx\]            ;EAX ＝ IMAGE\_IMPORT\_BY\_NAME  
          add eax,2                            ;跳过 Hint  
          add eax,\[ebp+dwImageBase\]     ;EAX = API NAME  
          PUSH EAX  
          CALL EnDeCryptString                ;解密 APINAME  
          POP  EAX  
          mov edi,eax                    ; 保存 API 名字指针，后面会用来销毁 API 名字  
          push edx  
          push ecx          ;保存 IMAGE\_THUNK\_DATA 指针  
          push eax                      
          push ebx  
          call \[ebp+\_GetProcAddress\]            ;获取函数地址  
          .IF eax == NULL  
             pop ecx  
             pop edx  
             jmp SkipInitIt  
          .ENDIF  
          pop ecx  
          pop edx  
          ;销毁 API 名字，这段很像 Acprotect 中销毁字符串的代码  
            PUSHAD  
          test \[ebp+PROTECTION\_FLAGS\],DESTROY\_IMPORT\_FLAG          
            JZ  DontKillApiName  
            LEA EAX, \[EBP+OFFSET DontKillApiName\]         ;保存返回地址  
            PUSH EAX  
          MOV EAX, EDI  
          JMP KillString            ;销毁  
   DontKillApiName:            
          POPAD  
          ;-> paste API address  
          mov dword ptr \[edx\],eax              ;API 地址到填充 IAT  
          jmp @@NextThunkPlease              
            
   @@OrdinalImp:  
              ; process an ordinal import  
              push edx  
              push ecx                                         ; save the thunk pointers  
              mov dword ptr eax,\[ecx\]        ;取得函数序号  
              sub eax,080000000h                ;去掉高位  
              push eax  
              push ebx  
              call \[ebp+\_GetProcAddress\]            ;获取 API 函数地址  
              test eax,eax  
              jz SkipInitIt  
              pop ecx  
              pop edx  
              mov dword ptr \[edx\],eax                ;写到 IAT  
                
   ;下面是 IAT 重定向的处理             
   @@NextThunkPlease:  
             ; eax = Current Api address  
             ; ebx = dll base  
             ; edx = non-org thunk pointer  
             TEST \[EBP+PROTECTION\_FLAGS\], API\_REDIRECT\_FLAG  
             .IF !ZERO?  
                .IF \[EBP+bNT\]            ;如果是 NT 下的非系统 DLL 则跳过  
                    .IF EBX < 070000000h || EBX > 077FFFFFFh  
                        JMP SkipThunkRed  
                    .ENDIF  
                .ELSE  
                    .IF EBX < 080000000h                ;98 下的非系统 DLL 同样跳过  
                        JMP SkipThunkRed  
                    .ENDIF  
                .ENDIF  
                PUSH EDI  
                PUSH ESI  
                LEA  EDI, \[EBP+Buff\]  
                ASSUME EDI : PTR sReThunkInfo  
                MOV  ESI, \[EDI\].pNextStub  
                MOV  \[EDX\], ESI        ; make the thunk point to stub mem  
                SUB  EAX, ESI  
                SUB  EAX, 5            ; eax - esi -5 得到 JMP xxxxx 后面的偏移  
                MOV  BYTE PTR \[ESI\], 0E9h        ;写入 E9--JMP   
                MOV  DWORD PTR \[ESI+1\], EAX    ;将函数地址写进去  
                ADD  \[EDI\].pNextStub, SIZEOF sApiStub        ;指向一个 sApiStub  
                ASSUME EDI : NOTHING  
                POP  ESI  
                POP  EDI  
          SkipThunkRed:  
             .ENDIF  
               
          add ecx,4            ;下一个 IMAGE\_THUNK\_DATA  
          add edx,4            ;下一个 IMAGE-IMPORT\_BY\_NAME  
       .ENDW      
       add esi,SIZEOF sItInfo                    ; ESI = 下一个 IID  
    .ENDW  
    assume esi:nothing  
    xor eax,eax  
    inc eax  
SkipInitIt:  
  
    .IF eax != TRUE  
       ; exit  
       popad  
       ret  
    .ENDIF  
      
      ;----- ERASE PE HEADER ------  
      test \[ebp+PROTECTION\_FLAGS\],ERASE\_HEADER\_FLAG  
      jz SkipEraseHeader  
        
;下面是销毁 PE 头。好猥琐!  
    mov edi,\[ebp+dwImageBase\]  
    add edi,\[edi+03Ch\]                                        ;PE 头  
    assume edi : ptr IMAGE\_NT\_HEADERS              
    mov ecx,\[edi\].OptionalHeader.SizeOfHeaders        ;大小  
    mov esi,\[ebp+dwImageBase\]                                ;ESI= 镜像基址  
    assume edi : nothing  
   ZeroMemLoop:                ;用 0 填充掉  
        mov byte ptr \[esi\],0  
        inc esi  
        loop ZeroMemLoop  
  SkipEraseHeader:  
    
      ;------ CHECK AGAIN LOADER CRC & COMPARE ------  
      LEA  EAX, \[EBP+DepackerCode\]  
      MOV  ECX, LOADER\_CRC\_CHECK\_SIZE  
      JMP SM10  
      DB   0E9h                ;花指令  
      SM10:  
      CALL GetChecksum            ;校验和  
      JMP SM11  
      DB   0C7h                ;花指令  
      SM11:  
      MOV  EBX, \[EBP+dwLoaderCRC\]  
      XOR  EAX, EBX            ;检验校验和是否相等  
      .IF !ZERO?                ;不等返回  
         JMP SM12  
         DB  02Ch  
         SM12:  
         POPAD  
         JMP SM13  
         DB  0E8h  
         SM13:  
         RET  
      .ENDIF  
    
 ;解密    OEP\_JUMP\_CODE   
      LEA  EDI, \[EBP+OFFSET OEP\_JUMP\_CODE\_START\]  
      MOV  ESI, EDI  
      MOV  ECX, CRYPT\_OEP\_JUMP\_SIZE  
    XOR  EBX, EBX  
   OepJumpDecryptLoop:            ;下面是解密算法  
        LODSB  
       XOR  AL, OEP\_JUMP\_ENCRYPT\_NUM  
       SUB  AL, BL  
       ROL  AL, 2  
       STOSB  
       INC EBX  
       LOOP OepJumpDecryptLoop  
 ;跳到 OEP  
OEP\_JUMP\_CODE\_START:  
    ;----- CHECK FOR DEBUG API's -----  
    LEA EAX, \[EBP+OFFSET szIsDebuggerPresent\]  
    PUSH EAX  
    PUSH \[EBP+dwKernelBase\]  
    CALL \[EBP+\_GetProcAddress\]            ;获取 IsDebuggerPresent 函数地址  
    OR   EAX, EAX        ; 95 系统下不支持这个函数  
    .IF !ZERO?  
       CALL EAX            ;调用 IsDebugPresent 检测调试器  
       OR   EAX, EAX  
       .IF  !ZERO?        ;存在调试器，直接返回  
           POPAD  
          RET  
       .ENDIF  
    .ENDIF  
      
    ;------ SECOND SI CHECK ------  
    ; doesn't work on NT  
    ; install SEH frame  
    TEST \[EBP+PROTECTION\_FLAGS\], CHECK\_SI\_FLAG  
    JZ   SkipSICheck2  
    LEA  ESI,\[EBP+SEH\]  
    ASSUME ESI : PTR sSEH  
    LEA  EAX, \[EBP+OFFSET SICheck2\_SP\]  
    MOV  \[ESI\].SaveEip, EAX  
        ASSUME ESI : NOTHING  
        XOR  EBX, EBX  
    LEA  EAX, \[EBP+OFFSET SehHandler2\]  
    PUSH EAX  
    PUSH FS:\[EBX\]  
    MOV  FS:\[EBX\], ESP  
    MOV  EDI, EBP  
  
    MOV  EAX, 4400h  
    JMP SM4  
    DB 0C7h  
   SM4:  
    INT  68h  
   SICheck2\_SP:      
        XOR  EBX, EBX  
    POP  FS:\[EBX\]  
    ADD  ESP, 4  
      
    .IF DI == 01297h || DI == 01277h || DI == 01330h  
       JMP SM5  
       DB 0FFh  
           SM5:         
       POPAD  
       JMP SM6  
       DB 0E8h  
       SM6:  
       RET  
    .ENDIF  
   SkipSICheck2:  
   LEA  EAX, \[EBP+OFFSET OepJumpCodeCont\]  
   PUSH EAX  
   RET      
     
     
 ;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
  ;在这个异常处理会跳到 OEP 去  
    SehHandler\_OEP\_Jump PROC C pExcept:DWORD,pFrame:DWORD,pContext:DWORD,pDispatch:DWORD  
        PUSH EDI  
        MOV  EAX,pContext  
        ASSUME EAX : PTR CONTEXT  
          
        ; 恢复原来的 SEH 链  
        MOV  EDI, \[EAX\].regEsp  
        PUSH \[EDI\]  
        XOR  EDI, EDI  
        POP  FS:\[EDI\]  
          
        ;恢复原来的 SEH 链  
        ADD  \[EAX\].regEsp, 8  
          
        MOV  EDI, \[EAX\].regEbx    ; EDI ＝ 加密的 OEP  
        ROL  EDI, 7                ;解密得到真正的 OEP  
        MOV  \[EAX\].regEip, EDI    ;设置 EIP 到 OEP  
          
        mov  EAX,ExceptionContinueExecution            ;继续执行，会跳到 OEP  
        ASSUME EAX : NOTHING  
        POP  EDI  
        RET  
    SehHandler\_OEP\_Jump ENDP  
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
  
   OepJumpCodeCont:  
     
    ;用 0 填充掉 DepackerCode 到 SehHandler\_OEP\_Jump 的代码  
    XOR  AL,AL  
    LEA  EDI, \[EBP+OFFSET DepackerCode\]  
    MOV  ECX, (OFFSET SehHandler\_OEP\_Jump - OFFSET DepackerCode)  
   LoaderZeroLoop:  
    STOSB  
    LOOP LoaderZeroLoop  
      
    ;用 0 填充掉从 LOADER\_CRYPT\_END 到 OEP\_JUMP\_CODE\_END 的代码  
    LEA  EDI, \[EBP+OFFSET OEP\_JUMP\_CODE\_END\]  
    MOV  ECX, (OFFSET LOADER\_CRYPT\_END - OFFSET OEP\_JUMP\_CODE\_END)  
   LoaderVarZeroLoop:  
    STOSB  
    LOOP LoaderVarZeroLoop  
  
    POPAD    ; RESTORE STARTUP REGS  
        ; After this POPAD:  
        ; EAX - OEP Seh handler  
        ; EBX - OEP (rored)  
      
      ;安装 SEH,在 SehHandler 中跳到 OEP---      
    PUSH EAX  
    XOR  EAX, EAX  
    PUSH FS:\[EAX\]  
    MOV  FS:\[EAX\], ESP  
  
    JMP  SM3  
    DB   087H        ;花指令  
   SM3:         ; 跳到这里执行，因为下面的代码已经用 0 填充，所以会异常。中到  
  
OEP\_JUMP\_CODE\_END:  
  
; EAX = ASCII string address  
KillString:  
    .WHILE byte ptr \[eax\] != 0  
       mov byte ptr \[eax\],0  
       inc eax  
    .ENDW  
    ret  
      
SehHandler1 PROC C pExcept:DWORD,pFrame:DWORD,pContext:DWORD,pDispatch:DWORD  
    PUSH EDI  
    MOV  EAX,pContext  
    ASSUME EAX : PTR CONTEXT  
    MOV  EDI, \[EAX\].regEdi          
    push \[EDI+SEH.SaveEip\]        ;SaveEip 压栈  
    pop  \[eax\].regEip                ;弹出到 regEip。更改 EIP 的地址，程序会转到 SaveEip 去执行  
    MOV  \[eax\].regEbp, EDI        ;还原 EBP  
    MOV  \[EAX\].regEax, 4         ; SI NOT detected !  
    mov  EAX,ExceptionContinueExecution  ;返回到 SaveEip 去  
    ASSUME EAX : NOTHING  
    POP  EDI  
    RET  
SehHandler1 ENDP  
  
SehHandler2 PROC C pExcept:DWORD,pFrame:DWORD,pContext:DWORD,pDispatch:DWORD  
    PUSH EDI  
    MOV  EAX,pContext  
    ASSUME EAX : PTR CONTEXT  
    MOV  EDI, \[EAX\].regEdi  
    push \[EDI+SEH.SaveEip\]  
    pop  \[eax\].regEip  
    MOV  \[eax\].regEbp, EDI  
    MOV  \[EAX\].regEdi, 0         ; SI NOT detected !  
    mov  EAX,ExceptionContinueExecution  
    ASSUME EAX : NOTHING  
    POP  EDI  
    RET      
SehHandler2 ENDP  
  
;----- LOADER STRUCTS -----  
sItInfo STRUCT  
    DllNameRVA       dd ?  
    FirstThunk       dd ?  
    OrgFirstThunk    dd ?  
sItInfo ENDS  
  
sSEH STRUCT  
    OrgEsp           dd ?  
    OrgEbp           dd ?  
    SaveEip          dd ?  
sSEH ENDS  
  
sReThunkInfo STRUCT  
    ApiStubMemAddr   DD ?  
    pNextStub        DD ?  
sReThunkInfo ENDS  
  
sApiStub STRUCT                ; UNUSED !  
    JumpOpc          DB ?  
    JumpAddr         DD ?  
sApiStub ENDS  
      
;----- LOADER VARIABLES -----  
  
dwImageBase             dd 0  
dwOrgEntryPoint         dd 0  
PROTECTION\_FLAGS        dd 0  
dwCalcedCRC             dd 0  
dwLoaderCRC             DD 0  
bNT                     DD 0  
  
IIDInfo                 db (SIZEOF sItInfo \* MAX\_IID\_NUM) dup (0)  
  
SEH                     sSEH <0>  
  
\_LoadLibrary            dd 0  
\_GetProcAddress         dd 0  
  
; some API stuff  
szKernel32              db "Kernel32.dll",0  
dwKernelBase            dd 0  
szGetModuleHandle       db "GetModuleHandleA",0  
\_GetModuleHandle        dd 0  
szVirtualProtect        db "VirtualProtect",0  
\_VirtualProtect         dd 0  
szGetModuleFileName     db "GetModuleFileNameA",0  
\_GetModuleFileName      dd 0  
szCreateFile            db "CreateFileA",0  
\_CreateFile             dd 0  
szGlobalAlloc           db "GlobalAlloc",0  
\_GlobalAlloc            dd 0  
szGlobalFree            db "GlobalFree",0  
\_GlobalFree             dd 0  
szReadFile              db "ReadFile",0  
\_ReadFile               dd 0  
szGetFileSize           db "GetFileSize",0  
\_GetFileSize            dd 0  
szCloseHandle           db "CloseHandle",0  
\_CloseHandle            dd 0  
szIsDebuggerPresent     db "IsDebuggerPresent",0  
LOADER\_CRYPT\_END:  
  
; This variables won't be crypted:  
TlsBackupLabel:  
TlsBackup               IMAGE\_TLS\_DIRECTORY32 <0>  
  
ChecksumLabel:  
dwOrgChecksum           dd 0  
  
Buff                    db 0    ; buffer for some stuff, its size: 2000h(VS) - DEPACKER\_CODE\_SIZE  
  
;----- END OF PE LOADER CODE -----  
DepackerCodeEnd:  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
  
**PER.ASM**  
  
; -> Polymorphic En-/Decryption routine generator for per byte encryption <-  
; by yoda  
  
;---- STRUCTs ----  
sPERTable STRUCT  
    dwSize           DD ?  
    dwEncrypt        DD ?  
    dwDecrypt        DD ?  
    RandNumType      DD ?  
sPERTable ENDS  
  
; RandNumType:  
; 0 - no random num needed  
; 1 - 3th byte must be a random number  
; 2 - 2nd byte must be a random number  
  
;----- EQUs -----  
PERItems                   EQU 14  
  
;----- CONST ----  
.CONST  
; all opcodes are in reverse order  
  
;下面保存的是加密和解密的成对代码表  
PERTable                     
                DD 1  
               DD 090h                ; NOP  
               DD 090h                ; NOP  
               DD 0  
                 
               DD 1  
               DD 0F9h                ; STC  
               DD 0F9h                ; STC  
               DD 0  
                 
               DD 1  
               DD 0F8h                ; CLC  
               DD 0F8h                ; CLC  
               DD 0                 
  
               DD 2  
               DD 0C0FEh                ; INC  AL  
               DD 0C8FEh                            ; DEC  AL  
               DD 0  
                 
               DD 2  
               DD 00004                ; ADD AL, 0  
               DD 0002Ch                ; SUB AL, 0  
               DD 2  
  
               DD 2  
               DD 0002Ch                ; SUB AL, 0  
               DD 00004                ; ADD AL, 0  
               DD 2  
                 
               DD 2  
               DD 0C102h                ; ADD AL, CL  
               DD 0C12Ah                ; SUB AL, CL  
               DD 0  
  
               DD 2  
               DD 0C12Ah                ; SUB AL, CL  
               DD 0C102h                ; ADD AL, CL  
               DD 0  
  
               DD 2  
               DD 00034h                ; XOR AL, 0  
               DD 00034h                ; XOR AL, 0  
               DD 2  
  
               DD 3  
               DD 000C8C0h                          ; ROR  AL, 0  
               DD 000C0C0h                ; ROL  AL, 0  
               DD 1  
  
               DD 3  
               DD 000C0C0h                ; ROL  AL, 0  
               DD 000C8C0h           ; ROR  AL, 0  
               DD 1  
                 
               DD 3  
               DD 0E801EBh                ; Self modifing  
               DD 0E801EBh                ; Self modifing  
               DD 0  
                 
               DD 3  
               DD 0E901EBh                ; Self modifing  
               DD 0E901EBh                ; Self modifing  
               DD 0  
                 
               DD 3  
               DD 0C201EBh                ; Self modifing  
               DD 0C201EBh                ; Self modifing  
               DD 0                 
  
.DATA  
dwRandVal                  DD 0  
  
.CODE  
  
; srand should only called one time !!!  
;初始化随机数种子。此函数只会被调用 一次  
InitRandom PROC  
    ; manage the random generator   
    CALL GetTickCount  
    PUSH EAX  
    CALL srand  
    RET  
InitRandom ENDP  
  
  
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
;函数功能：    得用上面的加解密表产生随机的加密和解密代码  
;参数：        pEncryptBuff      加密代码  
;                pDecryptBuff        解码代码，和加密代码对应  
;                dwSize                要填充的缓冲区的大小  
;                返回值                没有  
;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
  
MakePER PROC pEncryptBuff : LPVOID, pDecryptBuff : LPVOID, dwSize : DWORD  
    LOCAL dwCurRandNum : DWORD  
      
    ; prepare some things  
    MOV  EDI, pEncryptBuff                    ; EDI -> EncryptBuffer  
    MOV  ESI, pDecryptBuff                    ; ESI -> DecryptBuffer  
    ADD  ESI, dwSize                    ; ESI will be filled from down to top  
      
    ; generate !  
    .REPEAT     
       ; get a random PER Item  
       PUSH PERItems  
       CALL rand            ;产生一个不大于 14 的随机数  
       MOV  EBX, SIZEOF sPERTable;  
       XOR  EDX, EDX  
       MUL  EBX  
       ADD  EAX, OFFSET PERTable  
       XCHG EAX, EDX                       ;EDX 指向一组加解密指令  
         
       ASSUME EDX : PTR sPERTable  
         
       ; is this item too big  
       MOV  EBX, \[EDX\].dwSize  
       CMP  EBX, dwSize                    ;比较有没有足够的空间来保存代码  
       JG   Retry  
         
       ;把表中的加密指令数据复制到加密代码区  
       MOV  ECX, \[EDX\].dwSize  
       MOV  EAX, \[EDX\].dwEncrypt         
  
       MOV  ECX, \[EDX\].dwSize  
       .WHILE ECX != 0  
           MOV  BYTE PTR \[EDI\], AL        ;把多态指令的一个字节填充到缓冲区  
           ADD  EDI, 1  
           ROR  EAX, 8        ;右移 8 位,AL 指向下一字节  
           DEC  ECX  
       .ENDW  
  
       ; generate the random num  
       MOV  EAX, \[EDX\].RandNumType  
       .IF  EAX == 1 || EAX == 2        ;判断指令对中是否需要填充随机数  
           MOV  EBX, EDI  
           SUB  EBX, 1            ;EBX 指向\[EDX\].Encrypt 的最后一个字节  
           PUSH 0F8h  
           CALL rand                ;产生一个小于 0F8h 的随机数  
           INC  EAX                ; 加 1 避免产生的随机数为 0!  
           MOV  dwCurRandNum, EAX            ;保存现在的随机数,解密的时候要用到  
           MOV  BYTE PTR \[EBX\], AL        ;填写随机数到\[EDX\].Encrypt 的最后一个字节  
  
       .ENDIF         
   
       ; update variables/pointers  
       MOV  EAX, \[EDX\].dwSize  
       SUB  dwSize, EAX            ;重新计算 dwSize 大小  
         
       ;-> decryption buffer  
       MOV  ECX, \[EDX\].dwSize  
       MOV  EAX, \[EDX\].dwDecrypt         
       SUB  ECX, 1  
       .WHILE ECX != 0            ;使 EAX 指向\[EDI\].dwDecrypt 末尾  
            ROR  EAX, 8  
            DEC  ECX  
       .ENDW  
  
       MOV  ECX, \[EDX\].dwSize  
       .WHILE ECX != 0  
           SUB  ESI, 1             ;ESI 指向 pDecryptBuff 的首地址  
           MOV  BYTE PTR \[ESI\], AL        ;填充 AL 到 pDEcryptBuff  
           ROL  EAX, 8            ;这里用左移 EAX 取得下一字节  
           DEC  ECX  
       .ENDW  
         
       ; generate the random num  
       MOV  EAX, \[EDX\].RandNumType  
       .IF  EAX == 1  
           MOV  EBX, ESI  
           ADD  EBX, 2        ;加 2 指向最后一个字节  
           MOV  EAX, dwCurRandNum      
           MOV  BYTE PTR \[EBX\], AL        ;填充随机数到最后一个字节  
             
       .ELSEIF EAX == 2  
           MOV  EBX, ESI  
           ADD  EBX, 1            ;加 1 指向最后一个字节  
           MOV  EAX, dwCurRandNum  
           MOV  BYTE PTR \[EBX\], AL            ;填充随机数到最后一个字节  
                
       .ENDIF         
         
       ASSUME EDX : NOTHING  
         
       Retry:  
    .UNTIL dwSize == 0                ;循环  
    RET  
MakePER ENDP  
  
  
;随机数产生函数\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
  
rand PROC USES edx ebx, dwRange : DWORD  
    MOV  EAX, dwRandVal  
      
    ; save new random number  
    ADD  EAX, 0567h  
    ROL  EAX, 1  
    MOV  dwRandVal, EAX  
      
    ; get new random number  
    XOR  EDX, EDX  
    MOV  ECX, 32  
    BitLoop:  
        SHR   EAX, 1  
        .IF   CARRY?  
            XOR  EAX, 013245769h  
        .ENDIF  
    LOOP BitLoop  
      
    ; force dwRange  
    XOR  EDX, EDX  
    MOV  EBX, dwRange  
    DIV  EBX  
      
    XCHG EAX, EDX  
  
    RET  
rand ENDP  
  
;随机种子函数\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
  
srand PROC dwRandNum : DWORD  
    MOV  EAX, dwRandNum  
    MOV  dwRandVal, EAX  
    RET  
srand ENDP  
  
  
  
  
  
**yoda' cryptor 分析总结**  
  
这个壳机基本上实现了加密壳要实现的东西。总结其特点如下  
1.用到了多态(作者说是多态)，动态的生成加解密函数  
2.API 重定向，不过也仅仅是重定向。在重定向到的代码中直接跳到原系统函数了。很容易还原  
3.兼容性比较好，各种可能性都考虑到了  
4.我分析的这个版本没有处理重定位,不能对 DLL 加壳  
5.作了一些手脚来反调试器和防止脱壳，例如调用 IsDebuggerPresent.壳代码执行完后擦除 PE 头。  
  修改 LDR\_MODULE 的进程映像大小来 anti dump。用 SEH 异常来跳到 OEP，加花指令等等  
  
  
  
最后我注后的这些文件不能编译成功，老提示 windows.inc 里面有错误。可能是我分析 的时候不小心修改了哪 里造成的错误。我也懒得找了。谁找到的可以告诉我<0\_O>，编译可以直接下载原来没有注释的文件来编译。    
  
                  By Demoscene  
                  2010.05.18

上传的附件
| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) || yoda's crypt 1.2 源码.rar  
 |