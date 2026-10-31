
# 浅谈VB6逆向工程(1)

## 前言

VB的编译有两种方式，一种是P-Code方式，一种是本机代码。P-Code方式是VB从早期版本保留下来的，也是比较难逆向的一种。而本机代码方式编译的程序已经和VC很接近了。这里只探讨以本机代码方式编译的程序。由于微软在VB的实现方面的资料很少，使人觉得VB的逆向比较难。其实掌握一些规律，VB的程序还是很容易入门的。

这里所提到的知识都是个人的经验，没有什么官方的资料作为依据。所以错误之处难免，如果你发现什么错误，欢迎指正。

## 1. 从简单变量的实现开始

一个VB简单变量的表示至少需要12个字节。通常前2个字节是表示类型信息的。从第5个字节到第8个字节并不总是使用到，实际上很少被使用。我们不妨先叫它辅助类型信息。从第9个字节开始就是真正的变量的值了。这里有可能存储一个指针值，也可能是数据，具体是什么取决于变量类型。

另一个值得注意的事实是VB的内存是以4个字节对齐的。即使你使用一个字节，那至少也要4个字节来表示。而且编译器只初始化它需要的那些字节，剩余的字节可能是随机数据。下面我们将会看到这些。

想弄明白编译器在内部怎么实现的，最好的方法就是编一段程序跟踪运行看看。我编写的代码如下：

```vb
Dim a, i As Byte
Dim b, j As Integer
Dim c, k As Long
Dim d, l As Boolean
Dim e, m As String
Dim f, n As Date
Dim g, o As Double
Dim h, p As Single

a = &H30
b = 330
c = 66000
d = True
e = "hello"
f = Now
g = 3.1415
h = 1.27

i = a
j = b
k = c
l = d
m = e
n = f
o = g
p = h
```

这段代码在VB的默认设置（速度优化）下编译。然后用od反汇编出来如下：

去掉了部分无关内容，其余的我在这段代码的实现里做了注释：

