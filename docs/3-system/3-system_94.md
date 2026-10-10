

# 关于 TLS 编程

> 原文：[`www.pediy.com/kssd/pediy10/78759.html`](https://www.pediy.com/kssd/pediy10/78759.html)

![](img/9d43c9aee695f95b024aafd1b69ffa5a.jpg "伤心")   
  
最近想写一个程序，想在 OD 载入时自己检测 OEP 的第一个字节是否被替换为 CC，  
  
OD 载入后，会在 OEp 处下个 CC,所以要在 OD 在 OEP 处停下之前检测，就像 Execryptor 2.4.1,要调试的话，必须删除 OD 的 one-shot 断点那样，问了很多大牛，都说可以在 TLS 中做手脚，我搜索了坛子，找到一个 Asm 语言的 TLS 例子，没怎么明白，希望有大牛能解释一下~给个 VC++例子或是 Delphi 例子![](img/792ac17acdd6fd3e39fc1aacdcf4c3d9.jpg "124.gif")

代码:

```
#include "stdafx.h"
#include <windows.h>
#include "winbase.h"
#pragma comment(linker, "/INCLUDE:\_\_tls\_used")

int main();

void do\_anti(){
  ExitThread(0);
}

void NTAPI Anti\_CC\_In\_OEP(PVOID h, DWORD reason, PVOID pv){
  if( reason == DLL\_PROCESS\_ATTACH ){
    IMAGE\_DOS\_HEADER \*dos\_head=(IMAGE\_DOS\_HEADER \*)GetModuleHandle(NULL);
    IMAGE\_NT\_HEADERS32 \*nt\_head=(IMAGE\_NT\_HEADERS32 \*)(((DWORD)dos\_head)+((DWORD)dos\_head->e\_lfanew));
    BYTE \*OEP=(BYTE \*)(nt\_head->OptionalHeader.AddressOfEntryPoint+(DWORD)dos\_head);
    if(\*OEP==0xcc){
      do\_anti();
    }
    OEP=(BYTE \*)main;
    if(\*OEP==0xcc){
      do\_anti();
    }
  }
  return;
}

#pragma data\_seg(".CRT$XLB")
  PIMAGE\_TLS\_CALLBACK p\_thread\_callback\[\] = {Anti\_CC\_In\_OEP,0};
#pragma data\_seg()

int main(){
  printf("Hello,World");
}

```

漫谈 TLS\_CallBack：原理、编程、手工感染及检测  
http://www.team509.com/modules.php?name=News&file=article&sid=73 

引用: *最初由 **ben 中文**发布* ![查看帖子](http://bbs.pediy.com/showthread.php?p=584922#post584922) 附件法下，有人能上一下？！ 

机器上有份:)

上传的附件
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || TLS\_CallBack.rar  
 |