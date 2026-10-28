
```assembly
; 查找文件管理器窗口并获取进程ID，然后打开进程

;********************************************************************
        invoke        FindWindow,addr szDesktopClass,addr szDesktopWindow
        invoke        GetWindowThreadProcessId,eax,offset dwProcessID
        mov        dwThreadID,eax
        invoke        OpenProcess,PROCESS_CREATE_THREAD or PROCESS_VM_OPERATION or \
                PROCESS_VM_WRITE,FALSE,dwProcessID
        .if        eax
                mov        hProcess,eax
;********************************************************************
; 在进程中分配空间并将执行代码拷贝过去，然后创建一个远程线程
;********************************************************************
        invoke        VirtualAllocEx,hProcess,NULL,REMOTE_CODE_LENGTH,MEM_COMMIT,PAGE_EXECUTE_READWRITE
        .if        eax
                mov        lpRemoteCode,eax
                invoke        WriteProcessMemory,hProcess,lpRemoteCode,\
                        offset REMOTE_CODE_START,REMOTE_CODE_LENGTH,NULL
                invoke        WriteProcessMemory,hProcess,lpRemoteCode,\
                        offset lpLoadLibrary,sizeof dword * 3,NULL
                mov        eax,lpRemoteCode
                add        eax,offset _RemoteThread - offset REMOTE_CODE_START
                invoke        CreateRemoteThread,hProcess,NULL,0,eax,0,0,NULL
                invoke        CloseHandle,eax
        .endif
        invoke        CloseHandle,hProcess
    .else
        invoke        MessageBox,NULL,addr szErrOpen,NULL,MB_OK or MB_ICONWARNING
    .endif
    invoke        ExitProcess,NULL
;>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
        end        start
```

4.局限性：有时会遇到申请内存失败。

总结：以上的几种方法，相互配合将发挥更为强大的力量。但由于都是动作在ring3，有着先天不足的缺点，都逃不过内核模块的监测。

参考文献：<<Rootkits: Subverting the Windows Kernel>> 一本专门介绍rootkits的好书 可在网上baidu到
<<Win32汇编程序设计>> 罗云彬

---

2007年04月08日 22:15:49

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
