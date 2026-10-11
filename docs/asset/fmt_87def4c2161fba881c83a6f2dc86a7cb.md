
```
00391F2D    897E 04         mov dword ptr ds:[esi+4],edi

00391F30    8973 0C         mov dword ptr ds:[ebx+C],esi

00391F33    68 7CC52A05     push 52AC57C

00391F38    56              push esi

00391F39    E8 8C9BAE79     call mscorwks.79E7BACA

00391F3E    897B 0C         mov dword ptr ds:[ebx+C],edi

00391F41    8B4E 08         mov ecx,dword ptr ds:[esi+8]

00391F44    8946 08         mov dword ptr ds:[esi+8],eax

00391F47    8BC1            mov eax,ecx

00391F49    83C4 04         add esp,4

00391F4C    5A              pop edx

00391F4D    59              pop ecx

00391F4E    89EC            mov esp,ebp

00391F50    5D              pop ebp

00391F51    83C4 04         add esp,4

00391F54    5F              pop edi

00391F55    5E              pop esi

00391F56    5B              pop ebx

00391F57    5D              pop ebp

00391F58    83C4 08         add esp,8

00391F5B    C3              retn
```

好，下面我们让代码对 `calcClass` 进行初始化，再看 `refrence` 处的内容（执行到最后的 `retn` 处停住）：

```
00A73080  00 00 04 00 0C 00 00 00 02 04 06 00 04 00 00 00  .. .....   . ...

00A73090  18 9C 0F 79 14 2C A7 00 C0 30 A7 00 58 13 A7 00   ?y ,???X ?

00A730A0  00 00 00 00 00 00 00 00 EC 4B 35 79 C0 39 35 79  ........霮 5y?5y

00A730B0  B0 39 35 79 C0 A4 34 79 D0 00 DA 00 D4 30 A7 00  ?5y 坤 4y????

00A730C0  80 00 00 00 00 00 00 00 B8 70 30 A7 00 89 ED E9  €.......竝 0?夗

00A730D0  38 EE 91 FF B8 78 30 A7 00 89 ED E9 2C EE 91 FF  8 顟�竫 0?夗?顟�

00A730E0  00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  ................
```

![](img/b5c761f5687075da03297b37f849b489.jpg)

注意，`00A730B8` 处的内容已经改变，这正是 `calcCode` 的本地代码地址。

把整个过程小结一下。在第一次调用还没有进行 JIT 的方法时，`VTable` 的地址总是指向一个 `prework`，它会调用 `mscorwks`，继而是 JIT 引擎，对该方法进行编译。编译完后的地址代码地址被直接写入方法表中，这样下次再调用该方法就是直接跳转到该地址，而不用再次进行编译。`CLR` 中的方法调用大多数是 `__fastcall` 形式，利用寄存器来传递参数。而内联，在 .NET 中由 JIT 引擎在动态编译时自行决定的。

本文利用 `OllyDbg` 对 .NET 下方法的调用和 JIT 进行一些跟踪，其中提到的一些中断的位置和技巧对 Crack Fans 们是很有用的，而了解 .NET 内部的一些运行机制，也会对我们的逆向过程有所帮助。

`4dd466d0e1e0cc123afbe504da2462ca`
