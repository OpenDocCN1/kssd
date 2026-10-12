
# 第一次尝试逆向分析，这是一个游戏里的部分函数.用来实现用户结构体不停的变换防止外挂.

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-676.htm`](https://www.pediy.com/kssd/pediy09/pediy09-676.htm)

```asm
:00436100 56               push esi               ;esi 入栈,因为 esi 要保存到后面要使用
:00436101 8BF1             mov esi, ecx           ;esi=ecx
:00436103 8B4E08           mov ecx, dword ptr [esi+08];ecx=内存 esi+08 处
:00436106 85C9             test ecx, ecx          ;测试 ecx
:00436108 7419             je 00436123            ;为 0 则跳转到 00436123 处
:0043610A E821FCFEFF       call 00425D30          ;调用 425d30 处函数
:0043610F 8B4E08           mov ecx, dword ptr [esi+08];ecx=esi+8
:00436112 85C9             test ecx, ecx          ;测试 ecx
:00436114 7406             je 0043611C            ;为 0 跳转
:00436116 8B01             mov eax, dword ptr [ecx]   ;ecx=内存 ecx 的值
:00436118 6A01             push 00000001          ;压栈(传一个参数)
:0043611A FF10             call dword ptr [eax]    ;调用函数,函数地址在内存 eax 处

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00436114(C)
|

:0043611C C7460800000000     mov [esi+08], 00000000    ;内存 esi+08=0

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00436108(C)
|

:00436123 8B0D34E09000       mov ecx, dword ptr [0090E034];ecx=内存 90e034 的值
:00436129 5E                 pop esi         ;esi 出栈
:0043612A E931371F00         jmp 00629860           ;跳转到 629860
:0043612F 90                 nop

* Referenced by a CALL at Address:
|:00435435
|

