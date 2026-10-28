
```asm
invoke GetModuleHandle,DllName

mov ModuleHwnd,eax

.if ModuleHwnd==0

ret

.endif

invoke GetModuleInformation,WProcess,ModuleHwnd,addr ModuleInformation,size MODULEINFO

.if eax ==0

jmp UninstallHook

ret

.endif

;原DLL的基地址

mov eax,Dllbase

assume eax:ptr

push ModuleInformation.lpBaseOfDll

pop [eax]

invoke GetProcessHeap

mov hMainHeap,eax

invoke HeapAlloc, hMainHeap, 0, ModuleInformation.SizeOfImage ;分配内存块来保存DLL

;invoke VirtualAlloc, 0,ModuleInformation.SizeOfImage,4096,PAGE_EXECUTE_READWRITE

mov lpBuffer,eax

.if lpBuffer==0

jmp UninstallHook

ret

.endif

;备份DLL

invoke WriteProcessMemory,WProcess,lpBuffer, ModuleInformation.lpBaseOfDll,ModuleInformation.SizeOfImage,0

.if eax==0

jmp UninstallHook

ret

.endif

mov eax,lpBuffer

ret

BakDll endp

;**********************************************************************

End DllEntry
```

代码里面已经写的很详细了,我就不多说了,希望对大家有帮助!

来源: [看雪论坛](http://bbs.pediy.com/showthread.php?t=308009)

参考文章: [汇编ring3下实现HOOK API](http://bbs.pediy.com/showthread.php?t=28895)

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

上传的附件：

- [Hookapi.rar](javascript:void(0);)（17.22kb，304次下载）
