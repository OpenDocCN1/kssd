
## 步骤六：实现自己的操作

从上面代码可以看到，程序断点调试事件收到后会调用`onExceptionEvent`函数，这个也是自己书写的函数，他的内容便是修改弹窗标题。那要如何获得弹窗的标题呢？我们可以使用`GetThreadContext`来进行获取`CONTEXT`结构体，再从其中来获取我们的标题。

[对照`MessageBoxA`文档:](elink@5c0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3V1k6%4K9h3^5K6x3W2)9J5c8X3q4H3K9g2)9J5c8Y4N6A6L8Y4g2K6k6i4u0Q4x3V1k6F1k6W2)9J5k6s2N6A6L8Y4g2K6k6i4u0Q4x3X3c8E0k6i4y4K6j5h3N6W2j5X3!0^5j5b7`.`.)

```
int MessageBoxA(
HWND hWnd,
LPCSTR lpText,
LPCSTR lpCaption,
UINT uType
);
```

观察文档可以看到，我们要的标题在第三个参数（`lpCaption`）。在提取标题前我们必须知道一个汇编的知识：**`ESP`代表调用栈的栈顶，当调用一个函数时，根据调用约束的不同，对函数的参数进行不同方式的传递。而C/C++的默认约定便是将参数压入栈。**

听不懂不要紧，我们只要知道`ESP+4`是第一个参数，`ESP+8`是第二个参数，`ESP+C`就是我们要的标题。

而所谓的`ESP`我们可以从`CONTEXT`结构体获得：

```
CONTEXT context;

context.ContextFlags = CONTEXT_CONTROL;

GetThreadContext(ori.hThread, &context);

DWORD ptitle;

ReadProcessMemory(ori.hProcess, (LPCVOID)(context.Esp + 0xC), &ptitle, sizeof(DWORD), NULL);
```

然后通过`WriteProcessMemory`更改标题即可：

```
CStringA newTitle = "z";
WriteProcessMemory(ori.hProcess, (LPVOID)ptitle, newTitle.GetBuffer(), titleSize, NULL)
```

最后我们要把更改成CC了的地址恢复为原样，并把`EIP`复原。因为`EIP`的值指示着当前代码的地址，地址改了当然`EIP`也要改：

```
WriteProcessMemory(ori.hProcess, address, &oribyte, sizeof(byte), NULL);
context.Eip = (DWORD)address; // EIP复原
SetThreadContext(ori.hThread, &context);
```

## 最终步骤：让程序继续运行

当调试事件发生后，程序会断下来。而我们为了在修改程序后还能继续运行，必须通知程序没事了你继续吧。[因此我们可以调用`ContinueDebugEven`函数来通知：](elink@d32K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3V1k6%4K9h3^5K6x3W2)9J5c8X3q4H3K9g2)9J5c8X3c8W2j5Y4g2Y4j5i4m8A6i4K6u0r3L8X3k6Q4x3X3c8V1k6h3u0#2k6$3q4H3K9g2)9J5k6r3y4G2L8Y4c8A6L8Y4g2W2k6r3g2T1N6h3N6W2N6X3g2F1N6l9`.`.)

```
BOOL ContinueDebugEvent(
DWORD dwProcessId,
DWORD dwThreadId,
DWORD dwContinueStatus
);
```

# 结果

例子是将弹框标题t改成z：

![1](upload/attach/202008/899080_69VDEUQV3K66G5R.png)

![3](upload/attach/202008/899080_CCKT9HAB7UJX6HH.png)

![2](upload/attach/202008/899080_2JTPNNU8WGXPMXC.png)

# 函数列表

以下为一些较为关键的函数（仅名称）：

`DebugActiveProcess`——附加程序，成为目标程序的调试者

`WaitForDebugEvent`——接收调试事件

`GetProcAddress`——获取要下断的目标地址

`GetThreadContext`——获取上下文，以此来获取对应数据，如弹窗标题内容等。

`SetThreadContext`——上下文设置，用来恢复原来的地址

`ContinueDebugEvent`——让程序继续运行

`ReadProcessMemory`——读取目标相关数据

`WriteProcessMemory`——更改目标数据，如写入CC断点，修改弹窗标题，内容等。

`VirtualProtectEx`——更改虚拟内存权限，若无法写入数据可使用

# 总结

随着学习的不断深入，知识的难度越来越难是理所当然的。在写这篇文章的时，如何去用简洁的语言去描述过程，并让那些没有编程基础的读者也能看的明白就成为了一个问题。因为知识难度越高，解释中用到的其他的知识铺垫也会越来越多，这就会陷入一个“不断解释”的问题。

