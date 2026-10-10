
# 关于 ELF 文件格式的实验

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-15.htm`](https://www.pediy.com/kssd/pediy07/pediy7-15.htm)

现代 Linux 采用 ELF 做为其可连接和可执行文件的格式，因此 ELF 格式也向我们透出了一点 Linux 核内的情景，就像戏台维幕留下的一条未拉严的缝。本文着重讲述 32 位 ELF 的同时附带了 64 位的信息，这两种格式如此雷同，以致于初次接触 ELF 的读者不必兼顾左右。如果你对 Windows 比较熟悉，本文还将时时把你带回到 PE 中，在它们的相似之处稍做比较。ELF 文件以“ELF 头”开始，后面可选择的跟随着程序头和节头。地理学用等高线与等温线分别展示同一地区的地势和气候，程序头和节头则分别从加载与连接角度来描述 EFL 文件的组织方式。下面我们进入正文。

## 一、ELF 头

ELF 头也叫 ELF 文件头，它位于文件中最开始的地方。我用系统的是 Fedora Core 2，它在 `elf.h` 文件中同时给出了 ELF 头在 32 位系统和 64 位系统下的结构，我们先来看一下：

```c
typedef struct
{
  unsigned char e_ident[EI_NIDENT];
  Elf32_Half    e_type;
  Elf32_Half    e_machine;
  Elf32_Word    e_version;
  Elf32_Addr    e_entry;
  Elf32_Off e_phoff;
  Elf32_Off e_shoff;
  Elf32_Word    e_flags;
  Elf32_Half    e_ehsize;
  Elf32_Half    e_phentsize;
  Elf32_Half    e_phnum;
  Elf32_Half    e_shentsize;
  Elf32_Half    e_shnum;
  Elf32_Half    e_shstrndx;
} Elf32_Ehdr;

typedef struct
{
  unsigned char e_ident[EI_NIDENT];
  Elf64_Half    e_type;
  Elf64_Half    e_machine;
  Elf64_Word    e_version;
  Elf64_Addr    e_entry;
  Elf64_Off e_phoff;
  Elf64_Off e_shoff;
  Elf64_Word    e_flags;
  Elf64_Half    e_ehsize;
  Elf64_Half    e_phentsize;
  Elf64_Half    e_phnum;
  Elf64_Half    e_shentsize;
  Elf64_Half    e_shnum;
  Elf64_Half    e_shstrndx;
} Elf64_Ehdr;
```

`elf.h` 中关于 ELF 格式所有结构给出的定义，其成员字段的类型声名都是 C 语言基本类型的别名，不会再嵌套结构。可以看出 32 位系统和 64 位系统下 ELF 头的结构基本相同，不同的是两种结构中的某个成员字段占用字节个数有所变化。比如 `e_entry` 由 32 位下占 4 个字节的 `Elf32_Addr` 变为 64 位下占 8 个字节的 `Elf64_Addr`，这是因为两种系统下 CPU 寻址能力不同造成的。同理文件偏移也从 4 字节的 `Elf32_Off` 变为 8 字节的 `Elf64_Off`。有些成员字段虽然类型声名从 `Elf32_XXXX` 变成了 `Elf64_XXXX`，该域所占的字节个数并未改变。如 `Elf32_Half` 和 `Elf64_Half` 都占两个字节，`Elf32_Word`、`Elf32_Sword`、`Elf64_Word`、`Elf64_Sword` 全都是 4 个字节。尽量使用 `elf.h` 中的现有定义将使我们写的程序具有很强的可移植性。另外 ELF 格式在两种系统下的这种雷同也使得我们可放心的抛弃它们的差别，专心研究其中的一种，然后再轻松的掌握另一种。

ELF 头中每个字段的含意如下：

### `e_ident`

这个字段是 ELF 头结构中的第一个字段，在 `elf.h` 中 `EI_NIDENT` 被定义为 16，因此它占用 16 个字节。

`e_ident` 的前四个字节顺次应该是 `0x7f`、`0x45`、`0x4c`、`0x46`，也就是"\\177ELF"。这是 ELF 文件的标志，任何一个 ELF 文件这四个字节都完全相同。它让熟悉 Windows 的人想起 `MZ` 和 `PE\O\O`。

第 5 个字节标志了 ELF 格式是 32 位还是 64 位，32 位是 1，64 位是 2。

第 6 个字节，在 0x86 系统上是 1，表明数据存储方式为低字节优先。

第 10 个字节，指明了在 `e_ident` 中从第几个字节开始后面的字节未使用。

### `e_type`

ELF 文件的类型，1 表示此文件是重定位文件，2 表示可执行文件，3 表示此文件是一个动态连接库。

### `e_machine`

CPU 类型，它指出了此文件使用何种指令集。如果是 Intel 0x386 CPU 此值为 3，如果是 AMD 64 CPU 此值为 62 也就是 16 进制的 `0x3E`。

### `e_version`

ELF 文件版本。为 1。

### `e_entry`

可执行文件的入口虚拟地址。此字段指出了该文件中第一条可执行机器指令在进程被正确加载后的内存地址！（PE 可执行文件指出的是入口的相对虚拟地址 RVA，它是相对于文件加载起始地址的一个偏移值，因此理论上 PE 文件可被加载到进程序空间任何位置，而 ELF 可执行文件只能被加载到固定位置）。

### `e_phoff`

程序头在 ELF 文件中的偏移量。如果程序头不存在此值为 0。

### `e_shoff`

节头在 ELF 文件中的偏移量。如果节头不存在此值为 0。

### `e_ehsize`

它描述了“ELF 头”自身占用的字节数。

### `e_phentsize`

程序头中的每一个结构占用的字节数。程序头也叫程序头表，可以被看做一个在文件中连续存储的结构数组，数组中每一项是一个结构，此字段给出了这个结构占用的字节大小。`e_phoff` 指出程序头在 ELF 文件中的起始偏移。

### `e_phnum`

此字段给出了程序头中保存了多少个结构。如果程序头中有 3 个结构则程序头在文件中占用了 3×`e_phentsize` 个字节的大小。

### `e_shentsize`

节头中每个结构占用的字节大小。节头与程序头类似也是一个结构数组，关于这两个结构的定义将分别在讲述程序头和节头的时候给出。

### `e_shnum`

节头中保存了多少个结构。

### `e_shstrndx`

这是一个整数索引值。节头可以看作是一个结构数组，用这个索引值做为此数组的下标，它在节头中指定的一个结构进一步给出了一个“字符串表”的信息，而这个字符串表保存着节头中描述的每一个节的名称，包括字符串表自己也是其中的一个节。

至此为止我们已经讲述了“ELF 头”，在此过程中提前提到的一些将来才用的概念，不必急于了解。现在读者可自己编写一个小程序来验证刚学到的知识，这有助于进一步的学习。`ELF.elf.h` 文件一般会存在于 `/usr/include` 目录下，直接 `include` 它就可以。但我们能够验证的知识有限，当更多知识联系在一起的时候我们的理解正误才可以得到更好的验证。接下来我们再学习程序头。

它需要大量测试而我不能满足它，希望各位能下载试用，帮我发现 BUG。先行谢过了！！

图形界面支持 AMD64 指令集的动态反汇编程序。`hdasm64`

本身是一个 Win32 程序（PE32 格式）在下载的包里有 64 位的 PE 和 ELF 文件各一个，32 位 ELF 文件一个，供大家研究！

支持实模式、保护模式、64 位模式三种模式指令集的动态反汇编. 支持 DOS 系统 COM 和 EXE 可执行文件格式，支持 32 位和 64 位 PE 文件格式（windows 可执行文件），支持 32 位和 64 位 ELF 文件格式（Linux 可执行文件）,共计六种文件格式。支持部分指令的虚拟执行调试。

下载页面：

[`pay500.com/s/s56504.htm`](http://pay500.com/s/s56504.htm)

### 程序头

程序头有时也叫程序头表，它保存了一个结构数组。程序头是从加载执行的角度看待 ELF 文件的结果，从它的角度 ELF 文件被分成许多个段。每个段保存着用于不同目的的数据，有的段保存着机器指令，有的段保存着已经初始化的变量;有的段会做为进程映像的一部分被操作系统读入内存，有的段则只存在于文件中。熟悉 Windows 的读者很容易理解，因为从这个角度来讲程序头的作用有点像 PE 文件中的节表。后面还会讲到 ELF 的节头，节头把 ELF 文件分成了许多节。ELF 文件的一部分常常是既在某一段中又在某一节中。Linux 和 Windows 的进程空间都采用的是平坦模式，没有 x86 的段概念，这里 ELF 中提到的段仅是文件的分段与 x86 的段没有任何联系。

我们仍然先看一下程序头中结构的定义，它们在 32 位系统与 64 系统下是多么雷同！

```c
typedef struct
{
  Elf32_Word    p_type;
  Elf32_Off p_offset;
  Elf32_Addr    p_vaddr;
  Elf32_Addr    p_paddr;
  Elf32_Word    p_filesz;
  Elf32_Word    p_memsz;
  Elf32_Word    p_flags;
  Elf32_Word    p_align;
} Elf32_Phdr;

