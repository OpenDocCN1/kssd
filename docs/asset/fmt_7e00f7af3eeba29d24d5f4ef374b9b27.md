
```
f9c7cd50 7c92eb94 00000000 00000000 00000003 nt!KiFastCallEntry+0xf8

0055ee58 77d1f229 77d3b22a 00000000 00000000 ntdll!KiFastSystemCallRet

0055f110 77d3b12b 0055f26c 0055f340 0174fb14 USER32!NtUserGetDCEx+0xc

0055f260 77d65fdf 0055f26c 00000028 00000000 USER32!MessageBoxWorker+0x2ba

0055f2b8 77d50574 00000000 0174a1ac 0174a1a8 USER32!MessageBoxTimeoutW+0x7a

0055f2d8 77d6615b 00000000 0174a1ac 0174a1a8 USER32!MessageBoxExW+0x1b

0055f2f4 0174a19f 00000000 0174a1ac 0174a1a8 USER32!MessageBoxW+0x45

WARNING: Stack unwind information not available. Following frames may be wrong.

0055f350 7c9211a7 01740000 00000001 00000000 32+0xa19f

0055f370 7c93cbab 0174b158 01740000 00000001 ntdll!LdrpCallInitRoutine+0x14

0055f478 7c936178 00000000 c0150008 00000000 ntdll!LdrpRunInitializeRoutines+0x344

0055f724 7c9362da 00000000 00163910 0055fa18 ntdll!LdrpLoadDll+0x3e5

0055f9cc 7c801bb9 00163910 0055fa18 0055f9f8 ntdll!LdrLoadDll+0x230

0055fa34 7c801d6e 7ffdcc00 00000000 00000000 KERNEL32!LoadLibraryExW+0x18e

0055fa48 7c801da4 016e0000 00000000 00000000 KERNEL32!LoadLibraryExA+0x1f

0055fa64 7c8341cc 016e0000 0055fac0 00000000 KERNEL32!LoadLibraryA+0x94

0055faac 7c92eac7 7c801d77 016e0000 00000000 KERNEL32!BaseDispatchAPC+0x50

0055fff4 00000000 00000000 000000c8 0000012a ntdll!KiUserApcDispatcher+0x7

STACK_COMMAND: kb
MODULE_NAME: memory_corruption
IMAGE_NAME: memory_corruption
FOLLOWUP_NAME: memory_corruption
MEMORY_CORRUPTOR: LARGE
FAILURE_BUCKET_ID: MEMORY_CORRUPTION_LARGE
BUCKET_ID: MEMORY_CORRUPTION_LARGE
Followup: memory_corruption
```

---------

```
kd> lmvm win32k
```

```
start    end        module name
bf800000 bf9c0200   win32k     (pdb symbols)          D:\Program Files\Symbols\win32k.pdb\D48935A134F249CDA985C45051FBF0462\win32k.pdb

    Loaded symbol image file: win32k.sys
    Image path: \SystemRoot\System32\win32k.sys
    Image name: win32k.sys
    Timestamp:        Wed Aug 04 14:17:30 2004 (41107F7A)
    CheckSum:         001C663B
    ImageSize:        001C0200
    File version:     5.1.2600.2180
    Product version:  5.1.2600.2180
    File flags:       0 (Mask 3F)
    File OS:          40004 NT Win32
    File type:        3.7 Driver
    File date:        00000000.00000000
    Translations:     0804.04b0
    CompanyName:      Microsoft Corporation
    ProductName:      Microsoft(R) Windows(R) Operating System
    InternalName:     win32k.sys
    OriginalFilename: win32k.sys
    ProductVersion:   5.1.2600.2180
    FileVersion:      5.1.2600.2180 (xpsp_sp2_rtm.040803-2158)
    FileDescription:  Multi-User Win32 Driver
    LegalCopyright:   (C) Microsoft Corporation. All rights reserved.
```

可见这个 BSOD 是由于内存访问违规引起的。由于 `win32k.sys` 中的这条指令，读取了 `0xf0006c74` 的内容，而 `0xf0006c74` 不在内存中，于是引起了蓝屏。

```
0008:bf838c27  8b7108    mov esi,dword ptr [ecx+8]     ; ds:0023:f0006c74=????????
```

回想这个蓝屏发生的原因，由于 `LoadLibrary` 了一个 `cmdShell` 到 `csrss.exe` 的第二个线程中，结果就出现了这个蓝屏。查看蓝屏前的堆栈，可以看到这样三行：

```
0055fa34 7c801d6e 7ffdcc00 00000000 00000000 KERNEL32!LoadLibraryExW+0x18e
0055fa48 7c801da4 016e0000 00000000 00000000 KERNEL32!LoadLibraryExA+0x1f
0055fa64 7c8341cc 016e0000 0055fac0 00000000 KERNEL32!LoadLibraryA+0x94
```

这说明了 `LoadLibraryA` 确实被执行了。也就是说如果我们写一段 `ShellCode` 到 `CSRSS` 中，就可以利用 `CSRSS` 的权限作我们想做的事情。至于注入 `CSRSS` 所需要的权限，例如管理员、`SeDebugPrivilege` 等，参见其他注入教程中关于 `OpenThread` 和 `OpenProcess` 的部分。

---------------------------------------------------------------------------------------

**附件 `APC_src.rar` 说明**：

`APC 注入_D2009\AnyAPC_Boom.rar` 里面是 `APC` 注入的主程序，用法见源码目录下的 `readme.txt`

`注射用 DLL\多线程 cmdshell\s32.dll` 改成 `32.dll` 丢到 `Windows` 安装目录中，注入后监听 `3820` 端口，可以 `Telnet` 试

相关的程序源码都在里面，自己看吧。另外，建议先扫一遍毒再运行![](img/c72cb8eba397ac67dda6e1ac58a9b627.jpg "微笑")

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) | `APC_src.rar` `[解压密码：pediy]` |
| :--- | :--- |
