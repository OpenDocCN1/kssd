
# 可执行程序的自删除

> 原文：`www.pediy.com/kssd/pediy10/63213.html` (https://www.pediy.com/kssd/pediy10/63213.html)

当然有很多方法了，不过这个方法不知道有没有人用过，我在无意间发现的：

```asm
.data

    explorer     db "Progman",0
    exp_class    db "Program Manager",0
    ken          db "Kernel32.dll",0
    LoadLib      db "LoadLibraryA",0
    delef        db "DeleteFileA",0

.data?

    hInstance    HINSTANCE ?
    rs_addr      dd ?
    system_dir   db 260 dup(?)
    explorer_hd  dword <>
    vm_hd        dword <>
    bw_file      dd ?
    open_proc_hd dword <>
    ken32        dword <>

.code

start:

    invoke GetModuleHandle, 0
    mov    hInstance, eax
    invoke FindWindow, addr explorer, addr exp_class
    invoke GetWindowThreadProcessId, eax, offset dwProcessID
    invoke OpenProcess, PROCESS_VM_WRITE or PROCESS_CREATE_THREAD or PROCESS_VM_OPERATION, TRUE, dwProcessID
    mov    open_proc_hd, eax
    invoke VirtualAllocEx, open_proc_hd, 0, 260, MEM_COMMIT, PAGE_EXECUTE_READWRITE
    mov    vm_hd, eax
    invoke LoadLibrary, addr ken
    mov    ken32, eax
    invoke GetModuleFileName, hInstance, addr system_dir, MAX_PATH
    invoke WriteProcessMemory, open_proc_hd, vm_hd, addr system_dir, 250, rt_hd
    invoke GetProcAddress, ken32, addr delef
    invoke CreateRemoteThread, open_proc_hd, NULL, 0, eax, vm_hd, 0, NULL
    ;invoke CloseHandle, open_proc_hd
    invoke ExitProcess, NULL

end start
```
