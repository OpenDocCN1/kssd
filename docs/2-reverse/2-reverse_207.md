

# WinDbg 插件编写——基础篇

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-30.htm`](https://www.pediy.com/kssd/pediy09/pediy09-30.htm)

> WinDbg 插件编写——基础篇  
>   
>     本文只是简略的描述了一下 WinDbg 扩展命令的编写步骤及方法，更详细的请看他的帮助文档，起到一个抛砖引玉的作用就行了。  
> WinDbg 是老盖自己家出的调试器，功能很强大。即可调试用户态，也可以调试核心态程序。本文主要介绍他插件（准确讲应该是扩展命令）的编写方法。WinDbg 的扩展命令和 OD 一样是通过 DLL 文件导出的。  
> 一．插件的类型  
> 他有两种不同类型的 DLL 文件：  
> 1．DbgEng 类型 DLL。这种类型的 DLL 都已“dbgeng.h” 头文件为基础。扩展命令的功能比较强大，可以用 Debugger Engine API（这种调试 API 不依赖 WinDbg,可以用来编写自己的调试器），也可以用 WdbgExts API。  
> 2．WdbgExts 类型 DLL。这种类型的 DLL 以“Wdbgexts.h” 头文件为基础，它只能用 WdbgExts API.  
> 二．两种插件的编写方法。  
> 一般来说编写一个扩展命令需要四个文件：.c/c++文件，.def 文件，sources 文件和 makefile 文件，和写一般 DLL 文件一样。下面主要讲 c/c++文件和 def 文件，应为他们在两种类型的 DLL 中差异比较大。  
> 1.DbgEng 类型。  
> (1) c/c++文件,可以用 dbgeng.h 标准的 C++代码调试接口也可以用 wdbgexts.h 中的 c 代码接口。如果在这个类型中用 wdbgexts API，需要用到 wdbgexts.h 和 KDEXT\_64BIT 宏,如：  
>    #define KDEXT\_64BIT  
>    #include wdbgexts.h  
>    #include dbgeng.h  
> 这个文件主要依据 dbgeng.h 和 wdbgexts.h 中的调试接口函数实现特定功能的函数代码。第一行主要是编译 64 位的，两个头文件中的函数都有 32 和 64 位，如 ReadIoSpace64 和 ReadIoSpace,你可以在这种类型的扩展中完全用 64 位的函数，既可以在 64 位 cpu 也可以在 32 为 cpu 运行。  
> (2)def 文件，定义导出函数和自己的扩展命令  
> DebugExtensionInitialize 函数必须导出，当 DLL 被加载时用来初始化全局变量。其他的根据实际情况而定。  
> 2．WdbgExts 类型  
> 这种类型扩展的功能比上一种来说差一些，但一般情况下也能完成很多任务。  
> （1）C 文件。  
>    I.只能用 wdbgexts.h 中的 c 接口，如果用 64 位，简单的包含  
> #define KDEXT\_64BIT  
> #include wdbgexts.h  
> 只是编译 32 位的话，就无需包含第一句。  
>    II.必须用 DECLARE\_API 宏  
> 这个宏定义在 wdbgexts.h 中，  
> #define DECLARE\_API(s)                             \\  
>     CPPMOD VOID                                    \\  
>     s(                                             \\  
>         HANDLE                 hCurrentProcess,    \\  
>         HANDLE                 hCurrentThread,     \\  
>         ULONG64                dwCurrentPc,        \\  
>         ULONG                  dwProcessor,        \\  
>         PCSTR                  args                \\  
>      )  
> 如果是 32 位 dwCurrentPc 的类型就要换成 ULONG.  
> (2)def 文件  
> 必须输出 WinDbgExtensionDllInit.和 ExtensionApiVersion 两个函数。  
> 三．编译  
> 1．需要 Windows Driver Kit(WDK),必须在 windows 2003 server build environment 环境下。  
> 2．设置 DBGSDK\_INC\_PATH 和 DBGSDK\_LIB\_PATH 环境变量。  
> 3．切换到你的原文件目录下  
> 4．执行 build ?cZMg  
> 四．加载  
> 把编译出的含 DLL 的文件夹放到 WinDbg 安装目录下，运行 WinDbg 后用.load dllname 就可以了。然后就尽情的享受你自己编写的命令把。  
>   
> \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
> 实例：  
> 下来用一个主要是以 wdbgexts.h API 调试接口的例子。这个 dll 文件主要是自己实现 read(读指定地址值）,  
> edit(编辑指定地址的值),stack(取得堆栈值),help(显示以上三个命令的帮助信息).  
> (1)编写 simple.c 文件  
> (2)编写 simple.def 文件  
> (3)编写 sources 文件  
> (4)编写 makefile 文件  
> (5)编写 rc 资源文件  
> (6)编译并产生 simple.dll  
> (7)加载并执行自己的扩展命令  
> \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
> (1)c 文件  
> //  
> //头文件和宏定义  
> #include <windows.h>  
> #include "simple.h"  
>   
> #define KDEXT\_64BIT    
> #include <wdbgexts.h>  
> #include <ntverp.h>  
> //  
> // 全局变量  
> //  
> EXT\_API\_VERSION         ApiVersion = { (VER\_PRODUCTVERSION\_W >> 8), (VER\_PRODUCTVERSION\_W & 0xff), EXT\_API\_VERSION\_NUMBER64, 0 };  
> WINDBG\_EXTENSION\_APIS   ExtensionApis;  
> ULONG SavedMajorVersion;  
> ULONG SavedMinorVersion;  
> //  
> //主程序  
> //  
>   
> DllInit(  
>     HANDLE hModule,  
>     DWORD  dwReason,  
>     DWORD  dwReserved  
>     )  
> {  
>     switch (dwReason) {  
>         case DLL\_THREAD\_ATTACH:  
>             break;  
>   
>         case DLL\_THREAD\_DETACH:  
>             break;  
>   
>         case DLL\_PROCESS\_DETACH:  
>             break;  
>   
>         case DLL\_PROCESS\_ATTACH:  
>             break;  
>     }  
>   
>     return TRUE;  
> }  
>   
>   
> VOID  
> WinDbgExtensionDllInit(  
>     PWINDBG\_EXTENSION\_APIS lpExtensionApis,  
>     USHORT MajorVersion,  
>     USHORT MinorVersion  
>     )  
> {  
>     ExtensionApis = \*lpExtensionApis;  
>   
>     SavedMajorVersion = MajorVersion;  
>     SavedMinorVersion = MinorVersion;  
>   
>     return;  
> }  
>   
> LPEXT\_API\_VERSION  
> ExtensionApiVersion(  
>     VOID  
>     )  
> {  
>      
>     return &ApiVersion;  
> }  
>   
>   
> VOID  
> CheckVersion(  
>     VOID  
>     )  
> {  
>     return;  
> }  
> //  
> // 读目标双字值  
> //  
> DECLARE\_API( read )  
> {  
>     ULONG cb;  
>     ULONG64 Address;  
>     ULONG   Buffer\[4\];  
>   
>     Address = GetExpression(args);  
>   
>       
>     if (ReadMemory(Address, &Buffer, sizeof(Buffer), &cb) && cb == sizeof(Buffer)) {  
>         dprintf("%I64lx: %08lx %08lx %08lx %08lx\\n\\n", Address,  
>                 Buffer\[0\], Buffer\[1\], Buffer\[2\], Buffer\[3\]);  
>     }  
> }  
>   
> //  
> // 编辑目标的双字值  
> //    
> //    !edit <address> <value>  
> //  
> DECLARE\_API( edit )  
> {  
>     ULONG cb;  
>     ULONG64 Address;  
>     ULONG   Value;  
>   
>     if (GetExpressionEx(args, &Address, &args)) {  
>         Value = (ULONG) GetExpression( args);  
>     } else {  
>         dprintf("Usage:   !edit <address> <value>\\n");  
>         return;  
>     }  
>   
>       
>     if (WriteMemory(Address, &Value, sizeof(Value), &cb) && cb == sizeof(Value)) {  
>         dprintf("%I64lx: %08lx\\n", Address, Value);  
>     }  
> }  
>   
>   
> //  
> // 提取堆栈值  
> //  
> DECLARE\_API ( stack )  
> {  
>     EXTSTACKTRACE64 stk\[20\];  
>     ULONG frames, i;  
>     CHAR Buffer\[256\];  
>     ULONG64 displacement;  
>   
>   
>     // 在当前线程中获得堆栈祯值   
>     frames = StackTrace( 0, 0, 0, stk, 20 );  
>   
>     if (!frames) {  
>         dprintf("Stacktrace failed\\n");  
>     }  
>   
>     for (i=0; i<frames; i++) {  
>   
>         if (i==0) {  
>             dprintf( "ChildEBP RetAddr  Args to Child\\n" );  
>         }  
>   
>         Buffer\[0\] = '!';  
>         GetSymbol(stk\[i\].ProgramCounter, (PUCHAR)Buffer, &displacement);  
>           
>         dprintf( "%08p %08p %08p %08p %08p %s",  
>                  stk\[i\].FramePointer,  
>                  stk\[i\].ReturnAddress,  
>                  stk\[i\].Args\[0\],  
>                  stk\[i\].Args\[1\],  
>                  stk\[i\].Args\[2\],  
>                  Buffer  
>                  );  
>   
>         if (displacement) {  
>             dprintf( "+0x%p", displacement );  
>         }  
>   
>         dprintf( "\\n" );  
>     }  
> }  
>   
> /\*  
>   编写 help 扩展命令  
> \*/  
>   
> DECLARE\_API ( help )   
> {  
>     dprintf("Help for extension dll simple.dll\\n"  
>  "read  <addr>  - It reads and dumps 4 dwords at<addr>\\n"  
>  "edit  <addr> <val> - It modifies a dword value to <val> at <addr>\\n"  
>  "stack              - Printd current stack trace\\n"  
>  "help               - Shows this help\\n"  
>             );  
>   
> }  
>   
> \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
> (2)def 文件  
> ;--------------------------------------------------------------------  
> ;     
> ;   
> ; Module:  
> ;    simple.def  
> ;--------------------------------------------------------------------  
>   
> EXPORTS  
>   
> ;--------------------------------------------------------------------  
> ; These are the extensions exported by dll  
> ;--------------------------------------------------------------------  
>     read  
>     edit  
>     help  
>     stack  
>   
> ;--------------------------------------------------------------------  
> ;  
> ; these are the extension service functions provided for the debugger  
> ;  
> ;--------------------------------------------------------------------  
>   
>   
>     CheckVersion  
>     WinDbgExtensionDllInit  
>     ExtensionApiVersion  
> \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
> (3)source 文件  
> TARGETNAME=simple  
> TARGETPATH=obj  
> TARGETTYPE=DYNLINK  
>   
> DLLENTRY=\_DllMainCRTStartup  
>   
> !if "$(DBGSDK\_INC\_PATH)" != ""  
> INCLUDES = $(DBGSDK\_INC\_PATH);$(INCLUDES)  
> !endif  
> !if "$(DBGSDK\_LIB\_PATH)" == ""  
> DBGSDK\_LIB\_PATH = $(SDK\_LIB\_PATH)  
> !else  
> DBGSDK\_LIB\_PATH = $(DBGSDK\_LIB\_PATH)\\$(TARGET\_DIRECTORY)  
> !endif  
>   
> TARGETLIBS=$(SDK\_LIB\_PATH)\\kernel32.lib   
>   
> USE\_MSVCRT=1  
>   
> UMTYPE=windows  
>   
> SOURCES= simple.c   \\  
>          simple.rc  
> \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
> (4)makefile 文件  
> #  
> # DO NOT EDIT THIS FILE!!!  Edit .\\sources. if you want to add a new source  
> # file to this component.  This file merely indirects to the real make file  
> # that is shared by all the components of Windows  
> #  
> !INCLUDE $(NTMAKEENV)\\makefile.def  
> \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
> (5)rc 文件  
> #include <windows.h>  
> #include <ntverp.h>  
>   
> #define VER\_FILETYPE                VFT\_DLL  
> #define VER\_FILESUBTYPE             VFT2\_UNKNOWN  
> #define VER\_FILEDESCRIPTION\_STR     "Sample Debugger Extensions"  
>   
> #define VER\_INTERNALNAME\_STR        "simple.DLL"  
> #define VER\_ORIGINALFILENAME\_STR    "simple.DLL"  
>   
> #include "common.ver"  
> \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
> (6)编译并产生 simple.dll  
> i.切换到 windows server 2003 ddk 编译环境  
> ii.设置 windbg sdk 环境变量  
>    set DBGSDK\_INC\_PATH=%debuggers%\\sdk\\inc  
>    set DBGSDK\_LIB\_PATH=%debuggers%\\sdk\\lib  
>   %debuggers%要换成你自己的目录  
> iii.切换到包含源文件的文件夹，并执行 build -cZMg  
> \*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*  
> (7)加载并执行  
> .load %debuggers%\\simple  
> 然后就可以执行我们的扩展命令。   
>   
> 后记：最新版的 windbg 6.6.00007.5 的安装时，要选择 Custom,然后手工把 SDK 选中才会把 SDK 安装上。而且关于扩展命令编写的详细方法也集成到了帮助文档中。用 Windbg 还可以写.net 调试命令扩展，如 sos.dll，这一点是其他调试器所不具备的吧。 

 4dd466d0e1e0cc123afbe504da2462ca