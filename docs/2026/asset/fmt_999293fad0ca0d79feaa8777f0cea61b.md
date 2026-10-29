
```cpp
int __stdcall RtlIsDosDeviceName_Ustr(UNICODE_STRING *String)
{
// [COLLAPSED LOCAL DECLARATIONS. PRESS KEYPAD CTRL-"+" TO EXPAND]
v1 = String->Buffer;
v2 = 0;
v3 = RtlDetermineDosPathNameType_U(v1);
if ( v3 >= 0 )
{
if...
if...
}
str = String->Buffer;
*(DWORD *)&DestinationString.Length = *(DWORD *)&String->Length; // 这里修改的其实也是0x0014AF444，只不过内容还不是5C
w_len = String->Length >> 1;
DestinationString.Buffer = str;
if...
if... // 忽略路径最后可能存在的'：',payload中没有
if...
last_char = str[w_len - 1];
do... // 忽略路径最后可能存在的'.'或者空格，payload中没有
v7 = 0;
if ( w_len )
{
for ( i = &str[w_len - 1]; i >= str; --i ) // 从后向前遍历
{
v9 = *i;
if ( *i == '\\' || v9 == '/' || v9 == ':' && i == str + 1 )// 搜索'\'字符能够找到
{
next_char = i + 1;
v11 = *next_char | 0x20; // 转大写
if ( v11 != 'l' && v11 != 'c' && v11 != 'p' && v11 != 'a' && v11 != 'n' )// '\'后面的字母应该为'L' 'C' 'P' 'A' 'N'中的一个，否则直接返回
return 0;
v7 = (char *)next_char - (char *)str;
RtlInitUnicodeString(&DestinationString, next_char);// 关键步骤在这个函数中
str = DestinationString.Buffer;
w_len = (DestinationString.Length >> 1) - v2;
DestinationString.Length += -2 * v2;
break;
}
}
}
```

