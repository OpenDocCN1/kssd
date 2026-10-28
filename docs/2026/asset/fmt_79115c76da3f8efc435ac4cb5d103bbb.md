
# [原创]文件句柄分配的原理探测（三）---全面深入

看过我上一篇文章《深入VC流》的朋友想必对VC流系统的认识有了一定基础，文章的着重点在于流类库的两大基类，关于`streambuf`类和`ios`类及其常用的派生类`iostream`,`ostream`,`iostream`从原理和实验两方面已经介绍的差不多了，而对`streambuf`类的派生类以及文件流类`fstream`,`ifstream`,`ofstream`并没有涉及。这篇文章就是在此基础上使对VC流的认识更上一层楼，着重介绍文件流相关的原理并在此基础上通过反汇编逐渐靠近目标--透视文件句柄分配的原理。

在文章的开始，我还是依照自己的惯例介绍一下在调试时会涉及到内容的基本原理。

# 一。原理篇

## 1。`filebuf`类：`streambuf`类的派生类，其结构原理可以参见《深入VC流》这里不再赘述。其本质也是系统在配置缓冲区处理的时候划分一块固定的内存区域--保留区域,只不过是属于文件流专用。

先说明一点：既然“保留区域”是由`streambuf`对象配置的，那么“文件保留区域”就是`filebuf`对象配置的。总之，清楚“保留区域是由其所对应的对象所配置的”就可以了。在这里强调这一点是因为当我调试进行到尾声的时候才发现自己一直把对象与保留区域混为一谈，例如`rdbuf()`函数返回的是对象指针而不是保留区域指针。关于这一点在《深入VC流》中也忽略了。切记！！！

`filebuf`类也有属于自己的成员变量及相关的函数：

1）。`x_fd`：文件描述符，用于标识文件保留区域是否于文件绑定；

2）。`x_fOpened`：

3）。`filebuf()`;构造空的文件保留区域（实际由系统分配）

`filebuf(filedesc fd)`;构造文件保留区域并且绑定到指定的文件

`filebuf(filedesc fd, char* pr, int nLength)`;

构造文件保留区域并与文件绑定，同时初始化保留区域

`~filebuf()`;析构函数

`fd`:文件描述符

`pr`:文件保留区域的指针

`nLength`:文件保留区域的大小

4）。下面是4个文件保护模式

`openprot`：default share/prot mode

`sh_none`: Exclusive mode ― no sharing.

`sh_read` : Read sharing allowed.

`sh_write` : Write sharing allowed.

5）。下面是2个读取文件的模式

`binary`:二进制模式

`text`:文本模式

6）。`filebuf* attach(filedesc fd)`；

绑定文件保留区域到由文件描述符指定的文件

7）。`filedesc fd()const { return (x_fd==-1)?EOF:x_fd; }`

取得文件描述符，若保留区域对象没有与文件绑定则返回`EOF`，本质就是查看标识`x_fd`;

8）。`int is_open() const { return (x_fd!=-1); }`

用于检测文件保留区域是否与文件绑定

9）。`filebuf* open(const char* szName, int nMode, int nProt = filebuf::openprot)`；

打开磁盘文件且绑定文件保留区域

10）。`filebuf* close()`;刷新文件输出缓冲区（继承`streambuf`特性，保留区域由输入缓冲区和输出缓冲区两部分，但不一定同时存在），关闭打开的文件并且撤消保留区域与文件之间的绑定关系（回收系统资源）。

11）。`int setmode(int nMode = filebuf::text)`;设置读取文件模式

到此`filebuf`类的成员基本到齐，剩下的是几个需函数，这里就不介绍了。

感觉上这个类的内容很少，你可以任由这种感觉继续，但必须搞清楚几个问题：

1）。保留区域需要绑定到具体的流对象才能正真起作用

2）。`filedesc`:a typedef equivalent to int

相当于`typedef int filedesc;`3）。

3）。文件描述是什么，为什么能代表文件？

4）。既然文件描述符能代表文件，它和文件之间必然存在一一对应的关系，这不是文件句柄的特性。那么文件描述符和句柄到底是什么关系，或者它们根本就是一回事？

5）。文件描述符是在哪里被设置即`x_fd`是在哪里被赋值？观察上面介绍的成员函数只有两种可能性：构造函数和`open`函数；其它如`is_open`和`fd`都只是根据标识`x_fd`来进行判断并没有改便它的值。

又到了从实验种找答案的时刻，带着这些问题去实验，我相信最终会成功。

## 2。`ifstream`,`ofstream`,`fstream`有一些同名又同功能的函数：

`filebuf* rdbuf() const;`返回与当前流对象相关的`filebuf` buffer object的指针

`filedesc fd() const;`

`void attach(filedesc fd);`

`int is_open() const;`

`int setmode(int nMode = filebuf::text);`

`void open(const char* szName, int nMode, int nProt = filebuf::openprot);`

功能与`filebuf`类的成员函数类同，不再赘述。

# 二。实验篇

调试环境：`VC6.0`调试器（也可以结合`OD`，它的对代码分析模块化的功能可以大大简化代码分析时的工作）

这里听取`nbw`巴神老哥的建议从反汇编的角度来阐述问题。

