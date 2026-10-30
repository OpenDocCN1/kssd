
# windows 2000系统的一个0day（大概）漏洞的发现过程

## 0x00-前言

其实安装这个系统的本意是折腾老游戏，装点菠萝1之类的玩玩，结果找到了个漏洞，不错。

这个漏洞位于`cmd.exe`，是一个缓冲区溢出漏洞，我在公开资料上没有查询到该漏洞的任何信息，也不知道是我没查到，还是确实没人公开它，如果有谁查到这个漏洞的信息了记得提醒我一下（……）

最后，作为本科生我水平有限，如果文章中出现错误或是有其它建议，欢迎各位大佬指教|д`)

**实验环境**

Microsoft Windows 2000 5.00.2195 Service Pack 4

**用到的软件工具**

- 010Editor
- ollydbg
- IDA pro

## 0x01-漏洞简述

`cmd.exe`是windows系统传统的命令行小工具，从最早的windows系统一直到现在的windows 11都有这个小工具，当利用这个小工具对**FAT32格式**的磁盘目录进行`dir`操作时，若该目录里的文件的文件名长度**刚好为260个字符**时，`cmd.exe`进程崩溃闪退。

![崩溃闪退的过程](https://bbs.kanxue.com/upload/attach/202302/968912_BDK6JSJ4AV2TCD5.gif)

## 0x02-`cmd.exe`的漏洞发现过程

**文件构造**

本漏洞需要在磁盘上构造一个文件名长度刚好260个字符的特殊文件，但是windows系统在资源管理器或者命令行工具里不允许文件名全路径长度超过260个字符，所以需要借助二进制编辑软件对磁盘分区卷进行直接的编辑长文件名目录项，构造这种特殊的文件。这里我用了010Editor。

![](https://bbs.kanxue.com/upload/attach/202302/968912_RH6QBVT33AZHDNA.png)

我在U盘H分区根目录建立一个共有20个长文件名目录项的文件，由于一个目录项32字节里有26个字节可以存储13个宽字符的文件名，这个文件的长文件名为`aaaaaaaa…..aaaagf`，一共有260个宽字符。

**漏洞跟踪**

将构造好的U盘插入Windows 2000系统，在ollydbg中开启`cmd.exe`进程，如下图：

![](https://bbs.kanxue.com/upload/attach/202302/968912_H4FXNMXRMKU4Z9R.png)

在命令行窗口执行`dir e:`，进程出现异常。这里我用了污点数据追踪的方法来找程序的crash point，也就是说在堆栈区找到我们构造的污点数据`aaaaaaaa…..aaaagf`，在这个数据附近寻找函数一开始压入的返回地址，从而追踪到溢出点，整个过程如下图：

![](https://bbs.kanxue.com/upload/attach/202302/968912_5GMKEJRH5XT6M7K.png)

> 顺便一提，在不同的机器上程序的crash point可能不同，比如说在家里的另一台电脑上进行实验，crash时的Call stack是长这样的：
>
> ![](https://bbs.kanxue.com/upload/attach/202302/968912_WWSV4BS6V5FNJJF.png)
>
> 直接就可以定位到`4AD0E060`的位置，都不用追踪污点数据了。

在`4AD17B34`处调用的`wcscpy`，将字符串拷贝至栈空间`ebp-20c`，该空间大小`0x20c`可以放置262个宽字符，似乎不能造成栈溢出，但事实上在`4AD17B34`处设置断点，再次进行`dir e:`，如下图：

![](https://bbs.kanxue.com/upload/attach/202302/968912_8MS3FHVQFDGQW99.png)

执行完`wcscpy`后，函数的返回地址被覆盖，如下图（执行前、执行后栈`12EA70`处的值）：

![](https://bbs.kanxue.com/upload/attach/202302/968912_78PXS8EWZ49UZSF.png)

![](https://bbs.kanxue.com/upload/attach/202302/968912_M82B744KXAZ7F7D.png)

堆栈缓冲区的溢出，覆盖了函数在栈`12EA70`处的函数返回地址和后续的栈结构，继续进程执行由于栈的破环造成对非法地址的访问而异常终止。

## 0x03-缓冲区溢出漏洞的成因

现在的问题是，我构造的长文件名只有260个宽字符，占有`0x208`个字节的空间，`wcscpy`拷贝函数的目的地址栈空间明明有`0x20c`个字节，为什么有足够的栈空间也造成了溢出?

进一步的追踪源字符串的来源，发现`cmd.exe`程序调用了系统内核模块`kernel32.dll`的导出函数`FindNextFileW`对目录中的文件进行遍历，依次得到目录中的长文件名，如下图：

![](https://bbs.kanxue.com/upload/attach/202302/968912_48VAGKGMFBJ7VBB.png)

Windows2000的`FindNextFileW`函数在对刚好为260个宽字符的长文件名进行的遍历时，获得数据存储在类型为`WIN32_FIND_DATA`的结构中，该结构体的定义如下：

```
typedef struct _WIN32_FIND_DATA {
    DWORD dwFileAttributes; //文件属性
    FILETIME ftCreationTime; // 文件创建时间
    FILETIME ftLastAccessTime; // 文件最后一次访问时间
    FILETIME ftLastWriteTime; // 文件最后一次修改时间
    DWORD nFileSizeHigh; // 文件长度高32位
    DWORD nFileSizeLow; // 文件长度低32位
    DWORD dwReserved0; // 系统保留
    DWORD dwReserved1; // 系统保留
    TCHAR cFileName[MAX_PATH]; // 长文件名
    TCHAR cAlternateFileName[14]; // 8.3格式文件名
} WIN32_FIND_DATA, *PWIN32_FIND_DATA;
```

在windows系统中常常把`MAX_PATH`宏定义为260，`cmd.exe`通过`FindNextFileW`函数遍历得到长文件名和短文件名在内存空间中相邻存储，当磁盘上的长文件名数据刚好260个宽字符时，造成了两个字符串的合并，如下图：

![](https://bbs.kanxue.com/upload/attach/202302/968912_PQD3AK4QXUNYCAF.png)

进一步研究发现`wcscpy`的第二个参数指向的源字符串为从磁盘上得到的长文件名，其后紧接着的为该文件的短文件名，正常情况下的长文件名后按照字符串的结尾规则以`\0`字符结尾表示字符串结束，但当长文件名长度为260个宽字符时，长文件名最后没有结束字符`\0`,长文件名和短文件名变成了同一个字符串，短文件名占据了12个宽字符，这样源字符串的长度为272个宽字符，占据了`0x220`的堆栈空间，从而造成了返回地址及栈结构的覆盖。

## 0x04-Windows XP SP3系统以及后续的Windows系统中是否存在该漏洞?

进一步的，我对Windows XP SP3的`cmd.exe`小工具进行了逆向分析，发现与windows 2000的代码极为相似，不同的是Windows XP SP3启用了GS保护，大多数函数中都进行了堆栈cookie的校验，但是GS保护只能增加漏洞利用的难度，并不能杜绝缓冲区溢出对进程堆栈结构造成的破坏，若程序代码相同应该存在同样的漏洞。如下图：

![](https://bbs.kanxue.com/upload/attach/202302/968912_ZNZP3C73GSPEB5C.png)

在XP系统中，目的字符串的空间为`0x210`仍然小于长文件名+短文件名后的最大长度`0x220`，仍然具备潜在的被溢出覆盖的可能性。

然而事实上，在XP系统下对构造后的U盘进行`dir` 操作时，并没有造成进程闪退，而是将文件名的最后一个字符舍弃后显示文件名，如下图：

![](https://bbs.kanxue.com/upload/attach/202302/968912_YXS44YBXNJZHE5N.png)

这说明在XP系统下该漏洞并不存在，为了进一步澄清两个系统的不同，我对XP系统的`kernel32.dll`的`FindNexeFile`函数进行了逆向，如下图：

![](https://bbs.kanxue.com/upload/attach/202302/968912_DAX4Q46YFJVJKU2.png)

## 0x05-shellcode

Windows 2000系统没有任何GS，ALSR，DEP等后续Microsoft公司在安全方面的提升，所以可以很方便地利用该漏洞编写shellcode执行任意的程序，这里写一个启动计算器的，以下是构造的文件与对应的代码：

![](https://bbs.kanxue.com/upload/attach/202302/968912_9XC5MVX93BXS8DK.png)

![](https://bbs.kanxue.com/upload/attach/202302/968912_NP596T893G4J8CE.png)

最后的效果如下：

![启动计算器的过程](https://bbs.kanxue.com/upload/attach/202302/968912_GGCXQ2A3EWAEEN6.gif)
