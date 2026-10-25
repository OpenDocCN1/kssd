
# 一个猥琐的 SEH’S HOOK[可扩展的]

> 原文：[www.pediy.com/kssd/pediy10/81472.html](https://www.pediy.com/kssd/pediy10/81472.html)

SEH 机制大家都很熟悉，但 TRY-EXCEPT 的 CODE 与 SEH 的组合可能不是很了解，都是未公开的，说得再细也不如，运行调试一下代码便知。全球没有第二家！！！![](img/e71d9210a215d5b66755d038631e0c83.jpg "恐怖!")

希望能抛砖引玉。

源码如下：

```c
#include <windows.h>

#include <stdio.h>

//该结构是 try except 块的向量表（一数组），是未公开的

typedef struct __vect_handler
{
  DWORD vFlag;
  PVOID TryHandler;/*try-except{code}*/
  PVOID NextContiueCode;/*异常处理后的下一条指令地址*/

}vect_handler;

typedef struct __EXCEPTION_LIST
{
  __EXCEPTION_LIST* next;//next 一个 SEH 链
  PVOID             handler;//异常处理程序
  vect_handler*     vecthandler;
  DWORD             IndexVechandler;/*当前处理的 try-except 块号,抛出异常（在 try-except 里） 都会修改它*/

}EXCEPTION_LIST;

char* ff = "try";

DWORD WINAPI handler()
{
  /*突破 except{}里不能定义局部变量，增加局部变量*/
  /*调试器监测*/
  /*加解密 TRY 块代码*/

      __asm pushad
    __asm push 0
    __asm push 0
    __asm push ff
    __asm push 0
    __asm mov eax,MessageBoxA
    __asm call eax
    __asm popad
  //  __asm jmp OldExceptCode 实现该行才算完美 hook
    return EXCEPTION_EXECUTE_HANDLER/*jmp Contiue 函数*/;

};

void   Contiue()
{
        MessageBoxA( 0,"contiue",0,0);
    ExitProcess( 0 );

};

int main( int argn, char** argv )
{
  char k;
  char* v = 0;//&k;
  DWORD prepageprotect;
  DWORD ntry = 0;
  EXCEPTION_LIST* pe;
  EXCEPTION_LIST* currentpe;
  __asm mov eax, fs:[0]
    __asm mov pe, eax
  currentpe = pe;

  BOOL ok = VirtualProtect( &currentpe->vecthandler[0].TryHandler, 8, PAGE_EXECUTE_READWRITE, &prepageprotect );

  currentpe->vecthandler[0].TryHandler = handler;//hook try-except 【0 】block
  currentpe->vecthandler[0].NextContiueCode = Contiue;/*hook try-except 【0】 block 可以调到任何 IP，可自己扩展功能*/

//以下 do-while 为显示 SEH-try 链没啥作用

    do
  {
    printf( "Next:%08x,handler:%08x\n", pe->next, pe->handler );
    if( pe->vecthandler )
    {
      vect_handler* v = pe->vecthandler;
      while( v->vFlag == 0xffffffff )
      {
        printf( "------try:%d,handler:%08x,NextCode:%08x\n", ntry, v->TryHandler, v->NextContiueCode );
        v++;
        ntry++;
      };
    };
    pe = pe->next;
  }while( pe!=(EXCEPTION_LIST*)0xffffffff );
  __try{
    /*ASM 这里会有 mov [ebp-4]，0*///try-except 【0】对应 vecthandler[0]
    *v = 0;//将抛出异常，它已 hooked
  }__except(EXCEPTION_EXECUTE_HANDLER)
  {
                      // 该异常处理将变化为调用 handler 函数
           //异常处理完好后将调用 Contiue 函数;
  };

  __try{
    /*ASM 这里会有 mov [ebp-4]，1*///try-except 【1】对应 vecthandler[1]
    *v = 0;
  }__except(EXCEPTION_CONTINUE_EXECUTION)
  {
  };

  getchar();
  return 0;
}
```
