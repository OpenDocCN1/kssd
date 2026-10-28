
# [求助]C语言内联汇编如何定义返回？

请问如何解决 2 warning(s) 这个问题？

`int add(int,int);` 和 `int sub(int,int);` 这两个函数无法用 `void` 类型，如果用 `int` 类型那么没办法 `return`，不 `return` 的话程序可以正常运行，但是下面的 2 个警告怎么解决？我想知道能使这个程序完美的方法。

```cpp
#include <iostream>
using namespace std;

int add(int,int);
int sub(int,int);

int main()
{
    int a,b;
    cout<<"Pleash input a and b:"<<endl;
    cin>>a>>b;
    cout<<"A-B="<<sub(a,b)<<endl;
    cout<<"A+B="<<add(a,b)<<endl;
    return 0;
}

int sub(int a,int b)
{
    __asm
    {
        mov eax,a
        sub eax,b
    }
}

int add(int a, int b)
{
    __asm
    {
        mov eax,a
        add eax,b
    }
}
```

```text
D:\C++\Test\Test.cpp(25) : warning C4035: 'sub' : no return value
D:\C++\Test\Test.cpp(34) : warning C4035: 'add' : no return value
Test.obj - 0 error(s), 2 warning(s)
```

[\[招生\]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
