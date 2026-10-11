
# 利用进程空闲为壳软件打补丁 3 例

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-440.htm`](https://www.pediy.com/kssd/pediy08/pediy8-440.htm)

## 一、利用 Sleep 延迟进程启动

```asm
.586
.model flat, stdcall
option casemap :none
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
include windows.inc
include kernel32.inc
include user32.inc
include macros.inc
includelib kernel32.lib
includelib user32.lib
.data
FileName    db 'crackme.exe',0
NewBytes    db 74h                         ;写入的新字节
OldBytes    db 75h                         ;原来的字节
.data?
startinfo     STARTUPINFO <>
pi            PROCESS_INFORMATION <>
>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
.code
start:
     invoke GetStartupInfo,addr startinfo
     invoke CreateProcess,addr FileName,NULL,NULL,NULL,FALSE,CREATE_NEW_PROCESS_GROUP,NULL,NULL,addr startinfo,addr pi
         .if eax == 0
             invoke MessageBox,NULL,CTXT("请置于程序目录"),CTXT("错误！"),MB_OK
         .endif
     invoke Sleep,500                                       ;延迟 500 毫秒等待壳的解压
     invoke SuspendThread,pi.hProcess                       ;然后迅速挂起
     invoke WriteProcessMemory,pi.hProcess,004010F9h,addr NewBytes,sizeof NewBytes,addr OldBytes    ;写入字节
     invoke ResumeThread,pi.hProcess                         ;恢复线程
end start
```

## 二、利用 WaitForInputIdle 等待系统空闲

```asm
.code
start:
    invoke GetModuleHandle,NULL
    mov hInstance,eax
    invoke CreateProcess,addr FileName ,NULL,NULL,NULL,FALSE,NORMAL_PRIORITY_CLASS,NULL,NULL, addr startinfo,addr pi
    cmp eax,0
    jz eror_1
    invoke WaitForInputIdle,pi.hProcess,INFINITE   ;等候进程进入空闲状态
    invoke WriteProcessMemory,pi.hProcess,004010F9h,CTXT(74h),1,NULL
    cmp eax,0
    jz eror_2
    fin:
    invoke ExitProcess,NULL
    eror_1:
    invoke MessageBox,NULL,CTXT("请置于程序目录"),CTXT("错误！"),MB_OK
    jmp fin
    eror_2:
    invoke MessageBox,NULL,CTXT("没有找到补丁位置"),CTXT("错误！"),MB_OK
    jmp fin
end start
```

## 三、查找窗口名

```asm
.code
start:
      invoke  GetStartupInfo,addr stStartUp
      invoke  CreateProcess, ADDR szFileName, NULL, NULL, NULL, FALSE, 0, 0, 0, ADDR stStartUp, ADDR psInfo
       .IF     EAX == 0
                invoke MessageBox,NULL,CTXT("请置于程序目录"),CTXT("错误！"),MB_OK
       .ENDIF
Search:
  invoke FindWindow,NULL,CTXT("CrackeMe8 of Laomms")
  test eax,eax
  jz Search        ; 循环，直到找到正确的窗口名字
  invoke  WriteProcessMemory,psInfo.hProcess,004010F9h, CTXT(74h),1,NULL

   .if  eax==FALSE
            invoke MessageBox,NULL,CTXT("文件不匹配！"),CTXT("错误"),MB_OK
    .endif

        invoke ExitProcess,NULL

End start
```

## 附件下载

`4dd466d0e1e0cc123afbe504da2462ca`