:00436130 6AFF               push FFFFFFFF      ;FFFFFFFF 入栈
:00436132 684B758200         push 0082754B      ;入栈
:00436137 64A100000000       mov eax, dword ptr fs:[00000000];eax=SEH 链指针(NT_TIB 结构第一项指向的 EXCEPTION_REGISRATION 结构的 Prev(下一个结构的地址)
:0043613D 50                 push eax      ;SEH 地址入栈
:0043613E 64892500000000     mov dword ptr fs:[00000000], esp;设置函数的当前的 SEH 链,链地址是 82754b
:00436145 51                 push ecx      ;ecx 入栈
:00436146 56                 push esi      ;esi 入栈
:00436147 8BF1               mov esi, ecx      ;esi=ecx
:00436149 57                 push edi      ;edi 入栈
:0043614A 68440B0000         push 00000B44      ;b44 入栈
:0043614F 8B4608             mov eax, dword ptr [esi+08]  ;eax=esi+08
:00436152 8B781C             mov edi, dword ptr [eax+1C]  ;edi=eax+1c
:00436155 E816702B00         call 006ED170      ;函数 006ed170
:0043615A 83C404             add esp, 00000004    ;丢弃一个堆栈值
:0043615D 89442408           mov dword ptr [esp+08], eax  ;esp+8=eax
:00436161 85C0               test eax, eax      ;测试 eax 是否为 0
:00436163 C744241400000000   mov [esp+14], 00000000  ;esp+14=0
:0043616B 740A               je 00436177      ;根据上面 eax 的测试结果来跳转
:0043616D 57                 push edi      ;edi 入栈
:0043616E 8BC8               mov ecx, eax      ;ecx=eax
:00436170 E8FB700200         call 0045D270      ;函数 45d270
:00436175 EB02               jmp 00436179      ;跳转到 436179

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0043616B(C)
|

:00436177 33C0               xor eax, eax      ;根据上面的 eax 为 0 跳转到这 eax 再次清 0(其实这句可要可不要,因为 eax 已经是 0 了,可见程序员写的代码

还是有点重复)

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00436175(U)
|

:00436179 894624             mov dword ptr [esi+24], eax  ;关键地方:(这就是血的基址说明有以下 2 点 : 1.当 eax 为 0 时,就表示用户正在登录游戏，或者在选择人物

的时候. 用户的数据结构体当时还没有取到所以为 0   2.就是用户已经正常登录了,用来不停的更新用户数据，比如当前血量、蓝值等.esi 为基址再加上偏移 24 就查找到了内存某一块地方 x, 其实 x 就是一个结构 定义了人物的各种属性 结构+偏移 25ch 就得到了当前用户结构，如:当前多少血量,反过来想的话就是用程序到此处去取结构体的地址，结构体的地址再加偏移就是具体的用户值

)
:0043617C 5F                 pop edi      ;edi 出栈
:0043617D 85C0               test eax, eax      ;再次测试 eax 是否为 0
:0043617F C7442410FFFFFFFF   mov [esp+10], FFFFFFFF  ;esp+10=ffffffff
:00436187 5E                 pop esi      ;esi 出栈
:00436188 7525               jne 004361AF      ;如果 eax 不为 0 就跳转,如果 eax 为 0 就从下面去创建用户信息，,不为 0 就不需要创建了直接跳到 4361af 处
:0043618A 68AC030000         push 000003AC      ;入栈

* Possible StringData Ref from Data Obj ->"CECGameRun::CreateHostPlayer"
                          |
:0043618F 6820648C00         push 008C6420      ;入栈
:00436194 6A02               push 00000002      ;入栈
:00436196 E8A5A90000         call 00440B40      ;函数
:0043619B 83C40C             add esp, 0000000C    ;丢弃一个 4 字节栈
:0043619E 32C0               xor al, al      ;al=0 ,这个值是用作这个函数的返回值,值上级函数查询使用
:004361A0 8B4C2404           mov ecx, dword ptr [esp+04]  ;ecx=esp+4
:004361A4 64890D00000000     mov dword ptr fs:[00000000], ecx;撒消 SEH 链
:004361AB 83C410             add esp, 00000010    ;还给堆栈 4 个字
:004361AE C3                 ret        ;函数返回

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00436188(C)
|

:004361AF 8B4C2404           mov ecx, dword ptr [esp+04]  ;这里是用户已经登录的返回
:004361B3 B001               mov al, 01      ;al=1,这个值是用作这个函数的返回值,值上级函数查询使用
:004361B5 64890D00000000     mov dword ptr fs:[00000000], ecx;撒消 SEH 链
:004361BC 83C410             add esp, 00000010    ;还给堆栈 4 个字
:004361BF C3                 ret        ;函数返回

下面代码实现读取用户当前血量，重启游戏后还是能再次读取到。
```

代码:

***



```asm
        .386
        .model  flat,stdcall
        option  casemap:none
   include  windows.inc
   include  user32.inc
   include  kernel32.inc
   includelib  user32.lib
   includelib  kernel32.lib
   include  hook.inc
   includelib  hook.lib
   IDD_DIALOG1        equ               101
   IDC_EDIT1          equ               1000
   IDC_EDIT2          equ               1001
   IDC_EDIT3          equ               1002
   IDC_EDIT4          equ               1003
   IDC_BUTTON1        equ               1004
   IDC_STATIC         equ               -1
   IDT_TIME         equ           1
   ELE_RAW    EQU  02E0A424h
   ELE_HP    EQU  06E68270H
   ELE_MP    EQU  025CH
   ELE_MAXHP  EQU  06E68288H
   ELE_MAXMP  EQU  06E68284H

        .data?
   hInstance  dword  ?
   hProcess    dword  ?;打开的进程
   stMsg       MSG  <?>;消息结构
   dwRaw       dword  ?
   dwHp        dword  ?
   dwMp        dword  ?
   dwMaxHp     dword  ?
   dwMaxMp     dword  ?

        .data
   szErr       byte  '游戏并没有启动,请先启动游戏!',0
   szErr2      byte  '权限不够，打开进程失败!',0
   szTitle     byte  'QElementClient Window',0

        .code
   _DiaProc    proc  uses  ebx esi edi hWnd,uMsg,wParam,lParam

        local    @ProcessId
        mov    eax,uMsg
        .if    eax==WM_COMMAND
              mov    eax,wParam
              .if    ax==IDC_BUTTON1
                    invoke  _msg
              .endif
        .elseif  eax==WM_TIMER
              invoke  ReadProcessMemory,hProcess,ELE_RAW,addr dwRaw,4,NULL
              add    dwRaw,ELE_MP
               invoke  ReadProcessMemory,hProcess,dwRaw,addr dwMp,4,NULL
               invoke  SetDlgItemInt,hWnd,IDC_EDIT1,dwMp,FALSE

        .elseif  eax==WM_CLOSE
              invoke  UninstallHook
              invoke  DestroyWindow,hWnd
              invoke  PostQuitMessage,-1
        .elseif  eax==WM_INITDIALOG
              invoke  SetTimer,hWnd,IDT_TIME,1000,NULL
              invoke  FindWindow,addr szTitle,NULL
              mov    ebx,eax
              invoke  GetWindowThreadProcessId,ebx,addr @ProcessId
              invoke  OpenProcess,PROCESS_VM_OPERATION OR PROCESS_VM_READ OR PROCESS_VM_WRITE OR PROCESS_CREATE_THREAD,NULL,@ProcessId
              .if    eax
                    mov    hProcess,eax
              .else
                    invoke  MessageBox,NULL,addr szErr2,0,MB_OK
               .endif
              invoke  InstallHook,hWnd
        .else
              mov  eax,FALSE
              RET
        .endif
        mov    eax,TRUE
        RET

   _DiaProc    endp
   start:
        invoke  GetModuleHandle,NULL
        mov    hInstance,eax
        invoke  FindWindow,addr szTitle,NULL
         .if    eax
               invoke  CreateDialogParam,hInstance,IDD_DIALOG1,eax,addr _DiaProc,NULL
              invoke  ShowWindow,eax,SW_SHOW
              .while  TRUE
                    invoke  GetMessage,addr stMsg,NULL,0,0
                    .break  .if  !eax
                    invoke  TranslateMessage,addr stMsg
                    invoke  DispatchMessage,addr stMsg
              .endw
        .else
              invoke  MessageBox,NULL,addr szErr,0,MB_OK
        .endif
        invoke  ExitProcess,-1
        end    start
```

第一次做这样的工作，如有任何不足之处请见谅.（本章只是学习，就不具体指出是哪个游戏）

`4dd466d0e1e0cc123afbe504da2462ca`
