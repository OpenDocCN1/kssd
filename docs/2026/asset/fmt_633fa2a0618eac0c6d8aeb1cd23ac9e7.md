
# [原创]LDR链调试手记

无论是编写ShellCode还是外壳程序，都需要动态的获取各个api的实际地址，最通用的方法之一，莫过于通过得到各个DLL模块的基址，再遍历其导出表。其中，获得各个模块基址中，通过PEB结构来获取的方法尤为的精简和通用。这里是我之前调试和学习时碰到的一些问题的总结，于是就有了这一篇手记。

## 1.认识LDR链

`FS`段寄存器作为选择子指向当前的活动线程的`TEB`结构`Thread Environment Block`注释。在`TEB`偏移的`0x30`处，就是指向`PEB` `Process Environment Block`注释② 的指针。而在`PEB`偏移的`0x0c`处是指向`PEB_LDR_DATA`结构的指针。`PEB_LDR_DATA`的结构如下：

```c
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

我们看到，在`PEB_LDR_DATA`结构中，又包含三个`LIST_ENTRY`结构体分别命名为:
`InLoadOrderModuleList`; 模块加载顺序
`InMemoryOrderModuleList`; 模块在内存中的顺序
`InInitializationOrderModuleList`; 模块初始化装载顺序

`LIST_ENTRY`其结构定义如下：

```c
typedef struct _LIST_ENTRY {
   struct _LIST_ENTRY *Flink;
   struct _LIST_ENTRY *Blink;
} LIST_ENTRY, *PLIST_ENTRY, *RESTRICTED_POINTER PRLIST_ENTRY;
```

微软是怎么解释`LIST_ENTRY`结构中成员作用的呢?来看看MSDN

The head of a doubly-linked list that contains the loaded modules for the process. Each item in the list is a pointer to an `LDR_DATA_TABLE_ENTRY` structure

这个双链表指向进程装载的模块，结构中的每个指针，指向了一个`LDR_DATA_TABLE_ENTRY` 的结构：

我们来看看这个结构的定义：

```c
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

可以根据上述的定义，画出一个直观寻址的示意图：

