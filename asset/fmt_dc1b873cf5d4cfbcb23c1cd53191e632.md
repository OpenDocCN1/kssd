
# 77c3f0f3 55 push ebp

```
77c3f0f3 55             push    ebp  
77c3f0f4 8bec           mov     ebp,esp  
77c3f0f6 81ec50020000   sub     esp,0x250  
77c3f0fc 33c0           xor     eax,eax  
77c3f0fe 8945d8         mov     [ebp-0x28],eax  
77c3f101 8945f0         mov     [ebp-0x10],eax  
77c3f104 8945ec         mov     [ebp-0x14],eax  
77c3f107 8b450c         mov     eax,[ebp+0xc]  
```

```
0:000> u  
```

`MSVCRT!_output+0x17:`

```
77c3f10a 53             push    ebx  
77c3f10b 8a18           mov     bl,[eax]  
```

即使你不懂汇编，你也会发现一些东西。首先，我们可以发现这个内存地址在`EAX`当中。它是CPU的一个寄存器，但是我们可以把它看作一个变量。环绕`EAX`的`[]`符号相当于C语言中的`*MyPointer`。这意味着我们正在引用`EAX`指向的地址。那么`EAX`又是怎么来的？`EAX`来自`[EBP+0Ch]`，你可以把它看作`DWORD *EBP, EAX=EBP[3]`。这是因为在汇编语言中，没有类型。`EAX`是一个32位寄存器，`EBP+12`相当于一个`DWORD`指针加3。

下面我们可以看到`MOV EBP, ESP`。`ESP`是栈指针。参数都是压入栈中，返回地址和局部变量都是在栈中。`ESP`指向栈的位置。在内存中，一个C函数调用约定的存放如下:

```
[Parameter n]  

...  

[Parameter 2]  

[Parameter 1]  

[Return Address]  
```

并且，我们看到了`PUSH EBP`。`PUSH`就是把某个东西压入栈，因此我们在栈中保存了`EBP`以前的值。此时，栈的状态如下:

```
[Parameter n]  

...  

[Parameter 2]  

[Parameter 1]  

[Return Address]  

[Previous EBP]  
```

然后我们又把`EBP`设为`ESP`，我们可以把它看作是一个指针，栈就相当于一个`DWORD`类型的数组。因此，栈中各个变量与`EBP`的对应如下:

```
[Parameter n]     ==  [EBP + n*4 + 4]  (The formula)  

...  

[Parameter 2]     ==  [EBP + 12]  

[Parameter 1]     ==  [EBP + 8]  

[Return Address]  ==  [EBP + 4]  

[Previous EBP]    ==  [EBP + 0]  
```

对于我们这个例子来说，我们的变量是`_output`的第二个参数。那么，下面该怎么办呢？我们来反汇编调用函数!我们知道`EBP+4`指向返回地址，或者我们也可以得到栈回溯。

```
0:000> kb  

ChildEBP RetAddr   Args to Child  

0012fd60 77c3e68d 77c5aca0 00000000 0012fdb0 MSVCRT!_output+0x18  

0012fda4 0040102f 00000000 00000000 00403010 MSVCRT!printf+0x35  

0012ff4c 00401125 00000001 00323d70 00322ca8 temp!main+0x2f  

0012ffc0 77e814c7 77f944a8 00000007 7ffdf000 temp!mainCRTStartup+0xe3  

0012fff0 00000000 00401042 00000000 78746341 kernel32!BaseProcessStart+0x23  

0:000>  
```

“`KB`”命令能得到栈回溯。我们不会总是得到完整的栈回溯，我们会在更加深入的教程中讲解这点。在这篇简单的教程里，我们假定我们得到了完整的栈回溯。我们注意到，这个函数是`printf`，并且`printf`调用`_output`。我们来反汇编`printf`，注意我们不用每次都反汇编整个函数，将其分割成几段即可。有时，我们能从栈回溯中很简单就找到错误发生点，而这些函数也都非常简单，我们可以非常轻松的跟踪他们。

