
```
// minicrt.h
#ifndef __MINI_CRT_H__
#define __MINI_CRT_H__

#ifdef __cplusplus
extern "C" {
#endif

// malloc
#ifndef NULL
#define NULL (0)
#endif
#ifndef _SIZE_T_DEFINED
typedef unsigned int        size_t;
#define _SIZE_T_DEFINED
#endif  /* _SIZE_T_DEFINED */

void free(void* ptr);
void* malloc(unsigned size);
// static int brk(void* end_data_segment);
static void* mmap2(void *addr, unsigned len, int prot, int flags, int fd, int offset);
int mini_crt_heap_init();

// 字符串
char* itoa(int n, char* str, int radix);
int strcmp(const char* src, const char* dst);
char* strcpy(char* dest, const char* src);
unsigned strlen(const char* str);

// 文件与IO
typedef int FILE;
#define EOF (-1)

#ifdef WIN32
#define stdin   ((FILE*)(GetStdHandle(STD_INPUT_HANDLE)))
#define stdout  ((FILE*)(GetStdHandle(STD_OUTPUT_HANDLE)))
#define stderr  ((FILE*)(GetStdHandle(STD_ERROR_HANDLE)))
#else
#define stdin   ((FILE*)0)
#define stdout  ((FILE*)1)
#define stderr  ((FILE*)2)
#endif

int mini_crt_io_init();
FILE* fopen(const char* filename, const char* mode);
int fread(void* buffer, int size, int count, FILE* stream);
int fwrite(const void* buffer, int size, int count, FILE* stream);
int fclose(FILE* fp);
int fseek(FILE* fp, int offset, int set);

// printf
int fputc(int c, FILE* stream);
int fputs(const char* str, FILE* stream);
int printf(const char* format, ...);
int fprintf(FILE* stream, const char* format, ...);

// internal
void do_global_ctors();
void mini_crt_call_exit_routine();

// atexit

typedef void (*atexit_func_t) (void);
int atexit(atexit_func_t);
// 下面两个是C++库用到的
typedef void(*cxa_func_t)(void*);
int __cxa_atexit(cxa_func_t func, void* arg, void*);

#ifdef __cplusplus
}
#endif

#endif  //__MINI_CRT_H__
```

接下来的问题是如何编译得到库文件了。由于动态库的实现比静态库要复杂，所以 Mini CRT 仅仅以静态库的形式提供给最终用户，在 Windows 下它是 `minicrt.lib`；在 Linux 下它是 `minicrt.a`。在不同平台下编译和制作库文件的步骤如下所示，**Linux** 下的命令行为：

```
gcc -c -fno-builtin -nostdlib -fno-stack-protector -m32 entry.c malloc.c stdio.c string.c printf.c
ar -rs minicrt.a malloc.o printf.o stdio.o string.o
```

- 这里的 `-fno-builtin` 参数是指关闭 GCC 的内置函数功能，默认情况下 GCC 会把 `strlen`、`strcmp` 等这些常用函数展开成它内部的实现。

- `-nostdlib` 表示不使用任何来自 glibc、GCC 的库文件和启动文件，它包含了 `-nostartfiles` 这个参数。

- `-fno-stack-protector` 是指关闭堆栈保护功能，最近版本的 GCC 会在 `vfprintf` 这样的变长参数函数中插入堆栈保护函数，如果不关闭，我们在使用 Mini CRT 时会发生 `stack chk fail` 函数未定义的错误。

- `-m32` 是指生成32位的中间目标文件，因为我的实验环境是64位的，而实现的运行库是32位的

在**Windows** 下，Mini CRT 的编译方法如下：

```
cl /c /DWIN32 /utf-8 /GS- entry.c malloc.c printf.c stdio.c string.c
lib entry.obj malloc.obj printf.obj stdio.obj string.obj /OUT:minicrt.lib
```

- `/DWIN32` 表示定义 `WIN32` 这个宏，这也正是在代码中用于区分平台的宏。

- `/GS-` 表示关闭堆栈保护功能，MSVC 和 GCC 一样也会在不定参数中插入堆栈保护功能。如果不关闭，在最后链接时会发生 `__security_cookie` 和 `__security_check_cookie` 符号未定义错误。

- 如果出现了编码问题，则可以使用 `/utf-8` 来解决

为了测试 Mini CRT 是否能够正常运行，我们专门编写了一段测试代码，用于测试 Mini CRT 的功能：

```
// test.c
#include "minicrt.h"

int main(int argc, char* argv[]) {
    int i;
    FILE* fp;
    // 1.拷贝参数到堆
    char** v = malloc(argc * sizeof(char*));
    for (i = 0; i < argc; i++) {
        v[i] = malloc(strlen(argv[i]) + 1);
        strcpy(v[i], argv[i]);
    }
    // 2.写入文件
    fp = fopen("test.txt", "w");
    for (i = 0; i < argc; i++) {
        int len = strlen(v[i]);
        fwrite(&len, sizeof(int), 1, fp);
        fwrite(v[i], strlen(v[i]) + 1, 1, fp);
    }
    fclose(fp);
    // 3.读取文件
    fp = fopen("test.txt", "r");
    for (i = 0; i < argc; i++) {
        int len;
        char* buf;
        fread(&len, sizeof(int), 1, fp);
        buf = malloc(len + 1);
        fread(buf, len + 1, 1, fp);
        printf("%d %s\n", len, buf);
        free(buf);
        free(v[i]);
    }
    fclose(fp);
    return 0;
}
```

