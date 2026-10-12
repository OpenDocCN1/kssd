
# 编写 IDA 插件

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-780.htm`](https://www.pediy.com/kssd/pediy08/pediy8-780.htm)

## 编写交互式反汇编器（*IDA*）插件

| 译者：月中人 (`[PTG]`) |
| :--- |
| 时间：2006.8.28 |
| 原文标题：Writing Interactive Disassembler (*IDA*) Plugins |
| 原文来源："Reverse Engineering and Program Understanding" 第 6 小节 |
| 原文作者：Greg Hoglund, Gary McGraw. |

*IDA*是对 Interactive Disassembler (交互式反汇编器) 的简称，它是软件逆向工程最常用的工具之一。*IDA*支持插件模块，因此客户能够扩充功能和自动化任务。为了写这本书我们创建了一个简单的*IDA*插件，它能扫描两个二进制文件并且比较它们。该插件会加亮任何被改变了的代码区域。这可以被用来比较打补丁前的和打补丁后的可执行文件，确定哪几行被修补过。

在许多情况下，软件厂商“秘密地”修补安全缺陷。我们在这里提供的工具能帮助一个攻击者找到这些秘密补丁。事先声明，这个插件可能会标注许多根本没有改变的位置。如果编译器选项被改变或者在函数之间的填充码被变更，该插件会返回相当多的假阳性 [误报]。尽管如此，这仍是一个用来举例说明该如何开始编写*IDA*插件的很棒的例子。

我们的例子也强调与 'penetrate-and-patch [潜入和补丁] 安全' 相关的最大问题。补丁简直就是攻击地图，而且聪明的攻击者知道该如何读他们。为了编译这个代码，你需要有*IDA*软件开发工具箱（*SDK*），这可以从你的*IDA*产品中得到。它们在源代码行内有注解，而且都是标准的头文件。你需要包含其他哪些头文件，取决于你要使用哪些 API 调用。注意，我们禁止了某一个警告信息而且也包含了 Windows 头文件。因为这样我们才能够使用 Windows 图形用户接口（*GUI*）代码实现弹出对话框等等。当你使用标准模板库的时候，会抛出警告 4273，所以习惯上我们禁止它。

```
#include <windows.h>
#pragma warning( disable:4273 )
#include <ida.hpp>
#include <idp.hpp>
#include <bytes.hpp>
#include <loader.hpp>
#include <kernwin.hpp>
#include <name.hpp>
```

因为我们的插件是以*SDK*提供的一个插件样本为基础，所以下列代码只是样本的一部份。这些都是必需要的函数，而且样本中已经有注解这部份。

```
//--------------------------------------------------------------------------
// This callback is called for UI notification events.
static int sample\_callback(void \* /\*user\_data\*/, int event\_id, va\_list /\*va\*/)
{
 if ( event\_id != ui\_msg )   // Avoid recursion.
  if ( event\_id != ui\_setstate
   && event\_id != ui\_showauto
   && event\_id != ui\_refreshmarked ) // Ignore uninteresting events
     msg("ui\_callback %d\\n", event\_id);
 return 0;           // 0 means "process the event";
                     // otherwise, the event would be ignored.
}
//--------------------------------------------------------------------------
// A sample of how to generate user-defined line prefixes
static const int prefix\_width = 8;

static void get\_user\_defined\_prefix(ea\_t ea,
                                    int lnnum,
                                    int indent,
                                    const char \*line,
                                    char \*buf,
                                    size\_t bufsize)
{
 buf\[0\] = '\\0';    // Empty prefix by default

 // We want to display the prefix only on the lines which
 // contain the instruction itself.

 if ( indent != -1 ) return;       // A directive
 if ( line\[0\] == '\\0' ) return;    // Empty line
 if ( \*line == COLOR\_ON ) line += 2;
 if ( \*line == ash.cmnt\[0\] ) return;  // Comment line. . .

 // We don't want the prefix to be printed again for other lines of the
 // same instruction/data. For that we remember the line number
 // and compare it before generating the prefix.

 static ea\_t old\_ea = BADADDR;
 static int old\_lnnum;
 if ( old\_ea == ea && old\_lnnum == lnnum ) return;

 // Let's display the size of the current item as the user-defined prefix.
 ulong our\_size = get\_item\_size(ea);
 // Seems to be an instruction line. We don't bother with the width
 // because it will be padded with spaces by the kernel.

 \_snprintf(buf, bufsize, " %d", our\_size);
 // Remember the address and line number we produced the line prefix for.
 old\_ea = ea;
 old\_lnnum = lnnum;

}

