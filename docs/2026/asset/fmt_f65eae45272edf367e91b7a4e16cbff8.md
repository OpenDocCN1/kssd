
# [原创]分析一款击键木马(shellcode直接注入)

[toc]

(看不懂IDA shellcode分析的看我之前的文章<<病毒分析之shellcode的分析方法>>)

## MD5

![](upload/attach/202011/830337_SQWW5VQPDESZT25.png)

## 查壳

![](upload/attach/202011/830337_6NRH4UMKFYBQUE9.png)

![](upload/attach/202011/830337_RAT7K94E898FAYK.png)

无壳

该病毒包含一个`exe`,一个`dll`

## 基础静态分析

![](upload/attach/202011/830337_FMWRXGW4YEV96QX.png)

`cmd.exe`和`command.com`表明也许用到了shell进程

![](upload/attach/202011/830337_WU7H9952ZVDP8SP.png)

`IsBadReadPtr`用于验证一个进程是否对一块内存具有读取权限，这里看到了`MapViewOfFile`和`UnMapViewOfFile`表明程序用到了内存映射文件进行进程间通信。`GetFileSize`,`CreateFileA`,`CopyFileA`表明创建了一个文件。`GetCommdandLineA`读取了命令行参数控制程序的执行，`GetModuleFileNameA`读取了可执行文件的路径，可能用来安装病毒。

![](upload/attach/202011/830337_4C2HUUBS2YVCM3C.png)

![](upload/attach/202011/830337_5UR9M39RXBF4Y7R.png)

