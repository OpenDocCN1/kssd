
这是msvc编译器的问题。

比如下面的代码：

```C
//newtest.cpp
#include "iostream"
#include "string"
using namespace std;
int main(int argc,char* argv[]){
string* msg = new string("Hello World");
cout << *msg << endl;
delete msg;
return 0;
}
```

用IDA查看`main`函数的反汇编

![](upload/attach/202012/825187\_HZY78UFXESKX3KW.jpg)

我们发现，并不是直接使用`cout`这个全局变量，而是进行拷贝构造，而且是强转`cout`为父类型的指针后进行的拷贝构造，所以调用的是**父类的拷贝构造**，这还不关键。

![](upload/attach/202012/825187\_24H6V9B9U3KYYJZ.jpg)

![](upload/attach/202012/825187\_XT4CYNVNB5RP6VU.jpg)

关键的是在`endl`函数调用之前，拷贝构造出来的对象就**被析构了**，不要紧，因为我写的析构函数不会改变属性，还是`fp`存储的还是`stdout`，**但是存储`this`指针的是`esp`**。

![](upload/attach/202012/825187\_EBZEKKE8UZNVU57.jpg)

那么就导致在开栈的时候，就把属性`fp`给改变了，就导致`endl`函数的调用失败，我搞不懂msvc编译器怎么不用`ebp-xxx`来存储局部变量

**相反g++编译器就很正常**

局部变量`ebp-xxx`作为`this`对全局变量`cout`进行拷贝构造，然后打印字符串，调用`endl`函数，然后才析构

### 7.2 `minicrt.lib(ctor.obj)` : warning `LNK4210`: 存在 `.CRT` 节；可能有未处理的静态初始值设定项或结束符

这是在编译运行C++库时出现的警告，但不影响运行库的使用

### 7.3 `new`和`delete`的小心得

`new`对象数组的话，会多分配一个`int`，用于在开头存储元素的个数，用于循环析构和传递`delete[]`第二个参数，而对于不是`new`对象数组，那么没必要分配一个`int`，因为不需要循环析构，如下代码：

```C
//test.cpp
#include "iostream"
#include "string"
class Person{
private:
int age;
char* name;
public:
Person(){
printf("Person\n");
}
~Person(){
printf("~Person\n");
}
};
int main(int argc,char* argv[]){
Person* arr = new Person[2];
return 0;
}
```

![](upload/attach/202012/825187\_Z4GQXKQ7G5DB9GR.jpg)

对于`delete[]`的使用，不是对象数组，那么`delete`和`delete[]`没有区别，但是对于对象数组，只有`delete[]`才会析构每个对象和正确释放内存，但如果用`delete`对象数组的话，只会调用第一个对象的析构函数，并且释放的地址是**原地址+`sizeof(int)`**，这是不对的，就是简单当成一个对象进行析构和释放内存了(以为当前地址就是申请的地址，当然对于一个对象来说就是如此)

### 7.4 附完整库编译命令

```text
C运行库库编译命令
gcc -c -fno-builtin -nostdlib -fno-stack-protector -m32 entry.c malloc.c stdio.c string.c printf.c && ar -rs minicrt.a malloc.o printf.o stdio.o string.o && gcc -c -ggdb -fno-builtin -nostdlib -fno-stack-protector -m32 test.c && ld -static -m elf_i386 -e mini_crt_entry entry.o test.o minicrt.a -o test
cl /c /DWIN32 /utf-8 /GS- entry.c malloc.c printf.c stdio.c string.c && lib entry.obj malloc.obj printf.obj stdio.obj string.obj /OUT:minicrt.lib && cl /c /DWIN32 /utf-8 test.c && link test.obj minicrt.lib kernel32.lib /NODEFAULTLIB /DEBUG /entry:mini_crt_entry
———————————————————————————————————————————————————————
C++运行库库编译命令
cl /c /DWIN32 /GS- /utf-8 entry.c malloc.c printf.c stdio.c string.c atexit.c && cl /c /DWIN32 /GS- /GR- /utf-8 crtbegin.cpp crtend.cpp ctor.cpp new_delete.cpp iostream.cpp && lib entry.obj malloc.obj printf.obj stdio.obj string.obj ctor.obj new_delete.obj atexit.obj iostream.obj /OUT:minicrt.lib && cl /c /DWIN32 /GR- /utf-8 test.cpp && link test.obj minicrt.lib kernel32.lib /NODEFAULTLIB /DEBUG /entry:mini_crt_entry
gcc -c -m32 -fno-builtin -nostdlib -fno-stack-protector entry.c malloc.c stdio.c string.c printf.c atexit.c && g++ -c -m32 -nostdinc++ -fno-rtti -fno-exceptions -fno-builtin -nostdlib -fno-stack-protector crtbegin.cpp crtend.cpp ctor.cpp new_delete.cpp sysdep.cpp iostream.cpp && ar -rs minicrt.a malloc.o printf.o stdio.o string.o ctor.o atexit.o iostream.o new_delete.o sysdep.o && g++ -c -m32 -nostdinc++ -fno-rtti -fno-exceptions -fno-builtin -nostdlib -fno-stack-protector test.cpp && ld -static -m elf_i386 -e mini_crt_entry entry.o crtbegin.o test.o minicrt.a crtend.o -o test
```

