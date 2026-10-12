
`fromDir` 是依赖文件所在目录，`toDir` 是依赖文件所在目录。目录可以使用宏来表示。大括号是必须的，且每个扩展名只能指定一个目录路径，`{}`（空的大括号）表示当前目录。

例如，`.obj` 文件被编译输出到 `Debug` 目录，源文件在当前目录下：

代码:

```
outdir=.\\Debug

objects=$(outdir)\\main.obj \

$(outdir)\\func.obj

All:$(objects)

link $\*\*

.cpp{$(outdir)}.obj::

cl /Fo"$(outdir)/" /c $<
```

在执行命令之前 `Debug` 目录必须存在，否则编译命令将失败。命令成功执行后，会在 `Debug` 目录下生成 `.obj` 文件，最终的可执行文件生成在当前目录下。

为了能更好的理解上面的推理规则，可以将上面的宏进行展开：

代码:

```
All:.\\Debug\\main.obj .\\Debug\\func.obj

link $\*\*

.cpp{.\\Debug}.obj::

cl /Fo".\\Debug/" /c $<
```

最后需要说明的是，在批模式下，必须使用 `$<` 宏来指定依赖文件项目。

如果你理解了宏，预处理和推理规则，那么已经可以阅读大多数 Windows 下开源项目的 `Makefile` 文件了，并且已经可以自己写出实用的 `Makefile`。

剩下的就是多加练习，试着为 IDE 向导生成的项目编写 `Makefile`，多尝试几次之后你会发现原来 `Makefile` 很简单。

上传的附件

| ![](img/668feaf58c8e7ddb6fbdbf839a5a19db.jpg) || Windows 下的 Makefile 编写（三）推理规则.pdf

|
