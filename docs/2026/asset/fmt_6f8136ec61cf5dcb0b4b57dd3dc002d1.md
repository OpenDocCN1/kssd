
`do_files()` 函数的功能是处理命令行传入的多个文件，函数声明在`work.cpp`中，相对应`work.cpp`中有处理单一文件的函数`do_one_file()`，多文件时使用`do_files()`先初步处理，然后再让`do_one_file()`挨个根据`opt->cmd` 执行不同的操作：

- 如果 `opt->cmd` 是 `CMD_COMPRESS` ，则对每个文件调用 `do_one_file()` 压缩
- 如果 `opt->cmd` 是 `CMD_DECOMPRESS` ，则对每个文件调用 `do_one_file()` 解压缩
- 在处理每个文件时，能够处理异常，返回相应的错误码。
- 在处理完成所有文件后，调用 `UiPacker` 的相应函数，提供总结信息（`UiPacker`负责UPX中和用户交互相关的所有功能，包括进度显示、交互事件、统计信息收集等）
- 返回 `0` 表示成功处理所有文件，返回 `-1` 表示有一个文件处理时出现致命错误。

### `do_one_file()` 函数的具体操作流程：

```
void do_one_file(const char *iname, char *oname) {
int r;
struct stat st;
mem_clear(&st);
#if HAVE_LSTAT
r = lstat(iname, &st);
#else
r = stat(iname, &st);
...
}
```

- 打开输入文件(`iname`)并获取文件信息,如大小、权限等
- 检查文件是否为普通文件，并且大小大于零
- 打开输出文件(`oname`)，可以是一个文件或者标准输出
- 根据 `opt->cmd` 的值,调用`PackMaster`执行不同的操作：
  - 如果 `opt->cmd` 是 `CMD_COMPRESS`,则调用 `pm.pack()` 对文件进行压缩
  - 如果`opt->cmd` 是 `CMD_DECOMPRESS`,则调用 `pm.unpack()` 对文件进行解压缩
- 如果需要，则复制输入文件的时间戳等属性到输出文件，然后关闭输入输出文件
- 如果生成了临时输出文件，则保留输出文件并删除输入文件(即完成重命名)，或者生成备份并删除输入文件
- 处理异常,抛出相应的异常或错误

此处`pm`是`PackMaster`的对象：

```
// handle command - actual work is here
PackMaster pm(&fi, opt);
if (opt->cmd == CMD_COMPRESS)
pm.pack(&fo);
else if (opt->cmd == CMD_DECOMPRESS)
pm.unpack(&fo);
else if (opt->cmd == CMD_TEST)
pm.test();
else if (opt->cmd == CMD_LIST)
pm.list();
else if (opt->cmd == CMD_FILEINFO)
pm.fileInfo();
else
throwInternalError("invalid command");
```

`PackMaster` 类的定义在 `src/packmast.h` 中，注释写得十分清楚："dispatch to a concrete subclass of class Packer; see work.cpp"，这个部分后面会讲。

```
#pragma once
class Packer;
class InputFile;
class OutputFile

/*************************************************************************
// dispatch to a concrete subclass of class Packer; see work.cpp
**************************************************************************/

class PackMaster final {
public:
explicit PackMaster(InputFile *f, Options *o = nullptr) noexcept;
~PackMaster() noexcept;

void pack(OutputFile *fo);
void unpack(OutputFile *fo);
void test();
void list();
void fileInfo();

typedef Packer *(*visit_func_t)(Packer *p, void *user);
static Packer *visitAllPackers(visit_func_t, InputFile *f, const Options *, void *user);

private:
OwningPointer(Packer) packer = nullptr; // owner
InputFile *fi = nullptr; // reference

static Packer *getPacker(InputFile *f);
static Packer *getUnpacker(InputFile *f);

// setup local options for each file
Options local_options;
Options *saved_opt = nullptr;
};

/* vim:set ts=4 sw=4 et: */
```

`PackMaster` 类的主要功能是：