![](https://bbs.kanxue.com/upload/attach/201204/374528_bvu2h2xcmoqm7kk.jpg)

在定位到`PEB_LDR_DATA` 后，我们就可以根据之前介绍`LDR_DATA_TABLE_ENTRY`的结构，来定位每个模块的基址了。`PEB_LDR_DATA`地址知道了，那么，怎么根据它去找到`LDR_DATA_TABLE_ENTRY`呢？

前面提到过，`PEB_LDR_DATA` 中的三个`LIST_ENTRY`全部是双向链表结构，它的两个成员`Flink`，`Blink`都指向`LDR_DATA_TABLE_ENTRY`，那么，在真实的系统环境中，`PEB_LDR_DATA` 中的每一个`LIST_ENTRY`填充的是什么呢？与`LDR_DATA_TABLE_ENTRY`的关系又是怎么连接起来的呢？沐浴更衣，亮出神器`windbg`，一探究竟：

追本溯源，首先查看`PEB`结构:

```
0:000> !peb
PEB at 7ffd4000
    Ldr.InInitializationOrderModuleList:      00202988 . 00202cb0
    Ldr.InLoadOrderModuleList:            002028e8 . 00202db8
        Ldr.InMemoryOrderModuleList:         002028f0 . 00202dc0
```

于此同时也获得各个加载模块的基地址：

```
        400000 3db8972c Oct 25 08:58:20 2002 C:\Users\JoRrYChEn\Desktop\test.exe
       77130000 4ec49caf Nov 17 13:33:35 2011 C:\Windows\SYSTEM32\ntdll.dll
       76e00000 4e21132a J1ul 16 12:27:22 2011 C:\Windows\system32\kernel32.dll
       751b0000 4e21132b Jul 16 12:27:23 2011 C:\Windows\system32\KERNELBASE.dll
```

我们看到，`Ldr`中有的三个`List`，根据MSDN规定`LIST_ENTRY`的结构，其中有两个成员：`Flink`和`Blink`，他们分别指向着一个`LDR_DATA_TABLE_ENTRY` 结构。各个`LIST_ENTRY`的第一个填充就是链表的`Flink`,指向下一个链表节点，第二个填充的是链表的`Blink`，指向前一个链表节点。那么根据`windbg`提供的`PEB_LDR_DATA` 中的三个`LIST_ENTRY`的`Flink`，来看看其填充的各项值：

`InLoadOrderModuleList`: `Flink`：`00202988` . `Blink`：`00202cb0`

```
0:000> dd 002028e8
002028e8  00202978 7720788c 00202980 77207894
002028f8  00000000 00000000 00400000 0040720b
0:000> dd 00202978
00202978  00202ca0 002028e8 00202ca8 002028f0
00202988  00202dc8 7720789c 77130000 00000000
0:000> dd 00202ca0
00202ca0  00202db8 00202978 00202dc0 00202980
00202cb0  7720789c 00202dc8 76e00000 76e51065
0:000> dd 00202db8
00202db8  7720788c 00202ca0 77207894 00202ca8
00202dc8  00202cb0 00202988 751b0000 751b7afd
0:000> dd 7720788c
7720788c  002028e8 00202db8 002028f0 00202dc0
7720789c  00202988 00202cb0 00000000 00000000
```

我们看到`002028e8`所在的模块地址是`00400000`，是原始模块加载基地址，它的`Flink`是`00202978` ，所在的模块地址`77130000` ，是`ntdll.dll`的加载基地址，它的`Flink`是`00202ca0` ，所在的模块地址是`76e00000` ，是`kernel32.dll`的加载基地址，它的`Flink`是`00202db8` ，所在模块地址是`751b0000`，是`KERNELBASE.dll`的加载基地址。之后，便是一个循环，`Flink`又指向了`002028e8`。不难发现，这是个循环链表！

同理，我们依此遍历`InMemoryOrderModuleList` 和 `InInitializationOrderModuleList`两条链：

`Ldr.InMemoryOrderModuleList`:

```
0:000> dd 002028f0
002028f0  00202980 77207894 00000000 00000000
00202900  00400000 0040720b 00008010 00460044
0:000> dd 00202980
00202980  00202ca8 002028f0 00202dc8 7720789c
00202990  77130000 00000000 0013c000 003c003a
0:000> dd 00202ca8
00202ca8  00202dc0 00202980 7720789c 00202dc8
00202cb8  76e00000 76e51065 000d4000 00420040
0:000> dd  00202dc0
00202dc0  77207894 00202ca8 00202cb0 00202988
00202dd0  751b0000 751b7afd 0004a000 00460044
0:000> dd 77207894
77207894  002028f0 00202dc0 00202988 00202cb0
772078a4  00000000 00000000 00000000 00000000
```

`InInitializationOrderModuleList`:

```
0:000>dd 00202988
00202988  00202dc8 7720789c 77130000 00000000
0:000> dd 00202dc8
00202dc8  00202cb0 00202988 751b0000 751b7afd
0:000> dd 00202cb0
00202cb0  7720789c 00202dc8 76e00000 76e51065
0:000> dd 7720789c
7720789c  00202988 00202cb0 00000000 00000000
```

同`InLoadOrderModuleList` 一样，最终，这也是双向的循环链表结构。根据我们遍历出来的结构，我们可以画出如下的示意图（test文件为例）：

![](https://bbs.kanxue.com/upload/attach/201204/374528_yixkmzpn72ff8ki.jpg)

不难发现，`PEB_LDR_DATA`给出的是三个`List`（`InLoadOrderModuleList`，`InMemoryOrderModuleLis`以及`InInitializationOrderModuleList`）模块加载首个基址，也可以看成是整个`List`双向链表的表头，然后通过这个双向循环链表的不断的遍历，来依此获取不同`List`加载的顺序。同时系统为每一个`DLL`维护的一个`LDR_DATA_TABLE_ENTRY`，该结构中，每一个`DLL`在不同`List`中，不但包含着着前继加载模块或者后继加载模块，还有着非常详细的各个加载模块的信息，包括加载基址和`DLL`名称等等。

这样，我们可以关于之前根据`PEB_LDR_DATA` 后找到`LDR_DATA_TABLE_ENTRY`就非常的显而易见了：

![](https://bbs.kanxue.com/upload/attach/201204/374528_nkco54wwgmlen8b.jpg)

通过不断地遍历，读取其中的各项结构，至此，我们可以得出每一个`List`的在测试系统(`win7`，目标`EXE`文件 `test`)下加载的依此顺序：

**遍历 `InInitializationOrderModuleList` 得到次序：

`NTDLL.DLL`-> `KERNELBASE.DLL`->`KERNEK32.DLL`->`NULL`

遍历 `InLoadOrderModuleList` 得到次序

`TEST.EXE`->`NTDLL.DLL`-> `KERNEK32.DLL`->`KERNELBASE.DLL`->`NULL`

遍历 `InMemoryOrderModuleList` 得到次序

`TEST.EXE`->`NTDLL.DLL`-> `KERNEK32.DLL`->`KERNELBASE.DLL`->`NULL`**

## 2.PEB_LDR_DATA链表的应用

我们都知道，以前在`xp`下写`shellcode`，最通用获得`kernel32`基址代码：

```
mov ebx, fs:[ 0x30 ]       // 获得PEB
mov ebx, [ ebx + 0x0C ]    // 获得PEB_LDR_DATA
mov ebx, [ ebx + 0x1C ]    // InitializationOrderModuleList 第一项
mov ebx, [ ebx ]           // InitializationOrderModuleList 第二项
mov ebx, [ ebx + 0x8 ]    // 获得完整的路径地址
```

但是这段代码放到`win7`就不通用， 为什么？前三句没问题，首先是获得`PEB`基地址，然后在偏移`0X0C`处获得`LDR`，然后再取得`LDR`的偏移 `0x1C`处，获得`InInitializationOrderModuleList`加载的下一个模块基址，恰恰问题是出在第5句上，`mov ebx,[ebx]`，此时，`ebx`中的值按照前面所述，应该是第二个`LDR_DATA_TABLE_ENTRY`，可是根据之前的观察，在`win7`的环境之下，`InInitializationOrderModuleList`第二个`LDR_DATA_TABLE_ENTRY`不是`kernel32`了，而是，`KERNELBASE`了(见图2)，`kernel32`排到了第三个了。所以，若要获取`win7`下`kernel32`的基址，只需将之前的代码略加修改：

```
mov ebx, fs:[ 0x30 ]       // 获得PEB
mov ebx, [ ebx + 0x0C ]    // 获得PEB_LDR_DATA
mov ebx, [ ebx + 0x1C ]    // InitializationOrderModuleList第一项
mov ebx, [ ebx ]           // InitializationOrderModuleList第二项
mov ebx, [ ebx ]           // InitializationOrderModuleList第三项
mov ebx, [ ebx + 0x8 ]    // 获得完整的路径地址
```

当然，若要想写一个通用的`shellcode`，就必须先要判断系统版本号再根据情况来确定是取链的第二项还是第三项。除开现有的字符串查找方法，有木有一个更为快捷的方法呢？我们试着来探讨一下：

前辈们的经验告诉我们，`winXp`下，`InitializationOrderModuleList` 加载的`DLL`模块顺序是固定的，通过遍历其链，`kernel32.dll`必然是在第二位。但是在`win7`条件下，这个`xp`下固定的“第二位”就已经换成了`KERNELBASE.DLL`了。那么另外两条`InMemoryOrderModuleList`和 `InLoadOrderModuleList`是不是模块加载顺序也变了呢？是不是也能像类似于查找`InitializationOrderModuleList`顺序链的方式找到通用的加载顺序呢？

带着疑问，写一个遍历`PEB.LDR.InLoadOrderModuleList`程序验证一下：

`Win2000+Sp4`下：

![](https://bbs.kanxue.com/upload/attach/201204/374528_vo871nbqh5mr9it.jpg)

`winxp+sp3`
