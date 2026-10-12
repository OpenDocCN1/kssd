
# 闲着无聊，手脱 TTP 壳全过程

> 原文：[`www.pediy.com/kssd/pediy12/136452.html`](https://www.pediy.com/kssd/pediy12/136452.html)

### 文章信息

*   【文章标题】: 闲着无聊，手脱 TTP 壳全过程(菜鸟第一次写的脱文,有错之处请大家别见怪)
*   【文章作者】: `nisycc`
*   【软件名称】: 加了 TTP 壳的记事本
*   【下载地址】: 自己搜索下载
*   【软件介绍】: `OD` `LORDPE` `IMPORT` `REC`
*   【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

---

### 详细过程

闲着无聊，就给一个记事本加了下 TTP 的壳，来脱下。

先介绍下什么是 TTProtect：

TTProtect 是专为软件开发者设计的保护软件不被非法修改或反编译的专业加密软件。

TTProtect 是新一代软件保护工具，吸收众多保护工具的优点，并加入了独有的特色功能。

TTProtect 使用了特别的反跟踪方法，使得对保护对象的非法调试非常困难，并且对保护代码进行全局优化、乱序和混淆，使得非法分析相当困难。

TTP 这款壳是反硬件执行断点的，所以要用到一个插件，`PhantOm` 这款插件，和 `SOD` 一起用，效果还不错哈，直接进入主题了，先跑 `OEP` 吧！我是给 XP 的记事本加的 TTP，所以是相当于给 C++的软件加了一个壳吧。

我们运行程序，暂停，点击按钮“K”，在

*(图片丢失)*

调用来自 `006DD40F` 这里，右键====》显示调用，此时停在 `WinMain` 函数上面，如果一个程序很了解的话，就知道在软件的退出的的上面就是 `WinMain` 函数，`WinMain` 函数上面就是 `GetMoudleHandleA` 函数了。

```
006DD405   6A 0A         push    0A
006DD407   58           pop     eax
006DD408   50           push    eax
006DD409   56           push    esi
006DD40A   53           push    ebx
006DD40B   53           push    ebx
006DD40C   FFD7         call    edi                           ; GetMoudleHandleA
006DD40E   50           push    eax
006DD40F   E8 22559200  call    ttp.01002936                  ; WinMain
```

此时向上面翻，会找到 `OEP`

```
006DD29F   016A 70      add     dword ptr [edx+70], ebp
006DD2A2   68 98180001  push    1001898
006DD2A7   E8 BCA29200  call    ttp.01007568
006DD2AC   33DB         xor     ebx, ebx
006DD2AE   53           push    ebx
006DD2AF   8B3D CC100001 mov    edi, dword ptr [10010CC]
006DD2B5   FFD7         call    edi
006DD2B7   66:8138 4D5A cmp     word ptr [eax], 5A4D
```

`006DD29F` 这里就是 `OEP` 了，可是我们知道，记事本的 `OEP` 的头一句是 `PUSH 70`，我们可以发现，壳把第一句变形了，我们来给他 `NOP` 一字节，就会发现，程序的 `OEP` 正常了！

```
006DD2A0   6A 70        push    70
006DD2A2   68 98180001  push    1001898
006DD2A7   E8 BCA29200  call    ttp.01007568
006DD2AC   33DB         xor     ebx, ebx
006DD2AE   53           push    ebx
006DD2AF   8B3D CC100001 mov    edi, dword ptr [10010CC]
006DD2B5   FFD7         call    edi
006DD2B7   66:8138 4D5A cmp     word ptr [eax], 5A4D
```

如果我们细心的看下，就会发现，记事本的 `OEP` 的地址应该是在 `0100739D`，而加了 TTP 壳的记事本的 `OEP` 地址是 `006DD2A0`，我们点击“M”，发现 `006DD2A0` 在壳段，就像是把代码抽到壳段执行了！

我们 `CTRL+G` 转到 `0100739D` 来看下，会发现一堆空代码，所以我们 `DUMP` 程序之前，需要将壳代码中的 `OEP` 复制到 `0100739D` 这里，否则程序是不会运行的。我们复制下整个 `OEP` 的二进制代码！

```
6A 70 68 98 18 00 01 E8 BC A2 92 00 33 DB 53 8B 3D CC 10 00 01 FF D7 66 81 38 4D 5A 75 1F 8B 48
3C 03 C8 81 39 50 45 00 00 75 12 0F B7 41 18 3D 0B 01 00 00 74 1F 3D 0B 02 00 00 74 05 89 5D E4
EB 27 83 B9 84 00 00 00 0E 76 F2 33 C0 39 99 F8 00 00 00 EB 0E 83 79 74 0E 76 E2 33 C0 39 99 E8
00 00 00 0F 95 C0 89 45 E4 89 5D FC 6A 02 FF 15 38 13 00 01 59 83 0D 9C AB 00 01 FF 83 0D A0 AB
00 01 FF FF 15 34 13 00 01 8B 0D B8 9A 00 01 89 08 FF 15 30 13 00 01 8B 0D B4 9A 00 01 89 08 A1
2C 13 00 01 8B 00 A3 A4 AB 00 01 E8 A4 A2 92 00 39 1D 08 96 00 01 75 0C 68 F4 75 00 01 FF 15 28
13 00 01 59 E8 74 A2 92 00 68 10 90 00 01 68 0C 90 00 01 E8 5A A2 92 00 A1 B0 9A 00 01 89 45 DC
8D 45 DC 50 FF 35 AC 9A 00 01 8D 45 D4 50 8D 45 D0 50 8D 45 CC 50 FF 15 20 13 00 01 89 45 C8 68
08 90 00 01 68 00 90 00 01 E8 24 A2 92 00 83 C4 24 A1 1C 13 00 01 8B 30 89 75 E0 80 3E 22 75 3A
46 89 75 E0 8A 06 3A C3 74 04 3C 22 75 F2 80 3E 22 75 04 46 89 75 E0 8A 06 3A C3 74 04 3C 20 76
F2 89 5D AC 8D 45 80 50 FF 15 D0 10 00 01 F6 45 AC 01 74 11 0F B7 45 B0 EB 0E 80 3E 20 76 D8 46
89 75 E0 EB F5 6A 0A 58 50 56 53 53 FF D7 50 E8 22 55 92 00 8B F0 89 75 C4 39 5D E4 75 07 56 FF
15 18 13 00 01 FF 15 00 13 00 01 EB 2D 8B 45 EC 8B 08 8B 09 89 4D D8 50 51 E8 88 A1 92 00 59 59
C3 8B 65 E8 8B 75 D8 83 7D E4 00 75 07 56 FF 15 F0 12 00 01 FF 15 F4 12 00 01 83 4D FC FF 8B C6
E8 3E A1 92 00 C3
```

复制下来后，我们来找 `IAT`，我们在 `006DD2AF 8B3D CC100001 mov edi, dword ptr [10010CC]`，右键====》数据窗口中跟随=====》内存地址，发现大部分的 `IAT` 都被加密了，只有少数部分没有被加密，所以我们来找下 `Magic JUMP`，来跳过 `IAT` 加密！

我们重新载入，在 `VirtualProtect` 函数下的 `RETN` 处下断点，并把数据窗口转到 `10010CC` 处，然后 `F9` 运行，观察数据窗口，大概 6 次左右吧，会发现数据窗口中的 `IAT` 都出现了，再按一次会发现 `IAT` 都被加密了，这就是我们的返回时机，`ALT+F9` 返回，然后 `F8` 单步走，发现走过 `10007A7F` 后，`IAT` 被加密了！

```
10007A66   33F6         xor     esi, esi
10007A68   8D4C24 14    lea     ecx, dword ptr [esp+14]
10007A6C   51           push    ecx
10007A6D   8D8C24 8C010000 lea  ecx, dword ptr [esp+18C]
10007A74   E8 D49BFFFF  call    1000164D
10007A79   8B15 D10A0210 mov    edx, dword ptr [10020AD1]        ; ttp.01000000
10007A7F   52           push    edx
10007A80   E8 C8DFFFFF  call    10005A4D          //IAT 被加密了
10007A85   6A 01        push    1
10007A87   FF15 E10A0210 call    dword ptr [10020AE1]           ; ttp.01013000
```

我们再次重新载入，再次来到 `10007A80` 处，在数据窗口 `0100108C` 处下硬件写入断点====》byte 断点！然后 `F9` 运行！

```
01001088  00000000
0100108C  7C8097B8  kernel32.7C8097B8
01001090  7C80932E  ASCII "LL.VerSetConditionMask"
01001094  7C80A4B7  kernel32.7C80A4B7
01001098  7C80A864  kernel32.7C80A864
0100109C  7C809FA0  kernel32.7C809FA0
010010A0  7C83378D  kernel32.7C83378D
010010A4  7C833FEB  kernel32.7C833FEB
010010A8  7C80FFA9  kernel32.7C80FFA9
010010AC  7C80FF12  kernel32.7C80FF12
010010B0  7C810CFD  kernel32.7C810CFD
010010B4  7C809420  kernel32.7C809420
010010B8  7C8017E9  kernel32.GetSystemTimeAsFileTime
010010BC  7C801E1A  kernel32.TerminateProcess
```

断在了这里

```
10006C36   897C24 18    mov     dword ptr [esp+18], edi
10006C3A   8B4424 18    mov     eax, dword ptr [esp+18]
10006C3E   8945 00      mov     dword ptr [ebp], eax      ；开始加密了！
10006C41   8B4424 24    mov     eax, dword ptr [esp+24]         ; 断在这里
10006C45   8B78 04      mov     edi, dword ptr [eax+4]
```

我们向上面翻，找能跳过这个加密的跳转：

```
10006AAF   /74 19       je      short 10006ACA
10006AB1   |3D 4DE75D66 cmp     eax, 665DE74D
10006AB6   |74 12       je      short 10006ACA
10006AB8   |3D 5FF05127 cmp     eax, 2751F05F
10006ABD   |74 0B       je      short 10006ACA
10006ABF   |3D 59F04E75 cmp     eax, 754EF059
10006AC4   |0F85 70010000 jnz   10006C3A                ；Magic Jump
10006ACA   \68 F4010000  push    1F4
10006ACF    E8 F93D0000 call    1000A8CD
```

发现 `10006AC4` 这个跳转可以跳过我们的 `IAT` 加密，所以 `10006AC4` 是我们要找的 `MAGIC JUMP`，找到后，我们再次重新载入，删除硬件写入断点，再次来到 `10007A80`，然后我们 `F7` 单步进入，寻找到我们的 `Magic Jump`，将三个 `JE` 给 `NOP` 掉，将 `JNZ` 给改成 `JMP`，此时在 `VirtualProtect` 函数下的 `RETN` 处下断点不要取消，再次 `F9`，我们发现所有的 `IAT` 都恢复了，我们 `ALT+F9` 返回后，我们直接搜索 `JMP EAX`，发现提示错误

怎么解决呢？我用的是最简单的方法，点击 `M`，来到写有 `SFX` 的区段，我们右键设置访问====》全部访问，就可以了，我们再次搜索，发现我们找到了！我们在 `JMP EAX` 处下 `F2` 断点，然后 `F9` 断下后，`F7` 跟进，此时到达了我们的 `OEP`，这些代码都是没有加密的，我们现在的任务就是来到 `0100739D` 处，将我们复制的二进制代码拷贝过去，拷贝好以后，我们在 `0100739D` 这里新建 `EIP`，然后打开 `LORDPE` 来 `DUMP` 程序，这个壳有 `ANTI DUMP`，我们把 `VirtualProtect` 的页面属性给改成可读可写和可执行就可以了，呵呵，我在这里就不说了！`DUMP` 好以后，`IMPORT REC` 进行修复，修复成功后，运行，可以正常运行，此时我们的脱壳就完成了。

---

### 经验总结

闲着无聊来脱的一个壳，侥幸被我搞定了，呵呵！

---

### 版权声明

本文原创于 `nisycc`, 转载请注明作者并保持文章的完整, 谢谢!
