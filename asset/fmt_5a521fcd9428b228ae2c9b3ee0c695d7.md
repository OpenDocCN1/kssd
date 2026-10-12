
# VC6 中直接调用成员类函数地址（`__thiscall` 函数）的方法

> 原文：[`www.pediy.com/kssd/pediy10/91391.html`](https://www.pediy.com/kssd/pediy10/91391.html)

有些函数调用方式：

代码:

```assembly
mov ecx, PTR
push c
push b
push a
call xxxx
```

这是典型的 `__thiscall`。

但我用的 VC6，不能声明 `__thiscall`（不知高级版本 vc 能声明 `__thiscall` 么？），编译器报错：`'__thiscall' keyword reserved for future use`。我又想不用内联汇编。

怎么办呢？

网上查了下，有方法（参见：`http://www.vckbase.com/document/viewdoc/?id=1818`），我又不是很喜欢，而且这个方法应该是有问题的。

下面介绍我的方法：

一个函数：

代码:

```c
typedef float (__thiscall*) t_WWWdotMOKOdotCC(PVOID ptr, char* a, int b, int c);
t_WWWdotMOKOdotCC WWWdotMOKOdotCC

void main()
{
    WWWdotMOKOdotCC = 0xFFFFFFFF;
    WWWdotMOKOdotCC(thisptr,"是个好网站",b,c);
}
```

显然 VC6 编译不通过。报错:

不能声明 `__thiscall`（编译器报错：`'__thiscall' keyword reserved for future use`。）

解决办法：

代码:

```c
typedef float (__stdcall*) t_WWWdotMOKOdotCC(char* a, int b, int c);
t_WWWdotMOKOdotCC WWWdotMOKOdotCC

_declspec(naked) float __stdcall this_WWWdotMOKOdotCC(PVOID ptr, char* a, int b, int c)
{
_asm{
     pop eax
     pop ecx
     push eax
     JMP WWWdotMOKOdotCC
}
}

void main()
{
    WWWdotMOKOdotCC = 0xFFFFFFFF;
    this_WWWdotMOKOdotCC(thisptr,"是个好网站",b,c);
}
```

这样就可以了。

`pop eax`  
`pop ecx`  
`push eax`

将最后一个参数存到 `ecx` 并从 stack 中移除。

这个问题的详细简介，网上转的：

引用: 我们知道，成员函数和普通函数最大的区别就是成员函数包含一个隐藏的参数 `this` 指针，用来表明成员函数当前作用在那一个对象实例上。根据调用约定（Calling Convention）的不同，成员函数实现 `this` 指针的方式也不同。如果使用 `__thiscall` 调用约定，那么 `this` 指针保存在寄存器 `ECX` 中，VC 编译器缺省情况下就是这样的。如果是 `__stdcall` 或 `__cdecl` 调用约定，`this` 指针将通过栈进行传递，且 `this` 指针是最后一个被压入栈的参数，相当于编译器在函数的参数列表中最左边增加了一个 `this` 参数。

这里还有件事不得不提，虽然 vc 将 `__thiscall` 类型作为成员函数的默认类型，但是 vc6 却没有定义 `__thiscall` 关键字!如果你使用 `__thiscall` 来定义一个函数，编译器报错：`'__thiscall' keyword reserved for future use`。

知道这些就好办了，我们只要根据不同的调用约定，准备好 `this` 指针，然后象普通函数指针一样的使用成员函数地址就可以了。