typedef struct
{
  Elf64_Word    p_type;
  Elf64_Word    p_flags;
  Elf64_Off p_offset;
  Elf64_Addr    p_vaddr;
  Elf64_Addr    p_paddr;
  Elf64_Xword   p_filesz;
  Elf64_Xword   p_memsz;
  Elf64_Xword   p_align;
} Elf64_Phdr;
```

注意有几三个字段改为 `Elf64_Xword` 类型它们都占 64 个二进制位。如果手头有一个 ELF 文件（当然有），把文件指针移到在 ELF 头中 `e_phoff` 字段给出的位置，然后读出的内容就是程序头了。程序头中保存着 `e_phnum`（ELF 头的成员）个 `Elf32_Phdr` 或 `Elf64_Phdr` 结构，每一个这样的结构描述了一个段，下面通过了解结构中每个字段来了解程序头和这些段吧！

#### `p_type`

段的类型，它能告诉我们这个段里存放着什么用途的数据。此字段的值是在 `elf.h` 中定义了一些常量。例如 1（`PT_LOAD`）表示是可加载的段，这样的段将被读入程序的进程空间成为内存映像的一部分。段的种类再不断增加，例如 7（`PT_TLS`）在以前就没有定义，它表示用于线程局部存储。

#### `p_flags`

段的属性。它用每一个二进制位表示一种属，相应位为 1 表示含有相应的属性，为 0 表示不含那种属性。其中最低位是可执行位，次低位是可写位，第三低位是可读位。如果这个字段的最低三位同时为 1 那就表示这个段中的数据加载以后既可读也可写而且可执行的。同样在 `elf.h` 文件中也定义了一此常量（`PF_X`、`PF_W`、`PF_R`）来测试这个字段的属性，做为一个好习惯应该尽量使用这此常量。

#### `p_offset`

该段在文件中的偏移。这个偏移是相对于整个文件的。

#### `p_vaddr`

该段加载后在进程空间中占用的内存起始地址。

#### `p_paddr`

该段的物理地地址。这个字段被忽略，因为在多数现代操作系统下物理地址是进程无法触及的。

#### `p_filesz`

该段在文件中占用的字节大小。有些段可能在文件中不存在但却占用一定的内存空间，此时这个字段为 0。

#### `p_memsz`

该段在内存中占用的字节大小。有些段可能仅存在于文件中而不被加载到内存，此时这个字段为 0。

#### `p_align`

对齐。现代操作系统都使用虚拟内存为进程序提供更大的空间，分页技术功不可没，页就成了最小的内存分配单位，不足一页的按一页算。所以加载程序数据一般也从一页的起始地址开始，这就属于对齐。

尽管我给出了描述每个段信息的程序头结构，但我并不打算介绍任何一个具体类型的段所存储的内容，大多数情况下它们和节中保存的内容是一致的。我们只关心可以加载的段，但上面给出的信息应该足够了。好啦，你现在就是操作系统，你已经知道了组成程序的指令和数据都存放在文件的各个段中，通过程序头你知道它们在文件中的偏移和它们在文件中的大小，你就可以把这个段读到它的进程空间中以 `p_vaddr` 开始的地址处。水平所限，我所能表达的必然不是精确的，为了更好理解程序头与进程加载，我设计了一个小实验并给出 C 语言代码——代码可以精确的说明一切！

现在掌握了 ELF 头和程序头，从加载执行程序的角度可以说已对 ELF 文件有了初步的了解。为更好理解它，做个试验吧！

回忆一下程序头表把 ELF 文件分成了许多段，并告诉操作系统怎样把这些段读到内存里去。当操作系统已按程序头表的指示把 ELF 文件各个段的数据读入到内存中相应的地方以后，就可以说操作系统已建立了完整且正确的进程映像（如果不考虑依赖），下一步就是要执行程序了。ELF 头的 `e_entry` 给出了第一条机器指令在内存中的地址，操作系统只要在某个时候将指令流引向那里就可以了。

这个猜测对不对呢，下面的这个实验将从某种角度来证明它。首先准备好一段代码，把这段代码写到 ELF 文件中，代码写入的位置恰恰是 ELF 文件的第一条机器指令在文件中的位置。这样当系统把这个修改过的可执行程序加载到内存时，它原来入口处的指令已经换成了我们准备的这段代码，程序的行为被完全改变。可是 ELF 头的 `e_entry` 给出的是内存地址而不文件偏移，所以这需要我们自己找到这个文件偏移。怎么找？运用刚刚掌握的知识。程序头不是给出了文件中每一段对应的内存起始地址吗，还有每一段在内存中占了多少字节。只要遍历程序头中的每一个结构，看看哪个段的起始内存地址小于等于 `e_entry` 并且该地址加上该段内存大小又大于 `e_entry`，那么这个段就是程序第一条指令所在的段。第一条指令在段中偏移就是 `e_entry` 减去该段的 `p_vaddr` 所得的值，第一条指令在整个文件中偏移 ＝该段的 `p_offset` ＋（`e_entry` －该段的 `p_vaddr`）。

下面就是我准备的那段代码，它是一个没有参数的 C 函数 `exit_print`。对于这段代码有三点需要说明。1）这个函数中不能调用常用的库函数，因为若从 so 中取函数我们现在无法解决动态引入;如果采用静态连接，被调用函数有可能再调用其它函数，而被调用函在内存映像的地址、大小都不易掌握。2）这个段代码最好是位置无关代码，这样能减少这个实验的代码量，而使用全局或静态变量将使我们花更大代价来实现位置无关，所以这个函数不使用它们。3）这个函数只能在 IA32 机器上运行，若想在其它环境下做此实验必须修改它的一段汇编代码。另外我们没有判断 ELF 文件是否为可执行文件。为了确信这段代码被运行，它将在控制台输出“hello!Hangj!”之后就结束整个程序。鉴于上面的两点说明，我们不能使用 `printf` 和 `malloc` 输出字符串和为它分配内存，也没有把完整的字符串做为变量存储，而是用了堆栈中的局部变量,这将导致栈中内存分配。把字符串放到 `strHello` 中用了四条 C 语句。注意，前三条中每条语句放入的四个字符的顺序是颠倒的，这是 x86 低字节优先存储造成的。最后一条 C 语句放入一个回车符‘\\n’，字符串没有以 0 结尾。

```c
void exit_print()
{
    char strHello[20];
    *((unsigned long*)&strHello[0])='lleH';
    *((unsigned long*)&strHello[4])='aH!o';
    *((unsigned long*)&strHello[8])='!jgn';
    strHello[12]='\\n';
    __asm__ volatile ("int $0x80; movl $0,%%ebx; movl $1,%%eax; int $0x80"
            :
            : "a"((long) 4),
              "b" ((long)1),
              "c" ((long)(&strHello[0])),
```



`d` ((long)13));

}

`exit_print` 用到了一些汇编语法，不防在这里先复习下汇编,如果你不喜欢看汇编，可以直接阅读后面给出的完整 C 代码，我可以保证它实现上面想要的功能。`gcc` 内部汇编以`__asm__`开始，关键字 `volatile` 告诉 `gcc` 不要优化。汇编体以一对小括号包围并以分号结束：输入部分把寄存器 `EAX` 置为 4，这是 `write` 系统调用的功能号;`EBX` 置为 1，这 `write` 系统调用使用的文件句柄，1 代表标准输出设备;寄存器 `ECX` 置为字符串的起始地址;寄存器 `EDX` 置为 13，这代表字符串的长度是 13 个字节;我们不关心系统返回值因此输出部分没有内容;接下来 `int $0x80` 把刚才的设置到寄存器的参数传给内核完成打印功能！后面在把寄存器 `EBX` 置 0、`EAX` 置 1 后又是一次系统调用，它将结束当前进程并把 `EBX` 中的 0 返回给父进程。函数 `exit_print` 说明完毕！

下面给出这个试验程序的完整代码，它被存为 `mod_entry.c` 文件，`exit_print` 函数也在其中：

```
//文件名      :mod_entry.c
//功能      :  覆盖 ELF 可执行文件指令入口
//创建      :  2004.11.28
//修改日期    :  2004.11.28
//作者      :  
//  
////////////////////////////////////////////////////////////////////////
#include "stdio.h"
#include "unistd.h"
#include "fcntl.h"
#include "elf.h"
void exit_print()
{
    char strHello[20];
    *((unsigned long*)&strHello[0])='lleH';
    *((unsigned long*)&strHello[4])='aH!o';
    *((unsigned long*)&strHello[8])='!jgn';
    strHello[12]='\\n';
    __asm__ volatile ("int $0x80; movl $0,%%ebx; movl $1,%%eax; int $0x80"\
            :" \
            : "a"((long) 4),\
             "b" ((long)1),\
             "c" ((long)(&strHello[0])),\
             "d" ((long)13));
}
/*
AMD 64 下的调用 write 系统调用可能有如下形式,其中__syscall 是中断调用指令，__NR_write 是系统功能号：
__asm__ volatile (__syscall \
  : \
  : "a" (__NR_write),"D" ((long)(1)),"S" ((long)(&strHello[0])),"d" ((long)(13)) : "r11","rcx","memory" );
*/
//简单判断是否是 ELF 文件
int IsElf(Elf32_Ehdr *pEhdr)
{
  if(pEhdr->e_ident[EI_MAG0]!=0x7f  
    ||pEhdr->e_ident[EI_MAG1]!='E'
    ||pEhdr->e_ident[EI_MAG2]!='L'
    ||pEhdr->e_ident[EI_MAG3]!='F'
    ||pEhdr->e_machine!=EM_386)//是否在 x86 上运行
    return 0;
  return 1;
}
//从指定的位置读文件
int ReadAt(int hFile,int pos,void *buf,int count)
{
  if(pos==lseek(hFile,pos,SEEK_SET))
  {
    return read(hFile,buf,count);
  }
  return -1;
}
//从指定的位置写文件
int WriteAt(int hFile,int pos,void* buf,int count)
{
  if(pos==lseek(hFile,pos,SEEK_SET))
  {
    return write(hFile,buf,count);
  }
  return -1;
}
//找到程序第一条指令所在的段，并把该段的程序头结构读到 pPhdr 指向的结构中
int  FileEntryIndex(int hFile,Elf32_Ehdr*pEhdr,Elf32_Phdr *pPhdr,unsigned long entry)
{
  int i;
  for(i=0;i<pEhdr->e_phnum;i++)
  {
      if(sizeof(*pPhdr)!=ReadAt(hFile,
                                      pEhdr->e_phoff+i*pEhdr->e_phentsize,
                                      pPhdr,
                                      sizeof(*pPhdr)))
    return 0;
      if(entry >= pPhdr->p_paddr && 
                       entry < (pPhdr->p_paddr + pPhdr->p_memsz))
      {
    return 1;
      }
  }
  return 0;
}
int main()
{
  int hFile;
  int offset,size;
  Elf32_Ehdr ehdr;
  Elf32_Phdr phdr;
  //以读写方式打开文件
  hFile=open("/home/hangj/hello",O_RDWR,0);    
  if(hFile<0)
    return -1;
  //读取 ELF 头
  if(sizeof(ehdr)!=ReadAt(hFile,0,&ehdr,sizeof(ehdr)))
    goto error;
  //判断是否是 ELF 文件
  if(!IsElf(&ehdr))
    goto error;
  //找到该文件第一条指令所在的段并读出这个段的程序头结构信息
  if(!FileEntryIndex(hFile,&ehdr,&phdr,ehdr.e_entry))
    goto error;
  //计算第一条指令在整个文件中的位置
  offset=ehdr.e_entry-phdr.p_paddr;
  offset+=phdr.p_offset;
  //计算 exit_print 函数体的字节数
  size=(int)(&IsElf)-(int)(&exit_print);
  //修改 ELF 文件，注意写入缓冲区的起始地址是 exit_print。
  if(size!=WriteAt(hFile,offset,exit_print,size))
    goto error;
  printf("write Elf file success!\\n");
error:
  close(hFile);
  return 0;
}
```

编译的时候 `gcc` 会有如下警告提示！

```
[root@hangj elf]# gcc mod_entry.c -o mod_entry
mod_entry.c:20:37: warning: multi-character character constant
mod_entry.c:21:37: warning: multi-character character constant
mod_entry.c:22:37: warning: multi-character character constant
[root@hangj elf]#
```

不用在意这个警告，它毫无防碍。这个程序非常简单，因为它忽略了许多本该注意的问题，比如被修改的 ELF 文件的那个段是否足够大可以容下我们的 `exit_print` 函数体？实事上我们的函数很小，它几乎总能使你的试验成功。我把最经典的 `hello` 程序放到 `/home/hangj/` 目录下用来完成这个实验，你也可修改为使用其它 ELF 可执行文件进行这个实验。

### 节头

节头也叫节头表。ELF 头的 `e_shoff` 字段给出了节头在整个文件中的偏移（如果节头存在的话），节头可看做一个在文件中连续存储的结构数组，数组的长度由 ELF 头的 `e_shnum` 字段给出，数组中每个结构的字节大小由 ELF 头的 `e_shentsize` 字段给出。把文件指针移到在 ELF 头中 `e_shoff` 给出的位置，然后读出的内容就是节头了。节头表是从连接角度看待 ELF 文件的结果，所以从节头的角度 ELF 文件分成了许多的节，每个节保存着用于不同目的的数据，这些数据可能被前面提到的程序头重复引用。关于节的内容非常琐碎，完成一次任务所的需的信息往往被分散到不同的节里。相对而言，PE 中的资源表、引入表、导出表都集中给出了所有相关的信息，理解起来真是方便多了。由于节中数据的用途不同，节被分为不同的类型，每种类型的节都有自己组织数据的方式。有的节存储着一些字符串，例如前面提过的字符串表就是一种类型的节;有的节保存一张符号表，程序从动态连接库中引入的函数和变量都会出现在一个叫做”动态符号表“的节中;重定位表则包含在重定位节中。不管这些节是何种类型，在节头中都用相同的结构保存着与这些节有关的信息。先来看一看节头中用来保存这些信息的结构吧：

```c
typedef struct
{
  Elf32_Word  sh_name;
  Elf32_Word  sh_type;
  Elf32_Word  sh_flags;
  Elf32_Addr  sh_addr;
  Elf32_Off   sh_offset;
  Elf32_Word  sh_size;
  Elf32_Word  sh_link;
  Elf32_Word  sh_info;
  Elf32_Word  sh_addralign;
  Elf32_Word  sh_entsize;
} Elf32_Shdr;
typedef struct
{
  Elf64_Word  sh_name;
  Elf64_Word  sh_type;
  Elf64_Xword sh_flags;
  Elf64_Addr  sh_addr;
  Elf64_Off   sh_offset;
  Elf64_Xword sh_size;
  Elf64_Word  sh_link;
  Elf64_Word  sh_info;
  Elf64_Xword sh_addralign;
  Elf64_Xword sh_entsize;
} Elf64_Shdr;
```

略过 32 位与 64 位的雷同不说，我们逐个字段的讲解这个结构！

`sh_name`:
这个整数占用两个字节，它能告诉你这个节的名字。讲字符串表的时候会再来研究这个字段。

`sh_type`:
节的类型。或者说它能告诉你这个节里存放的是什么样的数据。随着 ELF 的发展，用于不同目的的节会不断增多，节的类型值是在 `elf.h` 中定义的一些常量。例如字符串表是 `SHT_STRTAB`，符号表是 `SHT_SYMTAB` 等。

`sh_flags`:
节的属性。这个字段在 32 位下占 4 个字节，64 位下占 8 个字节。与程序头中的 `p_flags` 字段一样，它用每一个二进制位表示一种属。其中最低位如果为 1 表示此节在进程执行过程中可写，次低位为 1 表示此节的内容加载时要读到内存中去，第三低位为 1 表示这个节中的数据是可执行的机器指令。一些常量，如 `SHF_INFO_LINK`，帮助用来测试节的属性。

`sh_addr`:
如果此节的内容将出现在进程空间里，这个字段给出了该节在内存中起始地址。

`sh_offset`：
如果此节在文件中占用一定的字节，这个字段给出了该节在整个文件中的起始偏移量。

`sh_size`：
如果此节在文件中占用一定的字节，这个字段给出了该节在文件中的字节大小，如果此节在文件中不存在但却存在于内存中那么此字段给出了此节在内存中的字节大小。

`sh_link`：
如果另一个节与这个节相关联，这个字段给出了相关的节在节头中的索引。

`sh_info`：
这个字段如果用到再说。

`sh_addralign`：
地址对齐。这个数是 2 的整数次幂，对齐只能是 2 字节对齐、4 字节对齐、8 字节对齐等。如果这个数是 0 或 1 表示这个节不用对齐。

`sh_entsize`：
这个字段是一个代表字节大小的数，对某些节才有意义。例如对动态符号节来说这个字段就给出动态符号表中每个符号结构的字节大小。

节头的结构讲完了，很多一时用不到的知识被暂时抛弃。对于节的知识我们掌握了很少，但我希望至少能够知道每个节的名字。所以我们必须开始接触我们将要学习的几种类型节的第一类————字符串表！

### 字符串表

用到字符串表的地方很多，也这是我们需要首先了解它的原因！
如果一个节头结构的 `sh_type` 字段值为 `SHT_STRTAB`，那么相应的节就是一个字符串表。字符串表保存着一系列以 NULL 结尾的的字符串，这符合 C 语言的习惯。值得注意的是字符串表的第一个字节为 0，它代表了一个空串。字符串表的最后一个字节也为 0，它是表中最后一个字符串的结尾。没有任何字节的字符串表也允许存在这时对字符串的引用是无效的。字符串表试图节省空间，所以字符串的子串也会被引用。表中的字符串是怎样被具体引用的呢？获取节的名称就是我们说明如何使用字符串表的第一个例子。节头结构的 `sh_name` 字段是个两字节的整数，这个整数是从字符串表第一个字节开始的偏移值。所以如果我们已经拿到了一个字符串表，那么从它的第 `sh_name` 个字节开始直到遇到一个 0 结束，之间所有的字符构成了我们所要的字符串。看一下 ELF 头的最后一个字段 `e_shstrndx`，还记得对它的解释吗？对，它指出的那个节就是一个字符串表，文件中各个节的名字都存在这里！ELF 中用到字符串表的地方给出的都是一个字符串表在节头表中的索引，就象 `e_shstrndx`，我们看一下如何使用。

下面给出一些 C 函数，它们将以 ELF32 为例打印文件中每一个节的名称和其它信息。此处未给出的函数，都在“覆盖 ELF 可执行文件入口指令”实验中给出过，为省篇幅不再重复给出。为了方便我们先写一个辅助函数 `ReadSection`，这个函数接受的头两个参数是 ELF 的文件句柄和 ELF 头结构的指针，第三个参数是需要读取的节在节头中的索引。如果读取成功它将用 `malloc` 分配内存并返回这块内存的指针，调用者负责释放内存。真正打印节名和其它节信息的是函数 `printSection`，这个函数接受一个 ELF 的文件名做为参数。读者可自行编写 `main` 函数来调用 `printSection`。后面每讲到一种节,我都会给出相应的函数用于打印这个些节中包含的内容。一方面是为了更好的了解这些节存储信息的方式，一方面也是为后续的学习积累工具。

```c
char *ReadSection(int hFile,Elf32_Ehdr*pEhdr,int index)
{
  char *pbuf;
  Elf32_Shdr shdr;
  int offset;
  if(index<0||index>pEhdr->e_shnum)
    return NULL;
  //相应的节头结构在文件中的偏移＝节头偏移＋每个节头结构的字节大小×节索引
  offset=pEhdr->e_shoff+pEhdr->e_shentsize*index;
  //从文件中读取一个节头结构
  if(sizeof(shdr)!=ReadAt(hFile,offset,&shdr,sizeof(shdr)))
    return NULL;
  //分配与节字节大小相同的内存块
  pbuf=(char *)malloc(shdr.sh_size);
  if(pbuf!=NULL)
  {
    //把该节全部读入内存，成功就返回内存地址，否则释放内存返回 NULL
```




```
      if(shdr.sh_size==ReadAt(hFile,shdr.sh_offset,pbuf,shdr.sh_size))  

          return pbuf;  

      free(pbuf);  

  }  

  return NULL;  

}  

void printSection(char *fileNameOfElf)  

{  

  int hFile;  

  int offset;  

  Elf32_Ehdr ehdr;  

  Elf32_Shdr shdr;  

  char *strTable;  

  strTable=NULL;  

  //以只读方式打开 ELF 文件  

  hFile=open(fileNameOfElf,O_RDONLY,0);    

  if(hFile<0)  

  {  

      printf("can not open file:%s\n",fileName);  

      return;  

  }  

  //读取 ELF 头  

  if(sizeof(ehdr)!=ReadAt(hFile,0,&ehdr,sizeof(ehdr)))  

      goto error;  

  //判断是否是 ELF 文件  

  if(!IsElf(&ehdr))goto error;  

  //判断节头是否存在  

  if(ehdr.e_shnum<=0||ehdr.e_shoff==0)  

  {  

      printf("this ELF have not Section Head Table!\n");  

      goto close_file;  

  }  

  //把字符串表读到内存中，strTable 保存了字符串表在内存中的地址  

  strTable=ReadSection(hFile,&ehdr,ehdr.e_shstrndx);  

  if(strTable==NULL)  

      goto error;  

  int i;  

  //循环读取每一个节头结构，打印出节的信息  

  for(i=0;i<ehdr.e_shnum;i++)  

  {  

      //读取第 i 个节头结构  

      if(sizeof(shdr)==ReadAt(hFile,  

        ehdr.e_shoff+i*ehdr.e_shentsize,  

        &shdr,  

        sizeof(shdr)))  

      {  

    //调用 printf 打印包括节名在内的节信息  

    printf("section name=%s\n"  
                       "\tfstart=0x%x,\tfsize=0x%x,\tmemstart=0x%x\n",  
        &strTable[shdr.sh_name],//节名字符串地址  
        shdr.sh_offset,    //节在文件中的偏移  
        shdr.sh_size,    //节的字节大小  
        shdr.sh_addr);    //节在内存中的地址  

      }    

  }  

  goto close_file;  

error:  

  printf("read section info error!\n");  

close_file:  

  if(strTable)  

      free(strTable);//释放内存  

  close(hFile);//释放文件句柄  

}  
```

我的函数写得很笨拙，但请读者把注意力放到如何读取一个节和如何用索引来查找字符串上。请你调用 `printSection` 函数，你会从这个实验结果中发现：你的 ELF 文件的第一个节没有名字，而且这个节的其它信息都为 0。是的每个 ELF 文件的头一个节都被保留不用。

如果一个节头结构的 `sh_type` 字段为 `SHT_SYMTAB` 或 `SHT_DYNSYM`，那么相应的节就是一个符号节，符号节里存放的是一张符号表。符号表也是一个连续存储的结构数组。为了说明方便我把结构的定义从 `elf.h` 中拿出来放到下面。什么是符号呢？编程过程中用到的变量和函数都可以称之为符号。一个 ELF 文件中并不只有一个符号节，通常是两个。一个叫动态符号节，从打印节名的实验中你可以找到名为“.dynsym 的”节，它的类型值是 `SHT_DYNSYM`，所有引入的外部符号部在这里有所体现。另一个类型值为 `SHT_SYMTAB`，名字如你所见是“.symtab”，它保存了所有有用的符号的信息。一般我不会依靠节的名称来寻找一个节，虽然它们都在《ELF 规范》中有定义。

```
typedef struct  

{  

  Elf32_Word  st_name;  

  Elf32_Addr  st_value;  

  Elf32_Word  st_size;  

  unsigned char  st_info;  

  unsigned char  st_other;  

  Elf32_Section  st_shndx;  

} Elf32_Sym;  

typedef struct  

{  

  Elf64_Word  st_name;  

  unsigned char  st_info;  

  unsigned char st_other;  

  Elf64_Section  st_shndx;  

  Elf64_Addr  st_value;  

  Elf64_Xword  st_size;  

} Elf64_Sym;  
```

我们看到 32 位与 64 位下两种结构的定义有了较大的不同，但只是字段出现的序顺改变，我不明白这是为什么！但这不防碍进一步学习每个字段的意义：

`st_name`：  
    又是一个名字索引。凡是字符串索引必然和一个字符串表相关联，那么与这个字段相关的字符串表在哪里呢？往下看。

`st_value`：  
    该字段给出了相应的符号值。根据符号类型和用途的不同它可能是一个内存地址也可能是一个绝对值。

`st_size`：  
    符号的字节大小。如果是数据对象可能是该对象占用的字节大小，如果是函数符号则可能是函数体的指令占用字节数。如果符号没有大小或大小不可知此字段为 0。

`st_info`：  
    这个字段包含两部分。低 4 位用来表示符号的类型，对于函数这字段的低 4 位应该等于 `STT_FUCN`;高 4 位是这个符号的绑定类型，对于从动态库中引入的函数这个字段的高 4 位应为 `STB_GLOBAL`,表示这个符号是全局符号。在 `elf.h` 中，给出了 `ELF32_ST_TYPE` 和 `ELF32_ST_BIND` 两个宏分别用于获取这个字段的低 4 位与高 4 位。相应 64 位的宏不过是它们的别名。

`st_other`：  
    此字段无用，恒为 0。

`st_shndx`：  
    每个符号都和某些节相关，这个字段给出了一个节头索引。如果函数体所在的节不存于当前文件中，此字段为 0。`Elf64_Section` 和 `Elf32_Section` 我们头一次遇到，它们都是占两字的整数。

    讲到这里我们忍不住提前讨论一下动态连接，动态连接无非两种情况：一是导出;一是引入。如果一个模块想把函数导出给别人使用，它必须能告诉模块的加载者自己能够提供哪些函数、这些函数的函数体在什么位置。ELF 的符号表能包含这两条信息。每个符号结构的 `st_name` 字段指出了一个符号名称，`st_value` 字段又指出了符号的内存地址（可能需要跟据模块被加载的位置修改）。那么函数导出就轻易解决了。函数的引入相对复杂一些。“显式”引入动态库中的函数，可以通过 `dlopen`、`dlsym`、`dlclose` 三个在 `dlfcn.h` 中定义的函数完美解决，我们并不关心。以我们从其它系统——譬如 Windows——获得的经验，若要“隐式”的从一个动态库引入一个函数必须具备两条信息：一是函数名，可以通过这里介绍的符号结构诉操作系统;一是动态库的模块名，后面将要介绍的“动态节”会给出这一信息。事实上大家都知道，Linux 上的动态连接是由第三方连动态接器而不是由操作系统解决的，但我们把动态连接器看作操作系统的一部分不去了解。既使如此，Linux 上的符号引入没有 Windows 那么简单。下面给出打印 ELF32 中所有符号的工具函数，之后我们再介绍动态节与重定位节。

```
void printSybmol(Elf32_Sym *pSym)  

{  

#define TEST_NAME(name)\\  
case name:\\  
  strcpy(typeName,#name);\\  
  break  

  char typeName[20];  

  switch(pSym->st_info&0xf)  

  {  

  TEST_NAME(STT_NOTYPE);  

  TEST_NAME(STT_OBJECT);  

  TEST_NAME(STT_FUNC);  

  TEST_NAME(STT_SECTION);  

  TEST_NAME(STT_FILE);  

  default:  

    strcpy(typeName,"unkown type");  

  }  

  //打印符号类型和属性  

  printf("type=%d(%s)\tattrib=%d\n",  
               ELF32_ST_TYPE(pSym->st_info),//符号的类型  
               typeName,  
               ELF32_ST_BIND(pSym->st_info));//绑定类型  

  //打印其它符号信息  

  printf("\tst_shndx=%d,\tst_value=0x%x(%d),\tst_size=%d\n",  
               pSym->st_shndx,//与符号相关的节  
               pSym->st_value,//符号的值  
               pSym->st_value,  
               pSym->st_size);//符号大小  

#undef TEST_NAME  

}  
```

```
void printAllSymbol(char *fileName)  

{  

  int hFile;  

  int offset,size;  

  Elf32_Ehdr ehdr;  

  Elf32_Shdr shdr;  

  char *strTable;  

  strTable=NULL;  

  hFile=open(fileName,O_RDWR,0);    

  if(hFile<0)  

  {  

    printf("can not open file:%s\n",fileName);  

    return;  

  }  

  if(sizeof(ehdr)!=ReadAt(hFile,0,&ehdr,sizeof(ehdr)))  

    goto error;  

  if(!IsElf(&ehdr))goto error;  

  if(ehdr.e_shnum<=0||ehdr.e_shoff==0)  

  {  

    printf("this ELF have not Section Head Table!\n");  

    goto close_file;  

  }  

  //读取节名符号表  

  strTable=(char*)ReadSection(hFile,&ehdr,ehdr.e_shstrndx);  

  if(strTable==NULL)  

    goto error;  

  char *symName=NULL;  

  int i,j;  

  Elf32_Sym sym;  

  //遍历每个一个节头结构，找出每个保存了符号表的节  

  for(i=0;i<ehdr.e_shnum;i++)  

  {  

      if(sizeof(shdr)==ReadAt(hFile,  
                                    ehdr.e_shoff+i*ehdr.e_shentsize,  
                                    &shdr,  
                                    sizeof(shdr)))  

      {  

      //如果此节为两种符号节中的一种  

    if(shdr.sh_type==SHT_SYMTAB||shdr.sh_type==SHT_DYNSYM)  

    {  

        //打印节的名字和节的类型  

        printf("%s(type=%s)\n\t",  
                               &strTable[shdr.sh_name],  
                               shdr.sh_type==SHT_SYMTAB?"SHT_SYTAB":"SHT_DYNSYM");  

                    //打印节的其它信息  

                    printf("offset=0x%x,\tvaddr=0x%x,\tsize=0x%x\n\n\n",  
                            shdr.sh_offset,  
                            shdr.sh_addr,  
                            shdr.sh_size);  

                    //读取与此符号表相关联的节，这个节是个字符串表，它保存了符号名。  

                    symName=ReadSection(hFile,&ehdr,shdr.sh_link);  

                    //获取符号表在文件中的偏移，把它赋值给 offset  

                    offset=shdr.sh_offset;  

                    while(offset<shdr.sh_offset+shdr.sh_size)  

                    {  

      //读取一个符号结构  

      if(shdr.sh_entsize==ReadAt(hFile,offset,&sym,sizeof(sym)))  

      {  

          //打印符号名字  

          printf("Sym:%s\n\t",symName+sym.st_name);  

          //打印符号其它信息  

          printSybmol(&sym);  

      }  

      offset+=shdr.sh_entsize;  

                     }  

                     //释放符号名字符串表  

                     free(symName);  

    }  

      }    

  }  

  goto close_file;  

error:  

  printf("read section info error!\n");  

close_file:  

  if(strTable)  

    free(strTable);  

  close(hFile);  

}  
```

对上面的函数稍做说明。只有一个参数的 `printAllSymbol` 函数将打印出你指定的 ELF 文件中所有两种类型的符号表内容，在打印每个符号信息的时候它调用了 `printSybmol`。每个符号结构的 `st_name` 字段给出了符号名在一个字符串表中的索引。如果仔细阅读上面的代码，你就知道这个保存着符号名的字符串表的索引就保存在这个符号节的节头结构的 `sh_link` 字段中！

    停下你的思路，做一个实验是有好处的。请调用 `printAllSymbol` 函数打印一个你自己的 ELF 可执行文件，你至少能从它的输出中找到一个名为“main”的符号，而且你将发这个符号存在于类型为 `SHT_SYTAB` 的符号表中。仔细观察这个符号，你又发现符号类型为 `STT_FUNC`,这表示“main”是一个函数;符号属性是 1，也就是 `STB_GLOBAL`，这表示全局只能有一个“main”函数。另外结合你刚打印的节头表信息，看看 `st_shndx` 字段指向了哪个节！同样每个符号表的第一个符号是保留不用的。

```
typedef struct  

{  

  Elf32_Sword  d_tag;  

  union  

    {  

      Elf32_Word d_val;  

      Elf32_Addr d_ptr;  

    } d_un;  

} Elf32_Dyn;  

typedef struct  

{  

  Elf64_Sxword  d_tag;  

  union  

    {  

      Elf64_Xword d_val;  

      Elf64_Addr d_ptr;    



} `d_un`;

} `Elf64_Dyn`;

动态节的类型值是 `SHT_DYNAMIC`。从动态结构的定义，我们可以看出它内容很少，其实只有两个字段。动态节也叫动态表，这样的节的里面保存着一个由动态结构形成的结构数组。尽管动态结构看起来简单，但有许多种类型。它的 `d_tag` 字段给出这个结构的类型值，而 `d_un` 字段是由 `d_val` 来决定还是由 `d_ptr` 来决定要看这个结构的类型了。真正使我们简单的是我对除 `DT_NEEDED` 类型以外的动态结构只字不提。`DT_NEEDED` 在 `elf.h` 中被定义为常量 `1`。对于 `DT_NEEDED` 类型的动态结构它的 `d_val` 字段是一个字符串表中的索引，它给出的字符串就是这个 ELF 文件所依赖的外部动态库的名称。你的 ELF 文件需要依赖多少个动态库就有多少个 `DT_NEEDED` 类型的动态结构出现在动态表中。熟悉 PE 的读者又会发问：`DT_NEEDED` 类型的动态结构给出了程序引用到的外部模块，符号表给出了程序引用的外部函数，可它是怎么告诉系统哪些函数应该到哪个模块中去找呢？ELF 中没有这样的信息，我们不能从一个 ELF 中看出哪个函数属于哪个 so 文件。与符号的名称类似：保存着动态库名字的字符串表的索引就保存在这个动态节的节头结构中，由 `sh_link` 字段给出。我们不再过多讲述动态结构，打印动态结构信息的函数读者可以仿照打印符号表的方式自己写，为省篇幅不再给出。

有了符号名和动态库的名字操作系统就可以为我们引入函数了，但在我们的程序中是谁会用这些外部函数呢，系统解析出来的函数地址应该给谁呢？这就是重定位表的功劳了！

## 重定位表

重定位表的类型有两种 `SHT_REL` 和 `SHT_RELA`，我们只谈 `SHT_REL`。

```c
typedef struct
{
  Elf32_Addr  r_offset;
  Elf32_Word  r_info;  
} Elf32_Rel;

typedef struct
{
  Elf64_Addr  r_offset;
  Elf64_Xword r_info;
} Elf64_Rel;
```

`Elf32_Rel` 或 `Elf64_Rel` 结构也很简单，它的 `r_offset` 字段给出了需要重定位的内容的地址。而它的 `r_info` 字段给出了两条信息，一条是与此重定位内容相关的符号;一条是重定位的类型。`elf.h` 中定义了四个宏：`ELF64_R_SYM`、`ELF64_R_TYPE`、`ELF32_R_SYM`、`ELF32_R_TYPE`，分别用于从这个字段中于提取符号信息和重定位类型。符号信息就是一个符号表中的索引，32 位下占这个字段的高 24 位，64 位下占高 32 位。`r_info` 字段剩余的 8 位或 32 位是重定位类型。在 i386 上从外部引入的动态函数重定位类型是 `R_386_JMP_SLOT`。由这个类型的名称可以看出动态连接在 Linux 上是处理器密切相关的东西。刚说 `r_info` 字段包含了一个符号表中的索引，对于从外部引入的动态函数来说那个符号表就是“动态符号表”，它在节头结构中的类型值为 `SHT_DYNSYM`。`r_offset` 字段是一个地址，加载之初被 `r_offset` 指向的内容——也就是一个外部符号的地址——并不正确，操作系统就根据重定位信息引用的符号找到那个地址，然后修改 `r_offset` 所指向的内容。

下面给出打印 ELF32 文件中所有重定位结构的函数，这些函数同样将成为我们后续学习工具的一部分！

```c
void printRel(Elf32_Rel *pRel)
{
#define TEST_NAME(name)\
case name:\
  strcpy(typeName,#name);\
  break
  char typeName[20];
  switch(ELF32_R_TYPE(pRel->r_info))
  {
  TEST_NAME(R_386_NONE);
  TEST_NAME(R_386_32);
  TEST_NAME(R_386_PC32);
  TEST_NAME(R_386_GOT32);
  TEST_NAME(R_386_PLT32);
  TEST_NAME(R_386_COPY);
  TEST_NAME(R_386_GLOB_DAT);
  TEST_NAME(R_386_JMP_SLOT);
  TEST_NAME(R_386_RELATIVE);
  TEST_NAME(R_386_GOTOFF);
  TEST_NAME(R_386_GOTPC);
  default:
    strcpy(typeName,"unkown type");
  }
  //打印重定位类型、在符号表中的索引、重位内容的地址。
  printf("type=%d(%s)\tSYM=%d\t offset=0x%x\n",
               ELF32_R_TYPE(pRel->r_info),//重定向类型
               typeName,
               ELF32_R_SYM(pRel->r_info),//重定向用到的符号
               pRel->r_offset);//需要系统修改的内容地址
#undef TEST_NAME
}
```

```c
void printAllRel(char *fileName)
{
  int hFile;
  int offset,size;
  Elf32_Ehdr ehdr;
  Elf32_Shdr shdr;
  char *strTable;
  strTable=NULL;
  hFile=open(fileName,O_RDWR,0);    
  if(hFile<0)
  {
    printf("can not open file:%s\n",fileName);
    return;
  }
  if(sizeof(ehdr)!=ReadAt(hFile,0,&ehdr,sizeof(ehdr)))
    goto error;
  if(!IsElf(&ehdr))goto error;
  if(ehdr.e_shnum<=0||ehdr.e_shoff==0)
  {
    printf("this ELF have not Section Head Table!\n");
    goto close_file;
  }
  strTable=(char*)ReadSection(hFile,&ehdr,ehdr.e_shstrndx);
  if(strTable==NULL)
    goto error;
  Elf32_Rel *symName=NULL;
  Elf32_Rel rel;
  int i,j;
  Elf32_Sym sym;
  //遍历节头表寻找类型为 SHT_REL 或 SHT_RELA 的节，它们是重定位表
  for(i=0;i<ehdr.e_shnum;i++)
  {
      if(sizeof(shdr)==ReadAt(hFile,
                                    ehdr.e_shoff+i*ehdr.e_shentsize,
                                    &shdr,
                                    sizeof(shdr)))
      {
    if(shdr.sh_type==SHT_REL||shdr.sh_type==SHT_RELA)
    {
        //打印包含重定位表的节名称和节类型
                    printf("%s(type=%s)\n\t",
                           &strTable[shdr.sh_name],
                           shdr.sh_type==SHT_REL?"SHT_REL":"SHT_RELA");
                    //打印此节的其它信息
                    printf("offset=0x%x,\t vaddr=0x%x,\tsize=0x%x\n\n\n",
                             shdr.sh_offset,
                             shdr.sh_addr,
                             shdr.sh_size);
                    //获取重定位表在文件中的偏移，保存在 offset 变量中
                    offset=shdr.sh_offset;
                    //循环读取每一个重定位结构
                    while(offset<shdr.sh_offset+shdr.sh_size)
                    {
      if(shdr.sh_entsize==ReadAt(hFile,offset,&rel,sizeof(rel)))
      {
                            //打印此重定位结构的内容
                            printRel(&rel);
      }
      offset+=shdr.sh_entsize;
                    }
    }
      }  
  }
  goto close_file;
error:
  printf("read section info error!\n");
close_file:
  if(strTable)
    free(strTable);
  close(hFile);
}
```

在 Linux 上我们常用的标准库函数都尽量被采用动态连接的方式连入我们的可执行文件。`libc.so.6` 就是一个动态连接库。我们用一个最简单的程序开始我们的实践，这段代码大家都见过几万遍了：

```c
#include "stdio.h"
int main()
{
    printf("Hello,Linux!");
    return 0;
}
```

`[root@hangj elf]#gcc hello.c -o hello`
`[root@hangj elf]#`

为了能看到这个代码编译后的样子，我们用 GDB 加载它。

`[root@hangj elf]#gdb hello`

这段代码的反汇编结果如下：

```
(gdb) disass main
Dump of assembler code for function main:
0x08048370 <main+0>:    push   %ebp
0x08048371 <main+1>:    mov    %esp,%ebp
0x08048373 <main+3>:    sub    $0x8,%esp
0x08048376 <main+6>:    and    $0xfffffff0,%esp
0x08048379 <main+9>:    mov    $0x0,%eax
0x0804837e <main+14>:   sub    %eax,%esp
0x08048380 <main+16>:   sub    $0xc,%esp
0x08048383 <main+19>:   push   $0x804846c
0x08048388 <main+24>:   call   0x80482b0 <_init+56>
0x0804838d <main+29>:   add    $0x10,%esp
0x08048390 <main+32>:   mov    $0x0,%eax
0x08048395 <main+37>:   leave
0x08048396 <main+38>:   ret
0x08048397 <main+39>:   nop
End of assembler dump.
```

显然唯一的一条函数调用“call   0x80482b0 <\_init+56>”应该就是 `printf`，看看 `0x80482b0` 处有什么？

```
(gdb) x/10i 0x80482b0
0x80482b0 <_init+56>:   jmp    *0x8049570
0x80482b6 <_init+62>:   push   $0x8
0x80482bb <_init+67>:   jmp    0x8048290 <_init+24>
0x80482c0 <_start>:     xor    %ebp,%ebp
0x80482c2 <_start+2>:   pop    %esi
0x80482c3 <_start+3>:   mov    %esp,%ecx
0x80482c5 <_start+5>:   and    $0xfffffff0,%esp
0x80482c8 <_start+8>:   push   %eax
0x80482c9 <_start+9>:   push   %esp
0x80482ca <_start+10>:  push   %edx
```

原来是一条跳转指令，要跳到的地址被保存在 `0x8049570` 处，按照我们对 PE 文件的经验 `0x8049570` 处一定存放着 `printf` 的真正地址。系统加载 `hello` 时从重定位表和动态表中知道我们引用了 `libc.so.6` 中的 `printf`，然后它就加载 `libc.so.6`，然后它就从 `libc.so.6` 的符号表中找到了 `printf`,然后它就把 `hello` 的 `0x8049570` 处的四个字节改成 `printf` 的地址！一定是这样！那么这一条跳转就是去执行 `printf` 了。看看再说。

```
(gdb) x/4xw 0x8049570
0x8049570 <_GLOBAL_OFFSET_TABLE_+16>:   0x080482b6      0x00000000      0x00000000      0x0804948c
```

“0x080482b6”这不是紧挨着跳转指令的下一条指令地址吗？怎么不是 `printf`！Linux 与 Windows 是不同的，《ELF 规范》把 `0x80482b0` 处的这段东西叫做过程连接表（PLT）又把 `0x8049570` 处的东西称为全局偏移表（GOT）,这两个概念我们不过多引入。请大家注意 `0x080482b6` 开始的那条 `push` 指令，它把 `0x8` 压入堆栈。`0x8` 就我们的关键！我们退出 GDB 使用我们自己前面写下的工具。先打印符号表，由于内容太多我只把动态符号表中的情况摘抄在下面，动态符号表是类型为 `SHT_DYNSYM` 节。

`.dynsym(type=SHT_DYNSYM)`
`offset=0x174,   vaddr=0x8048174,        size=0x60`
`Sym:`
`type=0(STT_NOTYPE)      attrib=0`
`st_shndx=0,     st_value=0x0(0),        st_size=0`
`Sym:__libc_start_main`
`type=2(STT_FUNC)        attrib=1`
`st_shndx=0,     st_value=0x0(0),        st_size=239`
`Sym:printf`
`type=2(STT_FUNC)        attrib=1`
`st_shndx=0,     st_value=0x0(0),        st_size=57`
`Sym:_IO_stdin_used`
`type=1(STT_OBJECT)      attrib=1`
`st_shndx=14,    st_value=0x8048468(134513768),  st_size=4`
`Sym:_Jv_RegisterClasses`
`type=0(STT_NOTYPE)      attrib=2`
`st_shndx=0,     st_value=0x0(0),        st_size=0`
`Sym:__gmon_start__`
`type=0(STT_NOTYPE)      attrib=2`
`st_shndx=0,     st_value=0x0(0),        st_size=0`

可以看到动态符号表共有六项，除第 1 项保留不用，共引入 5 个外部符号，其中两个是函数，`printf` 位于第 3 项。再打印重定位表：



### .rel.dyn 重定位节

```
.rel.dyn(type=SHT_REL)
offset=0x260, vaddr=0x8048260, size=0x8
type=6(R_386_GLOB_DAT) SYM=5 offset=0x804955c
```

### .rel.plt 重定位节

```
.rel.plt(type=SHT_REL)
offset=0x268, vaddr=0x8048268, size=0x10
type=7(R_386_JMP_SLOT) SYM=1 offset=0x804956c
type=7(R_386_JMP_SLOT) SYM=2 offset=0x8049570
```

在重定位表中请注意名为`.rel.plt`的重定位节，这个重定位表中只有两项，它们的重定位类型都是`R_386_JMP_SLOT`，它们与动态符号表中仅有的两个函数一一对应。第一个的符号表索引是`1`，对应着`__libc_start_main`，我们不关心；第二个符号表索引是`2`，恰与`printf`对应，我们看到它的`r_offset`字段的值是`0x8049570`，这个地址正好保存着`printf`要跳转到的地址。重定位信息告诉操作系统要修改这个地方可是系统并没修改，修改任务于是就必须由`0x80482b6`处的指令来完成。`0x8`——这个压入栈中的数值就我们的关键。重定位表中每个结构的大小恰是`8`个字节，于是你大胆猜测这个`0x8`就是外部函数的重定位信息在重定位表中的偏移量。这个猜测可以通过引入多个不同的函数加以验证。`push`指令后的跳转可认为是去调用一个函数，而`push`本身仅是向那个函数传递这个参数罢了，而那个函数一定会找到`printf`并调用它。那么为什么要如此大费周折呢，我想有些人一定猜到了结果：那个用重定位表偏移做参数的函数一定在得到`printf`地址后随后修改了`0x8049570`处保存的值，下一次再调用`printf`时就会直接由`0x80482b0`处跳到真正的函数体内了——怎么有点像Win9x中“VXD CALL”。通过引入多个外部函数我们又发现：多个外部函数在`.rel.plt`中的排列顺序与它们对应内容在GOT中的排列顺序完全一致，不同的是它们不是从GOT的偏移`0`开始的。

继续我们的实验，重新用`GDB`打开`hello`，在`0x804838d`处下一个断点，它将使程序在调用`printf`的语句之后停止不动。然后用`r`命令执行`hello`。

```
(gdb) b *0x804838d
Breakpoint 1 at 0x804838d
(gdb) r
```

经过几行输出，`GDB`最后打印了一条信息`Breakpoint 1, 0x0804838d in main ()`等待我们的命令。我们再来看一下`0x8049570`处的内容，它果然变了，由`0x080482b6`变成了`0x005d62a0`。这恰是`printf`的真正地址，我们还能看到`printf`还调用了`vfprintf`。

```
(gdb) x/4x 0x8049570
0x8049570 <_GLOBAL_OFFSET_TABLE_+16>: 0x005d62a0 0x00000000 0x00000000 0x0804948c
(gdb) disass printf
Dump of assembler code for function printf:
0x005d62a0 <printf+0>: push %ebp
0x005d62a1 <printf+1>: mov %esp,%ebp
0x005d62a3 <printf+3>: sub $0x10,%esp
0x005d62a6 <printf+6>: mov %ebx,0xfffffffc(%ebp)
0x005d62a9 <printf+9>: mov 0x8(%ebp),%edx
0x005d62ac <printf+12>: lea 0xc(%ebp),%ecx
0x005d62af <printf+15>: call 0x5a990d <__i686.get_pc_thunk.bx>
0x005d62b4 <printf+20>: add $0xd5d48,%ebx
0x005d62ba <printf+26>: mov %ecx,0x8(%esp)
0x005d62be <printf+30>: mov 0xfffffe7c(%ebx),%ecx
0x005d62c4 <printf+36>: mov %edx,0x4(%esp)
0x005d62c8 <printf+40>: mov (%ecx),%edx
0x005d62ca <printf+42>: mov %edx,(%esp)
0x005d62cd <printf+45>: call 0x5cd620 <vfprintf>
0x005d62d2 <printf+50>: mov 0xfffffffc(%ebp),%ebx
0x005d62d5 <printf+53>: mov %ebp,%esp
0x005d62d7 <printf+55>: pop %ebp
0x005d62d8 <printf+56>: ret
0x005d62d9 <printf+57>: nop
0x005d62da <printf+58>: nop
0x005d62db <printf+59>: nop
0x005d62dc <printf+60>: nop
0x005d62dd <printf+61>: nop
0x005d62de <printf+62>: nop
0x005d62df <printf+63>: nop
End of assembler dump.
(gdb)
```

实验内容就这么多，更进一步的细节我宁愿当它是一个黑盒，根据参数实现功能。

有一点我需要再三重复。在ELF文件中没有信息把`printf`与`libc.so.6`联系在一起，也就是说加载程序不知道`printf`的函数体在`libc.so.6`中。所以加载程序只能根据`DT_NEEDED`类型的动态结构加载所有程序需要的`so`模块，然后在所有`so`模块中寻找`printf`。这也是符号结构要带有“绑定类型”信息的原因。这样的连接方式有些好处。例如PE中没有使用的“懒模式”，不管程序执行过程中是否会用到，PE文件执行之前它引入的所有外部函数都必须被系统解析出来，而Linux下正如刚才看到的用到时才会解析。“懒模式”有它的危害，如果用到时才加载`so`模块，必然使程序的运行不够流畅，所以Linux一次性加载所有模块而用时解析函数应该是对此问题的解决方案。没有把模块与函数绑定在一起也使Linux的驱动开发者受益，可加载模块能够引用内核符号并能导出符号供其它模块使用。

文章已完，虎头蛇尾。自我感觉最后是机械式的堆砌。有些结论是看来的，有些是实验来的，有些是意断来的。如有不对、不足的地方请大家跟贴补充，算是顶我吧（抛砖引玉的话就不说了）！

`4dd466d0e1e0cc123afbe504da2462ca`
