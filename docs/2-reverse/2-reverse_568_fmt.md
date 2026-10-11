
# C 与 C++ 函数调用约定之间的转换

> 原文：[`www.pediy.com/kssd/pediy11/108111.html`](https://www.pediy.com/kssd/pediy11/108111.html)

C 语言的初学者常犯的一个失误是调用系统或第三方 API 时忘了在函数声明中标注 `WINAPI` 调用方式，而菜鸟 C++ 程序员也每每困惑为什么成员函数必须指示为 `CALLBACK` 再添加 `static` 关键字才能作为回调函数。在 x86 系统中，存在多种函数调用约定。如果调用者与被调用者采用不同的调用约定，很可能导致堆栈破坏、非法访问等致命错误。

也许您会得出一个结论，除非借助汇编指令，否则调用约定之间的转换是不可能的。不过很多事情都不是绝对的。我们先看下面的例子：

```c
#include <stdio.h>

int __cdecl CDeclFunction()
{
  printf("From CDecl function\n");
  return 1;
}

int main()
{
  printf("Begin call STDCall function\n");
  typedef int(__stdcall * STDCALLFUNCTION)();
  STDCALLFUNCTION pfn = (STDCALLFUNCTION)CDeclFunction;
  int i=pfn();
  printf("End call STDCall function\n");
  return i;
}
```

这是一个简单的函数调用的例子，唯一特别的地方是函数定义为`__cdecl`，而调用时采用`__stdcall` 方式。我们把这段代码编译后执行。嗬，一切正常。如果您不服气，再加一个 `for` 循环看看。

我们再来看另一个更特别的例子：

```c
#include <stdio.h>

int __stdcall StdCallFunction (int i, char* pszString)
{
  printf("From STDCall function\n");
  printf(pszString);
  return i;
}

int main()
{
  printf("Begin call FastCall function\n");
  typedef int(__fastcall * FASTCALLFUNCTION)(int, int, int, char*);
  FASTCALLFUNCTION pfn = (FASTCALLFUNCTION)StdCallFunction;
  int i=pfn(0,0,3,"test\n");
  printf("End call FastCall function\n");
  return i;
}
```

这次不光调用方式不同，连参数的个数都不一样。别担心，我们编译后执行。还是一切正常！

如果您对上述两个例子百思不得其解的话，现在该是解开谜底的时候了。这个谜底就是函数调用约定本身。网上介绍调用约定的资料已经相当多了，不过几乎都侧重比较各种调用方式的差别，而本文将把重点放在这些调用方式之间的联系上面（这里也略去诸多与主题无关的细节，要想了解函数调用约定的所有方式，相应的汇编代码，以及在参数或返回值超过 32 位等复杂情况的处理方式，请查阅本文列出的参考书目）。

调用函数时，需要传递参数，并接收返回值。对于 C++ 非静态成员函数，还要考虑如何传递 `this` 指针。这要么采用寄存器，要么采用堆栈。如果采用堆栈，还要考虑由谁负责恢复堆栈的平衡。调用约定即是调用者和被调用者之间传递参数和接收返回值的规范。对于 C 语言中的函数和 C++ 语言中的全局函数、静态成员函数，常用的调用约定有 C 调用约定（用`__cdecl` 关键字指示，默认编译环境下可省略该关键字）、标准调用约定（用`__stdcall` 关键字或 `WINAPI`、`CALLBACK` 宏指示）、快速调用约定（用`__fastcall` 关键字指示）3 种。这三种方式都使用 `EAX` 寄存器接收返回值，且参数都是按从右到左的顺序压入堆栈，先压最后一个参数。其中，C 调用是调用者收回参数压栈占用的空间，这样可支持变长参数；而标准调用是被调用函数自己负责恢复堆栈的平衡；快速调用也是被调用函数清理堆栈，但函数的前两个参数通过 `ECX` 和 `EDX` 寄存器传递。

对于 C++ 非静态成员函数，默认采用 `THIS` 调用约定（不需要关键字指示）。该方式除用 `ECX` 寄存器传递对象的 `this` 指针外，其他方面同标准调用。另外一种资料上较少提及的非静态成员函数调用方式为 C++ 标准调用（同样用`__stdcall` 关键字指示），这种方式把 `this` 指针视为第一个参数，其他方面完全按 C 函数标准调用约定处理。

在上述 5 种调用方式中，完全相同的地方是都使用 `EAX` 寄存器传递返回值，这扫除了彼此转换的第一个障碍。其次，较一致的地方是除了 C 调用外，其他调用约定都由被调用函数负责恢复堆栈平衡，这意味着在转换这些调用方式时，只需关注如何传递参数（及 `this` 指针，对于 C++ 非静态成员函数的话）就可以了。对于 C 调用与其他调用约定的转换，堆栈的平衡便成为其无法逾越的鸿沟。但当函数没有参数时，不需要考虑参数压栈引起的堆栈平衡问题，C 调用与其他调用约定之间的转换便成为可能。这正是第一个例子正常运行的原因所在。当包含参数时，C 调用与其他调用约定之间的转换要么借助汇编指令，要么采用“对等补偿”的方式（不推荐）。如下面的代码所示：

```c
#include <stdio.h>

int __cdecl CDeclFunction(int a,char* pszString)
{
  printf("From CDecl function\n");
  printf(pszString);
  return a;
}

int __stdcall StdCallFunction (int i)
{
  printf("From STDCall function\n");
  return i;
}

int main()
{
  printf("Begin call STDCall function\n");
  typedef int(__stdcall * STDCALLFUNCTION)(int,char*);
  STDCALLFUNCTION pfn1 = (STDCALLFUNCTION)CDeclFunction;
  int i=pfn1(2,"test\n");
  printf("End call STDCall function\n");
  
  printf("Begin call CDecl function\n");
  typedef int(__cdecl * CDECLFUNCTION)(int);
  CDECLFUNCTION pfn2 = (CDECLFUNCTION)StdCallFunction;
  int j=pfn2(3);
  int k=pfn2(4);
  printf("End call CDecl function\n");
  
  return i+j+k;
}
```

以上代码要正常运行，需关闭编译器的堆栈帧检查功能（取消`/RTCs` 编译选项）。在我使用的 Visual C++ 2005 开发环境中，在项目的属性页，选择“代码生成”，把“基本运行时检查”由“两者”改为“未初始化的变量”。

现在我们撇开 C 调用，专注于其他几种调用方式之间的转换。如前所述，我们只需考察参数是如何传递的。如果把 C++ 非静态成员函数的 `this` 指针看作是第一个参数，我们会发现这些调用约定更为相似。当用堆栈压参数时，参数在堆栈中的顺序是一致的，只要能定位其中一个参数，就可以定位堆栈上的其他参数。标准调用和 C++ 标准调用都使用堆栈传递所有参数，这意味着两种调用方式可以直接相互转换。即形如 `RETURN_TYPE __stdcall NonStaticFunction (Type1 param1, Type2 param2... Typen paramn)` 的 C++ 非静态成员函数与形如 `RETURN_TYPE __stdcall StaticFunction (PVOID pThis, Type1 param1, Type2 param2... Typen paramn)` 的全局函数或静态成员函数是等价的。

其他两种调用方式使用寄存器传递前面的一两个参数：`THIS` 调用约定用 `ECX` 寄存器传递第一个参数（即 `this` 指针）；快速调用约定用 `ECX` 和 `EDX` 寄存器传递前两个参数。快速调用的第三个参数，与 `THIS` 调用的第二个参数（不考虑 `this` 指针则为第一个参数），与标准调用的第一个参数，在堆栈上的位置是相同的。我们无法用 C/C++ 语言直接操纵寄存器，不过可以忽略寄存器。通过增加虚拟的用寄存器传递的参数，我们可以用 `THIS` 调用模拟标准调用，或用快速调用模拟 `THIS` 调用和标准调用。即在下列 3 个函数原型中，后者可以模拟前者。本文前面的第二个例子，就是基于这样的原理。

```c
RETURN_TYPE __stdcall StdCallFunction (Type1 param1, Type2 param2... Typen paramn);

class AClass
{
  RETURN_TYPE MemberFunction (Type1 param1, Type2 param2... Typen paramn);
};

RETURN_TYPE __fastcall FastCallFunction (ECXType ecx_param, EDXType edx_param, Type1 param1, Type2 param2... Typen paramn);
```

## 参考书目

* Visual C++ 语言参考手册：`Calling Conventions`。
* Debugging Tools for Windows 联机手册：`x86 Architecture`。
* Debugging Applications for Microsoft .NET and Microsoft Windows：`Chapter 7: Advanced Native Code Techniques with Visual Studio .NET`。`x86 Assembly Language`
