
# 病毒技术之搜索（一）

> 原文：[`www.pediy.com/kssd/pediy11/113310.html`](https://www.pediy.com/kssd/pediy11/113310.html)

```asm
;这个没意思，先热热身，主要是看下一个。

;made by correy

;Email:leguanyuan@163.com

;QQ:112426112

;rc me.rc

;ml /coff test.asm /link /subsystem:windows me.res

.386

.model flat,stdcall

option casemap:none

include windows.inc

include kernel32.inc

include user32.inc

includelib user32.lib

includelib kernel32.lib

.data

correy db "made by correy",0

filename db "\*",0

found WIN32_FIND_DATA <>

d db "c:",92,0

.data?

buffer db 256 dup(?)

hfindfile dd ?

.code

searchfile proc lpszPath

local stFindFile:WIN32_FIND_DATA

local hFindFile

local szPath[260]:byte

local szSearch[260]:byte

local szFindFile[260]:byte

pushad

invoke lstrcpy,addr szPath,lpszPath

invoke lstrlen,addr szPath

lea esi,szPath

add esi,eax

cmp byte ptr [esi-1],92

je next

mov word ptr [esi],0092

next:

invoke lstrcpy,addr szSearch,addr szPath

invoke lstrlen,addr szSearch

lea esi,szSearch

add esi,eax

mov word ptr [esi],0042

invoke FindFirstFile,addr szSearch,addr stFindFile

cmp eax,0

je exit

mov hFindFile,eax

begin:

invoke lstrcpy,addr szFindFile,addr szPath

invoke lstrcat,addr szFindFile,addr stFindFile.cFileName

test stFindFile.dwFileAttributes,10h ;test dword ptr stFindFile,10h

je processfile

cmp stFindFile.cFileName,46

je findfile

invoke searchfile,addr szFindFile

jmp findfile

processfile:

invoke MessageBox,0,addr szFindFile,addr correy,0

invoke MessageBox,0,addr stFindFile.cFileName,addr correy,0

findfile:

invoke FindNextFile,hFindFile,addr stFindFile

cmp eax,0

jne begin

invoke FindClose,hFindFile

exit:

popad

ret

searchfile endp

start:

invoke searchfile,addr d

invoke FindFirstFile,addr filename,addr found

.if eax != INVALID_HANDLE_VALUE

mov hfindfile,eax

begin:

mov eax,offset found

add eax,sizeof WIN32_FIND_DATA

sub eax,274

invoke MessageBox,0,eax,addr correy,0

invoke FindNextFile,hfindfile,addr found

cmp eax,0

jne begin

invoke FindClose,hfindfile

.endif

ret

end start
```
