

# 利用 windbg 脚本调试简单实例

> 原文：[`www.pediy.com/kssd/pediy12/142841.html`](https://www.pediy.com/kssd/pediy12/142841.html)

之前想写一篇用 windbg 脚本调试的东西,但是一直比较懒,懒得动手,懒得想例子,把时间都放在了游戏上面,所以想法虽好一直没有实施...  
    这次写一个非常简单的利用 windbg 脚本进行动态调试的小文,博君一笑.  
    // Windbgscript.cpp : Defines the entry point for the console application.  
//  
  

代码:

```
#include "stdafx.h"

#include <iostream>
using namespace std;


int ret100()
{
  int x = 2;  //2-3
  x\*=33;
  ++x;

  return x;
}

int add(int x, int y)
{
  x = 0; //多余的
  return x+y;

}


int \_tmain(int argc, \_TCHAR\* argv\[\])
{

  int x = 100;

  int y = add( x, ret100());

  cout << y;

  getchar();
  return 0;
}

```

    上面代码都非常简单,ret100 里面做了一个非常简单的操作,我希望它的返回值是 100,但现在的返回值是 67, 当然改法太多了,我这里假设是初始化时 x 应该初始化为 3.   
    同样 add 里面也有问题,就是对于参数 x 赋值 0,认为它是一条多余的操作, 因为我们的目的是返回 x+y 的结果.  
现在执行结果是:  
  
![](img/11727527ebeed8f0cd6e802c78fbd96f.jpg)  
  
如何利用 windbg 去动态的修改并且不需要我们始终去手动交互是我们的主要目的, 首先肯定是要了解这两个函数的内部实现.  
uf 伺候:  
  
代码:

```
0:001> uf windbgscript!ret100
   11 00401000 55              push    ebp
   11 00401001 8bec            mov     ebp,esp
   11 00401003 51              push    ecx
   12 00401004 c745fc02000000  mov     dword ptr \[ebp-4\],2
   13 0040100b 8b45fc          mov     eax,dword ptr \[ebp-4\]
   13 0040100e 6bc021          imul    eax,eax,21h
   13 00401011 8945fc          mov     dword ptr \[ebp-4\],eax
   14 00401014 8b4dfc          mov     ecx,dword ptr \[ebp-4\]
   14 00401017 83c101          add     ecx,1
   14 0040101a 894dfc          mov     dword ptr \[ebp-4\],ecx
   16 0040101d 8b45fc          mov     eax,dword ptr \[ebp-4\]
   17 00401020 8be5            mov     esp,ebp
   17 00401022 5d              pop     ebp
   17 00401023 c3              ret

```

 mov     dword ptr \[ebp-4\],2  
对应的就是 x=2;的指令, 我们的目的是将 x 赋值 3, 这里改法很多,比如将 ebp-4 处赋值 3 并跳过该指令...  
这里选择最简单的,让该指令执行,再执行下一条指令前修改 x 的值.  
代码:

```
bp windbgscript!ret100 + 0xb " $$ 0xb 代表从函数 ret100 开始处到当前断点位置的偏移
  ed (ebp-4) 3; $$当代码执行到此处时,ebp-4 即代表 x 的内存地址， ed 命令为在制定地指处 赋值
 gc;  $$gc 命令表示脚本执行后，程序会继续执行
"

```

代码:

```
0:001> uf windbgscript!add
   20 00401030 55              push    ebp
   20 00401031 8bec            mov     ebp,esp
   21 00401033 c7450800000000  mov     dword ptr \[ebp+8\],0
   22 0040103a 8b4508          mov     eax,dword ptr \[ebp+8\]
   22 0040103d 03450c          add     eax,dword ptr \[ebp+0Ch\]
   24 00401040 5d              pop     ebp
   24 00401041 c3              ret

```

mov     dword ptr \[ebp+8\],0  
即为那条多余的指令 x=0; 我们的目的是让 cpu 忽略这条指令， 做法是在这条指令被执行前修改 eip 的值，始之跳转到下一条指令处继续执行  
代码:

```
bp windbgscript!add + 0x03 "
 r @eip = @eip+0xa; $$ @符号会让脚本解释程序认为后面跟随的名称代表为寄存器， 节省查找时间，提高效率
 gc;
"

```

加载 windbg 脚本后程序的执行结果：  
![](img/5d8c6f26706bc48f1547f9bea35e7927.jpg)  
  
实在是找不出好的例子来，所以就想出了这么个下三滥的例子，莫笑。。。  
   实际中这么简单的程序谁也不会用 windbg 调试的，编写脚本的时间可能远远比重新编译，部署，调试所消耗的时间更长。  
   个人认为这种脚本还是比较适合用在重现问题比较困难， 尤其是一种场景， 一个进程开始的时候没有任何问题，但是跑一段时间（比如 2-3 个小时）开始不断重复出现一连串的错误（非崩溃），这种场景用 windbg 脚本还是非常合适的。  
   windbg 脚本里的命令非常丰富，基本平时能用到的功能都有了，感兴趣翻翻帮助文档学学还是很不错的。  
  
  谢谢