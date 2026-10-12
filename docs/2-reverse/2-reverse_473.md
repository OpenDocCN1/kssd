
# C 语言-常量

> 原文：[`www.pediy.com/kssd/pediy12/134555.html`](https://www.pediy.com/kssd/pediy12/134555.html)

说明：东西都比较简单，个人水平也比较一般，表述难免不清或错误。自己学习的一点总结，还请各位多指教不足和要改进的地方，我会及时更新，这里先谢过。

上一篇关于基本数据类型和局部变量链接：

[`bbs.pediy.com/showthread.php?p=963344#post963344`](http://bbs.pediy.com/showthread.php?p=963344#post963344)

## C 语言之常量

C 语言的常量主要只在编译前值就确定的数值，类型主要包括：整型常量，浮点型常量，字符型常量和枚举型常量。

环境为 `win32`，`Vc`

示例代码：

```
#define MAX_NUM 256
const int g_cnTest = 123;

int main(int argc, char* argv[])
{
    //bool
    bool isTest = true;
    //char
    char cChar  = 'T';
    //int
    int nNum     = 0x123;
    nNum = MAX_NUM;
    int nMax = g_cnTest;
    //枚举
    typedef  enum EnTest{en_0, en_1, en_2};
    EnTest et = en_2;
    //float(double)
    float fValue = 1.2f;
    //double
    double dfValue1 = 1.2;
    double dfValue2 = 1.2;
    //字符串
    printf("Hello World!\r\n");
    char* lpTest = (char*)("Hello World!\r\n");

    return 0;
}
```

### 数值型常量的表现

类型为 `bool`，`char`，`int`，`float`，`double`，`enum`

代码:

```
15:       //bool
16:       bool isTest = true;
0040E9D8 C6 45 FC 01           mov         byte ptr [ebp-4],1
17:       //char
18:       char cChar  = 'T';
0040E9DC C6 45 F8 54           mov         byte ptr [ebp-8],54h
19:       //int
20:       int nNum     = 0x123;
0040E9E0 C7 45 F4 23 01 00 00 mov         dword ptr [ebp-0Ch],123h
21:       nNum = MAX_NUM;
0040E9E7 C7 45 F4 00 01 00 00 mov         dword ptr [ebp-0Ch],100h
22:       int nMax = cnTest;
0040E9EE C7 45 F0 7B 00 00 00 mov         dword ptr [ebp-10h],7Bh
23:       //枚举
24:       typedef  enum EnTest{en_0, en_1, en_2};
25:       EnTest et = en_2;
0040E9F5 C7 45 EC 02 00 00 00 mov         dword ptr [ebp-14h],2
28:       //float
29:       float fValue = 1.2f;
0040EA03 C7 45 E4 9A 99 99 3F mov         dword ptr [ebp-1Ch],3F99999Ah
30:       //double
31:       double dfValue1 = 1.2;
0040EA0A C7 45 DC 33 33 33 33 mov         dword ptr [ebp-24h],33333333h
0040EA11 C7 45 E0 33 33 F3 3F mov         dword ptr [ebp-20h],3FF33333h
32:       double dfValue2 = 1.2;
0040EA18 C7 45 D4 33 33 33 33 mov         dword ptr [ebp-2Ch],33333333h
0040EA1F C7 45 D8 33 33 F3 3F mov         dword ptr [ebp-28h],3FF33333h
```

从汇编代码中可以得到一些规律：

1）`bool`，`char`，`int`(包括 `#define` )的字面常量值编成了操作数(立即数)，直接使用，不需要寻址操作

如 `nNum = 0x123`

对应的机器码为 `C7 45 E0 23 01 00 00` => 对应数值为 `0x123` 小尾方式存储。

2）枚举型和 `const` 类型整型变量也当常量使用，直接编译到代码中。因为其值定义并初始化时，其值就是固定的，不会被修改。来试着修改下

a) 直接修改

`nMax = ++g_cnTest;`

编译时提示需要左值，说明没有存储空间，编译器将其视作常量对待(编译器处理)。

b) 通过取地址方式来修改，编译通过，执行时。。。

代码:

