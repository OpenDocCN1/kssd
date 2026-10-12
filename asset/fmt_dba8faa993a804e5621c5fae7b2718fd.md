
```asm
0041170D lea     edi, dword ptr [ebp-CC]          
00411713 mov     ecx, 33                          

00411718 mov     eax, CCCCCCCC                    

0041171D rep     stos dword ptr es:[edi]          

0041171F pop     ecx                              

00411720 mov     dword ptr [ebp-8], ecx           ；将 this 指针保存到 dword ptr [ebp-8]中  

00411723 mov     esi, esp                         

00411725 mov     eax, dword ptr [<&MSVCP80D.std::  

0041172A push    eax                              

0041172B mov     ecx, dword ptr [ebp-8]           

0041172E mov     edx, dword ptr [ecx]             ；将 this 指针的所指向的内容送给 edx  

00411730 mov     edi, esp                         

00411732 mov     ecx, dword ptr [ebp-8]           

00411735 mov     eax, dword ptr [edx]             ; 又将 edx 给 eax  

00411737 call    eax                              

00411739 cmp     edi, esp                         ; 获取体积，调用 eax  

0041173B call    004111D1                         

00411740 mov     edi, esp                         

00411742 push    eax                              

00411743 push    004179A8                         

00411748 mov     ecx, dword ptr [<&MSVCP80D.std::  

0041174E push    ecx                              

0041174F call    00411186                         

00411754 add     esp, 8                           

00411757 mov     ecx, eax                         

00411759 call    dword ptr [<&MSVCP80D.std::basic  

0041175F cmp     edi, esp                         

00411761 call    004111D1                         

00411766 mov     ecx, eax                         

00411768 call    dword ptr [<&MSVCP80D.std::basic  

0041176E cmp     esi, esp                         

00411770 call    004111D1                         

00411775 pop     edi                              

00411776 pop     esi                              

00411777 pop     ebx                              

00411778 add     esp, 0CC                         

0041177E cmp     ebp, esp                         

00411780 call    004111D1                         

00411785 mov     esp, ebp                         

00411787 pop     ebp                              

00411788 retn                                       
```

；========================================================

从上面的代码中，出现了我想看到的代码：`call eax`；奇怪的是 `eax` 的内容是传进来的 `this` 指针所指向的内容，但是我从之前了解的知识，`this` 指针指向的是类对象的首地址，也就是说是类的数据成员的地址。怎可能会根据类的数据成员的值去调用 `Volume()` 函数呢。难道是定义了虚函数后存储类对象的数据变了，首地址是一个虚函数的地址，而不是类的数据成员的地址了。为了证实这个想法，我又在 `main` 中加入了如下代码：

```cpp
//显示“myMinBox”的大小
cout << "myMinBox's size is " << sizeof(myMinBox) << endl;
```

输出结果如下：

```text
Volume is 6
volume is 12
myMinBox's size is 16
```

果然按照以前的知识 `myMinBox` 的大小应该是 12，而不是 16。既然这样的话，需要查看在类对象声明的时候到底发生了什麽。因此，通过 `main` 函数的反汇编代码，找到 `CBox` 的构造函数地址：`00411023`，跟进去后，代码如下：

；========================================================

```asm
00411600 push    ebp
00411601 mov     ebp, esp
00411603 sub     esp, 0CC
00411609 push    ebx
0041160A push    esi
0041160B push    edi
0041160C push    ecx
0041160D lea     edi, dword ptr [ebp-CC]
00411613 mov     ecx, 33
00411618 mov     eax, CCCCCCCCC
0041161D rep     stos dword ptr es:[edi]
0041161F pop     ecx
00411620 mov     dword ptr [ebp-8], ecx   ；将 myBox 的地址保存到 ptr [ebp-8]
00411623 mov     eax, dword ptr [ebp-8]
00411626 mov     dword ptr [eax], offset CBox::`vftable'              
                                 ；将“offset CBox::`vftable"给 myBox 对象，也就是我们调用的 volume()的  实际的地址
0041162C mov     eax, dword ptr [ebp-8]       
0041162F mov     ecx, dword ptr [ebp+8]
00411632 mov     dword ptr [eax+4], ecx ；初始化默认参数 1
00411635 mov     eax, dword ptr [ebp-8]
00411638 mov     ecx, dword ptr [ebp+C]
0041163B mov     dword ptr [eax+8], ecx   ; 初始化默认参数 2
0041163E mov     eax, dword ptr [ebp-8]
00411641 mov     ecx, dword ptr [ebp+10]
00411644 mov     dword ptr [eax+C], ecx   ；初始化默认参数 3
00411647 mov     eax, dword ptr [ebp-8]
0041164A pop     edi
0041164B pop     esi
0041164C pop     ebx
0041164D mov     esp, ebp
0041164F pop     ebp
00411650 retn    0C
```

；========================================================

我看到了"`vftable`"，想起很久之前看大牛们的文章谈到的虚拟函数表啥的，在数据窗口中查看 `offset CBox::`vftable` 的内容，发现是一个函数的地址：经分析是基类 `CBox` 的函数 `volume` 的地址，这段汇编代码就不贴了。

理可以查看 `CCandyBox` 默认构造函数的实现，也是类似。

## 四、总结

通过以上的分析，难道是在声明一个类的对象的时候，会将这个类的虚函数的地址都保存到一个表中，然后将这个虚函数表的起始地址赋给这个类的对象。当编译器发现一个函数的调用时虚函数时，知道这个函数在这个表中的偏移位置，但是它不知道这个表的起始地址，而这个起始地址就是保存在类对象声明时，在默认构造函数中初始化的，因此当调用一个类对象的虚函数时，只要从类对象中取出虚函数表的起始地址，然后再加上相应的偏移就可以从相应类的虚函数表中取出实际的虚函数地址，从而实现虚函数的调用。

从而我得出的初步结论如下：

1. 一个类的虚函数的地址是根据这个类的虚拟函数表起始地址再加上这个函数的在虚拟函数表中的偏移得到的。而虚拟函数表中的偏移是在编译的时候就确定的，而虚拟函数的起始地址则是在相应对象初始化时，在默认构造函数中得到的。
2. 每个类只有一份虚拟函数表，这个表的起始地址是全局的，所有这个类的对象都会使用这个起始地址。
3. 应该所有类，包括基类和子类，的虚函数表中函数个数是相同的，顺序也是一样的，比如基类的 `volume` 是在这个表的第一个位置，那麽派生类的 `volume` 也是这个表的第一个位置。虚拟函数表应该是以 0 为结束的。

上面的结论只是我初步的想法和猜测，还需要去实践验证。由此我也大概了解了 C++的虚函数的实现机制，我知道这里面肯定会有我想错的地方，并且是很不全面的，由于我的 C++只到这个水平，只能下面继续再学习了，后续我想再多写一些虚函数，因为目前只是存在一个虚函数。另外还需要看看多重继承是怎样实现的。而且后面还有好多 C++的东西没有完全看懂。这需要我继续努力。

写的太乱，再加上我的语言表达能力不好，请谅解，我写这样的东西原因是我作为一个初学者有对于技术上有很多困惑，通过把我学习的一些想法和学习的过程写出来，一方面能够知道自己思考的是否正确，请大牛们指点，另一方面提供给像我这样的初学者一些鼓励。

已经快到凌晨 4 点了，我得睡觉了，明天还要上班！