```asm
00401B02  MOV ESI,DWORD PTR DS:[<&MSVBVM60.__vbaVa>;  MSVBVM60.__vbaVarMove
00401B08  XOR EDI,EDI
00401B0A  MOV EBX,2
00401B0F  MOV DWORD PTR SS:[EBP-DC],EDI
00401B15  LEA EDX,DWORD PTR SS:[EBP-DC]
00401B1B  LEA ECX,DWORD PTR SS:[EBP-4C]
00401B1E  MOV DWORD PTR SS:[EBP-28],EDI
00401B21  MOV DWORD PTR SS:[EBP-4C],EDI
00401B24  MOV DWORD PTR SS:[EBP-5C],EDI
00401B27  MOV DWORD PTR SS:[EBP-6C],EDI
00401B2A  MOV DWORD PTR SS:[EBP-7C],EDI
00401B2D  MOV DWORD PTR SS:[EBP-8C],EDI
00401B33  MOV DWORD PTR SS:[EBP-9C],EDI
00401B39  MOV DWORD PTR SS:[EBP-AC],EDI
00401B3F  MOV DWORD PTR SS:[EBP-BC],EDI
00401B45  MOV DWORD PTR SS:[EBP-CC],EDI
00401B4B  MOV DWORD PTR SS:[EBP-D4],30     //30h = &H30
00401B55  MOV DWORD PTR SS:[EBP-DC],EBX    //EBX = 2: integer型
00401B5B  CALL ESI                                 ;  <&MSVBVM60.__vbaVarMove>
                                           ***** a =&H30  即[ebp-4c]
00401B5D  LEA EDX,DWORD PTR SS:[EBP-DC]
00401B63  LEA ECX,DWORD PTR SS:[EBP-5C]    //变量b
00401B66  MOV DWORD PTR SS:[EBP-D4],14A    //14Ah = 330
00401B70  MOV DWORD PTR SS:[EBP-DC],EBX    //EBX = 2: integer型
00401B76  CALL ESI
                                           ***** b = 330  即[ebp-5c]
00401B78  LEA EDX,DWORD PTR SS:[EBP-DC]
00401B7E  LEA ECX,DWORD PTR SS:[EBP-6C]
00401B81  MOV DWORD PTR SS:[EBP-D4],101D0  //101D0 = 66000
00401B8B  MOV DWORD PTR SS:[EBP-DC],3      //3:long型
00401B95  CALL ESI
                                           ***** c = 66000 即[ebp-6c]
00401B97  LEA EDX,DWORD PTR SS:[EBP-DC]
00401B9D  LEA ECX,DWORD PTR SS:[EBP-7C]
00401BA0  MOV DWORD PTR SS:[EBP-D4],-1     //-1 TRUE
00401BAA  MOV DWORD PTR SS:[EBP-DC],0B     //12:boolean型
00401BB4  CALL ESI
                                           ***** d = TRUE 即[ebp-7c]
00401BB6  LEA EDX,DWORD PTR SS:[EBP-DC]
00401BBC  LEA ECX,DWORD PTR SS:[EBP-8C]
00401BC2  MOV DWORD PTR SS:[EBP-D4],工程1.00401948   ;  UNICODE "hello" //"hello"
00401BCC  MOV DWORD PTR SS:[EBP-DC],8      //8:string型
00401BD6  CALL DWORD PTR DS:[<&MSVBVM60.__vbaVarCo>;  MSVBVM60.__vbaVarCopy
                                           ***** e = "hello" 即[ebp-8c]
00401BDC  LEA EAX,DWORD PTR SS:[EBP-CC]
00401BE2  PUSH EAX
00401BE3  CALL DWORD PTR DS:[<&MSVBVM60.#546>] //Now
00401BE9  LEA EDX,DWORD PTR SS:[EBP-CC]    //Now
00401BEF  LEA ECX,DWORD PTR SS:[EBP-9C]
00401BF5  CALL ESI
                                           ***** f = Now  即[ebp-9c]
00401BF7  MOV EBX,5
00401BFC  LEA EDX,DWORD PTR SS:[EBP-DC]
00401C02  LEA ECX,DWORD PTR SS:[EBP-AC]
00401C08  MOV DWORD PTR SS:[EBP-D4],C083126F
00401C12  MOV DWORD PTR SS:[EBP-D0],400921CA //3.1415
00401C1C  MOV DWORD PTR SS:[EBP-DC],EBX    //EBX = 5 :double型
00401C22  CALL ESI
                                           ***** g = 3.1415 即[ebp-ac]
00401C24  LEA EDX,DWORD PTR SS:[EBP-DC]
00401C2A  LEA ECX,DWORD PTR SS:[EBP-BC]
00401C30  MOV DWORD PTR SS:[EBP-D4],851EB852
00401C3A  MOV DWORD PTR SS:[EBP-D0],3FF451EB //1.27
00401C44  MOV DWORD PTR SS:[EBP-DC],EBX    //EBX = 5 :double型
00401C4A  CALL ESI
                                           // h = 1.27
00401C4C  LEA ECX,DWORD PTR SS:[EBP-4C]
00401C4F  PUSH ECX
00401C50  CALL DWORD PTR DS:[<&MSVBVM60.__vbaUI1Va>;  MSVBVM60.__vbaUI1Var
                                           //取byte, al中
                                           ***** i = a
00401C56  LEA EDX,DWORD PTR SS:[EBP-5C]
00401C59  PUSH EDX
00401C5A  CALL DWORD PTR DS:[<&MSVBVM60.__vbaI2Var>;  MSVBVM60.__vbaI2Var
                                           //取integer, ax中
                                           ***** j = b
00401C60  LEA EAX,DWORD PTR SS:[EBP-6C]
00401C63  PUSH EAX
00401C64  CALL DWORD PTR DS:[<&MSVBVM60.__vbaI4Var>;  MSVBVM60.__vbaI4Var
                                           //取long, eax中
                                           ***** k = c
00401C6A  LEA ECX,DWORD PTR SS:[EBP-7C]
00401C6D  PUSH ECX
00401C6E  CALL DWORD PTR DS:[<&MSVBVM60.__vbaBoolV>;  MSVBVM60.__vbaBoolVar
                                           //取boolean, ax中
                                           ***** l = d
00401C74  LEA EDX,DWORD PTR SS:[EBP-8C]
00401C7A  PUSH EDX
00401C7B  CALL DWORD PTR DS:[<&MSVBVM60.__vbaStrVa>;  MSVBVM60.__vbaStrVarCopy
                                           //取string, eax中是地址
                                           ***** m = e
00401C81  MOV EDX,EAX
00401C83  LEA ECX,DWORD PTR SS:[EBP-28]
00401C86  CALL DWORD PTR DS:[<&MSVBVM60.__vbaStrMo>;  MSVBVM60.__vbaStrMove
00401C8C  LEA EAX,DWORD PTR SS:[EBP-9C]
00401C92  PUSH EAX
00401C93  CALL DWORD PTR DS:[<&MSVBVM60.__vbaDateV>;  MSVBVM60.__vbaDateVar
                                           //取date, 浮点栈中
                                           ***** n = f
00401C99  LEA ECX,DWORD PTR SS:[EBP-AC]
00401C9F  FSTP ST
00401CA1  PUSH ECX
00401CA2  CALL DWORD PTR DS:[<&MSVBVM60.__vbaR8Var>;  MSVBVM60.__vbaR8Var
                                           //取double, 浮点栈中
                                           ***** o = g
00401CA8  LEA EDX,DWORD PTR SS:[EBP-BC]
00401CAE  FSTP ST
00401CB0  PUSH EDX
00401CB1  CALL DWORD PTR DS:[<&MSVBVM60.__vbaR4Var>;  MSVBVM60.__vbaR4Var
00401CB7  FSTP ST
                                           //取single, 浮点栈中
                                           ***** p = h
```

---

先总结一下：`byte`和`integer`一样，内部都是使用`integer`类型表示，`single`和`double`一样，内部都是使用`double`类型表示。`date`占用8个字节。`boolean`类型占用2个字节`&HFFFF`表示`TRUE`，而`0`表示`FALSE`。

这些简单变量在内存中的表示如下：前4个字节含有变量类型，接着4个字节（意义还不清楚），我们不妨先叫他辅助类型信息，过了这8个字节就是真正的变量数据了。

如变量`a`的内存表示如下：

```
63F3C4 : 02 00 00 00
63F3C8 : 00 00 00 00
63F3CC : 30 00 00 00
```

可以看到，`63F3CC`处是真正的变量值。如果是字符串，这个位置可以看到一个指针。

简单变量里没有提到的类型是`Currency`，`Decimal`，这些你可以自己实际观察一下。

关于简单变量就总结这些，不知道你看懂了没有。我没有办法把机器搬上来让你看。所以还是那句话：如果你想弄明白我所讲的到底是怎么一回事，就把上面那段程序编译一下，然后用`ollydbg`跟踪一下看看。

在本文即将结束时要提到的一个事实是：VB并不是强类型语言。例如你可以把一个`boolean`类型的变量赋值给一个整数，然后再打印出这个整数的值，你可以看到`TRUE`的输出为`-1`，而`FALSE`的输出为`0`。当然在你决定这样做的时候你要保证两个变量所占内存空间大小一样或者被赋值的变量的空间要大于所赋值的变量的空间。否则你可能会得到一个溢出错误。你可以试试把`date`类型的变量赋值给一个`integer`类型的变量 :)
