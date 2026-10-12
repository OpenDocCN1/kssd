
# PE 文件格式

> 原文：[`www.pediy.com/kssd/tutorial/chap8-1-2.htm`](https://www.pediy.com/kssd/tutorial/chap8-1-2.htm)

## PE 教程 2: 检验 PE 文件的有效性

本教程中我们将学习如何检测给定文件是否为一有效 PE 文件。

下载 范例

### 理论

如何才能校验指定文件是否为一有效 PE 文件呢? 这个问题很难回答，完全取决于想要的精准程度。您可以检验 PE 文件格式里的各个数据结构，或者仅校验一些关键数据结构。大多数情况下，没有必要校验文件里的每一个数据结构，只要一些关键数据结构有效，我们就认为是有效的 PE 文件了。下面我们就来实现前面的假设。

我们要验证的重要数据结构就是 PE header。从编程角度看，PE header 实际就是一个 `IMAGE_NT_HEADERS` 结构。定义如下:

```
IMAGE_NT_HEADERS STRUCT
   Signature dd ?
   FileHeader IMAGE_FILE_HEADER <>
   OptionalHeader IMAGE_OPTIONAL_HEADER32 <>
IMAGE_NT_HEADERS ENDS
```

`Signature` 一 dword 类型，值为 `50h, 45h, 00h, 00h`（`PE\0\0`）。 本域为 PE 标记，我们可以此识别给定文件是否为有效 PE 文件。

`FileHeader` 该结构域包含了关于 PE 文件物理分布的信息， 比如节数目、文件执行机器等。

`OptionalHeader` 该结构域包含了关于 PE 文件逻辑分布的信息，虽然域名有"可选"字样，但实际上本结构总是存在的。

我们目的很明确。如果 `IMAGE_NT_HEADERS` 的 `signature` 域值等于 `"PE\0\0"`，那么就是有效的 PE 文件。实际上，为了比较方便，Microsoft 已定义了常量 `IMAGE_NT_SIGNATURE` 供我们使用。

```
IMAGE_DOS_SIGNATURE equ 5A4Dh
IMAGE_OS2_SIGNATURE equ 454Eh
IMAGE_OS2_SIGNATURE_LE equ 454Ch
IMAGE_VXD_SIGNATURE equ 454Ch
IMAGE_NT_SIGNATURE equ 4550h
```

接下来的问题是: 如何定位 PE header? 答案很简单: DOS MZ header 已经包含了指向 PE header 的文件偏移量。DOS MZ header 又定义成结构 `IMAGE_DOS_HEADER` 。查询 `windows.inc`，我们知道 `IMAGE_DOS_HEADER` 结构的 `e_lfanew` 成员就是指向 PE header 的文件偏移量。

现在将所有步骤总结如下:

1. 首先检验文件头部第一个字的值是否等于 `IMAGE_DOS_SIGNATURE`，是则 DOS MZ header 有效。
2. 一旦证明文件的 DOS header 有效后，就可用 `e_lfanew` 来定位 PE header 了。
3. 比较 PE header 的第一个字的值是否等于 `IMAGE_NT_HEADER`。如果前后两个值都匹配，那我们就认为该文件是一个有效的 PE 文件。

### Example

```
.386
.model flat,stdcall
option casemap:none
include \masm32\include\windows.inc
include \masm32\include\kernel32.inc
include \masm32\include\comdlg32.inc
include \masm32\include\user32.inc
includelib \masm32\lib\user32.lib
includelib \masm32\lib\kernel32.lib
includelib \masm32\lib\comdlg32.lib

SEH struct
PrevLink dd ?      ; the address of the previous seh structure
CurrentHandler dd ?    ; the address of the exception handler
SafeOffset dd ?      ; The offset where it's safe to continue execution
PrevEsp dd ?        ; the old value in esp
PrevEbp dd ?        ; The old value in ebp
SEH ends

.data
AppName db "PE tutorial no.2",0
ofn OPENFILENAME <>
FilterString db "Executable Files (*.exe, *.dll)",0,"*.exe;*.dll",0
            db "All Files",0,"*.*",0,0
FileOpenError db "Cannot open the file for reading",0
FileOpenMappingError db "Cannot open the file for memory mapping",0
FileMappingError db "Cannot map the file into memory",0
FileValidPE db "This file is a valid PE",0
FileInValidPE db "This file is not a valid PE",0

.data?
buffer db 512 dup(?)
hFile dd ?
hMapping dd ?
pMapping dd ?
ValidPE dd ?

.code
start proc
LOCAL seh:SEH
mov ofn.lStructSize,SIZEOF ofn
mov ofn.lpstrFilter, OFFSET FilterString
mov ofn.lpstrFile, OFFSET buffer
mov ofn.nMaxFile,512
mov ofn.Flags, OFN_FILEMUSTEXIST or OFN_PATHMUSTEXIST or OFN_LONGNAMES or OFN_EXPLORER or OFN_HIDEREADONLY
invoke GetOpenFileName, ADDR ofn
.if eax==TRUE
   invoke CreateFile, addr buffer, GENERIC_READ, FILE_SHARE_READ, NULL, OPEN_EXISTING, FILE_ATTRIBUTE_NORMAL, NULL
   .if eax!=INVALID_HANDLE_VALUE
      mov hFile, eax
      invoke CreateFileMapping, hFile, NULL, PAGE_READONLY,0,0,0
      .if eax!=NULL
         mov hMapping, eax
         invoke MapViewOfFile,hMapping,FILE_MAP_READ,0,0,0
         .if eax!=NULL
            mov pMapping,eax
            assume fs:nothing
            push fs:[0]
            pop seh.PrevLink
            mov seh.CurrentHandler,offset SEHHandler
            mov seh.SafeOffset,offset FinalExit
            lea eax,seh
            mov fs:[0], eax
            mov seh.PrevEsp,esp
            mov seh.PrevEbp,ebp
            mov edi, pMapping
            assume edi:ptr IMAGE_DOS_HEADER
            .if [edi].e_magic==IMAGE_DOS_SIGNATURE
               add edi, [edi].e_lfanew
               assume edi:ptr IMAGE_NT_HEADERS
               .if [edi].Signature==IMAGE_NT_SIGNATURE
                  mov ValidPE, TRUE
               .else
                  mov ValidPE, FALSE
               .endif
            .else
               mov ValidPE,FALSE
            .endif
FinalExit:
            .if ValidPE==TRUE
               invoke MessageBox, 0, addr FileValidPE, addr AppName, MB_OK+MB_ICONINFORMATION
            .else
               invoke MessageBox, 0, addr FileInValidPE, addr AppName, MB_OK+MB_ICONINFORMATION
            .endif
            push seh.PrevLink
            pop fs:[0]
            invoke UnmapViewOfFile, pMapping
         .else
            invoke MessageBox, 0, addr FileMappingError, addr AppName, MB_OK+MB_ICONERROR
         .endif
      invoke CloseHandle,hMapping
      .else
         invoke MessageBox, 0, addr FileOpenMappingError, addr AppName, MB_OK+MB_ICONERROR
      .endif
      invoke CloseHandle, hFile
   .else
      invoke MessageBox, 0, addr FileOpenError, addr AppName, MB_OK+MB_ICONERROR
   .endif
.endif
invoke ExitProcess, 0
start endp

SEHHandler proc uses edx pExcept:DWORD, pFrame:DWORD, pContext:DWORD, pDispatch:DWORD
   mov edx,pFrame
   assume edx:ptr SEH
   mov eax,pContext
   assume eax:ptr CONTEXT
   push [edx].SafeOffset
   pop [eax].regEip
   push [edx].PrevEsp
   pop [eax].regEsp
   push [edx].PrevEbp
   pop [eax].regEbp
   mov ValidPE, FALSE
   mov eax,ExceptionContinueExecution
   ret
SEHHandler endp
end start
```

### 分析

本例程打开一文件，先检验 DOS header 是否有效，有效就接着检验 PE header 的有效性，ok 就认为是有效的 PE 文件了。这里，我们还运用了结构异常处理(SEH)，这样就不必检查每个可能的错误: 如果有错误出现，就认为 PE 检测失效所致，于是给出我们的报错信息。其实 Windows 内部普遍使用 SEH 来检验参数传递的有效性。若对 SEH 感兴趣的话，可阅读 Jeremy Gordon 的 [文章](http://bigluo.tz163.net/program/iczelion/Exceptionhandling.html)。

程序调用打开文件通用对话框，用户选定执行文件后，程序便打开文件并映射到内存。并在有效性检验前建立一 SEH:

```
   assume fs:nothing
   push fs:[0]
   pop seh.PrevLink
   mov seh.CurrentHandler,offset SEHHandler
   mov seh.SafeOffset,offset FinalExit
   lea eax,seh
   mov fs:[0], eax
   mov seh.PrevEsp,esp
   mov seh.PrevEbp,ebp
```

一开始就假设寄存器 `fs` 为空（`assume fs:nothing`）。 记住这一步不能省却，因为 MASM 假设 `fs` 寄存器为 `ERROR`。接下来保存 Windows 使用的旧 SEH 处理函数地址到我们自己定义的结构中，同时保存我们的 SEH 处理函数地址和异常处理时的执行恢复地址，这样一旦错误发生就能由异常处理函数安全地恢复执行了。同时还保存当前 `esp` 及 `ebp` 的值，以便我们的 SEH 处理函数将堆栈恢复到正常状态。

```
   mov edi, pMapping
   assume edi:ptr IMAGE_DOS_HEADER
   .if [edi].e_magic==IMAGE_DOS_SIGNATURE
```

成功建立 SEH 后继续校验工作。置目标文件的首字节地址给 `edi`，使其指向 DOS header 的首字节。为便于比较，我们告诉编译器可以假定 `edi` 正指向 `IMAGE_DOS_HEADER` 结构(事实亦是如此)。然后比较 DOS header 的首字是否等于字符串`"MZ"`，这里利用了 `windows.inc` 中定义的 `IMAGE_DOS_SIGNATURE` 常量。若比较成功，继续转到 PE header，否则设 `ValidPE` 值为 `FALSE`，意味着文件不是有效 PE 文件。

```
      add edi, [edi].e_lfanew
      assume edi:ptr IMAGE_NT_HEADERS
      .if [edi].Signature==IMAGE_NT_SIGNATURE
         mov ValidPE, TRUE
      .else
         mov ValidPE, FALSE
      .endif
```

要定位到 PE header，需要读取 DOS header 中的 `e_lfanew` 域值。该域含有 PE header 在文件中相对文件首部的偏移量。`edi` 加上该值正好定位到 PE header 的首字节。这儿可能会出错，如果文件不是 PE 文件，`e_lfanew` 值就不正确，加上该值作为指针就可能导致异常。若不用 SEH，我们必须校验 `e_lfanew` 值是否超出文件尺寸，这不是一个好办法。如果一切 OK，我们就比较 PE header 的首字是否是字符串`"PE"`。这里在此用到了常量 `IMAGE_NT_SIGNATURE`，相等则认为是有效的 PE 文件。

如果 `e_lfanew` 的值不正确导致异常，我们的 SEH 处理函数就得到执行控制权，简单恢复堆栈指针和基栈指针后，就根据 `safeoffset` 的值恢复执行到 `FinalExit` 标签处。

```
FinalExit:
   .if ValidPE==TRUE
      invoke MessageBox, 0, addr FileValidPE, addr AppName, MB_OK+MB_ICONINFORMATION
   .else
```



```
invoke MessageBox, 0, addr FileInValidPE, addr AppName, MB_OK+MB_ICONINFORMATION
```

.endif

上述代码简单明确，根据`ValidPE`的值显示相应信息。

```
push seh.PrevLink
pop fs:[0]
```

一旦`SEH`不再使用，必须从`SEH`链上断开。

翻译：`iamgufeng` [Iczelion's Win32 Assembly Homepage](http://win32asm.cjb.net/) [LuoYunBin's Win32 ASM Page](http://asm.yeah.net/)

.

`4dd466d0e1e0cc123afbe504da2462ca`
