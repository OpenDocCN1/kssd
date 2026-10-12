
# 从逆向工程的角度来看 C++ (八)

> 原文：[`www.pediy.com/kssd/pediy10/87594.html`](https://www.pediy.com/kssd/pediy10/87594.html)

进入本次正题：

(八)\[ 标准 C++ 之 内置类型--容器 \]

今次来看看标准 C++内置类型之 `vector` 是个什么样子:

先看 Debug 版本的,等下再看 Release 的.

我就直接贴代码了.

还是把 `cpp` 代码贴上:

```cpp
#include <iostream>

#include <string>

#include <vector>

using namespace std;

//容器

void fuck_vector()

{
     int i =0;
     vector< int > ivec(4);
             __asm nop
             __asm nop
             __asm nop
             __asm nop
    printf("sizeof ivec %d \n",sizeof(ivec));
     ivec[2] += 2;
     i += ivec[2];
}

//函数模板

template <class T>

T t_func(T a ,T b)

{
     return a < b ? a : b ;
}

//类模板

template<class T>

class A

{
public:
     A();
     A(T _a,T _b);
     T sum();
private:
     T a;
     T b;
};

template <class T>

A<T>::A()

{
     a=0;b=0;
}

template<class T>

A<T>::A(T _a,T _b)

{
     a=_a;b=_b;
}

template<class T>

T A<T>::sum()

{
     return (a+b);
}

int main(int argc, char *argv[], char **env)

{
     int i =0;
     __asm int 3
     fuck_vector();        //容器测试
     i = t_func(2.2,3.3); //函数模板测试
     A<int> ai(3,4);                                                     //
     A<double> ad(3.1,4.0);                                     //     类模板测试
     cout<<ai.sum()<<" "<<ad.sum()<<endl;      //
     return 0;
}

--D:\Reverse\C++\Lesson8\Lesson8.cpp --------------------------------------------------------------

1:    #include <iostream>
2:    #include <string>
3:    #include <vector>
4:    using namespace std;
5:
6:    //容器
7:    void fuck_vector()
8:    {
00401650 55                 push        ebp
00401651 8B EC              mov         ebp,esp
00401653 6A FF              push        0FFh
00401655 68 79 87 44 00     push        offset __ehhandler$?fuck_vector@@YAXXZ (00448779)
0040165A 64 A1 00 00 00 00  mov         eax,fs:[00000000]
00401660 50                 push        eax
00401661 64 89 25 00 00 00 00 mov         dword ptr fs:[0],esp

//从这行向上数 5 句是按 SHE 的.所以我们知道使用 vector 的 call 体的内部是要被编译器 SHE 的.

00401668 83 EC 60           sub         esp,60h
0040166B 53                 push        ebx
0040166C 56                 push        esi
0040166D 57                 push        edi
0040166E 8D 7D 94           lea         edi,[ebp-6Ch]
00401671 B9 18 00 00 00     mov         ecx,18h
00401676 B8 CC CC CC CC     mov         eax,0CCCCCCCCh
0040167B F3 AB              rep stos    dword ptr [edi]
:        int i =0;
0040167D C7 45 F0 00 00 00 00 mov         dword ptr [ebp-10h],0

                             //因为上面装 SHE 使得栈抬高了 0ch.

0:       vector< int > ivec(4);
0401684 C7 45 D8 00 00 00 00 mov         dword ptr [ebp-28h],0 //如果我们把 i 作为 var1, 则这里可以假定为 var7. var7 下面除了 var1 外还有 var2-6 5 个 DWORD 值. 这里它要干什么我们先不管.

0040168B 8D 45 DC           lea         eax,[ebp-24h]
0040168E 50                 push        eax          //将 var6 的地址入栈.
0040168F 8D 4D D8           lea         ecx,[ebp-28h]
00401692 51                 push        ecx          //var7 的地址入栈
00401693 6A 04              push        4            //vector 的元素个数入栈
00401695 8D 4D E0           lea         ecx,[ebp-20h] //这里有点像 this 指针. 不过 vector 的内存是在堆上分配的, 所以我猜这里的一段栈内存只是 vector 取数据后存放的一个 buffer, Release 版本的应该不会这样, 先接着看吧.
00401698 E8 E8 FB FF FF     call        @ILT+640(std::vector<int,std::allocator<int>

>::vector<int,std::allocator<int> ) (00401      //调用构造函数,智能的分配内存.

0040169D C7 45 FC 00 00 00 00 mov         dword ptr [ebp-4],0 //把上面那个压进去的 SHE 的标志置零. 告诉 SHE 一些信息 . :D
1:       ivec[2] += 2;                       //容器的取数.
004016A4 6A 02              push      2    //下标拿出来.
004016A6 8D 4D E0           lea         ecx,[ebp-20h]    //var5 的地址入栈. (记得 var1 是 i 啊 , 那么你想 var 2,3,4,5 正好 4 个 DWORD 了. )
004016A9 E8 14 FB FF FF     call        @ILT+445(std::vector<int,std::allocator<int> >::operator[]) (004011c2)                 //vector 这里是重载了[]运算符. 取数. 在这里它取出的不是 vector[2]的数值, 而是其地址.
004016AE 89 45 D4           mov         dword ptr [ebp-2Ch],eax   //var8 保存地址
004016B1 8B 55 D4           mov         edx,dword ptr [ebp-2Ch]   //再用 edx 保存地址; 这里你就可以体会到 Debug 版本的嗦了. 不过有时用 debug 版本看思路会清晰些.
004016B4 8B 02              mov         eax,dword ptr [edx]
004016B6 83 C0 02           add         eax,2
004016B9 8B 4D D4           mov         ecx,dword ptr [ebp-2Ch] //地址又放到 ecx.
004016BC 89 01              mov         dword ptr [ecx],eax   //这四句完成对 ivec 的加 2 操作.
2:       i += ivec[2];                         //这句跟上面差不多, 到这里我发现, 上面提到的 var2-5 并没有被使用, 那个 ecx 也不是 this 指针. 可能是 Debug 版本的原因吧, 到了 Release,就不会有这个问题了.
004016BE 6A 02              push        2
004016C0 8D 4D E0           lea         ecx,[ebp-20h]
004016C3 E8 FA FA FF FF     call        @ILT+445(std::vector<int,std::allocator<int> >::operator[]) (004011c2)
004016C8 8B 55 F0           mov         edx,dword ptr [ebp-10h]
004016CB 03 10              add         edx,dword ptr [eax]
004016CD 89 55 F0           mov         dword ptr [ebp-10h],edx
3:   }
004016D0 C7 45 FC FF FF FF FF mov         dword ptr [ebp-4],0FFFFFFFFh    //是 SHE 相关的信息处理. 善后处理吧.
004016D7 8D 4D E0           lea         ecx,[ebp-20h]
004016DA E8 0F FC FF FF     call        @ILT+745(std::vector<int,std::allocator<int> >::~vector<int,std::allocator<int> ) (0040    //析构对象 , 看这里是不是觉得 ecx 特像 this 指针, 后面告诉你答案.
004016DF 8B 4D F4           mov         ecx,dword ptr [ebp-0Ch]
004016E2 64 89 0D 00 00 00 00 mov         dword ptr fs:[0],ecx
004016E9 5F                 pop         edi
004016EA 5E                 pop         esi
004016EB 5B                 pop         ebx
004016EC 83 C4 6C           add         esp,6Ch
004016EF 3B EC             cmp         ebp,esp
004016F1 E8 CA 01 02 00    call        __chkesp (004218c0)
004016F6 8B E5             mov         esp,ebp
004016F8 5D                 pop         ebp
004016F9 C3                 ret
```

Debug 版本容器小结: 刚才大致跟了下, 发现上面几处说到的 `ecx` 中并不是存放 `this` 指针的, 真正存了 `ivec` 这个对象的指针的是 `DWORD ptr [ecx+4]` (= `30xxxxxx`, 一个堆上的地址.), 所以我觉得 `debug` 版本的,用于直接在 `VC6` 的界面中看 `Dasm` 有时还是行的(它符号齐全), 不过要真分析起来, 还是 `Release` 版本的跟实际接近些. 下面看看 `Release` 版本的吧.

////Release:

//main 调用 `fuck_vector()`

```asm
00401050 /$ 56             push    esi                          ; 这里为什么要这么压栈了?我想应该是为了开辟 8 个字节的局部变量空间.(这里是 main 函数,进来就直接两个 push,优化的挺厉害的.)
00401051 |. 57             push    edi                          ; 如上.
00401052 |. E8 A9FFFFFF     call    Lesson8.00401000
//进来这个子呼叫.
00401000 /$ 55             push    ebp
00401001 |. 8BEC           mov     ebp, esp
00401003 |. 83EC 10        sub     esp, 10
00401006 |. 6A 10          push    10                           ; 容器中 4 个 int 所以压入大小 16
00401008 |. E8 1C520000    call    Lesson8.00406229             ; 初始化这个容器.
0040100D |. 83C4 04        add     esp, 4                       ; 没想到容器初始化竟是 C 调用, 没错吧这里?
00401010 |. 8BC8          mov     ecx, eax                     ; 刚才那个函数返回值为容器地址.在堆上.
00401012 |. BA 04000000    mov     edx, 4                       ; 容器中个数.
00401017 |> 85C9          /test    ecx, ecx
00401019 |. 74 06         |je      short Lesson8.00401021       ; 堆申请失败则玩完
0040101B |. C701 00000000 |mov     dword ptr [ecx], 0
00401021 |> 83C1 04      |add     ecx, 4
```



00401024 |. 4A             |dec     edx

00401025 |.^ 75 F0         \\jnz     short Lesson8.00401017            ; 多谢,容器帮我们将元素初始化为零

00401027 |. 8D48 10       lea     ecx, \[eax+10\]

0040102A |. 894D F8       mov     \[local.2\], ecx                     ; var2 = &ivec\[4\] ? 这里不太理解.,

0040102D |. 90          nop

0040102E |. 90          nop

0040102F |. 90          nop

00401030 |. 90          nop

00401031 |. 8B50 08     mov     edx, \[eax+8\]                          ; edx = ivec\[2\]

00401034 |. 50         push    eax                              ; 为析构函数压参, 跟下面这句可没关系.

00401035 |. 83C2 02     add     edx, 2

00401038 |. 8950 08     mov     \[eax+8\], edx                     ; ivec\[2\] += 2

0040103B |. E8 00290000   call    Lesson8.00403940                  ; 容器的析构函数.(跟进去会有 VirtualFree)

00401040 |. 83C4 04     add     esp, 4                            ; 从这也可知析构函数为 C 调用且一个参数.

00401043 |. 8BE5         mov     esp, ebp

00401045 |. 5D          pop     ebp

00401046 \\. C3         ret

```
00401024 |. 4A             |dec     edx

00401025 |.^ 75 F0         \\jnz     short Lesson8.00401017            ; 多谢,容器帮我们将元素初始化为零

00401027 |. 8D48 10       lea     ecx, \[eax+10\]

0040102A |. 894D F8       mov     \[local.2\], ecx                     ; var2 = &ivec\[4\] ? 这里不太理解.,

0040102D |. 90          nop

0040102E |. 90          nop

0040102F |. 90          nop

00401030 |. 90          nop

00401031 |. 8B50 08     mov     edx, \[eax+8\]                          ; edx = ivec\[2\]

00401034 |. 50         push    eax                              ; 为析构函数压参, 跟下面这句可没关系.

00401035 |. 83C2 02     add     edx, 2

00401038 |. 8950 08     mov     \[eax+8\], edx                     ; ivec\[2\] += 2

0040103B |. E8 00290000   call    Lesson8.00403940                  ; 容器的析构函数.(跟进去会有 VirtualFree)

00401040 |. 83C4 04     add     esp, 4                            ; 从这也可知析构函数为 C 调用且一个参数.

00401043 |. 8BE5         mov     esp, ebp

00401045 |. 5D          pop     ebp

00401046 \\. C3         ret
```

小结: 其实这从没对容器的构造及析构函数好好的跟踪, 有时间仔细的跟跟, 这次也只是有个大概了解. 其实在逆向中, 知道这些也就够了, 因为现在的工具太好用了, 已经可以让 `OD` 自动把 `string` 啊 `vector` 啊什么的构造函数或者重载的运算符自己显示出来了.

下一 `Lesson` 再分析下模板. 今天就到这吧.
