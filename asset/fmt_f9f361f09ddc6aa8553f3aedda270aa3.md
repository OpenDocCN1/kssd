
```cpp
    for (size_t i=0; i!=CurrentSize+1; ++i)
    {
      for (size_t j=i+1; j!=CurrentSize+1; ++j)
      {
        if (pList[i] > pList[j])
        {
          pList[i] = pList[i] ^ pList[j];
          pList[j] = pList[i] ^ pList[j];
          pList[i] = pList[i] ^ pList[j];
        }
      }
    }
    return true;
  }
  if (con==2)
  {
    for (size_t i=0; i!=CurrentSize+1; ++i)
    {
      for (size_t j=i+1; j!=CurrentSize+1; ++j)
      {
        if (pList[i] < pList[j])
        {
          pList[i] = pList[i] ^ pList[j];
          pList[j] = pList[i] ^ pList[j];
          pList[i] = pList[i] ^ pList[j];
        }
      }
    }
    return true;
  }
  if (con==3)
  {
    for (size_t i=0; i!=CurrentSize+1; ++i)
    {
      for (size_t j=i+1; j!=CurrentSize+1; ++j)
      {
        if (pList[i]==pList[j])
        {
          for (size_t k=i; k!=CurrentSize+1; ++k)
          {
            pList[k] = pList[k+1];
          }
          --j;
          --i;
          --CurrentSize;
        }
      }
    }
    return true;
  }
  if (con==4)
  {
    for (size_t i=0,j=CurrentSize; i!=(CurrentSize+1)/2; ++i,--j)
    {
      pList[i] = pList[i] ^ pList[j];
      pList[j] = pList[i] ^ pList[j];
      pList[i] = pList[i] ^ pList[j];
    }
  }
}
```

//假设法求最大值与最小值
```cpp
template<typename Type> void SqList<Type>::MaxAndMin()
{
  Type Max = pList[0];
  Type Min = pList[0];
  for (size_t inx=0; inx!=CurrentSize; ++inx)
  {
    if (pList[inx]>Max)
    {
      Max = pList[inx];
    }
    if (pList[inx]<Min)
    {
      Min = pList[inx];
    }
  }
  cout<<"Maximum:"<<Max<<" "<<Minmum:"<<Min<<endl;
}
```

//PE 区段表就是一个典型的顺序表。只不过它的数据元素为 `IMAGE_SECTION_HEADER`
```cpp
void ShowPeSecInfo(LPVOID lpBase)
{
  PIMAGE_DOS_HEADER pdos = (PIMAGE_DOS_HEADER)lpBase;
  PIMAGE_NT_HEADERS pnt = (PIMAGE_NT_HEADERS)((PBYTE)lpBase + pdos->e_lfanew);
  PIMAGE_SECTION_HEADER pfirst = (PIMAGE_SECTION_HEADER)((PBYTE)lpBase + pdos->e_lfanew + sizeof(IMAGE_NT_SIGNATURE) +
sizeof(IMAGE_FILE_HEADER) + pnt->FileHeader->SizeOfOptionalHeader);
  for (size_t inx=0; inx!=pnt->FileHeader.NumberOfSections; ++inx)
  {
    cout<<pfirst[inx].Name<<endl;
    //cout<<(pfirst+inx)->Name<<endl;
  }
}
```

对于 `PE` 区段的添加、删除等操作也是一样的类似。

整个 `PE` 结构可以理解为一链式顺序表，即通过顺序表存储地址的方式。

//`oop` 中的对象数据也是一个顺序表，有 `this` 指针指明对象数据成员的起始地址，通过 `this + inx` 的方式访问
```cpp
#include <iostream>
using namespace std;

class test
{
  int a,b;
public:
  test(int a=0,int b=0);
  void show()const;  //类外定义排除 `inline` 内联
};

test::test(int a/* =0 */,int b/* =0 */)
{
  this->a = a;
  this->b = b;
}

void test::show()const
{
  cout<<a<<" "<<b<<endl;
}

int wmain()
{
  test fk(1,1);    //构造函数在汇编级返回当前对象的 `this` 指针，所以下面使用的 `eax` 而非 `ecx`
  __asm mov [eax],10
  __asm mov [eax+4],10
  fk.show();
  return 0;
}
```

//多维数组也是顺序表
```cpp
int wmain()
{
  int a[2][2][2][2] = {1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16};
  int *p = NULL;
  __asm lea eax,a
  __asm mov p,eax
  for (size_t inx=0; inx!=16; ++inx)
  {
    cout<<p[inx]<<endl;
  }
  return 0;
}
```

//位操作指令也是一个顺序表
`bt/bts/btr/btc reg/mem reg/imm`

第一个操作数为 `pstart` 第二个操作数 `inx`

`bt` -> `pstart[inx]->cf`
`bts` -> `pstart[inx]->cf` `pstart[inx] = 1`
`btc` -> `pstart[inx]->cf` `pstart[inx] = ~pstart[inx]`
`btr` -> `pstart[inx]->cf` `pstart[inx] = 0`

### 2、基本的整数表示

原码：

反码：最大值 - 当前值

补码：以最右侧的 1 为起始，将其左边的二进制位求反。

### 3、对齐问题

地址对齐 `if (addr & align - 1)`
大小对齐 `if (dwsize % align)`

### 4、逻辑运算

### 5、熟悉位运算

除了上面的 `bt/btc/btr/bts` 还需要理解 `bsf/bsr` (可以快速析出程序中的第一个 1 的位置)、`test` 指令。