该病毒在`C:\WINDOWS\System32\`目录下安装了`inet_epar32.dll`，`zzz69806582`可能是个加密字符串，可能是命令的加密，`net start cisvc`启动了索引服务

下面来看下`dll`

![](upload/attach/202011/830337_J5URGGGEF3X3GMS.png)

`GetActiveWindow`用于获取最上层窗口的句柄，字符串`"dddd,MMMM dd,yyyy"`有加密特征，接着就是一些月份，我们不知道这些月份用来干嘛的

还有周一到周日

![](upload/attach/202011/830337_K356MZ9T952MDXF.png)

`CreateMutexA`创建互斥量，可能是为了在同一时刻只允许系统中运行一个病毒的实例`CreateThread`创建了一个新的线程，`GetWindowTextA`获取窗口的输入，`GetForegroundWindow`获取最上层的窗口，`GetAsyncKeyState`扫描键盘的状态，`GetCommandLineA`从命令行获取命令控制病毒的行为

![](upload/attach/202011/830337_WKWJ2CS5ZDCSG3R.png)

这里和`exe`中看到了同一个字符串`zzz69806582`,`<SHIFT>`这个按键

很明显，这是一个击键记录器

看下`exe`导入了哪些函数

![](upload/attach/202011/830337_XHVVUB24KZXSSQM.png)

这里，和我们在`strings`下看到的一样

![](upload/attach/202011/830337_TYFMSMHH87QNY2K.png)

这里，`dll`导出了`zzz69806582`这个函数，原来不是一个字符串

![](upload/attach/202011/830337_BYY9Q69X9HQRV3B.png)

`USER32.DLL`中只导入了三个函数，我们更加确信这是一个击键记录器

## 基础动态分析

打开`procmon`

![](upload/attach/202011/830337_D6U4M2VS35ZKF22.png)

这里我们看到该病毒在`C:\WINDOWS\System32`目录下创建了`inet_epar3.dll`这个文件，用`LoadPE`比较一下

![](upload/attach/202011/830337_3QZ2MGTEFC9YB26.png)

![](upload/attach/202011/830337_MWRHWY4G659EWC4.png)

该病毒获取了`cisvc`的句柄，然后启动了命令行

![](upload/attach/202011/830337_ZBX3W5YR44XWUEZ.png)

最后，通过发送`net start cisvc`来启动索引服务。

用`procexp`查看进程树

![](upload/attach/202011/830337_C4WRXXY9APW9ARG.png)

`cisvc`服务正在运行

## 高级动静态分析

将可执行文件载入`IDA`

![](upload/attach/202011/830337_Z282Z8JP824K5NR.png)

病毒将`async.dll`复制到`C:\WINDOWS\System32\`系统目录中,接着创建字符串`C:\WINDOWS\System32\cisvc.exe`并作为`sub_401070`的参数,接着调用系统命令`net start cisvc`启动索引服务.`enter`跟进`sub_401070`

![](upload/attach/202011/830337_2HS6PRUV4GS8VKH.png)

`cisvc.exe`被映射到内存,`MapViewOfFile`返回内存映射视图的基址,`UnmapViewOfFile`调用之后对该文件的所有修改都会被写入硬盘,我们看看病毒向该程序写入了什么

![](upload/attach/202011/830337_MK42E69D5PPETW3.png)

`lpBaseAddress`表示`cisvc.exe`映射到内存的基址,该基址的`var_28`偏移处是将要被写入的位置,写入`4E*4=312`个字节,写入的内容是`byte_409030`处的内容,双击进入

### shellcode注入分析

这里是一段`shellcode`,按`C`将其转换为代码

(看不懂IDA shellcode分析的看我之前的文章<<病毒分析之shellcode的分析方法>>)

![](upload/attach/202011/830337_C4WWH87UUHD9TCH.png)

这就是将要注入到`cisvc.exe`的代码,一直到`409168`

![](upload/attach/202011/830337_3SQ6927HVKFJ47K.png)

![](upload/attach/202011/830337_3E8T9F5J2THUKMA.png)

这里，开头直接跳到了`loc_409134`处

![](upload/attach/202011/830337_T4U5J4NF6AC4KP4.png)

调用的第一个函数是`sub_4090dd`

查看引用

#### IDA 静态分析shellcode

![](upload/attach/202011/830337_PT622CUTUQ2BWWV.png)

该函数直接调用了两个函数，`sub_4090bc`和`sub_40905f`

![](upload/attach/202011/830337_H7SS6RUH5W7GC83.png)

`4090dd`的开头是`pop ebx`,这里`ebx`保存的是字符串`"C:\WINDOWS\System32\inet_epar32.dll"`以及导出函数名`"zzz69806582"`的地址。接着调用了`sub_4090bc`

![](upload/attach/202011/830337_UFDYZZ6VD898C4P.png)

首先清零`eax`,接着访问`PEB`结构（`mov eax,fs:[eax+30]`),`fs`指向当前活动线程的`TEB`结构，，其中偏移`30`处是指向`PEB`结构的指针,`PEB`结构偏移`0ch`处是指向`PEB_LDR_DATA`结构的指针,`PEB_LDR_DATA`结构的`0x1c`偏移处存放着一些指向动态链接库信息的链表地址，第一个是`ntdll.dll`

`PEB_LDR_DATA`结构如下

```
typedef struct _PEB_LDR_DATA
{
	ULONG Length; // +0x00
	BOOLEAN Initialized; // +0x04
	PVOID SsHandle; // +0x08
	LIST_ENTRY InLoadOrderModuleList; // +0x0c
	LIST_ENTRY InMemoryOrderModuleList; // +0x14
	LIST_ENTRY InInitializationOrderModuleList;// +0x1c
} PEB_LDR_DATA,*PPEB_LDR_DATA; // +0x24
```

`LIST_ENTRY`结构定义如下

```
typedef struct _LIST_ENTRY {
struct _LIST_ENTRY *Flink;
struct _LIST_ENTRY *Blink;
} LIST_ENTRY, *PLIST_ENTRY, *RESTRICTED_POINTER PRLIST_ENTRY;
```

这个双链表指向进程装载的模块，结构中的每个指针指向一个`LDR_DATA_TABLE_ENTRY`结构

```
typedef struct _LDR_DATA_TABLE_ENTRY
{
LIST_ENTRY InLoadOrderLinks;
LIST_ENTRY InMemoryOrderLinks;
LIST_ENTRY InInitializationOrderLinks;
PVOID DllBase;
PVOID EntryPoint;
ULONG SizeOfImage;
UNICODE_STRING FullDllName;
UNICODE_STRING BaseDllName;
ULONG Flags;
WORD LoadCount;
WORD TlsIndex;
union
{
LIST_ENTRY HashLinks;
struct
{
PVOID SectionPointer;
ULONG CheckSum;
};
};
union
{
ULONG TimeDateStamp;
PVOID LoadedImports;
};
_ACTIVATION_CONTEXT * EntryPointActivationContext;
PVOID PatchInformation;
LIST_ENTRY ForwarderLinks;
LIST_ENTRY ServiceTagLinks;
LIST_ENTRY StaticLinks;
} LDR_DATA_TABLE_ENTRY, *PLDR_DATA_TABLE_ENTRY;
```

`mov esi,[eax+1ch]`
`lodsd`将`LDR_TABLE_ENTRY.InInitializationOrderLinks.Flink`的值给`eax`
`mov eax,[eax+8]` `eax`获得`LDR_DATA_TABLE_ENTRY.DllBase`的值

这段代码的作用是在内存中找到`kernel32.dll`

![](upload/attach/202011/830337_9YRVZDW247BKRFA.png)

这里，`kernel32.dll`的基址指针和`753a4fc`这个常量做为`sub_40905f`的参数被压栈，并且调用了两次`sub_40905f`

![](upload/attach/202011/830337_XS5W4GTXZ94ZNC3.png)

这里对参数进行重命名

![](upload/attach/202011/830337_HRPNYSMCVN5Z6U9.png)

在`sub_40905f`中调用了`sub_40903e`,enter

![](upload/attach/202011/830337_KMRV73RW7HQ78N4.png)

这里是一个解码循环

我们开始分析`sub_40905f`这个函数

![](upload/attach/202011/830337_ST7WJSXJKH4SDXW.png)

`409060`到`40906d`开始解析`kernel32.dll`这个`PE`文件，并获取了指向`PE`特征域的指针

`40906d`到`409079`加上了正确的偏移，获得了指向`IMAGE_EXPORT_DIRECTORY`的指针,`edx:=IMAGE_EXPORT_DIRECTORY`的地址

`409073`处开始解析`IMAGE_EXPORT_DIRECTORY`结构体，加载`NumberOfNames`值及`AddressOfNames`指针

`AddressOfName`中的每个字符串指针都被传递到`sub_40903e`这个函数中，然后将这个计算结果与`409092`处传递给这个函数的参数进行比较

![](upload/attach/202011/830337_PB7PP48G9NVGWJ4.png)

一旦`AddressOfNames`的正确索引被找到，它将作为位置`40909d`处的`AddressOfNameOrdinals`数组的索引，来获得对应的序号的数值，这个值被当作`4090a6`处的`AddressOfFunctions`数组的索引使用，这是病毒想要的值，它在`4090b4`处被写到了栈上，这里我们已经知道`sub_40905f`函数的作用：根据哈希值获取导出符号名，第一个`kernell32.dll`的导出函数地址被存放在`[ebp-4]`中，第二个函数被存放在`[ebp-0ch]`中，接着调用了第一个函数，第二个函数，如图所示

![](upload/attach/202011/830337_E7JUGXNNWVXBYP8.png)

在`shellcode`的末尾出现了`inet_epar32.dll`路径和导出函数名，说明这个`shellcode`加载了这个`DLL`并调用了它的导出函数

我们知道`cisvc.exe`被注入了`shellcode`但是用`IDA`我们看不出来该病毒导出了`kernell32.dll`的哪两个函数（其实第一个我们可以猜到是`LoadLibrary`），我们恢复虚拟机到快照查看一下`cisvc.exe`感染前后的变化

![](upload/attach/202011/830337_K42JGMJVC6PHD28.png)

![](upload/attach/202011/830337_SMHA9SAT4T7K29V.png)

### OD动态分析shellcode

`shellcode`修改了`cisvc.exe`的入口点，载入`OD`

在调用`kernel32.dll`第一个导出函数的地方下端断（忘了的看上面`IDA`的分析）

![](upload/attach/202011/830337_RKYVPHAHHYNQWJH.png)

`F9`运行,在第一个函数处断了下来,如图

![](upload/attach/202011/830337_XRBQX9D79RKQ4YK.png)

我们猜对了,这里调用的第一个函数是`LoadLibraryExA`

往下5行,是第二个函数的调用,`GetProcAddress`

这里我们看到`LoadLibraryExA`的第三个参数在`eax`中,我们在数据窗口中右键选择

`go expresion`

![](upload/attach/202011/830337_Y59YY2PYCGGTKCM.png)

这里我们看`eax`存储的是字符串`C:\WINDOWS\System32\inet_epar32.dll.zzz69806582`

`LoadLobraryExA`将`inet_epar32.dll`载入内存.

在`1001b1b`处,该病毒将`zzz69806582`作为参数调用`GetProcAddress`,获取导出函数的地址最后一次函数调用是什么

在`1001b21`处下断及`call dword ptr ss:[ebp-0x8]`处下断,我们来看看该处是什么调用

![](upload/attach/202011/830337_9N33A5J22CTRXRE.png)

如图,这里调用的是`inet_epa.zzz69806582`

最后`1001b27`处执行`jmp cisvc.100129b`跳转到`cisvc`原始的入口点.

接下来分析键盘记录器

来到`StartAddress`

![](upload/attach/202011/830337_5HTFPDX5ZHRN4B5.png)

该病毒调用`CreateMutexA`创建了一个名为`MZ`的互斥量,这个互斥量避免了同一时刻`PC`上有多个病毒实例在运行,如果该互斥量已经存在则`OpenMutexA`会终止这个线程(开头),创建互斥量之后在`100014d4`处调用`CreateFileA`创建了`kernel64x.dll`的文件,获取该文件的句柄后调用`SetFilePointer`将文件指针指向末尾

![](upload/attach/202011/830337_PQDVV7BQ2TAZWQM.png)

之后调用了`sub_10001380`这个函数,双击进入

![](upload/attach/202011/830337_Z27DWTWB726GWAZ.png)

`sub_10001380`调用了四个函数,分别是:

`sub_10001030`,`sprintf`,`WriteFile`,`Sleep`

双击进入`sub_10001030`

`sub_10001030`调用了四个函数,分别是

`sub_10001000`,`mbsnbcmp`,`GetAsyncKeyState`,`sprintf`

先看`sub_10001000`

![](upload/attach/202011/830337_8UBVPHN7BBQZJN2.png)

`GetForegroundWindow`识别当前聚焦的窗口(即正在输入的窗口),它告诉击键记录器当前哪个应用程序正在执行输入,然后获取窗口的标题

![](upload/attach/202011/830337_Q6Q9PGTVW7MT7J4.png)

这里先判断`<shift>`键有没有按下,`10h`对应`<shift>`键,接着判断大写锁定键有没有按下,对应`14h`

接下来的部分是一个大的循环结构用来遍历所有的按键

![](upload/attach/202011/830337_VANXSKV5FZ6TAF9.png)
