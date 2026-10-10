

# 贴一个恢复 CrypKeyInstant 被抽代码工具源码

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-524.htm`](https://www.pediy.com/kssd/pediy09/pediy09-524.htm)

> 第一次有耐心脱壳，脱完后，才在这里知道是 CrypKeyInstant，我脱的版本到底是多少这就不太清楚![](img/d40b5575a143b1b5e17e7f31e3f35d26.jpg "恐怖!") ，但是和很下面这个链接上的讨论很像：双进程，大量 cc，抽代码。  
> http://bbs.pediy.com/showthread.php?t=33856  
>   
> 这个壳代码很清晰，并且抽出代码没有加密，所以搞起来也比较容易，没有多少技术含量，高手可以略过。![](img/d40b5575a143b1b5e17e7f31e3f35d26.jpg "恐怖!") ![](img/d40b5575a143b1b5e17e7f31e3f35d26.jpg "恐怖!")   
>   
>   
> /////////////////////////////////////////////////////////////////////////////////////////////////////  
> // 写的很乱, 写的时候很有针对性。  
> // 如果要要改, 把 protected\_module\_table  改为发生异常后第一个循环查找的 call 的返回值  
> // ![](img/d40b5575a143b1b5e17e7f31e3f35d26.jpg "恐怖!") ![](img/d40b5575a143b1b5e17e7f31e3f35d26.jpg "恐怖!")   
>    
> #include <windows.h>  
> #include <stdio.h>  
>   
> //char\* protected\_module\_table = (char\*)0x4034B0;  
> //   
> typedef unsigned long u32;  
>   
> #include <PSHPACK1.H>  
> struct trap\_record   
> {  
>   char  attr; //   
>   u32    trap\_addr;  
>   u32    patch\_code\_record\_off;  
>   u32    patch\_code\_record\_size;  
> };  
> #include <poppack.h>  
>   
> // struct patch\_code\_record   
> // {  
> //   char is\_used;  
> // //  char patch\_code\[1\];  
> // };  
>   
> //base --> 0x20  
> //first --> 0x48aea  
>   
> HANDLE \_\_hProcess1, \_\_hProcess2;  
> u32 \_\_trap\_hash\_table;  
> void\* \_\_big\_buf = NULL;  
>   
> void \_\_fastcall xf(u32 p\_tr)  
> {  
>   trap\_record tr;  
>   u32 tmp;  
>   
>   if (   
>     !ReadProcessMemory(\_\_hProcess1, (void\*)p\_tr, &tr, sizeof (tr), &tmp)   
>     )  
>   {  
>     printf("读取地址 %08X 失败!\\n", p\_tr);  
>     return ;  
>   }  
>     
>   if ( tr.attr == 0 )  
>   {  
>     xf(tr.patch\_code\_record\_off + \_\_trap\_hash\_table);  
>     xf(tr.patch\_code\_record\_size + \_\_trap\_hash\_table);  
>   }  
>   else  
>   {  
>     printf(  
>       "trap\_record.attr                   = 0x%08X\\n"  
>       "trap\_record.trap\_addr              = 0x%08X\\n"  
>       "trap\_record.patch\_code\_record\_off  = 0x%08X\\n"  
>       "trap\_record.patch\_code\_record\_size = 0x%08X\\n"  
>       "-------------------------------------------------------\\n",  
>       tr.attr,   
>       tr.trap\_addr,   
>       tr.patch\_code\_record\_off,   
>       tr.patch\_code\_record\_size  
>       );  
>   
>       ReadProcessMemory(\_\_hProcess1, (PVOID)(\_\_trap\_hash\_table + tr.patch\_code\_record\_off + 1), \_\_big\_buf, tr.patch\_code\_record\_size, &tmp);  
>       WriteProcessMemory(\_\_hProcess2, (PVOID)tr.trap\_addr, \_\_big\_buf, tr.patch\_code\_record\_size, &tmp);  
>       FlushInstructionCache(\_\_hProcess2, (PVOID)tr.trap\_addr, tr.patch\_code\_record\_size);  
>   }  
> }  
>   
> int main(int argc, char\* argv\[\])  
> {  
>   DWORD dwPid1, dwPid2;  
>   
>   printf("请分别输入调试和被调试进程的 PID: \\n");  
>   scanf("%d %d", &dwPid1, &dwPid2);  
>     
> //   BOOL bOK = DebugActiveProcess(dwPid1); // 暂停调试器  
> //   if ( !bOK )  
> //   {  
> //     printf("绑定 %d 进程失败!\\n", dwPid1);  
> //     return -1;  
> //   }  
> //     
> //   DEBUG\_EVENT de;  
> //   if ( !WaitForDebugEvent(&de, 5000) )  
> //   {  
> //     printf("等待 %d 进程的调试事件失败!\\n");  
> //     return -2;  
> //   }  
> //    
> //  HANDLE hProcess1 = de.u.CreateProcessInfo.hProcess;  
>   
>   // 最好在打开这两个进程的之前已经暂停  
>   HANDLE hProcess1 = OpenProcess(PROCESS\_VM\_READ | PROCESS\_VM\_OPERATION, FALSE, dwPid1);   
>   HANDLE hProcess2 = OpenProcess(PROCESS\_VM\_WRITE | PROCESS\_VM\_OPERATION, FALSE, dwPid2);  
>     
>   \_\_hProcess1 = hProcess1;  
>   \_\_hProcess2 = hProcess2;  
>   
>   if ( hProcess2 == NULL || hProcess1 == NULL )  
>   {  
>     printf("打开 %d 进程进行写操作失败!\\n");  
>     return -3;  
>   }  
>     
>   u32 \*index\_table = NULL;  
>   \_\_try  
>   {  
>     u32 tmp;  
>     u32 a\[3\];  
>   
>     ReadProcessMemory(hProcess1, (PVOID)0x4034B0, a + 0, 4, &tmp);  
>     ReadProcessMemory(hProcess1, (PVOID)(a\[0\] + 0x18), a + 1, 4, &tmp);  
>      ReadProcessMemory(hProcess1, (PVOID)(a\[0\] + 0x38), a + 2, 4, &tmp);  
>       
>     \_\_big\_buf = VirtualAlloc(NULL, 1024 \* 1024, MEM\_COMMIT, PAGE\_READWRITE);  
>     \_\_trap\_hash\_table = a\[2\];  
>   
>     xf(a\[1\] + a\[2\]);  
>   }  
>   \_\_except ( EXCEPTION\_EXECUTE\_HANDLER )  
>   {  
>     printf("发生了一个异常!\\n");  
>   }  
>     
>   if ( \_\_big\_buf != NULL )  
>   {  
>     VirtualFree(\_\_big\_buf, 1024 \* 1024, MEM\_DECOMMIT);  
>     VirtualFree(\_\_big\_buf, 0, MEM\_RELEASE);  
>   
>     \_\_big\_buf = NULL;  
>   }  
>   
>   printf("ok Dump!!\\n");  
>   scanf("%d", &dwPid1);  
>   
>   return 0;  
> } 

 4dd466d0e1e0cc123afbe504da2462ca