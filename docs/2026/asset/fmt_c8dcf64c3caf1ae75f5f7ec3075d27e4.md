
> 此成员给出从文件头到该段第一个字节的偏移

- `p_vaddr`

> 此成员给出段的第一个字节将被放到内存中的虚拟地址

- `p_paddr`

> 此成员仅用于与物理地址相关的系统中。因为 System V 忽略所有应用程序的物理地址信息，此字段对与可执行文件和共享目标文件而言具体内容是未指定的。

- `p_filesz`

> 此成员给出段在文件映像中所占的字节数。可以为 0。

- `p_memsz`

> 此成员给出段在内存映像中占用的字节数。可以为 0。

- `p_flags`

> 此成员给出与段相关的标志。

- `p_align`

> 可加载的进程段的 `p_vaddr` 和 `p_offset` 取值必须合适，相对于对页面大小的取模而言。此成员给出段在文件中和内存中如何对齐。数值 0 和 1 表示不需要对齐。否则 `p_align` 应该是个正整数，并且是 2 的幂次数，`p_vaddr` 和 `p_offset` 对 `p_align` 取模后应该相等。

目录

- ELF 文件格式

- ELF Header

- [节区(Sections)](https://bbs.kanxue.com/#节区(sections))

- [程序头部](https://bbs.kanxue.com/#程序头部)

- [ELF表](https://bbs.kanxue.com/#elf表)

- [字符串表](https://bbs.kanxue.com/#字符串表)

- [符号表](https://bbs.kanxue.com/#符号表)

- [重定位信息](https://bbs.kanxue.com/#重定位信息)

- [so加载原理](https://bbs.kanxue.com/#so加载原理)

- [开始分析linker](https://bbs.kanxue.com/#开始分析linker)

- [find_library](https://bbs.kanxue.com/#find_library)

- [find_library_internal](https://bbs.kanxue.com/#find_library_internal)

- ElfReader::Load

- [链接so文件](https://bbs.kanxue.com/#链接so文件)

- [soinfo_link_image](https://bbs.kanxue.com/#soinfo_link_image)

- 重定位修复: `soinfo_relocate`

- 查找符号地址: `soinfo_do_lookup`

## ELF表

### 字符串表

字符串表节区包含以NULL结尾的字符序列.

一般，第一个字节(索引为 0)定义为一个空字符串。类似的，字符串表的最后一个字节也定义为 NULL，以确保所有的字符串都以 NULL 结尾。索引为 0 的字符串在不同的上下文中可以表示无名或者名字为 NULL 的字符串。

例如:

```
\0name\0chp\0seg\0\0xxx
```

里面包含3个字符串

1. name

2. chp

3. seg

### 符号表

目标文件的符号表中包含用来定位、重定位程序中符号定义和引用的信息。符号表索引是对此数组的索引。索引0表示表中的第一表项，同时也作为未定义符号的索引。

符号表定义如下:

```
typedef struct {
    Elf32_Word  st_name;
    Elf32_Addr  st_value;
    Elf32_Word  st_size;
    unsigned char   st_info;
    unsigned char   st_other;
    Elf32_Half  st_shndx;
} Elf32_sym;
```

| 名称 | 含义 |
| --- | --- |
| `st_name` | 包含目标文件符号字符串表的索引，其中包含符号名的字符串表示。如果该值非 0，则它表示了给出符号名的字符串表索引，否则符号表项没有名称。注:外部 C 符号在 C 语言和目标文件的符号表中具有相同的名称。 |
| `st_value` | 此成员给出相关联的符号的取值。依赖于具体的上下文，它可能是一个绝对值、一个地址等等。 |
| `st_size` | 很多符号具有相关的尺寸大小。例如一个数据对象的大小是对象中包含的字节数。如果符号没有大小或者大小未知，则此成员为 0。 |
| `st_info` | 此成员给出符号的类型和绑定属性。下面给出若干取值和含义的绑定关系。 |
| `st_other` | 该成员当前包含 0，其含义没有定义。 |
| `st_shndx` | 每个符号表项都以和其他节区间的关系的方式给出定义。此成员给出相关的节区头部表索引。某些索引具有特殊含义。 |

- `st_info`的说明

> `st_info` 包含符号类型和绑定信息.

```
#define ELF32_ST_BIND(i)    ((i)>>4)        //获取绑定信息
#define ELF32_ST_TYPE(i)    ((i)&0xf)       //获取符号类型
#define ELF32_ST_INFO(b, t) (((b)<<4) + ((t)&0xf))
```

- **绑定类型(ST_BIND)**

| 名称 | 取值 | 说明 |
| --- | --- | --- |
| `STB_LOCAL` | 0 | 局部符号在包含该符号定义的目标文件以外不可见。相同名称的局部符号可以存在于多个文件中，互不影响。 |
| `STB_GLOBAL` | 1 | 全局符号对所有将组合的目标文件都是可见的。一个文件中对某个全局符号的定义将满足另一个文件对相同全局符号的未定义引用。 |
| `STB_WEAK` | 2 | 弱符号与全局符号类似，不过他们的定义优先级比较低。 |

全局符号与弱符号之间的区别主要有两点:

(1). 当链接编辑器组合若干可重定位的目标文件时，不允许对同名的 `STB_GLOBAL` 符号给出多个定义。另一方面如果一个已定义的全局符号已经存在，出现一个同名的弱符号并不会产生错误。链接编辑器尽关心全局符号，忽略弱符号。类似地，如果一个公共符号(符号的 `st_shndx` 中包含 `SHN_COMMON`)，那么具有相同名称的弱符号出现也不会导致错误。链接编辑器会采纳公共定义，而忽略弱定义。

(2). 当链接编辑器搜索归档库(archive libraries)时，会提取那些包含未定义全局符号的档案成员。成员的定义可以是全局符号，也可以是弱符号。连接编辑器不会提取档案成员来满足未定义的弱符号。未能解析的弱符号取值为 0。

- **符号类型**

| 名称 | 取值 | 说明 |
| --- | --- | --- |
| `STT_NOTYPE` | 0 | 符号的类型没有指定 |
| `STT_OBJECT` | 1 | 符号与某个数据对象相关，比如一个变量、数组等等 |
| `STT_FUNC` | 2 | 符号与某个函数或者其他可执行代码相关 |
| `STT_SECTION` | 3 | 符号与某个节区相关。这种类型的符号表项主要用于重定位，通常具有 `STB_LOCAL` 绑定。 |
| `STT_FILE` | 4 | 传统上，符号的名称给出了与目标文件相关的源文件的名称。文件符号具有 `STB_LOCAL` 绑定，其节区索引是`SHN_ABS`，并且它优先于文件的其他 `STB_LOCAL` 符号 (如果有的话) |

### 重定位信息

重定位是将符号引用与符号定义进行连接的过程.例如,当程序调用了一个函数时,相关的调用指令必须将控制传输到适当的目标执行地址.

**重定位表项**

```
typedef struct {
    Elf32_Addr r_offset;
    Elf32_Word r_info;
} Elf32_Rel;
typedef struct {
    Elf32_Addr r_offset;
    Elf32_Word r_info;
    Elf32_Word r_addend;
} Elf32_Rela;
```

| 名称 | 说明 |
| --- | --- |
| `r_offset` | 此成员给出了重定位动作所适用的位置。对于一个可重定位文件而言，此值是从节区头部开始到将被重定位影响的存储单位之间的字节偏移。对于可执行文件或者共享目标文件而言，其取值是被重定位影响到的存储单元的虚拟地址。 |
| `r_info` | 此成员给出要进行重定位的符号表索引，以及将实施的重定位类型。例如一个调用指令的重定位项将包含被调用函数的符号表索引。如果索引是 `STN_UNDEF`，那么重定位使用 0 作为“符号值”。重定位类型是和处理器相关的。当程序代码引用一个重定位项的重定位类型或者符号表索引，则表示对表项的 `r_info` 成员应用 `ELF32_R_TYPE` 或者 `ELF32_R_SYM` 的结果。<br />`#define ELF32_R_SYM(i) ((i)>>8)`<br />`#define ELF32_R_TYPE(i) ((unsigned char)(i))`<br />`#define ELF32_R_INFO(s, t) (((s)<<8) + (unsigned char)(t))` |
| `r_addend` | 此成员给出一个常量补齐，用来计算将被填充到可重定位字段的数值。 |

## so加载原理

### 开始分析linker

我们知道加载一个so文件的时候调用的是**dlopen**.经过分析,**dlopen**其实只是简单的调用位于linker模块的**do_dlopen**.

```
void* dlopen(const char* filename, int flags) { 
  ScopedPthreadMutexLocker locker(&gDlMutex); 
  soinfo* result = do_dlopen(filename, flags); 
  if (result == NULL) { 
    __bionic_format_dlerror("dlopen failed", linker_get_error_buffer()); return NULL;
  } 
  return result;
}
```

我们先看一下do_dlopen(位于安卓系统源码目录[xref/bionic/linker/linker.cpp](https://bbs.kanxue.com/elink@5feK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3q4F1k6s2u0G2K9h3c8^5M7X3g2X3i4K6u0W2j5$3!0E0i4K6u0r3y4q4)9J5k6e0c8Q4x3X3f1@1i4K6g2X3M7U0q4Q4x3V1k6^5M7X3g2X3i4K6u0r3j5X3W2G2L8X3W2U0i4K6u0r3L8r3W2F1K9$3g2J5i4K6u0r3L8r3W2F1K9$3g2J5i4K6u0W2j5%4m8H3i4K6t1K6z5o6t1K6))

```
soinfo* do_dlopen(const char* name, int flags) {
  if ((flags & ~(RTLD_NOW|RTLD_LAZY|RTLD_LOCAL|RTLD_GLOBAL)) != 0) {
    DL_ERR("invalid flags to dlopen: %x", flags);
    return NULL;
  }
  set_soinfo_pool_protection(PROT_READ | PROT_WRITE);
  soinfo* si = find_library(name);
  if (si != NULL) {
    si->CallConstructors();//调用init函数
  }
  set_soinfo_pool_protection(PROT_READ);
  return si;
}
```

- 调用`find_library` 返回`soinfo`结构

- 调用`CallConstructors`.

我们先看一下`soinfo`这个结构体,这个结构将贯穿整个分析过程.

```
struct soinfo {
     public:
      char name[SOINFO_NAME_LEN];
      const Elf32_Phdr* phdr;
      size_t phnum;
      Elf32_Addr entry;
      Elf32_Addr base;
      unsigned size;

      uint32_t unused1;  // DO NOT USE, maintained for compatibility.

      Elf32_Dyn* dynamic;

      uint32_t unused2; // DO NOT USE, maintained for compatibility
      uint32_t unused3; // DO NOT USE, maintained for compatibility

      soinfo* next;
      unsigned flags;

      const char* strtab;
      Elf32_Sym* symtab;

      size_t nbucket;
      size_t nchain;
      unsigned* bucket;
      unsigned* chain;

      unsigned* plt_got;

      Elf32_Rel* plt_rel;
      size_t plt_rel_count;

      Elf32_Rel* rel;
      size_t rel_count;

      linker_function_t* preinit_array;
      size_t preinit_array_count;

      linker_function_t* init_array;
      size_t init_array_count;
      linker_function_t* fini_array;
      size_t fini_array_count;

      linker_function_t init_func;
      linker_function_t fini_func;

    #if defined(ANDROID_ARM_LINKER)
      // ARM EABI section used for stack unwinding.
      unsigned* ARM_exidx;
      size_t ARM_exidx_count;
    #elif defined(ANDROID_MIPS_LINKER)
      unsigned mips_symtabno;
      unsigned mips_local_gotno;
      unsigned mips_gotsym;
    #endif

      size_t ref_count;
      link_map_t link_map;

      bool constructors_called;

      // When you read a virtual address from the ELF file, add this
      // value to get the corresponding address in the process' address space.
      Elf32_Addr load_bias;

      bool has_text_relocations;
      bool has_DT_SYMBOLIC;

      void CallConstructors();
      void CallDestructors();
      void CallPreInitConstructors();

     private:
      void CallArray(const char* array_name, linker_function_t* functions, size_t count, bool reverse);
      void CallFunction(const char* function_name, linker_function_t function);
};
```

我们来写一个简单的ndk验证一下这个soinfo结构