```c
#include "stdafx.h"
#include "fstream.h"

int main(int argc, char* argv[])
{
00401020 push ebp
00401021 mov ebp,esp
00401023 push 0FFh
00401025 push offset __ehhandler$_main (004163e9)
0040102A mov eax,fs:[00000000]
00401030 push eax
00401031 mov dword ptr fs:[0],esp
00401038 sub esp,84h
0040103E push ebx
0040103F push esi
00401040 push edi
00401041 lea edi,[ebp-90h]
00401047 mov ecx,21h
0040104C mov eax,0CCCCCCCCh
00401051 rep stos dword ptr [edi]
 ifstream fin;
00401053 push 1
00401055 lea ecx,[ebp-4Ch]
00401058 call ifstream::ifstream (0040c820) //初始化文件流对象，指针为00921D00
0040105D mov dword ptr [ebp-4],0
 fin.open("test.txt");
00401064 mov eax,[filebuf::openprot (00428fe4)]
00401069 push eax //nProt = filebuf::openprot
0040106A push 1 //nMode=1 ( in )
0040106C push offset string "test.txt" (0042801c) //szName="test.txt"
00401071 lea ecx,[ebp-4Ch]
00401074 call ifstream::open (0040ce60)
 fin.close();
00401079 lea ecx,[ebp-4Ch]
0040107C call ifstream::close (0040cec0)
 return 0;
00401081 mov dword ptr [ebp-50h],0
00401088 mov dword ptr [ebp-4],0FFFFFFFFh
0040108F lea ecx,[ebp-4Ch]
00401092 call @ILT+0(ifstream::`vbase destructor') (00401005)
00401097 mov eax,dword ptr [ebp-50h]
}
0040109A mov ecx,dword ptr [ebp-0Ch]
0040109D mov dword ptr fs:[0],ecx
004010A4 pop edi
004010A5 pop esi
004010A6 pop ebx
004010A7 add esp,90h
004010AD cmp ebp,esp
004010AF call __chkesp (00401520)
004010B4 mov esp,ebp
004010B6 pop ebp
004010B7 ret
```

```
//来自00401074 call ifstream::open (0040ce60)的调用
ifstream::open:
0040CE60 push ebp
0040CE61 mov ebp,esp
0040CE63 push ecx
0040CE64 mov dword ptr [ebp-4],ecx
0040CE67 mov ecx,dword ptr [this]
0040CE6A call ifstream::is_open (0040cdd0) //检测文件保留区域是否于文件绑定
0040CE6F test eax,eax //在这个时候还没有绑定
0040CE71 jne ifstream::open+35h (0040ce95) //不跳转，执行绑定
0040CE73 mov eax,dword ptr [prot]
0040CE76 push eax
0040CE77 mov ecx,dword ptr [mode]
0040CE7A or ecx,1
0040CE7D push ecx
0040CE7E mov edx,dword ptr [name]
0040CE81 push edx //open的三个参数入栈
0040CE82 mov ecx,dword ptr [this]
0040CE85 call ifstream::rdbuf (0040cac0) //取得filebuf对象的指针
0040CE8A mov ecx,eax //此处是EAX=00921D00
0040CE8C call filebuf::open (0040e9c0) //打开磁盘文件并且绑定文件保留区域
0040CE91 test eax,eax //返回值是filebuf对象的指针00921D00
0040CE93 jne ifstream::open+58h (0040ceb8) //跳转
0040CE95 mov eax,dword ptr [this]
0040CE98 mov ecx,dword ptr [eax]
0040CE9A mov edx,dword ptr [ecx+4]
0040CE9D mov eax,dword ptr [this]
0040CEA0 mov ecx,dword ptr [eax+edx+8]
0040CEA4 or ecx,2
0040CEA7 push ecx
0040CEA8 mov edx,dword ptr [this]
0040CEAB mov eax,dword ptr [edx]
0040CEAD mov ecx,dword ptr [this]
0040CEB0 add ecx,dword ptr [eax+4]
0040CEB3 call ios::clear (0040cd80)
0040CEB8 mov esp,ebp
0040CEBA pop ebp
0040CEBB ret 0Ch
```

调试到这里，对反汇编代码以及原理篇中所讲述的文件流相关函数有了更清晰的了解，但至于在实验前所提出的问题并没有涉及，甚至连“文件描述符”也没出现。

显然需要对`filebuf::open`进行深入的调试，根据原理篇的介绍`open`函数的功能是“打开磁盘文件且绑定文件保留区域”，同时考虑到第五个问题“文件描述符是在哪里被设置？”此处是用`open`函数来执行打开函数功能的，因而设置“文件描述符”的位置必定在这个函数中。

这里从逻辑上坚定了调试的信心，经历过漫长调试的朋友都清楚在进入一个函数之前必定要对它的基本功能做一个全局的分析，不论分析的结果如何，但都已经在无意识间把这个函数放到程序这个大局中，而不仅仅局限于代码，这样会事半功倍。

重新调试，来到`open`函数。

`filebuf::open`:
```
0040E9C0 push ebp
0040E9C1 mov ebp,esp
0040E9C3 sub esp,18h
0040E9C6 mov dword ptr [ebp-14h],ecx
0040E9C9 mov eax,dword ptr [this]
0040E9CC cmp dword ptr [eax+30h],0FFh //检测文件是否已经打开
0040E9D0 je filebuf::open+19h (0040e9d9) //跳转
0040E9D2 xor eax,eax
0040E9D4 jmp filebuf::open+256h (0040ec16)
0040E9D9 mov ecx,dword ptr [mode] //ecx=1 ( nMode=in )
0040E9DC and ecx,80h //判断nMode?=binary;
0040E9E2 neg ecx
0040E9E4 sbb ecx,ecx
0040E9E6 and ecx,4000h
0040E9EC add ecx,4000h //状态标志字“stdio”
0040E9F2 mov dword ptr [dos_mode],ecx //保存状态标志字
0040E9F5 mov edx,dword ptr [mode] //edx=1 ( nMode=in )
0040E9F8 and edx,20h //判断nMode?=nocreate
0040E9FB test edx,edx
0040E9FD jne filebuf::open+48h (0040ea08) //不跳转
```
