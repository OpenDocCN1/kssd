
# 恶意代码分析实战 Lab03-03(PE映像切换技术)

这个样本比较有意思，利用了 `PE` 映像切换技术，《逆向工程核心原理》有类似例子，最近也是上课看书，偶然看到这部分，写个文章巩固下知识点。

## PE映像

我们首先来康康什么是 `PE` 映像。

我们都知道创建一个进程时，操作系统会为该进程分配一个 `4GB` 大小的虚拟进程地址空间。`PE` 映像就是 `PE` 文件在进程内存中的映射形态。

![](upload/attach/202109/926755_8CAHE583T57T2M8.jpg)

并且 `PE` 文件和 `PE` 映像在形态上也不会是完全相同的，会有一些差别，如下。

![](upload/attach/202109/926755_8FAXYSF68XPEPPB.jpg)

## PE映像切换

那么什么是 `PE` 映像切换呢，简单来说，假设现在有 `A`、`B`、`setup` 三个 `exe` 文件，我们会使用 `setup.exe` 来创建 `A` 进程，按道理本该执行 `A` 进程，但是却使用 `PE` 映像切换技术来达到运行 `B` 进程的目的。

下面来大概总结其步骤，是 `setup.exe` 的代码步骤。

- 将 `B.exe` 这个文件存放到开辟的某个内存空间，利用 `CreateFile`、`VirtualAlloc` 函数，或者其他类似作用的函数。
1. 以挂起模式创建 `A` 进程。
2. 利用 `GetThreadContext` 函数获得 `A` 进程的主线程的上下文结构，得到 `CONTEXT.Eax`，`ReadProcessMemory` 函数读取 `PEB.Image`，也就是 `A` 进程的 `image base` 和 `EP`。
3. 调用 `NtUnmapViewOfSection` 函数卸载 `A` 进程的 `PE` 映像，防止将 `B.exe` 文件映射到该地址时发生冲突(实际上，若 `image base` 不相同，可以不卸载)。
4. 利用 `VirtualAllocEx` 函数在 `A` 进程中以 `B` 进程的 `image base` 地址申请其 `size of image` 大小的空间。
5. 映射 `PE` 文件头
6. `for` 循环映射 `PE` 节区
7. 调用 `WriteProcessMemory` 将 `A` 进程的 `PEB.ImageBase` 修改为 `B` 进程的 `image base`。
8. 将 `EP`(`CONTEXT.Eax`) 修改为 `B.exe` 的 `EP`。
9. 调用 `ResumeThread`，恢复运行 `A` 进程，但是实际上运行的是 `B` 进程。

在看完本 `lab` 和《逆向工程核心原理》中的例子后，产生了两个问题

- 上面的步骤中，修改 `image base` 这一操作的先后顺序是否有影响呢？我认为是没有影响的，因为无论哪种方式，都可以达到在 `A` 进程中为 `B` 进程的 `PE` 映像分配了内存空间，并且将 `A` 进程的 `PEB.ImageBase` 修改为 `B` 进程的 `image base` 的目的。
- 为什么对于映射 `PE` 文件头，映射 `PE` 节区需要分开来？其实也很简单，答案就在上面的图片中，`PE` 文件和 `PE` 映像在形态上不会是完全相同的。

![](upload/attach/202109/926755_FBQCJZD5QFGF88A.jpg)

到这里，`PE` 映像切换的原理差不多就讲完了，感兴趣可以去看看书中的详细讲解，下面来分析恶意样本，与书中例子有小差别，但大体原理一致。

## Lab03-03 样本分析

先看看行为吧，可以看到创建了一个进程，并且退出了。

![](upload/attach/202109/926755_Z7N6XTGAVJ2GKAT.jpg)

查壳，无壳，先拖进 `IDA` 静态分析分析。

`main` 函数

![](upload/attach/202109/926755_SZQBKAYMZNF79X6.jpg)

`sub_40132C` 函数

![](upload/attach/202109/926755_M2U9WVXRU4T98HM.jpg)

我们后面在来解密这个文件，并分析它。

`sub_4010EA` 函数，这个函数的参数有两个，一个是 `C:\Windows\System32\svchost.exe`，用来创建进程，一个是申请的空间地址，保存着解密后的资源文件。

![](upload/attach/202109/926755_VKFVZMFQ9AY83B3.jpg)

## 资源文件分析

利用 `resource hacker` `dump` 出资源文件，然后解密，现在来分析解密后的资源文件。

```c
#include<stdio.h>
#include<math.h>
int main(void){
FILE *p;
char v[0x5FE0]={0};
int i;
p=fopen("abc.bin","rb");
fread(&v, 1, 0x5FE0, p);
for(i=0;i<0x5FE0;i++)
{
v[i]=v[i]^65;
}
FILE *p1 = fopen("def.bin", "wb");
fwrite(&v, 1, 0x5FE0, p1);
}
```

先看 `main` 函数

![](upload/attach/202109/926755_52AKVW8CYKFXD8S.jpg)

`fn` 函数

![](upload/attach/202109/926755_WDRRG5S8B23MRRZ.jpg)

`sub_4010C7` 函数

![](upload/attach/202109/926755_5ZHNQDEVGJGR65Q.jpg)

效果如下

![](upload/attach/202109/926755_VF78ZE9FPPJ4GZV.jpg)

最终我们得出结论，该恶意软件是一个键盘记录器，它在 `svchost.exe` 上使用进程替换。

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

[#病毒木马](https://bbs.kanxue.com/forum-4-1-6.htm)

上传的附件：
- [资源.zip](javascript:void(0);)
  （242.86kb，21次下载）
