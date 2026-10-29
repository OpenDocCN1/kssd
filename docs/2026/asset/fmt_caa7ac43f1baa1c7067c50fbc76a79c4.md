
在调试漏洞的时候，经常需要定位导致漏洞的代码和函数，比如导致堆溢出的字节复制指令`rep movsz`等，前面的堆尾检查方式主要是堆被破坏的场景，不利于定位导致漏洞的代码。为此引入了页堆的概念，开启后，会在堆块中增加不可访问的栅栏页，溢出覆盖到栅栏页就会触发异常。

### 开启页堆

```
gflags.exe -i F:\vulns\Release\heapoverflow +hpa
```

![](upload/attach/201907/823237_JDMHTJ6E7VK7UPZ.png)

用`windbg`加载`heapoverflow`，运行！`gflag`命令开启了页堆，然后`g`运行后在`cmd`按下回车键断下

```
0:000> g
(46c.b74): Access violation - code c0000005 (!!! second chance !!!)
eax=00000021 ebx=01795ff0 ecx=00000004 edx=77d364f4 esi=0012ff38 edi=01796000
eip=00401084 esp=0012ff10 ebp=01790000 iopl=0 nv up ei pl nz na po nc
cs=001b ss=0023 ds=0023 es=0023 fs=003b gs=0000 efl=00010202
image00400000+0x1084:
00401084 f3a5 rep movs dword ptr es:[edi],dword ptr [esi]
0:000> dd esi
0012ff38 41414141 41414141 41414141 41414141
0012ff48 00407000 00401327 00000001 01699fb0
0012ff58 0169bf70 00000000 00000000 7ffdd000
0012ff68 c0000005 00000000 0012ff5c 0012fb1c
0012ff78 0012ffc4 00402c50 004060b8 00000000
0012ff88 0012ff94 76281174 7ffdd000 0012ffd4
0012ff98 77d4b3f5 7ffdd000 77cb48a4 00000000
0012ffa8 00000000 7ffdd000 c0000005 76292b35
0:000> dc edi
01796000 ???????? ???????? ???????? ???????? ????????????????
01796010 ???????? ???????? ???????? ???????? ????????????????
01796020 ???????? ???????? ???????? ???????? ????????????????
01796030 ???????? ???????? ???????? ???????? ????????????????
01796040 ???????? ???????? ???????? ???????? ????????????????
01796050 ???????? ???????? ???????? ???????? ????????????????
01796060 ???????? ???????? ???????? ???????? ????????????????
01796070 ???????? ???????? ???????? ???????? ????????????????
```

可以发现程序在复制`A`字符串的时候触发了异常，程序复制到`0x11`字节的时候被断下，此时异常还未破坏到堆块，直接定位导致溢出的复制指令`rep movs`

```
0:000> kb
ChildEBP RetAddr Args to Child
WARNING: Stack unwind information not available. Following frames may be wrong.
0012ff48 00401327 00000001 01699fb0 0169bf70 image00400000+0x1084
*** ERROR: Symbol file could not be found. Defaulted to export symbols for C:\Windows\system32\kernel32.dll -
0012ff88 76281174 7ffdd000 0012ffd4 77d4b3f5 image00400000+0x1327
0012ff94 77d4b3f5 7ffdd000 77cb48a4 00000000 kernel32!BaseThreadInitThunk+0x12
0012ffd4 77d4b3c8 0040b000 7ffdd000 00000000 ntdll!RtlInitializeExceptionChain+0x63
0012ffec 00000000 0040b000 7ffdd000 00000000 ntdll!RtlInitializeExceptionChain+0x36
0:000> ub image00400000+0x1327
image00400000+0x1301:
00401301 e847120000 call image00400000+0x254d (0040254d)
00401306 e8af0e0000 call image00400000+0x21ba (004021ba)
0040130b a150994000 mov eax,dword ptr [image00400000+0x9950 (00409950)]
00401310 a354994000 mov dword ptr [image00400000+0x9954 (00409954)],eax
00401315 50 push eax
00401316 ff3548994000 push dword ptr [image00400000+0x9948 (00409948)]
0040131c ff3544994000 push dword ptr [image00400000+0x9944 (00409944)]
00401322 e8d9fcffff call image00400000+0x1000 (00401000)
```

