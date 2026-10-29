
# [原创]通用shellcode开发原理与实践

学习来源：0day安全一书

## 概述

`shellcode`即一段植入进程的代码。

## 定位shellcode

在实际漏洞利用过程中，由于动态链接库的装入和卸载等原因，Windows进程的函数栈帧很有可能会产生“位移”，即shellcode在内存中的地址是动态变化的。因此我们需要找到一个跳板，使得程序的执行流程总是能找到我们的shellcode。

### jmp esp

在上一节中`verify_password`函数返回后栈中的情况如下所示：

1. 实线体现了代码植入的流程：将返回地址淹没为我们手工查出的shellcode起始地址`0x0012FAF0`，函数返回时，这个地址被弹入`EIP`寄存器中，处理器按照`EIP`寄存器中的地址取指令，最后栈中的数据被处理器当成指令执行。

2. 虚线则点出了这样一个细节：在函数返回时候，`ESP`恰好指向栈帧中返回地址的后一个位置。

图一 栈帧位移示意图

![图片描述](upload/attach/202304/879928\_SWU767PHQE2H9TV.png)

图二 溢出发生时栈、寄存器与代码之间的关系

![图片描述](upload/attach/202304/879928\_EH7R4FABCVT9XKC.png)

一般情况下，`ESP`寄存器中的地址总是指向系统栈中且不会被溢出的数据破坏。函数返回时，`ESP`所指的位置恰好是我们所淹没的返回地址的下一个位置，如图三。

图三 使用“跳板”的溢出利用流程

![图片描述](upload/attach/202304/879928\_SRZ65RQGTWKJJCU.png)

由于`ESP`寄存器在函数返回后不被溢出数据干扰，且始终指向返回地址之后的位置，我们就可以使用图三所示的这种定位shellcode的方法来进行动态定位。

1. 用内存中的任意一条`jmp esp`指令的地址覆盖函数的返回地址，而不是用原来的手工查询出的shellcode起始地址直接覆盖。

2. 函数返回地址被重定向去执行内存中的这条`jmp esp`指令，而不是直接开始执行shellcode。

3. 由于`ESP`在函数返回时仍指向栈区（函数返回地址之后），`jmp esp`指令被执行后，处理器会到栈区函数返回地址之后的地方取指令执行。

4. 重新布置shellcode。在淹没函数返回地址后，继续淹没一片栈空间。将缓冲区前边一段地方用任意数据填充，把shellcode恰好摆放在函数返回地址之后。这样，`jmp esp`指令执行过后会恰好跳进shellcode。

这种定位shellcode的方法使用进程空间里的一条`jmp esp`指令作为“跳板”，无论栈帧怎么“位移”，都能精确地跳回栈区，从而适应程序运行中shellcode内存地址的动态变化。

当然这只是定位shellcode的一种方式，还有其他许多种定位shellcode的方式。

## 开发通用shellcode

### 环境

```
操作系统： Windows 10 x64
编译器： vs 2019
```

### 定位shellcode使用API的原理

通过手工查出来的API地址会在其他计算机上失效，在shellcode中使用静态函数地址来调用API会使exploit的通用性受到很大的限制，所以，实际中使用的shellcode必须还要能动态地获得自身所需的API的函数地址。

Windows的API是通过动态链接库中的导出函数来实现的，例如，内存操作等函数在`kernel32.dll`中实现；大量的图形界面相关的API则在`user32.dll`中实现。Win32平台下的shellcode使用最广泛的方法，就是通过从进程环境块中找到动态链接库的导出表，并搜索出所需的API地址，然后逐一调用。

几乎所有Win32程序都会加载`ntdll.dll`和`kernel32.dll`这两个基础的动态链接库。如果想要在Win32平台下定位`kernel32.dll`中的API地址，可以采取如下办法。

**64位系统**

1. 首先通过选择字`GS`在内存中找到当前存放着指向当前线程环境块`TEB`。在`GS`中存储的是`TEB`在`GDT(Global Descriptor Table)`中的序号，通过`GDT`获取`TEB`的基址。

2. 线程环境块偏移位置为`0x60`的地方存放着指向进程环境块`PEB`的指针（即`GS[0x30]`）。

3. 进程环境块中偏移位置为`0x18`的地方存放着指向`PEB_LDR_DATA`结构体的指针，其中存放着已经被进程装载的动态链接库的信息。

4. `PEB_LDR_DATA`结构体偏移位置为`0x20`的地方存放着指向模块初始化链表的头指针`InInitializationOrderModuleList`。

5. 模块初始化链表`InInitializationOrderModuleList`中按顺序存放着`PE`装入运行时初始化模块的信息，第一个链表结点是`ntdll.dll`，第二个链表结点就是`kernelbase.dll`，第三个节点才是`kernel32.dll`。

6. 找到属于`kernel32.dll`的结点后，在其基础上再偏移`0x20`就是`kernel32.dll`在内存中的加载基地址。

7. 从`kernel32.dll`的加载基址算起，偏移`0x3C`的地方就是其`PE`头。

8. `PE`头偏移`0x88`的地方存放着指向函数导出表的指针。

**32位系统**

1. 首先通过选择字`FS`在内存中找到当前存放着指向当前线程环境块`TEB`。在`FS`中存储的是`TEB`在`GDT(Global Descriptor Table)`中的序号，通过`GDT`获取`TEB`的基址。

2. 线程环境块偏移位置为`0x30`的地方存放着指向进程环境块`PEB`的指针（即`FS[0x30]`）。

3. 进程环境块中偏移位置为`0x0C`的地方存放着指向`PEB_LDR_DATA`结构体的指针，其中存放着已经被进程装载的动态链接库的信息。

4. `PEB_LDR_DATA`结构体偏移位置为`0x1C`的地方存放着指向模块初始化链表的头指针`InInitializationOrderModuleList`。

5. 模块初始化链表`InInitializationOrderModuleList`中按顺序存放着`PE`装入运行时初始化模块的信息，第一个链表结点是`ntdll.dll`，第二个链表结点就是`kernel32.dll`。

6. 找到属于`kernel32.dll`的结点后，在其基础上再偏移`0x08`就是`kernel32.dll`在内存中的加载基地址。

7. 从`kernel32.dll`的加载基址算起，偏移`0x3C`的地方就是其`PE`头。

8. `PE`头偏移`0x78`的地方存放着指向函数导出表的指针。

9. 至此，我们可以按如下方式在函数导出表中算出所需函数的入口地址，如图四所示。

图四 在shellcode中动态定位API的原理

![图片描述](upload/attach/202304/879928\_JZQEMUZ3AFZP3UV.png)

### 代码实现

#### 使用汇编生成shellcode

x64 弹出计算器shellcode
