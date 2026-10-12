
# <<剑走偏峰来自一个中国人的 C++教程>>系列四

> 原文：[`www.pediy.com/kssd/pediy11/109861.html`](https://www.pediy.com/kssd/pediy11/109861.html)

## C++的导出数据类型与结构化程序设计

热烈庆祝本群 90569473（编程<->逆向群）人数组成水浒英雄队伍，欢迎更多的朋友加入该群，讨论思想，分享自由与快乐！

### 目录

一：导出数据类型与函数递归

1.1:导出数据类型

1.2:函数与递归

二：形形色色的指针

2.1:指针基本知识

2.2:指向指针的指针

2.3:`const`、指针、引用

2.4:（略）`void` 指针与 `this` 指针

2.5:指针与数组

2.6:函数与指针

2.7:函数指针数组

三：函数

3.1:函数重载与函数模板

3.2：参数传递

3.3：反汇编下的函数

接下来继续讨论 C++ 的基础知识导出数据类型与结构化程序设计！计算机之所以称为电脑是因为它的作用是在模拟人脑的功能，所以作为编程爱好者来说，个人觉得必须有一个计算机 `CPU` 式的人脑，还要让计算机具备一个人脑式的 `CPU`，前者是对计算机系统底层知识的学习，后者则是人工智能的内容！

## 一：导出数据类型与递归

### 1.1:导出数据类型

在 `N` 年前我们的祖先解决了为什么学习的问题，即学习是为增加人脑的判断力，推而广之，计算机要学习的知识是什么？答程序，程序的作用是什么？答让电脑有自己的判断能力。人脑的判断能力可以狭隘的称为推理能力（事实上世间任何事物都有其自成系统的推理能力，比如树会通过推理找到有水的地方，老马能在人类迷路的时候用于识途）。与广义的推理相比，数学中的推理是一种更加规则性的有前提得到结论中的过程，正如上面所说的，程序的作用是为了让电脑有判断力，那我们就举一个看上去很傻的例子：

命题：向计算机中的内存中保存一个整型数 2,并判断其值是否为 0

C++对命题进行符号化：`if(int a=2)`

计算机的判断过程：

设 `a` 对应的内存为 `ebp-4`,那么 `IA32 CPU` 会做如下判断:

`Mov dword ptr ss:[ebp-4],2`

`Mov eax,dword ptr ss:[ebp-4]`

`Cmp eax,0`

结论：`ZF=0`，即 `a!=0`

上面的过程是典型的数理逻辑的推理过程，在 C++ 中基本控制结构实际上就是一个命题！接下来，再举一个例子！

闭上眼睛想象一下，在太阳系中有十大行星，每个行星皆以太阳系为参考系，那么每个行星都有一个位置，这个位置往往用光年来描述，接下来收缩你的思维，地球上任何事物都有一个位置，这个位置用经纬度来描述，继续收缩你的思维，内存有一个个的字节组成，我们记做集合 `A＝{内存字节}`，对于 `CPU` 来说其地址总线所能描述的地址种类即为内存字节的位置即地址范围，在 `IA32 CPU` 中地址总线有 32 位，排列组合分步相乘得 `2³²` 种描述方法，即地址集合 `B＝{0x00000000-0xFFFFFFFF}`，C++将内存地址再次抽象为集合 `C={变量名}`，

上面的想象有两点需要单独说明：

1：集合 `B`、集合 `C` 是对同一种事物的不同描述，即对内存字节的不同符号化，或者说别名，这种思维在 C++ 中再次被使用，即引用！

2：要得到集合 `C` 中的变量与之对应的集合 `B` 的内存地址，需要使用 C++ 中的运算符 `&`，即集合 `B` 与集合 `C` 产生一种函数关系，描述为 `B＝f(&C)`，定义域 `C` 是一个有效的 C++ 标识符，值域 `B` 为 `CPU` 地址总结所能描述的地址，在 `IA32 CPU` 中为 `0x000000000xFFFFFFFF`！`&` 操作相应的变量得到的内存地址是一种信息，我们知道，在计算机中任何信息用二进制进行描述，但是正如你看到的现象一样，在操场上老师点名没有说按细胞（二进制）来统计存在的人数（地址），即内存地址集合 `B` 中的每一个元素是一个单独的个体或数据类型，C++ 需要一定的变量（内存）来存储该信息，这种保存变量与之对应的内存地址的方法即为指针！

接下来看下面的语句：

`int a,b,c,d,e,f,g,h,i,j,k,l,m,n,o,p,q,r,s,t,u,v,w,x,y,z;`

我们定义了 26 个变量，对这一现象抽象，将 26 个变量抽象为有限的多个数据元素，将 `int` 抽象为同一种数据类型，那么会产生有有限多个的同一种数据类型组成的元素集合，这个集合我们称为数组，计做 `int array[26];` 继续将 `int array[26]` 抽象为一种数据类型 `x`，要建立有 2 个 `x` 类型元素的数组，则为 `x [2]`,即为 `int array[2][26]`,这称为二维数组，继续将 `int arrary[2][26]` 抽象为一种数据类型 `y`，要建立 3 个 `y` 类型元素的数组，则为 `y[3]`,即为 `int array[3][2][26]`，以此类推，即可以建立任何多维数的数组！

这是 C++ 导出数据类型中数组的理解过程，但是事实上面的推理过程已经被现象所迷惑，回归根本，C++ 中不管内置类型还是导出数据类型，都是人为的指明数据类型所需要的内存空间，即内存空间有人决定，那有没有一种方法让数据类型决定其需要的空间，即让程序决定其使用的内存空间，在 C++ 中提供了动态内存分配以适应这种方法，C++ 的标准库提供了 `vector` 容器来替代数组，即程序根据需要分配相应的内存，回想老子的一句话有容乃大即是这种思想，所以标准库将 `vector` 称为容器没有什么称奇的！对于标准库类型，本系列不会过分讨论，因为它涉及的内容不需要过多的思考，更像是件家电设备，您只需要读一下说明书即可以知道其用法了！我们要做的只是看透其原理就可以了！

### 1.2:函数与递归

程序的作用实际上可以理解为用于实现存储程序原理，即先输入信息，然后通过电脑的判断力分析信息，输出信息的过程，那么作为结构化程序设计的核心内容函数也是为了解决这个问题面被发明出来的，因此函数的设计便分解为如下问题：

1:如何获得输入信息形参

2:如何处理信息函数体

3:如何输出信息返回值

即 C++ 的函数定义为

`<返回类型><函数名>(形参列表)`

```
{
  //函数体
}
```

在函数部分要说难点可能要算是递归了，递归即函数调用自身的过程，结合一个有意思的玩意九连环（传说是诸葛孔明为自己的妻子解闷而发明的）来讨论它！实际上咱们中国人早就在使用这种思维模式，只要我们善于学习并理解我们祖先的留下的一些看上去很老的东西，您一定会有所收获！

问题：编程计算九连环各环的解锁次数！

问题抽象：行为抽象解锁，则集合 `解锁＝{圈上，圈下}` 定义为 `int jiulianhuan()`，数据抽象 `int up ,down` 标识次数。将要解锁的环数抽象为集合 `环数＝{1,2,3,4,5,6,7,8,9}`，那么每一环的解锁过程如下：

过程分析：

`1 下 1`

`2 上 1 下 2 （下 1）`

`3 上 2 上 1 下 1 下 3 (上 1 下 2 (下 1))`

`4 上 2 上 1 下 1 上 3 上 1 下 2 下 1 下 4 （上 2 上 1 下 1 下 3 上 1`

`下 2 下 1 ）`

`.....................`

注意括号的括起来的数据:

继续分析得出:

`1up=0 down=1`

`2 up+2-1 down+2-1 jiulianhuan(1)`

`3 up+3-1 down+3-1 jiulianhuan (2)`

`4up+4-1 down+4-1  jiulianhuan(3)`

`……………….`

编程实现：

```cpp
#include "head.h"

int up=0;/////////////////////////
///target:枚举 INT
///author:reduta
/////////////////////////
////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////
#include <iostream>
#include <bitset>
#include <windows.h>

using namespace std;

typedef struct PEPMAP
{
  HANDLE hfile;
  HANDLE hmap;
  LPVOID lpbase;
}pemap;

bitset<32> bt_con(0x00000000);
////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////

void ShowError(DWORD error)
{
  HLOCAL hlocal = NULL;
  //输出系统错误
  FormatMessage(FORMAT_MESSAGE_FROM_SYSTEM | FORMAT_MESSAGE_ALLOCATE_BUFFER | FORMAT_MESSAGE_IGNORE_INSERTS,
    NULL,error,MAKELANGID(LANG_NEUTRAL,SUBLANG_NEUTRAL),(PSTR)&hlocal,0,NULL);
  cout<<(PSTR)hlocal<<endl;
}

//善后工作
int End(pemap &p)
{
  if (bt_con.test(2)) UnmapViewOfFile(p.lpbase);
  if (bt_con.test(1)) CloseHandle(p.hmap);
  if (bt_con.test(0)) CloseHandle(p.hfile);
  return 0;
}
////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////////

int main(int argc,char *argv[])
{
  if(argc!=2)
  {
    cout<<"useage:showint filename"<<endl;
    return 0;
  }
  /*内存文件映射*/
  DWORD dw_error = 0;
  pemap file = {NULL};
  if (INVALID_HANDLE_VALUE==(file.hfile = CreateFile(argv[1],GENERIC_READ | GENERIC_WRITE,FILE_SHARE_READ | FILE_SHARE_WRITE,NULL,
    OPEN_EXISTING,FILE_ATTRIBUTE_NORMAL,NULL)))
  {
    dw_error = GetLastError();
    ShowError(dw_error);
    bt_con.set(0);
    return End(file);
  }
  if (NULL==(file.hmap = CreateFileMapping(file.hfile,NULL,PAGE_READWRITE,0,0,NULL)))
  {
    dw_error = GetLastError();
    ShowError(dw_error);
    bt_con.set(1);
    return End(file);
  }
  if (NULL==(file.lpbase = MapViewOfFile(file.hmap,FILE_MAP_ALL_ACCESS,0,0,0)))
  {
    dw_error = GetLastError();
    ShowError(dw_error);
    bt_con.set(2);
    return End(file);
  }
  /*检验 PE 有效性*/
  PIMAGE_DOS_HEADER pdos = (PIMAGE_DOS_HEADER)file.lpbase;
  PIMAGE_NT_HEADERS pnt = (PIMAGE_NT_HEADERS)((PBYTE)file.lpbase + pdos->e_lfanew);
  if (pdos->e_magic!=IMAGE_DOS_SIGNATURE || pnt->Signature!=IMAGE_NT_SIGNATURE)
  {
    cout<<"not pe!"<<endl;
    bt_con.set();
    return End(file);
  }
  /*计算节偏移*/
  UINT secoffset = 0;
  DWORD dw_size = 0;
  PIMAGE_SECTION_HEADER psec = (PIMAGE_SECTION_HEADER)((PBYTE)file.lpbase + pdos->e_lfanew + 0x18
    + pnt->FileHeader.SizeOfOptionalHeader);
  PIMAGE_IMPORT_DESCRIPTOR prva = (PIMAGE_IMPORT_DESCRIPTOR)(pnt->OptionalHeader.DataDirectory[1].VirtualAddress);
  for (size_t inx = 0;inx!=pnt->FileHeader.NumberOfSections;++inx)
  {
    dw_size = psec->Misc.VirtualSize;
    if (dw_size==0) dw_size = psec->SizeOfRawData;
    //判断 INT 在哪个节中
    if ((UINT)prva - (UINT)psec->VirtualAddress >=0 && (UINT)prva < (UINT)(psec->VirtualAddress + dw_size))
    {
      secoffset = psec->VirtualAddress - psec->PointerToRawData;
      break;
    }
    ++psec;
  }
  /*输出 IID 中的 DLLNAME 及每个 DLL 输入的函数名*/
  PIMAGE_IMPORT_DESCRIPTOR piid = (PIMAGE_IMPORT_DESCRIPTOR)((PBYTE)file.lpbase
    + pnt->OptionalHeader.DataDirectory[1].VirtualAddress - secoffset);
  PCHAR dll_name = NULL;
  PDWORD ptemp = NULL;
  PIMAGE_IMPORT_BY_NAME pfunc = NULL;
  while (piid->OriginalFirstThunk)
  {
    dll_name = (PCHAR)((PBYTE)file.lpbase + piid->Name - secoffset);
    ptemp = (PDWORD)((PBYTE)file.lpbase + piid->OriginalFirstThunk - secoffset);//指向 IMAGE_THUNK_DATA32
    cout<<"############ "<<dll_name<<" ###################"<<endl;
    while (*ptemp)//*ptemp 为 IMAGE_THUNK_DATA32 的内容
    {
      if ((*ptemp & 0x80000000)==0x80000000) //如果使用 ordianl 寻址，则 IMAGE_THUNK_DATA32 中必然 0x80000xxxx
      {
        cout<<"Ordinal:"<<hex<<*ptemp - 0x80000000<<endl;
      }
      else
      {
        pfunc = (PIMAGE_IMPORT_BY_NAME)((PBYTE)file.lpbase + *ptemp - secoffset);
        cout<<pfunc->Hint<<"  "<<pfunc->Name<<endl;
      }
      ++ptemp;
    }
    ++piid;
  }
  //把位域全部设置为 1,退出程序
  bt_con.set();
  return End(file);
}

int down=0;
```



# 递归函数示例

```
int jiulianhuan(int x)

{

  //没有解锁环数

  if(x<=0)exit(1);

  //解第环只要解下来就可了

  if(x==1)

  {

    return ++down;

  }

  //解锁环数大于 2 时

  if(x>1)

  {

    up+=x-1;

    down+=x-1;

    jiulianhuan(x-1);

  }

  return 0;

}

int main()

{

  int x;

  cout<<"输入解锁的环数:";

  cin>>x;

  jiulianhuan(x);

  cout<<"up:"<<up<<" down:"<<down<<endl;

  return 0;

}
```

通过这个例子，相信朋友们对于递归有了一定的认识！

# 二：形形色色的指针

## 2.1: 指针基本知识

如上所述，指针是一个 `B=f(&C)` 的函数，其值域为 `0x000000000xFFFFFFFF`，即指针为无符号整数！理解这一点很关键，因为只要明确指针为无符号整数，则指针即可以进行加减算术运算、大小关系运算，那么为什么不能进行乘除算术运算呢？还是上面所说的，指针作为数据是一个整体，就像人一样，我们不可能用细胞个数来统计人数一样！如你所见，任何在 `0x000000000xFFFFFFFF` 范围内的整数都可以直接在 C++ 源码中通过操作该范围内的地址保存数据:

```
#include "head.h"

int main()

{

  __try

  {

    *(unsigned int *)0=1;

  }

  __except(1)

  {

    cout<<"指针是 unsigned int 类型!"<<endl;

  }

  return 0;

}
```

`__try` 与 `__except` 是 `cl` 编译器内置的关键字，在讨论 C++ 的异常处理时会再次分析，因为默认 WINDOWS 的内存分区特性中 `0x00000000-0x00010000` 为 NULL 空指针区，所以上面的程序会产生异常，因此使用了内置异常处理关键字！

## 2.2：保存指针地址的指针

有上可知，指针保存的是集合 B 中的一个内存地址，作为指针来说也依然遵循 `B=f(&C)` 函数，即任何指针也有一个集合 B 中的地址所对应，在 C++ 源码中要保存这个地址，我们需要使用指向指针的指针，定义格式为 `int **p;`

```
#define FAST

#include "head.h"

int main()

{

  int a=2;

  int *p=&a;

  int **pp=&p;

  cout<<"now output 2:"<<a<<" "<<*p<<" "<<*( *pp)<<endl;

  return 0;

}
```

值得注意 `*<指针名>` 即表示该指针名所保存的地址所对应的集合 A 中的数据！上面的程程序都会输出 2, 对于指针的操作 IA32 CPU 对应的指令为 `lea`，反汇编上面的程序：

```
//反汇编代码
00401288    C745 FC 02000000  mov dword ptr ss:[ebp-4],2  ;int a=2
0040128F    8D45 FC           lea eax,dword ptr ss:[ebp-4]  ;&a
00401292    8945 F8           mov dword ptr ss:[ebp-8],eax  ;int *p=&a
00401295    8D4D F8           lea ecx,dword ptr ss:[ebp-8]  ;&p
00401298    894D F4           mov dword ptr ss:[ebp-C],ecx  ;int **p=&p
```

对比源码与反汇编代码，不难发现，其关系就是通过 `&` 得到相应的地址，即实现是 `B=f(&c);`

而 `*` 所表示的地址内容即为 `A=f(*B)`（定义域为集合 B 中的内存地址范围），值域为集合 A 中所保存的数据！

## 2.3: const、指针、引用

在系列 2 中讨论了常规 const 常量的机制，即编译时进行替换，事实上并不是所有的 const 常量都是在编译时进行替换的，比如下面的代码：

```
int i;

cin>>i;

const int ci=i;
```

只有当程序运行时，才能知道 const 常量的值！`const` 的作用除了描述符号常量外，更大的作用是作数据保护：

```
//const 用于符号常量，实现编译级的数据保护
#define FAST

#include "head.h"

int main()

{

  const int i=2;

  __asm mov dword ptr ss:[ebp-4],10

  cout<<i<<endl;

  return 0;

}
```

上面的程序虽然通过汇编指令修改了符号常量 `i` 的存储内容，但是当 `cout` 对象调用 `<<` 操作符重载函数时，因为编译时进行了替换，即为 `cout<<2<<endl;` 所以上面的汇编指令实际上并不能改变程序！

指针也经常用到 `const` 进行修饰，它有两种修饰方法，如下面的语句所述：

```
int a=0x61,b=0x62;

int *const p=&a;

//p=&b;error

int const *p=&b;

//*p=10;error
```

上面的语句中 `int *const p=&a;` `const` 修饰的是指针 `p`，即 `p` 是一个常量，所以必须初始化，如 `int *const p;` 这样的语句是不对的，因为在 C++ 中符号常量必须进行初始化！即然 `p` 为常量则 `p=&b;` 必然是错误的，因为常量数据不能修改！

接下来 `int const *p=&b;` `const` 修饰的是 `*p`，`*p` 为指针 `p` 所保存的的地址对应的数据内容，即该地址为一常量或者说 `p` 保存的变量 `b` 的地址应该是一个常量，所以 `*p=10;` 是错误的，接下来看一下它们的机制：

```
//const 修饰指针的机制
#define FAST

#include "head.h"

int main()

{

  int a=0x61,b=0x62;

  int *const p=&a;

  __asm

  {

    lea eax,dword ptr ss:[ebp-8]

    mov dword ptr ss:[ebp-c],eax//error

  }

  int const *pp=&b;

  __asm

  {

    lea ecx,dword ptr ss:[ebp-4]

    mov dword ptr ss:[ebp-10],ecx

  }

  return 0;

}
```

上面的程序会提示无法编译，错误在于 `mov dword ptr ss:[ebp-c],eax`，因为我们前面定义的 `p` 为 `const` 类型的指针，它是一个常量，所以赋值错误，即 `const` 类型的指针并不是像 `const` 常量那样通过编译时替换实现保护，而是有编译器进行的安全检测，以实现数据保护！

```
//const 类型指针实际上即是一个引用
#define FAST

#include "head.h"

int main()

{

  int a=0x61;

  int &b=a;

  int *const p=&a;

  return 0;

}
```

反汇编上面的程序：

```
00401288    C745 FC 61000000  mov dword ptr ss:[ebp-4],61  ；int a=0x61
0040128F    8D45 FC           lea eax,dword ptr ss:[ebp-4]  ;&a
00401292    8945 F8           mov dword ptr ss:[ebp-8],eax  ;int &b=a;
00401295    8D4D FC           lea ecx,dword ptr ss:[ebp-4]  ;&a
00401298    894D F4           mov dword ptr ss:[ebp-C],ecx  ;int *const p=&a
```

对比反汇编指令 `const` 类型的指针与引用的指令是一样的，上面已经提到 `int *const p=&a`，如果再编写 `p=&b；` 类似的语句是编译器会提示错误，那么我们执行下面的语句呢？

```
int a=61,b=0x62;

int &c=a;

&c=b;//error
```

正因为引用是一种 `const` 类型的指针，所以引用必须初始化，类似的如上面的语句也是错误的.

一句话，`const` 修饰指针 `p`，则 `p` 为常量，不能再赋值，即为引用，修饰 `*p` 则 `*p` 为常量

即无法通过 `*p` 修改其指向地址的内容！还需要注意，`const` 修饰指针用于数据安全是有编译器安全检测得知的！

## 2.4：void 与 this 指针

这种指针在讨论基本数据类型时已经进行了说明，这里再次说明，是为了纠正系列二中我的错误，关于 `void` 类型用于 `main` 函数程序无法正常退出的原因不在于 `void` 类型，而在于 `return 0` 的语句，朋友们可以看一下网友的纠正，表示感谢！

有关 `this` 指针的内容，我想在下一系列讨论面向对象时进行说明！

## 2.5: 指针与数组

这部分不外乎两种组合，即指针数组是一种有指针类型组成的数组和数组指针是一种指向数组的指针！

```
//指针数组
#define FAST

#include "head.h"

int main()

{

  int *p[2];

  int a=0x61,b=0x62;

  p[0]=&a;

  p[1]=&b;

  cout.setf(ios::hex,ios::basefield);

  cout<<p[0]<<" "<<*p[0]<<endl
    <<p[1]<<" "<<*p[1]<<endl;

  return 0;

}
```

上面的程序我们 `int *p[2];` 定义了有两个指针的数组简称指针数组，它本质上还是对指针运用数组的推导过程，无需多说！

```
//数组指针
#define FAST

#include "head.h"

int main()

{

  int a[2]={1,2};

  int *p=a;

  int i=0;

  while(i!=2)

  {

    cout<<*p<<endl;

    p++;

    i++;

  }

  return 0;

}
```

上面的程序我们用 `int *p=a;` 将指针的第一个元素的地址保存于 `p` 中，在 C++ 中 `[]` 与 `*` 实际上是相通的，如下面两个程序是相等的:

```
//1.exe
#include "head.h"

int main(int argc,char *argv[])

{

  if(argc==2&&strcmp(argv[1],"fuck")==0)

  {

    cout<<"fuck the world!"<<endl;

  }

  return 0;

}

//2.exe
#include "head.h"

int main(int argc,char **argv)

{

  if(argc==2&&strcmp(argv[1],"fuck")==0)

  {

    cout<<"fuck the world!"<<endl;

  }

  return 0;

}
```

如上两个程序中，`char **argv` 与 `char *argv[]` 是等价的，故有相同的输出，值得注意 `argc` 与 `*argv[]` 是两个需要 windows 启动函数配合使用的主函数参数，故需要将 `FAST` 宏开关去掉！

## 2.6: 函数与指针

如 2.5 所述一样，函数与指针也有两种组合，即指针函数即返回值为集合 B 中地址的函数，函数指针即指向函数的指针！

```
//指针函数
#define FAST

#include "head.h"

int g_test;

int *retn()

{

  return &g_test;

}

int main()

{

   cout<<retn()<<endl;

   return 0;

}
```

上面的函数 `retn` 会返回一个 `int` 类型的指针，故声明为 `int *retn(int p)`！

```
//指针函数
#define FAST

#include "head.h"

void Cout()

{

  cout<<"hello world"<<endl;

}

int main()

{

  void (*func)();

  func=Cout;

  func();

  return 0;

}
```

上面的程序使用 `void (*func)();` 定义了一个指向 `void` 返回值参数为空的函数指针 `func`，它的作用是指向该函数的地址！在 C++ 的源码级任何函数即为一个地址，如下面的程序；

```
//函数名即一个地址
#define FAST

#include "head.h"

int main()

{

  cout<<"hello world!"<<endl;

  __asm jmp main

  return 0;

}
```

此程序通过 `jmp main` 无条件跳转到 `main` 函数处，从而导致程序不断的输出 hello world，值得一提的是上面的程序并不是一个死循环，这是有栈空间耗尽导致的，默认 windows 给予 1MB 的栈空间，有链接器通过 PE 结构中的 `IMAGE_OPTIONAL_HEADER` 结构中的成员 `SizeOfStackCommit` 指明！

函数指针在应用程序编程中经常用到，一般我们将函数指针使用 `typedef` 关键字定义为一种数据类型，比如在进程管理时，要挂起一进程，实际上这句话说的不够严谨，因为众所周知，现代操作系统中线程为 CPU 调度的基本单位，所以每次挂起进程实际上是在挂起线程，而在 `NTDLL.DLL` 动态链接库依然存在直接挂起进程的函数 `ZwSuspedProcess`，因此我们可以使用函数指针得到该函数的地址，然后通过函数指针调用它，下面的程序演示了上面描述的过程：

```
typedef DWORD(WINAPI *pfsuspend)(HANDLE hThread);

pfsuspend SuspendProcess;

void SuspendPro(DWORD pid)

{

  hdll=LoadLibrary("NTDLL.DLL");

  if(hdll==NULL)

  {

    cout<<"Load Dll error"<<endl;

    exit(1);

  }

  SuspendProcess=(pfsuspend)GetProcAddress(hdll,"ZwSuspendProcess");

  HANDLE hpro;

  if((hpro=OpenProcess(PROCESS_ALL_ACCESS,NULL,pid))==NULL)

  {

    cout<<"Openprocess Error"<<endl;

    exit(1);

  }

  SuspendProcess(hpro);

  FreeLibrary(hdll);

}
```

`Typedef` 在前面的系列中曾经讨论过，它和 `#define` 的不同之处，在于后者是单纯的进行宏替换，而 `typedef` 则不是，虽然明确的进行了说明，但是对于下面的语句，可能有时还有些迷糊！

```
typedef char * pchar

const pchar x;
```

上面的语句中很多朋友们可能会习惯性的以为 `const pchar x` 为 `const char * x;` 即 `char const *x`，这时候你已经开始忽略 typede 与 `#define` 的不同之处了，而实际上 `const pchar x;` 中 `const` 修饰的是 `x`，即应该等价于 `char *const x;`

## 2.7：函数指针数组

函数指针数组即保存函数指针元素的数组即有多个指向函数的指针组成的数组！

我们用一个看上去很傻的例子来理解函数指针数组：

问题：用函数指针数组编写输出 hello world 的程序！

```
#define FAST

#include "head.h"

void C1()

{

  cout<<"hello";

}

void C2()

{

  cout<<" world";

}

void C3()

{

  cout<<"!";

}

void C4()

{

  cout<<endl;

}

int main()
```



# 函数指针与数组

```cpp
{  
  void (*func[4])();  
  for(int i=0;i<4;i++)  
  {  
    switch(i)  
    {  
    case 0:  
      func[0]=C1;  
      break;  
    case 1:  
      func[1]=C2;  
      break;  
    case 2:  
      func[2]=C3;  
      break;  
    case 3:  
      func[3]=C4;  
      break;  
    }  
  }  
  func[0]();  
  func[1]();  
  func[2]();  
  func[3]();  
  return 0;  
}  
```

这估计是您见过的最复杂的 hello world 程序，上面的程序中使用 `void (*func[4])()` 定义了一个有 4 个函数指针的数组 `func`，然后通过 `for` 结构与 `switch` 结构对函数指针赋值，最后调用这函数指针数组中的元素用于调用函数！

## 三：函数

### 3.1：函数重载与函数模板

我们按照 `<返回值><函数名>(形参表){函数体}` 来讨论函数源码级的知识：

**返回值**：返回值是函数输出运算结果的值，其类型必须与函数体中的 `return` 语句返回的数据类型一致，即返回值对于函数来说是一个可变值，它不能完全代表一个函数

**函数名**：是一个内存地址，它可以完全代表一个函数！

**形参表**：是函数接收信息输入的方法，输入的信息不同，表示该函数的功能不同，故可以用于区别一个函数！

**函数体**：信息处理的过程，一般都会返回处理的结果，没有结果即无返回值，则返回值类型定义为 `void` 类型，同样 `void` 返回值类型的函数也可以使用 `return` 语句，只不过为 `return ;` 它的作用是表示结束，而并不具备 `return 信息;` 的功能！

总上所述，区分两个函数的方法如下：

1. 函数名不同  
2. 形参不同：形参不同又可以分为形参类型不同、形参数量不同  

相同的函数名，有形参决定不同的函数的编程方法在 C++ 中称为函数重载，它实际上是一种事物的多种不同形式，即多态性，比如下面的程序定义了多个相同名字参数不同的 `add` 函数

```cpp
//函数重载  
#define FAST  
#include "head.h"  
#include <string>  

void add(string s1, string s2)  
{  
  cout << s1 + s2 << endl;  
}  

void add(int a, int b)  
{  
  cout << a + b << endl;  
}  

void add(float a, float b)  
{  
  cout << a + b << endl;  
}  

int main()  
{  
  string s1("hacker is "), s2("hacking!");  
  add(s1, s2);  
  add(1, 2);  
  add(1.1F, 2.2F);  
  return 0;  
}  
```

用老子的话说道生一，一生二，二生三，三生万物，返回去即为，万物为三，三为二，二为一，一为道，道为无，这里的道即为类，而无则为模板！也就是说思维逻辑上本身就可以存在逆推的！C++ 可以将上面的函数定义为函数模板！上面的程序等价于：

```cpp
#define FAST  
#include "head.h"  
#include <string>  

template <typename T> void add(T a, T b)  
{  
  cout << a + b << endl;  
}  

int main()  
{  
  string s1("hacker is "), s2("hacking!");  
  add(s1, s2);  
  add(1, 2);  
  add(1.1F, 2.2F);  
  return 0;  
}  
```

### 3.2：参数传递

如上面函数重载与函数模板的例子，我们不难发现，函数在调用时，举例：

```
add(1,2);

template <typename T> void add(T a, T b)

T a=1;
T b=2;
```

上面 `add(1,2)` 中的 `1` 和 `2` 我们称为实参，`add (T a, T b)` 中的 `a` `b` 我们称为形参，这个过程是一个最简单的赋值表达式的再抽象，即参数传递是赋值表达式的函数结构化，那么为什么是赋值表达式的再抽象呢？

想象一下，太极图中的阴阳，它给人们的第一感觉就是在转动，即世物有阴阳组成不假，但是阴阳若不动就没有生命力，比如人的血液不动则人已死亡，比如雨天过后的积水，长时间不流动终究会干涸，而信息在计算机中我们往往称为 `bit` 流，这个流说明了它的属性，`bit` 信息只有不断的流动才能存储于不同的存储位置，才能产生我们想要的信息，而在 C++ 中，赋值是惟一实现数据流动的方法，对于赋值以外的任何运算符，它们都是原地的运算，而赋值则使它们灵活起来，所以汇编教材中，第一个学会的指令即为 `mov` 指令！

好了，到这里理解了参数传递为赋值表达式的函数结构化，如果每次参数的传递都使用上面的赋值方法复制信息（以下简称传值），如果是比较大型的信息比如 `int array[1024][1024[1024]`，那么这样信息复制起来将相当的复杂，程序在算法时间上将处于劣势，特别是引入面向对象后，大型程序越来越普遍，如果再使用传统的传值方法将严重影响大型程序的运行与开发，再返回到第一部分，数据对应于集合 `A`，而任何数据存放的位置都有一个地址集合 `B` 与之对应，因此，我们可以通过传递信息地址的方法来简化传值代来的算法时间消耗，这称为传址（区别传值），要传递地址，一方面我们可以使用指针，另一方面可以使用 `const` 类型的指针即引用，比如上面的问题 `int array[1024][1024][1024]`，我们只要将 `array` 的第一个元素的地址传递给函数，函数通过地址访问信息就可以了！当然正如指针中讨论的 `const` 修饰一样，为了安全期间，一般使用引用，因为它可以保证不修改输入的数据！

上面两部分内容，我想我说的可能复杂了许多，不妨通过反汇编的方法，来深入的理解一下。

### 3.3：反汇编下的函数

先来看一下函数结构与汇编指令级的对应

* 返回值 `EAX` 保存  
* 函数名内存地址  
* 形参 `push` 入栈操作  
* 函数调用 `call` 指令  
* 返回 `retn` 指令  

函数运动我们暂且将函数的调用与返回的自动进行称为函数运动，它需要栈帧来完成，每个函数都有一个栈帧结构，每个栈帧结构产生都会首先保存原有栈帧的数据以备恢复，然后开辟新的栈帧，以用于控制当前函数，每个栈帧都有一个栈底和栈顶，分别用 `ebp` 和 `esp` 寄存器指示，除了说明栈帧外还需要说明栈，栈是一种数据结构，它里面的数据元素是后进先出！

```cpp
//函数结构对应  
#define FAST  
#include "head.h"  

void show(int x)  
{  
  cout << x << endl;  
}  

int main()  
{  
  show(1);  
  return 0;  
}  
```

```asm
//反汇编  
004012F0 > 55                push ebp    ;保存前栈帧的栈底，以备恢复  
004012F1    8BEC             mov ebp,esp  ;保存新栈帧的栈底即为当前栈顶  
004012F3    83EC 40          sub esp,40   ;开辟一定的栈空间  
004012F6    53                push ebx    ;保存 EBX  
004012F7    56                push esi    ;保存 ESI  
004012F8    57                push edi    ;保存 EDI  
004012F9    8D7D C0          lea edi,dword ptr ss:[ebp-40]  ;将开辟空间的最后一个栈的地址保存到 EDI 中  
004012FC > B9 10000000       mov ecx,10   ;接下来进行栈初始化，这里设置初始化的次数，有开辟的栈空间大小决定，上面为 0x40 的栈空间，每个栈 4 字节，故为 0x10 次  
00401301    B8 CCCCCCCC       mov eax,CCCCCCCC ;保存初始化时使用的值  
00401306    F3:AB            rep stos dword ptr es:[edi] ;重复 ECX 中的次数，将栈空间初始化 EAX 中的值 0xcccccccc，即 int3 断点  
00401308    6A 01            push 1      ;show(1)中的实参 1 入栈  
0040130A    E8 F6FCFFFF       call api.00401005 ;对应于 c++源码中的()运算符  
0040130F    83C4 04          add esp,4   ;回收实参 1 使用的一个栈空间  
00401312    33C0             xor eax,eax ;eax 进行模为二 2 加法操作，模为 2 的加法操作遵守补码操作，不管 eax 为何值，都会得 0,比如 EAX=11111111，xor 操作后为 22222222，模为 2 补码操作后为 00000000，相反如果 EAX=00000000，则相加后都是 0，与 C++中的^运算符等价！  
00401314    5F                pop edi    ;edi 出栈与开始的 push edi 相对应  
00401315    5E                pop esi    ;esi 出栈与开始的 push esi 相对应  
00401316    5B                pop ebx    ;ebx 出栈与开始的 ebx 相对应  
00401317    83C4 40          add esp,40  ;回收栈空间，与 sub esp,40 相对应  
0040131A    3BEC             cmp ebp,esp ;比较 ebp 是否与 esp 相等，作用是检测栈空间是否平衡，此时 ZF=1，  
0040131C    E83B00000        call api._chkesp        ;此处调用的是 chkesp 函数，这个函数只有一句代码（jnz short MSVCRT.77C05BBF  
) 不相等则跳转到 77C05BFF 处，此处有一 int3 断点，引发一个异常处理！                  ;   
00401321    8BE5             mov esp,ebp ;对应于 mov ebp,esp  
00401323    5D                pop ebp    ;原栈帧的栈底出栈  
00401324    C3                retn       ;返回到 call main 处的下一条地址中继续执行  
```

通过上面的反汇编，我们理解了函数参数的传递，即函数的形参先入栈，然后才调用函数，因此我们可以得出如下结论：

**结论**：

任何函数的形参都为内部 `auto` 存储类型，即进栈，对于传址的参数传递方式，形参为地址，此地址也为内部 `auto` 存储类型，即进栈！当函数调用完毕后，一切形参将被销毁，如果调用函数与被调用函数要共享数据，除了通过被调用函数的返回值外，只能通过使用外部 `auto` 存储类型来存储信息，即外部变量！

上面的程序只有一个形参，那么如果函数有多个形参，则有一个问题需要讨论即哪个参数先入栈：

修改上面的源码为如下：

```cpp
//参数入栈顺序  
#define FAST  
#include "head.h"  

void show(int x, int y)  
{  
  cout << x << y << endl;  
}  

int main()  
{  
  show(1, 2);  
  return 0;  
}  
```

```asm
//反汇编核心代码  
00401328    6A 02           push 2  
0040132A    6A 01           push 1  
0040132C    E8 D9FCFFFF     call api.0040100A  
```

通过代码可以看到 `2` 先入的栈，然后是 `1` 入的栈，得出如下结论：

参数入栈顺序为从右到左入栈的，不同的编程语言参数入栈的顺序不同，对于 C 语言来说，有三种参数入栈约定：

* `__cdecl`
* `__stdcall`
* `__fastcall`

这三种函数参数入栈约定都是从右到左的将参数入栈，不同之处在于栈平衡最后有调用函数还是被调用函数完成，对于 `__cdecl` 调用约定来说，它是由调用函数完成栈平衡，另外两个则通过子函数完成，默认 vc 开发环境中使用 `__stdcall` 函数调用约定！

函数参数入栈的顺序问题解决了，在上面的两个例子，我们一直在使用 `show(1)` 或 `show(1,2)` 的方法来向函数传递实参，前面已经提到函数调用是一个赋值表达式的函数结构化，那么对于 `show(1)` 和 `show(1,2)` 两种调用方法在机器级来描述即为立即寻址方式，即我们通过上面的函数根本看不到传值与传址两种参数传递方法的不同，因此我们更换一下程序进行反汇编：

```cpp
//理解传值  
#define FAST  
#include "head.h"  

int add(int x, int y)  
{  
  return (x + y);  
}  

int main(int argc, char *argv[])  
{  
  int a = 1;  
  int b = 2;  
  add(a, b);  
  return 0;  
}  
```

```asm
//主函数相应的汇编代码  
mov dword ptr ss:[ebp-4], 1  
mov dword ptr ss:[ebp-8], 2  
mov eax, dword ptr ss:[ebp-8]  
push eax  
mov ecx, dword ptr ss:[ebp-4]  
push ecx  
call add  
```

通过指令分析，我们看到参数入栈时为先将相应的信息内容即 `1` 和 `2` 分别保存到寄存器 `eax` 和 `ecx` 中，然后通过寄存器寻址方式将信息保存到栈中，即传值传递的是信息内容！

```cpp
//理解传址  
#define FAST  
#include "head.h"  

int add(int &x, int &y)  
{  
  return (x + y);  
}  

int main(int argc, char *argv[])  
{  
  int a = 1;  
  int b = 2;  
  add(a, b);  
  return 0;  
}  
```

```asm
//主函数关键代码的汇编指令  
mov dword ptr ss:[ebp-4], 1  
mov dword ptr ss:[ebp-8], 2  
lea eax, dword ptr ss:[ebp-8]  
push eax  
lea ecx, dword ptr ss:[ebp-4]  
push ecx  
call add  
```

分析指令，参数入栈时，和传值时不同，传址使用的是 `lea` 指令将信息对应的内存地址保存到 `eax` 和 `ecx` 中，然后函数通过寄存器间接寻址方式，定位信息即（`1,2`），在这个过程中传递的是地址，使用的是寄存器间接寻址方式定位操作数！

总上所述，传值与传址的不同在于如下两点：

1. 传值是传递的信息内容，传址是传递的信息存储于存储器的地址  
2. 传值使用立即寻址方式或寄存器寻址方式，而传址，则使用寄存器间接寻址方式，即寄存器中保存有操作数的有效地址！

如你所见，在上面的过程中，我们并没有讨论栈帧更详细的结构，要理解函数栈帧的配合，首先需要理解 `call` 指令与 `retn` 指令做了什么？以上面的 `show(1,2)` 函数为例进行说明：

```asm
//形参入栈  
00401328    6A 02           push 2  
0040132A    6A 01           push 1  
0040132C    E8 D9FCFFFF     call api.0040100A  
00401331    83C4 08         add esp,8  
```

当 F8 单步到 `40132c` 时，此时栈中数据：

```
0012FF6C   00000001；左面的参数 2
0012FF70   00000002; 右面的参数 1
```

F7 跟进程序：

```asm
//跟进后的代码  
0040100A   /E9 71020000     jmp api.show  
```

此时栈中数据：

```
0012FF68   00401331         ;返回到 api.00401331 来自 api.0040>
0012FF6C   00000001   
0012FF70   00000002
```

对比 F7 前后的栈数据不难发现栈中多了一个 `00401331` 的地址，此地址为 `call api.00400100A` 指令的下一条指令地址，即 `call` `api.00400100A` 指令会先将其下一指令的地址入栈，然后执行 `jmp 0040100A` 的指令！

```asm
//执行 jmp 指令后的代码  
```



```
00401280 >  55              push ebp  

00401281   8BEC             mov ebp,esp  

00401283   83EC 40          sub esp,40  

00401286   53               push ebx  

00401287   56               push esi  

00401288   57               push edi  

00401289   8D7D C0          lea edi,dword ptr ss:\[ebp-40\]  

0040128C   B9 10000000      mov ecx,10  

00401291   B8 CCCCCCCC      mov eax,CCCCCCCC  

00401296   F3:AB            rep stos dword ptr es:\[edi\]  
```

和我们上面描述的汇编指令是一样的，接下来执行 `00401296` 处，然后查看栈中的数据，只看其中关键的地方:

```
0012FF5C   00000000  

0012FF60   00000000       ; 

0012FF64  /0012FFC0       ；前一个栈帧的基址  

0012FF68  |00401331        ；   函数返回地址  

0012FF6C  >|00000001    ;形参 1  

0012FF70  >|00000002 ；形参 2  
```

继续执行程序到如下代码处：

```
004012DD   5F               pop edi  

004012DE   5E               pop esi  

004012DF   5B               pop ebx  

004012E0   83C4 40          add esp,40  

004012E3   3BEC             cmp ebp,esp  

004012E5   E8 94000000      call api.\_chkesp                        ; jmp 到 MSVCRT.\_chkesp  

004012EA   8BE5             mov esp,ebp  

004012EC   5D               pop ebp  

004012ED   C3               retn  
```

观察栈中数据变化，直行到最后一条指令 `retn` 时，栈中数据为

```
0012FF68   00401331         返回到 api.00401331 来自 api.0040>  

0012FF6C   00000001  

0012FF70   00000002  
```

此时 `F8`，则返回到栈中保存的 `00401331` 地址处，通过这样的跟踪不难发现，所谓栈帧可以理解栈帧的关系结构大体上为:

```
show()的栈帧空间  

main()的栈帧基址  

返回地址  

形参 1  

形参 2  

。。。。。。。。。  
```

不难发现，函数能实现自动化运行的一个很关键的东西为返回地址，如果将这个地址修改掉，则程序流程将发生变化！即函数是 `C++`中的一种高级控制结构！

在这里我们不难发现函数可以如下定义，即函数是对赋值表达式进行抽象的一种高级控制结构！

洋洋洒洒写了这么多，只要理解了函数是对赋值表达式进行抽象的一种高级控制结构就可以了！
