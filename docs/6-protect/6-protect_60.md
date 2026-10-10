

# 无 PUSH 参数,无 CALL 调用 API 的一个例子

> 原文：[`www.pediy.com/kssd/pediy11/118853.html`](https://www.pediy.com/kssd/pediy11/118853.html)

**先来看看 OD 上的样子**  
  

代码:

```
00401000   $  55            push    ebp
00401001   .  8BEC          mov     ebp, esp
00401003   .  83C4 E4       add     esp, -1C
00401006   .  B9 07000000   mov     ecx, 7
0040100B   .  33C0          xor     eax, eax
0040100D   .  57            push    edi
0040100E   .  8D7D E4       lea     edi, dword ptr \[ebp-1C\]
00401011   .  F3:AB         rep     stos dword ptr es:\[edi\]
00401013   .  5F            pop     edi
00401014   .  C745 E8 00000>mov     dword ptr \[ebp-18\], 0            ;  (initial cpu selection)
0040101B   .  C745 FC 04000>mov     dword ptr \[ebp-4\], 4
00401022   .  C745 F4 00304>mov     dword ptr \[ebp-C\], 00403000      ;  信息框测试!--------123456
00401029   .  C745 F8 1A304>mov     dword ptr \[ebp-8\], 0040301A      ;  信息框标题!
00401030   .  C745 E4 3D104>mov     dword ptr \[ebp-1C\], 0040103D
00401037   .  68 64104000   push    <jmp.&kernel32.GetModuleHandleA>
0040103C   .  C3            retn                                     ;  RET 用作跳转到 00401064
0040103D   .  C745 EC 4A104>mov     dword ptr \[ebp-14\], 0040104A
00401044   .  68 58104000   push    <jmp.&user32.MessageBoxA>
00401049   .  C3            retn
0040104A   .  C9            leave
0040104B   .  C3            retn
0040104C >/$  E8 AFFFFFFF   call    00401000
00401051  |.  6A 00         push    0                                ; /ExitCode = 0
00401053  \\.  E8 06000000   call    <jmp.&kernel32.ExitProcess>      ; \\ExitProcess
00401058   .- FF25 0C204000 jmp     dword ptr \[<&user32.MessageBoxA>>;  user32.MessageBoxA
0040105E   .- FF25 04204000 jmp     dword ptr \[<&kernel32.ExitProces>;  kernel32.ExitProcess
00401064   >- FF25 00204000 jmp     dword ptr \[<&kernel32.GetModuleH>;  kernel32.GetModuleHandleA

```

**然后看看源码:**  
  
代码:

```
    .386
    .model flat, stdcall
    option casemap :none
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
; Include 文件定义
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
include    windows.inc
include    user32.inc
includelib  user32.lib
include    kernel32.inc
includelib  kernel32.lib
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
; 代码段
.data
str1  db '信息框测试!--------123456',0
str2  db '信息框标题!',0
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
    .code
\_NewCallAPI proc  ;此构造缺点是,你不能随便申请一个与 CALL 无关的局部变量
local utype,lpcption,lptext,hwnd,MessageBoxret  ;申请 API 局部变量参数要倒着来,返回地址写最后
local lpModuleHandle,GetModuleHandleret
mov  ecx,7;局部变量个数,必需是 DD 类的
xor  eax,eax
push  edi
lea  edi,GetModuleHandleret
rep  stosd  ;初始化局部变量值
pop  edi
mov  lpModuleHandle,0
  mov  utype,MB\_YESNO
  mov  lptext,offset str1
  mov  lpcption,offset str2
mov  GetModuleHandleret,$+13
push  GetModuleHandle
retn
mov  MessageBoxret,$+13
push  MessageBox
retn
;push  ExitProcess
;retn
leave
retn
\_NewCallAPI endp
start:
invoke \_NewCallAPI
invoke  ExitProcess,NULL
end  start

```

5 年前写过类似的一个，拿出来改了一下：  
   
masm  

代码:

```
 
mycall macro funcname, args:VARARG
    local ofs, retaddr
    sub esp, argcount(args)\*4+8
    ofs = 0
    mov dword ptr \[esp+ofs\], funcname
    ofs = ofs + 4
    mov dword ptr \[esp+ofs\], retaddr
    ofs = ofs + 4
    for arg, <args>
        mov dword ptr \[esp+ofs\], repargof(arg)
        ofs = ofs + 4
    endm
    retn
 
retaddr:
endm
start:
    mycall Sleep, 100
    mycall MessageBoxA, 0, "call w/o pushin' arguments", 0, 0
    mycall ExitProcess, 0
end start

```

  
ollydbg  
  
代码:

```
00401000 >/$  83EC 0C       sub     esp, 0C
00401003  |.  C70424 701040>mov     dword ptr \[esp\], <jmp.&kernel32.>
0040100A  |.  C74424 04 1B1>mov     dword ptr \[esp+4\], mycall.004010>
00401012  |.  C74424 08 640>mov     dword ptr \[esp+8\], 64
0040101A  \\.  C3            retn
0040101B   .  83EC 18       sub     esp, 18
0040101E   .  C70424 761040>mov     dword ptr \[esp\], <jmp.&user32.Me>
00401025   .  C74424 04 4E1>mov     dword ptr \[esp+4\], mycall.004010>
0040102D   .  C74424 08 000>mov     dword ptr \[esp+8\], 0
00401035   .  C74424 0C 003>mov     dword ptr \[esp+C\], mycall.004030>;  ASCII "call w/o pushin' arguments"
0040103D   .  C74424 10 000>mov     dword ptr \[esp+10\], 0
00401045   .  C74424 14 000>mov     dword ptr \[esp+14\], 0
0040104D   .  C3            retn
0040104E   .  83EC 0C       sub     esp, 0C
00401051   .  C70424 6A1040>mov     dword ptr \[esp\], <jmp.&kernel32.>
00401058   .  C74424 04 691>mov     dword ptr \[esp+4\], mycall.004010>
00401060   .  C74424 08 000>mov     dword ptr \[esp+8\], 0
00401068   .  C3            retn
00401069      CC            int3
0040106A   .- FF25 04204000 jmp     \[<&kernel32.ExitProcess>\]        ;  kernel32.ExitProcess
00401070   .- FF25 00204000 jmp     \[<&kernel32.Sleep>\]              ;  kernel32.Sleep
00401076   .- FF25 0C204000 jmp     \[<&user32.MessageBoxA>\]          ;  user32.MessageBoxA
 

```