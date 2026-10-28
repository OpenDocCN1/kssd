
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24  25  26  27  28  29  30  31  32  33  34  35  36  37  38 |
```
rule APT9002Code
{
    meta:
    description = "9002 code features"
    author = "Seth Hardy"
    last_modified = "2014-06-25"
    strings:
    /`/` start code block
    $ = { B9 7A 21 00 00 BE ?? ?? ?? ?? 8B F8 ?? ?? ?? F3 A5 }
    /`/` decryption from other variant with multiple start threads
    $ = {
    8A 14 3E 8A 1C 01 32 DA 88 1C 01 8B 54 3E 04 40 3B C2 72 EC }
    condition:
    any of them
}
rule APT9002Strings
{
    meta:
    description = "9002 Identifying Strings"
    author = "Seth Hardy"
    last_modified = "2014-06-25"
    strings:
    $ = "POST http://%ls:%d/%x HTTP/1.1"
    $ = "%%TEMP%%\\%s_p.ax" wide ascii
    $ = "%TEMP%\\uid.ax" wide ascii
    $ = "%%TEMP%%\\%s.ax" wide ascii
    /`/` also triggers on surtr
    $ = "mydll.dll\x00DoWork"
    $ = "sysinfo\x00sysbin01"
    $ = "\\FlashUpdate.exe"
    condition:
    any of them
}
```

1. 字符串

2. 二进制代码

### 3.1.1 排查

可以先对代码中的所有字符串进行注释，排查是否有敏感字符串。

如果没有敏感可以再对代码进行注释二分查找。

如果都没有问题，那么就是启发扫描的原因。

### 3.1.2 绕过

对于字符串被杀，可以对其进行加密，使用时在解密。

对与二进制代码被杀，可以加入花指令、混淆，或者修改代码逻辑。

对与启发扫描，可以对程序添加资源、图标、描述信息、签名等等来增加程序的可信度，或者直接`shellcode`执行。

总的来说：`分离加载`是一劳永逸的方案。

## 3.2 执行时被查杀

执行时被查杀，这就是100%的被主动防御拦截了。

### 3.2.1 原理

对于`Windows API`的调用，最终会由`ntdll.dll`转入内核，那么只要`hook`掉`ntdll`导出的函数就能对`api`的调用进行监控。`hook ntdll`的技术也分为应用层和内核层。

对于应用层，可以直接`inline hook jmp`到自己的`hook`函数就行。

对于驱动层，`x32`系统可以使用`hook ssdt`函数来进行监控，`x64`系统由于`PatchGuard`机制的存在是不能直接`hook`的，但是也会有一些骚操作来绕过`pg`实现`hook`，比如`ETW`（https://bbs.pediy.com/thread-258352.htm）、`VT`（360核晶）。

### 3.2.2 绕过

对于应用层`hook`的杀软，`直接系统调用是最好的选择`，具体实现有两种方案：

1. 自己用汇编实现一遍`api`的代码，直接从`ida`里复制就行。

2. 手工`map`一份`ntdll`到内存，然后调用这段内存里的接口（`Themida`的实现方案）。

对于驱动层`hook`的杀软，在应用层基本上是无能为力的，最多就是能找一个白名单的进程，在它内部调用是最好的。（白加黑）

## 3.3 执行后被查杀

这种情况就是程序被后台上传到云端，被丢进了样本集中训练，如果需要绕过它，就需要了解一下机器学习。

机器学习的诞生，是用来解决算法解决不了的问题，需要靠样本来训练，而且只能解决一种问题，比如用来做人脸识别的样本是识别不了花的品种的。那么绕过机器学习的方法，就是产生一个机器不认识的程序。

换句话说也就是一个非常规的`pe`文件结构（混淆），现成的方案有：

1. 花指令

2. 虚拟机

3. 使用其他编译型语言，依靠自身编译器的特性，产生特定的`pe`文件结构

4. 自实现编译器，这个就有点扯了，不过实现`OLLVM`的`IR`是具有可行性的

# 4、展望

以上是我本人对`C/C++`免杀的一些思考，对于真正的一线实战，早已不局限于此。基于`powershell` `.net` `python`等非编译型语言的代码，反而在解释器（正规签名，自带白属性）的保护下能有更大的操作空间。不过我对此也还没有开始进行深入研究，希望有机会和大家一起交流。

`免责声明：此文章只在看雪论坛发布，只可用于看雪论坛的朋友交流学习，禁止任何人转载到其他站点，禁止用于任何非法用途。如有任何人凭此做何非法事情，均于本人无关，特此声明。`

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于 2021-12-26 19:46

被劫局丶编辑

，原因：