## 8. 完整源码下载

**见附件 密码均为：kanxue**

`C_minicrt.zip`：C运行库源码

`CPP_minicrt.zip`：C++运行库源码

![](https://bbs.kanxue.com/upload/attach/202012/825187_SEEZF3KSU25Q6UQ.jpg)

## 1. 背景

为了总结在《程序员的自我修养--链接、装载与库》这本书中的“运行库实现”章节学到的知识，现编写“运行库实现”一文。因为要讲清楚实现运行库的细节是比较复杂的，同时也限于篇幅和避免复杂繁琐，所以本文只会提及关键实现处并给出源代码，同时给出相关知识点在《程序员的自我修养--链接、装载与库》中的相关章节和相关参考资料，如有不理解的地方请回帖或私信我，鉴于本人才疏学浅，如有错误，欢迎指正。

## 2. 声明

1.为了便于读者理解，本文大量复制《程序员的自我修养--链接、装载与库》中的文字，如有侵权，请管理员删帖或联系我删帖。

2.《程序员的自我修养--链接、装载与库》已经给出了“运行库实现”的完整源代码，且本文中的绝大多数源代码都是直接复制其中的代码（如有侵权，请管理员删帖或联系我删帖），但是因为书籍年代久远（09年出版的）和实验环境的改变的原因，我修改了一部分源代码以顺利运行。

## 3. 目的

实现**32位简单版**Windows和Linux双平台的C++运行库

- CRT库功能
  - 入口函数
  - 初始化
  - 堆管理
  - 基本IO
- C++库功能
  - `new`/`delete`
  - 类：`stream`、`string`

## 4. 意义

帮助理解CRT和C++运行库的结构，从而举一反三

## 5. 实现

我们先实现CRT，然后再添加C++运行库的功能

### 5.1 注意点

1.Linux部分的实现代码中的内联汇编使用的是[AT&T汇编](https://bbs.kanxue.com/elink@eacK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3W2T1K9h3u0D9K9h3!0Q4x3X3g2G2M7X3N6Q4x3V1k6Y4k6X3g2J5k6#2)9J5c8X3I4V1M7q4)9J5c8V1N6o6b7#2)9J5k6p5W2F1L8r3W2F1k6g2)9J5k6p5q4K6M7$3g2E0j5X3I4&6i4K6u0V1d9p5!0i4g2p5!0Q4x3X3g2Z5N6r3#2D9i4K6t1K6M7K6b7`.)，和x86汇编相似，但是区别也很大

2.规定源代码中的函数执行失败返回`-1`

3.规定《程序员的自我修养--链接、装载与库》简称《修养》

4.实验环境：

- Linux实验环境：18.04.1-Ubuntu 64位，`gcc` version 7.5.0 (Ubuntu 7.5.0-3ubuntu1~18.04)
- Windows实验环境：Win10 2004 64位，`cl` 19.26.28805 版

### 5.2 C语言运行库实现

在开始实现Mini CRT之前，首先要对它进行基本的规划。“麻雀虽小五脏俱全”，虽然Mini CRT很小，但它应该具备CRT的基本功能以及遵循几个基本设计原则，这些我们归结为如下几个方面:

- 首先Mini CRT应该以ANIS C的标准库为目标，尽量做到与其接口相一致。
- 具有自己的入口函数（`mini_crt_entry`）。
- 基本的进程相关操作（`exit`）。
- 支持堆操作（`malloc`、`free`）。
- 支持基本的文件操作（`fopen`、`fread`、`fwrite`、`fclose`、`fseek`）。
- 支持基本的字符串操作（`strcpy`、`strlen`、`strcmp`）。
- 支持格式化字符串和输出操作（`printf`、`sprintf`）。
- 支持`atexit()`函数。
- 最后，Mini CRT应该是跨平台的。我们计划让Mini CRT能够同时支持Windows和Linux两个操作系统。
- Mini CRT的实现应该尽量简单，以展示CRT的实现为目的，并不追求功能和性能，基本上是“点到为止”

为了使CRT能够同时支持Linux和Windows两个平台，必须针对这两个操作系统环境的不同进行条件编译。在Mini CRT中，我们使用宏`WIN32`为标准来决定是Windows还是Linux。因为实际的代码常呈现这样的结构:

```c
#ifdef WIN32
//Windows部分代码
#else
//Linux部分实现代码
#endif
```

通常我们会把CRT的各个函数的声明放在不同的头文件中，比如IO相关的位于`stdio.h`；字符串和堆相关的放在`stdlib.h`中。为了简单起见，将Mini CRT中所有函数的声明都放在`minicrt.h`中。

#### 5.2.1 入口函数

##### 5.2.1.1 入口函数实现流程图

![](https://bbs.kanxue.com/upload/attach/202012/825187_48TV96Y69GD5W25.jpg)

##### 5.2.1.2 命令行解析

因为我们的`main`函数需要两个参数：参数个数和参数字符串指针数组，所以我们得进行命令行解析

###### Windows实现部分

Windows用`GetCommandLineA`获取输入字符串，然后手动提取参数，不过该提取算法有缺陷，如果有多余的空格，会得到`"\0"`的字符串

实现代码如下：

```c
int flag = 0;
int argc = 0;
char* argv[16];
char* cl = GetCommandLineA();
//解析命令行
//算法缺陷：有多余的空格，会得到"\0"的字符串
argv[0] = cl;
argc++;
while (*cl)
{
    if (*cl == '\"')
    {
        if (flag == 0)
        {
            flag = 1;
        }
        else
        {
            flag = 0;
        }
    }
    else if (*cl == ' ' && flag == 0)
    {
        if (*(cl+1))
        {
            argv[argc] = cl + 1;
            argc++;
        }
        *cl = '\0';
    }
    cl++;
}
```

###### Linux实现部分

当进入入口函数的时候，堆栈依次保存着参数个数、参数字符串指针、环境变量字符串指针和其它信息（**注意保存没有返回地址，因为这是入口函数**），此时`ESP`指向`argc`，堆栈信息如下：

![](https://bbs.kanxue.com/upload/attach/202012/825187_WMTEGGEV2WN8MKP.jpg)

然而因为我们的入口函数不是**裸函数**，所以会在函数开头生成如下指令

```asm
...
push ebp
mov ebp,esp
...
```

所以执行完上述指令后，此时`EBP`指向旧的`EBP`，新的堆栈信息如下：

![](https://bbs.kanxue.com/upload/attach/202012/825187_X4NYTPSR3FS7DE3.jpg)

所以此时可以通过`EBP`加上偏移的方式获取我们所需的参数

实现代码如下：

```c
int argc = 0;
char** argv = 0;
char* ebp_reg = 0;
// ebp_reg = %ebp;
asm("movl %%ebp,%0      \n"
    :"=r"(ebp_reg));
argc = *(int*)(ebp_reg + 4);
argv = (char**)(ebp_reg + 8);
```

##### 5.2.1.3 堆初始化

参考本文“5.2.2 堆的实现”中的`mini_crt_heap_init`函数

```c
if (mini_crt_heap_init() == -1)
{
    crt_fatal_error("heap initialize failed");
}
```

##### 5.2.1.4 IO初始化

参考本文“5.2.3 IO与文件操作”中的`mini_crt_io_init`函数

```c
if (mini_crt_io_init() == -1)
{
    crt_fatal_error("IO initialize failed");
}
```

##### 5.2.1.5 调用初始化函数

调用需要在`main`函数之前执行的函数，如全局对象的构造函数，在C++运行库部分进行实现

```c
//do_global_ctors();
```

##### 5.2.1.6 调用`main`

也就是调用我们程序的入口函数

```c
ret = main(argc,argv);
```

##### 5.2.1.7 退出