根据栈回溯，调用`rep movs`的上一层函数位于`image00400000+0x1084`的上一条指令，也就是`00401322`，此处调用了`00401000`函数，很容易发现这是主入口函数：

```
0:000> uf 00401000
image00400000+0x1000:
00401000 83ec24 sub esp,24h
00401003 b908000000 mov ecx,8
00401008 53 push ebx
00401009 55 push ebp
0040100a 56 push esi
0040100b 57 push edi
0040100c be44704000 mov esi,offset image00400000+0x7044 (00407044)
00401011 8d7c2410 lea edi,[esp+10h]
00401015 f3a5 rep movs dword ptr es:[edi],dword ptr [esi]
00401017 68ffff0000 push 0FFFFh
0040101c 6800100000 push 1000h //堆块大小压入
00401021 6a04 push 4
00401023 a4 movs byte ptr es:[edi],byte ptr [esi]
00401024 ff150c604000 call dword ptr [image00400000+0x600c (0040600c)]//调用HeapCreate创建堆块
0040102a 8be8 mov ebp,eax
0040102c a16c704000 mov eax,dword ptr [image00400000+0x706c (0040706c)]
00401031 48 dec eax
00401032 a36c704000 mov dword ptr [image00400000+0x706c (0040706c)],eax
00401037 7808 js image00400000+0x1041 (00401041)
image00400000+0x1039:
00401039 ff0568704000 inc dword ptr [image00400000+0x7068 (00407068)]
0040103f eb0d jmp image00400000+0x104e (0040104e)
image00400000+0x1041:
00401041 6868704000 push offset image00400000+0x7068 (00407068)
00401046 e896000000 call image00400000+0x10e1 (004010e1)
0040104b 83c404 add esp,4
image00400000+0x104e:
0040104e 6a10 push 10h
00401050 6a00 push 0
00401052 55 push ebp
00401053 ff1508604000 call dword ptr [image00400000+0x6008 (00406008)] //调用HeapAlloc分配0x10的堆块
00401059 8bd8 mov ebx,eax //分配的堆块地址
0040105b 53 push ebx
0040105c 6830704000 push offset image00400000+0x7030 (00407030)
00401061 e84a000000 call image00400000+0x10b0 (004010b0)
00401066 8d7c2418 lea edi,[esp+18h]
0040106a 83c9ff or ecx,0FFFFFFFFh
0040106d 33c0 xor eax,eax
0040106f 83c408 add esp,8
00401072 f2ae repne scas byte ptr es:[edi]
00401074 f7d1 not ecx //获取str长度
00401076 2bf9 sub edi,ecx
00401078 53 push ebx
00401079 8bc1 mov eax,ecx
0040107b 8bf7 mov esi,edi //str = 0x20
0040107d 8bfb mov edi,ebx //分配的堆块只有0x10
0040107f 6a00 push 0
00401081 c1e902 shr ecx,2
00401084 f3a5 rep movs dword ptr es:[edi],dword ptr [esi]
00401086 8bc8 mov ecx,eax
00401088 55 push ebp
00401089 83e103 and ecx,3
0040108c f3a4 rep movs byte ptr es:[edi],byte ptr [esi] //0x20 < 0x10 循环复制导致溢出
0040108e ff1504604000 call dword ptr [image00400000+0x6004 (00406004)]
00401094 55 push ebp
00401095 ff1500604000 call dword ptr [image00400000+0x6000 (00406000)]
0040109b 5f pop edi
0040109c 5e pop esi
0040109d 5d pop ebp
0040109e 33c0 xor eax,eax
004010a0 5b pop ebx
004010a1 83c424 add esp,24h
004010a4 c3 ret
```

## 0x04整数溢出漏洞原理

整数分为有符号和无符号两类，有符号数以最高位作为符号位，正整数最高位为`1`，负整数最高位为`0`，不同类型的整数在内存中有不同的取值范围，`unsigned int = 4`字节，`int = 4`字节，当存储的数值超过该类型整数的最大值就会发生溢出。

在一些有符号和无符号转换的过程中最有可能发生整数溢出漏洞。

![](upload/attach/201907/823237_XXJ4WN4XMZJZW55.png)

![](upload/attach/201907/823237_Z8U3BWFNKYR94WF.png)

### 基于栈的整数溢出

基于栈的整数溢出的例子：

