
# UltraEdit 中文版背后的故事

> 原文：[`www.pediy.com/kssd/pediy11/111688.html`](https://www.pediy.com/kssd/pediy11/111688.html)

自己先搞了一下 然后又看了一下网上的爆破方法

先说一下网络上流传的爆破方法： `OD` 载入后 `Ctrl+S` 搜索 `PUSH 6E6D` 这个是弹出过期的资源对话框的 ID 值

我们通过启动时判断是否注册来定位程序的验证点。用工具 `eXeScope` 查看 `ueres.dll`

![](img/1ec77e531d720023c014e4a5ad85daec.jpg)

这个方法很快 我用的是另一种方法 点程序关于的时候 根据对话框文字内容不同 在内存下硬件读取断点来定位

最终效果都是相同的 定位到算法 `CALL` 的关键跳：

### Cr（1032 版）

```
00806282     /EB 02          JMP SHORT Uedit32.00806286                 ; 修改为 JMP
00806284   |. |8AC3          MOV AL,BL
```

如果只是这样的话 是不是很爽的 我查看了一下老外的版本 发现也是这样搞的

```
0046333B    E8 A078FFFF      CALL Uedit32.0045ABE0
00463340    8B2D 201CB500    MOV EBP,DWORD PTR DS:[<&USER32.SendMessa>; user32.SendMessageA
00463346    83C4 14          ADD ESP,14
00463349    84C0             TEST AL,AL
0046334B    0F84 1D030000    JE Uedit32.0046366E                        ; 16.0.0.1038 就是把这里 NOP 掉就 OK 了
00463351    391D E478D900    CMP DWORD PTR DS:[D978E4],EBX
00463357    0F85 11030000    JNZ Uedit32.0046366E
0046335D    395C24 14        CMP DWORD PTR SS:[ESP+14],EBX
00463361    74 28            JE SHORT Uedit32.0046338B
00463363    8B4C24 24        MOV ECX,DWORD PTR SS:[ESP+24]
00463367    8B5424 28        MOV EDX,DWORD PTR SS:[ESP+28]
0046336B    83EC 10          SUB ESP,10
0046336E    8BC4             MOV EAX,ESP
00463370    8908             MOV DWORD PTR DS:[EAX],ECX
00463372    8B4C24 3C        MOV ECX,DWORD PTR SS:[ESP+3C]
00463376    8950 04          MOV DWORD PTR DS:[EAX+4],EDX
00463379    8B5424 40        MOV EDX,DWORD PTR SS:[ESP+40]
0046337D    8948 08          MOV DWORD PTR DS:[EAX+8],ECX
00463380    8950 0C          MOV DWORD PTR DS:[EAX+C],EDX
00463383    E8 680CFFFF      CALL Uedit32.00453FF0                      ; 该 CALL 中调用 6E6D 资源
```

而中文版则不然，英文版程序大小为 9.66M 而中文版大小为 12M

![](img/cf2c0a694ada00f6fcc2a5c09bdb54d1.jpg)

英文版下载链接（1038 版）： [`www.ultraedit.com/files/ue_english.zip`](http://www.ultraedit.com/files/ue_english.zip)

中文版下载链接（1032 版）： [`www.ultraedit.com/files/ue_chinese.zip`](http://www.ultraedit.com/files/ue_chinese.zip)

那多出来的代码是什么呢 ？ 通过调试得中文版的 `UE` 做了很多 `BT` 的事情，`OD` 调试的时候首先是无法下 `INT3` 断点的。程序在启动时会效检代码段数据是否被修改过，所以我们下 `BP ReadProcessMemory` 函数或者对我们下 `INT3` 断点的数据下内存读写断点（我喜欢后者）。然后还没有结束，程序会用效检数值解密代码并将数据填充到代码段某几段数据，说的俗点就是 `SMC` 解码代码段的几段数据。

我调试的是 1032 的版本，笔记比较乱，我详细说思路和方法，所以简单贴上关键点，对于读取代码段数据进行效检部分的代码就不贴了，留给有心人去自己调试吧，这里只贴出程序 `SMC` 填写那三段数据的函数：

```
00C21D7F    8B4D 08          MOV ECX,DWORD PTR SS:[EBP+8]
00C21D82    51               PUSH ECX                                   ; ECX 是长度
00C21D83    8B55 C8          MOV EDX,DWORD PTR SS:[EBP-38]
00C21D86    52               PUSH EDX                                   ; EDX 是源代码段
00C21D87    8B45 0C          MOV EAX,DWORD PTR SS:[EBP+C]
00C21D8A    50               PUSH EAX                                   ; EAX 是要目的代码段
00C21D8B    E8 6040DFFF      CALL Uedit32\_.00A15DF0                     ; 该 CALL 实现 SMC 写入
```

这样一共 `SMC` 动态填充程序的三段数据。问题就来了：该函数的第二个参数（`EDX` 的源数据）是如何解密的来的呢？ 这里有兴趣的朋友可以继续内存断点来分析。我是这样想的，如果是动态效检的结果再去计算得到的源数据，而调用的函数都是同一个，与其处理效检值，不若直接把这个 `COPY` 函数 `NOP` 掉，手动把正确的代码 `Patch` 回去，然后把这个函数 `NOP` 掉好了。这个是破解壳的时候对付 `SMC` 的常见手法哈。还好啦，就三段，不然会死人啦 ……

程序跑起来之后基本上就没啥了，但是 `BT` 的 `UE` 又开始效检，又把我们自己 `Patch` 进去的三段代码给乱码写回去，真够猥琐的。于是乎把那里也咔嚓掉。看来 `UE` 对中文用户特别关照了，这 3M 真不白给哈。有兴趣的朋友可以一起分析一下 `UE` 中文版本的效检。就到这吧，有有时间再来灌水 ~~
