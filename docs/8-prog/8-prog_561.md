
# 自己归纳的 VC 编程细节

> 原文：[`www.pediy.com/kssd/pediy10/80568.html`](https://www.pediy.com/kssd/pediy10/80568.html)

都是一些比较细节的东西，有没有用大家看着办吧。

## C/C++技术细节（VC 编译器）

### 1. Jmp stub Function

考虑以下函数：

```
Void Test(){}

Void main()
{ test();}
```

在 debug 情况下，编译后

```
Void Jmp\_Test()
{
    Jmp to Real\_test();
}

Void main()
{jmp\_test()}
```

如何让编译器不产生 Jmp Stub function。

解决方法，在 Link 选项中去掉 `Increment link`.

### 2. 自定义节，数据代码合并在一个节

`#pragma data\_seg(".mydat") //指定该.cpp/.c（其实应该是某个.obj 文件） 数据存放的节`

注意：`Dword a; // 未初始化，数据仍保存在.data 节`

`Dword a=0; //这样 a 就保存到.mydat 节了`

`#pragma code\_seg(".shell")//指定该.cpp/.c（其实应该是某个.obj 文件） 代码存放的节`

`#pragma comment(linker,"/SECTION:.mydat,RWE") //修改节属性`

`#pragma comment(linker,"/SECTION:.shell,RWE")`

`#pragma comment(linker,"/MERGE:.mydat=.shell")//两节合并到.shell 节`

### 3. `__declspec(naked)` 声明的函数不能使用局部变量

### 4. `char[]`

`char test[]=“test”//假定 此处地址为 0x4001000`

```
void main()
{
    printf(“%x”,test);
    printf(“%x”,&test);
}
```

执行以上代码分别输出什么

答案：两个都输出 `0x4001000`,哈哈，没想到吧。

### 5. `char*` 与 `char[]`

`char* test1=”test”;`

`char test2[]=”test”`

考虑下执行该程序会产生什么状况

```
void main()
{
    test1=”new test”;
    test2=”new test”;
}
```

考虑下再看答案

编译后是这样的。

```
atest db ‘test’,0
Test1  dd  offset of attest
Test2 db ‘test’,0
```

所以 `test2=”new test”;` 编译过不了，当然我们仍然能够通过 `memcpy` 给 `test2` 赋值

### 6. 数据对齐

`Word a;`

`Char str[]=”ddddd”;`

```
Void main()
{
  Printf(”%d”,&str-&a);//输出什么，考虑一下
}
```

答案：可能输出 `2`，也可能输出 `4`
