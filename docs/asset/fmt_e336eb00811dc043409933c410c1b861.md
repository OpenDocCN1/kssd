
# 编程获取 PE 文件信息的方法

> 原文：[`www.pediy.com/kssd/pediy10/79108.html`](https://www.pediy.com/kssd/pediy10/79108.html)

前几天，一个学习破解的同学问我一个问题：pe 文件的入口点是怎么获取的。我就给他说了一些 pe 文件格式的东西，并用 C 编程实现获取了 OEP，想想可能很多新手也有此问题，于是就写点东西出来，希望对初学者有所帮助吧。

pe 文件的相关内容，看雪论坛上有很多资料的`http://www.pediy.com/Document.htm`还有一些 pe 文件的结构图，望初学者一定看下，对您的学习会有很大的帮助的。我在这里只写一些常用的 pe 结构信息了，建议大家结合着详细的 pe 结构图（`http://www.pediy.com/document/PE.rar`）看此文章。

今天，我以获取 pe 文件的 oep 为例，介绍一个获取 pe 文件信息的方法，当然只是一个比较笨的方法，一般笨的都是不需要很多知识的，应该符合初学者的口味吧。

为了便于大家阅读本文，我也做了个几个简单的图片，希望可以对大家有所帮助。

看下面这个图片

![](img/56ffe678ca48b1753175ed758c3ee59b.jpg)

这个是对 pe 文件头的简单展开，信息并不全，本文只讨论 oep 的获取，其他信息的获取和此法一样，大家要学会举一反三，相信大家都有这个能力。

也许有人会认为获取 pe 文件信息不会很简单，因为 pe 文件是二进制的。其实不然，虽然是二进制的，但毕竟是硬盘上的数据，只要有数据，我就可以读取呀，二进制的确难搞，但是我们可以转换下，转换成我们能够看懂的不就行了。

在说方法之前，大家先看下面这个图片，这 pe 是我自己写的，弹出一个对话框的程序。

![](img/18c13c69244b08ddceb17634a00da122.jpg)

可以看到里面有 MZ、PE，分别是 DOS 头标志和 PE 头标志，还有几个段（也就是节区）的标记。从中可以知道，有些信息我们还可以从记事本里看得到的，我们甚至可以直接在记事本里修改一些信息而不会破坏程序的运行，但由于编码问题，大多数情况下会破坏 pe 文件的结构，从而导致 pe 文件无法正确地执行。

好了其他方面的不多说了，现在专心点吧，只说下 oep 的获取，我把原理画成了一张图，大家可以先看图，图后来解释。

![](img/6d0215a667115533ad7c2f841e2d6671.jpg)

其实在第一个图中我就画出了，获取 oep 的方法，下面分析下：

Oep 是保存在 `AddressOfEntryPoint` 里面的，这个是 `OptionalHeader` 中的一个成员，`OptionalHeader` 是 pe 头的中的一个成员，所以只要获取了 pe 头 `peHeader`，那么 `(peHeader.OptionalHeader).AddressOfEntryPoint` 就是 oep 的值了，而 pe 头怎么获取呢，在 dos 头中有一个 `e_lfanew` 里面保存的是 pe 头的地址，那么就要获得 dos 头，那么 dos 头该怎么获取呢？如果你也有这样问，而且没有想到答案的话，可以知道，你的编程水平的确有待提高，或者要多写点代码了。

这和我那个同学当时问的一样，我就有笑了。

Dos 头是打开文件的首指针指的就是 dos 头的标志信息的第一个字符 M，所以 dos 头就不用定位了，打开一个 pe 文件，指针指的就是 dos 头了。所以我们的方法可分以下几个步骤（结合上图）：

1. 打开一个 pe 文件
2. 把从 dos 头起始复制长度为 `sizeof(dosHeader)` 的数据，然后写到一个 `IMAGE_DOS_HEADER` 结构的变量中，再次命名为 `dosHeader`.
3. 读取 `dosHeader.e_lfanew` 的值，这是 pe 头的地址，有了起始地址，用和获取 `dosHeader` 同样的方法获取 pe 头信息。
4. 最后获取 oep 信息。

代码如下：

```c
#include <stdio.h>

#include <windows.h>

void main(int m, char *args[])
{
    char *fileName = args[1];
    HANDLE hFile = CreateFile(fileName, GENERIC_READ, FILE_SHARE_READ, 0, OPEN_EXISTING, FILE_FLAG_SEQUENTIAL_SCAN, 0);
    
    if (hFile == INVALID_HANDLE_VALUE)
    {
        printf("Open error\n");
        return;
    }

    IMAGE_DOS_HEADER dosHeader;
    BOOL bRet;
    DWORD readSize;
    
    bRet = ReadFile(hFile, &dosHeader, sizeof(dosHeader), &readSize, NULL);
    
    if (!bRet)
    {
        printf("Read Error!\n");
        CloseHandle(hFile);
        return;
    }

    printf("%X\n", dosHeader.e_lfanew);
    IMAGE_NT_HEADERS peHeader;
    
    SetFilePointer(hFile, dosHeader.e_lfanew, NULL, FILE_BEGIN);
    bRet = ReadFile(hFile, &peHeader, sizeof(peHeader), &readSize, NULL);
    
    if (!bRet)
    {
        printf("Read Error!\n");
        return;
        CloseHandle(hFile);
    }

    IMAGE_OPTIONAL_HEADER32 imOpHeader;
    imOpHeader = peHeader.OptionalHeader;
    
    printf("%X\n", imOpHeader.AddressOfEntryPoint);
    
    CloseHandle(hFile);
}
```

编译环境：`visual C++ 6.0`

在我的 `xp sp3` 上运行情况如下：

![](img/6e5d47733b44a530cf221a949d30f1c0.jpg)

查看 `C:\1.exe` 的 oep，获取的和 `OD` 的是一致的，说明方法是正确的。还有一种方法是从内存镜像中获取，对初学者来说会有一定的难度，就先不讨论了。

如果你有些函数看不明白，可以看微软的 `msdn`，建议看英文的，全！以前在中文的上面找，很多东西都找不到，在英文版生都能够找的到。

这个仅获取了 pe 文件的 oep，你可以用此方法获取 pe 文件的其他信息，还是那句话，要举一反三了。熟能生巧。

在此感谢一个同学帮我整理了图片。

大家有问题可以和我交流下。
