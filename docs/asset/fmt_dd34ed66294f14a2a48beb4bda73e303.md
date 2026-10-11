
# DLL 输出类使用研究手记

> 原文：[`www.pediy.com/kssd/pediy06/pediy6181.htm`](https://www.pediy.com/kssd/pediy06/pediy6181.htm)

贴一篇我以前写的文章，改头换面在杂志上登过，感觉还有点意思。

在写一个程序时，我想使用一个共享软件中的 C++类。该类名为 `Crypt`，封装在一个 DLL 中，文件名为 `Crypt.dll`。通过 SoftIce 和 IDA Pro，我已基本弄清了其成员函数的用法。现在的问题是，没有相应的 `.H` 文件及 `.LIB` 文件（当然更没有源码）。另外，其成员函数显然不能以 `GetProcAddress` 取得地址后直接调用。

该软件是用 Borland C++写的。

先用 Borland C++提供的工具获取必要的文件

```
C:\bc5\bin\impdef Crypt.def Crypt.dll //得到 Crypt.def 文件
C:\bc5\bin\implib Crypt.lib Crypt.dll //得到 Crypt.lib 文件
```

`Crypt.def` 中的相关内容如下：

```
@Crypt@$bctr$qpxc@1; Crypt::Crypt(const char\*)
@Crypt@$bctr$qpxuci@2; Crypt::Crypt(const unsigned char\*,int)
@Crypt@DecodeFrom$qpuct1l @3; Crypt::DecodeFrom(unsigned char\*,unsigned char\*,long)
@Crypt@EncodeTo$qpuct1l @4; Crypt::EncodeTo(unsigned char\*,unsigned char\*,long)
```

从分号后的注释，可以得到 Demangled 后的成员函数原型，但是没有类的定义，我们不知道这个类包含什么数据成员(以及别的未 exported 的成员函数，这一点不重要，因为原来的编程者在使用这个封装在 DLL 中的类时，也只能使用 exported 的函数)。如何构造一个正确的头文件？先来看看原来的代码是如何使用这个类的。以下为 IDA Pro 的输出：

```
00453E3E 0F0push   8
00453E40 0F4lea    eax, [ebp+var_8]
00453E43 0F4push   eax
00453E44 0F8lea    ecx, [ebp+var_E0]
00453E4A 0F8push   ecx
00453E4B 0FCcall   Crypt::Crypt(uchar \*,int)
00453E50 0FCadd    esp, 0Ch
```

这段代码调用 `Crypt::Crypt(uchar \*,int)`成员函数，使用 `__cdecl` 调用规则，由调用者维护堆栈。函数有 2 个参数，向堆栈中压入了 3 个值，最后一个 push 入栈的是指向当前 `Crypt` 对象的 `this` 指针，即变量 `var_E0` 就是在栈上分配的 `Crypt` 类对象。从 IDA Pro 中可看到，`var_E0` 覆盖了从 `FFFFFF20` 到 `FFFFFF80` 共 96 字节的空间。我们知道，C++类的成员函数、静态数据成员是不放在对象内的，对象只含有数据成员（若类中或其基类中定义有虚函数，还包含 `vptr`）。也就是说，`Crypt` 类的所有数据成员共占据 96 字节。具体细节请参照 Stanley Lippman 的《深度探索 C++对象模型》。

由此，我们可以自己定义 `Crypt` 类的数据成员（使用字节数组），使其占据同样的内存空间，与原来的类在内存布局上一致即可。实际上，只要我们给出的类定义保证能分配足够的内存空间，原来的构造函数就可以在分配的内存中创建出正确的对象。这种方法与 COM 的思想有相似之处，都是在二进制的级别上保证内存布局的兼容。我写的头文件如下：

```cpp
class _import Crypt
{
public:
    Crypt(const char* lpszPassword);
    Crypt(const unsigned char* lpszPassword,int cbBuffer);
    EncodeTo(unsigned char* lpSource,unsigned char* lpDestination,int nSize);
    DecodeFrom(unsigned char* lpSource,unsigned char* lpDestination,int nSize);
public:
    char dummy[96];   //Bingo!:-)
};
```

将此头文件及前面的 `Crypt.lib` 文件加入项目，证明此方法是可行的。测试代码如下：

```cpp
Crypt obj("123456");
obj.EncodeTo(lpData,lpData,nSize);
```

以上的尝试都是在 Borland C++下做的，与原来的程序具有同样的环境，如果想在 Visual C++下使用该类又如何实现？从 DLL 中输出类的技术细节是因编译器厂商而异的，显然不能再如法炮制（VC++甚至不能识别用 `implib` 生成的 `Crypt.lib` 文件）。我们可以变通一下，自己用 VC++写一个 `Crypt.dll`,包裹在原来的 DLL 外面，输出与原有 DLL 相同名字和序号的函数，用 VC++写的客户程序使用这个 Wrapper DLL，由其再去调用原来的 DLL。这种编写一个包在原有 DLL 外面的动态链接库的方法，相关资料很多，这里不再详细解释。

将原来的 DLL 改名为 `OldDll.dll`。源代码如下：

头文件 `Crypt.h`：

```cpp
#ifdef CRYPT_EXPORTS
#define CRYPT_API __declspec(dllexport)
#else
#define CRYPT_API __declspec(dllimport)
#endif

class CRYPT_API Crypt {
public:
    Crypt& operator=(const Encrypt& rhs);//赋值运算符，禁止
    Crypt(const Encrypt& rhs);//拷贝构造函数，禁止
public:
    Crypt(const char* lpszPassword);
    Crypt(const unsigned char* lpszPassword,int cbBuffer);
    void __cdecl EncodeTo(unsigned char* lpSource,unsigned char* lpDestination,int nSize);
    void __cdecl DecodeFrom(unsigned char* lpSource,unsigned char* lpDestination,int nSize);
public:
    char dummy[96];   //Bingo!:-)
};
```

实现文件 `Crypt.cpp`：

```cpp
#include "stdafx.h"
#include "Crypt.h"

static HINSTANCE hOldDll=NULL;
static DWORD dwRet;
static DWORD dwRetAddr;
static FARPROC lpCrypt1;//带 1 个参数的构造函数
static FARPROC lpCrypt2;//带 2 个参数的构造函数
static FARPROC lpEncodeTo;
static FARPROC lpDecodeFrom;

BOOL APIENTRY DllMain( HANDLE hModule, DWORD ul_reason_for_call, LPVOID lpReserved )
{
    BOOL bRet=false;
    switch (ul_reason_for_call)
    {
        case DLL_PROCESS_ATTACH:
            //加载原来的 DLL，获取原函数地址
            hOldDll=LoadLibrary("c:\\test\\OldDll.dll");
            if(hOldDll)
            {
                lpCrypt1=::GetProcAddress(hOldDll,MAKEINTRESOURCE(0x1));
                lpCrypt2=::GetProcAddress(hOldDll,MAKEINTRESOURCE(0x2));
                lpEncodetTo=::GetProcAddress(hOldDll,MAKEINTRESOURCE(0x3));
                lpDecodeFrom=::GetProcAddress(hOldDll,MAKEINTRESOURCE(0x4));
                bRet=true;
            }
            break;
        case DLL_THREAD_ATTACH:
            break;
        case DLL_THREAD_DETACH:
            break;
        case DLL_PROCESS_DETACH:
            if(hOldDll)
            {
                ::FreeLibrary(hOldDll);
            }
            break;
    }
    return bRet;
}

__declspec(naked) Crypt::Crypt(const char *lpszPassword)
{
    //手工模仿__cdel 调用规则
    _asm
    {
        pop eax//弹出并保存返回地址
        mov dwRetAddr,eax
        push ecx//压 this 指针入栈
        call lpCrypt1 //调用原函数
        mov dwRet,eax //保存调用返回值
        mov eax,dwRetAddr
        push eax//重新压返回地址入栈
        mov eax,dwRet //恢复调用返回值
        ret 8 //返回，丢弃 2 个 dword(参数和 this 指针)
    }
}

__declspec(naked) Crypt::Crypt(const unsigned char* lpszPassword,int cbBuffer)
{
    //手工模仿__cdel 调用规则
    _asm
    {
        pop eax
        mov dwRetAddr,eax
        push ecx
        call lpCrypt2
        mov dwRet,eax
        mov eax,dwRetAddr
        push eax
        mov eax,dwRet
        ret 0xC //丢弃 3 个 dword
    }
}

void __declspec(naked) __cdecl Crypt::EncodeTo(unsigned char* lpSource, unsigned char* lpDestination,int nSize)
{
    //直接跳转到原函数
    _asm jmp far dword ptr lpEncodeTo
}

void __declspec(naked) __cdecl Crypt::DecodeFrom(unsigned char* lpSource, unsigned char* lpDestination,int nSize)
{
    //直接跳转到原函数
    _asm jmp far dword ptr lpDecodeFrom
}
```

有几处需要注意：首先，这里使用了 `naked` 调用规则（Borland C++不支持），以便于直接操作堆栈及用内嵌的汇编语言编程。另外，虽然我们的类中并没有包含虚函数或对象成员，VC++编译器却仍生成了 member-wise 的拷贝构造函数和 bit-wise 赋值运算符，并导致原来的 DLL 中的构造函数不能正确创建对象。为了禁止编译器自动生成不必要的代码，在头文件中定义了赋值运算符和拷贝构造函数，但并未提供实现。两个构造函数有点特殊，我发现无论指定何种调用规则，生成的代码总是使用 `thiscall` 调用规则，即在 `ecx` 寄存器中传递 `this` 指针，为此构造函数需要特殊处理，用汇编代码手工模仿 `__cdecl` 调用规则去调用原来 DLL 中的函数，包括维护栈指针。

其余的代码已作了注释，易于理解，不再赘述。

`4dd466d0e1e0cc123afbe504da2462ca`