```
0:000> u MSVCRT!_output  

MSVCRT!_output:  

77c3f0f3 55             push    ebp  
77c3f0f4 8bec           mov     ebp,esp  
77c3f0f6 81ec50020000   sub     esp,0x250  
77c3f0fc 33c0           xor     eax,eax  
77c3f0fe 8945d8         mov     [ebp-0x28],eax  
77c3f101 8945f0         mov     [ebp-0x10],eax  
77c3f104 8945ec         mov     [ebp-0x14],eax  
77c3f107 8b450c         mov     eax,[ebp+0xc]  
```

```
0:000> u  
```

`MSVCRT!_output+0x17:`

```
77c3f10a 53             push    ebx  
77c3f10b 8a18           mov     bl,[eax]  
77c3f10d 33c9           xor     ecx,ecx  
77c3f10f 84db           test    bl,bl  
77c3f111 0f8445070000   je      MSVCRT!_output+0x769 (77c3  
77c3f117 56             push    esi  
77c3f118 57             push    edi  
77c3f119 8bf8           mov     edi,eax  
```

```
0:000> u MSVCRT!printf  

MSVCRT!printf:  

77c3e658 6a10           push    0x10  
77c3e65a 68e046c177     push    0x77c146e0  
77c3e65f e8606effff     call    MSVCRT!_SEH_prolog (77c354  
77c3e664 bea0acc577     mov     esi,0x77c5aca0  
77c3e669 56             push    esi  
77c3e66a 6a01           push    0x1  
77c3e66c e8bdadffff     call    MSVCRT!_lock_file2 (77c394  
77c3e671 59             pop     ecx  
```

```
0:000> u  
```

`MSVCRT!printf+0x1a:`

```
77c3e672 59             pop     ecx  
77c3e673 8365fc00       and     dword ptr [ebp-0x4],0x0  
77c3e677 56             push    esi  
77c3e678 e8c7140000     call    MSVCRT!_stbuf (77c3fb44)  
77c3e67d 8945e4         mov     [ebp-0x1c],eax  
77c3e680 8d450c         lea     eax,[ebp+0xc]  
77c3e683 50             push    eax  
77c3e684 ff7508         push    dword ptr [ebp+0x8]  
```

```
0:000> u  
```

`MSVCRT!printf+0x2f:`

```
77c3e687 56             push    esi  
77c3e688 e8660a0000     call    MSVCRT!_output (77c3f0f3)  
```

`_output`的第二个参数是`[EBP+8]`,并且有`PUSH EBP`和`MOV EBP, ESP`，因此这和我之前说的情况一样。但是也不总是这样的，要视具体情况而定，我们慢慢再深入讲解。

因此，我们可以确定`printf`的第一个参数在内存中的哪个地方，并且`printf`是我们的程序发出的调用。从错误信息，我们知道`EAX`是0，因此我们在对一个空指针解引用。

```
77c3f10b 8a18    mov bl,[eax]   ds:0023:00000000=??  
```

下面是我们写的代码:

```
int main(int argc, char *argv[])  
 {    
  char *TheLastParameter[100];  
  sprintf(*TheLastParameter, "The last parameter is %s", argv[argc]);  
  printf(*TheLastParameter);  
  return 0;  
 }  
```

这段代码有很多问题，然后因为空指针，错误发生在`printf`。奇怪的是，并没有在`sprintf()`处出现错误。那么，我们该怎样只用`KB`命令解决这个问题呢？

```
0:000> kb  

ChildEBP RetAddr   Args to Child  

0012fd60 77c3e68d 77c5aca0 00000000 0012fdb0 MSVCRT!_output+0x18  

0012fda4 0040102f 00000000 00000000 00403010 MSVCRT!printf+0x35  

0012ff4c 00401125 00000001 00323d70 00322ca8 temp!main+0x2f  

0012ffc0 77e814c7 77f944a8 00000007 7ffdf000 temp!mainCRTStartup+0xe3  

0012fff0 00000000 00401042 00000000 78746341 kernel32!BaseProcessStart+0x23  

0:000>  
```

我们有符号和栈回溯，第一个参数是0，并且我们调用了这个函数。这是很简单的情况，我会试着讲述一些技巧，让你发现问题的所在之处。知道栈是如何组成的，在内存中的状态，这对你找出问题所在之处非常有帮助，因为仅仅一个“`kb`”命令不会让你一眼就能看出问题。

## 不按预期运行的程序

这也是一个很常见的错误。你运行了程序，但是你并没有看到正确的输出，或者总是有一些错误信息。这个常见的问题有可能非常容易解决，也可能很复杂。那么解决这种问题的步骤如何呢？

