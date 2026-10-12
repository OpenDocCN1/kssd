
# 文件的自删除代码

> 原文：[`www.pediy.com/kssd/pediy05/pediy50317.htm`](https://www.pediy.com/kssd/pediy05/pediy50317.htm)

```asm
;                 exe 运行时文件自删除的实现（适用于 9x/2K)

;Theory:

;1）为什么运行时的 exe 文件不能被删除？

;Windows 操作系统为内核对象保存着一个计数，只要这个计数不为 0，那么这个内核

;对象就不会被释放，对其进行的删除操作等也就不能成功。为了达成我们的目的，必须

;在执行 deletefile 前将运行的 exe 本身的内核计数减为 0。

;2）9x 和 NT/2K 实现的不同

;9x 下只要对 exe 本身先执行 FreeLibrary 操作即可解除 exe image 在内存的映射,

;2K/NT 下需要先关闭硬编码为 4(exe 本身对应 image 的句柄),然后对 exe 本身的内存

;映像 unmapview 即可解除映射,最后的操作是相同的就是执行 DeleteFile.

;3)其他方法

;文件自删除的方法不止一种,很多,但这里的可能是最简洁的.

;3)为什么还写一遍?

;包括 Gary Nebbett 等高人都写过类似代码,但我还没发现在 9X 和 NT 上全部正常的代码

;实际上思路很简单,就是让上述两种平台的 API 代码全部执行一遍,在一种平台上可能

;会有几个失败,有几个会成功,没有关系!最后的结果是 exe 文件在退出前就被删除了!

;最后还是为了使用我喜欢的 fasm.

;4)价值

;适用于反安装程序最后的自删除或者你要写个木马、病毒首次安装的自毁.

;注意:此方法在 Xp 上无效.(还未发现在 xp 平台下用此类方法的代码)

;经 9X 和 2K 下测试通过.

;compile:

;fasm %1 selfkill.exe

;

;

;                             YJ.Hume.冷雨飘心

;                             humeasm.yeah.net

format  PE GUI 4.0

entry  StArT

section 'CODE' code readable writeable executable

StArT:

        ;int 3

        sub    esp,260

        mov    edi,esp

        push    260

        push    edi

        push    0

        call    [GetModuleFileName]

        push    0

        call    [GetModuleHandle]

        push    0

        push    0

        push    edi

        push    [ExitProcess]

        push    eax

        push    [DeleteFile]

        push    eax

        push    [UnmapViewOfFile]

        push    4

        push    [FreeLibrary]

        push    [CloseHandle]

        ret

;------------------------[IAT 数据]-----------------

section '.idata'  import data readable writeable

      dd 0,0,0,rva zk32_names,RVA k32_first_thunk_array

      dd 0,0,0,rva zu32_names,RVA u32_first_thunk_array

        times 5 dd 0

k32_first_thunk_array:

ExitProcess            dd RVA __aExitProcess

GetModuleFileName      dd RVA __aGetModuleFileName

DeleteFile             dd RVA __aDeleteFile

GetModuleHandle        dd RVA __aGetModuleHandle

FreeLibrary            dd RVA __aFreeLibrary

CloseHandle            dd RVA __aCloseHandle

UnmapViewOfFile        dd RVA __aUnmapViewOfFile

                dd 0

__aExitProcess  dw 0

                db "ExitProcess",0

__aGetModuleFileName dw 0

                db "GetModuleFileNameA",0

__aDeleteFile  dw 0

                db "DeleteFileA",0

__aGetModuleHandle dw 0

                db "GetModuleHandleA",0

__aFreeLibrary  dw 0

                db "FreeLibrary",0

__aCloseHandle  dw 0

                db "CloseHandle",0

__aUnmapViewOfFile dw 0

                db "UnmapViewOfFile",0

u32_first_thunk_array:

MessageBox      dd RVA __aMessageBox

wsprintf        dd RVA __wsprintf

                dd 0

__aMessageBox  dw 0

                db "MessageBoxA",0

__wsprintf      dw 0

                db "wsprintfA",0

zk32_names db "KERNEL32.DLL",0

zu32_names db "USER32.DLL",0
```
