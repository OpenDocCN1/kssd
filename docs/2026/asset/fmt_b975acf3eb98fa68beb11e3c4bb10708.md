
## ELF文件头信息

文件头包含魔数，还包含其他关键信息。

文件头的`size`是多少呢？其实armv8文件头是固定的长度，armv7文件头也是固定的长度。长度只与当前so库对应的平台相关。

比如apk中`libs/armeabi-v7a/`下面的所有so库都拥有一样长度的文件头。

使用ida工具打开一个arm64的so库，我们看看工具所解析的elf文件头包含的信息:

![](https://bbs.kanxue.com/upload/attach/202105/855618_M4568B5H2NBJ5PA.png)

结合该图，再来看看`linker`的源码，增加大家对ELF文件头的印象

源码位置`aosp7.1 bionic/linker/linker_phdr.cpp`

### 信息校验

```
//通过pread64将文件头的内容读到内存header_中
bool ElfReader::ReadElfHeader() {
    ssize_t rc = TEMP_FAILURE_RETRY(pread64(fd_, &header_, sizeof(header_), file_offset_));
    if (rc < 0) {
        DL_ERR("can't read file \"%s\": %s", name_.c_str(), strerror(errno));
        return false;
    }

    if (rc != sizeof(header_)) {
        DL_ERR("\"%s\" is too small to be an ELF executable: only found %zd bytes", name_.c_str(),
            static_cast<size_t>(rc));
        return false;
    }
    return true;
}
//校验elf文件头
bool ElfReader::VerifyElfHeader() {
    //1.校验魔数信息
    if (memcmp(header_.e_ident, ELFMAG, SELFMAG) != 0) {
        DL_ERR("\"%s\" has bad ELF magic", name_.c_str());
        return false;
    }

    //2.校验文件类别，64位运行时只能加载64位so库，32位运行时只能加载32位so库
    // Try to give a clear diagnostic for ELF class mismatches, since they're
    // an easy mistake to make during the 32-bit/64-bit transition period.
    int elf_class = header_.e_ident[EI_CLASS];
#if defined(__LP64__)
    if (elf_class != ELFCLASS64) {
        if (elf_class == ELFCLASS32) {
            DL_ERR("\"%s\" is 32-bit instead of 64-bit", name_.c_str());
        } else {
            DL_ERR("\"%s\" has unknown ELF class: %d", name_.c_str(), elf_class);
        }
        return false;
    }
#else
    if (elf_class != ELFCLASS32) {
        if (elf_class == ELFCLASS64) {
            DL_ERR("\"%s\" is 64-bit instead of 32-bit", name_.c_str());
        } else {
            DL_ERR("\"%s\" has unknown ELF class: %d", name_.c_str(), elf_class);
        }
        return false;
    }
#endif

    //3.检验大小端对齐方式，目前安卓上为小端对齐
    if (header_.e_ident[EI_DATA] != ELFDATA2LSB) {
        DL_ERR("\"%s\" not little-endian: %d", name_.c_str(), header_.e_ident[EI_DATA]);
        return false;
    }

    //4.校验文件类型是否为动态库
    if (header_.e_type != ET_DYN) {
        DL_ERR("\"%s\" has unexpected e_type: %d", name_.c_str(), header_.e_type);
        return false;
    }

    //5.校验版本号，安卓上目前只能是EV_CURRENT
    if (header_.e_version != EV_CURRENT) {
        DL_ERR("\"%s\" has unexpected e_version: %d", name_.c_str(), header_.e_version);
        return false;
    }

    //6.校验machine
    if (header_.e_machine != GetTargetElfMachine()) {
        DL_ERR("\"%s\" has unexpected e_machine: %d", name_.c_str(), header_.e_machine);
        return false;
    }

    return true;
}
```

### 获取程序头表信息

```
bool ElfReader::ReadProgramHeaders() {
    phdr_num_ = header_.e_phnum;

    // Like the kernel, we only accept program header tables that
    // are smaller than 64KiB.
    if (phdr_num_ < 1 || phdr_num_ > 65536/sizeof(ElfW(Phdr))) {
        DL_ERR("\"%s\" has invalid e_phnum: %zd", name_.c_str(), phdr_num_);
        return false;
    }

    // Boundary checks
    size_t size = phdr_num_ * sizeof(ElfW(Phdr));
    if (!CheckFileRange(header_.e_phoff, size, alignof(ElfW(Phdr)))) {
        DL_ERR_AND_LOG("\"%s\" has invalid phdr offset/size: %zu/%zu",
            name_.c_str(),
            static_cast<size_t>(header_.e_phoff),
            size);
        return false;
    }

    //将elf文件的程序头表内容映射到内存中,程序头表的文件偏移为header_.e_phoff
    if (!phdr_fragment_.Map(fd_, file_offset_, header_.e_phoff, size)) {
        DL_ERR("\"%s\" phdr mmap failed: %s", name_.c_str(), strerror(errno));
        return false;
    }

    phdr_table_ = static_cast<ElfW(Phdr)*>(phdr_fragment_.data());
    return true;
}
```

### 获取节头表信息

```
bool ElfReader::ReadSectionHeaders() {
    shdr_num_ = header_.e_shnum;

    if (shdr_num_ == 0) {
        DL_ERR_AND_LOG("\"%s\" has no section headers", name_.c_str());
        return false;
    }

    size_t size = shdr_num_ * sizeof(ElfW(Shdr));
    if (!CheckFileRange(header_.e_shoff, size, alignof(const ElfW(Shdr)))) {
        DL_ERR_AND_LOG("\"%s\" has invalid shdr offset/size: %zu/%zu",
            name_.c_str(),
            static_cast<size_t>(header_.e_shoff),
            size);
        return false;
    }

    //将elf文件的节头表内容映射到内存中，节头表的文件偏移为header_.e_shoff
    if (!shdr_fragment_.Map(fd_, file_offset_, header_.e_shoff, size)) {
        DL_ERR("\"%s\" shdr mmap failed: %s", name_.c_str(), strerror(errno));
        return false;
    }

    shdr_table_ = static_cast<const ElfW(Shdr)*>(shdr_fragment_.data());
    return true;
}
```

这两个表大家要牢记，elf文件头包含了这两个表的信息。

### 重要的事情说三遍：

#### ELF文件头包含了PHT,SHT这两个表的信息

#### ELF文件头包含了PHT,SHT这两个表的信息

#### ELF文件头包含了PHT,SHT这两个表的信息

![图片描述](https://bbs.kanxue.com/upload/attach/202105/855618_PUBGXDZT5C8YUNE.png)

## 通过程序头表拿到动态节区

先啰嗦一下*表*这个关键字：elf文件中，一个区域存储了很多个一摸一样的结构体，这个区域就可以视作表。比如got表，符号表，字符串表等等。这些表后边也会提到。

程序头表上面已经从ELF文件头信息中拿到了。通过循环遍历这个表，可以找到一个`PT_DYNAMIC`类型的程序头，这个程序头中存储了动态表的偏移大小等信息。我把demo中的so库用010editor这个工具解析出来贴了个图

![](https://bbs.kanxue.com/upload/attach/202105/855618_MAYN8NJM4QQ3SZJ.png)

大家可能有疑问，程序头表中有很多个程序头，为什么仅仅要拿到`PT_DYNAMIC`这个程序头，拿到动态表呢？

其实程序头表中的信息都很重要，比如图中的前两个程序头(loadable segment)，这两个程序头包含了一些装载信息。

`linker`会用到装载信息。比如so库装载到内存中需要多大空间，内存中的属性(可读可写可执行等)。

既然提到了装载，那就简单说一下装载和链接吧。

### 装载：

so库包含了程序运行需要用的数据以及指令，这些数据以及指令需要加载到内存中，加载到内存的过程就可以简单理解为装载。

### 链接：

so库一般会使用到外部的一些符号，比如`open`、`close`、`mmap`、`strlen`等导入符号，这些符号的真实地址不在本so库，可能在系统的`libc`库等等。那么本so库要调用`open`这些函数的话，肯定需要一些特殊的方式拿到其真实地址。这些真实地址其实保存在so库的got表中。`linker`将so库装载后, 会将这些符号进行重新定位，重新定位简称为重定位。其实重定位不仅仅包含外部的一些符号，还包含本so库的一些符号。重定位修复的整个过程可以理解为链接。

上边提到了前两个程序头(`loadable_segment`)与装载有关系。那么链接过程需要用的重定位信息在哪里呢？寻找这个答案，就需要`PT_DYNAMIC`这个程序头, 程序头有一个字端为`p_vaddr`，值为`0x2D28`，这个地址就是**动态节区，因为这个区域又是数组，后边就把这个节区称为动态表**。

## dynamic表的内容(用IDA工具看看0x2D28这个地方)

![](https://bbs.kanxue.com/upload/attach/202105/855618_2Q72YRG2T5FPGAF.png)

动态表中的`DT_JMPREL`这个结构体，包含了重定位表的信息，继续用ida工具查看`0x9f8`->重定位表

## 重定位表(地址0x9f8)

![](https://bbs.kanxue.com/upload/attach/202105/855618_RH5ZD7XE7J7EYDT.png)

重定位表包含了需要重定位的函数信息，包括`__open_2`函数。

前边提到了`linker`对so库链接的过程会进行符号重定位，重定位的信息就在重定位表中。

继续以`__open_2`这个重定位信息为例

`Elf64_Rela<0x2F90, 0xc00000402, 0>`，IDA已经解析出来了，这个结构体中`0x2f90`是个偏移，指向的地址就是got表, `0xc00000402`这个数据其实是一个info，会解析出符号索引，通过这个索引拿到符号名(`__open_2`)，也可以拿到这个重定位信息类型。

```
ElfW(Word) type = ELFW(R_TYPE)(rel->r_info);//符号的类型
ElfW(Word) sym = ELFW(R_SYM)(rel->r_info);//符号的索引
```

`ELFE(R_TYPE)`与`ELFW(R_SYM)`宏定义在`libc/kernel/uapi/linux/elf.h`中

```
#if defined(__LP64__)
#define ELFW(what) ELF64_ ## what
#else
#define ELFW(what) ELF32_ ## what
#endif

#define ELF32_R_SYM(x) ((x) >> 8)
#define ELF32_R_TYPE(x) ((x) & 0xff)
#define ELF64_R_SYM(i) ((i) >> 32)
#define ELF64_R_TYPE(i) ((i) & 0xffffffff)
```

刚才提到的`0x2f90`这个偏移也贴个图吧

![](https://bbs.kanxue.com/upload/attach/202105/855618_BNAG89AZH7FEKG5.png)

`linker`重定位修复的数据就包括got表，比如`0x2f90`这个地方，使用8个字节存放了`__open_2`函数的真实地址(64位系统需要8个字节存放一个地址，这个地址的数据类型简单理解为`void*`)。

关键的地方就在这里，既然`linker`通过重定位，将`0x2f90`这个地方的数据改了。那我们也仿照`linker`搞一个重定位的修复，将这个地方的数据改为我们的函数地址，是不是就可以实现hook了。答案是yes。

### 重要的事情说三遍

#### 仿照`linker`源码搞一个重定位修复，实现hook

#### 仿照`linker`源码搞一个重定位修复，实现hook

#### 仿照`linker`源码搞一个重定位修复，实现hook

### aosp中`linker`重定位的源码(`bionic/linker/linker.cpp`)
