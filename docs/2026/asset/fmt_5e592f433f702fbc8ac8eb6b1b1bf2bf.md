
# [原创]SYSENTER-HOOK

# SYSENTER-HOOK

## 实验环境
win7虚拟机

示例是对内核层进行SYSENTER-HOOK实现保护进程的功能

SYSENTER-HOOK也成称为KiFastCallEntry-Hook，它相当于是内核层的Inline-Hook，通过修改`SYSENTER_EIP_MSR`寄存器使其指向我们自己的函数，那么我们就可以在自己的的函数中对所有来自3环的函数调用进行第一手过滤。

一会儿会用到的API是`OpenProcess`，在3环也就是用户层调用此API它保存一些信息传入到0环内核层后实际调用的是`ZwOpenProcess`函数，所以先使用OD随意打开一个.exe可执行程序,然后在kernel32模块里面查找`OpenProcess`函数，经过2个jmp后进入下一层，找到一个call进入其中便是`ZwOpenProcess`的调用。

![](upload/attach/201906/847228_W3Z2MPZFSPUUTH6.jpg)

![](upload/attach/201906/847228_GVP585A4VJW2J2B.jpg)

下面调用API代码才是我们需要注意的，每个API函数调用时在进入关键函数之前都有：

* 一句汇编代码`mov eax,0xXX`，这是用eax保存一个调用号。
* `mov edx,0x7ffe0300`，这用edx保存的是`KiFastSystemCall`函数

![](upload/attach/201906/847228_9JFEESFDCXEFNCJ.jpg)

* 调用号

调用号的作用就相当于给函数指定一个序号，通过这个序号就能在内核层中找到我们要调用的函数。进入0环时调用号是eax传递的，但这个调用号并不只是一个普通的数字作为索引序号，系统会把他用32位数据表示，拆分成19：1：12的格式，如下：

![](upload/attach/201906/847228_A7PUDM7BSR76CHM.jpg)

其中0-11这低12位组成一个真正的索引号，第12位表示服务表号，13-31位没有使用。

* `KiFastSystemCall`函数

而`0x7ffe0300`保存的`KiFastSystemCall`函数的地址，因为下一条指令`SYSENTER`就是进入内核层，由于每个线程都有一套线程上下文，都有一个独立的栈.。进入到内核后，内核也会使用自己的内核栈，所以这里先用edx保存栈顶esp。

![](upload/attach/201906/847228_BMQSQ2EHUE8NQ4G.jpg)

那么就要关注一个最重要的问题, `SYSENTER` 执行之后就进入到内核层, 那到底进入到内核层的哪个地址? 要想弄明白这个问题就需要了解执行 `SYSENTER` 指令系统到底做了些什么。

首先，`SYSENTER` 执行的时候，会读取三个特殊寄存器,从这三个特殊寄存器中取出内核栈的栈顶( esp ) ，内核代码段段选择子( cs ) ，以及代码的首地址( eip )，保存这三个值得寄存器是MSR寄存器组。这组寄存器没有名字，只有编号，由于没有名字，无法通过正常的汇编指令来存取值，Intel提供了两条指令来读写这些寄存器:

* `rdmsr` 读取MSR寄存器 其中高32位存放在EDX 低32位存放在EAX(64位和32位是一样，只是64位时rdx和rcx的高32位会被清零),使用ECX传递寄存器编号
* `wrmsr` 写入MSR寄存器，和读取一样写入时是用EDX表示高32位,EAX表示低32位，使用ECX传递寄存器编号

也就是说, Windows在启动,进行初始化的时候会将内核栈栈顶,内核CS段选择子,以及代码段地址（`KiFastCallEntry` 函数）的地址一一存放到MSR寄存器组的这几个编号的寄存器中。当 `SYSENTER` 被执行,，CPU就直接使用这些寄存器的值来初始化真正的CS , EIP , ESP 寄存器。因此, `SYSENTER` 执行之后, 就跑到内核的 `KiFastCallEntry` 函数中执行代码了。

而进行SYSENTER-HOOK时我们只需要关注代码的地址( `SYSENTER_EIP_MSR` )即可，它的编号是`0x176`。用类似于3环的Inline-Hook的方法，直接把该地址改为jmp xxx，而xxx是我们自己的函数地址，这样就能实现HOOK了。具体用法如下：

**安装钩子**

