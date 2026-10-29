
# [翻译]向正在执行的linux程序中注入代码

ps：文章是介绍Linux注入的，个人认为Linux和Android的注入基本相同，故放到Android安全板块了。

原链接：

5abK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3y4G2k6r3g2H3M7X3!0B7k6h3y4@1i4K6u0W2j5$3!0E0i4K6u0r3b7i4u0@1K9h3y4D9k6i4y4Q4x3V1j5K6x3K6x3@1x3q4)9J5c8V1y4G2k6r3g2Q4x3X3c8u0L8X3A6W2j5%4c8A6L8$3&6Q4x3X3c8A6L8Y4c8G2i4K6u0V1f1Y4g2F1L8X3W2F1k6#2)9J5k6p5I4A6L8Y4g2^5i4K6u0V1b7i4m8H3L8r3W2U0j5i4c8A6L8$3^5`.

## 介绍

假设Linux平台上有一个正在执行的程序，并且这个程序在很长的时间内都不会退出，就像UNIX的daemons程序。需求是矛盾的，现在需要对这个程序做一些很微小的修改，但是又不想终止程序，这时应该怎么办？可以考虑向程序中注入需要修改的代码，当另一个已存在的函数被调用时会触发注入的代码。这个有点虚构的例子是为了说明我们的需求，为什么有时候需要向正在执行的程序中注入代码呢？这也和病毒的注入技术有关。

这篇文章里面，我会阐述向执行着的Linux程序中注入C函数的可行性。我们还将要讨论Linux的目标文件ELF（Executable and Linkable Format）：节区（sections）、符号（symbols）和重定位（relocations）。

## 代码总览

我会用下面简单的例子一步一步来解释代码注入技术。示例包括3个部分：

1. 动态连接库（Dynamic library）`libdynlib.so`，由C++代码`dynlib.hpp`和`dynlib.cpp`生成。
2. 目标程序`app`，由代码`app.cpp`和`libdynlib.so`库链接生成。
3. 要注入的代码，即`injection.cpp`文件。

首先回顾下三部分的代码：

```cpp
// dynlib.hpp

extern "C" void print();
```

头文件`dynlib.hpp`声明了`print()`函数。

```cpp
// dynlib.cpp

#include <stdlib.h>
#include <iostream>
#include "dynlib.hpp"
using namespace std;

extern "C" void print()
{
    static unsigned int counter = 0;
    ++counter;
    cout << counter << ": PID " << getpid() << ": In print() " << endl;
}
```

源文件`dynlib.cpp`实现了`print()`函数，`print()`函数只打印一个计数器（这个计数器每有一次调用就增加1），程序获取PID并输出一个消息。

```cpp
// app.cpp

#include <dlfcn.h>
#include <iostream>
#include "dynlib.hpp"
using namespace std;

int main()
{
    while (1)
    {
        print();
        cout << "Going to sleep ..." << endl;
        sleep(3);
        cout << "Waked up ..." << endl;
    }
    return 0;
}
```

上面的`app.cpp`程序在一个循环里面调用`print()`函数（`print()`函数位于`libdynlib.so`动态库中），然后睡眠几秒钟，之后继续执行这样的循环。

```cpp
// injection.cpp

#include <stdlib.h>
extern "C" void print();

extern "C" void injection()
{
    print(); // do the original job, call the function print()
    system("date"); // do some additional job
}
```

在`main()`函数里调用`print()`函数会被`injection()`函数替换。`injection()`函数首先调用原始的`print()`函数，之后会做一些额外的操作。比如，你可以用`system()`函数来执行一些其他可执行文件，或者就像我做的直接打印当前日期。

## 编译并执行程序

接下来我们用`g++` C++编译器和`gcc` C编译器来编译程序。

```bash
g++ -ggdb -Wall dynlib.cpp -fPIC -shared -o libdynlib.so
g++ -ggdb app.cpp -ldynlib -ldynlib -L./ -o app
gcc -Wall injection.cpp -c -o injection.o
```

```text
-rwxr-xr-x  1 gregory ftp  52248 Feb 12 02:05 app
-rw-r--r--  1 gregory ftp   1088 Feb 12 02:05 injection.o
-rwxr-xr-x  1 gregory ftp  52505 Feb 12 02:05 libdynlib.so
```

需要注意的是动态连接库`libdynlib.so`编译链接的时候需要用`-fPIC`参数，这样生成的代码是位置无关的（我也不懂什么是位置无关的，原文是 produces position independent code，有懂的请不吝赐教），编译`injection`的目标文件需要用C编译器。编译完成就可以执行`app`程序了。

```text
[lnx63:code_injection] ==> ./app
1: PID 4184: In print()
Going to sleep ...
Waked up ...
2: PID 4184: In print()
Going to sleep ...
Waked up ...
3: PID 4184: In print()
Going to sleep ...
```

## 进入调试器

`app`程序刚执行几个循环，但我们假定它已经跑了好几个礼拜了，这样的话我们就可以现在注入我们的代码了。在注入的过程中会用到Linux `gdb`调试器。首先需要把调试器附加到4184进程上，看上面打印出来的PID（application process id）。

