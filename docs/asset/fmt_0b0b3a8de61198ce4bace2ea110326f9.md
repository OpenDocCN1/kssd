
# 直接执行函数地址

> 原文：[`www.pediy.com/kssd/pediy12/140213.html`](https://www.pediy.com/kssd/pediy12/140213.html)

## 汇编代码分析

```
7C8021CE      90            nop
7C8021CF      90            nop
7C8021D0 >    8BFF          mov edi,edi                        ;  ReadProcessMemory
7C8021D2      55            push ebp
7C8021D3      8BEC          mov ebp,esp
7C8021D5      8D45 14       lea eax,dword ptr ss:[ebp+0x14]     我要从这里执行
7C8021D8  .   50            push eax
7C8021D9  .   FF75 14       push dword ptr ss:[ebp+0x14]
7C8021DC  .   FF75 10       push dword ptr ss:[ebp+0x10]
```

不直接调用 `ReadProcessMemory` 因为有可能被 inline hook 呢。

直接写入参数 然后 转到 `ReadProcessMemory+5` 这里执行。

我自己写的老出错。。

不知道为什么。。。

那个函数不好用。。。单次执行啥问题没有，，，但是我大量执行，问题就来了。。总是出错。。大家看看我那里没有写好。

大概执行 5 次就会出错。。。

使用 `test_ReadProcessMemory` 大量执行的时候总出错，，，，但是用 `ReadProcessMemory` 就没问题。

## 初始代码示例

```
vodi test_ReadProcessMemory(  HANDLE hProcess,    LPCVOID lpBaseAddress,    LPVOID lpBuffer,    DWORD nSize,    LPDWORD lpNumberOfBytesRead)

{

  __asm
  {
    mov edi,edi
    push ebp
    mov ebp,esp
    mov eax,ReadProcessMemory
    add eax,5
    jmp eax
  }

}

void main()

{
  char cc[0x1024] ;
  int tmpaddr = 0x400000;
  while(tmpaddr < 0x7FFFFFFF)
  {
    tmpaddr = tmpaddr + 0x1024;
    test_ReadProcessMemory((LPVOID)-1, (LPCVOID)tmpaddr, (LPVOID)&cc, 1024, 0);
    printf(" %x %x \n" ,  tmpaddr, &cc);
  }
  system("pause");
}
```

## 问题分析

你的代码有几个问题

`tmpaddr` 是带符号整数，所以那个 `while` 永远为真，不过这不是那个错误信息的原因

问题可能在`&cc` 上，`cc` 已经是数组了，不需要这个`&`，有可能是你用的编译器比较老，编译器把`&cc` 编译成了一个独立变量，造成堆栈错误，导致 `tmpaddr`，以及堆栈中的 0 被重写了，看那个地址似乎是函数在尝试写入 `lpNumberOfBytesRead`

另外还有个不是问题的问题，数组长度一会 1024， 一会 0x1024，统一起来比较好

拿 vc6 编译了一下，是堆栈被重写导致的，但前面关于`&cc` 的猜测是错的，而是因为 `test_ReadProcessMemory` 是个普通函数，你需要再加一个 `WINAPI` 的声明，否则 VC6 会在函数调用后调整 ESP（5 个参数，所以加了 0x14）

## 修正代码

```
__declspec(naked) WINAPI void test_ReadProcessMemory(  HANDLE hProcess,    LPCVOID lpBaseAddress,    LPVOID lpBuffer,    DWORD nSize,    LPDWORD lpNumberOfBytesRead)
```

这么写就不会有问题了

引用: 最初由 `kissxrl` 发布 我用 `ReadProcessMemory` 就可以的。。

但是用 `test_ReadProcessMemory` 就不行，，，大概循环几次就 出错呢。

因为 `test_ReadProcessMemory` 每次调用结束后 esp 都会增加 0x14, 你改成如下声明就没事了

```
__declspec(naked) WINAPI void test_ReadProcessMemory(  HANDLE hProcess,    LPCVOID lpBaseAddress,    LPVOID lpBuffer,    DWORD nSize,    LPDWORD lpNumberOfBytesRead)
```

## 最终结果与注意事项

终于可以了。。感谢楼上所有大虾们。。。我基础太差了。。。

不过加了 `WINAPI` 就有这个提示。。。编译器提示 不过程序可以正常运行。

```
__declspec(naked) WINAPI void test_ReadProcessMemory(  HANDLE hProcess,    LPCVOID lpBaseAddress,    LPVOID lpBuffer,    DWORD nSize,    LPDWORD lpNumberOfBytesRead)

Compiling...

tcp.cpp

d:\program files\vc6\myprojects\tcp\tcp.cpp(379) : warning C4518: 'void ' : storage-class or type specifier(s) unexpected here; ignored

d:\program files\vc6\myprojects\tcp\tcp.cpp(379) : warning C4230: anachronism used : modifiers/qualifiers interspersed, qualifier ignored

Linking...
```

抱歉是我疏忽了，

把 `void` 去掉或是放在 `WINAPI` 前面
