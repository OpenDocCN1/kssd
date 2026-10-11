
# 从逆向工程的角度来看 C++ (五)

> 原文：[`www.pediy.com/kssd/pediy10/87591.html`](https://www.pediy.com/kssd/pediy10/87591.html)

## (五) C++ 之 I/O 流操作

由于这一章东西比较多，现在仅仅弄一点点开个头，以后在慢慢扩充。

### C++ 代码

```cpp
// Lesson5.cpp : Defines the entry point for the console application.

#include "stdafx.h"

#include <iostream.h>

int main(int argc, char* argv[])
{
int i;

__asm int 3

cin>>i;

float f;

cin>>f;

char c;

cin>>c;

char buf[100];

cin>>buf;

cout<<"i="<<i<<endl;

cout<<"buf="<<buf<<endl;

cout<<flush;

cout<<hex<<"0x"<<i<<endl;

return 0;
}
```

### 汇编代码

```assembly
00401070  > 55                push    ebp
00401071     8BEC             mov     ebp,esp
00401073     81EC B0000000    sub     esp,0B0
00401079     53               push    ebx
0040107A     56               push    esi
0040107B     57               push    edi
0040107C     8DBD 50FFFFFF    lea     edi,dword ptr ss:[ebp-B0]
00401082     B9 2C000000      mov     ecx,2C
00401087     B8 CCCCCCCC      mov     eax,CCCCCCCC
0040108C     F3:AB            rep     stos  dword ptr es:[edi]
0040108E     90               nop
0040108F     8D45 FC          lea     eax,dword ptr ss:[ebp-4]
00401092     50               push    eax                            ; >>的参数
00401093     B9 80464300      mov     ecx,offset Lesson5.cin        ; ecx 传 cin 符号.
00401098     E8 F3E20000      call    Lesson5.istream::operator>>   ; 被重载的>>
0040109D     8D4D F8          lea     ecx,dword ptr ss:[ebp-8]
004010A0     51               push    ecx
004010A1     B9 80464300      mov     ecx,offset Lesson5.cin
004010A6     E8 E5E10000      call    Lesson5.istream::operator>>
004010AB     8D55 F4          lea     edx,dword ptr ss:[ebp-C]
004010AE     52               push    edx
004010AF     B9 80464300      mov     ecx,offset Lesson5.cin
004010B4     E8 67E10000      call    Lesson5.istream::operator>>
004010B9     8D45 90          lea     eax,dword ptr ss:[ebp-70]
004010BC     50               push    eax
004010BD     B9 80464300      mov     ecx,offset Lesson5.cin
004010C2     E8 19DB0000      call    Lesson5.istream::operator>>
004010C7     68 14104000      push    Lesson5.00401014              ; endl 的地址.
004010CC     8B4D FC          mov     ecx,dword ptr ss:[ebp-4]
004010CF     51               push    ecx                            ; i
004010D0     68 28E04200      push    Lesson5.0042E028              ; ASCII "i="
004010D5     B9 40464300      mov     ecx,offset Lesson5.cout       ; 又是把 cout 给 ecx
004010DA     E8 B1C90000      call    Lesson5.ostream::operator<<   ; 继续调用重载
004010DF     8BC8             mov     ecx,eax                       ; 呵呵, 这里直接把返回值给 ecx,继续输出
004010E1     E8 8AC60000      call    Lesson5.ostream::operator<<
004010E6     8BC8             mov     ecx,eax                       ; 同上
004010E8     E8 18FFFFFF      call    Lesson5.00401005
004010ED     68 14104000      push    Lesson5.00401014
004010F2     8D55 90          lea     edx,dword ptr ss:[ebp-70]
004010F5     52               push    edx
004010F6     68 20E04200      push    Lesson5.0042E020              ; ASCII "buf="
004010FB     B9 40464300      mov     ecx,offset Lesson5.cout
00401100     E8 8BC90000      call    Lesson5.ostream::operator<<
00401105     8BC8             mov     ecx,eax
00401107     E8 84C90000      call    Lesson5.ostream::operator<<
0040110C     8BC8             mov     ecx,eax
0040110E     E8 F2FEFFFF      call    Lesson5.00401005
00401113     68 32104000      push    Lesson5.00401032
00401118     B9 40464300      mov     ecx,offset Lesson5.cout
0040111D     E8 E3FEFFFF      call    Lesson5.00401005
00401122     68 14104000      push    Lesson5.00401014
00401127     8B45 FC          mov     eax,dword ptr ss:[ebp-4]
0040112A     50               push    eax
0040112B     68 1CE04200      push    Lesson5.0042E01C              ; ASCII "0x"
00401130     68 0F104000      push    Lesson5.0040100F
00401135     B9 40464300      mov     ecx,offset Lesson5.cout
0040113A     E8 E4FEFFFF      call    Lesson5.00401023
0040113F     8BC8             mov     ecx,eax
00401141     E8 4AC90000      call    Lesson5.ostream::operator<<
00401146     8BC8             mov     ecx,eax
00401148     E8 23C60000      call    Lesson5.ostream::operator<<
0040114D     8BC8             mov     ecx,eax
0040114F     E8 B1FEFFFF      call    Lesson5.00401005
00401154     33C0             xor     eax,eax
00401156     5F               pop     edi
00401157     5E               pop     esi
00401158     5B               pop     ebx
00401159     81C4 B0000000    add     esp,0B0
0040115F     3BEC             cmp     ebp,esp
00401161     E8 4A030000      call    Lesson5._chkesp
00401166     8BE5             mov     esp,ebp
00401168     5D               pop     ebp
00401169     C3               retn
```

`Release` 差不多了，就不贴了。

### 小结

`>>` 和 `<<` 还有很多都是运算符重载，再重载调用前，先把 `>>` 后面的参数压栈，再将 `cin` 或 `cout` 的地址放到 `ecx` 中去，然后调用被重载的 `>>` 函数。

`PS`：这一章东西蛮多，像 `cin`、`cout` 等 I/O 输入输出在 `Win32` `Gui` 程序中也用不到，这一章就以后再扩充吧。

今天 `3.26` 晚 `8` 点。来补充一点。

`C++` 的标准库不带 `.h` 的。带 `.h` 的是旧的标准库，已被某某委员会抛弃了。

但是新的标准库因为在名字空间 `std` 中（这是为了防止名字污染），所以我们要显示的指定名字空间。而使用旧的则不需要指定名字空间。如：

```cpp
#include <iostream>
#include <string>
#include <vector>
using namespace std;
```

旧的：

```cpp
#include <iostream.h>
#include <string.h>
/....
```

来看看 `C++` 标准库的一些内置数据类型：

`string`： 这个也是以类的形式实现的。 `string str("test");` `string str2(str);` `string str;` 这三种方式都是利用 `string` 的构造函数进行赋值的。

`string str = "test" ;` 这是利用运算符重载进行赋值的。

`const char *cstr = str.c_str();` 返回 `c` 字符数组。 必须是 `const`。

`vector` : 这个是以模板和类的形式实现的。下一节具体分析。
