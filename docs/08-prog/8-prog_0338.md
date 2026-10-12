
# 水晶排课，注册插件 ASM 源码

> 原文：[`www.pediy.com/kssd/pediy12/129911.html`](https://www.pediy.com/kssd/pediy12/129911.html)

很久没发过东西，一个很很老的东西，基以看雪 ID：“非安全”的代码修改而来，东西很好，要会利用，所有代码都是调试的结果。

功能是利用 HOOK 进 `msvbvm60.dll` 的函数修改对应的东西，不想说那么多了，高手漂过。

![](img/906f120cfe711ccbc8463fd6eb86eee9.jpg "尴尬")，生成的 DLL，自己找吧。

引用: `.486`

```.486

.model flat,stdcall

option casemap:none

include    windows.inc

include    kernel32.inc

includelib kernel32.lib

include    user32.inc

includelib user32.lib

include     cinvoke.inc

include     msvbvm60.inc

includelib msvbvm60.lib

include     Advapi32.inc

includelib Advapi32.lib

include     SHELL32.inc

includelib SHELL32.lib

include    ucmacros.asm

DLG_MAIN  equ  1000

_PROCVAR0  typedef proto

PROCVAR0  typedef ptr _PROCVAR0

HOOKAPI struct

a1  byte ?

a   byte ?

PMyapi DWORD ?

d byte ?

e byte ?

f byte ?

HOOKAPI ends

;子程序声明

WriteApi proto :DWORD,:DWORD,:DWORD,:DWORD

patch    proto :DWORD,:DWORD,:DWORD,:DWORD

MyAPI1   proto

MyAPI2   proto

GetApi   proto  :DWORD,:DWORD

_SetIDMRGE proto  :DWORD

;已初始化数据

.data

hInstance dd 0

WProcess  dd 0

hacker HOOKAPI <>

CommandLine LPSTR ?

Papi0 DWORD ?

Papi1 DWORD ?

Papi2 DWORD ?

Papi3 DWORD ?

Myapi1 DWORD ?

Papi00 DWORD ?

Papi10 DWORD ?

Papi20 DWORD ?

Papi30 DWORD ?

Myapi10 DWORD ?

My1 DWORD ?

My2 DWORD ?

My3 DWORD ?

My4 DWORD ?

ApiBak1 db 10 dup(?)

ApiBak10 db 10 dup(?)

DllName1  db "msvbvm60.dll",0     ;这是要 HOOK 的 DLL 模块

ApiName1  db "rtcMidBstr",0  ;HOOK 程序调用的第一个 API

ApiName2  db "rtcLeftBstr",0 ;HOOK 程序调用的第二个 API

szCaption  db  'MsageBox',0

szText    db  '我只是想告诉您,HOOK 补丁失败了!点确定后程序运行',0

;IDM 注册表信息

szKeyIDM db 'S-1-5-18\Software\Microsooft\CyPk',0

szKeyIDM1 db '.DEFAULT\Software\Microsooft\CyPk',0

szValueFName  db 'BianHao',0

szValueLName  db 'BianHao_GudinMa',0

szValueEmail  db 'BianHao',0

szValueSerial  db 'BianHao_GudinMa',0

;--------------------------------------------------

szValueFNamesj db '88888888888888888888888888888888',0

szValueLNamesj db '3815711B58558E05C28F761C5F0B1663  366EEB73602B02075B14E01814ED7423',0

szValueEmailsj  db '88888888888888888888888888888888',0

szValueSerialsj db '3815711B58558E05C28F761C5F0B1663  366EEB73602B02075B14E01814ED7423',0

;--------------------------------------------------

ebb          db 0EBh ;JMP

;未初始化数据

.data?

hHook dd ?

hWnd dd ?

hHook1 dd ?

hWnd1 dd ?

;程序代码段

.code

include     _Reg.asm

;*********************************************************************DLL 入口点*********************************************************************

DllEntry proc hInst:HINSTANCE, reason:DWORD, reserved1:DWORD

  .if reason==DLL_PROCESS_ATTACH                   ;当 DLL 加载时产生此事件

        push hInst

        pop hInstance

invoke GetCommandLine

mov CommandLine,eax                                ;取程序命令行

;初始化

mov hacker.a1,050h     ;push eax

mov hacker.a, 0B8h     ;mov eax,

;mov hacker.d PMyapi   ;0x000000

mov hacker.d, 0FFh     ;jmp XXXXXXXX

mov hacker.e, 0E0h     ;eax

mov hacker.f, 090h     ;nop

invoke   GetCurrentProcess                              ;取进程伪句柄

mov WProcess ,eax

invoke GetApi,addr DllName1,addr ApiName1                ;取 API 地址

  mov Papi0,eax ;把地址保存一下

  add eax,1Dh

  mov Papi1,eax                                    ;保存 API 地址

  mov eax,Papi0

  add eax,29h

  mov Papi2,eax

  mov eax,Papi0

  add eax,29h

  mov Papi3,eax

invoke GetApi,addr DllName1,addr ApiName2                ;取 API 地址

  mov Papi00,eax

  mov Papi10,eax                                    ;保存 API 地址

  mov eax,Papi00

  add eax,11h

  mov Papi20,eax

  mov eax,Papi00

  add eax,9h

  mov Papi30,eax

invoke ReadProcessMemory,WProcess,Papi1,addr ApiBak1,8,NULL  ;备份原 API 的前 8 字节

  mov hacker.PMyapi,offset MyAPI1   ;0x000010                    ;要替代 API 的函数地址

  invoke WriteApi,WProcess,Papi1, addr hacker ,size HOOKAPI    ;HOOK API

invoke ReadProcessMemory,WProcess,Papi10,addr ApiBak10,8,NULL  ;备份原 API 的前 8 字节

  mov hacker.PMyapi,offset MyAPI2   ;0x000010                    ;要替代 API 的函数地址

invoke WriteApi,WProcess,Papi10, addr hacker ,size HOOKAPI    ;HOOK API

.endif

.if  reason==DLL_PROCESS_DETACH

invoke WriteApi,WProcess,Papi1, addr ApiBak1 ,8               ;还原 API

invoke WriteApi,WProcess,Papi10, addr ApiBak10 ,8               ;还原 API

.endif

  mov  eax,TRUE

    ret

DllEntry Endp

InstallHook proc

InstallHook endp

UninstallHook proc

UninstallHook endp

;********************************************************************

GetApi proc DllNameAddress:DWORD,ApiNameAddress:DWORD

invoke  GetModuleHandle,DllNameAddress     ;取 DLL 模块句柄

  .if eax==NULL

  invoke LoadLibrary ,DllNameAddress       ;加载 DLL

   .endif

 invoke GetProcAddress,eax,ApiNameAddress  ;取 API 地址

mov eax,eax

ret

GetApi endp

;***********************************下面是 HOOK API 核心部分***********************************

WriteApi proc Process:DWORD ,Papi:DWORD,Ptype:DWORD,Psize:DWORD

LOCAL mbi:MEMORY_BASIC_INFORMATION

LOCAL msize:DWORD

;返回页面虚拟信息

invoke VirtualQueryEx,Process, Papi,addr mbi,SIZEOF MEMORY_BASIC_INFORMATION

;修改为可读写模式

invoke VirtualProtectEx,Process, mbi.BaseAddress,8h,PAGE_EXECUTE_READWRITE,addr mbi.Protect

;开始写内存

invoke  WriteProcessMemory,Process, Papi, Ptype,Psize ,NULL

PUSH eax

;改回只读模式

invoke VirtualProtectEx,Process,mbi.BaseAddress,8h,PAGE_EXECUTE_READ,addr mbi.Protect

pop eax

ret

WriteApi endp

;********************************************************************

;**********补丁不可读写地址特殊处理部分**********

patch proc Process:DWORD ,Papi:DWORD,Ptype:DWORD,Psize:DWORD

LOCAL mbi:MEMORY_BASIC_INFORMATION

LOCAL msize:DWORD

;返回页面虚拟信息

invoke VirtualQueryEx,Process, Papi,addr mbi,SIZEOF MEMORY_BASIC_INFORMATION

;修改为可读写模式

invoke VirtualProtectEx,Process, mbi.BaseAddress,14h,PAGE_EXECUTE_READWRITE,addr mbi.Protect

;开始写内存

invoke  WriteProcessMemory,Process, Papi, Ptype,Psize ,NULL

cmp eax,0

jne @F

invoke WriteApi,WProcess,Papi1, addr ApiBak1 ,8                        ;如果没补丁到,先还原 API

invoke  MessageBox,NULL,offset szText,offset szCaption,MB_ICONSTOP ;提示补丁失败

invoke  ExitProcess,NULL

@@:

;PUSH eax

;改回只读模式

;invoke VirtualProtectEx,Process,mbi.BaseAddress,14h,PAGE_EXECUTE_READ,addr mbi.Protect

;pop eax

ret

patch endp

;********************************************************************

;********替代的 API,参数要和原来一样********

MyAPI1 proc

pop eax  ;恢复 EAX

MOV EAX,DWORD PTR SS:[ESP+10h] ;恢复原来的代码

PUSH EAX  ;保存要破坏的寄存器

PUSH ECX

PUSH EDX

PUSH EBX

MOV ecx,DWORD PTR DS:[EAX]    ;6 位码 1 位和 2 位 U 码

MOV edx,DWORD PTR DS:[EAX+4h] ;6 位码 3 位和 4 位 U 码

MOV ebx,DWORD PTR DS:[EAX+8h] ;6 位码 5 位和 6 位 U 码

MOV AL,BYTE PTR DS:[EAX+0Ch]  ;把第 7 位存入 EAX，这样就能保存断码为 0

CMP AL,0h ;AL 如果为 0，说明 6 位数找到了

NOP ;无用代码

JNZ ok  ;如果是 6 位码则不走了，可以下面保存

call @F

@@:    ;重定代码，获取当入地址

pop eax  ;@@:地址存入 EAX

sub eax,07h ;减 7H 就是 JNE OK 地址

mov My4,eax ;保存下

mov My1,ecx ;保存我位重要的数据

mov My2,edx

mov My3,ebx

mov eax,My4 ;恢复下

invoke patch,WProcess,eax,addr ebb,1h ;修改 JNE OK 为 JMP OK

;mov BYTE PTR DS:[eax],0EBH  ;SMC 相信不过

;--------------------------------------------------------------------

 invoke  _SetIDMRGE,1h ;汗,写下注册表信息  注册表信息不可重复写入
```



`POP EBX` `POP EDX` `POP ECX` `POP EAX`

; `invoke MessageBoxW, NULL, eax, eax, MB_ICONSTOP` ; `invoke ExitProcess, NULL`

`MOV EBX, EDI` `TEST EAX, EAX` `je @F` `MOV ESI, DWORD PTR DS:[EAX-4h]` `jmp DWORD PTR DS:[Papi3]` `@@:` `XOR ESI, ESI` `jmp DWORD PTR DS:[Papi2]`

; -----------------------------DLL 补丁结束--------------------------------------

; `invoke WriteApi, WProcess, Papi1, addr ApiBak1, 8` ; 先还原 API ; `invoke rtcMidBstr` ; `invoke WriteApi, WProcess, Papi1, addr hacker, sizeof HOOKAPI` ; 调用完后再改回来 `MyAPI1 endp`

`MyAPI2 proc` `pop eax` `CMP AH, 0` `JE @F` `CMP DWORD PTR DS:[EAX+4h], 320037h` `JNZ @F` `CMP DWORD PTR DS:[EAX+8h], 300034h` `push ebx` `mov ebx, [My1]` `mov DWORD PTR DS:[EAX], ebx` `mov ebx, [My2]` `mov DWORD PTR DS:[EAX+4h], ebx` `mov ebx, [My3]` `mov DWORD PTR DS:[EAX+8h], ebx` `pop ebx` `@@:` `PUSH ESI` `MOV ESI, DWORD PTR SS:[ESP+0Ch]` `TEST ESI, ESI` `Jl @F` `jmp DWORD PTR DS:[Papi30]` `@@:` `jmp DWORD PTR DS:[Papi20]` `MyAPI2 endp`

; 写注册表模块

`_SetIDMRGE Proc _dwFlag` `.if _dwFlag` `invoke _RegSetValue, addr szKeyIDM, addr szValueFName, addr szValueFNamesj, REG_SZ, 100` `invoke _RegSetValue, addr szKeyIDM, addr szValueLName, addr szValueLNamesj, REG_SZ, 100` `invoke _RegSetValue, addr szKeyIDM1, addr szValueEmail, addr szValueEmailsj, REG_SZ, 100` `invoke _RegSetValue, addr szKeyIDM1, addr szValueSerial, addr szValueSerialsj, REG_SZ, 100` `.else` `invoke _RegDelValue, addr szKeyIDM, addr szValueSerial` `.endif` `ret` `_SetIDMRGE endp`

`End DllEntry`