调试`RtlIsDosDeviceName_Ustr`函数，可以发现该函数中的`DestinationString`所在地址正好就在`0x0014AF444`，所以关键代码就在函数`RtlInitUnicodeString`中，该函数的第一个参数指向了我们关注的目标地址，用于存储UNICODE结果字符串，第二个参数是payload最后一段字符串（最后一个`\`字符后面）。

```cpp
// 记住参数Destination指向的就是0x0014AF444
void __stdcall RtlInitUnicodeString(PUNICODE_STRING DestinationString, PCWSTR SourceString)
{
PCWSTR v2; // edi
int v3; // ecx
bool v4; // zf
unsigned int v5; // ecx
v2 = SourceString;
*(DWORD *)&DestinationString->Length = 0; // 这里设置目标地址为0
DestinationString->Buffer = (wchar_t *)SourceString;
if ( SourceString )
{
v3 = -1;
do // 找到字符串末尾
{
if ( !v3 )
break;
v4 = *v2++ == 0;
--v3;
}
while ( !v4 );
v5 = 2 * ~v3; // 计算所占字节长度
if ( v5 > 65534 )
LOWORD(v5) = -2;
DestinationString->MaximumLength = v5;
DestinationString->Length = v5 - 2; // 这里设置了0x0014AF444的数值为所占字节长度-2
}
}
```

该函数的最后一句代码修改了`0x0014AF444`！！！

![图片描述](upload/attach/202106/600394_9S83ZV7FUBNN2V3.png)

### 2.4.3 payload分析

继续调试回到正常的exploit流程（这里我理解有误，结果导致要重新调试，所以基址发生了变化），在要执行`wcscpy`函数的时候，F7步入。

**为什么这里要F7步入？**

*导致我重新调试的原因就在这里，我一开始一直F8步过，结果程序总是直接退出了，后来我才意识到原因。

之前实验中接触的栈溢出，使用`strcpy(dest, src)`才实现溢出，`dest`位于当前栈帧栈顶的高地址处，所以溢出时覆盖的是当前栈帧的放回地址。但是这次实验发生溢出的目标地址位于当前栈帧栈顶的低地址处，发生溢出时覆盖的实际上是调用`wcscpy`函数栈帧所对应的返回地址。*

![图片描述](upload/attach/202106/600394_YCM563S6QGNDXCW.png)

这里有一点后面会提到，就是在`retn`之前的`pop ebp`指令，将返回地址前面的`0x00020408`放入了`ebp`中。

可以看到返回地址被覆盖成了`0x58FC16E2`（通过查看内存，这个地址位于`AcGenral.dll`中），进入这个地址，可以看到程序调用了`NtSetInformationProcess`，

![图片描述](upload/attach/202106/600394_4J2DFKS27BQZYUN.png)

在0day关于绕过DEP的部分介绍过这个方法：

> 一个进程的DEP设置标识位于`KPROCESS`结构中的`_KEXECUTE_OPTIONS`上，该标识可以通过API函数`ZwQueryInformationProcess`和`ZwSetInformationProcess`查询和修改。

```cpp
ULONG ExecuteFlags = MEM_EXECUTE_OPTION_ENABLE;
ZwSetInformationProcess(
NtCurrentProcess(), // (HANDLE)-1
ProcessExecuteFlags, // 0x22
&ExecuteFlags, // ptr to 0x2
sizeof(ExecuteFlags)); // 0x4
```

这里有一个问题，`NtSetInformationProcess`的第三个参数是一个指向`0x2`的地址，这个地址通过寄存器`ebp`获得，因此`ebp+0x8`需要指向一个可写的地址，这也是我们前面提到`pop ebp`的原因。

关闭DEP之后，返回地址为`0x58FBF727`，这里执行了`call esi`的指令，这里就是跳板指令了，此时`esi`指向了`0x01A0F496`，这个地址在`payload`中，执行了一个`jmp`指令`EB 62`，跳转到真正的代码（前面还有一部分垃圾指令），之后程序进行了解码操作，然后跳转到解码后的代码处：

![图片描述](upload/attach/202106/600394_6D8JZJ8MVQASCR7.png)

![图片描述](upload/attach/202106/600394_CJTE2V8WE9S5VAH.png)

总结下来payload在内存中的变化情况如下：

![图片描述](upload/attach/202106/600394_EYA52YPV5NN258S.png)

其中黄色字段的几个关键值用红色标记，解释如下：

![图片描述](upload/attach/202106/600394_374G7CJKBKN2Y7Z.png)

### 2.4.4 总结

根据上面的分析，payload的总体结构如下：

![图片描述](upload/attach/202106/600394_WS3NMUCNXCE9EYZ.png)

netapi32.dll版本: Win XP SP2

# 1. 漏洞存在位置

`NetpwPathCanonicalize` → `CanonicalizePathName` → `RemoveLegacyFolder`

`RemoveLegacyFolder`函数用于移除路径中的`.\`和`..\`。

# 2. 漏洞原理

测试代码：

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24  25  26  27  28  29  30  31  32  33  34  35  36  37  38  39  40  41 | `#include <windows.h>`  `#include <stdio.h>`    `typedef` `int` `(__stdcall` `*``MYPROC) (LPWSTR,  LPWSTR, DWORD,LPWSTR, LPDWORD,DWORD);`    `int` `main(``int` `argc, char``*` `argv[])`  `{`  `WCHAR path[``256``];`  `WCHAR can_path[``256``];`  `DWORD` `type` `=` `1000``;`  `int` `retval;`  `HMODULE handle` `=` `LoadLibrary(``".\\netapi32.dll"``);`  `MYPROC Trigger` `=` `NULL;`    `if` `(NULL` `=``=` `handle)`  `{`  `wprintf(L``"Fail to load library!\n"``);`  `return` `-``1``;`  `}`    `Trigger` `=` `(MYPROC)GetProcAddress(handle,` `"NetpwPathCanonicalize"``);`  `if` `(NULL` `=``=` `Trigger)`  `{`  `FreeLibrary(handle);`  `wprintf(L``"Fail to get api address!\n"``);`  `return` `-``1``;`  `}`    `path[``0``]` `=` `0``;`  `wcscpy(path, L``"\\aaa\\..\\..\\bbbb"``);`  `can_path[``0``]` `=` `0``;`  `type` `=` `1000``;`  `wprintf(L``"BEFORE: %s\n"``, path);`  `retval` `=` `(Trigger)(path, can_path,` `1000``, NULL, &``type``,` `1``);`  `wprintf(L``"AFTER : %s\n"``, can_path);`  `wprintf(L``"RETVAL: %s(0x%X)\n\n"``, retval?L``"FAIL"``:L``"SUCCESS"``, retval);`    `FreeLibrary(handle);`    `return` `0``;`  `}` |

## 2.1 与漏洞无关的问题

一开始没注意0day提供的源码，直接对之前调试payload的代码进行了一些修改，结果不管怎么改在输出`can_path`的时候结果都不对。经过调试，发现代码中获取的`can_path`的地址是错误的。

以我浅薄的编程经验，觉得问题只能出在`MYPROC`这个类型的声明或者`Trigger`的调用上，因为这里没有按照正常的函数调用语法来写，而是自己定义了一个函数类型。但是具体是什么原因就不清楚了。

最终经过和0day提供的源码的比较与多次修改实验，最终才发现`MYPROC`这个类型的定义上面使用了`__stdcall`调用约定。

查找资料注意到：

> `__cdecl`是C/C++和MFC程序默认使用的调用约定，也可以在函数声明时加上`__cdecl`关键字来手工指定。采用`__cdecl`约定时，函数参数按照从右到左的顺序入栈，并且由调用函数者把参数弹出栈以清理堆栈。因此，实现可变参数的函数只能使用该调用约定。

`__stdcall`调用约定用于调用Win32 API函数。采用`__stdcal`约定时，函数参数按照从右到左的顺序入栈，被调用的函数在返回前清理传送参数的栈，函数参数个数固定。由于函数体本身知道传进来的参数个数，因此被调用的函数可以在返回前用一条`ret n`指令直接清理传递参数的堆栈。

也就是说`NetpwPathCannonicalize` 函数已经自己完成了堆栈的清理，如果没有使用`__stdcall`调用约定，那么我们的代码还会自己再做一次堆栈清理，结果堆栈就不平衡了。这就是导致`can_path`没有正常输出的原因。

## 2.2 漏洞函数功能实现分析

以下是`RemoveLegacyFolder`的注释版本代码：

|  |  |

| --- | --- |