```
 int* lpInt = (int*)&g_cnTest;
 *lpInt = 0x111;
```

![](img/1c1ece8005ca6bf02c97bd50d0a8b5fe.jpg)

因为 `const` 定义的值在常量区，而常量区的内存属性通常为可读不可写，有写操作时会触发系统的内存保护机制，就给你弹框。

3) 机器码中数值长度跟具体的数据类型相关 (枚举这里貌似也是)

4）浮点数是 `IEEE` 编码方式存储

5) 此外还有一种情况就是在编译时，变量的值是可预测的，不变的(有的叫契约常量)。

代码:

```
    int nNum = 0x123;
    int nTest = nNum;
    printf("%d", nTest);
```

Debug 方式下：

代码:

```
   int nNum = 0x123;
001F432E  mov         dword ptr [nNum],123h 
    int nTest = nNum;
001F4335  mov         eax,dword ptr [nNum] 
001F4338  mov         dword ptr [nTest],eax 
    printf("%d", nTest);
001F433B  mov         esi,esp 
001F433D  mov         eax,dword ptr [nTest] 
001F4340  push        eax   
001F4341  push        offset string "%d" (1F7818h) 
001F4346  call        dword ptr [__imp__printf (1FB2F0h)] 
```

Release 方式：

代码:

```
text:00401003 8B 3D A0 20 40+    mov edi, ds:__imp__printf
.text:00401009 68 23 01 00 00     push 123h
.text:0040100E 68 14 21 40 00     push offset Format               ; "%d"
.text:00401013 FF D7             call edi ; __imp__printf
```

### 字符串

代码:

```
33:       //字符串
34:       printf("Hello World!\r\n");
0040EA26 68 1C 40 42 00       push        offset string "My Hello World!\r\n" (0042401c)
0040EA2B E8 60 26 FF FF       call        printf (00401090)
0040EA30 83 C4 04             add         esp,4
35:       char* lpTest = (char*)("Hello World!\r\n");
0040EA33 C7 45 D0 1C 40 42 00 mov         dword ptr [ebp-30h],offset string "My Hello World!\r\n" (0042401c)
```

同样观察得到一些规律为：

1） 字符串常量保存的在内存常量区中(属性一般为可读不可写)，没有和代码编译在一起直接做值使用，在指令中通过获取指定地址来使用

这里通过 `0x42401c` 来获取字符串内容

`0042401C 48 65 6C 6C 6F 20 57 6F 72 6C 64 21 0D 0A 00 00 7B Hello World!`

2）值相等的字符串在内存中只会有一份，需要时获取即可。

这里 `printf` 使用的字符串和初始化指针的地址都为【`0042401c`】

3）对常量的修改同样是受到系统内存属性保护的约束, 一个 `C0xx05` 的错误没的跑。

代码:

```
 lpTest[0] = 'I';
0040EA43 8B 4D D0             mov         ecx,dword ptr [ebp-30h]
0040EA46 C6 01 49             mov         byte ptr [ecx],49h
```

![](img/82f9caa52b9513f0f77a968894b0a9e3.jpg)

在调试状态，直接修改内存

代码:

```
0042401C  48 65 6C 6C 6F 20 57 6F 72 6C 64 21 0D 0A 00 00 7B 00   Hello World!  <--原来的
0042401C  48 65 6C 6C 6F 20 57 4F 52 4C 44 21 0D 0A 00 00 7B 00   Hello WORLD!     <--直接修改内存
```

这里怎么又可以修改了呢？

主要是因为调试器的权限，比如下个软断点(修改代码+`CC` 即 `int3`)，代码都能改，调试器还有啥做不到的呢。

### 总结：

1）`VC` 处理数值型常量时(`bool`, `char`, `int`, `enum`, `const xx`，`float`，`double`)，数值会编译到代码中，作为立即数来使用；

2）在代码中的数据的长度和类型相关；

3）常量区得数据会受到系统内存属性的保护；

4）`const` 类型在编译时修改，会受到编译的检查；

5）字符串常量放置在常量区，相同的字符串只存在一份

水平很一般，错误难免，望多指教！

五边形 2011 年 5 月
