

# IDA Pro 插件编写

> 原文：[`www.pediy.com/kssd/pediy06/pediy6362.htm`](https://www.pediy.com/kssd/pediy06/pediy6362.htm)

在研究一个采用 Flexlm 进行保护的软件时，当使用了相应版本 Flexlm SDK 的 Sig 文件后，IDA Pro 顺利地识别出了若干符号。美中不足的是，大部分符号（函数名）被做成了 repeatable comment。函数名字仍然是类似 sub\_45A7F8 或 unknown\_libname\_15 的东西。要是能用对应的注释为函数命名，就可以使用相应的插件，实现在 SoftIce 中进行符号化调试。   
  
　　这样的注释太多，手工完成是不明智的。第一想法是用 IDC Script 来做这件事。但尝试了几次，结果总是不如人意。用 IDC 的几个内置函数（如 NextFunction，FindFuncEnd 等等）遍历 IDB 文件中的函数时，不知什么原因，总是有不少函数会被漏掉。另外，在我的 IDA Pro 4.3.0.740a 上，无论是用 IDC Script 的 Message 函数还是插件的 msg 函数在状态窗口中输出信息时，经常出错，象是越界访问数组。代码看起来没什么错，不知道是否有 bug。   
  
　　以下用插件来实现，查找所有具有 library 属性的函数（从 Sig 文件中得到的符号，都有此属性？）。检查函数名，若为 sub\_XXXX 或 unknown\_libname\_XX 形式，则取其 repeatable 注释。若存在，将函数改为此名。注意有的注释不止一行，如该 vendor daemon 中 42C41B 处的注释为：   
  
　 \_reg   
    \_l\_timer\_signal   
  
      在命名函数时，不允许存在回车换行符。对这种情况，只用最后一行。对于重名函数，后续命名时使用递增的函数名，如\_l\_timer\_signal\_1，\_l\_timer\_signal\_2。在调试代码时发现，若要使用存储在由 IDA 分配的内存中的数据时，必须及时保存，否则该数据可能会损坏。   
  
　　我不知道 IDA 的 Script 和 Plugin 有什么区别？请高手指教。好象 Plugin 功能更强，可以完成 Script 的功能，提供了更多的函数。因为可在 VC 中编程，还可使用 IDA SDK 以外的函数。   
  
　　以下的代码针对具体的应用，并不具有通用性，只是探讨 Plugin 的应用。所有的代码都在 run 函数中。直接从 SDK 的 sample 修改而来。经过以下处理，该 vendor daemon 中没有明确命名的函数已所剩无几。   
  
     
#include <ida.hpp>   
#include <idp.hpp>   
#include <bytes.hpp>   
#include <loader.hpp>   
#include <kernwin.hpp>   
#include <name.hpp>   
  
...   
  
int init(void)   
{   
    return PLUGIN\_OK;   
}   
  
  
void run(int arg)   
{   
     func\_t \* pFunc\_t;   
     ea\_t nStart;          //函数地址   
   char szAnswer\[256\];   //原来的函数名   
   char szOrigCmt\[256\];  //函数的 repeatable 注释   
   char szTemp\[256\];   
    char szNewCmt\[256\];   //调整后的注释(重名函数用)   
    char szAsc\[10\];   
  
    int nCounter=0;  //累加   
   char \* lpszCmt=0;    //指向函数注释的指针   
   char \* lpszDelimiter=0;  //分隔符(换行=0xA)   
    bool bRet;   
  
     
    int nNumOfFuncs=get\_func\_qty();  //取函数总数   
  
   for(int i=0;i<nNumOfFuncs;i++)   
   {   
       pFunc\_t=getn\_func(i);   
       if(pFunc\_t->flags & FUNC\_LIB)  //只处理库函数   
     {   
               nStart=pFunc\_t->startEA;   
  
               get\_func\_name(nStart,szAnswer,sizeof(szAnswer));   
  
               if((0==strncmp(szAnswer,"sub\_",4))||   
   (0==strncmp(szAnswer,"unkn",4)))  //函数名以"sub\_"或"unkn"打头吗       
         {   
        szOrigCmt\[0\]=0;   //清当前内容   
           
      lpszCmt=get\_func\_comment(pFunc\_t,true);  //取注释   
  
      if(!lpszCmt) continue;   //无注释   
           
      strcpy(szOrigCmt,lpszCmt);  //保存当前注释   
     
      //检查注释是否不止 1 行   
  
      lpszDelimiter=0;   
       szTemp\[0\]=0;   
  
                     if(lpszDelimiter=strrchr(szOrigCmt,0xA)) //查找换行符   
     {   
            strcpy(szTemp,lpszDelimiter+1);   
           
                          szOrigCmt\[0\]=0;   
            strcpy(szOrigCmt,szTemp);   //用最后 1 行   
      }   
  
      bRet=set\_name(nStart,szOrigCmt);   
       
      nCounter=0;   
          while(0==bRet) //命名失败(函数名已被使用)   
      {   
    szAsc\[0\]=0;   
    \_itoa(++nCounter,szAsc,10);   
               
    szNewCmt\[0\]=0;   
    strcpy(szNewCmt,szOrigCmt);   
    strcat(szNewCmt,"\_");   
    strcat(szNewCmt,szAsc);  //创建递增的函数名   
               
    bRet=set\_name(nStart,szNewCmt);   
                     }   
                }   
          }   
     }   
}   
  
char comment\[\] = "My first plugin";   
  
char help\[\] =   
        "A sample plugin module   
"   
        "   
"   
        "This module shows you how to create plugin modules.   
"   
        "   
"   
        "It does nothing useful - just prints a message that is was called   
"   
        "and shows the current address.   
";   
  
char wanted\_name\[\] = "My first plugin";   
  
char wanted\_hotkey\[\] = "Alt-0";   
  
extern "C" plugin\_t PLUGIN = {   
  IDP\_INTERFACE\_VERSION,   
  0,                    // plugin flags   
  init,                 // initialize   
  
  term,              // terminate. this pointer may be NULL.   
  
  run,                // invoke plugin   
  
  comment,       // long comment about the plugin   
                        // it could appear in the status line   
                        // or as a hint   
  
  help,                 // multiline help about the plugin   
  
  wanted\_name,          // the preferred short name of the plugin   
  wanted\_hotkey         // the preferred hotkey to run the plugin   
};  
 4dd466d0e1e0cc123afbe504da2462ca