
# arm1.01

> 原文：[`www.pediy.com/kssd/pediy03/forum834.htm`](https://www.pediy.com/kssd/pediy03/forum834.htm)

使用 `loader` 方式破解 `arm 1.01`   

作者: `Fpc`   

`arm1.01` 是近期程序保护方式的一个典型例子 (详情请见不久前的贴子)：   

1、使用 `Aspack`(或 `Asprotect`)以及其它压缩软件为文件加壳；   

2、加密算法不可逆（昨天看了 `MD5` 的算法，应该就是这个），只有暴破一条路。只是现在我还不知道有没有可能通过逐渐逼近的方式得到注册码；   

3、脱壳比较困难，对我即如此（虽然 `Arm` 可以被 `Carspr` 脱壳）。   

这样我们就面对比较困难的选择：要么手动，要么等待解压软件。但这两者都是解决问题的正确途径。那么对菜鸟级选手来说，采用 `Loader` 方式就是另一种选择。   

以 `Arm` 为例，它的注册码计算过程至少被调用三次：启动时的检查、启动时对输入注册码的检查、启动后在‘关于’中输入注册码时的检查。显然在正常情况下，这些检查都是对同一个 `CALL` 的调用。因此要做的就是在外壳程序完成解压后、第一次对注册码的检查之前在内存中对这个 `CALL` 打补丁。这就是 `Loader` 的原理。   

本例的 `Loader` 的原型在 `tKC Tutor #25`，可能是由 `R!SC` 首先公布出来的，我只做了微小的改动：   

```asm
;-=-Larm101.asm-=-=-cut & paste me :)=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-

; based on original loader.exe by Hayras [tNO '98] used in

; Hayras's Neolite v1.22 memory patcher.

; (Loader is also a real mother!!!)

; This is slightly modifide by Fpc @2001.

;

; Special thanks to TNO for graciously allowing me to use this

; source on behalf of Hayras, who has now retired from the scene.

; Yes, Hayras's loader reversed by R!SC, then totally re-written

; & released to the public, so everyone can learn this shit :)

;

; Requires Tasm 5.0 & import32.lib to compile

; make a BatFile as following:

; mk.bat

; <your path>\tasm32 /ml %1.asm

; <your path>\tlink32 /Tpe /aa /c %1,%1,, <your path>\Lib\import32.lib

; [Eof]

; replace <your path> with whatever...

; (c)1999 R!SC (see what i do instead of cracking...duh) yey Prophecy!

.386P

Locals

jumps

.Model Flat ,StdCall

;Define the needed external functions and constants here.

Extrn       MessageBoxA:PROC

Extrn       WaitForInputIdle:PROC

Extrn       WriteProcessMemory:PROC

Extrn       ReadProcessMemory:PROC

Extrn       CreateProcessA:PROC

Extrn       CloseHandle:PROC

Extrn       ExitProcess:PROC

;-=-Normal data-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-

.Data

CSiR_Tag            db ' PE LOADER & CRACKER---R!SC 1999 ',0

CSiR_Error          db 'Error!!!',0

CSiR_Error1         db 'Something fucked up...',0

OpenERR_txt         db 'CreateProcess Error :(',0

ReadERR_txt         db 'ReadProcessMemory Error :(',0

WriteERR_txt        db 'WriteProcessMemory Error :P',0

VersionERR_txt      db 'Incorrect Version of application :(',0

CSiR_ProcessInfo    dd 4 dup (0)        ;process handles

CSiR_StartupInfo    db 48h dup (0)      ;startup info for the process were opening

CSiR_RPBuffer       db 10h dup (0)      ;read buffer, for checking data

;-=-Patch datas-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-

CSiR_AppName  db 'arm.exe',0

fuck          dd 495825h       ; address to read data from for version checking

sizeof        dd 10            ; in the new process

checkbytes    db 089h,045h,0f0h,0c7h,045h  ; the bytes to check for

                db 0f4h,01h,0h,0h,00h  ; if there not there, we have the wrong version??

patch_data_1  db 0c7h,045h,0f4h,01h,0h,0h,0h,08bh

                db 045h,0f4h,08bh,014h,085h,044h,060h,04bh,0h

                db 08bh,0c2h,089h,045h,0f0h,090h

patch_size_1  dd 23

patch_addr_1  dd 495825h

.Code

;-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-

Main:

    push    offset CSiR_Tag

    mov     dword ptr [CSiR_StartupInfo],44h ; (the size in bytes of the structure)

    push    offset CSiR_ProcessInfo           ; Typedef struct _PROCESS_INFORMATION

    push    offset CSiR_StartupInfo           ; Pointer to STARTUPINFO structure

    push    0

    push    0

    push    20h                               ; Creation flags

    push    0

    push    0

    push    0

    push    0

    push    offset CSiR_AppName                 ; Pointer to name of executable mod

    call    CreateProcessA

    test    eax,eax

    jz      OpenERR

Wait4Depack:

    push    800                               ; Timeout (in milliseconds, -1 = infinate)

    push    dword ptr [CSiR_ProcessInfo]      ; Here we take 0.8s

    call    WaitForInputIdle

Check_Data:

    push    0                               ; BytesRead

    push    dword ptr [sizeof]              ; Length

    push    offset CSiR_RPBuffer            ; Destination (to read them to)

    push    dword ptr [fuck]                ; Source

    push    dword ptr [CSiR_ProcessInfo]    ; Process whose memory we are to read

    call    ReadProcessMemory

    test    eax,eax

    jz      ReadERR

    ;...

    ;int 03 ;-)

    cld

    lea     esi, CSiR_RPBuffer

    lea     edi, checkbytes

    mov     ecx, 10

    rep     cmpsb

;    nop

    jnz     VersionERR

    ;...

Patch_the_mother:

    push    0                               ; Pointer to byteswritten (i like null though)

    push    dword ptr [patch_size_1]        ; Length

    push    offset patch_data_1             ; Source

    push    dword ptr [patch_addr_1]        ; Destination

    push    dword ptr [CSiR_ProcessInfo]    ; Process whose memory we are to patch

    call    WriteProcessMemory              ; Call Kernel32!WriteProcessMenory

    test    eax,eax

    jz      WriteERR

Close_This_app:

    push    dword ptr [CSiR_ProcessInfo]

    call    CloseHandle

    push    dword ptr [CSiR_ProcessInfo+4]

    call    CloseHandle

Exit_Proc:

    Push LARGE-1

    Call ExitProcess

;-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-

VersionERR:

    lea     eax, VersionERR_txt

    jmp     abort

ReadERR:

    lea     eax, ReadERR_txt

    jmp     abort

OpenERR:

    lea     eax, OpenERR_txt

    jmp     abort

WriteERR:

    lea     eax, WriteERR_txt

abort:

    push 0

    push offset CSiR_Error                  ; Title

    push eax                                ; Message

    push 0

    call MessageBoxA

    jmp Close_This_app

End Main

;-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-
```

这个程序是有缺点的，即机器的速度如果过快或过慢，运行可能就要出错。自己的学艺实在不精。肯请哪位虾虾出手，在其中加入判断机器速度、根据这个速度来确定打补丁的时间。我这里用的 `800 毫秒`（`0.8 秒`），在 `PII 300` 的条件下正常运行。

最后大家要注意一点，`Loader` 方式回避了对解压这一问题的处理，有点消极，算是没有办法的办法。软件的脱壳技术还是需要掌握的。
