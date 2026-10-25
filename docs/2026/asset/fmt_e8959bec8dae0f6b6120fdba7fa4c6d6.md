
# [原创]利用OD通过Call调用实现扫雷（win7旗舰版32位）

## 前言

一直对逆向感兴趣，就拿最简单的扫雷开始，对于XP系统中的扫雷，雷的数目以及雷区的地址都是固定的，可以直接通过Cheat Engine搜索出来，然后在OD中直接下内存访问断点，找到扫雷整个区域，获得行数和列数，再顺藤摸瓜找到左键点击的处理函数和右键点击的处理函数，读取雷区数据，一一比较，如果是雷就右键标记，如果不是可以直接左键点开，即可完成秒杀。

经过了大概半个月的时间，终于在查阅各种帖子和不断的摸索下，实现了Win7 32位系统扫雷的秒杀，哈哈，太开心了！

## 思路来源

因为XP系统的扫雷中雷区的地址是固定的，可以通过CE进行搜索出来，但Win7系统的扫雷是用C++写的，雷区不再是固定的，而是在鼠标第一次点击之后再产生雷的位置。对Win7扫雷一开始没有一点思路，后来看了很多牛人写的帖子，在他们的基础上才能慢慢的入手。可以参考：

- 《Vista的扫雷》：http://bbs.pediy.com/thread-40295.htm
- 《Win7自动扫雷》：http://bbs.pediy.com/thread-120152.htm

这2篇文章里的思路都是通过对`rand`函数下断，分析找到雷存放的地址，每个格子里存放0或1，如果是1则是雷，是0则不是雷，发送消息模拟鼠标的单击，进行自动扫雷。其中，在《Vista的扫雷》提到了重定位的处理，解决方案2对于Win7 32位下的扫雷并不适用，这篇文章中提到：

> 假设扫雷入口地址设为`dwCodeAddress`，则`[3931B8]`这个地址得到的方法是：`( dwCodeAddress & 0xFFFF0000 ) + 0x531B8`

## 重定位处理

但是在Win7 32位的扫雷中，并不是这样的，用PEiD打开扫雷：

![](https://bbs.kanxue.com/upload/attach/201707/681841_17uwcdcwzr4rjoa.png)

可以看到，扫雷的入口点是`0x0002e08f`，用OD打开扫雷，停在了入口点，代码是：

```
001CE08F >  $  E8 B5050000   call MineSwee.001CE649
001CE094   .^ E9 4DFDFFFF   jmp MineSwee.001CDDE6
```

所以扫雷中地址的计算方法是：扫雷模块地址 + 偏移

入口点地址`0x1ce08f` = 扫雷模块地址 + `0x2e08f`，所以此次加载扫雷模块地址为：

`MineSweeper.exe = 0x1A0000`

在此我找到了一种可以去除重定位的方法，PE头中的文件头的`Characteristics`字段指定了文件的类型，占用2字节，其第0比特为1，即表示文件中不存在重定位信息，用C32Asm打开扫雷，找到该字段：

![](https://bbs.kanxue.com/upload/attach/201707/681841_hkygbsomrdd4uoc.png)

该字段为`0x0102`，其第0比特为0，即存在重定位信息，将其修改为1，将会给我们分析带来很大的方便（地址固定），即将该字段修改为`0x0103`，在下面的分析中我就用修改后的文件进行分析，该修改并不影响分析结果，同样适用于没修改的扫雷（因为偏移是固定的）。分析前面的过程在《Vista的扫雷》和《Win7自动扫雷》中讲得很详细，在此不做赘述，简单走一遍，看过的可以直接跳过。

## 动态跟踪：对`rand`下断

首先，用OD加载扫雷，对`rand`函数下断：`bp rand`，然后先将该断点禁止，按F9运行起来，启动该断点，扫雷断下：

```
77C3C070 >  E8 59E5FFFF     call msvcrt.77C3A5CE                    ; rand 断点处
77C3C075    8B48 14         mov ecx,dword ptr ds:[eax+0x14]
77C3C078    69C9 FD430300   imul ecx,ecx,0x343FD
77C3C07E    81C1 C39E2600   add ecx,0x269EC3
77C3C084    8948 14         mov dword ptr ds:[eax+0x14],ecx
77C3C087    8BC1            mov eax,ecx
77C3C089    C1E8 10         shr eax,0x10
77C3C08C    25 FF7F0000     and eax,0x7FFF
77C3C091    C3              retn
```

一看便知该模块不是扫雷的模块，而是`msvcrt`。在堆栈窗口中看到：

![](https://bbs.kanxue.com/upload/attach/201707/681841_8vumqonh1q51118.png)

在第1行处反汇编跟随（按Enter），到：

```
0102D7F3  /$  8BFF          mov edi,edi                ;调用rand的小函数
0102D7F5  |.  55            push ebp
0102D7F6  |.  8BEC          mov ebp,esp
0102D7F8  |.  FF15 B8160001 call dword ptr ds:[<&msvcrt.rand>]       ; [rand
0102D7FE  |.  8B4D 0C       mov ecx,dword ptr ss:[ebp+0xC]
0102D801  |.  2B4D 08       sub ecx,dword ptr ss:[ebp+0x8]
0102D804  |.  99            cdq
0102D805  |.  41            inc ecx
0102D806  |.  F7F9          idiv ecx
0102D808  |.  8BC2          mov eax,edx
0102D80A  |.  0345 08       add eax,dword ptr ss:[ebp+0x8]
0102D80D  |.  5D            pop ebp
0102D80E  \.  C2 0800       retn 0x8
```

鼠标点到该函数第一行，OD在反汇编窗口下方会有提示：

```
本地调用来自 01020176, 01023169, 01023177, 01023BB3, 01027466, 010275B9, 01027E86, 01027E98
```

对这8个`call`都进行下断，按F9，会发现一直断在：

```
01023BB3  |.  E8 3B9C0000   call MineSwee.0102D7F3                   ;  一直断，没用删除
```

删除这个断点，继续按F9，回到游戏，不会中断了，开始游戏并点击一个格子，断在如下位置：

```
01020176  |.  E8 78D60000   |call MineSwee.0102D7F3
```

往上看，则如《Vista的扫雷》中一样，是根据第一次点击的格子产生序号并保存序号的代码，在此列举一下，看过的可以跳过。
