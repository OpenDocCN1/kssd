
# 调用门初级应用访问页表目录

> 原文：[`www.pediy.com/kssd/pediy12/128469.html`](https://www.pediy.com/kssd/pediy12/128469.html)

调用门初级应用访问页表目录

近日在看学学院下载了董岩的《Undocumented》，学习到第四章内存管理，有了关于调用门初级应用的总结。

参考了 `Troy` 的调用门驱动 [`http://bbs.pediy.com/showthread.php?t=59005`](http://bbs.pediy.com/showthread.php?t=59005)

参考了 `combojiang` 的门系列一 [`链接: http://bbs.pediy.com/showthread.php?t=62263`](http://bbs.pediy.com/showthread.php?t=62263)

书中对 `ring3` 程序通过调用门访问 `ring0` 内存页表有详尽的理论讲述，但好像没有简单、具体的实现。自己刚好看了调用门，也就算是学习总结加动手实践。

调用门直接用 `Troy` 的 `mykmd.sys`。尊重的五体投地。可以不用写新的驱动了。

改写的 `C` + 汇编混编代码：`showdir.cpp`

```c
#include <windows.h>

#include <stdio.h>

#include <string.h>

DWORD PageDirectory[1024];//保存页表的全局变量数组

void main()

{    WORD farcall[3];

     farcall[0] = 0x0;

     farcall[1] = 0x0;    //Troy 的调用门在 GDT 中 0x3e0 位置

     farcall[2] = 0x03e3;   //添加 CALLGATE 的选择子为 03e3H

     _asm{        //代码段选择子 03e8

   call fword ptr [farcall]  //以下进入 ring0 03e8:xxxxxxxx

   mov eax,esp

   mov esp,[esp+4]

   push eax

  }//页目录被映射在每个进程的虚地址 0xC0300000 中。此地址用户模式无法访问

  DWORD *PageDir=(DWORD *)0xC0300000

  for(int i=0;i<1024;i++)  //从 0xC0300000 处简单拷贝，用户模式代码可访问。

    PageDirectory[i]=PageDir[i];

  _asm{

    pop esp

    push offset ring3

    retf

  ring3:nop      //以下返回 ring3 代码段 01B：xxxxxxxx

  }

  int ctr = 0;

  printf("Page directory for the process, pid=%x\n", GetCurrentProcessId());

    for (i = 0; i<1024; i++)

{       if (PageDirectory[i] & 0x01)

{   if ((ctr % 3) == 0)     //每行打印 3 个页表项

                    printf("\n");

                  printf("%08x:%08x ", i << 22, PageDirectory[i] & 0xFFFFF000);

               ctr++;

          }

    }

    getchar();

}

/*听说不用写驱动也行,猜测物理内存？学习 ing![](img/2aa97d64a7e0b211d2f8f207eb382aa7.jpg "酷")D:![](img/2aa97d64a7e0b211d2f8f207eb382aa7.jpg "酷")![](img/c1d8523a7842d7b67917c210ac6dcf18.jpg "微笑")*/
```
