
# EncryptPE1.0 V1.2003.5.18

> 原文：[`www.pediy.com/kssd/pediy06/pediy6726.htm`](https://www.pediy.com/kssd/pediy06/pediy6726.htm)

**【破文标题】** 轻松+愉快 esp 定律+内存断点快速脱老王的 EncryptPE1.0 V1.2003.5.18 加壳的记事本

**【破文作者】** springkang[DFCG]

**【作者主页】** [www.chinadfcg.com](http://www.chinadfcg.com)

**【作者邮箱】** springkang2003@yahoo.com.cn

**【所属组织】** DFCG

**【软件名称】** EncryptPE1.0 加壳的一个记事本

**【下载地址】** 本地下地

**【破解工具】** `OD`, `impr`, `loadpe`

**【保护方式】**

**【软件下载】** [`bbs.pediy.com/showthread.php?s=&threadid=3506`](http://bbs.pediy.com/showthread.php?s=&threadid=3506)

**【破解难度】** 中等

---

### 软件介绍

老王老师的猛壳，曾经难倒众多高手!可是壳已经老了，因为老王的新版壳出来了!

---

### 破解声名

几个月前学习 `fly` 的脱文，可是水平很菜，脱不了。前几天有个网友要我帮忙搞一个软件，加的就是这个壳。没办法，只有硬着头皮试了。重新看了 `fly` 大侠的脱文，就开始上路!没想到这个猛壳不防 `esp` 定律，大菜鸟我又捡了个漏:D

---

### 【破解分析】

用 `fly` 的修改版载入(没有修改版也没关系，添加“同时忽略 `0EEDFADE`、`C0000008`、`009B25C`、`00953D74`”异常，就行了。否则会很难看的),忽略所有异常，再用 `IsDebugPresent` 插件隐藏一下!

```
0040D000 >  60               pushad   //停在这里，F8 单步运行二步。
0040D001    9C               pushfd
0040D002    64:FF35 0000000> push dword ptr fs:[0]  //到这里 esp=0012ffa0 ，在命令行下：hr 0012ffa0 F9 运行。
0040D009    E8 79010000      call NOTEPAD.0040D187
0040D00E    0000             add byte ptr ds:[eax],al
0040D010    0000             add byte ptr ds:[eax],al
```

几个入口点警告之后，出现注册对话框，点确定，中断在硬件断点处

```
711A37AE    25 00010000      and eax,100    //中断在这里，取消硬件断点，用 ALT+M 打开内存模块，下内存访问断点
711A37B3    3D 00010000      cmp eax,100
711A37B8    74 05            je short V1200351.711A37BF
711A37BA    E8 C996FFFF      call V1200351.7119CE88
711A37BF    6A 10            push 10
711A37C1    A1 8CC71B71      mov eax,dword ptr ds:[711BC78C]
711A37C6    8B00             mov eax,dword ptr ds:[eax]
711A37C8    E8 3713F8FF      call V1200351.71124B04
```

内存镜像, 项目 23

```
地址=00401000
大小=0000C000 (49152.)
Owner=NOTEPAD 00400000
Section=EPE0    //在这里下内存访问断点，因为加了密，所以就不是以前的 code 了，其实还是一样的。
类型=Imag 01001002
访问=R
初始访问=RWE
```

`F9` 运行，中断在入口处了

```
004010CC    55               push ebp        //中断在这里，是不是很熟悉？ :) 用 loadpe dump 后，用 impr 修复，
004010CD    8BEC             mov ebp,esp
004010CF    83EC 44          sub esp,44
004010D2    56               push esi
004010D3    FF15 E4634000    call dword ptr ds:[4063E4]
004010D9    8BF0             mov esi,eax
```

因为加了密，所以修复不了。看看 `fly` 写的文章，现在我们只要能找开加密点，进行修改就行了，至于为什么确定那里是加密点，此处暂不讨论!

先看看第一个无效的指针地址：`rva`：`000062e4`

按 `CTRL+F2` 重新开始，在命令行 `dd 004062e4`,下内存访问断点。`F9` 运行，

```
7112295D    F3:A5            rep movs dword ptr es:[edi],dword ptr ds:[esi]    //中断在这里，再按两次 F9 后（如果再按一次 F9 会怎样呢？自己试试吧），此时 CTRL+G 来到 711A33A1(这里要参考 fly 大侠的文章，或者自己去跟，体会一下，这里我没时间多说了)
7112295F    89C1             mov ecx,eax
71122961    83E1 03          and ecx,3
71122964    83C6 03          add esi,3
71122967    83C7 03          add edi,3
7112296A    F3:A4            rep movs byte ptr es:[edi],byte ptr ds:[esi]
711A33A1    8B55 F8          mov edx,dword ptr ss:[ebp-8]   //到这里，F2 下断，F9 运行。中断在此。我们在命令行 DD ebp-4,看到了什么？是 RegQueryValueExA,（在此内存区域上下看看，观察一下。）好的，修改指令。
711A33A4    8910             mov dword ptr ds:[eax],edx  //如果刚才再按一次 F9，就会发生内存中断，来到这里，呵呵，加密的指针已经处理完了，我们停在这里有什么用呢？所以就要在上一行下断，那里正好在处理加密的批针。
711A33A6    33C0             xor eax,eax
711A33A8    5A               pop edx
711A33A9    59               pop ecx
```

`711A33A1 8B55 F8 mov edx,dword ptr ss:[ebp-8]` //加密指针 ★

改为：

`711A33A1 8B55 FC mov edx,dword ptr ss:[ebp-4]` //正确函数

取消内存断点和 `F2` 断点，`F9` 运行（如果不取消，`F9`，我们会看到 `711A33A1` 处正在处理正确函数 ，呵呵！），虽然程序异常无法再运行，但是所有的函数均自动还原了。

运行 `ImportREC`，选择这个进程。把 `OEP` 改为 `000010CC`，点“`Get Import`”，

只修复了一些，还有很多没修复。没关系，再来一次。

现在看看还没修复的无效的指针地址，先看第一个：`rva`：`000062fc`

方法同上，只是有点不同。我这里是下 `hr 004062fc` ，（下内存访问断点不太行，会直接中断在 `711A33A4 8910 mov dword ptr ds:[eax],edx` 已经处理完了加密指针，中断在这里有什么用？），中断在

`7112295D F3:A5 rep movs dword ptr es:[edi],dword ptr ds:[esi]` 然后 `CTRL+G` 来到 `711A33A1`。后面的就一样做了。

经过两次修复后，一切 `OK` 了。

脱壳后文件有点大，按 `fly` 的方法重建也还是一样。没办法了，不管它，能运行就行 ：D

---

### 【总结】

本文参考了 `fly` 大侠的脱文，该文很有参考价值，省了偶不少时间，向 `fly` 大侠致敬了！另外还要向老王老师致敬，不好意思拿他的壳开了刀！

用这个方法脱了用 `EncryptPE` 加的一个程序同样有效！

---

### 【版权信息】

版权公有，人权私有！谢谢转载，注明作者！

`4dd466d0e1e0cc123afbe504da2462ca`
