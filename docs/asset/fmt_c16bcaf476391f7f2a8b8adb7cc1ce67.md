
# Windows 驱动学习路径与学习记录

## 发展阶段

若对 Windows 底层开发没有兴趣，不建议继续深究；若有些兴趣可以继续。

1. 先广泛打基础，比如 `C`/`MFC`/`C++`/`ASM`，再学习 `Windows` 核心编程，对 R3 上的一些开发有所熟悉；再系统的学习《操作系统》等书籍，理解整个系统的原理，构架，实现。

2. 有了以上基础，可以开始阅读一些驱动入门书籍，如毛德草的《Windows 内核情景分析》，多上 `google` 搜索资料；下一份 `WRK`/`ReactOS`，没事翻一翻，搭好驱动编译环境后，可以尝试写些小驱动，在实践蓝屏中摸索总结，从而积累初级的经验。

3. 等成了驱动开发初级工后，可开始学习 `Windbg`、`IDA` 的使用。多看源码多 `F1` 看帮助文档。

4. 等熟练以上工具后，开始 `Windbg` 动态调试，`IDA` 静态逆向其他驱动（包括微软自己的）。在这一过程中，你又会接触到脚本语言，汇编知识点，`Intel` 手册，加密解密，`Vmware` 等一堆的东西，同样你需要熟悉它们。

5. 等你熟悉以上东西后，可以给自己提些需求，并实现该需求；比如写个小型 `ARK`；在这个过程中，你可以切实感受到开发一个程序是一个系统的东西，你又需要回到 R3 写界面，重温 `MFC`、`WTL` 等设计与使用。当然又要写驱动程序，保证兼容性等问题；这个过程是漫长的，期间你会发现写一小部分功能，你可以扩展收获很多知识点。

6. 等你熟悉以上东西后，已经可以自己独立解决问题了。基本不需要到网上求助；遇到问题，会利用 `WRK`/`Windbg`/`IDA`/`Google`等方式自行搞定；如网上有现成的解决方法，借用之，取其精华，唾其糟粕；若网上没有现成的，则需要你 `IDA` 系统文件，`load pdb`；`windbg` 动态调试内核等手段自行挖掘；若网上只有类似的程序，你可以逆向其关键部分参考之，而后变成自己的东西。

7. 等你经历了以上 7 个阶段，你可以开始系统的了解 `WINDOWS` 的构架，站在产品的角度思考问题，分析问题，解决问题。对自己多提需求。进而再去实现之。这阶段是积累经验的阶段...

8. 基本能够胜任大多数公司的项目需求了。

## 学习近况

刚把日语 1 级的 99 个文法看完，感觉忘得好快，还在不断温习中，日语一级听力没有 `NHK` 说的快，但是有的听力我看了原文都没选出答案，350 块钱呢。前几天，我彻底下定决心不碰电脑方面的事，这四个月全都学习日语了冲刺一级了，最好是一级过了，大三的时候向各科老师请假，专心学习 `Windows` 驱动程序设计，然后在大三十月份的时候 `360` 来哈工大招人，我去哈工大试试，然后大四的时候 `Windows` 驱动 `IDA` `windbg` 就学的差不多了，即使去不了 `360` 找个做软件的小公司也应该能找得到了，在公司学到的东西是最多的了，因为会获得实际项目开发的经验。和 `sudami` 那个时候正好相反，我觉的将来我工作的岗位很可能是计算机方面的，而不是日语方面的。

感觉学计算机比学日语难的，但是对计算机比对日语有兴趣。寒假（寒假效率很高，一般早晨 9 点起来中午睡一个小时晚上俩点睡觉也玩过，加起来玩了 10 天包括过年的那几天）看了几本书，没有全看完，但是哪一本书都可以和新编日语 4 册加起来对抗一番。

- `Windows` 核心编程 16 开 1000 多页
- `Windows` 程序设计 上下册 32 开 1300 多页
- `Windows` 内核情景分析--基于 `ReactOS` 1400 多页
- `深入解析 Windows` 系统 16 开 800 多页

我的日语学的不好，英语也学的不怎么样，但是 `MSDN` 和 `WDK` 还看的挺顺畅的，

粘一段 `MSDN` `CreateProcess` 参数的调用 英语虽然不是很好 但是看 `MSDN` 里面的东西看懂大意还是不成问题的。

