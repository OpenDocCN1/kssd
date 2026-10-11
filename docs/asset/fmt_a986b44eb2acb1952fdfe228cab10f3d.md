
# DFCGVC 编译器优化分析-2

```
0040102B   |.  8D4424 04           lea eax,dword ptr ss:[esp+4]  
0040102F   |.  50                 push eax                                             ; /s  
00401030   |.  C74424 10 50654469 mov dword ptr ss:[esp+10],69446550                   ; |  
00401038   |.  C74424 14 59000000 mov dword ptr ss:[esp+14],59                         ; |  
00401040   |.  C74424 08 4C4F5645 mov dword ptr ss:[esp+8],45564F4C                     ; |  
00401048   |.  C64424 0C 00       mov byte ptr ss:[esp+C],0                            ; |  
```

第一句没有错，但为什么后面没有对`[esp+4]`操作了。。`push eax`。。这个。 因此相应的就应该是

从`esp+8`开始

以及对其。即使是`CHAR`也是一个`INT`所以在`WINDOWS`编程中应当尽量就行 4 字节操作（64 位为 8 字节）

今天就讲到这么里

以后我会介绍 算数运算的编译器优化，以及条件语句。

今天讲讲常见的算术运算的优化：

## 1. 先来看看加减法运算

```
__asm int 3    
    int a,b;  
  scanf("%d %d",&a,&b);  
  int c;  
  int d;  
  c=a+b;  
  d=a-b;  
  printf("\n%d %d",c,d);  
  __asm int 3  
```

汇编代码如下

```
00401020   /$  83EC 08            sub esp,8  
00401023   |.  CC                 int3  
00401024   |.  8D4424 04          lea eax,dword ptr ss:[esp+4]  
00401028   |.  50                 push eax  
00401029   |.  8D4C24 04          lea ecx,dword ptr ss:[esp+4]  
0040102D   |.  51                 push ecx  
0040102E   |.  68 50214000        push 编译器学.00402150                                 ; /format = "%d %d"  
00401033   |.  FF15 C4204000      call dword ptr ds:[<&MSVCR80.scanf>]                 ; \scanf  
00401039   |.  8B4424 0C          mov eax,dword ptr ss:[esp+C]  
0040103D   |.  8B4C24 10          mov ecx,dword ptr ss:[esp+10]  
00401041   |.  8BD0               mov edx,eax  
00401043   |.  2BD1               sub edx,ecx  
00401045   |.  52                 push edx                                             ; /<%d>  
00401046   |.  03C1               add eax,ecx                                          ; |  
00401048   |.  50                 push eax                                             ; |<%d>  
00401049   |.  68 58214000        push 编译器学.00402158                                 ; |format = "  
%d %d"  
0040104E   |.  FF15 C8204000      call dword ptr ds:[<&MSVCR80.printf>]                ; \printf  
00401054   |.  83C4 18            add esp,18  
00401057   |.  CC                 int3  
```

虽然我们声明了 4 个变量。但编译器会认为这等价于

```
printf("\n%d %d",a+b,c+d);
```

因此也只给我们分配了两个变量。

其实这也告诉编程序的人，没有必要写成`printf("\n%d %d",a+b,c+d);`

形式，这种形式显然更难维护和阅读。

```
00401039   |.  8B4424 0C          mov eax,dword ptr ss:[esp+C]  
0040103D   |.  8B4C24 10          mov ecx,dword ptr ss:[esp+10]  
00401041   |.  8BD0               mov edx,eax  
00401043   |.  2BD1               sub edx,ecx  
00401045   |.  52                 push edx                                             ; /<%d>  
00401046   |.  03C1               add eax,ecx                                          ; |  
```

这里比较巧妙由于涉及到两次运算，因此将两个变量统一读到两个寄存器中，而不象有些编译器会使用内存相加。

加减法没有什么好说的。主要说乘除法：

## 2. 乘除法

先来看乘法运算：

首先是有符号运算。

```
void study5()  
  {  
  __asm int 3  
    int a,b;  
  scanf("%d %d",&a,&b);  
  int c,d,e;  
  c=a*2;  
  d=a*b;  
  e=a*7;  
  printf("\n%d %d",c,d,e);  
  __asm int 3   
}  
```

```
00401023   |.  CC                 int3  
00401024   |.  8D4424 04          lea eax,dword ptr ss:[esp+4]  
00401028   |.  50                 push eax  
00401029   |.  8D4C24 04          lea ecx,dword ptr ss:[esp+4]  
0040102D   |.  51                 push ecx  
0040102E   |.  68 50214000        push 编译器学.00402150                           ; /format = "%d %d"  
00401033   |.  FF15 C4204000      call dword ptr ds:[<&MSVCR80.scanf>]           ; \scanf  
00401039   |.  8B4424 0C          mov eax,dword ptr ss:[esp+C]  
0040103D   |.  8BC8               mov ecx,eax  
0040103F   |.  0FAF4C24 10        imul ecx,dword ptr ss:[esp+10]  
00401044   |.  8D14C5 00000000    lea edx,dword ptr ds:[eax*8]  
0040104B   |.  2BD0               sub edx,eax  
0040104D   |.  52                 push edx  
0040104E   |.  51                 push ecx                                   ; /<%d>  
0040104F   |.  8D1400             lea edx,dword ptr ds:[eax+eax]             ; |  
00401052   |.  52                 push edx                                   ; |<%d>  
00401053   |.  68 58214000        push 编译器学.00402158                           ; |format = "  
%d %d"  
00401058   |.  FF15 C8204000      call dword ptr ds:[<&MSVCR80.printf>]       ; \printf  
0040105E   |.  83C4 1C            add esp,1C  
00401061   |.  CC                 int3  
```