这段代码用到了 Mini CRT 中绝大部分函数，包括 `malloc`、`free`、`fopen`、`fclose`、`fread`、`fwrite`、`printf`，并且测试了 `main` 参数。它的作用就是将 `main` 的参数字符串都保存到文件中，然后再读取出来，由 `printf` 显示出来。在**Linux** 下，可以用下面的方法编译和运行 `test.c`：

```
gcc -c -ggdb -fno-builtin -nostdlib -fno-stack-protector -m32 test.c
ld -static -m elf_i386 -e mini_crt_entry entry.o test.o minicrt.a -o test
```

![](https://bbs.kanxue.com/upload/attach/202012/825187_875NHBK2DVW7EPJ.jpg)

- `-e mini_crt_entry` 用于指定入口函数。

可以看到静态链接 Mini CRT 最后输出的可执行文件只有 **10964 字节**，这正体现出了 Mini CRT 的“迷你”之处，而如果静态链接 glibc 时，最后可执行文件则约为 **645KB**。在**Windows** 下，编译和运行 `test.c` 的步骤如下：

```
cl /c /DWIN32 /utf-8 test.c
link test.obj minicrt.lib kernel32.lib /NODEFAULTLIB /DEBUG /entry:mini_crt_entry
```

![](https://bbs.kanxue.com/upload/attach/202012/825187_FUGP63HBRBK8XDF.jpg)

与 Linux 类似，Windows 下使用 Mini CRT 链接的可执行文件也非常小，只有 **12800 字节**。如果我们使用 `dumpbin` 查看它的导入函数可以发现，它仅依赖于 `Kernel32.DLL` 中**我们使用到的函数**。

### 5.3 C++运行库实现

现在 Mini CRT 已经能够支持最基本的 C 语言程序运行了。C++ 作为兼容 C 语言的扩展语言，它的运行库的实现其实并不复杂，在这一节中将介绍如何为 Mini CRT 添加对 C++ 语言的一些常用的操作支持。

通常 C++ 的运行库都是独立于 C 语言运行库的，比如 Linux 下 C 语言运行库为 `libc.so/libc.a`，而 C++ 运行库为 `libstdc++.so/libstdc++.a`；Windows 的 C 语言运行库为 `libcmt.lib/msvcr90.dll`，而 C++ 运行库为 `libcpmt.lib/msvcp90.dll`。一般这些 C++ 的运行库都是依赖于 C 运行库的，它们仅包含对 C++ 的一些特性的支持，比如 `new`/`delete`、STL、异常处理、流（`stream`）等。但是它们并不包含诸如入口函数、堆管理、基本文件操作等这些特性，而这些也是 C++ 运行库所必需的，比如 C++ 的流和文件操作依赖于 C 运行库的基本文件操作，所以它必须依赖于 C 运行库。

本节中我们将在 Mini CRT 的基础上实现一个支持 C++ 的运行库，当然出于简单起见，将这个 C++ 运行库的实现与 Mini CRT 合并到一起，而不是单独成为一个库文件，也就是说经过这一节对 Mini CRT 的功能改进，最终编译出来的 `minicrt.a/minicrt.lib` 将支持 C++ 的诸多特性。

当然，要完整实现一个 C++ 的运行库是很费事的一件事，C++ 标准模板库 STL 包含了诸如流、容器、算法、字符串等，规模较为庞大。出于演示的目的，我们将对 C++ 的标准库进行简化，最终目标是实现一个能够成功运行如下 C++ 程序代码的运行库：

```
// test.cpp
#include "iostream"
#include "string"

using namespace std;

string abc("abcd");

class Person {
private:
    int age;
    char* name;
public:
    Person() {
        printf("Person\n");
    }
    ~Person() {
        printf("~Person\n");
    }
};

int main(int argc, char* argv[]) {
    Person* arr = new Person[2];
    string* msg = new string("Hello World");
    cout << *msg << endl;
    cout << abc << endl;
    delete msg;
    delete[] arr;
    return 0;
}
```

上面这段程序看似简单，实际上它用到了 C++ 运行库的诸多功能。我们将所用到的特性列举如下：

- `string` 类的实现。

- `stream` 类的实现，包括操纵符（`Manipulator`）（`endl`）。

- 全局对象构造和析构（`cout`、`abc`）。

- `new`/`delete`。

在开始本节之前，还是按照前面 Mini CRT 实现时的做法：在进入具体主题之前先列举一些实现的原则。在实现 Mini CRT 对 C++ 的支持时，我们遵循如下原则：

- HelloWorld 程序无须用到的功能就不实现，比如异常。

- 尽量简化设计，尽量符合 C++ 标准库的规范。

- 对于可以直接在头文件实现的模块尽量在头文件中实现，以免诸多的类、函数的声明和定义造成代码量膨胀，不便于演示。

- 与前面的 Mini CRT 实现一样，运行库代码要做到可以在 Windows 和 Linux 上同时运行，因此对于平台相关部分要使用条件编译分别实现。虽然 C++ 运行库几乎没有与系统相关的部分（全局构造和析构除外），C 运行库已经将大部分系统相关部分封装成 C 标准库接口，C++ 运行库只须要调用这些接口即可。

- 另外值得一提的是，模板是不需要运行库支持的，它的实现依赖于编译器和链接器，对运行库基本上没有要求。

#### 5.3.1 `new` 与 `delete`

##### 5.3.1.1 有了 `malloc`/`free` 为什么还要 `new`/`delete`