//--------------------------------------------------------------------------
//
//   Initialize.
//
//   IDA will call this function only once.
//   If this function returns PLGUIN\_SKIP, IDA will never load it again.
//   If this function returns PLUGIN\_OK, IDA will unload the plugin but
//   remember that the plugin agreed to work with the database.
//   The plugin will be loaded again if the user invokes it by
//   pressing the hot key or by selecting it from the menu.
//   After the second load, the plugin will stay in memory.
//   If this function returns PLUGIN\_KEEP, IDA will keep the plugin
//   in memory. In this case the initialization function can hook
//   into the processor module and user interface notification points.
//   See the hook\_to\_notification\_point() function.
//
//   In this example we check the input file format and make the decision.
//   You may or may not check any other conditions to decide what you do,
//   whether you agree to work with the database.
//
int init(void)
{
 if ( inf.filetype == f\_ELF ) return PLUGIN\_SKIP;

// Please uncomment the following line to see how the notification works:
// hook\_to\_notification\_point(HT\_UI, sample\_callback, NULL);

// Please uncomment the following line to see how the user-defined prefix works:
// set\_user\_defined\_prefix(prefix\_width, get\_user\_defined\_prefix);
 return PLUGIN\_KEEP;
}

//--------------------------------------------------------------------------
//   Terminate.
//   Usually this callback is empty.
//   The plugin should unhook from the notification lists if
//   hook\_to\_notification\_point() was used.
//
//   IDA will call this function when the user asks to exit.
//   This function won't be called in the case of emergency exits.

void term(void)
{
 unhook\_from\_notification\_point(HT\_UI, sample\_callback);
 set\_user\_defined\_prefix(0, NULL);
}
```

这里还要包含其他几个头文件和一些全局变量：

```
#include <process.h>
#include "resource.h"

DWORD g\_tempest\_state = 0;
LPVOID g\_mapped\_file = NULL;
DWORD g\_file\_size = 0;
```

这个函数把一个文件载入内存。这个文件将作为目标，和我们之前装入*IDA*的二进制文件做对比。典型地，你应该是先把未打补丁的文件装入*IDA*，然后用它比较打过补丁的文件：

```
bool load\_file( char \*theFilename )
{
    HANDLE aFileH =
        CreateFile( theFilename,
                    GENERIC\_READ,
                    0,
                    NULL,
                    OPEN\_EXISTING,
                    FILE\_ATTRIBUTE\_NORMAL,
                    NULL);

    if(INVALID\_HANDLE\_VALUE == aFileH)
    {
        msg("Failed to open file.\\n");
        return FALSE;
    }

    HANDLE aMapH =
           CreateFileMapping( aFileH,
                              NULL,
                              PAGE\_READONLY,
                              0,
                              0,
                              NULL);
    if(!aMapH)
    {
           msg("failed to open map of file\\n");
           return FALSE;
    }


    LPVOID aFilePointer =
           MapViewOfFileEx(
                  aMapH,
                  FILE\_MAP\_READ,
                  0,
                  0,
                  0,
                  NULL);

    DWORD aFileSize = GetFileSize(aFileH, NULL);

    g\_file\_size = aFileSize;
    g\_mapped\_file = aFilePointer;

    return TRUE;
}
```

这个函数 [根据用户提供的内存地址和长度从*IDA*的数据库中] 取一串操作码，然后在目标文件中扫描这些字节。如果在目标文件中没能找到操作码，那个内存位置将会被标记为有改变。显然这是简单的技术，但是它在许多情况下是有效力的。由于存在这一小节开始处所列出的那些问题，这种方式会引起假阳性问题。
