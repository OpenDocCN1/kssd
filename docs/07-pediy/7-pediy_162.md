
# diy pe5

> 原文：[`www.pediy.com/kssd/pediy05/pediy50243.htm`](https://www.pediy.com/kssd/pediy05/pediy50243.htm)

申明，这批文章没有什么技术含量，我仅仅是向大家提供一个得到注册码方法。

程序：某国产软件 `xxx.exe`

注册码检验方式：网络校验

原文件加壳：未知壳

脱壳用 `trw` 找到入口点，`pedump`，修复 `iat`，过程省略（很简单，不值得写）。

脱壳后继续跟踪找到注册码判断点：

```asm
mov edx, xxxxxx        ; edx 是网络注册码
mov eax, xxxxxx        ; eax 是自己输入的注册码
call xxxxxx            ; 这段是比较的 call
jz xxxxxx              ; 不等就完蛋
```

由于是网络校验，这个形式调用了很多次，每次都能 `d edx` 看到不同的注册码。可见是把你输入的注册码和网络数据库里的注册码，逐个比较，如果相等就成功，直接改那个 `jz` 就 `ok` 了，不过我们如何得到这个数据库所有的注册码呢？

我一开始的想法是用 `messagebox`，更改其跳转到自己的程序：

```asm
push 0
push edx
push edx
push 0
call messagebox
```

这样不是就能看到注册码么？但是有个缺点就是一次只能看到一个，而且还有可能导致网络超时。

于是就换种思路，再每次调用的时候保存 `dx` 的注册码，最后再 `ExitProcess` 的时候把所有的注册码写到一个文件里面。这样的话就需要先 `VirtualAlloc` 内存，然后 `creatfile`，在 `lstrcpy`，然后 `writefile`，最后 `closefile` 和 `VirtualFree` 这片内存。

真是麻烦之又麻烦，如果程序没有这些函数，还要自己构造相关函数，对我这种能简化就简化的人来说，是比较痛苦的一件事。但是在实际操作的时候，突然想到，我写上篇文章的时候有过写 `ini` 的 `api`，马上想到是 `WritePrivateProfileString`，查 `api` 手册看看：

```c
BOOL WritePrivateProfileString(
    LPCTSTR lpAppName,    // pointer to section name
    LPCTSTR lpKeyName,    // pointer to key name
    LPCTSTR lpString,     // pointer to string to add
    LPCTSTR lpFileName    // pointer to initialization filename
);
```

真是个好东东啊，不要分配内存，也不要打开文件句柄，只要这个四个指针就 `ok`。

实际操作的时候就可以这样来更改，当然首先更改原来的程序跳到自己的程序。

自己的程序：

`inc key` ＝＞每次都改变一下 `key` 的内容，不要把所有的注册码都写到同一个 `key` 里面，那岂不是白干了：）

`push filename` ==> 文件名 (`xxx.ini`) 的指针，`*` 自己在程序中找个空白的地方用 `winhex` 添一个就行了，然后转换成 `rav` `push` 这个 `rav`

`push edx` ==> 本来是 `push` 要更改的 `string` 地址，这里就是 `edx` 了，我们的注册码

`push key` ==> `key` 名称的指针

`push sectionname` ==> 段名称指针（我实际构造的时候这个段名称就是 `serial`）

`call WritePrivateProfileString`

`mov eax，xxxxxx` ＝》恢复原程序动作

`jmp xxxxxx` ＝》跳回原程序

程序执行完后，就可以去 `c:\windows`（也就是你安装 `windows` 的目录）里面找这个 `xxx.ini` 文件了，打开一看，哈哈所有的注册码排列得整整齐齐。

真是 `happy` 啊，由此拓展，用这样的方法可以记录很多有用的信息给我们，比如你想了解某个地址，堆栈，寄存器等的变化情况，又不想去一次一次中断看，就插一段这样的语句，是不是很省事情哦。

后记：

希望大家在操作的时候多动脑筋，没有作不到，只有想不到，很多看似简单的东西，其实还是有很大的用处的。

你的朋友：`pll621`  `4dd466d0e1e0cc123afbe504da2462ca`
