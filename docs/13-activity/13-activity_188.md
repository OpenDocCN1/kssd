
# “美丽的代码”自主命题--利用硬件栈实现树的遍历

> 原文：[`www.pediy.com/kssd/pediy10/73686.html`](https://www.pediy.com/kssd/pediy10/73686.html)

前一段时间写的东西,不知道合不合题...

相信这个程序的效率一定很高,代码也很简洁,哈哈

不管怎么样,show 一个(好像没有说不让用汇编,酷)

还有,俺很喜欢书,希望能得到一本,哈哈

`winxp`,`vs2008` 编译通过,其他 x86 编译平台也是可以的.

## 代码

```cpp
// stack_traversal2.cpp : 定义控制台应用程序的入口点。
//

#include "stdafx.h"

/*************************************************************************/
//这部分很混乱,还是参考源代码吧--sigh...
//┏━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┓
//┃功能：利用 IA-32 硬件的栈结构实现树的遍历               ┃
//┃                           ┃
//┃1、元素内容（一个简单的结构体 STU）                 ┃
//┃┏━━━━┳━━━━━━━┓                         ┃
//┃┃int ID   ┃ char* name  ┃                       ┃
//┃┗━━━━┻━━━━━━━┛                       ┃
//┃                           ┃
//┃2、树节点的结构（STU_NODE）                 ┃
//┃┏━━━━━━━━━━┳━━━━━━━━━━┳━━━━━━━━┓               ┃
//┃┃PSTU_NODE child   ┃PSTU_NODE right   ┃PSTU pValue     ┃               ┃
//┃┗━━━━━━━━━━┻━━━━━━━━━━┻━━━━━━━━┛               ┃
//┃ 采用孩子链的方法，容纳多孩子。                 ┃
//┃                ┃
//┃3、                ┃
//┃┏━━━┓              ┃
//┃┃ stu0 ┣━━┳━━━━━━━━━┳━━━━━━━━━┓      ┃
//┃┗━┳━┛      ┃            ┃          ┃      ┃
//┃┏━┻━┓┏━┻━┓    ┏━┻━┓               ┏━┻━┓      ┃
//┃┃ stu1 ┃┃ stu2 ┣━━┓    ┃ stu3 ┃  ┃ stu4 ┣━━┳━━━━┓  ┃
//┃┗━┳━┛┗━┳━┛      ┃    ┗━┳━┛  ┗━┳━┛       ┃            ┃  ┃
//┃┏━┻━┓┏━┻━┓┏━┻━┓┏━┻━┓  ┏━┻━┓┏━┻━┓┏━┻━┓  ┃
//┃┃ stu5 ┃┃ stu6 ┃┃ stu7 ┃┃ stu8 ┣━━┓  ┃ stu9 ┃┃stu10┃┃stu11┃  ┃
//┃┗━━━┛┗━━━┛┗━━━┛┗━┳━┛      ┃  ┗━━━┛┗━━━┛┗━━━┛  ┃
//┃        ┏━┻━┓┏━┻━┓      ┃
//┃        ┃stu12┃┃stu13┃      ┃
//┃        ┗━━━┛┗━━━┛      ┃
//┃栈结构更适合实现中-右-左的遍历顺序，所以最终遍历结果应当是：      ┃
//┃0-4-11-10-9-3-8-13-12-2-7-6-1-5          ┃
//┃                ┃
//┗━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━┛
/*************************************************************************/


#include <stdio.h>
#include <windows.h>


//元素内容
typedef struct _STU
{
  int ID;
  char* name;
}STU,*PSTU;


//树形结构的节点定义，采用孩子链的定义形式
typedef struct _STU_NODE
{
  _STU_NODE* child;
  _STU_NODE* right;
  PSTU pValue;
}STU_NODE,*PSTU_NODE;


/*************************************************************************/
//遍历算法
//PSTU_NODE top:  树根
//PVOID pFunc:  遍历到树节点后,调用的函数,
//        必须符合 BOOL __stdcall pFun(PSOME_STRUCT pStru)这样的定义
//        其中 PSOME_STRUCT 任意,可以由调用方规定
//        返回 TRUE  表示继续遍历,返回 FALSE 表示终止遍历
void stack_traversal(PSTU_NODE top, PVOID pFunc)
{
  _asm
  {
    push eax
    push ecx

    //压入 0，标志终止
    xor eax,eax
    push eax

    //压入树的头部
    push dword ptr [top]

    next_traversal:
    pop eax //弹出栈中的剩余数据
    test eax,eax
    je traversal_end //退出遍历

    //暂存这个从栈中弹出的目标
    mov ecx,eax

    //压入这个从栈中弹出的目标的第一个孩子
    mov eax,dword ptr [eax]
    test eax,eax
    je no_child //如果没有孩子，跳走
    push eax //压入第一个孩子
    //压入这个从栈中弹出的目标的其余所有孩子
next_right:
    mov eax,dword ptr [eax+4]
    test eax,eax
    je no_right //如果没有右兄弟，跳走
    push eax
    jmp next_right
no_right:
no_child:
    //游历这个从栈中弹出的目标
    mov eax,dword ptr [ecx+8]
    push eax
    call dword ptr [pFunc]
    test eax,eax
    jnz next_traversal//如果遍历函数 pFunc 返回继续
pop_not_zero:
    pop eax
    test eax,eax
    jnz pop_not_zero
traversal_end:
    pop ecx
    pop eax
  }
}
/*************************************************************************/


/*************************************************************************/
//遍历调用函数
BOOL __stdcall print_stu(PSTU pStu)
{
  printf("%d\t%s\n",pStu->ID,pStu->name);

  /*************************************************************************/
  //如果想要终止遍历,可以使用下面的代码
  //if(pStu->ID==7)
  //{
  //  return FALSE;
  //}
  /*************************************************************************/

  return TRUE;
}
/*************************************************************************/


/*************************************************************************/
//main 函数,构造树结构,调用遍历算法
int _tmain(int argc, _TCHAR* argv[])
{
  STU stu[14];

  int i;
  for(i=0;i<14;i++)
  {
    stu[i].ID = i;
  }

  stu[0].name = "stu0";
  stu[1].name = "stu1";
  stu[2].name = "stu2";
  stu[3].name = "stu3";
  stu[4].name = "stu4";
  stu[5].name = "stu5";
  stu[6].name = "stu6";
  stu[7].name = "stu7";
  stu[8].name = "stu8";
  stu[9].name = "stu9";
  stu[10].name = "stu10";
  stu[11].name = "stu11";
  stu[12].name = "stu12";
  stu[13].name = "stu13";

  PSTU_NODE node = new STU_NODE[14];
  for(i=0;i<14;i++)
  {
    node[i].child = 0;
    node[i].right = 0;
    node[i].pValue = &stu[i];
  }

  node[0].child = &node[1];//0-1,2,3,4
  node[1].right = &node[2];
  node[2].right = &node[3];
  node[3].right = &node[4];

  node[1].child = &node[5];//1-5

  node[2].child = &node[6];//2-6,7
  node[6].right = &node[7];

  node[3].child = &node[8];//3-8

  node[4].child = &node[9];//4-9,10,11
  node[9].right = &node[10];
  node[10].right = &node[11];

  node[8].child = &node[12];//8-12,13
  node[12].right = &node[13];

  stack_traversal(&node[0],print_stu);

  delete[] node;

  return 0;
}
/*************************************************************************/
```

## 打印输出

```
0       stu0
4       stu4
11      stu11
10      stu10
9       stu9
3       stu3
8       stu8
13      stu13
12      stu12
2       stu2
7       stu7
6       stu6
1       stu1
5       stu5
```

`stack_traversal2.rar`