- `PackMaster` 的构造函数会调用 `getPacker()` 函数,根据输入文件的类型分配对应的 `Packer` 子类。
- `pack()`、`unpack()` 等函数,实际上是通过 `packer` 属性调用对应的 `Packer` 子类的方法。
- `local_options` 是个 `Options` 结构体，它存储了独立于全局配置的选项。

`PackMaster` 可以在不修改 `Packer` 子类的情况下，支持不同的文件类型，这也是`packmast.cpp`的主要功能。虽然 `PackMaster` 使用了 `final` 关键字，让这个类不会再有子类，但是 `PackMaster` 类内部引用了 `Packer` 类，`Packer` 类的声明位于 `packer.h` 中，通过这个 `Packer` 子类对象再执行不同的操作。

## packmast.cpp

![图片描述](https://bbs.kanxue.com/upload/attach/202308/951654_WHJF6VEFWVVY9JY.png)

`packmast.cpp`在整个upx加壳流程中起到了分类的作用，其中`getPacker()`会根据文件格式选择合适的打包器类，比如说检测到PE文件则选择`pefile.cpp`。下面详细说明`packmast.cpp`实现的功能：

在`packmast.h`中声明了`PackMaster`类，后续`work.cpp`会用到这个类：

```
/*************************************************************************
// dispatch to a concrete subclass of class Packer; see work.cpp
**************************************************************************/

class PackMaster final {
public:
explicit PackMaster(InputFile *f, Options *o = nullptr) noexcept;
~PackMaster() noexcept;

void pack(OutputFile *fo);
void unpack(OutputFile *fo);
void test();
void list();
void fileInfo();

typedef Packer *(*visit_func_t)(Packer *p, void *user);
static Packer *visitAllPackers(visit_func_t, InputFile *f, const Options *, void *user);

private:
OwningPointer(Packer) packer = nullptr; // owner
InputFile *fi = nullptr; // reference

static Packer *getPacker(InputFile *f);
static Packer *getUnpacker(InputFile *f);

// setup local options for each file
Options local_options;
Options *saved_opt = nullptr;
};

/* vim:set ts=4 sw=4 et: */
```

在`packmast.cpp`中代码一开始使用构造函数`PackMaster::PackMaster(InputFile *f, Options *o) noexcept : fi(f)`来初始化`PackMaster`对象的状态。`PackMaster`类的实现提供了以下功能：

1. `PackMaster::PackMaster`：`PackMaster`类的构造器接受一个`InputFile`的指针和一个可选的`Options`指针。如果提供了`Options`，它会创建一个该选项的本地副本，并将全局的`opt`指针设置为指向这个本地副本。这样，`PackMaster`对象就可以使用自己的选项，而不影响全局的选项。析构器则在`PackMaster`对象被销毁时恢复全局的`opt`选项并删除`packer`对象。
2. `pack()`：用于执行压缩操作。它首先获取一个适合输入文件的`Packer`对象，然后调用该对象的`doPack()`方法来进行压缩。
3. `unpack()`：用于执行解压缩操作。它首先获取一个适合输入文件的`Packer`对象，然后调用该对象的`doUnpack()`方法来进行解压缩。
4. `test()`：用于测试已压缩的文件。它首先获取一个适合输入文件的`Packer`对象，然后调用该对象的`doTest()`方法来进行测试。
5. `list()`：用于列出已压缩文件的信息。它首先获取一个适合输入文件的`Packer`对象，然后调用该对象的`doList()`方法来进行列出。
6. `fileInfo()`：用于获取文件信息。它首先尝试获取一个适合解压缩输入文件的`Packer`对象，如果失败，再尝试获取一个适合压缩输入文件的`Packer`对象，然后调用该对象的`doFileInfo()`方法来获取文件信息。
7. `visitAllPackers()`：这个方法遍历所有可能的`Packer`类型，并对每个类型执行给定的函数。这个函数（`try_can_pack`或`try_can_unpack`）会检查该类型的`Packer`是否可以处理给定的输入文件。如果可以，那么该`Packer`对象就会被返回。
8. `getPacker()`和`getUnpacker()`：这两个方法都使用`visitAllPackers()`方法来找到一个适合处理输入文件的`Packer`对象。`getPacker()`找到的是可以压缩输入文件的`Packer`，而`getUnpacker()`找到的是可以解压缩输入文件的`Packer`。

## work.cpp

该源码从上到下包含的主要函数有三个：

1. `do_files`：这个函数处理从命令行传入的所有文件。它首先进行编译器的一致性检查，然后遍历所有输入的文件，对每个文件调用`do_one_file`进行处理，捕获并处理可能抛出的异常。在处理所有文件后，根据命令调用`UiPacker`的相应函数进行输出。
2. `do_one_file`：这个函数负责处理一个文件。主要步骤包括检查文件属性（是否是普通文件，文件大小是否合理，权限是否合理等），打开输入文件，打开或者创建输出文件，根据命令（压缩，解压缩，测试，列出，获取文件信息）调用`PackMaster`进行处理，复制时间戳，关闭文件，根据需要更改文件名或者删除文件，复制文件属性。
3. `unlink_ofile`：这个函数负责在发生异常时删除输出文件。`unlink_ofile` 在 `do_one_file` 和 `do_files` 内部使用，不导出。

这个地方的代码主要是根据`packmast.cpp`提供的`PackMaster`类构造`PackMaster`对象，例如下面的代码：

```
InputFile fi;
fi.st = st;
fi.sopen(iname, O_RDONLY | O_BINARY, SH_DENYWR);

...

// handle command - actual work is here
PackMaster pm(&fi, opt);
if (opt->cmd == CMD_COMPRESS)
pm.pack(&fo);
else if (opt->cmd == CMD_DECOMPRESS)
pm.unpack(&fo);
else if (opt->cmd == CMD_TEST)
pm.test();
else if (opt->cmd == CMD_LIST)
pm.list();
else if (opt->cmd == CMD_FILEINFO)
pm.fileInfo();
else
throwInternalError("invalid command");
```

这段代码首先创建了一个 `InputFile` 对象 `fi`，并打开了输入文件 `iname`，接着创建一个名为 `fo` 的 `OutputFile` 对象，然后使用这个输入文件 `fi` 和选项 `opt`，创建了一个 `PackMaster` 对象 `pm`。如果选项 `opt` 指定的命令是压缩 (`CMD_COMPRESS`)，那么就调用 `pm` 的 `pack` 方法，并传入输出文件对象 `fo`进行压缩操作。

`work.cpp`与`pefile.cpp`的关系是什么？

1. `work.cpp`中的`do_one_file()`会调用`PackMaster`进行单个文件的压缩处理。
2. `PackMaster`会根据文件类型创建合适的`Packer`子类实例,比如`PeFile`。
3. `PeFile`定义了`pack()`和`unpack()`方法实现具体的压缩和解压。
4. `work.cpp`最终通过`PackMaster`调用到`PeFile`的`pack()`方法实现对PE文件的压缩。
5. `PeFile::pack()`会调用各过程完成导入表、重定位等处理,并调用`compressWithFilters()`进行实际压缩。
6. 压缩结果会写入输出文件,完成整个压缩过程。
7. 对解压也是类似的过程,`work.cpp`通过`PackMaster`调用`PeFile::unpack()`。

**所以 `work.cpp` 控制总体流程，使用 `packmast.cpp` 来根据文件类型从 `packer.cpp` 中选择 `PeFile` 来实现特定格式PE文件的处理。**

## packer.cpp

`packer.cpp` 实现了 `Packer` 抽象类的具体函数，用来提供不同文件格式的打包和解包的基类。`Packer` 是 `PackMaster` 的子类，再作为 `pefile.cpp` 的基类，后续 `pefile.cpp` 会继承 `packer.cpp` 的一部分特性进行打包。

![图片描述](https://bbs.kanxue.com/upload/attach/202308/951654_PECXSXDC244SKGM.png)

`Packer` 抽象类提供了共享的函数，子类可以实现各自的压缩和解压算法。在源码里我们可以看到还有`packer_f.cpp`、`packer_c.cpp`、`packer_c.cpp`，这些文件提供了基于`packer.cpp`函数的共享函数，例如 `packer_c.cpp` 提供了的共享函数：
