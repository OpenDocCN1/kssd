

# 病毒技术之搜索（二）

> 原文：[`www.pediy.com/kssd/pediy11/113312.html`](https://www.pediy.com/kssd/pediy11/113312.html)

;made by correy  
;Email:leguanyuan@163.com  
;QQ:112426112  
;rc me.rc  
;ml /coff test.asm /link /subsystem:windows me.res  
.386  
.model flat,stdcall  
option casemap:none  
include windows.inc  
include kernel32.inc  
include user32.inc  
includelib user32.lib  
includelib kernel32.lib  
  
.data   
correy db "made by correy",0   
d db "c:",92,0  
  
.code  
  
processexefile proc ipszfile  
local ipszext  
local ipbuffer\[260\]:byte  
pushad  
invoke lstrcpy,addr ipbuffer,ipszfile  
;invoke MessageBox,0,addr ipbuffer,addr ipbuffer,0  
invoke lstrlen,addr ipbuffer  
sub eax,3;文件后缀名是 4 个的减去 4  
lea esi,ipbuffer  
add esi,eax  
;invoke MessageBox,0,esi,esi,0  
mov esi,\[esi\]  
cmp esi,"exe";or 657865h,或则"jpeg","aspx","dll"等,注意要反写哟！  
jne next  
invoke MessageBox,0,addr ipbuffer,addr ipbuffer,0  
jmp exit  
  
next:  
cmp esi,"lld"  
jne next1  
invoke MessageBox,0,addr ipbuffer,addr ipbuffer,0  
jmp exit  
  
next1:  
cmp esi,"gpj" ;jpg  
jne next2  
invoke MessageBox,0,addr ipbuffer,addr ipbuffer,0  
jmp exit  
  
next2:  
cmp esi,"cod" ;doc  
jne exit  
invoke MessageBox,0,addr ipbuffer,addr ipbuffer,0  
  
exit:popad  
ret  
processexefile endp  
  
searchfile proc lpszPath  
local stFindFile:WIN32\_FIND\_DATA  
local hFindFile  
local szPath\[260\]:byte  
local szSearch\[260\]:byte  
local szFindFile\[260\]:byte  
pushad  
invoke lstrcpy,addr szPath,lpszPath  
invoke lstrlen,addr szPath  
lea esi,szPath  
add esi,eax  
  
cmp byte ptr \[esi-1\],92  
je next  
mov word ptr \[esi\],0092  
next:  
  
invoke lstrcpy,addr szSearch,addr szPath  
invoke lstrlen,addr szSearch  
lea esi,szSearch  
add esi,eax  
mov word ptr \[esi\],0042  
  
invoke FindFirstFile,addr szSearch,addr stFindFile  
cmp eax,0  
je exit  
mov hFindFile,eax  
  
begin:  
invoke lstrcpy,addr szFindFile,addr szPath  
invoke lstrcat,addr szFindFile,addr stFindFile.cFileName  
  
test stFindFile.dwFileAttributes,10h ;test dword ptr stFindFile,10h  
je processfile  
cmp stFindFile.cFileName,46  
je findfile  
invoke searchfile,addr szFindFile  
jmp findfile  
  
processfile:  
invoke processexefile,addr szFindFile  
;invoke MessageBox,0,addr szFindFile,addr correy,0  
findfile:  
invoke FindNextFile,hFindFile,addr stFindFile  
  
cmp eax,0  
jne begin  
  
invoke FindClose,hFindFile  
exit:  
popad  
ret  
searchfile endp  
  
start:   
push offset d  
call searchfile  
ret  
end start