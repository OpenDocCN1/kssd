
![](https://bbs.kanxue.com/upload/attach/202111/835440_ED7WHGV4E5W4V6E.jpg)

而这个`eflags`寄存器的前后改变也只是第9位的1变成0，由`eflags`寄存器的格式可以知道，是把`IF`位置0

![](https://bbs.kanxue.com/upload/attach/202111/835440_5STG2FAEMQZA8YE.jpg)

接下来实验陷阱门，陷阱门和中断门的段描述符区别只是`type`的不同，所以陷阱门段描述符就是`` `0x0040EF00`00081020``，在`WinDbg`中更改以后

![](https://bbs.kanxue.com/upload/attach/202111/835440_UFK3Y2MMSBV9DZU.jpg)

继续用上面的代码进行测试，当程序触发`int 3`指令的时候可以在`WinDbg`中看到`cs`和`ss`段寄存器的权限已经提升，`eip`也执行了指定的地址，此时的`efl`的`IF`位并没有被置0

![](https://bbs.kanxue.com/upload/attach/202111/835440_XUSBSQDF8A48UJR.jpg)

继续看栈中内容，可以看到栈保存的内容和中断门是一样的。

![](https://bbs.kanxue.com/upload/attach/202111/835440_34SJ8W7MKSQGGWC.jpg)

> 中断门和陷阱门唯一的区别就是提权以后中断门的`eflags`寄存器的`IF`位会被置0

## 4.任务段和任务门

### a.任务段

上面说过当通过调用门或者中断门提权以后会更换新的`ss`段寄存器以及`esp`的值，这两个值其实来源于`TSS`任务段，该段大小为104字节，格式如下图所示

![](https://bbs.kanxue.com/upload/attach/202111/835440_R3KXJ75UXF5RXFV.jpg)

CPU通过`TR`寄存器来找到`TSS`段的基址和边界，如下图所示，可以看到`TR`寄存器也分为可见部分和不可见部分，其中的不可见部分的基址和边界就是用来说明`TSS`段在内存中的地址以及其大小，可以使用`LTR`和`STR`指令来实现`TR`寄存器的读写，但是这两条指令只能改变`TR`寄存器的值，无法改变`TSS`段中的内容且这两条指令只能在系统层使用

![](https://bbs.kanxue.com/upload/attach/202111/835440_EUWGJBKXR697FGZ.jpg)

> 所以在使用调用门等提权的时候，CPU就会去`TR`寄存器中查询`TSS`段的地址找到`TSS`段，在根据`TSS`段中保存的寄存器的数据来更换寄存器的内容

为了做到使用自己指定的`TSS`段，就需要使用到`TSS`段描述符，当段描述符的`S`位为0且`Type`位为`1011(0xB)`的时候，段描述符保存的就是`TSS`段描述符

![](https://bbs.kanxue.com/upload/attach/202111/835440_HQ9JHWF84DU5NJK.jpg)

此时的段描述符格式如下，此时段描述符的基址和边界描述的是`TSS`段的基址和边界

![](https://bbs.kanxue.com/upload/attach/202111/835440_YJ2RCRDDUUFCUSW.jpg)

当使用`jmp far`或者`call far`指令访问到一个`TSS`段描述符的时候，此时将根据段描述符修改`TR`寄存器中的内容，接着在由`TR`寄存器的基址找到`TSS`段，在使用`TSS`段中保存的寄存器的值来替换相应的寄存器，而访问段描述符以后要执行的代码地址则是由指令的偏移地址决定的。这样，就实现了任务的切换，当执行指定的代码的时候，CPU会将寄存器的值更换为指定的值。

在构造`TSS`段描述符的时候，其基地址说明了`tss`段的位置，所以要指定为我们使用的`tss`内存的地址，将使用以下代码来进行实验

```cpp
#include <cstdio>
#include <windows.h>

#pragma pack(1)
typedef struct _ADDRESS
{
    DWORD dwFuncAddr;
    WORD wSelector;
}ADDRESS;
#pragma pack()

void __declspec(naked) test()
{
    __asm
    {
        pushad
        pushfd
        push fs
        int 3 
        pop fs
        popfd
        popad
        iretd
    }
}

int main()
{
    DWORD dwCr3 = 0;
    char szStack[0x1000] = { 0 };
    PDWORD pTSS = (PDWORD)VirtualAlloc(NULL, 104, MEM_COMMIT, PAGE_READWRITE);
    ADDRESS addr = { 0 };

    if (pTSS)
    {
        RtlZeroMemory(pTSS, 104);
        printf("tss address:0x%X\n", (DWORD)pTSS);
        printf("input cr3:");
        scanf("%x", &dwCr3);

        pTSS[7] = dwCr3; //cr3
        pTSS[8] = (DWORD)test; // EIP
        pTSS[14] = (DWORD)szStack+0x800; // ESP
        pTSS[18] = 0x00000023; // ES
        pTSS[19] = 0x00000008; // CS 
        pTSS[20] = 0x00000010; // SS
        pTSS[21] = 0x00000023; // DS
        pTSS[22] = 0x00000030; // FS 
        pTSS[25] = 0x20ac0000; // I/O Map Base Address
        
        addr.wSelector = 0x0048;
        
        __asm call fword ptr [addr]
        
    }

    return 0;
}
```

在这段代码中对`tss`进行了必要的赋值，当它输出了以下的内容就获得了要使用的`tss`的地址

![](https://bbs.kanxue.com/upload/attach/202111/835440_G2MAEU2DPJMEDQS.jpg)

这样就可以构造`tss`段描述符为`` `0x0000E93A`00000068``，那么就可以使用`WinDbg`在相应位置加入段描述符

![](https://bbs.kanxue.com/upload/attach/202111/835440_RCYUFT9RVKRCJ2S.jpg)

接着使用`!process 0 0`来获得`test.exe`进程的`cr3`，这个值会在页机制中讲解，在我的电脑这个值是`0x02940240`。

![](https://bbs.kanxue.com/upload/attach/202111/835440_XY9D4DWMDSP46CB.jpg)

随后就在控制台出输入

![](https://bbs.kanxue.com/upload/attach/202111/835440_MU7XEX55SWXGRC3.jpg)

当触发`int 3`中断的时候，查看`WinDbg`中的寄存器，可以看到相应的值都改成了指定的内容

![](https://bbs.kanxue.com/upload/attach/202111/835440_6GXV8A6TJZGP9BU.jpg)

### b.任务门

当段描述符的`S`域为0且`type`为`0101(0x5)`的时候，此时的段描述符表示的就是任务门段描述符。

![](https://bbs.kanxue.com/upload/attach/202111/835440_VH6EJ7HE694XJTY.jpg)

任务门描述符提供了对任务的间接的、受保护的引用，段描述符格式如下图。它可以放置在`GDT`、`LDT`或`IDT`中。任务门描述符中的`TSS`段选择器字段指向`GDT`中的`TSS`描述符。

![](https://bbs.kanxue.com/upload/attach/202111/835440_KQR2GAJCMWS46CH.jpg)

任务门的运行流程如下图所示，首先从`LDT`或`IDT`表中拿到任务门描述符中的段选择子，该段选择子指向`GDT`表中的`TSS`段描述符，随后的操作就和任务段说的一样了。

![](https://bbs.kanxue.com/upload/attach/202111/835440_F82YTU42TV564TG.jpg)

要进行相应实验，只需要在`IDT`表中将任务门段描述符`` `0x0000E500`00480000``加入，随后在程序中通过`int`指令调用即可，这里就不再演示。

## 七.总结

Intel提供的段机制可以有效的实现了权限的控制，运行在ring3的程序想要访问或使用ring0的内容需要通过提供的各类门来完成。

## 八.参考资料

> - 《软件调试》（第二版）卷一：硬件基础

> - 《英特尔白皮书3-1》

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于  2022-1-6 18:52

被1900编辑

，原因：

[#其他内容](https://bbs.kanxue.com/forum-4-1-10.htm)
