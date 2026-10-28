
```cpp
#include "start.h"
#include <ntstrsafe.h>

VOID DriverUnload(PDRIVER_OBJECT pDriverObj)
{
    KdPrint(("Unload Driver\n"));
}

extern "C" long long myAdd(long long a, long long b);

extern "C" NTSTATUS DriverEntry(
    IN PDRIVER_OBJECT pRootDriverObj,
    IN PUNICODE_STRING pRegPath)
{
    NTSTATUS status = STATUS_UNSUCCESSFUL;
    pRootDriverObj->DriverUnload = DriverUnload;
    ULONG majIndex = 0;
    KdBreakPoint();
    myAdd(1, 2);
    return status;
}
```

目录结构为:

```
RootDir
amd64(DIR)
myadd.asm
start.cpp (驱动入口代码)
```

### 2.4 多驱动编译

如果驱动项目较多，想一下全部进行编译，那么就需要使用`DIRS`字段。

编译方法如下：

首先建立一个`DIRS`文件，文件的内容指明你想编译的驱动的文件目录即可。

但是你的目录里面要指明`SOURCES`文件，亦或者是新的`DIRS`。

```
DIRS = \
A \
B \
C
```

上述意思代表编译`A`、`B`、`C`三个文件目录下的驱动。

如果`A`目录下有`SOURCES`，则会读取`SOURCES`文件进行编译`A`目录；如果`B`目录又有内嵌的文件夹且有`DIRS`，那么会优先读取`DIRS`继续寻找`B`目录中的内嵌文件夹，直到找到有`SOURCES`存在的目录进行编译。

### 2.5 编译等级设置

如果你想让你的驱动编译的时候检测严格一点，则可以在`SOURCES`中定义如下字段：

```
MSC_WARNING_LEVEL = /W3 /WX
```

`/W3`是警告级别，`/W1`、`/W2`、`/W3`、`/W4`中`/W4`等级最为严格。如果参数不使用，则需要使用`UNREFERENCED_PARAMETER(pDriverObj);`来进行包含，否则在`/W4`登记下无法编译通过。

`/WX`是警告视为错误。

### 2.6 将驱动编译为库

驱动代码也可以变成库代码，可以给别的驱动使用。在高版本中的VS则直接生成即可，`wdk7600`则必须我们使用`SOURCES`指定了。

分为以下几点讲解：

我们可以将我们的驱动编译为库，这里涉及到库开发。分别是：

1. 驱动中如何生成库
2. 驱动中如何使用自定义的库

#### 2.6.1 驱动中如何生成库

首先如果你是以C/C++开发的话，那么就要给一个`.h`和一个`.cpp`文件。

假设以`test.h`、`test.cpp`为例，那么驱动的`SOURCES`文件内容应该如下：

```
TARGETNAME=test
TARGETPATH=.
TARGETTYPE=LIBRARY
DRIVERTYPE=FS

MSC_WARNING_LEVEL=/W3 /WX

INCLUDES=\
./test

SOURCES=test.cpp
```

其中我的目录结构为：

```
ROOTDIR
test(DIR)
test.h
test.cpp
```

`test.h`如下：

```cpp
#ifdef __cplusplus
extern "C" {
#endif

#include <ntddk.h>

#ifdef __cplusplus
}
#endif

class test
{
private:
    /* data */
public:
    test(/* args */);
    ~test();
    PVOID testprint();
};
```

`test.cpp`如下：

```cpp
#include "test.h"

test::test(/* args */)
{
}

test::~test()
{
}

PVOID test::testprint()
{
    DbgPrint("testprint");
    return NULL;
}
```

生成后则会生成`test.lib`库。

#### 2.6.2 驱动中使用库

使用库就很简单了，将头文件拷贝过来，然后在`SOURCE`里面指明即可。

`SOURCE`如下：

```
TARGETNAME=test1
TARGETPATH=.
TARGETTYPE=DRIVER

MSC_WARNING_LEVEL=/W3 /WX

INCLUDES=\
./test

TARGETLIBS=.\libs\test.lib
SOURCES=Driver.cpp
```

`test.h`同上一样。

`Driver.cpp`如下：

```cpp
#include "test\test.h"

VOID DriverUnLoad(
    PDRIVER_OBJECT DriverObject)
{
    KdPrint(("Exit"));
}

extern "C" NTSTATUS DriverEntry(
    PDRIVER_OBJECT pDriverObj,
    PUNICODE_STRING pReg)
{
    UNREFERENCED_PARAMETER(pDriverObj);
    UNREFERENCED_PARAMETER(pReg);
    KdBreakPoint();
    pDriverObj->DriverUnload = DriverUnLoad;
    test t;
    t.testprint();
    return STATUS_SUCCESS;
}
```

如果是C语言则直接编译即可。

关于`TARGETLIBS`还可以包含路径，例如：

```
TARGETLIBS = $(DDK_LIB_PATH)\xxx1.lib\
             $(DDK_LIB_PATH)\xxx2.lib\
```

例如包含`ntstrsafe.lib`库：

```
TARGETLIBS=$(DDK_LIB_PATH)\ntstrsafe.lib
```

系统提供的路径有如下：

```
DDK_LIB_PATH = WDKROOT\lib\Version\*
SDK_LIB_PATH = WDKROOT\lib\Version\*
CRT_LIB_PATH
```

### 2.7 C常量定义

在`SOURCES`文件中可以使用`C_DEFINES`，它的意思则是等价于你在`.c`文件中使用了`#define`来声明宏。

```
TARGETNAME=test1
TARGETPATH=.
TARGETTYPE=DRIVER

MSC_WARNING_LEVEL=/W3 /WX

!IFDEF DDKBUILDENV
C_DEFINES=$(C_DEFINES) -DDDK_BUILD
!ENDIF

INCLUDES=\
./test

TARGETLIBS=.\libs\test.lib
SOURCES=Driver.cpp
```

例子2：

```
C_DEFINES=$(C_DEFINES) /wd4996
```

### 2.8 `SOURCES`指明编译的文件 以及条件宏

在WDK中找的。可以为驱动编译资源，可以定义两个`SOURCES`分别指向要编译的文件，然后最终引用。

例子：

```
!if $(IA64)
xxxxx 条件使用 IA64
!endif

DIR_SOURCES=wacompen.c  \
            wacompen.rc \
            oempen.c    \
            errcodes.mc

STB_SOURCES=hid.c    \
            pnp.c       \
            serial.c    \
            errlog.c

SOURCES=$(DIR_SOURCES) $(STB_SOURCES)
```

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于 2023-6-6 14:14 被TkBinary编辑，原因：加图,和修改代码.

[#基础知识](https://bbs.kanxue.com/forum-41-1-130.htm)  
[#系统内核](https://bbs.kanxue.com/forum-41-1-131.htm)  
[#驱动开发](https://bbs.kanxue.com/forum-41-1-132.htm)
