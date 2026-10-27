
# [原创]Windows 下的 Makefile 编写（一）Makefile的基本规则

作者：cntrump

`Makefile`对于很多人来说是陌生的，特别是习惯于使用 IDE 的人来说，似乎没有听说过 `Makefile`，因为 `Makefile` 的工作都由 IDE 代劳了。但是 `Makefile` 的地位是不可忽略的，从 VC 诞生到现在 `Nmake` 这个实用程序就一直伴随着 VC 编译器一起发行。

很多大的工程都是基于 `Makefile` 编译和维护的，对于开源项目来说，大多数都使用 `Makefile` 进行编译，使用 IDE 来编译大型工程是不可想象的。

`Makefile`是什么？它是一个文本文件，里面记录着项目由哪些目标构成，以及各个目标的生成方式等信息，`Makefile`的核心任务是定义一系列的规则，然后由 `Nmake` 来解释执行，任何一个文本编辑器都可以用来编写 `Makefile`。

先来大概看一下 `Makefile` 的基本规则：

```
Target:dependent;command

Command

........
```

`Target` 是目标，目标可以是一个文件，也可以是一个标签，如果 `Target` 用作标签，则称之为伪目标。`Makefile` 至少要有一个目标。

`Dependent` 是依赖项目，指明目标所依赖的具体项目。依赖项目和目标之间用 `:` 号分隔。

`Command` 是命令，如果命令和依赖项目在同一行，则需要使用 `;` 号与之相隔，各个命令之间使用空格或 Tab 键分开，如果命令是单独一行，则需要使用 Tab 缩进。`Command` 命令由 `Nmake` 来执行。

上述内容简单地表明了一个依赖关系，生成 `Target` 目标依赖 `dependent` 中指定的文件，而生成的规则由 `Command` 来定义，`Nmake` 负责执行这些命令。

默认情况下，`Nmake` 会查找当前目录下任何名称为 `Makefile` 的文件（名称不区分大小写，并且没有后缀），如果你的 `Makefile` 文件名称是其他的，则需要使用 `-f` 参数指定。

以上就是 `Makefile` 的核心内容，任何系统的 `Makefile` 都是这样执行的。但是要写好一个 `Makefile`，仅仅这些还不够。

对于一个新知识，我更喜欢从做中学。下面举一个例子来说明上面的规则在实际应用中如何操作：

```
Test.exe:main.obj
    Link.exe main.obj /out:Test.exe

main.obj:main.cpp
#仅编译文件
    cl.exe main.cpp
```

在 `Makefile` 中注释使用 `#` 号开头，且仅有这一种注释方式。它的作用和 C++ 语法中的 `//` 注释是一样的。所不同的是 `#` 号必须放在行首。

上面的 `Makefile` 文件指定了两个目标，分别是 `Test.exe` 和 `main.obj`，生成 `Test.exe` 需要依赖 `main.obj` 文件，而生成 `main.obj` 文件依赖 `main.cpp`。在目标下方指明了生成该目标方法。

`main.cpp` 的内容如下：

```
#include <stdio.h>

int main()
{
  printf("Hello Makefile!\n");
  return 0;
}
```

将 `Makefile` 和 `main.cpp` 放置于同一目录下，在 VC 的命令提示符窗口中执行 `nmake` 命令，就会自动生成 `Test.exe` 和 `main.obj` 两个文件。再运行生成后的 `Test.exe` 测试一下：

![](https://bbs.kanxue.com/upload/attach/201012/48065_xiw8i2bimw4v8jj.jpg)

在 `Makefile` 中定义了两个目标，`Nmake` 默认只生成 `Makefile` 中的第一个目标，由于 `main.obj` 是 `Test.exe` 的依赖项，所以 `main.obj` 目标也得以执行。

是不是每执行一次 `nmake` 命令就会重新生成一次目标文件呢？答案：不是。每次都重新生成目标显然是一种资源浪费，`nmake` 是根据时间戳来决定是否需要重新生成目标。只有在依赖文件不存在或者依赖文件时间高于目标文件时，`nmake` 才会生成目标。

前面的例子已经生成了目标文件，如果修改了 `main.cpp` 中的代码或者删除了 `main.obj`，都将会重新生成 `Test.exe`。

一般来说，为了使清理中间文件或重新生成目标更加方便，都会在 `Makefile` 中加入一个伪目标来清理生成的中间文件。以删除 `main.obj` 为例，`Makefile` 修改如下：

```
Test.exe:main.obj
    Link.exe main.obj /out:Test.exe

Main.obj:main.cpp
# 仅编译文件
    cl.exe /c main.cpp

clean:
    @del *.obj
    @echo Project has clean.
```

`clean` 是一个伪目标，只是作为标签使用。`clean` 下的指令是命令行下的删除文件命令和显示一个字符串。在命令前加 `@` 符号是为了不显示命令本身。

默认情况下 `nmake` 只会生成第一个 `Test.exe` 目标，不会执行到 `clean` 目标，如果要指定生成目标，需要显式指定目标名称：

`nmake clean`

就指定要执行 `Makefile` 中的 `clean` 伪目标。执行之后会删除生成的 `main.obj` 并显示一行文字：

![](https://bbs.kanxue.com/upload/attach/201012/48065_ytjj8pwv34y67j4.jpg)

还可以指定多个目标，`nmake` 会从左往右依次生成目标。所以如果要在清除中间之后立即生成 `Test.exe`，可以这么做：

`nmake clean Test.exe`

![](https://bbs.kanxue.com/upload/attach/201012/48065_i8koczdz00f99g6.jpg)

一般来说，总是把最终生成的目标放在最前，而把清理中间文件的伪目标放到最后。下一回，我会结合实际的例子再介绍 `Makefile` 的其他内容。

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- [image002.jpg](javascript:void(0);)

  （3.53kb，676次下载）

- [image004.jpg](javascript:void(0);)

  （10.28kb，675次下载）

- [image006.jpg](javascript:void(0);)

  （22.01kb，676次下载）

- [Windows 下的 Makefile 编写（一）Makefile的基本规则.pdf](javascript:void(0);)

  （310.48kb，188次下载）
