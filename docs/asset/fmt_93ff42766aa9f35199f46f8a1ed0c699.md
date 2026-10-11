
# C++ 逆向学习笔记 一

> 原文：[`www.pediy.com/kssd/pediy10/84227.html`](https://www.pediy.com/kssd/pediy10/84227.html)

## 前言

为了早日走进逆向工程的大门，为了某日能达到牛人们一样看反汇编像看源码一样的境界（这个当然难就哦，~），最近在学习 C++ 的逆向。以后我会定期上传自己的学习笔记不断激励自己，希望可以在一个较短的时间内完成。

**环境说明**：平台：`VC6 SP6`，一般都是分析 `Release` 版本的程序，这样更接近真实情况。（除非特殊说明~~）

在我的分析中，可能难免会出现错误，或者理解上的失误，希望各位看官可以即时的帮我指正，那样将不胜感激 ~~

## C++ 逆向笔记一 之 数据类型

来看 `CPP` 源码：

```cpp
#include <stdio.h>  

#pragma pack(1)  

struct F  
{  
  char a;  
  int  b;  
  void fuck();  
};  

#pragma pack()  

struct B  
{  
  int a;  
  int b;  
  int c;  
  void  add(int,int);  
  void f(int);  
};  

void B::add(int a,int b)  
{  
  c = a + b ;  
}  

void B::f(int x)  
{  
  a += x;  
}  

void F::fuck()  
{  
  this->a = 'J' ;  
  this->b++;  
}  

enum EE  
{  
  red  =3,  
  blue ,  
  green,  
  yellow,  
};  

int main(int argc, char* argv[])  
{  
  B sb; F sf; EE ee;  
  sb.a = 3;sb.b = 4;sb.c = 5;  
  sf.a = 'X' ; sf.b = 0x9999;  
  sf.b = green;          
  
  __asm int 3      //跟 failwest 学的, 呵呵, 感觉不错.  
  printf("%d   ",sizeof(sb));  
  printf("%d   ",sizeof(sf));  
  printf("%d   ",sizeof(ee));  
  sb.add(sb.a,sb.b);  
  sb.f(4);  
  sf.fuck();  
  return 0;  
}  
```

结构体 `B`、`F` 两种抽象数据类型，默认的话，其结构体大小为数据成员个数乘以 4。（如果定义了对齐粒度的话就另当别论）。函数不会算到大小里面去（后面的虚函数除外），对于一般的结构体，都是当成一个内存块来处理的。对于 `B`、`F` 这种抽象数据类型，一般都是将 `ecx` 保存其指针（以后到了类就是 `this` 了）。

下面看反汇编调试过程吧。

```asm
00401030    55              push ebp  
00401031    8BEC            mov ebp,esp  
00401033    83EC 14         sub esp,14  
00401036    C745 EC 0300000>mov dword ptr ss:[ebp-14],3            ; 从这 5 行可以看出,在栈里面是按局部变量从小到大排的  
0040103D    C745 F0 0400000>mov dword ptr ss:[ebp-10],4  
00401044    C745 F4 0500000>mov dword ptr ss:[ebp-C],5  
0040104B    C645 F8 58      mov byte ptr ss:[ebp-8],58             ; 这里从 8 开始一个 char  
0040104F    C745 F9 9999000>mov dword ptr ss:[ebp-7],9999          ; 从 7 开始, 保持和上面的紧挨着.  
00401056    90              nop  
00401057    6A 0C           push 0C                                 ; 默认自然对齐为 4 字节, 总共 12 字节  
00401059    68 30704000     push lesson1.00407030                   ; ASCII "%d   "  
0040105E    E8 3D000000     call lesson1.004010A0                   ; 这里是 printf, 不知为啥不自己显示出来  
00401063    6A 05           push 5                                  ; 这里重设了对齐粒度,于是为 5  
00401065    68 30704000     push lesson1.00407030                   ; ASCII "%d   "  
0040106A    E8 31000000     call lesson1.004010A0                   ; printf  
0040106F    8B45 F0         mov eax,dword ptr ss:[ebp-10]           ; sb.b  
00401072    8B4D EC         mov ecx,dword ptr ss:[ebp-14]           ; sb.a  
00401075    83C4 10         add esp,10                               ; 两次的放一起平衡 esp.  
00401078    50              push eax  
00401079    51              push ecx  
0040107A    8D4D EC         lea ecx,dword ptr ss:[ebp-14]           ; sb 的地址.  相当于寄存器传参.  
0040107D    E8 7EFFFFFF     call lesson1.00401000                   ; sb.add(sb.a,sb.b);  
00401082    6A 04           push 4  
00401084    8D4D EC         lea ecx,dword ptr ss:[ebp-14]           ; sb 的地址.  
00401087    E8 84FFFFFF     call lesson1.00401010                   ; sb.f(4);  
0040108C    8D4D F8         lea ecx,dword ptr ss:[ebp-8]            ; sf 的地址  
0040108F    E8 8CFFFFFF     call lesson1.00401020                   ; sf.fuck  
00401094    33C0            xor eax,eax  
00401096    8BE5            mov esp,ebp  
00401098    5D              pop ebp  
00401099    C3              retn  

//  
00401000    8B4424 08       mov eax,dword ptr ss:[esp+8]           ; 这里没压 ebp, 所以为参数 2 : sb.b  
00401004    8B5424 04       mov edx,dword ptr ss:[esp+4]           ; 参数一: sb.a  
00401008    03D0            add edx,eax  
0040100A    8951 08         mov dword ptr ds:[ecx+8],edx           ; 这里 ecx 就定位了那个 sb 结构体了.  
0040100D    C2 0800         retn 8  
```

枚举类型就很简单了，跟 `BOOL` 一样了，`sizeof` 就是 4 了，然后使用中就直接被编译器换成了数值了，比如 `EE ee = red;` 直接就是 `mov dword ptr [ebp  index * 4] , 3`；联合体在 `C++` 也用的不多了，也简单，就不分析了。