1. 什么东西没有工作？
2. 哪些API或模块可能会和这个问题有关？
3. 什么原因导致这些API不正常的工作？

这些步骤不是必须的。下面让我们来看一个例子，关于打开文件的。

```
  HANDLE hFile;  
  DWORD dwWritten;  
  hFile = CreateFile("c:\\MyFile.txt", GENERIC_READ,  
                      0, NULL, OPEN_EXISTING, 0, NULL);  
  if(hFile != INVALID_HANDLE_VALUE)  
  {  
   WriteFile(hFile, "Test", strlen("Test"), &dwWritten, NULL);  
   CloseHandle(hFile);  
  }  
```

这是我们的代码。你可能会想使用`GetLastError()`，然后重新编译，并将错误打印出来。但是，你并不需要这样做，在这个例子中非常简单。下面我们来看看，打开调试器，并且会停在这个函数。有符号的帮助，一切都很简单，`CreateFile`是一个导出的符号，我们总是能断在这个函数上。

```
C:\programs\DirectX\Games\src\Games\temp\bin>cdb temp  

Microsoft (R) Windows Debugger  Version 6.3.0005.1  

Copyright (c) Microsoft Corporation. All rights reserved.  

CommandLine: temp  

Symbol search path is:  
    SRV*c:\symbols*http://msdl.microsoft.com/download/symbols  

Executable search path is:  

ModLoad: 00400000 00404000   temp.exe  

ModLoad: 77f50000 77ff7000   ntdll.dll  

ModLoad: 77e60000 77f46000   C:\WINDOWS.0\system32\kernel32.dll  

ModLoad: 77c10000 77c63000   C:\WINDOWS.0\system32\MSVCRT.dll  

(2a0.94): Break instruction exception - code 80000003 (first chance)  

eax=00241eb4 ebx=7ffdf000 ecx=00000004 edx=77f51310 esi=00241eb4 edi=00241f48  

eip=77f75a58 esp=0012fb38 ebp=0012fc2c iopl=0         nv up ei pl nz na pe nc  

cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000             efl=00000202  

ntdll!DbgBreakPoint:  

77f75a58 cc             int     3  
```

```
0:000> bp temp!main  

0:000> g  
```

我们在`main`函数上下一个断点，然后使用“`g`”命令让其运行。当我们到达断点，用“`p`”命令单步走，直到`CreateFile`函数。

```
Breakpoint 0 hit  

eax=77c5c9e4 ebx=7ffdf000 ecx=00322cf8 edx=00322cf8 esi=00000000 edi=00000000  

eip=00401000 esp=0012ff50 ebp=0012ffc0 iopl=0         nv up ei pl zr na po nc  

cs=001b  ss=0023  ds=0023  es=0023  fs=0038  gs=0000             efl=00000246  

temp!main:  

00401000 51             push    ecx  
```

```
0:000> p  

eax=77c5c9e4 ebx=7ffdf000 ecx=00322cf8 edx=00322cf8 esi=00000000 edi=00000000  

eip=00401001 esp=0012ff4c ebp=0012ffc0 iopl=0         nv up ei pl zr na po nc  

cs=001b  ss=0023  ds=0023  es=0023  fs=0038  gs=0000             efl=00000246  

temp!main+0x1:  

00401001 56             push    esi  
```

```
0:000>  

eax=77c5c9e4 ebx=7ffdf000 ecx=00322cf8 edx=00322cf8 esi=00000000 edi=00000000  

eip=00401002 esp=0012ff48 ebp=0012ffc0 iopl=0         nv up ei pl zr na po nc  

cs=001b  ss=0023  ds=0023  es=0023  fs=0038  gs=0000             efl=00000246  

temp!main+0x2:  

00401002 57             push    edi  
```

```
0:000>  

eax=77c5c9e4 ebx=7ffdf000 ecx=00322cf8 edx=00322cf8 esi=00000000 edi=00000000  

eip=00401003 esp=0012ff44 ebp=0012ffc0 iopl=0         nv up ei pl zr na po nc  

cs=001b  ss=0023  ds=0023  es=0023  fs=0038  gs=0000             efl=00000246  

temp!main+0x3:  
```
