
``` 
01010139 8b4df8          mov     ecx,dword ptr [ebp-8]   // 这里是BaseClass实例对象的地址 放到 ecx寄存器,Jit采用类似fastcall的调用协定,前2个不大于4字节的参数用 ecx edx来传递,而实例方法的调用第一个参数是隐含的this指针(托管对象在托管堆上的地址),如果是静态方法就不需要传this pointer了

0101013c 8b01            mov     eax,dword ptr [ecx]    // 托管堆上的对象(值类型装箱后也是一样)第一个4字节(64位8字节)是对象的方法表地址(MethodTable),这里是把方法表(MethodTable)地址赋给eax寄存器

0101013e ff5038          call    dword ptr [eax+38h]    // 这里就是实际的方法调用 上面说了 第一个虚方法在方法表的偏移28h位置,前4个是Object里的4个虚方法,所以 VirtualFun1 的入口在方法表地址(MT) + 28h + 4×4字节 也就是偏移38h的位置

01010141 90              nop

01010142 8b4df4          mov     ecx,dword ptr [ebp-0Ch]    // 这里是 ChlidClass的对象地址赋给ecx

01010145 8b01            mov     eax,dword ptr [ecx]    // 同样ChlidClass的方法表地址赋给eax

01010147 ff5038          call    dword ptr [eax+38h]    // 调用ChlidClass方法表偏移38h的方法,也是VirtualFun1 方法

0101014a 90              nop

0101014b 8b4df0          mov     ecx,dword ptr [ebp-10h]    // BrotherClass的对象地址赋给ecx

0101014e 8b01            mov     eax,dword ptr [ecx]    // BrotherClass方法表地址赋给eax

01010150 ff5038          call    dword ptr [eax+38h]    // 调用BrotherClass方法表偏移38h的方法,也是VirtualFun1 方法

01010153 90              nop

01010154 8b4dec          mov     ecx,dword ptr [ebp-14h]    // DerivedOfBrotherClass的对象地址赋给ecx

01010157 8b01            mov     eax,dword ptr [ecx]    // DerivedOfBrotherClass方法表地址赋给eax

01010159 ff5038          call    dword ptr [eax+38h]     // 调用DerivedOfBrotherClass方法表偏移38h的方法,也是VirtualFun1 方法

0101015c 90              nop

0101015d 8b4dec          mov     ecx,dword ptr [ebp-14h]    // 还是DerivedOfBrotherClass对象地址

01010160 8b01            mov     eax,dword ptr [ecx]    // DerivedOfBrotherClass的方法表赋给eax

01010162 ff503c          call    dword ptr [eax+3Ch]    // 这次偏移不一样了,第6个方法 VirtualFun2 (28h+5×4字节)

01010165 90              nop

01010166 8b4de8          mov     ecx,dword ptr [ebp-18h]    // 还是DerivedOfBrotherClass对象地址

01010169 8b01            mov     eax,dword ptr [ecx]    // DerivedOfBrotherClass的方法表赋给eax

0101016b ff5040          call    dword ptr [eax+40h]    // 这次偏移又不一样了,第7个方法 VirtualFun3 (28h+6×4字节)

```

可以看到 继承多态在CLR运行时的实现是通过方法表的偏移 间接调用的,而方法表内继承虚方法的构建顺序是先父类再子类,由于.NET是单一继承,这样就确保了在同一家族的同一虚方法的偏移量是一样的.

### Framework 4.0 对比

接下来用Framework 4.0 编译下源码,4.0 和2.0相比 在实现上多了一层间接寻址,但思路是一样的

```
%windir%\Microsoft.NET\Framework\v4.0.30319\csc.exe /debug /target:exe /out:e:\temp\Polymorphism01_4.0.exe e:\temp\Polymorphism01.cs

pause

```

- 运行 `Polymorphism01_4.0.exe`
- 启动windbg 附加进程 加载SOS (这里要加载对于4.0的sos.dll)
- 直接查找Main方法:

`!Name2EE Polymorphism01_4.0.exe Program.Main`

```
0:004> !Name2EE Polymorphism01_4.0.exe Program.Main

Module:      00b32ea4

Assembly:    Polymorphism01_4.0.exe

Token:       06000001

MethodDesc:  00b33838

Name:        Program.Main(System.String[])

JITTED Code Address: 033a0070

```

- 看Main方法的区别:`!u 033a0070`

这里只截取最重要的一段,调用构造器和其他的部分都先忽略