例如给别人解释2x3的意义，我们可以用加法来告诉他2x3就是2个3相加或者3个2相加。但这前提是他必须懂得加法。

那如果说我解释的东西用了n个知识铺垫呢？那我是不是得解释这n个知识。所以说，倘若读者不太看得懂这篇文章那只有两种可能：

1. 笔者文笔不好

2. 这n个的知识铺垫的缺失

所以说只有不断提高自己，才能向更高的层次进发。

# 问题：

当我们改写标题的时候，如果新标题的长度超过了4个字节，会使新标题溢出到弹窗内容区：

![4](upload/attach/202008/899080_V3ZCJSJ3CMXZTN7.png)

如上图所示，原标题为t，原内容为z。写入新标题I am title title title title后内容也变成了title tile tile tile。

这是为什么呢？因为在虚拟内存当中，弹窗的标题和内容是连在一起的：

![5](upload/attach/202008/899080_YNMJ73RRDYJ7SY2.png)

因此标题过长会覆盖到隔壁的内容区域。

可如果我们想用一段任意长的内容替代它而不造成溢出，该怎么做呢？笔者想过是否可以替换掉指向标题的地址:

![9](upload/attach/202008/899080_2WAHVUNMHYF8DRE.png)

可一直没成功，如有解决方法希望读者可以留言指点！

# 附件

工程源码及可直接运行的示例[54eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6K9r3q4J5k6g2)9J5k6i4N6W2K9i4W2#2L8W2)9J5k6h3y4G2L8g2)9J5c8V1W2t1b7K6c8&6d9e0N6C8](elink@5beK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6K9r3q4J5k6g2)9J5k6i4N6W2K9i4W2#2L8W2)9J5k6h3y4G2L8g2)9J5c8V1W2t1b7K6c8&6d9e0N6C8)

# 前言

著名的迪奥布兰多先生曾说过“人类是有极限的。”

确实如此，但也正因为我们不能飞，所以才造出了飞机。正因为我们跑的不快，所以才造出了汽车。人类确实存在极限，但人的学习能力却是无限的。而作为一个正在入门逆向萌新的来说，正确认识自身的极限和意识到学习的无穷是很重要的。

为什么这么说呢。回想几个星期前，笔者还在苦苦做论坛上总结的120个CM。每当破解并分析了一个CM感到满足的同时，也为自己耗费长时间去破解分析CM的行为是否值得而感到疑惑，似乎就在原地踏步，而没有学到新的知识。

后来我才意识到，这正是目前自身的极限。缺乏C++编程知识，缺乏系统的API知识，缺乏常规破解手段的知识等等。所以笔者才决定突破自身的极限，学习《逆向工程核心思想》一逆向书籍，才有了今天这篇文章。

可这谈何容易呢？要想突破极限就必须付出相应的努力，去提高自己极限的程度。笔者必须学习C++，必须跟着书籍编写win32程序。当我们知识储备扩展后，极限的程度也就高了，而只有不断提高自身的极限，或许才能成为真正的“大师”吧。

# 准备工作

本篇文章从原理和实现过程两个角度总结书中API钩取一章节，笔者也是萌新。因此难免会有令人困惑和误解甚至错误的地方。倘若存有疑问请务必留言指教。

为了读者能有更好的阅读体验，**最好但不一定必要**具备以下知识和工具：

1. C++编程基础

2. [到微软官方查看文档的能力](elink@d84K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8R3`.`.)

3. 用于编程的集成环境工具visual studio

4. 用于查看进程PID的监控软件Process Explorer

5. 文章附带的附件

再次提醒一下，就算读者没有C++编程基础，也可浏览此文到原理部分，实践部分若无编程基础可忽略不看。当熟知原理以及记住几个关键的函数过后，全文核心内容基本可算基本掌握。

# API钩取的作用

我们为什么要做API钩取，它有什么作用。在描述原理之前，这是我们必须要弄清楚的问题。如同我们去逆向分析软件，都会问自己为什么会这么做。可能是兴趣使然，可能是好玩，也可能是为了钻研等等。为什么很重要，因为它是我们唯一的学习动力。

API钩取它让我们能够直接修改程序中的流程或代码。说的通俗易懂一点，它可以让我们在别人的程序里为所欲为。

例如我们可以直接修改弹窗的标题或内容；又如可以改变我们输入。只要我们知道要修改的地址，我们就可以让他断下来，然后进行各种操作，来达到我们的目的。如果用过IDA或者olldbg的读者一定会发现，这不正是这些调试器的功能吗？