```c
#include "stdio.h"
#include "string.h"
int main(int argc, char *argv){
int i;
char buf[8]; // 栈缓冲区
unsigned short int size; // 无符号短整数取值范围：0 ~ 65535
char overflow[65550];
memset(overflow,65,sizeof(overflow)); // 填充为“A”字符
printf("请输入数值:\n");
scanf("%d",&i); //输入65540
size = i;
printf("size：%d\n",size); // 输出系统识别出来的size数值 4
printf("i：%d\n",i); // 输出系统识别出来的i数据 65540
if (size > 8) //边界检查
return -1;
memcpy(buf,overflow,i); // 栈溢出
return 0;
}
```

代码中`size`变量是无符号短整型，取值范围是`0~65535`，输入的值大于`65535`就会发生溢出，最后得到`size`为`4`，这样会通过边界检查，但是用`memcpy`复制数据的时候，使用的是`int`类型的参数`i`，这个值是输入的`65540`，就会发生栈溢出：

![](upload/attach/201907/823237_KQH6XEUXZXP5CZ6.png)

### 基于堆的整数溢出

基于堆的整数溢出的例子：

```c
#include "stdio.h"
#include "windows.h"
int main(int argc, char * argv)
{
int* heap;
unsigned short int size; // 无符号短整数取值范围：0 ~ 65535
char *pheap1, *pheap2;
HANDLE hHeap;
printf("输入size数值：\n");
scanf("%d",&size);
hHeap = HeapCreate(HEAP_GENERATE_EXCEPTIONS, 0x100, 0xfff); //创建一个堆块
if (size <= 0x50)
{
size -= 5; //输入2，size=-3=65533，
printf("size：%d\n",size);
pheap1 = HeapAlloc(hHeap, 0, size); // pheap1会分配过大的堆块，导致溢出！
pheap2 = HeapAlloc(hHeap, 0, 0x50);
}
HeapFree(hHeap, 0, pheap1);
HeapFree(hHeap, 0, pheap2);
return 0;
}
```

代码中的`size`是`unsigned short int`类型，当输入小于`5`，`size`减去`5`会得到负数，但由于`unsigned short int`取值范围的限制无法识别负数，得到正数`65533`，最后分配得到过大的堆块，溢出覆盖了后面的堆管理结构：

![](upload/attach/201907/823237_PAADHV3M4WG4EKJ.png)

## 0x05 格式化字符串漏洞原理

格式化漏洞产生的原因主要是对用户输入的内容没有做过滤，有些输入数据都是作为参数传递给某些执行格式化操作的函数的，比如：`printf`，`fprintf`，`vprintf`，`sprintf`。

恶意用户可以使用`%s`和`%x`等格式符，从堆栈和其他内存位置输出数据，也可以使用格式符`%n`向任意地址写入数据，配合`printf()`函数就可以向任意地址写入被格式化的字节数，可能导致任意代码执行，或者读取敏感数据。

以下面的代码为例讲解格式化字符串漏洞原理：

```c
#include <stdio.h>
#include <string.h>
int main (int argc, char *argv[])
{
char buff[1024]; // 设置栈空间
strncpy(buff,argv[1],sizeof(buff)-1);
printf(buff); //触发漏洞
return 0;
}
```

可以发现当输入数据包含`%s`和`%x`格式符的时候，会意外输出其他数据：

![](upload/attach/201907/823237_8U5FKBQVD2BMETM.png)

用`ollydbg`附加调试程序，执行前需要先设置命令行参数，调试-参数-命令行：`test-%x`

![](upload/attach/201907/823237_R9EWRBXBUNS3HJ2.png)

在运行程序后，传递给`printf`的参数只有`test-%x`，但是他把输入参数`test-%x`之后的另一个栈上数据当做参数传给了`printf`函数，因为`printf`基本类型是：

```
printf（“格式化控制符”，变量列表）；
```

传递给`printf`的参数只有一个，但是程序默认将栈上的下一个数据作为参数传递给了`printf`函数，刚好下一个数据是`strcpy()`函数的目标地址，就是`buff`变量，`buff`刚好指向`test-%x`的地址`0x0019fec4`，所以程序会输出`0x0019fec4`，如果后面再加一个`%x`就会将`src`参数的值也输出了，这样就可以遍历整个栈上数据了。

![](upload/attach/201907/823237_TNYP7VTVUVZCP9E.png)

除了利用`%x`读取栈上数据，还可以用`%n`写入数据修改返回地址来实现漏洞利用。
