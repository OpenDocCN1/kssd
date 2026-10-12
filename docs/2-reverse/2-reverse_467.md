
# 调用约定解释

> 原文：[`www.pediy.com/kssd/pediy12/141028.html`](https://www.pediy.com/kssd/pediy12/141028.html)

在长时间辛苦的学习 C++的 Windows 编程的过程中，你可能有时候有点好奇，某个时候在函数声明前会出现奇怪的符号，像 `__cdecl`, `__stdcall`, `__fastcall`, `WINAPI`, 等等，之后通过 MSDN 或其他一些参考，你可能会发现，这些符号是用于指定函数的调用约定，在这篇文章中，我将可能尝试解释 VC++不同的调用约定(也可能是其他 Windows 平台的 C/C++编译器中使用的约定)，我在这里强调一下，如果你想编写可移植代码，就不能使用这些调用约定。

你可能会问:声明是调用约定?当一个函数被调用的时候，通常会传递参数给它和检查返回值；那么，调用约定就描述了如何传递参数给函数和检查函数的返回值，它还指定了函数名是哪种声明；写优秀的 C++程序真的有必要了解调用约定吗；不是，但是它可能会对调试很有帮助；此外它是连接 C/C++和汇编代码必须使用的。

为了理解本文章，你需要一些很基本的汇编语言程序设计知识。

使用调用约定的时候可能会发生一些下面的情况:

1. 所有参数扩大到 4 字节(当然在 Win32 平台上)，并且储存在合适的内存位置，这个位置筒仓是栈，但也有可能是寄存器，它都是通过调用约定指定的。
2. 程序执行跳转到被调用函数的地址
3. 在函数内部，保存 `ESI`,`EDI`,`EBX` 和 `EBP` 到栈，执行这些操作代码的部分，被称为函数过程，一般是由编译器生成
4. 函数指定的代码被执行，并把返回值的放在 `EAX` 寄存器内
5. 从堆栈中恢复 `ESI`,`EDI`,`EBX` 和 `EBP` 寄存器，这段代码称为功能终止，与函数过程相对应，在大多数情况下，一般是由编译器生成。
6. 从堆栈中移除参数，这个操作称为清理堆栈，可能是调用函数内部执行或调用方执行，这个根据调用约定处理

调用约定的一个例子，我们将会使用一个简单的函数:

代码:

```
int sumExample (int a, int b)
{
    return a + b;
}
```

这个函数调用如下:代码:

```
int c = sum (2, 3);
```

关于 `__cdecl`, `__stdcall`, `__fastcall` 几种调用约定，我编译的示范代码是基于 C 语言的(不是 C++)函数声明，在后面的文章中提到 C 语言用能到的声明方式，C++的声明方式超出了这篇文章的范围。

### C 调用约定(`__cdecl`)

这个调用约定是 C/C++默认的(编译选项指定 `/GD`)，如果一个工程是使用的其他调用约定，我们仍然可以声明一个函数使用 `__cdecl` 调用约定：代码:

```
int __cdecl sumExample (int a, int b);
```

`__cdecl` 调用约定的主要特点是：

1. 参数从右到左传递到堆栈中
2. 堆栈的清理时有调用者执行
3. 声明函数名前面加上一个下划线 `"_"`

现在让我们看下 `__cdecl` 调用的一个例子：

代码:

```
; // 参数从右到左传递到堆栈中
push        3
push        2

; // 调用函数
call        _sumExample

; // 通过增加 ESP 寄存器大小清理堆栈参数
add         esp,8

; // 复制返回值到一个局部变量 (int c)
mov         dword ptr [c],eax
```

被调用的函数，如下所示:代码:

```
; // function prolog
push        ebp
mov         ebp,esp
sub         esp,0C0h
push        ebx
push        esi
push        edi
lea         edi,[ebp-0C0h]
mov         ecx,30h
mov         eax,0CCCCCCCCh
rep stos    dword ptr [edi]

; //    return a + b;
mov         eax,dword ptr [a]
add         eax,dword ptr [b]

; // function epilog
pop         edi
pop         esi
pop         ebx
mov         esp,ebp
pop         ebp
ret
```

### 标准调用约定(`__stdcall`)

这个是使用于 Win32API 函数的调用约定；事实上 `WINAPI` 只不过是 `__stdcall` 的另一个名称。

代码:

```
#define WINAPI __stdcall
```

我们可以显示的声明一个函数使用 `__stdcall` 调用约定。

代码:

```
int __stdcall sumExample (int a, int b);
```

另外，我们可以使用编译选项 `/Gz` 来指定一些其他调用约定不明确的函数使用此调用约定。

`__stdcall` 调用约定的主要特点如下:

1. 参数从右到左传递到堆栈
2. 堆栈清理由被调用函数执行
3. 函数名前加上一个下划线字符和附加一个 `"@"` 字符和堆栈所需的空间字节。

范例如下：

代码:

```
;
// 参数从右到左传递到堆栈
push        3
push        2
; // 调用函数
call        _sumExample@8
; //  复制返回值到一个局部变量 (int c)
mov         dword ptr [c],eax
```

被调用的函数如下所示:

代码:

```
;
// function prolog goes here (the same code as in the __cdecl example)
; //    return a + b;
mov         eax,dword ptr [a]
add         eax,dword ptr [b]
; // function epilog goes here (the same code as in the __cdecl example)
; // cleanup the stack and return
ret         8
```

`__stdcall` 调用约定，因为堆栈是由被调用函数清理的，使用的空间比 `__cdecl` 更小，其中，必须为每个函数调用生成清理堆栈的代码，另一方面，可变数目的参数（如 `printf()` 的）的函数必须使用 `__cdecl`，因为只有在调用方知道每个函数调用中的参数数量，因此只有调用者可以执行堆栈清理。

### 尽可能快速的调用约定(`__fastcall`)

`__fastcall` 约定表明参数可以放置在寄存器里面，而不是放置在堆栈上面；这减少了函数调用的成本，因为寄存器操作速度比堆栈快。

我们可以显式声明一个函数使用 `__fastcall` 约定，如下所示：

代码:

```
int __fastcall sumExample (int a, int b);
```

我们还可以使用的编译器选项 `/GR` 来指定一些其他的调用约定声明不明确的为 `__fastcall`。

`__fastcall` 调用约定的主要特点是：

1. 前两个函数的参数，需要 32 位或更少被放入寄存器 `ECX` 和 `EDX`。他们的其余部分都推由右至左的堆栈。
2. 所调用函数的参数是从堆栈中弹出。
3. 函数名声明，在函数名前面加上一个 `"@"` 字符和附加一个 `"@"` 的字节数（十进制）的参数所需的空间。

注：微软保留权利更改在未来版本的编译器传递参数的寄存器。

这里是一个例子：代码:

```
; // put the arguments in the registers EDX and ECX
mov         edx,3
mov         ecx,2

; // call the function
call        @fastcallSum@8

; // copy the return value from EAX to a local variable (int c)
mov         dword ptr [c],eax
```

函数代码：代码:

```
// function prolog

push        ebp
mov         ebp,esp
sub         esp,0D8h
push        ebx
push        esi
push        edi
push        ecx
lea         edi,[ebp-0D8h]
mov         ecx,36h
mov         eax,0CCCCCCCCh
rep stos    dword ptr [edi]
pop         ecx
mov         dword ptr [ebp-14h],edx
mov         dword ptr [ebp-8],ecx
; // return a + b;
mov         eax,dword ptr [a]
add         eax,dword ptr [b]
;// function epilog
pop         edi
pop         esi
pop         ebx
mov         esp,ebp
pop         ebp
ret
```

这是调用约定快速，对比下 `__cdecl` 和 `__stdcall`，设置编译器选项 `/GR`，比较执行时间。我没有找到 `__fastcall` 必须比其他调用约定更快的了，但你可能会得出不同的结果。

### Thiscall

`thiscall` 是默认调用 C++类（除了那些带有可变数量的参数）的成员函数调用约定。

`thiscall` 调用约定的主要特点是：

1. 从左向右传递参数放置到堆栈，这个是放在 `ECX` 里面
2. 是由被调用函数的堆栈清理

这个调用约定的例子有点不同。首先，作为 C++而不是 C 编译的代码，我们有一个成员函数的结构，而不是一个全局函数。

```
struct CSum
{
    int sum (int a, int b) {return a+b;}
};
```

函数调用的汇编代码看起来像这样：

```
push        3
push        2
lea         ecx,[sumObj]
call        ?sum@CSum@@QAEHHH@Z          ; CSum::sum
mov         dword ptr [s4],eax
```

这个函数它自己产生下面的代码：

```
push        ebp
mov         ebp,esp
sub         esp,0CCh
push        ebx
push        esi
push        edi
push        ecx
lea         edi,[ebp-0CCh]
mov         ecx,33h
mov         eax,0CCCCCCCCh
rep stos    dword ptr [edi]
pop         ecx
mov         dword ptr [ebp-8],ecx
mov         eax,dword ptr [a]
add         eax,dword ptr [b]
pop         edi
pop         esi
pop         ebx
mov         esp,ebp
pop         ebp
ret         8
```

如果我们现在有一个一个可变数目的参数的成员函数，会发生什么情况？在这种情况下，使用 `__cdecl`，这个是压入堆栈的最后。

原文：During the long, hard, but yet beautiful process of learning C++ programming for Windows, you have probably been curious about the strange specifiers that sometime appear in front of function declarations, like `__cdecl`, `__stdcall`, `__fastcall`, `WINAPI`, etc. After looking through MSDN, or some other reference, you probably found out that these specifiers specify the calling conventions for functions. In this article, I will try to explain different calling conventions used by Visual C++ (and probably other Windows C/C++ compilers). I emphasize that above mentioned specifiers are Microsoft-specific, and that you should not use them if you want to write portable code.

So, what are the calling conventions? When a function is called, the arguments are typically passed to it, and the return value is retrieved. A calling convention describes how the arguments are passed and values returned by functions. It also specifies how the function names are decorated. Is it really necessary to understand the calling conventions to write good C/C++ programs? Not at all. However, it may be helpful with debugging. Also, it is necessary for linking C/C++ with assembly code.

To understand this article, you will need to have some very basic knowledge of assembly programming.

No matter which calling convention is used, the following things will happen:

All arguments are widened to 4 bytes (on Win32, of course), and put into appropriate memory locations. These locations are typically on the stack, but may also be in registers; this is specified by calling conventions.

Program execution jumps to the address of the called function.

Inside the function, registers `ESI`, `EDI`, `EBX`, and `EBP` are saved on the stack. The part of code that performs these operations is called function prolog and usually is generated by the compiler.

The function-specific code is executed, and the return value is placed into the `EAX` register.

Registers `ESI`, `EDI`, `EBX`, and `EBP` are restored from the stack. The piece of code that does this is called function epilog, and as with the function prolog, in most cases the compiler generates it.



# 高级文档排版

Arguments are removed from the stack. This operation is called stack cleanup and may be performed either inside the called function or by the caller, depending on the calling convention used.

As an example for the calling conventions (except for this), we are going to use a simple function:

```c
int sumExample (int a, int b) 
{ 
    return a + b; 
}
```

The call to this function will look like this:

```c
int c = sum (2, 3);
```

For `__cdecl`, `__stdcall`, and `__fastcall` calling conventions, I compiled the example code as C (not C++). The function name decorations, mentioned later in the article, apply to the C decoration schema. C++ name decorations are beyond the scope of this article.

## C calling convention (`__cdecl`)

This convention is the default for C/C++ programs (compiler option `/Gd`). If a project is set to use some other calling convention, we can still declare a function to use `__cdecl`:

```c
int __cdecl sumExample (int a, int b);
```

The main characteristics of `__cdecl` calling convention are:

*   Arguments are passed from right to left, and placed on the stack.
*   Stack cleanup is performed by the caller.
*   Function name is decorated by prefixing it with an underscore character `_`.

Now, take a look at an example of a `__cdecl` call:

```asm
; // push arguments to the stack, from right to left 
push        3     
push        2     
; // call the function 
call        _sumExample 
; // cleanup the stack by adding the size of the arguments to ESP register 
add         esp,8  
; // copy the return value from EAX to a local variable (int c) 
mov         dword ptr [c],eax
```

The called function is shown below:

```asm
; // function prolog 
  push        ebp   
  mov         ebp,esp  
  sub         esp,0C0h 
  push        ebx   
  push        esi   
  push        edi   
  lea         edi,[ebp-0C0h] 
  mov         ecx,30h 
  mov         eax,0CCCCCCCCh 
  rep stos    dword ptr [edi] 
; //    return a + b; 
  mov         eax,dword ptr [a] 
  add         eax,dword ptr [b] 
; // function epilog 
  pop         edi   
  pop         esi   
  pop         ebx   
  mov         esp,ebp 
  pop         ebp   
  ret
```

## Standard calling convention (`__stdcall`)

This convention is usually used to call Win32 API functions. In fact, `WINAPI` is nothing but another name for `__stdcall`:

```c
#define WINAPI __stdcall
```

We can explicitly declare a function to use the `__stdcall` convention:

```c
int __stdcall sumExample (int a, int b);
```

Also, we can use the compiler option `/Gz` to specify `__stdcall` for all functions not explicitly declared with some other calling convention.

The main characteristics of `__stdcall` calling convention are:

*   Arguments are passed from right to left, and placed on the stack.
*   Stack cleanup is performed by the called function.
*   Function name is decorated by prepending an underscore character and appending a `'@'` character and the number of bytes of stack space required.

The example follows:

```asm
; // push arguments to the stack, from right to left 
  push        3     
  push        2     
; // call the function 
  call        _sumExample@8
; // copy the return value from EAX to a local variable (int c)   
  mov         dword ptr [c],eax
```

The function code is shown below:

```asm
; // function prolog goes here (the same code as in the __cdecl example) 
; //    return a + b; 
  mov         eax,dword ptr [a] 
  add         eax,dword ptr [b] 
; // function epilog goes here (the same code as in the __cdecl example) 
; // cleanup the stack and return 
  ret         8
```

Because the stack is cleaned by the called function, the `__stdcall` calling convention creates smaller executables than `__cdecl`, in which the code for stack cleanup must be generated for each function call. On the other hand, functions with the variable number of arguments (like `printf()`) must use `__cdecl`, because only the caller knows the number of arguments in each function call; therefore only the caller can perform the stack cleanup.

## Fast calling convention (`__fastcall`)

Fast calling convention indicates that the arguments should be placed in registers, rather than on the stack, whenever possible. This reduces the cost of a function call, because operations with registers are faster than with the stack.

We can explicitly declare a function to use the `__fastcall` convention as shown:

```c
int __fastcall sumExample (int a, int b);
```

We can also use the compiler option `/Gr` to specify `__fastcall` for all functions not explicitly declared with some other calling convention.

The main characteristics of `__fastcall` calling convention are:

*   The first two function arguments that require 32 bits or less are placed into registers `ECX` and `EDX`. The rest of them are pushed on the stack from right to left.
*   Arguments are popped from the stack by the called function.
*   Function name is decorated by prepending a `'@'` character and appending a `'@'` and the number of bytes (decimal) of space required by the arguments.

Note: Microsoft have reserved the right to change the registers for passing the arguments in future compiler versions.

Here goes an example:

```asm
; // put the arguments in the registers EDX and ECX 
  mov         edx,3 
  mov         ecx,2 
; // call the function 
  call        @fastcallSum@8 
; // copy the return value from EAX to a local variable (int c)   
  mov         dword ptr [c],eax
```

Function code:

```asm
; // function prolog 
  push        ebp   
  mov         ebp,esp 
  sub         esp,0D8h 
  push        ebx   
  push        esi   
  push        edi   
  push        ecx   
  lea         edi,[ebp-0D8h] 
  mov         ecx,36h 
  mov         eax,0CCCCCCCCh 
  rep stos    dword ptr [edi] 
  pop         ecx   
  mov         dword ptr [ebp-14h],edx 
  mov         dword ptr [ebp-8],ecx 
; // return a + b; 
  mov         eax,dword ptr [a] 
  add         eax,dword ptr [b] 
;// function epilog   
  pop         edi   
  pop         esi   
  pop         ebx   
  mov         esp,ebp 
  pop         ebp   
  ret
```

How fast is this calling convention, comparing to `__cdecl` and `__stdcall`? Find out for yourselves. Set the compiler option `/Gr`, and compare the execution time. I didn't find `__fastcall` to be any faster than other calling conventions, but you may come to different conclusions.

## Thiscall

`thiscall` is the default calling convention for calling member functions of C++ classes (except for those with a variable number of arguments).

The main characteristics of `thiscall` calling convention are:

*   Arguments are passed from right to left, and placed on the stack. `this` is placed in `ECX`.
*   Stack cleanup is performed by the called function.

The example for this calling convention had to be a little different. First, the code is compiled as C++, and not C. Second, we have a struct with a member function, instead of a global function.

```c
struct CSum 
{ 
    int sum ( int a, int b) {return a+b;} 
};
```

The assembly code for the function call looks like this:

```asm
  push        3     
  push        2     
  lea         ecx,[sumObj] 
  call        ?sum@CSum@@QAEHHH@Z            ; CSum::sum 
  mov         dword ptr [s4],eax
```

The function itself is given below:

```asm
  push        ebp   
  mov         ebp,esp 
  sub         esp,0CCh 
  push        ebx   
  push        esi   
  push        edi   
  push        ecx   
  lea         edi,[ebp-0CCh] 
  mov         ecx,33h 
  mov         eax,0CCCCCCCCh 
```



```asm
  rep stos    dword ptr [edi]  

  pop         ecx   

  mov         dword ptr [ebp-8],ecx   

  mov         eax,dword ptr [a]   

  add         eax,dword ptr [b]   

  pop         edi   

  pop         esi   

  pop         ebx   

  mov         esp,ebp   

  pop         ebp   

  ret         8
```

Now, what happens if we have a member function with a variable number of arguments? In that case, `__cdecl` is used, and this is pushed onto the stack last.
