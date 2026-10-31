
# [原创]VC6中直接调用成员类函数地址（`__thiscall`函数）的方法

有些函数调用方式：

```
mov ecx, PTR
push c
push b
push a
call xxxx
```

这是典型的`__thiscall`。

但我用的VC6，不能声明`__thiscall`（不知高级版本vc能声明`__thiscall`么？），编译器报错：`'__thiscall' keyword reserved for future use`。我又想不用内联汇编。

怎么办呢？

网上查了下，有方法（参见：`da9K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4k6U0K9$3u0S2M7$3g2Q4x3X3g2U0L8$3#2Q4x3V1k6V1L8$3y4#2L8h3g2F1N6q4)9J5c8Y4k6A6k6i4N6V1L8$3y4Q4x3V1k6Q4x3@1k6A6k6q4)9K6c8o6p5^5x3e0S2Q4c8f1k6Q4b7V1y4Q4z5o6W2Q4c8f1k6Q4b7V1y4Q4z5p5y4Q4c8e0k6Q4z5o6S2Q4z5e0q4Q4c8e0g2Q4z5p5k6Q4z5o6S2Q4c8e0c8Q4b7U0S2Q4z5p5c8Q4c8e0k6Q4z5e0S2Q4b7f1k6Q4c8e0g2Q4b7V1g2Q4z5o6S2Q4c8e0g2Q4z5e0k6Q4z5f1y4Q4c8e0k6Q4b7f1y4Q4b7e0u0Q4c8f1k6Q4b7V1y4Q4z5p5y4Q4c8e0S2Q4z5o6m8Q4z5p5y4Q4c8e0c8Q4b7U0S2Q4z5e0c8Q4c8e0S2Q4b7V1k6Q4z5e0W2Q4c8e0c8Q4b7U0S2Q4b7f1q4Q4c8e0k6Q4z5e0k6Q4b7U0W2Q4c8e0k6Q4b7U0y4Q4z5e0g2Q4c8e0g2Q4b7V1q4Q4z5e0c8Q4c8e0S2Q4b7f1k6Q4b7e0g2Q4c8e0k6Q4z5e0S2Q4b7f1k6Q4c8e0k6Q4z5f1y4Q4z5o6W2Q4c8e0W2Q4z5e0N6Q4b7f1g2Q4c8e0W2Q4b7e0u0Q4z5e0S2Q4c8e0N6Q4z5f1q4Q4z5o6c8Q4c8e0y4Q4z5o6m8Q4z5o6t1`）。

下面介绍我的方法：

一个函数：

```
typedef float (__thiscall*) t_WWWdotMOKOdotCC(PVOID ptr,char* a, int b, int c);

t_WWWdotMOKOdotCC WWWdotMOKOdotCC

void main()
{
    WWWdotMOKOdotCC = 0xFFFFFFFF;
    WWWdotMOKOdotCC(thisptr,"是个好网站",b,c);
}
```

显然VC6编译不通过。报错：
不能声明`__thiscall`（编译器报错：`'__thiscall' keyword reserved for future use`）。

解决办法：

```
typedef float (__stdcall*) t_WWWdotMOKOdotCC(char* a, int b, int c);

t_WWWdotMOKOdotCC WWWdotMOKOdotCC

_declspec(naked) float __stdcall this_WWWdotMOKOdotCC(PVOID ptr,char* a, int b, int c)
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

```
pop eax
pop ecx
push eax
```

将最后一个参数存到`ecx`并从stack中移除。

这个问题的详细简介，网上转的：

> 我们知道，成员函数和普通函数最大的区别就是成员函数包含一个隐藏的参数`this`指针，用来表明成员函数当前作用在那一个对象实例上。根据调用约定（Calling Convention）的不同，成员函数实现`this`指针的方式也不同。如果使用`__thiscall`调用约定，那么`this`指针保存在寄存器`ECX`中，VC编译器缺省情况下就是这样的。如果是`__stdcall`或`__cdecl`调用约定，`this`指针将通过栈进行传递，且`this`指针是最后一个被压入栈的参数，相当于编译器在函数的参数列表中最左边增加了一个`this`参数。
>
> 这里还有件事不得不提，虽然vc将`__thiscall`类型作为成员函数的默认类型，但是vc6却没有定义`__thiscall`关键字!如果你使用`__thiscall`来定义一个函数，编译器报错：`'__thiscall' keyword reserved for future use`。
>
> 知道这些就好办了，我们只要根据不同的调用约定，准备好`this`指针，然后象普通函数指针一样的使用成员函数地址就可以了。

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
