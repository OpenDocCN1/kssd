
# 编写一个文件补丁

> 原文：[`www.pediy.com/kssd/pediy06/pediy6571.htm`](https://www.pediy.com/kssd/pediy06/pediy6571.htm)

本来应该发在翻译区，但考虑到这里不少朋友都问补丁的情况，我看还是自己写补丁来的爽快。这个代码编译后才 `2.5K`。小的很。

第一次翻译东西，不足之处请原谅！

编写一个文件补丁（Make a standalone patch）

作者 : `Detten` `Detten@tiscali.be`

来源 : [`biw.rult.at/`](http://biw.rult.at/)

翻译 : `nbw` [`www.vxer.com`](http://www.vxer.com)

## 1、前言

一但破解了一个程序，你就想把成果共享给别人。为了不用把整个破解后的程序上传，你可以制作一个小补丁来修改程序中必要的字节。

那如何来写呢？

`===〉` 首先找到需要打补丁的文件。大多数补丁认为该文件处在自己当前目录下。

`===〉` 如果找到，打开该文件。

`===〉` 然后检查打开的文件是不是和所破解的文件。比如我们可以检查文件大小，或者随机检查一些字节，或者最好检查将要被修改的字节。

`===〉` 如果以上都无误，我们可以做需要调整:)

把文件指针移动到指定位置，然后写入新的操作码。

`===〉` 关闭文件，给出提示结果。

下面找一个你破解的文件，然后开始....

## 2、必要的 API

做这个程序需要用到什么 `API` 呢？

```
HANDLE CreateFile( 
LPCTSTR lpFileName, // pointer to name of the file 
DWORD dwDesiredAccess, // access (read-write) mode 
DWORD dwShareMode, // share mode 
LPSECURITY_ATTRIBUTES lpSecurityAttributes, // pointer to security attributes 
DWORD dwCreationDistribution, // how to create 
DWORD dwFlagsAndAttributes, // file attributes 
HANDLE hTemplateFile // handle to file with attributes to copy 
);
```

这个 `API` 函数用来打开或者创建文件。

`dwDesiredAccess` 应该设置为： `'GENERIC_WRITE OR GENERIC_READ'` ，因为需要读写文件；

`dwShareMode = 'FILE_SHARE_WRITE OR FILE_SHARE_READ'`

`dwCreationDistribution = 'OPEN_EXISTING'`  我们只需要打开文件，如果文件不存在函数将返回失败，然后我们给出提示信息。

可以察看 `WIN32.HLP` 获取详细信息，如果你没有这个 `API` 库，可以找相关资料。

如你所见，这个 `API` 函数返回我们需要文件句柄。我们可以利用这个句柄做下一步：写文件。

```
BOOL WriteFile( 
HANDLE hFile, // handle to file to write to 
LPCVOID lpBuffer, // pointer to data to write to file 
DWORD nNumberOfBytesToWrite, // number of bytes to write 
LPDWORD lpNumberOfBytesWritten, // pointer to number of bytes written 
LPOVERLAPPED lpOverlapped // pointer to structure needed for overlapped I/O 
);
```

我们用这个函数把 `2` 个字节写入需要打补丁的文件（当然是在正确的位置）\[译者：作者的例子是写 `2` 个字节，我们做的时候根据需要\]

涉及到的 `hFile` 句柄就是 `CreateFile` 函数的返回值。

`lpOverlapped` 应该指向一个 `OVERLAPPED` 结构，我们需要用它设定正确的文件指针。

```
typedef struct _OVERLAPPED { // o 

DWORD Internal; 

DWORD InternalHigh; 

DWORD Offset; 

DWORD OffsetHigh; 

HANDLE hEvent; 

} OVERLAPPED; 
```

`Offset` 的内容是需要写入的地址。（译者：注意不是内存中的虚拟地址）

## 3、目标

目标程序是 `Crackme5.exe`，假定我们现在获取不到正确的序列号，而需要对他打补丁（译者：这个程序我也没有，大家知道意思就行） 

当然你肯定是不错的 `Cracker:)` 并且很快找出来了需要修补的地方：

`Offset 53Fh : 74h, 15h -> 90h, 90h`

以上就是我们所需要的所有信息。

## 4、代码

```
386 

.model flat,stdcall 

option casemap:none 

include \masm32\include\windows.inc 

include \masm32\include\user32.inc 

include \masm32\include\kernel32.inc 

includelib \masm32\lib\user32.lib 

includelib \masm32\lib\kernel32.lib 

.data 

FileName db "Crackme5.exe",0 

AppName db "Crackme 5 Patch",0 

Done db "File patched succesfully !",0 

NoFile db "Can't find crackme5.exe !",0 

ReFile db "Wrong version of crackme5.exe !",0 

WrFile db "Error writing to crackme5.exe !",0 

RBuffer db 75h, 15h 

WBuffer db 90h,90h 

OffsetPos OVERLAPPED <NULL,NULL,53Fh,NULL,NULL> 

.data? 

hInstance HINSTANCE ? 

CommandLine LPSTR ? 

hwndname HWND ? 

hFile HANDLE ? 

Numb dd ? 

Buffer db 2 dup(?) 

.const 

.code 

start: 

invoke GetModuleHandleA, NULL 

mov hInstance,eax 

invoke CreateFile,ADDR FileName, GENERIC_READ OR GENERIC_WRITE, FILE_SHARE_READ OR FILE_SHARE_WRITE, NULL, OPEN_EXISTING, FILE_ATTRIBUTE_NORMAL, NULL 

.IF eax!=INVALID_HANDLE_VALUE 

mov hFile, eax ; 存储文件句柄 

Invoke ReadFile, hFile, ADDR Buffer, 2, ADDR Numb, ADDR OffsetPos ; 读取要修改的 2 个字节 

mov ax, word ptr [Buffer] 

.IF ax == word ptr [RBuffer] ; 如果判断正确 (75h,15h) 就覆盖他们 :-) 

Invoke WriteFile, hFile, ADDR WBuffer, 2, ADDR Numb, ADDR OffsetPos ;写入新的代码(90h 90h) 

.IF Numb == 2 ; 如果返回值为 2, 弹出成功信息 

push MB_OK 

push OFFSET AppName 

push OFFSET Done ; 弹出信息地址入栈 

.ELSE ;如果返回值不是 2，那么弹出错误信息 

push MB_OK OR MB_ICONINFORMATION 

push OFFSET AppName 

push OFFSET WrFile 

.ENDIF 

.ELSE ; 如果读取的 2 个字节不正确，弹出文件选择错误信息 

push MB_OK OR MB_ICONINFORMATION 

push OFFSET AppName 

push OFFSET ReFile 

.ENDIF 

.ELSE ; 如果未获得文件句柄，弹出文件不存在信息 

push MB_OK OR MB_ICONINFORMATION 

push OFFSET AppName 

push OFFSET NoFile 

.ENDIF 

push NULL 

Call MessageBox 

invoke CloseHandle, hFile ; 关闭文件 

invoke ExitProcess,eax ; 退出 

end start 
```

如果你看懂了上面的代码，就可以制作自己的文件补丁。当然也可以编写的更人性化一些，比如添加上选择文件路径的对话框，但那就是另外一个题目了。

4dd466d0e1e0cc123afbe504da2462ca
