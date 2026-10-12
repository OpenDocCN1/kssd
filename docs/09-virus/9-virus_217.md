
# Window 下基于 C/C++ 源码免杀理论及思路 (新手篇)

> 原文：[`www.pediy.com/kssd/pediy12/134354.html`](https://www.pediy.com/kssd/pediy12/134354.html)

作者：冷锋(LengF) [BHST]
博客：`www.81sec.com`
时间：2011-5-22

## 0x00 概述

最近搞一个国外的网站，对方是用的赛门铁克的诺顿杀毒软件，而内网又只对外开放了 80 端口，于是就想着传个端口转发或者端口复用的工具上去，可是一上传就没了。既然已经提权了，又不想轻易放弃这台服务器。于是就想着怎么办了。自己对非源码的免杀不熟悉，于是就想到自己手头有类似于 `LCX` 的工具 C 源码。对此，翻了以前学习源码免杀的笔记进行了一些总结。很少看到有朋友总结这方面的资料，我就以自己的一些经验总结一些关于源码免杀的一些技巧和方法，在文中我会以一些简单的例子来表达我的思路。为此，我分享我的经验的同时，也希望大家分享自己的免杀思路，毕竟一个人的力量有限，文章中难免有些错误，还望指出。

## 0x01 基于源码免杀原理

在学习源码免杀原理你要了解下 PE 结构，懂得如何去分析一个 PE 的各个部分。我们在源码免杀过程中可能涉及到下面一些情况：

1. 文件头的免杀(这种情况比较少，国外杀软杀的比较比较多)；
2. 代码区的免杀(最常见)；
3. 字符串的免杀(也很常见)；
4. 输入表和输出表的免杀；
5. 版权信息的免杀；
6. 等待你的补充……

在测试免杀的过程中我们经常会用到一些工具，比如：

`MyCCL` …………用于定位特征码,应用各种类型特征码定位
`C32ASM` ………16 进制查看和字符串查找,字符串免杀
`Ollydbg` ………反汇编动态调试工具，代码区的免杀
`LoadPE` ………查看 PE 结构信息，段区地址/输入表地址/输出表地址等查询

做好了工具准备，我们以一个测试的 `VC6.0` 编译出来为例来描述 PE 的各种结构，先给出代码：

```c
#include "stdio.h"
void PrintMsg()
{
  printf("Hello!LengF.\n");
}

int main()
{
  PrintMsg();
  getchar();
  return 0;
}
```

我们以 Release 编译输出，我们先用 `LoadPE` 打开看看，如图 01：

![](img/eef3106bbeaee0da892eda982013c9c3.jpg)

查看下入口点地址，还有子系统类型，另外就是区段表地址和输入表地址信息，对于下面分析有帮助。在用 `C32ASM` 打开这个程序，我分离各个我们在本文要涉及的区域，如图 02：

![](img/ac167f816362f90aec6ca99deff4621b.jpg)

这是对于 PE 结构的信息一些基本信息，我们往往并不需要准确定位各个区段的起始地址。我们在通过 `ollydbg` 查看下，了解下 `VC6.0` 程序的一些特点，如图 03：

![](img/3958162b6979028a589e91441c354b52.jpg)

了解完这些我们就可以开始对基于 C/C++ 的源代码进行免杀了。下面将根据不同部分提出自己的一些思路和经验。

## 0x02 免杀步骤

**步骤：**

1. **免杀前，我们做一些简单的花指令** (就是一些没有实际功能的指令，比如 `NOP`)，我们可以写这样一段函数：

   ```c
   Void NOPCode()    // 花指令代码
   {
     __asm
     {  // 我以 NOP 为例，NOP 的个数随便，不过建议不要太多
       NOP
       NOP
       NOP
   }
   }
   ```

   然后再需要地方调用这个函数就可以了。这一步我们并不知道该 PE 文件的特征码的位置，只是按照经验添加，比如在调用某些特定的 API 函数，或者某些敏感的字符串。一般在调用前添加就可以了，当然你也可以添加在后面或者中间，只要不影响程序的正常运行即可。当然也可以是其他花指令，我们把这一步叫做预处理。源码免杀的好处就是不用重新确定程序的入口地址，编译完会自己确定。花指令的变化很多，大家自行研究。

2. **利用 `MyCCL` 定位文件特征码的地址**，尝试两种定位方法，即采用正向定位和反向定位。

   经过多次 `MyCCL` 的定位得到一个特征码地址，然后通过 `C32ASM` 查看特征码在整个 PE 结构的中位置，就可以确定是处于文件头，还是代码区，还是输入表，还是字符串等位置。

3. **最后一步就是经过一些特殊源码处理实现源码的免杀**。下面将针对于各种类型的源码免杀思路。

## 0x03 常用源码免杀思路

### 1. 字符串的源码免杀

我们以上面的源码为例，仅作稍微修改：

```c
#include <windows.h>
#include <stdio.h>
void PrintMsg()
{
  MessageBox(NULL,"Hello!LengF",NULL,MB_OK);
}

int main()
{
  PrintMsg();
  return 0;
}
```

首先我们通过 `C32ASM` 打开该 release 的文件，假设我们利用 `MyCCL` 定位到地址 `00006030` 为特征码，其内容如图 04：

![](img/1e472a7e5ebe220280a1e58af767bf86.jpg)

我们发现是字符串 `LengF`，再到源码中搜索这个字符串。我们可以通过字符串的合并和分割方式来进行免杀，有几种方法：

**[1] 利用 `wsprintf`，`sprintf`，`strcat` 等进行字符串的格式化实现合并和分割**，我们把代码改成：

```c
char StrMsg[]={'H','e','l','l','o','!','L','e','n','g','F','\0'};
MessageBox(NULL,StrMsg,NULL,MB_OK);
```

再用 `C32ASM` 看下，发现已经找不到该字符串了。同理，我们可以先将两个字符串分成两个字符串，最后通过 `wsprintf` 或者 `sprintf` 或者 `strcat` 来合并起来。同理也是可以合并一些没用扰乱的字符串来蒙混过关。

**[2] 利用 API 函数倒序输出函数 `strrev` 函数**，我们将上面代码改成：

```c
char StrMsg[]="FgneL!olleH";
MessageBox(NULL,strrev(StrMsg),NULL,MB_OK);
```

这也是一种方法，能够躲过很多杀毒软件的查杀。

**[3] 字符串加密方式**，有些只要经过简单的变形加密即可，对于那些比较复杂算法加密的不大建议；

除了上面三种方法，大家兴许还有更好的方法，还望分享，不过我想上面的几种思路已经足以应付了，只要灵活应用。

### 2. 输入表函数源码免杀

输入表函数源码免杀只特征码被定位在 PE 文件的输入表函数(源码中的 API 函数)上的免杀处理方法。我们先通过 `LOADPE` 先看看我们上面字符串免杀那个程序的输入表的地址和大小，如图 05

![](img/6388f2894fad4a1a537f337fa98da38b.jpg)

假设我们的特征码定位在 `MessageBoxA` 函数的地址（只是举个例子，一般这个函数不会被杀，其他函数也类似方法）那么我们到源码里面发现，我们在 `PrintMsg` 函数中调用了这个 API 函数，那么我们就要对他进行处理。对于输入表被查杀我们一般的处理方法是修改函数的调用方式，比如采用 dll 动态调用，通过上图，我们知道我们是以非 unicode 方式编译的 `MessageBox`，他对应的非 Unicode 的函数为 `MessageBoxA`，同时我们也得到这个 `MessageBoxA` 函数在 `UER32.dll` 文件中。那么我们修改 `MessageBox` 的调用方式为 dll 的动态调用，其代码修改成：

```c
typedef int (WINAPI *MessageBoxAT)
(
  HWND hWnd,
  LPCSTR lpText,
  LPCSTR lpCaption,
  UINT uType
);

void PrintMsg()
{
  char StrMsg[]="FgneL!olleH";
  MessageBoxAT pMessageBoxA= (MessageBoxAT)GetProcAddress(LoadLibrary("user32.dll"),"MessageBoxA");
  pMessageBoxA(NULL,strrev(StrMsg),NULL,MB_OK);
}
```

这样以后我们就修改了输入表中调用 API 函数的名称了。其他 API 函数也可以类似的修改调用方式。

### 3. 版权免杀

这种没什么好讲的，只要修改源码中的版权信息即可。

### 4. 一些编译技巧的免杀方法(推荐)

在利用 `VC6.0` 进行编译前我们可以利用一些预定义方法来修改编译的一些参数设置。我们如果没有设置就是按照默认的编译方式编译，你会发现明明没写几行代码，这就是为什么当你用 `C32ASM` 打开查看的时候你会发现存在很多没用的代码的原因了。我们可以参考下 Microsoft 的文档进行设置，我这里只对免杀一些比较好用的方法进行描述。

**[1] 设置默认的程序入口函数名**

学 C 语言的第一个知道的就是其程序入口为 `main`，其实在操作系统装载应用程序后，完成初始化工作就进入 `main` 执行你写好的代码，在 `VC6.0` 下默认的控制台程序入口函数并不是 `main`，而是 `mainCRTStartup`，只是默认有一个预定义的编译参数：

```c
#pragma comment( linker, "/subsystem:windows /entry:mainCRTStartup" )
```

同理，我们知道 `Win32` 程序的默认入口函数是 `WinMain`，它对应的默认编译参数是：

```c
#pragma comment( linker, "/subsystem:windows /entry:WinMainCRTStartup" )
```

而具体是设置哪个入口点是可以由连接器“/subsystem:”选项来设置的，可以指定四种方式：“CONSOLE|WINDOWS|NATIVE|POSIX”如果这个选项参数的值为“WINDOWS”，则表示该应用程序运行时不需要控制台。我们在 `VC6.0` 也可以使用图形化界面来设置，如图 06：

![](img/f8c04ff0369a02f69986bb17b8160326.jpg)

当然我们也可以通过程序添加预定义代码来设置，比如：

```c
#pragma comment(linker,"/subsystem:windows")
```

设置这个以后，控制台程序运行的时候就不会运行控制台了（背后的那个黑黑窗口），那设置这个选项有对于源码免杀有什么好处呢？

(1) 修改程序的入口点可以让杀毒软件混淆，用 OD 载入后发现程序的入口点已经和原来不一样了，比如我设置：

```c
#pragma comment(linker, "/ENTRY:procentry")
```

那么此时你的主函数名就不是 `main` 而是 `procentry` 了。

(2) 设置子系统模式可以修改程序的运行方式了，更加隐蔽了，大部分木马服务端都是这样设置；

**[2] 优化编译选项**

程序的编译方式是由连接器默认指定的，但是你会发现默认指定的并不是最好最优化的方式，它主要考虑的是兼容性方面的问题。所以我们有必要对其进行优化设置，在免杀中我们经常需要设置有以下几种：

(1) 设置对齐方式

```c
#pragma comment(linker, "/FILEALIGN:16")
// 设置最小节的大小，数值越小文件的体积就越小，不过最小是 16
#pragma comment(linker, "/ALIGN:16")
```

(2) 优化选项

```c
// 清除从未引用的函数和/或数据
#pragma comment(linker, "/opt:ref")
#pragma comment (linker, "/OPT:ICF")
// 获得精简应用程序，减小体积
#pragma comment(linker, "/opt:nowin98")
```

作用：减小生成的 PE 文件体积，改变默认编译出来 PE 结构和地址分配。

(3) 区段的合并和独立

我们利用 `LoadPE` 可以看到一个 PE 程序有多少个区段，比如上面那个程序有 3 个区段：`.text`,`.rdata`,`data`。那么我们在编译一个程序的时候也可以对这个程序进行区段的合并和独立。在编辑区段前要设置区段的属性，使用预定义参数设置：

```c
// 设置 data 段属性为可读可写可执行
#pragma comment(linker, "/section:.data,RWE")
// 接下来就可以进行区段合并操作了比如，将上面的 text 和 rdata 都合并到 data 去
#pragma comment(linker, "/MERGE:.rdata=.data")
#pragma comment(linker, "/MERGE:.text=.data")
```

另外一种方法就是将一个文件的代码全部放置到一个自定义段中

```c
#pragma code_seg("section1")
```

我分别使用截取了这三种方式的区段表，如图 07：

![](img/35f25a3d8186322f820f5cabe336b3fc.jpg)

通过上面的这些免杀的简单思路，他们具有很多有有点，包括：有效减小了 release 的生成 PE 文件体积大小；同时改变了 PE 结构的分布，有效的实现了对杀毒软件的干扰。有时候我们并不进行其他的免杀方式就可以实现免杀了。

## 0x04 总结

对于非源码的免杀来说，源码免杀具有更多的灵活性和降低免杀的实现复杂度。源码的免杀如果要完整的写就要写一本书了，本文只是做一下简单的描述，适合初学者学习，高手掠过。本文只是抛砖引玉，再次希望大家也分享自己的免杀思路。有任何问题建议请联系 `cn_lgz[at]126.com`，或者博客(`www.81sec.com`)留言交流讨论。

**参考资料：**

- [`http://support.microsoft.com/kb/235956/zh-cn`](http://support.microsoft.com/kb/235956/zh-cn)
- [`http://blog.csdn.net/lostangels/archive/2008/01/06/2027642.aspx`](http://blog.csdn.net/lostangels/archive/2008/01/06/2027642.aspx)
- [`http://msdn.microsoft.com/zh-cn/wf5kss02.aspx`](http://msdn.microsoft.com/zh-cn/wf5kss02.aspx)
- [`http://msdn.microsoft.com/zh-cn/library/bxwfs976(d=lightweight,v=VS.90).aspx`](http://msdn.microsoft.com/zh-cn/library/bxwfs976(d=lightweight,v=VS.90).aspx)
- [`http://wenku.baidu.com/view/abb27e4c2b160b4e767fcfea.html`](http://wenku.baidu.com/view/abb27e4c2b160b4e767fcfea.html)
