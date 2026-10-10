

# 【部分原创】 CRT 启动函数学习笔记

> 原文：[`www.pediy.com/kssd/pediy10/89536.html`](https://www.pediy.com/kssd/pediy10/89536.html)

CRT 运行库启动函数分析：  
在进入 main 函数之前，系统会调用 CRT 运行库的启动函数，做如下工作：  
全局变量已完成初始化，  
堆的初始化，  
I/O 也完成了初始化，  
Main 调用  
  
1. \*\*\*\*\*CRTStartUp()的框架：  
  
\*\*\*\*\*\*CRTStartUp()  
{  
  /\*初始化一些操作系统版本的全局变量\*/  
  \_osver   =   GetVersion();     
      
  \_winminor   =   (\_osver   >>   8)   &   0x00FF   ;     
  \_winmajor   =   \_osver   &   0x00FF   ;     
  \_winver     =   (\_winmajor   <<   8)   +   \_winminor;     
  \_osver      =   (\_osver   >>   16)   &   0x00FFFF   ;     
  
  /\*初始化堆\*/  
  if   (   !\_heap\_init(1)   )  
      ……………..  
  
/\*初始化 I/O ，这样在 main 函数中才能直接使用 printf 之类的函数，使用 windows 的 SHE 机制\*/  
  
try {  
  \_ioinit();    
  
}\_\_except (\_XcptFilter(GetExceptionCode(),   GetExceptionInformation()) ){  
    \_exit(  GetExceptionCode() );  
}  
  
/\*取得命令行参数\*/  
\_wcmdln   =   (wchar\_t   \*)\_\_crtGetCommandLineW();      
\_wenvptr   =   (wchar\_t   \*)\_\_crtGetEnvironmentStringsW();    
  
/\*初始化 main 函数的 argv 参数\*/   
\_wsetargv();     
/\*初始化环境变量\*/  
\_wsetenvp();     
  
/\*初始化一些 C 数据,进行 C 库设置\*/  
\_cinit();  
  
/\*调用 main 函数\*/  
mainret   =   main(\_\_argc,   \_\_argv,   \_environ);     
  
/\*等待 main 函数返回，然后退出进程\*/  
  exit(mainret);     
}  
\--------------------------------------------------------------------------------------------------------------------  
下面逐一分析各个阶段：  
2. 初始化堆  
调用的是\_heap\_init, 分析该函数：  
int \_\_cdecl \_heap\_init (   int mtflag  /\*多线程标志\*/  )  
{  
        /\*调用 HeapCreate 创见进程堆，\*/  
        if ( (\_crtheap = HeapCreate( mtflag ? 0 : HEAP\_NO\_SERIALIZE,  
                                     BYTES\_PER\_PAGE, 0 )) == NULL )  
            return 0;  
  
        // Pick a heap, any heap  
        \_\_active\_heap = \_\_heap\_select();  
  
        return 1;  
}  
  
初始化堆是非常紧急的事情，否则其他的很多事情都做不了，如果堆初始化失败，那么进程就直接退出了。  
  
**\--------------------------------------下面的就是转载的  程序员的修养   
  懒得写了 --------------------------------------------------------------------------------**  
  
3. I/O 的初始化  
首先 I/O 初始化函数需要在用户空间中建立 stdin、stdout、stderr 及其对应的 FILE 结构，使得程序进入 main 之后可以直接使用 printf、scanf 等函数。（其实 printf 和 scanf 操作的是 FILE 结构。）  
在 linux 中 stdin stdout stderr 的 fd 分别为 1，2，3 进程打开的文件 fd 从 4 开始。  
MSVC 中 FILE 文件结构：  
struct \_iobuf {  
    char \*\_ptr;  
    int   \_cnt;  
    char \*\_base;  
    int   \_flag;  
    int   \_file;           //通过该变量得到内部二维句柄表数组的两个下标，从而找到句柄   
    int   \_charbuf;  
    int   \_bufsiz;  
    char \*\_tmpfname;  
    };  
typedef struct \_iobuf FILE;  
  
  
在 MSVC 的 CRT 中，已经打开的文件句柄的信息使用数据结构 ioinfo 来表示：  
typedef struct {  
    intptr\_t osfhnd;    //文件句柄  
    char osfile;  
    char pipech;  
}   ioinfo;  
  
在 crt/src/ioinit.c 中，有一个数组：  
int \_nhandle;  
ioinfo \* \_\_pioinfo\[64\]; // 等效于 ioinfo \_\_pioinfo\[64\]\[32\];  
这就是每个进程用户态的打开文件表  
  
通过 FILE 结构的 \_file 计算出二维数组下标，然后 取得 osfhnd 便可以得到句柄。  
  
计算二维数组下标的宏（CRT 内部使用）:  
#define \_osfhnd(i)  ( \_pioinfo(i)->osfhnd )  
其中宏函数\_pioinfo 的定义是：  
#define \_pioinfo(i) ( \_\_pioinfo\[(i) >> 5\] + ((i) & ((1 << 5) -  1)) )  
FILE：\_file 的第 5 位到第 10 位是第一维坐标（共 6 位），\_file 的第 0 位到第 4 位是第二维坐标（共 5 位）。  
  
  
  
MSVC 的 I/O 初始化就是要构造这个二维的打开文件表。MSVC 的 I/O 初始化函数\_ioinit 定义于 crt/src/ioinit.c 中。首先，\_ioinit 函数初始化了\_\_pioinfo 数组的第一个二级数组：  
mainCRTStartup -> \_ioinit()：  
if ( (pio = \_malloc\_crt( 32 \* sizeof(ioinfo) ))   //现在可以从对堆中分配内存了  
             == NULL )  
{  
    return -1;  
}  
\_\_pioinfo\[0\] = pio;  
\_nhandle = 32;  
for ( ; pio < \_\_pioinfo\[0\] + 32 ; pio++ ) {  
    pio->osfile = 0;  
    pio->osfhnd = (intptr\_t)INVALID\_HANDLE\_VALUE;  
    pio->pipech = 10;  
}  
在这里\_ioinit 初始化了的\_\_pioinfo\[0\]里的每一个元素为无效值，其中 INVALID\_ HANDLE\_VALUE 是 Windows 句柄的无效值，值为-1。接下来，\_ioinit 的工作是将一些预定义的打开文件给初始化，这包括两部分：  
(1)   从父进程继承的打开文件句柄，当一个进程调用 API 创建新进程的时候，可以选择继承自己的打开文件句柄，如果继承，子进程可以直接使用父进程的打开文件句柄。  
(2)   操作系统提供的标准输入输出。  
应用程序可以使用 API GetStartupInfo 来获取继承的打开文件，GetStartupInfo 的参数如下：  
void GetStartupInfo(STARTUPINFO\* lpStartupInfo);  
STARTUPINFO 是一个结构，调用 GetStartupInfo 之后，该结构就会被写入各种进程启动相关的数据。在该结构中，有两个保留字段为：  
typedef struct \_STARTUPINFO {  
    ……  
    WORD cbReserved2;  
    LPBYTE lpReserved2;  
    ……  
} STARTUPINFO;  
这两个字段的用途没有正式的文档说明，但实际是用来传递继承的打开文件句柄。当这两个字段的值都不为 0 时，说明父进程遗传了一些打开文件句柄。操作系统是如何使用这两个字段传递句柄的呢？首先 lpReserved2 字段实际是一个指针，指向一块内存，这块内存的结构如下：  
l           字节\[0,3\]：传递句柄的数量 n。  
l           字节\[4, 3+n\]：每一个句柄的属性（各 1 字节，表明句柄的属性，同 ioinfo 结构的\_osfile 字段）。  
l           字节\[4+n 之后\]：每一个句柄的值（n 个 intptr\_t 类型数据，同 ioinfo 结构的\_osfhnd 字段）。  
\_ioinit 函数使用如下代码获取各个句柄的数据：  
cfi\_len = \*(\_\_unaligned int \*)(StartupInfo.lpReserved2);  
posfile = (char \*)(StartupInfo.lpReserved2) + sizeof( int );  
posfhnd = (\_\_unaligned intptr\_t \*)(posfile + cfi\_len);  
其中\_\_unaligned 关键字告诉编译器该指针可能指向一个没有进行数据对齐的地址，编译器会插入一些代码来避免发生数据未对齐而产生的错误。这段代码执行之后，lpReserved2 指向的数据结构会被两个指针分别指向其中的两个数组，如图 11-6 所示。  
图 11-6  句柄属性数组和句柄数组  
接下来\_ioinit 就要将这些数据填入自己的打开文件表中。当然，首先要判断直接的打开文件表是否足以容纳所有的句柄：  
cfi\_len = \_\_min( cfi\_len, 32 \* 64 );  
然后要给打开文件表分配足够的空间以容纳所有的句柄：  
for ( i = 1 ; \_nhandle < cfi\_len ; i++ ) {  
    if ( (pio = \_malloc\_crt( 32 \* sizeof(ioinfo) )) == NULL )  
    {  
        cfi\_len = \_nhandle;  
        break;  
    }  
    \_\_pioinfo\[i\] = pio;  
    \_nhandle += 32;  
    for ( ; pio < \_\_pioinfo\[i\] + 32 ; pio++ ) {  
        pio->osfile = 0;  
        pio->osfhnd = (intptr\_t)INVALID\_HANDLE\_VALUE;  
        pio->pipech = 10;  
    }  
}  
在这里，nhandle 总是等于已经分配的元素数量，因此只需要每次分配一个第二维的数组，直到 nhandle 大于 cfi\_len 即可。由于\_\_pioinfo\[0\]已经预先分配了，因此直接从\_\_pioinfo\[1\]开始分配即可。分配了空间之后，将数据填入就很容易了：  
for ( fh = 0 ; fh < cfi\_len ; fh++, posfile++, posfhnd++ )   
{  
    if ( (\*posfhnd != (intptr\_t)INVALID\_HANDLE\_VALUE) &&  
               (\*posfile & FOPEN) &&  
               ((\*posfile & FPIPE) ||  
               (GetFileType( (HANDLE)\*posfhnd ) !=   
            FILE\_TYPE\_UNKNOWN)) )  
    {  
        pio = \_pioinfo( fh );  
        pio->osfhnd = \*posfhnd;  
        pio->osfile = \*posfile;  
    }  
}  
在这个循环中，fh 从 0 开始递增，每次通过\_pioinfo 宏来转换为打开文件表中连续的对应元素，而 posfile 和 posfhnd 则依次递增以遍历每一个句柄的数据。在复制的过程中，一些不符合条件的句柄会被过滤掉，例如无效的句柄，或者不属于打开文件及管道的句柄，或者未知类型的句柄。  
这段代码执行完成之后，继承来的句柄就全部复制完毕。接下来还须要初始化标准输入输出。当继承句柄的时候，有可能标准输入输出（fh=0,1,2）已经被继承了，因此在初始化前首先要先检验这一点，代码如下：  
for ( fh = 0 ; fh < 3 ; fh++ )   
{  
    pio = \_\_pioinfo\[0\] + fh;  
    if ( pio->osfhnd == (intptr\_t)INVALID\_HANDLE\_VALUE )   
    {  
        pio->osfile = (char)(FOPEN | FTEXT);  
        if ( ((stdfh = (intptr\_t)GetStdHandle( stdhndl(fh) ))  
                != (intptr\_t)INVALID\_HANDLE\_VALUE)   
                && ((htype =GetFileType( (HANDLE)stdfh ))   
                != FILE\_TYPE\_UNKNOWN) )  
        {  
            pio->osfhnd = stdfh;  
            if ( (htype & 0xFF) == FILE\_TYPE\_CHAR )  
                pio->osfile |= FDEV;  
            else if ( (htype & 0xFF) == FILE\_TYPE\_PIPE )  
                pio->osfile |= FPIPE;  
        }  
        else {  
            pio->osfile |= FDEV;  
        }  
    }  
    else  {  
        pio->osfile |= FTEXT;  
    }  
}  
如果序号为 0、1、2 的句柄是无效的（没有继承自父进程），那么\_ioinit 会使用 GetStdHandle 函数获取默认的标准输入输出句柄。此外，\_ioinit 还会使用 GetFileType 来获取该默认句柄的类型，给\_osfile 设置对应的值。  
在处理完标准数据输出的句柄之后，I/O 初始化工作就完成了。我们可以看到，MSVC 的 I/O 初始化主要进行了如下几个工作：  
l           建立打开文件表。  
l           如果能够继承自父进程，那么从父进程获取继承的句柄。  
l           初始化标准输入输出。  
在 I/O 初始化完成之后，所有的 I/O 函数就都可以自由使用了