```
//安装钩子
void __declspec(naked) InstallHook()
{
__asm
{
push eax;
push ecx;
push edx;
//保存原始函数
mov ecx, 0x176;//KiFastCallEntry函数地址的所在编号寄存器
rdmsr; //读取编号寄存器中的值到edx:eax
mov [g_OldKiFastCallEntry],eax;//保存
//替换自己的函数
mov eax, MyKiFastCallEntry;
xor edx, edx;
wrmsr;//把自己的函数地址写入进入
pop edx;
pop ecx;
pop eax;
ret;
}
}
```

**自己的函数**

```
//Hook关键代码
void _declspec(naked) MyKiFastCallEntry()
{
__asm
{
cmp eax, 0xbe;//对比是否是NtOpenProcess的调用号
jne _End; //不是则不处理
push eax; //保存寄存器
mov eax, [edx + 0x14];//获取第4个参数PCLIENT_ID
mov eax, [eax];//获取PCLIENT_ID第一个字段PID
//PCLIENT_ID->UniqueProcess的值
cmp eax, g_Pid;//判断是否是要保护的进程
pop eax;
jne _End;
cmp[edx + 0xc], 1;//判断是否是关闭操作
jne _End;
mov[edx + 0xc], 0;//是就把访问权限设为无
_End:
jmp g_OldKiFastCallEntry;//调用原来的函数
}
}
```

编译生成.sys文件后使用工具安装驱动服务，然后打开任务管理器，关闭被保护的进程，就可以看到拒绝访问，到此保护进程就成功了。在学习SYSENTER-HOOK时，我们用3环的Inline-Hook对比着学习，这样可以加快对这种HOOK方法的理解。

![](upload/attach/201906/847228_UKCZQ74J2THEQKB.jpg)

**内核层完整代码如下：**

```
#include <ntifs.h>
//原来的KiFastCallEntry
ULONG_PTR g_OldKiFastCallEntry = 0;
//要保护进程的ID
ULONG g_Pid = 5616;
ULONG g_Access = 1;//关闭权限的宏PROCESS_TERMINATE
//安装钩子
void InstallHook();
//卸载钩子
void UninstallHook();
//Hook关键代码
void MyKiFastCallEntry();
void OutLoad(DRIVER_OBJECT* obj)
{
obj;
//卸载钩子
UninstallHook();
}
//驱动入口主函数
NTSTATUS DriverEntry(DRIVER_OBJECT* driver, UNICODE_STRING* path)
{
path;
KdPrint(("驱动启动成功！\n"));
DbgBreakPoint();
//安装钩子
InstallHook();
driver->DriverUnload = OutLoad;
return STATUS_SUCCESS;
}
//安装钩子
void __declspec(naked) InstallHook()
{
__asm
{
push eax;
push ecx;
push edx;
//保存原始函数
mov ecx, 0x176;//KiFastCallEntry函数地址的所在编号寄存器
rdmsr; //读取编号寄存器中的值到edx:eax
mov [g_OldKiFastCallEntry],eax;//保存
//替换自己的函数
mov eax, MyKiFastCallEntry;
xor edx, edx;
wrmsr;//把自己的函数地址写入进入
pop edx;
pop ecx;
pop eax;
ret;
}
}
//卸载钩子
void UninstallHook()
{
__asm
{
push eax;
push ecx;
push edx;
//还原原来的函数地址
mov ecx, 0x176;
mov eax, [g_OldKiFastCallEntry];
xor edx, edx;
wrmsr;
pop edx;
pop ecx;
pop eax;
}
}
//Hook关键代码
void _declspec(naked) MyKiFastCallEntry()
{
__asm
{
cmp eax, 0xbe;//对比是否是NtOpenProcess的调用号
jne _End; //不是则不处理
push eax; //保存寄存器
mov eax, [edx + 0x14];//获取第4个参数PCLIENT_ID
mov eax, [eax];//获取PCLIENT_ID第一个字段PID
//PCLIENT_ID->UniqueProcess的值
cmp eax, g_Pid;//判断是否是要保护的进程
pop eax;
jne _End;
cmp[edx + 0xc], 1;//判断是否是关闭操作
jne _End;
mov[edx + 0xc], 0;//是就把访问权限设为无
_End:
jmp g_OldKiFastCallEntry;//调用原来的函数
}
}
```
