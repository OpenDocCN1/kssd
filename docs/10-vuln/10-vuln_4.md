

# 详解栈溢出 附代码

> 原文：[`www.pediy.com/kssd/pediy11/108123.html`](https://www.pediy.com/kssd/pediy11/108123.html)

![](img/45dd044a2743704cbb333715eae4a5f6.jpg "狂笑") 这篇文章感觉很难一蹴而就，因为自己还是一个刚刚入门的新手。  
  
栈溢出，就是针对栈空间的溢出，覆盖后面的内存区域，比如：  
  
stack.c  

代码:

```
#include "stdio.h"
int \_\_cdecl main(int \_\_argc,char\*\* \_\_argv, char\*\* \_\_environ){
  
  char buff\[8\];
  strcpy(buff, \_\_argv\[1\]);
  
  
  return 0;
} 

```

汇编形式为  
代码:

```
push    ebp
mov    ebp, esp
sub    esp, 8


mov    eax, dword ptr \[ebp+c\]  ;\_\_argv
mov    ecx, dword ptr \[eax+4\]  ;\_\_argv\[1\]
push    ecx

lea    edx, dword ptr \[ebp-8\]  ;buff
push    edx

call  \[strcpy\]
add    esp, 8

xor    eax, eax
mov    esp, ebp
pop    ebp
retn

```

我们分析 eip 执行到 call \[strcpy\] 这句时 栈中的状态吧。  
  
SP<---- buff(8) --- ebp(4) --- mainret(4) --- p1(4) --- p2(4) ---- p3(4) -----SS  
  
假如我们以命令: stack.exe abcd1234 来启动进程的话  
执行 strcpy,将会把 abcd1234 复制到 buff 中， 正好 8 个 byte，但是如果我们多输入一个字符呢，比如输入 stack.exe abcd12345  这个时候,abcd1234 将填充 buff 的 8 个字节，而后面的 ascii 字符'5'将填充到栈帧 ebp 中。  
  
如果我们想通过这种溢出机制，修改 mainret，那么我们就要越过栈帧 ebp,或者我们还可以通过溢出来修改 p1,p2,p3 等等父函数的参数哦，或者父父函数的内部变量等等，但这样要就要越过栈帧和返回地址 mainret 了。