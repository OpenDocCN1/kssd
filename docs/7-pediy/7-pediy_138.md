

# 某些游戏对于特征码检测修改程序段的实现

> 原文：[`www.pediy.com/kssd/pediy10/96877.html`](https://www.pediy.com/kssd/pediy10/96877.html)

未经作者许可,禁止转载.  
  
如果您有更好的方法.请跟贴告诉我..这个方法不知道别人有没有讲过..随便写写.不要丢鸡蛋给我(要丢就丢 money 吧![](img/f43c1e514327696dccbb0da997496b2a.jpg "扮鬼脸"))  
  
很多游戏对自身的程序段做了效验.如果修改的话,直接退出..比如冒险岛.我个人使用的方法实现很烦琐.有时候得直接内联很多游戏的函数去实现..如果能找到游戏的效验部分.那直接能解决..如果找不到呢?..那只好试试我的方法吧.毕竟功能能实现了.烦琐点又如何呢?.第一次发贴,如有不对之处,请指正  
  
  
现在开始讲.(游戏名字不说了)  
  
改之前  
004D2080    55              push    ebp  
004D2081    8BEC            mov     ebp, esp  
004D2083    56              push    esi  
004D2084    8BF1            mov     esi, ecx  
004D2086    8A86 6C040000   mov     al, byte ptr \[esi+46C\]  
004D208C    84C0            test    al, al  
004D208E    0F84 E5000000   je      004D2179        假如我们要把这里的 je 改成 jmp,从而实现我们需要的功能  
004D2094    E8 A790FBFF     call    0048B140  
004D2099    84C0            test    al, al  
004D209B    75 0B           jnz     short 004D20A8  
  
改之后  
004D2080    55              push    ebp  
004D2081    8BEC            mov     ebp, esp  
004D2083    56              push    esi  
004D2084    8BF1            mov     esi, ecx  
004D2086    8A86 6C040000   mov     al, byte ptr \[esi+46C\]  
004D208C    84C0            test    al, al  
004D208E    0F84 E5000000   jmp   004D2179        改了这里  
004D2094    E8 A790FBFF     call    0048B140  
004D2099    84C0            test    al, al  
004D209B    75 0B           jnz     short 004D20A8  
  
  
如果游戏对 4d2080-4d209B 的位置这里做了效验的话,那么没多久就会断线或者自动关闭,至少 np,hs 我遇到的都是这种情况,不改又实现不了我们的功能.  
  
我们可以返回上一级.看看, CTRL+F9 返回  
  
  
0042565F    8B07            mov     eax, dword ptr \[edi\]  
00425661    53              push    ebx  
00425662    53              push    ebx  
00425663    53              push    ebx  
00425664    53              push    ebx  
00425665    53              push    ebx  
00425666    8BCF            mov     ecx, edi  
00425668    FF90 84020000   call    dword ptr \[eax+0C\]  <--返回到这里.  
  
00425668    FF90 84020000   call    dword ptr \[eax+0C\]  这是个虚函数调用..  
所有程序都会有虚函数调用..  
  
我们可以从这里入手.我这里是打个比方,并非是所有 CALL 返回上一级都是虚函数调用的.也许有 2 级\`或者 3 级..4.5.6.级. 返回几级,就得内联几级函数调用..  
  
我们先来找 eax 的值..  
  
假设为 d \[\[908854\]\]+C. = call    dword ptr \[eax+0C\] (004D2080)  
  
写个 DLL..  
注入到目标程序内,注入代码我就不多废话了.  
自己写个函数  
\_\_declspec(naked) VODI WINAPI Test(DWORD dwUnknow1, DWORD dwUnknow2, DWORD dwUnknow3, DWORD dwUnknow4, DWORD dwUnknow5)  
{  
        \_\_asm  
       {  
                push    ebp  
                mov     ebp, esp  
                push    esi  
                mov     esi, ecx  
                mov     al, byte ptr \[esi+46C\]  
                test    al, al  
                jmp      004D2179     //直接修改这里为跳转 jmp  
                call    0048B140  
                test    al, al  
                jnz     short 004D20A8  
                .....  
                ....  
               .....  
               .....  
       }  
}  
  
  
OD 有直接复制代码函数的插件.直接抓出来就可以用了..  
  
直接将  d \[\[908854\]\]+C 的函数入口指向 Test 里即可实现所想要的功能了.  
\*(PDWORD)(\*(PDWORD)PtrToULong(\*(PDWORD)PtrToULong(0x00908854))+0xC) = ULongToPtr(Test);  
  
  
  
不管几层,都得内联,找到调用源为虚函数调用的位置即可修改.  
也可以自己加个全局变量控制功能的开关..  
  
睡觉去了![](img/022c916b9660e467a8663bd57a4ae6d1.jpg "狂笑")![](img/022c916b9660e467a8663bd57a4ae6d1.jpg "狂笑")![](img/022c916b9660e467a8663bd57a4ae6d1.jpg "狂笑")......(完毕)