```
0040103F   |.  0FAF4C24 10        imul ecx,dword ptr ss:[esp+10]  
```

没有什么说的，整数乘法

```
00401044   |.  8D14C5 00000000    lea edx,dword ptr ds:[eax*8]  
0040104B   |.  2BD0               sub edx,eax  
```

呵呵。看到这里是否想到我们小学的时候得简便运算。我们是乘以 7

但是编译器给我们编程了乘以 8 在减掉被乘数。神奇吧。

同样的道理

```
0040104F   |.  8D1400             lea edx,dword ptr ds:[eax+eax]             ; |  
```

乘以二就是两个这个数相加。。想象力丰富吧。

总结一下载`VC`中乘法如果在已知被除数的情况下用

```
lea edx,dword ptr ds:[eax*x]
```

的形式，

否则采用`IMUL`指令。

现在我们把代码变成这样

```
c=a*-2;  
  d=a*-4;  
  e=a*-7;  
```

测试一下有符号运算。

```
00401033   |.  FF15 C4204000      call dword ptr ds:[<&MSVCR80.scanf>]       ; \scanf  
00401039   |.  8B4424 0C          mov eax,dword ptr ss:[esp+C]  
0040103D   |.  8D14C5 00000000    lea edx,dword ptr ds:[eax*8]  
00401044   |.  8BC8               mov ecx,eax  
00401046   |.  2BD0               sub edx,eax  
00401048   |.  F7D9               neg ecx  
0040104A   |.  F7DA               neg edx  
0040104C   |.  52                 push edx  
0040104D   |.  03C9               add ecx,ecx  
0040104F   |.  03C9               add ecx,ecx  
00401051   |.  F7D8               neg eax  
00401053   |.  51                 push ecx                                 ; /<%d>  
00401054   |.  03C0               add eax,eax                              ; |  
00401056   |.  50                 push eax                                 ; |<%d>  
00401057   |.  68 58214000        push 编译器学.00402158                           ; |format = "  
%d %d"  
0040105C   |.  FF15 C8204000      call dword ptr ds:[<&MSVCR80.printf>]     ; \printf  
```

代码有些变化 除了用了`NEG`指令以外。还有

原来的

```
lea edx,dword ptr ds:[eax+eax]             ; |  
```

变成了

```
0040104D   |.  03C9               add ecx,ecx  
0040104F   |.  03C9               add ecx,ecx  
```

不过大体上没有什么好说的。

乘法总的来说都比较好识别

代码在改一改 改为

```
  d=short(a*b);  
  c=c*c*c;  
  e=a*100;  
```

我们来看看类型转换的问题

其他的都差不多主要说一下

```
0040104E   |.  0FBFD1             movsx edx,cx  
00401051   |.  8BC8               mov ecx,eax  
00401053   |.  0FAFC8             imul ecx,eax  
00401056   |.  0FAFC8             imul ecx,eax  
```

如果是`short`那么就会用`movsx`符号取得低位

```
c=c*c*c---〉imul ecx,eax；imul ecx,eax
```

乘法总的来说都比较容易识别也比较简单。

除法就不一样了，无符号有符号完全不同，另外以及不同编译器的处理也很不一样

且涉及到求模运算和除法运算两种。

由于`CPU`中只有加法器和乘法器 所以一般而言除法会转变为乘法运算。

我们先来看两组最简单的。

先讲无符号的明天再说有符号的

代码：

```
  __asm int 3  
  unsigned  int a,b;  
  scanf("%d %d",&a,&b);  
  unsigned int c,d,e;  
  d=a/2;  
  c=b/4;  
  printf("\n%d %d",c,d);  
  __asm int 3  
```

生成代码（部分）

```
00401039   |.  8B5424 0C            mov edx,dword ptr ss:[esp+C]  
0040103D   |.  8B4424 10            mov eax,dword ptr ss:[esp+10]  
00401041   |.  D1EA                 shr edx,1  
00401043   |.  52                   push edx                       ; /<%d>  
00401044   |.  C1E8 02              shr eax,2                      ; |  
```

当乘以 2 的 n 次方的时候可以采用位移的方式。

`Shr`向右移

其实就是一个乘方关系不是很难理解，但是要大家注意，不要看着`shr`就想着用〉〉指令可读性肯定没有用`/??`好

但是如果是求模式不同的

看代码：

```
d=a%2;  
  c=b%4;  
```

会变为

```
00401039   |.  8B5424 0C            mov edx,dword ptr ss:[esp+C]  
0040103D   |.  8B4424 10            mov eax,dword ptr ss:[esp+10]  
00401041   |.  83E2 01              and edx,1
```