```
...

e:\temp\Polymorphism01.cs @ 16:

033a0139 8b4df8          mov     ecx,dword ptr [ebp-8]  // 这个还是一样BaseClass对象的地址赋给ecx

033a013c 8b01            mov     eax,dword ptr [ecx]    // 还是对象的第一个4字节是方法表地址 赋给eax

033a013e 8b4028          mov     eax,dword ptr [eax+28h]    // 这里是和2.0的区别 所有继承的虚方法的起始地址保存在方法表偏移28h的位置,也就是偏移量不是从方法表地址开始算了

033a0141 ff5010          call    dword ptr [eax+10h]    // 这里的方式一样的 eax是虚方法的起始位置了,前4个是Object的4个虚方法,偏移10h是第5个方法 VirtualFun1

033a0144 90              nop



e:\temp\Polymorphism01.cs @ 17:

033a0145 8b4df4          mov     ecx,dword ptr [ebp-0Ch]    // ChlidClass对象地址赋给ecx

033a0148 8b01            mov     eax,dword ptr [ecx]    // ChlidClass方法表地址赋给eax

033a014a 8b4028          mov     eax,dword ptr [eax+28h]    // 虚表入口地址赋给eax

033a014d ff5010          call    dword ptr [eax+10h]    //还是偏移到第5个方法 VirtualFun1

033a0150 90              nop



e:\temp\Polymorphism01.cs @ 18:

033a0151 8b4df0          mov     ecx,dword ptr [ebp-10h]    // BrotherClass对象地址赋给ecx

033a0154 8b01            mov     eax,dword ptr [ecx]     // BrotherClass方法表地址赋给eax

033a0156 8b4028          mov     eax,dword ptr [eax+28h]    // 虚表入口地址赋给eax

033a0159 ff5010          call    dword ptr [eax+10h]    //还是偏移到第5个方法 VirtualFun1

033a015c 90              nop



e:\temp\Polymorphism01.cs @ 19:

033a015d 8b4dec          mov     ecx,dword ptr [ebp-14h]    // DerivedOfBrotherClass对象地址赋给ecx

033a0160 8b01            mov     eax,dword ptr [ecx]    // DerivedOfBrotherClass方法表地址赋给eax

033a0162 8b4028          mov     eax,dword ptr [eax+28h]    // 虚表入口地址赋给eax

033a0165 ff5010          call    dword ptr [eax+10h]    //还是偏移到第5个方法 VirtualFun1

033a0168 90              nop



e:\temp\Polymorphism01.cs @ 20:

033a0169 8b4dec          mov     ecx,dword ptr [ebp-14h]    // 上面同一个对象

033a016c 8b01            mov     eax,dword ptr [ecx]

033a016e 8b4028          mov     eax,dword ptr [eax+28h]

033a0171 ff5014          call    dword ptr [eax+14h]    // 这里比上面的调用多偏移了4个字节 也就是第6个方法 VirtualFun2

033a0174 90              nop



e:\temp\Polymorphism01.cs @ 21:

033a0175 8b4de8          mov     ecx,dword ptr [ebp-18h]    // 和上面不是同一个对象地址,但是是实例化同样类型的对象

033a0178 8b01            mov     eax,dword ptr [ecx]

033a017a 8b4028          mov     eax,dword ptr [eax+28h]

033a017d ff5018          call    dword ptr [eax+18h]     // 这里比上面的调用再多偏移了4个字节 也就是第7个方法 VirtualFun3

033a0180 90              nop

...

```

.NET 4.0 比2.0 多了一次间接寻址,就是先偏移到虚表的入口,再从这个入口开始偏移到相应的方法,这样的好处(个人觉得)虚表的存储位置可以更灵活 如果方法表(MT)包含多个可变长结构也没问题 只要入口地址保存在偏移28h的位置即可

个人页地址：2c6K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6W2M7%4m8A6k6r3g2J5i4K6u0W2k6$3W2@1K9s2g2T1i4K6u0W2K9h3!0Q4x3V1k6o6e0q4u0Q4x3V1k6A6L8X3S2W2M7X3W2@1j5h3&6U0k6g2)9J5k6s2m8G2L8s2W2E0L8%4u0H3K9r3W2K6L8g2)9J5c8R3`.

欢迎讨论～

> ## 参考文档：

> 4ecK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2E0K9h3y4J5L8%4y4G2k6Y4c8Q4x3X3g2U0L8$3#2Q4x3V1k6U0K9r3W2F1j5g2)9J5c8V1#2e0c8p5&6Q4x3V1k6D9K9h3u0J5j5i4u0&6i4K6u0r3L8X3g2@1c8Y4u0S2L8h3g2%4L8%4u0C8i4K6u0r3L8X3g2@1k6Y4u0S2L8h3g2%4L8%4u0C8i4K6u0r3d9V1W2f1b7$3!0E0M7r3W2D9k6i4u0Q4x3X3g2E0M7%4m8^5i4K6y4r3L8h3k6J5i4K6y4p5N6s2u0#2k6b7`.

> b66K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3y4G2k6r3g2H3M7X3!0B7k6h3y4@1i4K6u0W2j5$3!0E0i4K6u0r3b7i4u0@1K9h3y4D9k6i4y4Q4x3V1j5J5x3o6b7^5x3g2)9J5c8V1&6q4g2q4)9J5k6q4c8&6M7r3g2Q4x3X3c8u0L8Y4c8W2M7X3&6S2L8s2y4Q4x3X3c8r3M7X3!0E0i4K6u0V1j5g2)9J5k6p5#2A6j5%4u0G2M7$3!0X3N6q4)9J5k6p5y4x3f1W2)9J5k6q4m8W2M7Y4y4H3k6h3y4@1K9b7`.

> cf0K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0D9L8$3N6K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0Q4x3X3g2A6L8q4)9J5c8Y4y4S2M7$3S2S2i4K6u0r3x3U0l9I4x3W2)9J5c8U0l9K6i4K6u0r3x3e0g2Q4x3V1k6$3K9i4u0@1N6h3q4D9i4K6u0V1L8h3g2@1K9r3!0V1i4K6u0V1k6r3W2K6M7r3q4@1j5$3S2Q4x3X3c8S2L8X3c8Q4x3X3c8G2j5X3A6W2j5%4c8Q4x3X3c8D9j5i4W2G2N6i4c8Q4x3X3c8U0K9r3q4F1k6$3g2K6i4K6u0V1K9h3&6Q4x3X3c8U0L8s2u0Q4x3X3b7@1x3q4)9J5c8R3`.

> 3c2K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3y4F1j5X3I4G2k6%4y4Q4x3X3g2U0L8$3#2Q4x3V1k6n7L8s2g2W2g2s2A6S2M7W2)9J5c8X3q4J5N6r3W2U0L8r3g2K6i4K6u0r3z5o6R3@1y4U0V1@1i4K6u0W2K9s2c8E0L8l9`.

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于  2019-2-1 20:12

被admin编辑

，原因：

[#.NET平台](https://bbs.kanxue.com/forum-4-1-7.htm)
