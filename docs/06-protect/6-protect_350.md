
# anti trick Ⅰ －－ OpenProcess

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-631.htm`](https://www.pediy.com/kssd/pediy07/pediy7-631.htm)

Site: `www.ptteam.com`

by 来自轻院的狼\[`Immlep`\]

最近想写一些关于 Anti 的文章，可是没有什么时间,下面的文章写的比较粗糙，如果有什么问题，希望大家指导指导，谢谢！！！

可能由于 `DebugAPI` 的缘故，当一个程序被 `Ring3` 调试器调试(`Ollydbg` 等)时，它的权限会提升，本来它是不可以使用 `OpenProcess` 打开一些系统进程的，不过如果它被调试的话，它就可以 `OpenProcess` 打系统进程，因此我们可以借助这一点来反 `Ollydbg` 等，我测试当程序被调试后由本来不可以打开变成可以打开的进程有：

- `csrss.exe`
- `smss.exe`
- `lsass.exe`
- `svchost.exe`
- `winlogon.exe`

其中获取 `csrss` 的 `PID` 也可以用 `CsrGetProcessId` 这个函数，我用 `ASM` 写了一段例子代码：

```
searchproc proc
    LOCAL hSnapshot
    LOCAL hProcess
        invoke    CreateToolhelp32Snapshot, 2, 0
        mov       hSnapshot, eax
        mov       uProcess.dwSize, sizeof uProcess
        invoke    Process32First, hSnapshot, ADDR uProcess
        mov       hProcess, eax
        push esi
        xor esi,esi
        .while hProcess!=0
            ;--------------
            call @F
            szcsr db 'csrss.exe',0
            @@:
            lea edi,uProcess.szExeFile
            push edi
            call lstrcmp
            jnz @F
            invoke    OpenProcess, PROCESS_ALL_ACCESS, 1,uProcess.th32ProcessID
            cmp eax,0
            jz @F
            invoke MessageBox,NULL,addr szcsr,addr szOK,MB_OK
            @@:
            ;-----------------
            call @F
            szsmss db 'smss.exe',0
            @@:
            lea edi,uProcess.szExeFile
            push edi
            call lstrcmp
            jnz @F
            invoke    OpenProcess, PROCESS_ALL_ACCESS, 1,uProcess.th32ProcessID
            cmp eax,0
            jz @F
            invoke MessageBox,NULL,addr szsmss,addr szOK,MB_OK
            @@:
            ;---------------------
            call @F
            szlsa db 'lsass.exe',0
            @@:
            lea edi,uProcess.szExeFile
            push edi
            call lstrcmp
            jnz @F
            invoke    OpenProcess, PROCESS_ALL_ACCESS, 1,uProcess.th32ProcessID
            cmp eax,0
            jz @F
            invoke MessageBox,NULL,addr szlsa,addr szOK,MB_OK
            @@:            
            ;---------------
            ;---------------
            call @F
            szsvc db 'svchost.exe',0
            @@:
            lea edi,uProcess.szExeFile
            push edi
            call lstrcmp
            jnz @F
            invoke    OpenProcess, PROCESS_ALL_ACCESS, 1,uProcess.th32ProcessID
            cmp eax,0
            jz @F
            invoke MessageBox,NULL,addr szsvc,addr szOK,MB_OK
            @@:            
            ;---------------
            ;---------------
            call @F
            szwlg db 'winlogon.exe',0
            @@:
            lea edi,uProcess.szExeFile
            push edi
            call lstrcmp
            jnz @F
            invoke    OpenProcess, PROCESS_ALL_ACCESS, 1,uProcess.th32ProcessID
            cmp eax,0
            jz @F
            invoke MessageBox,NULL,addr szwlg,addr szOK,MB_OK
            @@:            
            ;---------------
            ;invoke TerminateProcess,eax,0 
        skip:
        invoke    Process32Next, hSnapshot, ADDR uProcess
        mov       hProcess, eax
    .endw
    pop esi
    invoke    CloseHandle, hSnapshot
    ret
```

有时用 `Olldybg` 第一次调试会没事，再次用 `ollydbg` 调试时就会中招！！

一个测试的 `debugme`，是以前写的，用了 `CsrGetProcessId`，只能在 `XP` 下运行，不过上面的代码是可以在 `2k` 以上的系统使用的。。

本地下载

`4dd466d0e1e0cc123afbe504da2462ca`
