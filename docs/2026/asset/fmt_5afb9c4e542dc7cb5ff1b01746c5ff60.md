
# 代码自定位 获取LoadLibrary的问题

哥们, 我第一次上来发帖, 问个问题, 这个问题比较麻烦, 一时还不大好说的清. 我慢慢说, 我想让一个程序完全脱离程序执行, 并且不调用任何的API函数, 这是可以做到的, 罗云彬的书上有很详细的讲解, 下面我把我的代码贴出来, 我遇到的问题主要是除了`kernel32`外任何的API函数都不能够使用. 按说用`LoadLibrary`导入其他的DLL就可以使用其他DLL的函数了,事实上也确实是这样, 因为我看<<windows 环境下32位汇编语言程序设计>>也是这样做的, 但是我这里这个代码却不知道出了什么问题? OD调试了很久也看不出问题. 大侠就指点了, 我表达能力非常有限, 我把代码贴出来了啊, 有好心的大侠就看看啊!

`Include masm32rt.inc`

`Include JoenWin.inc`

`.code`

;获取Kernel的基址 `ecx`, 中存放`kernel`的基址,

`CodeStart:`

`Include JGetApiAddr.asm`

;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

;存放函数字符串

;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

`szLoadLibrary`                byte         "`LoadLibraryA`",0

`szGetProcAddress`         byte        "`GetProcAddress`",0

`szGetModuleHandle`        byte        "`GetModuleHandleA`", 0

`szExitProcess`                byte        "`szExitProcess`", 0

`szAllocConsole`                byte        "`AllocConsole`", 0

`szUser32`                byte        "`user32.dll`", 0, 0, 0

`szMessageBox`                byte        "`MessageBoxA`", 0

;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

;存放地址

;\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

`hKernelModule`                dword        ?

`hUser32Module`                dword        ?

`lpLoadLibrary`                dword        ?        ;这2个地址尤其重要的

`lpGetProcAddress`        dword        ?

`lpMessageBox`                dword        ?        ;MessageBox的地址

`lpAllocConsole`                dword        ?

`lpExitProcess`                dword        ?

`Entry:`

        mov        `ecx`, `edx`                        ;这里是`CreateProcess`的入口

        call        @f                                ;自定位

        @@:

        pop        `ebx`

        sub        `ebx`, offset @b

        invoke        `JgetKernelBase`, `ecx`

        ;push        `ecx`

        ;mov        `eax`, [`ebx`+ offset `JgetKernelBase`]

        ;call        `JgetKernelBase`                                ;获取Kernel32的基址

        mov        [`ebx`+ `hKernelModule`], `eax`                ;保存下`kernel`的地址

        lea        `esi`, [`ebx`+offset `szGetProcAddress`]

        invoke        `JGetApi`, [`ebx`+`hKernelModule`],`esi`        ;获取`GetProcAddress`的地址

        mov        [`ebx`+offset `lpGetProcAddress`], `eax`        ;保存`GetProcAddress`的地址

        lea        `esi`, [`ebx`+offset `szLoadLibrary`]                ;获取`LoadLibrary`的地址

        push        `esi`

        push        [`ebx`+`hKernelModule`]

        call        [`ebx`+offset `lpGetProcAddress`]

        mov        [`ebx`+offset `lpLoadLibrary`], `eax`

        lea        `esi`, [`ebx`+offset `szAllocConsole`]        ;获取`AllocConsole`的地址

        push        `esi`

        push        [`ebx`+`hKernelModule`]

        call        [`ebx`+offset `lpGetProcAddress`]

        mov        [`ebx`+offset `lpAllocConsole`],`eax`

        lea        `esi`, [`ebx`+offset `szUser32`]

        push        `esi`

        mov        `edx`, [`ebx`+offset `lpLoadLibrary`]

        call        @f

        @@:

        jmp        `edx`

;到底要怎么样才可以呢?, 我试过很多次, 有时候可以有时候又, 不可以. 这是哪里出问题了?

        mov        [`ebx`+`hUser32Module`], `eax`                ;问题就在这里, 我`CALL`的地址也是对的啊, 我`RP`有问题???

        lea        `esi`, [`ebx`+offset `szMessageBox`]                ;如果逻辑正确, 这里就可以获取`MessageBox`的地址了

        push        `esi`

        push        [`ebx`+`hUser32Module`]

        call        [`ebx`+offset `lpGetProcAddress`]

        mov        [`ebx`+offset `lpMessageBox`], `eax`

        push        0

        push        [`ebx`+offset `szMessageBox`]                ;然后`CALLMessageBox`

        push        [`ebx`+offset `szMessageBox`]

        push        0

        call        `eax`

        lea        `esi`, [`ebx`+offset `szExitProcess`]                ;获取`ExitProcess`的地址

        push        `esi`

        push        [`ebx`+`hKernelModule`]

        call        [`ebx`+offset `lpGetProcAddress`]

        mov        [`ebx`+offset `lpExitProcess`], `eax`

        push        0                                        ;程序退出

        call        `eax`

`CodeEnd:`

;代码长度

`CODELENGTH`         equ        `CodeEnd` - `CodeStart`

`CODEENTRY`        equ         `Entry` - `CodeStart`

`Jmain:`

        mov        `edx`, [`esp`]                        ;保留好`esp`的值, 这里`kernel`的根本啊

        sub        `esp`, `CODELENGTH`

        mov        `esi`, `CodeStart`

        mov        `ecx`, `CODELENGTH`                        ;复制代码到堆栈中

        mov        `edi`, `esp`

        rep        movsb

        mov        `eax`, `esp`

        add        `eax`, `CODEENTRY`                        ;求出代码执行的偏移

        jmp        `eax`

end        `Jmain`

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
