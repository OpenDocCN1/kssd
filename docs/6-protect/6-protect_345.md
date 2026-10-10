

# 利用 SEH 改变程序流程以达到反跟踪的目的

> 原文：[`www.pediy.com/kssd/pediy03/forum1099.htm`](https://www.pediy.com/kssd/pediy03/forum1099.htm)

老东西，在 ASProtect 中大量使用，主要是想隐藏 OEP。当然仅靠这个无法对付冲击波和 icedump 的/tracex，但这两个都是运行在 Win9x 下的。   
  
如果用 SoftICE 跟下面的程序，在执行到 REPZ MOVSD 指令（在 inline 函数 memcpy 的函数体中）时，就会失去线索（petite 似乎也是这样）。除了\_\_try{ }\_\_except，还可以用\_\_try{ } \_\_finally，把这两个结合起来形成多重嵌套可以增加复杂度。   
  
#include <windows.h>   
#include <iostream.h>   
  
#define BUFFER\_SIZE 4096   
  
void \*MemAddr = NULL;   
void AccessViolation( );   
int  OEP( );   
int  Filter( );   
  
int main( )   
{   
    \_\_try   
    {   
        //........   
  
        AccessViolation( );   
  
        //You can insert some garbage code here   
    }   
    \_\_except(Filter( ))   
    {   
        //You can insert some garbage code here   
  
        return OEP( );   
    }   
  
    return 0;   
}   
  
void AccessViolation( )   
{   
        MemAddr = VirtualAlloc(NULL, BUFFER\_SIZE, MEM\_RESERVE | MEM\_COMMIT, PAGE\_READWRITE);   
        memcpy(MemAddr, (void \*)main, BUFFER\_SIZE + 5);   
  
        cout << "Here is some garbage, which will never be executed." << endl;   
        //You can insert some garbage code here   
}   
  
int OEP( )   
{   
    cout << "This is the Original Entry Point." << endl;   
    return 0;   
}   
  
int Filter( )   
{   
    //You can insert some garbage code here   
  
    if (MemAddr)   
    {   
        VirtualFree(MemAddr, BUFFER\_SIZE, MEM\_DECOMMIT | MEM\_RELEASE);   
    }   
  
    //You can insert some garbage code here   
  
    return EXCEPTION\_EXECUTE\_HANDLER;   
}  4dd466d0e1e0cc123afbe504da2462ca