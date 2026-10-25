
# Linux 执行程序的 Anti 静态反汇编

> 原文：[`www.pediy.com/kssd/pediy10/77415.html`](https://www.pediy.com/kssd/pediy10/77415.html)

Windows 下的 crack 我们已经司空见惯，虽然如此。但是如我一般的菜菜们仍然前赴后继的跟着大牛们的足迹往前冲。最近对 linux 下的破解想了解一番，于 windows 下的破解不同。在 linux 的世界里，需要的 crack 的软件不是很多。当你发现一款商业 linux 的软件包的时候，你会很快发现，要实现同样的功能的软件还有其他的开源软件。对于 linux 的软件的 crack，仅仅依靠着对技术的追求了。而在物质的世界里。这样的追求永远没有 windows 下的商业利益更能够吸进人的了。

知其然，才知其所以然，再一个就是知己知彼，方能百战不殆。

最近网上搜反静态反汇编的方法，记得不是很清楚了具体内容。也懒的再去搜索了，凭大致记忆做个实验，并形成文字纪念之。

大致有个印象说是单字节指令，与双字节指令的干扰什么的....，我已经大致想到些什么了。动手...

## 1、在用户名文件下建立一个空文档 `test.c`

在文件里写上

代码:

```
#include "stdio.h"
int main()
{
char outs[]="hello word!\\n";
asm volatile(            //用 asm volatile 是害怕 Gcc 自作主张的帮我给优化了。
"jmp lab1 \\n\\t" 
"lab0: \\n\\t"
"popl %eax \\n\\t"
"add $1,%eax \\n\\t"
"jmp *%eax \\n\\t"
"lab1: \\n\\t" 
"call lab0 \\n\\t"
"nop");
asm volatile(
"movl $0x0d,%%edx \\n\\t"  // 参数三：字符串长度
"movl %0,%%ecx \\n\\t"  //参数二：要显示的字符串
"movl $1,%%ebx \\n\\t"    //参数一：文件描述符(stdout) 
"movl $4,%%eax \\n\\t"    //系统调用号(sys_write) 
"int  $0x80 \\n\\t"       //调用内核功能
                                 //退出程序
"movl $0,%%ebx \\n\\t"     //参数一：退出代码
"movl $1,%%eax \\n\\t"     //系统调用号(sys_exit) 
"int $0x80"         //调用内核功能
:
:"r"(outs)
:"%eax","%ebx","%ecx","%edx");     
return 0;
}
```

## 2、下面我编译它，工作目录是的 `test.c` 的所在目录

代码:

```
[jun@beijihuCom ~]$ gcc -o test.o test.c
```

输出了一些

运行

代码:

```
[jun@beijihuCom ~]$ ./test.o
hello word!
```

一切正常。

ok，准备工作完毕。

开始对我们的程序作手脚了。

哦，对了我们现看一下没有做手脚之前的反汇编代码。

代码:

```
[jun@beijihuCom ~]$ objdump -d ./test.o
```

翻出来的汇编不是很长很容易找到我们之前写的代码之处。

![](img/31161120a6e6b1f4fde0e8eb4cb8c0b1.jpg)

`replace` 程序了。

`gHexEdit` 一个不错的十六进制的工具（当然也可以 `vi` 修改 2 进制的命令是：`%!xxd`）

代码:

```
8048391:       e8 f5 ff ff ff          call   804838b <lab0>
8048396:       90                      nop    
8048397:       8d 75 e7                lea    0xffffffe7(%ebp),%esi
```

`8048396` 处有俄 `90` 即 `nop` 我们要改的就是它了（实际偏移是 `0x396`。至于为什么。以后我解释！）

直接修改为 `8b` 是 `mov` 一个 `_ver(%ebp)` 到 `%ecx` 中的指令。实际可以不管他，只要是一个双元指令就可以。

ok 保存一下

检查一下我们的程序有没有坏掉，应该没有的!

代码:

```
[jun@beijihuCom ~]$ ./test.o
hello word!
```

又是一切正常。哈哈，离成功很接近

代码:

```
[jun@beijihuCom ~]$ objdump -d ./test.o
```

和刚刚的图片比较一下？

![](img/6d19d56b339fee98f972e2ba6631ae36.jpg)

嘿嘿，不一样了吧.

看看号称能买到的最强的静态反汇编工具 `IDA PRO` 反下看看........ 

![](img/630bdccba243f8b156253e8882cccf54.jpg)

成功欺骗...

实际上方法很简单就是利用 `call` 把下一行的地址获得，`pop` 给 `%eax` 加上 `1` 就是 `nop` 后的一个地址了。`nop` 是单元指令且没有意义。现在我们把它换成二元的，静态一分析，一准错误。哈哈。

这个方法不知道在 windows 下能不能实现，我想应该也是可以的。但也说不准......有时间实验一下。

上传的附件

| ![](img/fb8ade91a9e54b7bd6690c3ae5b1d1cc.jpg) || `test.zip` |
