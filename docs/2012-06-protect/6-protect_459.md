
# SuperBpm

> 原文：[`www.pediy.com/kssd/pediy05/pediy50007.htm`](https://www.pediy.com/kssd/pediy05/pediy50007.htm)

```c
comment /*
    你知道 SuperBpm 的 erase 和 monitor 的区别吗?如果不知道,可以往下看.
    
    最近看了一点老掉牙的 VxD,才看明白 superBpm 的工作原理,我用 masm 改写了一下,未经充分测试,只是比 EliCZ 这 win32 ASM Pioneer 更烦琐了写可能好理解点,有兴趣的看看.Loader 部分十分简单,主要是根据用户选择用 deviceIoControl 向驱动 VxD 发出命令,VxD 则执行相应动作.
    
    对于 VxD 部分,则是通过拦截\_VWIN32\_Set\_Thread\_Context 服务(即所谓的 hook)来实现对程序擦除或改变 Drx 寄存器的监视和恢复,Erase 模式仅仅是禁止应用程序用 context 改写 Drx 寄存器,这样 Seh 或者用 SetThreadContext 等把戏就不好用了,因为 VxD 工作在最地层,这种拦截是有效的.Monitor 模式是为了阻止那些比如直接修改中断等进入 ring0 修改寄存器的把戏的(我想是这样的...可能错误).
    
    但是这个版本明显对 Cr4 没有进行处理,因为 BPIO 指令是否有效与 CR4 的 DE 位有关,如果你聪明应该知道如何利用这一点.而且在 monitor 模式下对 Dr7 为 0 才恢复这一判断也是有问题的,等你的改进了.EliCZ 源程序中用到的从 VWIN32 服务函数表来直接查找\_VWIN32\_Set\_Thread\_Context 地址的方法我也没有找到详细资料,反正 VxD 快死了,我想知道也就足够了.
    
    VxD 部分我改写了一下,没有处理 TERMINATE\_THREAD 消息,但有问题,如果不怕死机,可以试试.仅供理解关键点参考.
    
    错误难免,如有发现,请指出:QQ:8709369(经常在线)
    
*/
```

### `Rc` 文件

```asm
;===============================Rc 文件

#include <d:\\masm32\\include\\resource.h>

100 DIALOG DISCARDABLE  0, 0, 113, 17

STYLE DS\_MODALFRAME | WS\_POPUP | WS\_CAPTION | WS\_SYSMENU |DS\_CENTER

CAPTION "SuperBPM,Rewrite in Masm32,hume"

FONT 9, "System"

BEGIN

    CONTROL        "擦除恢复",101,"Button",BS\_AUTORADIOBUTTON | WS\_GROUP,
                   2,3,45,10

    CONTROL        "监视恢复",102,"Button",BS\_AUTORADIOBUTTON,58,3,50,10

END
```

### `superBpm.exe.asm`

```asm
;===============================superBpm.exe.asm

.586

.model flat, stdcall

option casemap :none  ; case sensitive

include c:\\hd\\hd.h

include c:\\hd\\mac.h

DlgP             proto :DWORD,:DWORD,:DWORD,:DWORD

CallDriver         proto :DWORD

;;--------------

    .DATA

DrxUnHook  equ    800h

DrxHook    equ    801h

DrxMonitor equ    802h

SuperbpmDevice  db "\\\\.\\superbpm.vxd",0

Traceflg        dd 0

        .DATA?

hVxD    dd ?

hResume dd ?

hStop  dd ?

;;-----------------------------------------

    .CODE

\_\_Start:

        hTemplate      EQU 0

    invoke    CreateFile,addr SuperbpmDevice,GENERIC\_READ+GENERIC\_WRITE,\

                     FILE\_SHARE\_READ+FILE\_SHARE\_WRITE,\

                     0,\

                     OPEN\_EXISTING,FILE\_FLAG\_DELETE\_ON\_CLOSE,\

                     hTemplate

        inc    eax

        jne    CrSuccess

                                ;打开驱动(.vxd)失败,出错 

        invoke    MessageBox,0,CTEXT("Can't Load superbpm.vxd"),CTEXT("Err report:"),30h+1000h 

        jmp    \_\_Quit

    ;-----------------------------------------

CrSuccess:

        dec    eax

        mov    hVxD,eax             ;保存句柄

        invoke    GetModuleHandle,0

        invoke    DialogBoxParam,eax,100,0,offset DlgP,0

        invoke    CloseHandle,hVxD      ;要关闭

        ;----------------------------------------- 

\_\_Quit:

        invoke    ExitProcess,0

;################################################################################

DlgP    proc uses esi edi ebx hWnd:DWORD,wMsg:DWORD,wParam:DWORD,lParam:DWORD

        mov    eax,wMsg

        .IF eax==WM\_INITDIALOG

            invoke    GetDlgItem,hWnd,101

            mov    hResume,eax

            invoke    SendMessage,eax,BM\_SETCHECK,BST\_CHECKED,0

            invoke    SendMessage,hResume,WM\_COMMAND,101,0

            invoke    GetDlgItem,hWnd,102

            mov    hStop,eax            

            invoke    SetWindowPos,hWnd,HWND\_TOPMOST,0,0,0,0,SWP\_NOMOVE or SWP\_NOSIZE

                                ;以上是初始化

        .ELSEIF eax==WM\_CLOSE

            invoke    EndDialog,hWnd,0

            invoke    CallDriver,DrxUnHook

        .ELSEIF eax==WM\_COMMAND

            mov    edx,wParam

            .if    dx==101

                test Traceflg,1

                jnz @f

                mov  Traceflg,1

                invoke    CallDriver,DrxHook      ;擦除及恢复 drx 功能打开

            @@: 

            .elseif dx==102

                cmp  Traceflg,0

                je  @f

                mov  Traceflg,0                

                invoke    CallDriver,DrxMonitor    ;监视及恢复 Drx 功能 

            @@: 

            .endif

        .ELSE

            sub    eax,eax

            ret

        .ENDIF

        mov    eax,1

    ret

DlgP    endp

;;------------------------------------------------_

CallDriver PROC DrvFunc:DWORD        ;DeviceIoControl 包装

        mov    edx,DrvFunc

        sub    eax,eax

        invoke    DeviceIoControl, hVxD, edx, eax, eax, eax,eax,eax,eax

        ret

CallDriver ENDP

END    \_\_Start
```

### `superbpm.vxd.asm`

```asm
;==========================superbpm.vxd.asm 仅供参考,运行死机别找我

.586p

include C:\\98DDK\\inc\\win98\\vmm.inc

include C:\\98DDK\\inc\\win98\\vwin32.inc

DrxUnHook  equ    800h

DrxErase  equ    801h

DrxMonitor equ    802h

CmdUnhook EQU 0

CmdErase  EQU 1

CmdMonit  EQU 2

;------------------------------------

VxD\_LOCKED\_DATA\_SEG

        orgEntry      dd 0

        orgSTC        dd 0

        TheThread    DWORD 0

        SavedDRx    DWORD 6 DUP (?)

        PreserveMode BYTE CmdErase

        Saved        BYTE 0

        installed      db 0

VxD\_LOCKED\_DATA\_ENDS

VxD\_LOCKED\_CODE\_SEG

DECLARE\_VIRTUAL\_DEVICE SUPERBPM,1,0, MESSAGE\_Control, UNDEFINED\_DEVICE\_ID, UNDEFINED\_INIT\_ORDER

Begin\_Control\_Dispatch MESSAGE

    Control\_Dispatch W32\_DEVICEIOCONTROL , OnDeviceIoControl    ;处理相应发来的消息

End\_Control\_Dispatch MESSAGE

        ;-----------------------------------------DeviceIoControl 

        BeginProc      OnDeviceIoControl

            Assume esi:ptr DIOCParams 

                .IF \[esi\].dwIoControlCode==DIOC\_OPEN

                    sub  eax,eax

                    clc

                .ELSEIF \[esi\].dwIoControlCode==DrxUnHook      ;退出卸载恢复原来的入口

                     mov ecx,orgEntry 

                     mov dword ptr \[ecx\],OFFSET32 HookedSTC

                @@:

                    and  TheThread,0

                    and  Saved,0

                    sub  eax,eax

                    clc

                .ELSEIF \[esi\].dwIoControlCode==DrxErase

                    cmp installed,1

                    je  @f 

                    mov  eax, VWIN32\_DEVICE\_ID

                    VMMCall  Get\_DDB            ;VWIn32 的 DDB

                    mov ecx, \[ecx+38H\]          ;得到引出服务表的地址

                    add ecx, 8+8\*15H            ;第 16 项?我没找到详细资料,但确实是

                    cmp byte  ptr \[ecx+4\],2    ;后面存储的参数个数为 2?

                    jnz @f

                    mov eax,\[ecx\]

                    mov orgSTC,eax

                    mov dword ptr \[ecx\],OFFSET32 HookedSTC

                    mov orgEntry,ecx             ;替换入口,实现 hook 功能

                    mov installed,1

                @@:

                mov PreserveMode, CmdErase ;模式是 CmdErase

                ;-----------------------------------------

                    xor  eax,eax

                    clc

                .ELSEIF \[esi\].dwIoControlCode==DrxMonitor
```



```asm
mov PreserveMode, CmdMonit   

@@:   

xor eax,eax   

clc   

.endif   

ret   

EndProc OnDeviceIoControl   

HookedSTC PROC   

PUSHFD   

PUSH EAX   

PUSH ECX   

PUSH ESI   

PUSH EDI   

MOV ECX, [ESP+5*4 +4 +3*4] ;ecx->CONTEXT   

CMP PreserveMode, CmdErase   

JE HackContext   

@@:   

VMMCall Get_Cur_Thread_Handle ;如果是 Monitor 模式就得到当前有非法企图的 Thread 句柄   

;看看 dr7,dr7 是第 7 项   

CMP BYTE PTR [ECX+4*5+4], 0 ;dr7=0?看看 Seh 资料的 Context 结构   

JE Restore ;就恢复?   

CMP Saved, 1   

JE Done   

MOV TheThread, EDI ;保存句柄   

CLD   

MOV EDI, OFFSET SavedDRx ;保存 Drx   

MOV EAX, DR0   

STOSD   

MOV EAX, DR1   

STOSD   

MOV EAX, DR2   

STOSD   

MOV EAX, DR3   

STOSD   

MOV EAX, DR6   

STOSD   

MOV EAX, DR7   

STOSD   

MOV Saved, 1   

JMP Done   

Restore:   

CMP EDI, TheThread   

JNE Done   

CMP Saved, 1   

JE HackContext   

CLD   

MOV ESI, OFFSET SavedDRx ;恢复 Drx....   

LODSD   

MOV DR0, EAX   

LODSD   

MOV DR1, EAX   

LODSD   

MOV DR2, EAX   

LODSD   

MOV DR3, EAX   

LODSD   

MOV DR6, EAX   

LODSD   

MOV DR7, EAX   

MOV Saved, 1   

HackContext:   

AND BYTE PTR [ECX], NOT 10H ;这个标志是不更新除 Drx 系列寄存器外   

;恢复所有的寄存器,以此有些 Seh 把戏就无效了   

Done:   

POP EDI   

POP ESI   

POP ECX   

POP EAX   

POPFD   

JMP orgSTC   

HookedSTC ENDP   

VxD_LOCKED_CODE_ENDS   

end   

;-----------------Over-------------------   

; Hume 2002.3  4dd466d0e1e0cc123afbe504da2462ca
```