```text
[lnx63:code_injection] ==> gdb app 4184

GNU gdb 6.3
Copyright 2004 Free Software Foundation, Inc.
GDB is free software, covered by the GNU General Public License, and you are
welcome to change it and/or distribute copies of it under certain conditions.
Type "show copying" to see the conditions.
There is absolutely no warranty for GDB.  Type "show warranty" for details.
This GDB was configured as "i686-pc-linux-gnu"...
        Using host libthread_db library "/lib/tls/libthread_db.so.1".
Attaching to program: /store/fileril104/project/gregory/code_injection/app, process 4184
Reading symbols from
        /store/fileril104/project/gregory/code_injection/libdynlib.so...done.
Loaded symbols for /store/fileril104/project/gregory/code_injection/libdynlib.so
Reading symbols from /usr/lib/libstdc++.so.6...done.
Loaded symbols for /usr/lib/libstdc++.so.6
Reading symbols from /lib/tls/libm.so.6...done.
Loaded symbols for /lib/tls/libm.so.6
Reading symbols from /lib/libgcc_s.so.1...done.
Loaded symbols for /lib/libgcc_s.so.1
Reading symbols from /lib/tls/libc.so.6...done.
Loaded symbols for /lib/tls/libc.so.6
Reading symbols from /lib/ld-linux.so.2...done.
Loaded symbols for /lib/ld-linux.so.2
0x006e17a2 in _dl_sysinfo_int80 () from /lib/ld-linux.so.2
(gdb)
```

## 把注入代码加载到目标进程中

上面已经提到过，目标文件`injection.o`并不会初始化到`app`程序的进程中。首先我们需要把`injection.o`加载到`app`程序的地址空间里面。可以通过`mmap()`系统调用来把`injection.o`映射到`app`程序的地址空间里面。我们通过调试器来完成这个过程。

```text
(gdb) call open("injection.o", 2)
$1 = 3
(gdb) call mmap(0, 1088, 1 | 2 | 4, 1, 3, 0)
$2 = 1073754112
(gdb)
```

首先以读/写的权限（`O_RDWR`值为2）打开`injection.o`文件。需要写的权限是因为后面会对已经加载的注入代码做些修改。返回的分配好的文件描述符是3。接下来用`mmap()`调用把打开的文件写入（原文为bring the file into）到`app`的地址空间里面。`mmap()`接收的参数有文件大小（1088字节）、映射后文件的权限-`PROT_READ|PROT_WRITE|PROT_EXEC`（读/写/执行 对应的是`1|2|4`），还有文件描述符-3。返回的结果是映射到`app`地址空间里的文件的起始地址-1073754112。我们可以通过查看`/proc/[pid]/maps`（`pid`是可执行程序的进程id-我们的例子里面是4184）来确认`injection.o`被映射到`app`的地址空间了。`/proc/[pid]/maps`文件包括正在执行的进程的内存布局。

```text
[lnx63:code_injection] ==> cat /proc/4184/maps
006e1000-006f6000 r-xp 00000000 fd:00 394811     /lib/ld-2.3.4.so
006f6000-006f7000 r-xp 00015000 fd:00 394811     /lib/ld-2.3.4.so
006f7000-006f8000 rwxp 00016000 fd:00 394811     /lib/ld-2.3.4.so
006ff000-00824000 r-xp 00000000 fd:00 394812     /lib/tls/libc-2.3.4.so
00824000-00825000 r-xp 00124000 fd:00 394812     /lib/tls/libc-2.3.4.so
00825000-00828000 rwxp 00125000 fd:00 394812     /lib/tls/libc-2.3.4.so
00828000-0082a000 rwxp 00828000 00:00 0
00832000-00853000 r-xp 00000000 fd:00 394813     /lib/tls/libm-2.3.4.so
00853000-00855000 rwxp 00020000 fd:00 394813     /lib/tls/libm-2.3.4.so
0096e000-00975000 r-xp 00000000 fd:00 394816     /lib/libgcc_s-3.4.6-20060404.so.1
00975000-00976000 rwxp 00007000 fd:00 394816     /lib/libgcc_s-3.4.6-20060404.so.1
00978000-00a38000 r-xp 00000000 fd:00 45535      /usr/lib/libstdc++.so.6.0.3
00a38000-00a3d000 rwxp 000bf000 fd:00 45535      /usr/lib/libstdc++.so.6.0.3
00a3d000-00a43000 rwxp 00a3d000 00:00 0
08048000-08049000 r-xp 00000000 00:34 30468731   /store/fileril104/project/gregory/
                                                code_injection/app
08049000-0804a000 rwxp 00000000 00:34 30468731   /store/fileril104/project/gregory/
                                                code_injection/app
0804a000-0806b000 rwxp 0804a000 00:00 0
40000000-40001000 r-xp 00000000 00:34 30468725   /store/fileril104/project/gregory/
                                                code_injection/libdynlib.so
40001000-40002000 rwxp 00000000 00:34 30468725   /store/fileril104/project/gregory/
                                                code_injection/libdynlib.so
40002000-40003000 rwxp 40002000 00:00 0
40003000-40004000 rwxs 00000000 00:34 30468724   /store/fileril104/project/gregory/
                                                code_injection/injection.o
4000f000-40011000 rwxp 4000f000 00:00 0
bfffe000-c0000000 rwxp bfffe000 00:00 0
ffffe000-fffff000 ---p 00000000 00:00 0
```

【未完，续不续不知道![](https://bbs.kanxue.com/view/img/face/12.gif)】

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
