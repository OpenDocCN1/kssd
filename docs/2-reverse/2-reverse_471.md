

# C 语言基本数据类型和局部变量

> 原文：[`www.pediy.com/kssd/pediy12/134570.html`](https://www.pediy.com/kssd/pediy12/134570.html)

**说明：比较简单，水平也很一般，错误难免，欢迎批评指正，谢谢**  
  
**C 语言之基本数据类型变量**  
  
**基本数据类型**  
先看下 C 语言的基本数据类型：  
bool  布尔        1 字节  
char  字符型       1 字节  
int    整型        4 字节  
float  浮点型(单精度)    4 字节  
double  浮点型(双精度)    8 字节  
  
  数据类型是用来描述对应表示的数据的相关属性，如：大小(在内存中占几字节)，数值表示的形式(0，1，有符号是没…)。  
  看下各种数据类型的具体表示，这里不得不提到一个东西，就是变量。  
  先看示例代码：  

代码:

```
int \_tmain(int argc, \_TCHAR\* argv\[\])
{    
    //bool
    bool isTest = true;
    //char
    char cChar  = 'T';
    //int
    int nNum    = 0x123;
    //float(double)
    float fValue = 1.2f;
    //double
    double dfValue = 1.2;

    return 0;
}

```

代码:

```
    
//boolbool isTest = true;

```

表示真假 2 种状态，非 0 为真即 true，0 为假即 false  
bool 型的值只有 true 和 false，内存中值为 1 和 0，实际上除了为 0 的情况是 false，其他都为 true，比如下面的情况。  
代码:

```
    isTest = 123;
    isTest = 0.2;
    isTest = 'a';

```

内存中占用 1 字节，值都为 1 既是 true。  
0x001DFCF3  01 cc cc cc cc 48 fd 1d 00 b8 25 09 01 01 00 00  
  
代码:

```
   
 //char
char cChar  = 'T';

```

ASCII 码是 0-255，用 1 字节就可以表示，实际表示是 1 字节范围内的所有整型数值  
在内存的表现形式如下，占 1 字节，16 进制表示  
0x001DFCE7  **54** cc cc cc cc cc cc cc cc cc cc cc 01 cc cc cc  T  
表示的数据性质呈现一个环形的特性，一直+++或一直---总能回到原点，就跟地球是圆的一样(貌似没人说是方的了)  
代码:

```
    //char 型
    char cChar  = 0x4;
    for (; ; ++cChar)   //增加方向 0x5，0x6，0x7--->
    {
        printf("%02X\\r\\n", cChar);
        if (cChar == 0x3)
        {
            printf("我又回来啦！\\r\\n");
            break;
        }
}

```

输出结果：  
代码:

```
FFFFFFFE
FFFFFFFF
00
01
02
03
我又回来啦！
char cChar  = 0x4;    unsigned char cChar  = 0x4;(无符号)
FD
FE
FF
00
01
02
03
我又回来啦！

```

代码:

```
 //int
int nNum    = 0x123;

```

表示 4 字节的整型数值，在内存中如下表示  
0x001DFCD8 ** 23 01 00 00 **cc cc cc cc cc cc cc cc cc cc cc 54  
这里采用小端(小尾)的存储方式，也就是地位在低地址，高位在高地址  
0x00000123  从低到高地址为 0x1DFCD8-0x1DFCDB 对应存放 23 01 00 00  
另外一种是大端(大尾)的存储方式，主要是就高低位方向相反，看下示例  
在不同的硬件环境下对数据的处理会存在差异，内存中数据主要就是大小尾的问题。  
**比如数值 0x12  
在小尾存储方式的内存为 12 00 00 00，->小尾读(0x12), 大尾读(0x12000000)  
而在大尾方式下为      00 00 00 12  ->小尾读(0x12000000), 大尾读(0x12)**  
代码:

```
   
 \[B\]//float(double)\[/B
\]float fValue = 1.2f;
0x001DFCB4  9a 99 99 3f cc cc cc cc cc cc cc cc 02 00 00 00
    //double
double dfValue = 1.2;
0x001DFCA4  33 33 33 33 33 33 f3 3f cc cc cc cc cc cc cc cc  

```

  单精度为 4 字节，双精度为 8 字节，在内存中的表示方式是通过 IEEE 编码后的形式。  
  大概的编码形式为：\[指数位\]\[指数\]\[小数位\]  
  
**直接操作栈地址**  
看下代码中的数值在内存中的分布情况：  
代码:

```
int \_tmain(int argc, \_TCHAR\* argv\[\])
{    
    int nNum1    = 0x1;
    int nNum2    = 0x2;
    int nNum3    = 0x3;
    int nNum4    = 0x4;
    int nNum5    = 0x5;

    return 0;
}

```

变量对应的地址和内存中值  (每次值都不太一样，说明下先)  
代码:

```
+    &nNum1    0x0016f748  int \*
+    &nNum2    0x0016f73c  int \*
+    &nNum3    0x0016f730  int \*
+    &nNum4    0x0016f724  int \*
+    &nNum5    0x0016f718  int \*
    &nNum1-&nNum5   12  unsigned long
0x0021F874  05 00 00 00 cc cc cc cc cc cc cc cc  ....????????
0x0021F880  04 00 00 00 cc cc cc cc cc cc cc cc  ....????????
0x0021F88C  03 00 00 00 cc cc cc cc cc cc cc cc  ....????????
0x0021F898  02 00 00 00 cc cc cc cc cc cc cc cc  ....????????
0x0021F8A4  01 00 00 00 cc cc cc cc fc f8 21 00  ....??????!.

```

**说明：**  
局部变量是放在栈中，栈就是一块内存，栈的特性是先进后出，跟正常穿衣脱衣顺序类似，这里只要了解下即可。  
**1+2 **的计算，其实就是拿对应内存的值来做加法运算  
代码:

```
    nNum5 = \*(int\*)0x0021F8A4 + \*(int\*)0x0021F898  ;
     这里主要了解下就好，其实是指针的操作 \*(类型\*)内存地址 ->操作指定地址按指定类型使用
00CC1041  mov         eax,dword ptr ds:\[0021F8A4h\] 
00CC1046  add         eax,dword ptr ds:\[21F898h\] 
00CC104C  mov         dword ptr \[nNum5\],eax

```

在看下内存  
0x0021F874  03 00 00 00 cc cc cc cc cc cc cc cc  ....????????  
0x0021F880  04 00 00 00 cc cc cc cc cc cc cc cc  ....????????  
0x0021F88C  03 00 00 00 cc cc cc cc cc cc cc cc  ....????????  
0x0021F898  02 00 00 00 cc cc cc cc cc cc cc cc  ....????????  
0x0021F8A4  01 00 00 00 cc cc cc cc fc f8 21 00  
在来个复杂的运算 （因为 VC2008 占地址不固定每次都是动态取值，分布还是比较固定的说）  
\[code\]（1+2）\*3+4  
nNum5 = (\*(int\*)0x0018fa1c + \*(int\*)0x0018fa10)\* \*(int\*)0x0018fa04 + \*(int\*)0x0018f9f8 ;  
00FF1041  mov         eax,dword ptr ds:\[0018FA1Ch\]   
00FF1046  add         eax,dword ptr ds:\[18FA10h\]   
00FF104C  imul        eax,dword ptr ds:\[18FA04h\]   
00FF1053  add         eax,dword ptr ds:\[18F9F8h\]   
00FF1059  mov         dword ptr \[nNum5\],eax  
运行后内存 （1+2）\*3 + 4 => 3\*3+4 => （13）10   (D)16   
0x0018F9EC  0d 00 00 00 cc cc cc cc cc cc cc cc  ....????????  
0x0018F9F8  04 00 00 00 cc cc cc cc cc cc cc cc  ....????????  
0x0018FA04  03 00 00 00 cc cc cc cc cc cc cc cc  ....????????  
0x0018FA10  02 00 00 00 cc cc cc cc cc cc cc cc  ....????????  
0x0018FA1C  01 00 00 00 cc cc cc cc 74 fa 18 00  ....????t?..  
\[/code\]  
再来点有难度  
(((1+2\*3)²+3\*4-2)\*3/2+3-4)\*4  
这个也没问题，就是地址操作嘛。。。。。。  
 ZZZ 不过写的也太累了，要是再复杂点，地址成百上千的不用运算，我晕了。  
  
**主角闪亮登场”变量，变量名”**  
**变量**，字面理解就是会变化的值。  
**变量名**，字面理解就是会变化值的名字，你可以和前面的地址对应起来了嘛？  
  变量名要使用字面，\_，数字做为名称，数字不能第一个(不然谁搞的清是数字还是变量)，不能使用保留字，名字最好能望名知意。  
我们再用变量名来做下运算  
代码:

```
1+2 =>  nNum1+ nNum2
    nNum5 = nNum1 + nNum2;
00CE1EE1  mov         eax,dword ptr \[nNum1\] 
00CE1EE4  add         eax,dword ptr \[nNum2\] 
00CE1EE7  mov         dword ptr \[nNum5\],eax

```

和下面的代码其实是一回事，只是在调试时有符号名称，其实操作还是地址及其内容  
代码:

```
00CC1041  mov         eax,dword ptr ds:\[0021F8A4h\] 
00CC1046  add         eax,dword ptr ds:\[21F898h\] 
00CC104C  mov         dword ptr \[nNum5\],eax

```

变量名其实是地址的一个代号名称，高级语言给一个好的助记符来代替地址，在底层实际还是地址操作。在日常生活中也常用，比如电话号码，QQ 好友，总要给来个响亮的名字吧。  
这个可提供了非常大的便利，从此就脱离的低级趣味了，可以定义一堆喜欢的名字  
Int a；  
Int aa；  
Int aaa；  
Int aaaa；  
。。。。。。。。。。。。  
莫非又穿越了，跟记数字有区别嘛，好的名字成功的开始  
变量名的规范，个人习惯仅供参数  
bool 型      is\_xxx  
int 型    n\_xxxx  
char      cXxxxx  
float    fXxxxx  
DWORD    dwXxxx  
全局    g\_ 类型+名称    g\_nNumber  
静态    s\_ 类型+名字  
成员    m\_ 类型+名字   m\_xXxxx  
  。。。。。。。。  
总结：  
1）  数据类型是用来约束所标识的类型的数据特性；  
2）  在内存中数据都是 16 进制补码的形式存在，存在大小端的情况；  
3）  浮点数采用 IEEE 的编码方式保存；  
4）  变量名是对对应内存地址的一个别名，高级语言提高的一个友好的描述；  
5）  变量名称要能反映数据类型和作用，避免穿越。  
6）  内存中的数据和类型关联就有了相应的类型特性，也就是说通过指针方式来重新指定某地址的数据类型，说它是啥就是啥。OY！  
  
有名称代号了就好办事了，9527 给哥买张火车票去。哈哈！  
  
  
说明：水平有限，错误不妥难免，欢迎批评指点，谢谢！  
  
                                                    五边形   2011 年 5 月  
  
下一篇  关于常量链接：  
[`bbs.pediy.com/showthread.php?p=963240#post963240`](http://bbs.pediy.com/showthread.php?p=963240#post963240)