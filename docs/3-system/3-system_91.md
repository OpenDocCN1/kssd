

# PE 文件格式启发式学习 7-资源文件

> 原文：[`www.pediy.com/kssd/pediy10/65998.html`](https://www.pediy.com/kssd/pediy10/65998.html)

这一次，我们来谈谈 PE 中的资源，在 PE 文件格式启发式学习 4 中提到过资源感念，被匆匆掠过。于心不甘。  
现在来补齐。例子还以原来的 hello.exe 为例吧。一滴水也能照出太阳的光辉。  
  
问 6.1：资源，这个名字听得挺陌生的，资源到底是什么呢？  
答 6.1：在 dos 时代，是没有资源这个名字的， 到了 windows 时期，出现了资源的概念。  
资源其本质只是一堆只读性数据。随着程序越来越大，这些只读数据从数据区中被抽取出来，放在一起  
最后由链接程序把这些数据连到运行文件中。  
  
问 6.2： 不是有只读数据区吗，为什么还要加一个资源节呢。  
答 6.2：本来它们也是可以合并的，但这样区分也自有它的妙处。例如只读数据往往直接为代码服务。而资源  
往往是一些可视化数据，例如鼠标，光标，菜单的定义等。资源被有效的组织起来，有专门的 API 来访问资源数据。  
  
问 6.3： 资源都包括什么内容呢？  
答 6.3：其实你可以将任何只读数据存储到资源中。可以把资源看做一个文件，资源文件最后被链接到 exe 文件中。  
    常见的资源包括光标，鼠标外形，bitmap,菜单，加速键，字符串，对话框等，还有你自己定义的数据。  
      
问 6.4：这么多东西，我们怎么区分它们呢？  
答 6.4：很简单啦，用一个资源 ID 就做到了。  
  
RT\_CURSOR                            equ 1  
RT\_BITMAP                            equ 2  
RT\_ICON                              equ 3  
RT\_MENU                              equ 4  
RT\_DIALOG                            equ 5  
RT\_STRING                            equ 6  
RT\_FONTDIR                           equ 7  
RT\_FONT                              equ 8  
RT\_ACCELERATOR                       equ 9  
RT\_RCDATA                            equ 10  
  
RT\_VERSION         equ 16  
RT\_DLGINCLUDE         equ 17  
RT\_PLUGPLAY         equ 19  
RT\_VXD           equ 20  
RT\_GROUP\_CURSOR       equ 12  (RT\_CURSOR + DIFFERENCE)(DIFFERENCE=11)  
RT\_GROUP\_ICON         equ 14  (RT\_ICON + DIFFERENCE)  
RT\_ANICURSOR         equ 21  
RT\_ANIICON         equ 22  
RT\_HTML         equ 23  
RT\_MANIFEST                        equ 24  
上面是内定的资源 ID,别管太多，先从几个常用的开始，例如 icon,cursor。  
我们还可以自定义 ID, 或者用名字来区分不同的资源。  
  
问 6.5：哦想起来了，好像 PE 资源是按树形方式进行组织的，就象目录结构一样，采用分级的方法。为什么要分  
那么多级，用一级不可以吗？例如，将所有的资源数据组织在一起，前面用一个指针数组分别指向不同的资源  
数据，ICON,CURSOR,Dialog....  
答 6.5：您说得可能是简单的情况，例如只有一个 ICON, 一个 CURSOR,一个 Dialog。。。但假如我有多个 个 ICON, 多个  
Cursor 呢 ？  
学生答： 那我可以让原来指向 ICON 的指针指向一个 ICON 结构，该结构说明有几个 ICON,然后紧跟 ICON 数组。  
老师答： 好！这就是分级的概念。因为一层结构表达全部意思有些困难，所以我们要把它分层表达。  
  分级管理是管理之道。道是什么 ？“老子曰“，道比天大。没有天之前，道就形成了。天可灭，道不可灭。  
    
问 6.6：哪有那么神呢？ 不就是分级管理吗。  
答 6.6： 任何复杂事物都需要分级管理。不只是 PE 的资源是这样，其它事物也是这样。给你举个例子吧，胡主席要给你  
    一封信，可是他不认识你，怎么办呢。他认识各个省长，他把这份信交给了你们省长，省长交给县长，县长交给乡长，  
    乡长交给你们村长，村长就找到你了。把信交给了你。你让胡主席直接把信交给你，那相当于他直接管理 10 亿人，![](img/f43c1e514327696dccbb0da997496b2a.jpg "扮鬼脸")   
    哪能管过来呀！  
      
问 6.7：！！！？？？![](img/1e3c9433a1c444ac212c55a1ce70452a.jpg "狂怒")  ，还是直接看看资源是怎样管理的吧。  
答 6.7：还是以原来的 hello.exe 为例吧。 从目录项中找到资源表。  
00000A00   00 00 00 00 00 00 00 00  00 00 00 00 00 00 02 00   ................  
00000A10   03 00 00 00 20 00 00 80  0E 00 00 00 38 00 00 80   .... ......8..  
00000A20   00 00 00 00 00 00 00 00  00 00 00 00 00 00 01 00   ................  
00000A30   01 00 00 00 50 00 00 80  00 00 00 00 00 00 00 00   ....P..........  
00000A40   00 00 00 00 00 00 01 00  01 00 00 00 68 00 00 80   ............h..  
00000A50   00 00 00 00 00 00 00 00  00 00 00 00 00 00 01 00   ................  
00000A60   09 04 00 00 80 00 00 00  00 00 00 00 00 00 00 00   ...............  
00000A70   00 00 00 00 00 00 01 00  09 04 00 00 90 00 00 00   ............?..  
00000A80   A0 40 00 00 E8 02 00 00  00 00 00 00 00 00 00 00   ..?..........  
00000A90   88 43 00 00 14 00 00 00  00 00 00 00 00 00 00 00   ..............  
00000AA0   28 00 00 00 20 00 00 00  40 00 00 00 01 00 04 00   (... ...@.......  
00000AB0   00 00 00 00 00 02 00 00  00 00 00 00 00 00 00 00   ................  
00000AC0   00 00 00 00 00 00 00 00  00 00 00 00 00 00 80 00   ...............  
00000AD0   00 80 00 00 00 80 80 00  80 00 00 00 80 00 80 00   ..........  
00000AE0   80 80 00 00 80 80 80 00  C0 C0 C0 00 00 00 FF 00   ...览?...  
00000AF0   00 FF 00 00 00 FF FF 00  FF 00 00 00 FF 00 FF 00   ..........  
      
资源表的第一个数据结构叫资源目录，如下定义  
typedef struct \_IMAGE\_RESOURCE\_DIRECTORY {  
    DWORD   Characteristics;  
    DWORD   TimeDateStamp;  
    WORD    MajorVersion;  
    WORD    MinorVersion;  
    WORD    NumberOfNamedEntries;  
    WORD    NumberOfIdEntries;  
//  IMAGE\_RESOURCE\_DIRECTORY\_ENTRY DirectoryEntries\[\];  
} IMAGE\_RESOURCE\_DIRECTORY, \*PIMAGE\_RESOURCE\_DIRECTORY;  
从例子中可以看出，属性，时戳全为零，它们没有使用  
主版本号，次版本号也没有使用，再去 4 个字节，这样前 12 个字节没有使用。微软挺大方的，12 个字节  
都空着不用，这个结构的定义是微软官方定义，要我说，还不如简单点，这么定义：Byte NoUse\[12\].  
有用的只有两项就是资源项个数。  
  
NumberOfNamedEntries 为零，说明没有用名字定义资源。  
NumberOfIdEntries=2， 说明有 2 项以 ID 定义的资源。  
  
紧跟其后的就是资源目录项：  
typedef struct \_IMAGE\_RESOURCE\_DIRECTORY\_ENTRY {  
    union {  
        struct {  
            DWORD NameOffset:31;  
            DWORD NameIsString:1;  
        };  
        DWORD   Name;  
        WORD    Id;  
    };  
    union {  
        DWORD   OffsetToData;  
        struct {  
            DWORD   OffsetToDirectory:31;  
            DWORD   DataIsDirectory:1;  
        };  
    };  
} IMAGE\_RESOURCE\_DIRECTORY\_ENTRY, \*PIMAGE\_RESOURCE\_DIRECTORY\_ENTRY;  
#define IMAGE\_RESOURCE\_NAME\_IS\_STRING        0x80000000  
#define IMAGE\_RESOURCE\_DATA\_IS\_DIRECTORY     0x80000000  
  
这个结构怎么看着这么别扭呢，没关系，解释一下实例就清楚了,这个结构，其实包含着 8 个数据，  
只是根据数据的表现。会有不同的解释。  
  
03 00 00 00，因为最高位不为 1，我们取低 2bytes, WORD ID=03 为 ICON ID.    
20 00 00 80  因为最高位为 1,OffsetToDirectory = 0x20.  
不是说了 ID 表示的目录有 2 个吗，继续分析第二个。（相当于根目录下的第二个目录）。  
0E 00 00 00，我们取低 2bytes, ID=0E 为 ICON group ID.    
38 00 00 80  OffsetToDirectory=0x38; 最高位的 1 表示数据指向是一个目录。         
  
问：根目录下包含两个 ID 目录，它们怎样找到后面的数据呢？  
答：这个 OffsetToDirectory 却是一个从 rsrc 资源表算起的偏移。所以 0x20== offset 0xA20  
    我们 copy 下该数据：  
00000A20   00 00 00 00 00 00 00 00  00 00 00 00 00 00 01 00   ................ //资源目录个数  
这个结构我们已经不陌生了，就是那个 Byte NoUse\[12\],后面 4byte 说明用名字命名的目录数为 0， 以 ID 命名的目录数为 1 个。  
再看看这个目录的内容：  
00000A30   01 00 00 00 50 00 00 80              ....P.......... //资源目录项指向目录  
01 00 00 00，我们取低 2bytes, ID=01 这里的 01 并不代表 Cursor, 因为它在第二层，它代表 ICON 的 ID 号为 1  
50 00 00 80  OffsetToDirectory=0x50; 最高位的 1 表示数据指向是一个目录  
  
问 6.8：既然还没有取到数据，那继续往下跟吧，看起来分级管理还挺浪费空间的。  
答 6.8：主要是我们举的例子太简单了。你想用一个能管理 1 万人的团队，来管理 100 人，你肯定觉的管理人员太多。  
    何况微软在资源管理上，正好又有 12 个字节未用。不过你可以正好把它当成一个标志。  
    好，看 OffsetToDirectory 50 = offset a50  
00000A50   00 00 00 00 00 00 00 00  00 00 00 00 00 00 01 00   ................  //资源目录个数为 1  
00000A60   09 04 00 00 80 00 00 00            //资源目录项指向数据  
资源目录说有一个 ID 命名资源项， name/ID=0409, OffsetToData=0x80.这一次直接指向数据了，而不再是目录。  
因为后一个 DWORD 最高位为 0.   这里的 ID 0409 是代码页，表示语言设置为美式英语。 而设为 0936，则是 gb2312 了。  
  
现在来看数据的格式，这个结构给出了数据 RVA 和 size  
00000A80   A0 40 00 00 E8 02 00 00  00 00 00 00 00 00 00 00   ..?..........  //资源数据节点。  
typedef struct \_IMAGE\_RESOURCE\_DATA\_ENTRY {  
    DWORD   OffsetToData;  
    DWORD   Size;  
    DWORD   CodePage;  
    DWORD   Reserved;  
} IMAGE\_RESOURCE\_DATA\_ENTRY, \*PIMAGE\_RESOURCE\_DATA\_ENTRY;  
  
A0 40 00 00 OffsetToData=0x40a0  （RVA)  
E8 02 00 00 size = 0x02e8  
CodePage 第三级目录已经设过了，这里又出一个，没有用了，所以通常都是 0.  
reserved 保留，未使用，总为 0.  
  
这一次，数据指针是个 RVA,怎么搞的，微软怎么前面用偏移，最后又用 RVA 呢？没办法。  
只能听它的了，我还是喜欢偏移，方便阅读啊。值得庆幸的，终于着陆了，就好比老胡的信终于传到你手上了。  
另外一个 ICON\_GROUP 与此同，我就不分析了。  
如果你觉得不过瘾的话，你可以找一个复杂点的分析一下，其框架是一样的。  
  
最后看一下找到的数据 RVA 40a0 = offset AA0  
00000AA0   28 00 00 00 20 00 00 00  40 00 00 00 01 00 04 00   (... ...@.......  
00000AB0   00 00 00 00 00 02 00 00  00 00 00 00 00 00 00 00   ................  
00000AC0   00 00 00 00 00 00 00 00  00 00 00 00 00 00 80 00   ...............  
00000AD0   00 80 00 00 00 80 80 00  80 00 00 00 80 00 80 00   ..........  
这是一个 bitmap 头信息。其结构是如下定义的，  
typedef struct tagBITMAPINFOHEADER{  
        DWORD      biSize;  
        LONG       biWidth;  
        LONG       biHeight;  
        WORD       biPlanes;  
        WORD       biBitCount;  
        DWORD      biCompression;  
        DWORD      biSizeImage;  
        LONG       biXPelsPerMeter;  
        LONG       biYPelsPerMeter;  
        DWORD      biClrUsed;  
        DWORD      biClrImportant;  
} BITMAPINFOHEADER  
从 AA0 算起，去掉 0x28 个头信息，后面的就是真实的 bitmap 颜色数据了。  
  
最后总结一下该贴的主题：  
资源管理，三级目录结构，最后的数据节点，分别指向相应的数据。  
  
题外话：  
做了 N 年的程序，终于悟出来一个道理，这也是计算机中的”大道“。  
计算机是用来处理数据的。  
数据分两种，一种是指令，一种是数据 （此处的数据是...真数据）  
数据分两种，一种是地址，一种是数据  (此处的数据可理解...真真数据）  
数据分两种，一种是描述，一种是数据  (此处的数据可理解为...真真真数据)  
我本来想为每种数据起个名字，但我起不到合适的名字。奇妙的是计算机中的 0，1 二叉数  
又应验在数据的不断的分裂中。所以，我为数据起个名字，你每分裂一次，我前边就加一个真字，哈哈！![](img/0d3dbf5fb8604477afb4a1766fe8c72d.jpg "尴尬")   
凡事皆有阴阳，阴中有阳，阳中有阴，其大无外，其小无内。数据为阳，指令为阴。  
不管你研究的技术多么高深，什么影子，沙箱，rootkit, hack,crack, 我都不懂，但我知道，它们或者是数据，或者是想处理数据。  
怎么样？计算机技术就是我们中华文化阴阳理论的继承和实践![](img/f43c1e514327696dccbb0da997496b2a.jpg "扮鬼脸") 。  
  
  write by hjjdebug(june/2008)