```
// 
BOOL CreateProcess(
  LPCTSTR lpApplicationName,                // name of executable module
  LPTSTR lpCommandLine,                     // command line string
  LPSECURITY_ATTRIBUTES lpProcessAttributes, // SD
  LPSECURITY_ATTRIBUTES lpThreadAttributes,  // SD
  BOOL bInheritHandles,                     // handle inheritance option
  DWORD dwCreationFlags,                    // creation flags
  LPVOID lpEnvironment,                     // new environment block
  LPCTSTR lpCurrentDirectory,               // current directory name
  LPSTARTUPINFO lpStartupInfo,             // startup information
  LPPROCESS_INFORMATION lpProcessInformation // process information
);
```
Parameters

`lpApplicationName`

[in] Pointer to a null-terminated string that specifies the module to execute. The specified module can be a Windows-based application. It can be some other type of module (for example, MS-DOS or OS/2) if the appropriate subsystem is available on the local computer.

The string can specify the full path and file name of the module to execute or it can specify a partial name. In the case of a partial name, the function uses the current drive and current directory to complete the specification. The function will not use the search path. If the file name does not contain an extension, `.exe` is assumed. Therefore, if the file name extension is `.com`, this parameter must include the `.com` extension.

The `lpApplicationName` parameter can be `NULL`. In that case, the module name must be the first white space-delimited token in the `lpCommandLine` string. If you are using a long file name that contains a space, use quoted strings to indicate where the file name ends and the arguments begin; otherwise, the file name is ambiguous. For example, consider the string "c:\\program files\\sub dir\\program name". This string can be interpreted in a number of ways. The system tries to interpret the possibilities in the following order:

c:\\program.exe files\\sub dir\\program name
c:\\program files\\sub.exe dir\\program name
c:\\program files\\sub dir\\program.exe name
c:\\program files\\sub dir\\program name.exe

// 

大学给我的感觉是，老师是好老师，学生不是好学生。

今天是 3 月 10 号 7 月 3 号考试，我打算拼四个月，不管一级过不过，把日语放一放，学 `Windows` 驱动，我感觉这样我的就业几率大于光学日语的几率，而且还是在我的兴趣上就业。

给自己做了一下计划。

### 汇编方面

16 位汇编曾经看了一阵，不用学太多，知道一下中断及其调用就行，还有寻址方式。重头戏是 `Windows` 环境下的汇编。这方面的参考书以 `<<Windows 环境下 32 位汇编语言程序设计>>` 为主，寒假看过电子版，讲的非常好，罗云彬写的，自己买了本。以后用 `windbg` 以动态调试及 `IDA` 反汇编不能少了这方面的支持。汇编就硬着头皮上吧，幸亏以前看了一点 16 位汇编，有一点点帮助，这点比 `sudami` 好点。

### `Windows` 程序设计

学驱动之前必须要熟知 R3 下各种常见的 API 的调用，这方面的书以我买的 `<<Windows 程序设计>>` 为主，着重看一下进程线程文件内存网络方面的 API 调用，`MFC` 一定要学，文档、视类方面的需要的时候再看吧不要把他们当重点，`ATL` `WTL` 只听过名字，调研一下，看需不需要像 `MFC` 一样学习，需要的话还是找与底层联系紧密的方面当重点。

### 驱动方面

这方面以 `<<Windows 内核安全编程>>` 这本书为重点，这本书面面俱到，但是也有一个缺点讲的不细（对于我这种入门级别的小白）。`Windows` 架构方面，`Windows` 内核情景分析太贵了，打折后还要 150 元，手里只有他的电子书版本，以潘爱民老师的那本 `<<Windows 内核原理实现>>` 为准，应该再买一本驱动方面讲的比较细的书来，用来当字典查，由于 `Windows` 不开源，驱动方面的书实在是少之又少，以后还要花一段时间专门学习一下 `HOOK` `SSDT` `HOOK` `SHADOW SSDT` `Inline Hook` 这方面就得多上网查，一定要多多的收藏牛人的博客，那些愿意共享知识的牛人的博客

### 工具使用

以后要学习 `IDA` `windbg` 有可能的话软件软件逆向方面的三环调试器 `ollydbg` 也要学 知识到达一个层次了工具自然而然也就学会用了，虚拟机方面的 `vmare` 早就会用了，这点还比较好。

### 数据结构

一直不理解数据结构是什么东东，后来在某个地方看到了数据结构被加上了个定语，终于明白数据结构是怎么一回事了，数据结构就是计算机中数据结构。原来压根就是在电脑里面数据存在的方式。数据结构这方面就只知道一个栈，先入后出、后入先出。`Windows` 里面还有链表还有二叉树的遍历，这方面可以找找 `A` 帮忙，学数据结构和学 `C++` 一样，结合 `Windows` 中具体实现的方式应该不难。

字数统计了一下，写了不少。

写了这么多就是想和老师说，想不来上课去学写驱动，平常如果布置作业的话我会问一下同学，也会交作业的。也许自己有点任性，请老师原谅。
