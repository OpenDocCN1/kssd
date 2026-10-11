
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
