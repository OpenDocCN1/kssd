
# 使用 VMWare GDB 和 IDA 调试 Windows 内核

> 原文：[www.pediy.com/kssd/pediy12/135229.html](https://www.pediy.com/kssd/pediy12/135229.html)

最近喜欢用 IDA 搞一些内核的东西，于是就到处找 IDA 关于内核方面的东东![](img/d23858dd221cce728a3a61b46badb044.jpg "恐怖!")。这篇文章实在原文的基础上进行了一定的封装，也算是半原创的东东吧～希望大家不要拍砖撒～![](img/afde35a5f3b3a4675abb3221cf07c6d1.jpg "124.gif")

VMWare 的 GDB 调试器功能比较简单也比较基础，该调试器并不知道处理器和线程的任何信息（对于 Windows 系统），因而如果想要得到一些高等级的信息，我们需要自己做一些额外的工作。本文主要讲解了如何使用 IDAPython 脚本来让 IDA 处理已经加载的模块列表和加载符号库。

### 设置 VM 来进行调试

在进行这一步之前首先要保证你已经有了一个已经安装好的 Windows（32 位）的操作系统。在开始调试之前，首先要拷贝你想要看到符号的模块到系统目录下，如果你不确定要复制那些文件，可以将如下的文件复制到虚拟机目录下：位于 `System32` 目录下的 `nt\*.exe` 和 `hal.dll` 文件、整个 `System32\drivers` 目录。在这里我将文件复制到了 `E:\虚拟机系统\Windows 7\Shar4ed dll\`目录下。

编辑虚拟机的`.vmx` 文件来激活 GDB 调试器功能：

![](img/af136c7441cf9961daec8ef28c63d34b.jpg)

图 01

在文档末尾加入如下两行（VM 更多的功能可以查看 VM 的相关文档 http://wiki.osdev.org/VMWare）：

```
debugStub.listen.guest32 = "TRUE"
debugStub.hideBreakpoints= "TRUE"
```

修改之后保存文件并且启动虚拟机，等待虚拟机启动。

![](img/a89005fd6bae14d1132410f6ddb8056b.jpg)

图 02

虚拟机启动之后启动 IDA，如果出现如图 03 所示的提示窗口则直接点 GO 进入程序界面即可。

![](img/b218826759aa9120a15da472699d7de2.jpg)

图 03

进入主界面之后执行菜单的 `Debugger->Attach->Remote GDB Debugger` 如图 04 所示。

![](img/26d64151e8115d0455076400159503ec.jpg)

图 04

打开如图 05 所示的调试器附加窗口。

![](img/0957f8eb005d4b6435b871ea8467df73.jpg)

图 05

在 `Hostname` 中输入 `localhost`，端口输入 `8832`。点击确定之后将会打开如图 06 所示的进程选择窗口。

![](img/b5cee82e901a74467dd8dab8963a1bae.jpg)

图 06

选择 ID 为 `0` 的进程进行附加，如果附加成功将会弹出如图 07 所示的提示信息。

![](img/8648c338dc0bd60c2a434fe3c251d43c.jpg)

图 07

点击 `OK` 之后将会中断在如图 08 所示的代码处。

![](img/dd7898ce533f9db73ee8768bdea279f1.jpg)

图 08

此时将会中断在内核中地址大于 `0x80000000` 的地方，现在就可以进行单步调试了，但是没有任何的名称调试起来是非常不爽的，那么我们就来收集更多的信息让 IDA 的显示看起来更加的直观。

### 获取内核模块列表

内核模块列表保存在一个有 `PsLoadedModuleList` 符号指向的内核列表中。为了获取这个列表的地址就要用到 `KPCR` 的方法，`KPCR` 的全称是 Kernel Processor Control Region。内核用这个区域来存储每个处理器所包含的各种信息。它被放置在 `fs` 寄存器指向的区段中（类似于应用层中的 `TEB`）。它有一个区域叫做 `KdVersionBlock`，这个区域指向了内核调试使用的一个结构体。而这个结构体则包含了各种内存结构的指针，其中就包括 `PsLoadedModuleList`。

#### KPRC 的定义

这个结构体可以在很多地方找到，其中 IDA 的 `ntddk.til` 文件中也有这个结构的定义。现在我们只需要知道 `KdVersionBlock` 位于 `KPRC` 结构体的的 `0x34` 处，并且它指向了 `DBGKD_GET_VERSION64`。在 `DBGKD_GET_VERSION64` 的偏移量 `0x18` 处则可以找到 `PsLoadedModuleList`。

现在我们就可以写一个小的 Python 函数来找到这个指针的的值。为了得到 `fs` 指向的区段的基址我们可以使用 VMWare 的`r`调试命令。GDB 调试器插件注册了一个 IDC 函数，叫做`SendGDBMonitor()`来发送命令到监视器，所以我们可以使用 IDAPython 的`Eval()`函数来调用它。

```python
fs_str = Eval('SendGDBMonitor("r fs")')
```

返回的数据格式如下所示：

```
fs 0x30 base 0x82744a00 limit 0x00002008 type 0x3 s 1 dpl 0 p 1 db 1
```

我们需要的是在 `base` 标记之后的数值。

```python
kpcr = int(fs_str[13:23], 16) #extract and convert as base 16 (hexadecimal) number
```

然后来获取 `KdVersionBlock` 的数值：

```python
kdversionblock = Dword(kpcr+0x34)
```

最后我们来获取 `PsLoadedModuleList` 的地址：

```python
PsLoadedModuleList = Dword(kdversionblock+0x18)
```

#### 遍历内核模块

现在就可以根据上面的地址来遍历内核模块了，`PsLoadedModuleList` 被声明为 `PLIST_ENTRY`。`PLIST_ENTRY` 的定义如下所示（双向链表）：

```c
typedef struct _LIST_ENTRY
{
    PLIST_ENTRY Flink;
    PLIST_ENTRY Blink;
} LIST_ENTRY, *PLIST_ENTRY;
```

所以要遍历所有的模块只需要跟随 `Flink` 指针，直到我们回到开始的地方就可以了。每一个模块的结构定义如下所示：

```c
struct LDR_MODULE
{
  LIST_ENTRY InLoadOrderModuleList;
  LIST_ENTRY InMemoryOrderModuleList;
  LIST_ENTRY InInitializationOrderModuleList;
  PVOID BaseAddress;
  PVOID EntryPoint;
  ULONG SizeOfImage;
  UNICODE_STRING FullDllName;
  UNICODE_STRING BaseDllName;
  ULONG Flags;
  SHORT LoadCount;
  SHORT TlsIndex;
  LIST_ENTRY HashTableEntry;
  ULONG TimeDateStamp;
};
```

现在我们可以来编写一个小函数遍历这个链表并且为每个模块创建一个区段。

```python
#get the first module
cur_mod = Dword(PsLoadedModuleList)

while cur_mod != PsLoadedModuleList and cur_mod != BADADDR:
    BaseAddress  = Dword(cur_mod+0x18)
    SizeOfImage  = Dword(cur_mod+0x20)
    FullDllName  = get_unistr(cur_mod+0x24)
    BaseDllName  = get_unistr(cur_mod+0x2C)
    #create a segment for the module
    SegCreate(BaseAddress, BaseAddress+SizeOfImage, 0, 1, saRelByte, scPriv)
    #set its name
    SegRename(BaseAddress, BaseDllName)
    #get next entry
    cur_mod = Dword(cur_mod)
```

### 加载符号库

已经能够获取内核模块列表固然不错，但是如果不能加载符号库那么上面的工作也就没有多少用处。我们可以通过 IDA 的 `File->LoadFile->PDB file` 手动为每个模块加载符号库，但是这样做太蛋疼了。为什么不让它自动加载呢？

为了达到我们的目的这里就要用到 PDB 插件，通过查阅源代码（SDK）我们可以发现它支持下面的三个调用代码：

```c
//call_code==0: user invoked ‘load pdb’ command, load pdb for the input file
//call_code==1: ida decided to call the plugin itself
//call_code==2: load pdb for an additional exe/dll
//             load_addr: netnode("$ pdb").altval(0)
//             dll_name:  netnode("$ pdb").supstr(0)
```

第二个调用代码看起来正是我们需要的功能。但是，当前版本的 IDAPython 只包含一个非常基础的节点类，所以没有办法是用 Python 来实现这个功能。但是我们如果查看其它的调用代码可以看到这个插件重定义了模块的基址（通过`$ PE header`）和模块的路径（通过`get_input_file_path()`），并且我们可以通过`set_root_filename()`函数来设置输入文件的路径。同样如果我们使用第三个调用代码那么我们要避免出现“你是否想要加载符号库？”（"Do you want to load the symbols?"）的提示。

```python
#new netnode instance
penode = idaapi.netnode()

#create netnode the in database if necessary
penode.create("$PE header")

#set the imagebase (-2 == 0xFFFFFFFE)
penode.altset(0xFFFFFFFE, BaseAddress)

#set the module filename
idaapi.set_root_filename(filename)

#run the plugin
RunPlugin("pdb",3)
```

但是我们需要用使用前面的文件路径（想要看到符号库的文件）来取代内核模块路径：

```python
#path to the local copy of System32 directory
local_sys32 = r" E:\虚拟机系统\Windows 7\Shar4ed dll"

if FullDllName.lower().startswith(r"\\systemroot\\system32"):
    #translate into local filename
    filename = local_sys32 + FullDllName[20:]
```

也可以直接运行`vmmodules.py` 脚本，在执行的过程中将会弹出如图 09 所示的提示窗口。

![](img/75a707f10a82c232367514e2d75f9ee0.jpg)

图 09

如果 `Input file` 的路径不对则需要手工进行修改，点击确定之后将会弹出如图 10 所示的符号库加载确认窗口。

![](img/1afb6e53eaefde79484caf495a700abb.jpg)

图 10

点击 `yes` 之后就开始加载符号库了，但是比较繁琐的是每次加载一个新的模块的符号库时都会弹出图 09 种的确认窗口，直到所有的模块的符号加载完毕。这也是个痛苦的过程～

在执行完脚本之后再来看下程序的内存镜像将会是如图 11 所示。

![](img/35c7cd9cbb19ecd8ec561f50b8a1250f.jpg)

图 11

再来看下名称窗口可以发现只要加载符号库的模块中的名称都已经显示出来了，如图 12 所示。

![](img/fbaea9257d5aed01a70124374a0df36f.jpg)

图 12

现在看起来就好多啦，调试愉快！

PDF 和 Python 脚本见附件！

`Advanced Windows Kernel Debugging with VMWare and IDA’s GDB debugger.rar`
