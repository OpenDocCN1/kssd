
# [原创]游戏修改的常用方法之三——内存补丁(asm源码详注)

本程序在WinXP/SP2、ra2之1.006英文版（有中国超牛机器人的那个）运行通过且稳定无误。

```asm
;----------------------------------------------------------------------------------------
;用打补丁的方法改游戏，必需先跟踪游戏程序，找到金钱储存地址，然后直接改代码或打内存补丁
;----------------------------------------------------------------------------------------
.586
.model flat,stdcall
option casemap:none

   include windows.inc
   include kernel32.inc
   include user32.inc
   include psapi.inc

   includelib kernel32.lib
   includelib user32.lib
   includelib psapi.lib

ADD_DATA    equ 0830FFCh
ADD_CODE1    equ 0494A81h
ADD_CODE2    equ 049466Bh

.code

_GetProcessHandle proc
;comment \*在调试时用这段代码取得游戏进程的句柄为好
    LOCAL  info:PROCESSENTRY32
    LOCAL  handle:HANDLE

    invoke CreateToolhelp32Snapshot,TH32CS_SNAPPROCESS,0 ;进程快照
    mov    handle,eax
    mov    info.dwSize,sizeof PROCESSENTRY32
    invoke Process32First,handle,addr info

    .repeat
        mov   eax,@F
        invoke lstrcmpi,addr info.szExeFile,eax ;比较是否为我们要找的进程名，不区分大小写
        .if !eax
            invoke CloseHandle,handle
            ;invoke MessageBox,NULL,addr info.szExeFile,NULL,MB_OK
            invoke OpenProcess,PROCESS_ALL_ACCESS,FALSE,info.th32ProcessID
            jmp EXIT
        .endif
        invoke Process32Next,handle,addr info
    .until !eax

    invoke CloseHandle,handle
    xor eax,eax
EXIT:
    ret
@@:
    db "Game.exe",0
;    \*通常情况下也可以用下面的方法取得游戏进程的句柄，但要注意……
;    LOCAL ProcessId
;    invoke GetForegroundWindow ;你必须确保当前窗口为游戏界面窗口,这样才能正确取得游戏进程ID
;    lea    edx,ProcessId
;    invoke GetWindowThreadProcessId,eax,edx
;    invoke OpenProcess,PROCESS_ALL_ACCESS,FALSE,ProcessId
;    ret
_GetProcessHandle endp

;-----------------------------------------------------------------------
;之所以这么做，是为了给出别人一个未作弊的假象，如果用补丁代码直接改钱数，
;显示的钱数不会变，很容易被人发现你作弊了
;-----------------------------------------------------------------------
_WriteProcessData proc uses esi hProcess
    LOCAL DATA,IsRun,add2data

    mov    esi,2000 ;要锁定的金钱数，别太多，多了是会招贼来偷的:)
    mov    DATA,esi

    .repeat
        .if esi>=5h ;每五循环锁定一次
            invoke ReadProcessMemory,hProcess,ADD_DATA,addr add2data,sizeof add2data,NULL
            add add2data,228h;游戏中真正的保存金钱数的地址
            invoke WriteProcessMemory,hProcess,add2data,addr DATA,sizeof DATA,NULL ;写入钱数
            xor    esi,esi ;循环次数清0
        .endif

        invoke Sleep,1000 ;定时一秒
        inc    esi ;循环次数加一
        invoke GetExitCodeProcess,hProcess,addr IsRun ;游戏程序还在运行吗?
        invoke GetAsyncKeyState,VK_SUBTRACT ;按了“-”键吗?
    .until eax || IsRun!=STILL_ACTIVE ;如果游戏退出或按了“-”键则结束循环

    ret
_WriteProcessData endp

;---------------------
;写入补丁代码，共两处
;---------------------
_WritePathCode proc hProcess
    jmp @F

WRITE_CODE1_START:    ;补丁代码一，含义见下面
    db 0A3h,0FCh,0Fh,83h,00h,0FFh,52h,18h,33h,0C9h,0E9h,0E0h,0FBh,0FFh,0FFh
WRITE_CODE1_END equ this byte

;00494A81      A3 FC0F8300   mov     dword ptr [00830FFC], eax    ;补丁一源码
;00494A86      FF52 18       call    dword ptr [edx+18]
;00494A89      33C9          xor     ecx, ecx
;00494A8B    ^ E9 E0FBFFFF   jmp     00494670

WRITE_CODE2_START:    ;补丁代码二，含义见下面
    db 0E9h,11h,04h,00h,00h
WRITE_CODE2_END equ this byte

;0049466B     E9 11040000   jmp     00494A81    ;补丁二源码

@@:
    mov eax,WRITE_CODE1_START
    invoke WriteProcessMemory,hProcess,ADD_CODE1,eax,WRITE_CODE1_END-WRITE_CODE1_START,NULL    ;写入补丁一
    mov eax,WRITE_CODE2_START
    invoke WriteProcessMemory,hProcess,ADD_CODE2,eax,WRITE_CODE2_END-WRITE_CODE2_START,NULL    ;写入补丁二
    ret
_WritePathCode endp

;------------------------------------------------------------------------
;把本程序拷入游戏文件夹，运行本程序，游戏被启动，本程序在后台运行，无界面
;本程序运行后启动“ra2.exe”，再由“ra2.exe”启动游戏程序，之后“ra2.exe”
;无用了，停掉它以节约内存。
;------------------------------------------------------------------------
_StartGame proc
    LOCAL StartInfo:STARTUPINFO
    LOCAL PI:PROCESS_INFORMATION
    LOCAL GamehProcess

    invoke RtlZeroMemory,addr StartInfo,sizeof STARTUPINFO
    mov StartInfo.cb,sizeof STARTUPINFO

    mov ecx,GAME_NAME
    xor edx,edx
    invoke CreateProcess,edx,ecx,edx,edx,edx,edx,edx,edx,addr StartInfo,addr PI ;启动游戏

    .repeat
        invoke Sleep,100 ;等10秒，游戏程序应该启动了吧?
        invoke _GetProcessHandle
    .until eax

    mov GamehProcess,eax
    invoke _WritePathCode,eax    ;打补丁
    invoke Sleep,9000
    invoke TerminateProcess,PI.hProcess,0h ;没用了，停掉它
    invoke CloseHandle,PI.hProcess ;释放内存
    mov eax,GamehProcess
    ret

GAME_NAME:
    db "ra2.exe",0
_StartGame endp

start:
MAIN    proc
    LOCAL msg:MSG
    LOCAL hProcess,MutexName

    invoke _StartGame ;启动游戏
    mov hProcess,eax

    mov eax,"2ar"
    mov MutexName,eax
    invoke CreateMutex,NULL,TRUE,addr MutexName
    invoke GetLastError
    .if eax!=ERROR_ALREADY_EXISTS ;只让本程序的一个实例运行
        invoke RegisterHotKey,NULL,VK_MULTIPLY,0h,VK_MULTIPLY ;注册热键“*”
        invoke RegisterHotKey,NULL,'X',MOD_CONTROL OR MOD_ALT,'X' ;注册热键“Ctrl+Alt+X”
        .while 1
            invoke GetMessage,addr msg,NULL,NULL,NULL ;等待消息
            .if msg.message==WM_HOTKEY ;热键消息
                .break .if msg.wParam=='X' ;按了热键“Ctrl+Alt+X”则退出本程序
                .if msg.wParam==VK_MULTIPLY ;按了热键“*”
                    invoke MessageBeep,MB_OK ;发声提示游戏者，按键收到，该做的本程序都做过了
                    invoke SetCursorPos,9h,9h ;把鼠标置于屏幕左上角，提醒游戏者，第一遍搜索时特别有用
                    invoke GetCurrentProcess
                    invoke EmptyWorkingSet,eax ;减少自己的内存占用量，不和游戏争内存
                    invoke _WriteProcessData,hProcess;去锁定它!
                    .break .if eax==0 ;如果你没按“-”键，那一定是游戏退出了，咱也退出吧
                .endif
            .endif
        .endw

        invoke UnregisterHotKey,NULL,VK_MULTIPLY ;以下为退出前的清理工作
        invoke UnregisterHotKey,NULL,'X'
        invoke CloseHandle,hProcess
        invoke ExitProcess,NULL
    .endif
    ret
MAIN    endp

end start
```

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)
