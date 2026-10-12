
# 《精通 UNIX 下 C 语言编程及项目实践》学习笔记 01 起步篇

> 原文：[`www.pediy.com/kssd/pediy10/86639.html`](https://www.pediy.com/kssd/pediy10/86639.html)

《精通 UNIX 下 C 语言编程及项目实践》学习笔记目录

[`bbs.pediy.com/showthread.php?t=86392`](http://bbs.pediy.com/showthread.php?t=86392)

## 第一篇：起步篇

  千里之行，始于足下。

### 一、UNIX 初步

UNIX 自 1969 年诞生以来，已经发展为 `System III & V`、`BSD` 和 `Linux` 三大分支。

`Unix` 通过 `shell` 与用户交互，它是用户与系统间的界面。使用好 `shell` 对于学习使用 `UNIX` 来说是必须的。不需要你记住所有的命令，但基础的文件操作、目录操作及系统命令等却是必须的。

`Vi` 对于初学者是难点，不过只要通过一段时间的练习就能习惯；而且你会很快发现，它功能强大、更加灵活。这里不多说了：）

### 二、编程套件

学习 `UNIX` 对于初学者有几种选择。一种就是最直接的，在本机上安装 `UNIX` 系统，不如说 `Linux` 分支中的 `Red Hat` 等；一种是在 `Windows` 环境下使用虚拟机方式安装 `UNIX` 系统；另一种则是使用网络终端登录到网络环境中的某个 `UNIX` 系统中。

如果你相对黑洞洞的 `UNIX` 字符界面更喜欢舒适美观的 `Windows` 界面，那么推荐你选择第三种方式。开发套件包括：编辑器 `UltraEdit`、网络终端 `SecureCRT` 或其他。

编译器就不用说了，自然是 `cc`（`gcc` 或 `xlc` 系列）。这是 C 语言开发必不可少的。其中要注意`-I`（加载头文件路径）、`-L`（加载库文件路径）及`-D`（宏定义）参数的使用。

`Make` 工具使用。如何编写 `makefile` 是关键。后面项目中使用时会详细介绍。

`Gdb` 调试器。`Gdb` 乃符号级调试工具，它控制程序的内部执行，利用断点设置、单步运行等手段，将程序的执行过程逐步展示在调试者目前。这种调试方式在短代码中可以发挥得很好。事实上，随着软件项目的扩大化、复杂化和分布化，很少有程序员直接通过 `Gdb` 等工具调试；使用日志记录调试方法比 `Gdb` 等调试工具更为便捷和广泛。

C 工具：`lint` 检查源代码是否正确，`gprof` 分析程序时间消费量，`cflow` 生成 C 语言流程图。

### 三、库的使用

库分静态库和动态库两种。

静态库的操作工具：`ar` 命令。

#### 编写及使用静态库

(1) 设计库源码 `pr1.c` 和 `pr2.c`

代码:

```
[root@billstone make_lib]# cat pr1.c
void print1()
{
    printf("This is the first lib src!\n");
}
[root@billstone make_lib]# cat pr2.c
void print2()
{
    printf("This is the second src lib!\n");
}
```

(2) 编译 `.c` 文件

代码:

```
[bill@billstone make_lib]$ cc -O -c pr1.c pr2.c
[bill@billstone make_lib]$ ls -l pr*.o
-rw-rw-r-- 1 bill bill 804 4 月 15 11:11 pr1.o
-rw-rw-r-- 1 bill bill 804 4 月 15 11:11 pr2.o
```

(3) 链接静态库

为了在编译程序中正确找到库文件，静态库必须按照 `lib[name].a` 的规则命名，如下例中 `[name]=pr`。

代码:

```
[bill@billstone make_lib]$ ar -rsv libpr.a pr1.o pr2.o
a - pr1.o
a - pr2.o
[bill@billstone make_lib]$ ls -l *.a
-rw-rw-r-- 1 bill bill 1822 4 月 15 11:12 libpr.a
[bill@billstone make_lib]$ ar -t libpr.a
pr1.o
pr2.o
```

(4) 调用库函数代码 `main.c`

代码:

```
[bill@billstone make_lib]$ cat main.c
int main()
{
    print1();
    print2();

    return 0;
}
```

(5) 编译链接选项

`-L` 及`-l` 参数放在后面。其中，`-L` 加载库文件路径，`-l` 指明库文件名字。

代码:

```
[bill@billstone make_lib]$ gcc -o main main.c -L./ -lpr
[bill@billstone make_lib]$ ls -l main*
-rwxrwxr-x 1 bill bill 11805 4 月 15 11:17 main
-rw-rw-r-- 1 bill bill 50 4 月 15 11:15 main.c
```

(6) 执行目标程序

代码:

```
[bill@billstone make_lib]$ ./main
This is the first lib src!
This is the second src lib!
[bill@billstone make_lib]$
```

#### 编写动态库

(1) 设计库代码

代码:

```
[bill@billstone make_lib]$ cat pr1.c
int p = 2;

void print(){
    printf("This is the first dll src!\n");
}
[bill@billstone make_lib]$
```

(2) 生成动态库

代码:

```
[bill@billstone make_lib]$ gcc -O -fPIC -shared -o dl.so pr1.c
[bill@billstone make_lib]$ ls -l *.so
-rwxrwxr-x 1 bill bill 6592 4 月 15 15:19 dl.so
[bill@billstone make_lib]$
```

#### 动态库的隐式调用

在编译调用库函数代码时指明动态库的位置及名字，看下面实例

代码:

```
[bill@billstone make_lib]$ cat main.c
int main()
{
    print();

    return 0;
}
[bill@billstone make_lib]$ gcc -o tdl main.c ./dl.so
[bill@billstone make_lib]$ ./tdl
This is the first dll src!
[bill@billstone make_lib]$
```

当动态库的位置或名字发生改变时，程序将无法正常运行；而动态库取代静态库的好处之一则是通过更新动态库而随时升级库的内容。

#### 动态库的显式调用

显式调用动态库需要四个函数的支持，函数 `dlopen` 打开动态库，函数 `dlsym` 获取动态库中对象基址，函数 `dlerror` 获取显式动态库操作中的错误信息，函数 `dlclose` 关闭动态库。

代码:

```
[bill@billstone make_lib]$ cat main.c
#include <dlfcn.h>

int main()
{
    void *pHandle;
    void (*pFunc)();                  // 指向函数的指针
    int *p;

    pHandle = dlopen("./d1.so", RTLD_NOW);     // 打开动态库
    if(!pHandle){
        printf("Can't find d1.so \n");
        exit(1);
    }
    pFunc = (void (*)())dlsym(pHandle, "print");     // 获取库函数 print 的地址
    if(pFunc)
        pFunc();
    else
        printf("Can't find function print\n");
    p = (int *)dlsym(pHandle, "p");             // 获取库变量 p 的地址
    if(p)
        printf("p = %d\n", *p);
    else
        printf("Can't find int p\n");
    dlclose(pHandle);                    // 关闭动态库

    return 0;
}
[bill@billstone make_lib]$ gcc -o tds main.c -ldl
[bill@billstone make_lib]$ ./tds
This is the first dll src!
p = 2
[bill@billstone make_lib]$
```

上面的程序 `tds` 显式调用了共享库 `d1.so` 中的函数 `print` 和变量 `p`。

《精通 UNIX 下 C 语言编程及项目实践》学习笔记目录

[`bbs.pediy.com/showthread.php?t=86392`](http://bbs.pediy.com/showthread.php?t=86392)