![6](https://bbs.kanxue.com/upload/attach/202008/899080_SWT6NE6E2D9DWGU.png)

# 原理

那到底要怎么做呢？为什么别人写的程序本来好好的却能够突然停下来让你操作，这难道是什么魔法吗？

然而并不是，众所周知Window的编程是函数编程，说的明白一点，程序员在写诸如EXE的Window程序时，都会去使用微软早已做好的函数。而API钩取，也正是使用微软提供的函数去拦截程序使用微软的函数（有点拗口）。有点以毒攻毒的味道呢。

![7](https://bbs.kanxue.com/upload/attach/202008/899080_6PCCDCDCZM4MTFV.png)

当一个程序被注册了调试程序，他们关系就会发生3600°的变化。本来互不干扰的俩，被注册之后只要产生了调试事件，它就会过问调试自己的程序，把自己的控制权交给调试者。而我们正是利用这个原理，注册成为别人的程序的调试者，再促使被调试程序产生断点调试事件，来获得其生死大权。

如何促使被调试程序产生断点调试事件呢？这里有两个关键点：

1. 当程序被注册调试器成功后，会发送一个调试事件来告诉调试者你成功了。

2. 当被调试的程序遇到16进制为CC的指令就会产生中断的调试事件。

结合这两点，我们可以在注册成功的时候，改写相应的地址为CC，使其下次遇到CC时触发中断调试事件。

![8](https://bbs.kanxue.com/upload/attach/202008/899080_UNPUGVHT5AVB3TD.png)

这样我们接收CC的中断调试事件后便可更改被调试程序的相关数据。

# 实践

实践步骤需要有C++的基础知识，否则可能会难以下咽，没有C++的基础可以跳过。

整个过程如下：

附加要修改的程序使目标程序成为被调试者——>收取附加成功消息——>找到要修改的地址并下断点——>触发断点——>收到异常，程序断下——>实现自己的操作（修改数据）——>让程序继续运行

## 步骤一：使目标程序成为被调试者

为什么要先附加程序，使其成为被调试者？从流程上讲因为他是必须的，就像有了加法才有乘法，有了减法才有除法一样。

**而从原理上讲，附加程序就是向目标程序注册成为调试器，只有拥有调试与被调试的关系后，每当目标程序产生调试事件（如断点产生的异常）后才会报告给调试者。因此我们附加程序，是为了获得目标程序产生异常而造成的空隙，这样我们才有修改数据等操作的机会。**

从代码中看，使用`DebugActiveProcess`函数，同时传递目标程序PID作为参数，来使特定的程序成为被调试者，而自己的程序成为调试者。以下为[函数文档](elink@3dfK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3V1k6%4K9h3^5K6x3W2)9J5c8X3q4H3K9g2)9J5c8X3c8W2j5Y4g2Y4j5i4m8A6i4K6u0r3L8X3k6Q4x3X3c8V1k6h3u0#2k6$3q4H3K9g2)9J5k6r3c8W2j5Y4g2Y4j5h3y4@1K9i4k6W2M7s2u0G2j5$3g2K6M7H3`.`.)

```
BOOL DebugActiveProcess(

  DWORD dwProcessId

);
```

## 步骤二：收取附加成功消息

为了给特定地址下断点，我们需要合适的机会，而这个机会便是程序被附加成功的时候。因此我们只要在接收到附加成功的事件后进行断点操作即可。

[同样，贴心的微软提供了一个接收调试事件的方法给我们：](elink@6c5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3V1k6%4K9h3^5K6x3W2)9J5c8X3q4H3K9g2)9J5c8X3c8W2j5Y4g2Y4j5i4m8A6i4K6u0r3L8X3k6Q4x3X3c8V1k6h3u0#2k6$3q4H3K9g2)9J5k6s2N6S2K9i4c8X3L8%4u0V1k6h3u0#2k6$3g2$3k6h3&6@1)

```
BOOL WaitForDebugEvent(

  LPDEBUG_EVENT lpDebugEvent,  //  接收调试事件存储的地方

  DWORD         dwMilliseconds  //  等待事件的时间，可设置永久

);
```

`WaitForDebugEvent`函数会接收调试事件，例如附加成功，遇到断点产生异常等。那我们如何才知道哪个事件是附加成功呢？

[当事件接收成功后，会将一个叫`DEBUG_EVENT`的结构体存储在函数的第一个参数`lpDebugEvent`中，这个结构体的结构如下](elink@73bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3X3c8#2M7#2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3V1k6%4K9h3^5K6x3W2)9J5c8X3q4H3K9g2)9J5c8X3#2A6L8Y4N6A6L8X3u0S2M7$3g2Q4x3V1k6F1M7#2)9J5k6r3#2A6L8Y4N6A6L8X3u0S2M7$3g2Q4x3X3c8V1k6h3u0#2k6#2)9#2k6X3g2$3k6h3&6@1)
