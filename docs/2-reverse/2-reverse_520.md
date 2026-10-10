

# 我之前在 CSDN 上发表的一篇原创！通过汇编角度理解虚函数。

> 原文：[`www.pediy.com/kssd/pediy10/78341.html`](https://www.pediy.com/kssd/pediy10/78341.html)

这是我当时学习 C++时写的一篇文章，当时正好了解了一下汇编就写了出来。由于 CSDN 帐号注销了文章丢了。当时也放在了 QQ 空间上，由于字数限制，少了一点。但还是说明了原理。希望能够帮助到大家。o(∩\_∩)o...。高手略过。  
  
今天通过汇编角度在次理解一下虚函数.  
工作环境 VC6  
方法 DEBUG 跟踪 MEMORY  
测试程序代码 虽然是测试程序但希望大家还是养成良好习惯 不要污染命名空间  
测试代码:  
#include "iostream"  
using std::cout;  
using std::endl;  
class a  
{  
  int m\_a;  
public:  
  a(int x):m\_a(x) {}  
  virtual void display()  
  {  
    cout<<"a is running !"<<endl;  
  }  
};  
class b : public a  
{  
public:  
  b(int y):a(y) {}  
  void display()  
  {  
    cout<<"b is running !"<<endl;  
  }  
};  
class c : public b  
{  
public:  
  c(int z):b(z) {}  
  void display()  
  {  
    cout<<"c is running !"<<endl;  
  }  
};  
int main()  
{  
  a ca(1);  
  b cb(2);  
  c cc(3);  
  a \* p\[3\]={ &ca , &cb , &cc };  
  p\[0\]->display();  
  p\[1\]->display();  
  p\[2\]->display();  
  return 0;  
}  
输出结果就不用多说了,如果你还不能想到正确结果 那先看一下基础知识在看这篇文章.  
先看一下生成的汇编代码  
39:   int main()  
40:   {  
004011A0   push        ebp  
004011A1   mov         ebp,esp  
004011A3   sub         esp,64h  
004011A6   push        ebx  
004011A7   push        esi  
004011A8   push        edi  
004011A9   lea         edi,\[ebp-64h\]  
004011AC   mov         ecx,19h  
004011B1   mov         eax,0CCCCCCCCh  
004011B6   rep stos    dword ptr \[edi\]  
41:           a ca(1);  
004011B8   push        1  
004011BA   lea         ecx,\[ebp-8\]  
004011BD   call        @ILT+95(a::a) (00401064)  
42:           b cb(2);  
004011C2   push        2  
004011C4   lea         ecx,\[ebp-10h\]  
004011C7   call        @ILT+10(b::b) (0040100f)  
43:           c cc(3);  
004011CC   push        3  
004011CE   lea         ecx,\[ebp-18h\]  
004011D1   call        @ILT+105(c::c) (0040106e)  
44:  
45:           a \* p\[3\]={ &ca , &cb , &cc };  
004011D6   lea         eax,\[ebp-8\]  
004011D9   mov         dword ptr \[ebp-24h\],eax  
004011DC   lea         ecx,\[ebp-10h\]  
004011DF   mov         dword ptr \[ebp-20h\],ecx  
004011E2   lea         edx,\[ebp-18h\]  
004011E5   mov         dword ptr \[ebp-1Ch\],edx  
46:  
47:           p\[0\]->display();  
004011E8   mov         eax,dword ptr \[ebp-24h\]  
004011EB   mov         edx,dword ptr \[eax\]  
004011ED   mov         esi,esp  
004011EF   mov         ecx,dword ptr \[ebp-24h\]  
004011F2   call        dword ptr \[edx\]  
004011F4   cmp         esi,esp  
004011F6   call        \_\_chkesp (004092c0)  
48:           p\[1\]->display();  
004011FB   mov         eax,dword ptr \[ebp-20h\]  
004011FE   mov         edx,dword ptr \[eax\]  
00401200   mov         esi,esp  
00401202   mov         ecx,dword ptr \[ebp-20h\]  
00401205   call        dword ptr \[edx\]  
00401207   cmp         esi,esp  
00401209   call        \_\_chkesp (004092c0)  
49:           p\[2\]->display();  
0040120E   mov         eax,dword ptr \[ebp-1Ch\]  
00401211   mov         edx,dword ptr \[eax\]  
00401213   mov         esi,esp  
00401215   mov         ecx,dword ptr \[ebp-1Ch\]  
00401218   call        dword ptr \[edx\]  
0040121A   cmp         esi,esp  
0040121C   call        \_\_chkesp (004092c0)  
50:  
51:           return 0;  
00401221   xor         eax,eax  
  
我们只需要看一下重要的代码就可以了  
41:           a ca(1);  
004011B8   push        1  
004011BA   lea         ecx,\[ebp-8\]  
004011BD   call        @ILT+95(a::a) (00401064)  
构造 A 类对象,先压栈 常量 1 作为参数.  
将栈中的一个地址给 ECX ,此时 ECX 要作为该对象的 THIS 指针.现在的这块内存是全 CC CC CC CC....  
调用 A 类的构造函数.因为不是 VIRTUAL FNCTION 所以是 CALL 一个常量地址,到一个跳表  
00401064   jmp         a::a (00401260)  
进入到 A 类的构造函数.  
9:            a(int x):m\_a(x) {}  
00401260   push        ebp  
00401261   mov         ebp,esp  
00401263   sub         esp,44h  
00401266   push        ebx  
00401267   push        esi  
00401268   push        edi  
00401269   push        ecx  
0040126A   lea         edi,\[ebp-44h\]  
0040126D   mov         ecx,11h  
00401272   mov         eax,0CCCCCCCCh  
00401277   rep stos    dword ptr \[edi\]  
00401279   pop         ecx  
0040127A   mov         dword ptr \[ebp-4\],ecx  
0040127D   mov         eax,dword ptr \[ebp-4\]  
00401280   mov         ecx,dword ptr \[ebp+8\]  
00401283   mov         dword ptr \[eax+4\],ecx  
00401286   mov         edx,dword ptr \[ebp-4\]  
00401289   mov         dword ptr \[edx\],offset a::\`vftable' (0043201c)  
0040128F   mov         eax,dword ptr \[ebp-4\]  
00401292   pop         edi  
00401293   pop         esi  
00401294   pop         ebx  
00401295   mov         esp,ebp  
00401297   pop         ebp  
00401298   ret         4  
  
我们只看最重要的几行就可以了  
0040127A   mov         dword ptr \[ebp-4\],ecx  
.....  
0040128F   mov         eax,dword ptr \[ebp-4\]  
  
此时 ECX 是该对象的 THIS 指针.把 THIS 指针给一个变量(栈来实现) . 在给 EAX 所以此时 EAX 也是 THIS 指针.  
mov         ecx,dword ptr \[ebp+8\]  
实现了将刚才压入栈中的参数 1 给 ECX.  
mov         dword ptr \[eax+4\],ecx  
该值在给以 THISI 指针开始+4 到+8 的 4 个字节赋值(因为是 INT 型 所以为 4 个字节.)  因为 A 类有虚函数,我们知道 编译器会给 A 类对象的前 4 个字节赋予类 A 的 VTABLE 的地址,这 4 个字节就是我们常说的 VPTR 它指向虚表.所以 VPTR 后的 4 个字节才是存放对象的唯一一个数据成员(INT).  
此时该对象的内存为 CC CC CC CC 01 00 00 00  VPTR 还没赋值  
mov         edx,dword ptr \[ebp-4\]  
THIS 指针给 EDX  
mov         dword ptr \[edx\],offset a::\`vftable' (0043201c)  
把类 A 虚表地址 给 VPTR   也就是 THIS 指针开始的 4 个字节  
eax,dword ptr \[ebp-4\]  还是把 THIS 指针给 EAX 我认为这是一条废语句  
此时该对象的内存是  
0012FF78  1C 20 43 00 01 00 00 00    虚表指针  数据成员  
类 B  
42:           b cb(2);  
004011C2   push        2  
004011C4   lea         ecx,\[ebp-10h\]  
004011C7   call        @ILT+10(b::b) (0040100f)  
同样压栈 2 作为参数.  并找到下一个 8 字节空间作为类 B 对象使用的内存  
并进入跳表 0040100F   jmp         b::b (00401310)  
进入构造函数  
20:           b(int y):a(y) {}  
00401310   push        ebp  
00401311   mov         ebp,esp  
00401313   sub         esp,44h  
00401316   push        ebx  
00401317   push        esi  
00401318   push        edi  
00401319   push        ecx  
0040131A   lea         edi,\[ebp-44h\]  
0040131D   mov         ecx,11h  
00401322   mov         eax,0CCCCCCCCh  
00401327   rep stos    dword ptr \[edi\]  
00401329   pop         ecx  
0040132A   mov         dword ptr \[ebp-4\],ecx  
0040132D   mov         eax,dword ptr \[ebp+8\]  
00401330   push        eax  
00401331   mov         ecx,dword ptr \[ebp-4\]  
00401334   call        @ILT+95(a::a) (00401064)  
00401339   mov         ecx,dword ptr \[ebp-4\]  
0040133C   mov         dword ptr \[ecx\],offset b::\`vftable' (00432034)  
00401342   mov         eax,dword ptr \[ebp-4\]  
00401345   pop         edi  
00401346   pop         esi  
00401347   pop         ebx  
00401348   add         esp,44h  
0040134B   cmp         ebp,esp  
0040134D   call        \_\_chkesp (004092c0)  
00401352   mov         esp,ebp  
00401354   pop         ebp  
00401355   ret         4  
主要来看这段  
0040132A   mov         dword ptr \[ebp-4\],ecx  
0040132D   mov         eax,dword ptr \[ebp+8\]  
00401330   push        eax  
00401331   mov         ecx,dword ptr \[ebp-4\]  
00401334   call        @ILT+95(a::a) (00401064)  
00401339   mov         ecx,dword ptr \[ebp-4\]  
0040133C   mov         dword ptr \[ecx\],offset b::\`vftable' (00432034)  
00401342   mov         eax,dword ptr \[ebp-4\]  
ECX 还是该对象的 THIS 指针.给一个变量  
把参数 2 给 EAX,把 EAX 压栈 作为参数来调用基类的构造函数. 函数和上边说的一样,所以返回时该对象内存为  
1C 20 43 00 02 00 00 00 此时虚表指针指向的是类 A 的 VTABLE  
ecx,dword ptr \[ebp-4\]  
把 THIS 指针给 ECX  
dword ptr \[ecx\],offset b::\`vftable' (00432034)  
把 B 类的 VTABLE 地址给 THIS 指针开始的 4 个字节 (VPTR)  所以现在对象的内存是  
34 20 43 00 02 00 00 00  
  
类 C 的和前边两个基本相同.最后该对象的内存是 4C 20 43 00 03 00 00 00  
最后 3 个对象的内存为  
4C 20 43 00     03 00 00 00        34 20 43 00        02 00 00 00  烫烫 L C.....4 C.....  
0012FF78           1C 20 43 00       01 00 00 00  
45:           a \* p\[3\]={ &ca , &cb , &cc };  
004011D6   lea         eax,\[ebp-8\]  
004011D9   mov         dword ptr \[ebp-24h\],eax  
004011DC   lea         ecx,\[ebp-10h\]  
004011DF   mov         dword ptr \[ebp-20h\],ecx  
004011E2   lea         edx,\[ebp-18h\]  
004011E5   mov         dword ptr \[ebp-1Ch\],edx  
把 3 个对象的 THIS 指针赋值给 3 个 4 字节的指针数组.  
  
47:           p\[0\]->display();  
004011E8   mov         eax,dword ptr \[ebp-24h\]  
004011EB   mov         edx,dword ptr \[eax\]  
004011ED   mov         esi,esp  
004011EF   mov         ecx,dword ptr \[ebp-24h\]  
004011F2   call        dword ptr \[edx\]  
004011F4   cmp         esi,esp  
004011F6   call        \_\_chkesp (004092c0)  
48:           p\[1\]->display();  
004011FB   mov         eax,dword ptr \[ebp-20h\]  
004011FE   mov         edx,dword ptr \[eax\]  
00401200   mov         esi,esp  
00401202   mov         ecx,dword ptr \[ebp-20h\]  
00401205   call        dword ptr \[edx\]  
00401207   cmp         esi,esp  
00401209   call        \_\_chkesp (004092c0)  
49:           p\[2\]->display();  
0040120E   mov         eax,dword ptr \[ebp-1Ch\]  
00401211   mov         edx,dword ptr \[eax\]  
00401213   mov         esi,esp  
00401215   mov         ecx,dword ptr \[ebp-1Ch\]  
00401218   call        dword ptr \[edx\]  
0040121A   cmp         esi,esp  
0040121C   call        \_\_chkesp (004092c0)  
51:           return 0;  
00401221   xor         eax,eax  
  
这里就是虚函数调用,他们因为虚表不同,所以输出的结果 调用的函数是不同的.  
先看看  
47:           p\[0\]->display();  
004011E8   mov         eax,dword ptr \[ebp-24h\]  
004011EB   mov         edx,dword ptr \[eax\]  
004011ED   mov         esi,esp  
004011EF   mov         ecx,dword ptr \[ebp-24h\]  
004011F2   call        dword ptr \[edx\]  
004011F4   cmp         esi,esp  
004011F6   call        \_\_chkesp (004092c0)  
  
把第一个对象的 THIS 指针给 EAX,    
mov         edx,dword ptr \[eax\]  
THIS 指针开始的 4 个字节取内容.给 EDX, 此时 EDX 就是 VPTR,也就是 VTABLE 的地址  
mov         ecx,dword ptr \[ebp-24h\]  
把 THIS 指针在给 ECX  
call        dword ptr \[edx\]  
每个类的虚函数在虚表中的位置就靠它在类中声明的位置而定,没个虚函数的位置就做为了 VTABLE 中的 ID,因为测试代码为每个类只定义了一个 VIRTUAL FUNCTION 所以这里就 CALL 了 EDX 指向的 VTABLE 开始的 4 字节的那个为一一个 VIRTUAL 函数地址  
跟踪一下内存  
0012FF78  1C 20 43 00   
跟踪\[eax\]      EDX = 0043201C  
跟踪\[edx\]     0043201C         5F 10 40 00  
  
跟踪     0040105F   jmp         a::display (004012b0)  
于是调用了 A 类的 DISPLAY()函数  
后边的两个调用一样会利用 VTABLE 来调用虚函数.