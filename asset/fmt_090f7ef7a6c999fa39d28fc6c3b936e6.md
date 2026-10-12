
# WAR3 3C 与 MapHack 的游戏(一)(二［20 楼］)

> 原文：[`www.pediy.com/kssd/pediy10/81268.html`](https://www.pediy.com/kssd/pediy10/81268.html)

好久不在看雪写点玩意了。

最近跟大牛玩 `war3`。基本上是被打的到处跑![](img/0d3dbf5fb8604477afb4a1766fe8c72d.jpg "尴尬")

所以偶要考虑写一点辅助工具来帮助。

## 1. 背景介绍

本篇是这灌水系列文章的第一篇，先稍微灌水 1 下，简单介绍 1 点基本知识把。

三大 RTS 游戏按照时间顺序是 `红警`、`星际`、`魔兽`。

最老的产品是 `红警`，不过那时出来的时候偶还米电脑，`星际`出来的时候偶也没玩。

大概是 `魔兽 3` 出品的时候偶才真正有机会玩了几把，不过一组玩的不怎么样。

主要原因还是操作不敏捷，手感不灵活。哈。

从总体上说 `魔兽`是从 3 代开始正式引入英雄这个概念的，从此为 `魔兽`在 RTS 界成为新一代大作奠定了坚实的基础，基本上 `暴雪`出品的平衡性都不错，因此几乎都能长时间流行哈。

不多灌水了，详细的比较可以看 `http://games.qq.com/a/20060601/000313.htm`。

个人认为是 `魔兽`出兵比 `星际`慢，而且一个英雄对于战局的影响之大，是其他任何情况无法与之相比的，因此对于操作有更高的要求，不是只要出兵速度比死兵快就可以了。`魔兽`出兵过多要承担高额维修费用，这也是制约因素之一。

### 作弊工具与目的

首先要说明所有的 `maphack` 作弊战术都是用于实际的对战的，单机没有什么意义，也没有必要。作弊方法对于不同的平台有一些小差异，目前流行的三大对战平台，`浩方`、`QQ`、`VS`，还有一个 `GG`，不过在写文的时候偶无法访问官方站点，就先不做讨论把。

作弊最多的方式就是 `maphack`，就是打开地图，看见一切对手的动向，从而免去侦查的麻烦。另外还有一些利用游戏 bug 刷积分，欺骗平台等，不过偶认为意义不大。

重要的是做游戏辅助，在打开了地图之后，就能更加方便的实现来犯预警、自动反击、智能控制作战队形。偶看高手 `nn` 牛的录像，英雄在作战是飞速的绕圈攻击，深深佩服操作能力之高超。

## 2. 修改原理分析

首先我们要明白，所有的修改都是针对 `game.dll` 进行的。这个 `9M` 大小的文件，偶测试的版本是 `1.20e`，`dll` 版本号是 `Warcraft III 1.20.4.6074`。

修改方法有 2 种：
- 一种是直接修改文件，这种方法简单快捷，缺点是容易被检测到。
- 另外一种就是内存修改了，这种方法相比之下难于检测很多，也比较好针对性的避免某检测机制。

还有，国内目前没有基本上没有多少人玩官方站网，基本上都是使用上述三大平台提供的虚拟局域网模式，因此各大平台势必大同小异，如果修改比较适当是可以直接通杀的。

其中比较大的作弊工具站点是 `warmh.cn`，提供了不少工具。我们为了减小分析工作量要站在前人的肩膀上分析。正因为都是虚拟局域网之类的模式，因此平台不可能实际性的在服务器对游戏数据分析检测，因此只有可能在本地进行简单的数据效验或者代码效验，这样使得我们的工作量大大减轻了，基本上只需分析出队员数据就可以修改了。

### 修改限制

- 我们不能修改的：`金钱`等实际数值，这是游戏各个客户端互相计算效验的。
- 我们只能做：打开地图、自动显血、攻击力输出显示等游戏本来就提供的功能。
- 辅助功能：上一些键盘鼠标控制，可以紧急喝药、紧急回城、战斗辅助等功能。

这些多少在前人的作品中都有体现，在下一篇中偶将试图深入分析。

这个灌水就先到这里把。![](img/f43c1e514327696dccbb0da997496b2a.jpg "扮鬼脸")

## 3. Patch 点分析

本篇是灌水系列的第二篇。

来参照别人的 `mh` 获取我们需要的 `patch` 点。

如果你希望自己分析，有果子林的文章：

- `魔兽争霸 3` 的 `MapHack` 制作教程（一）显示敌对单位
  `http://alzzl.spaces.live.com/blog/cns!C1FF77E38059705D!274.entry`
- `魔兽争霸 3` 的 `MapHack` 制作教程（二）去除战争迷雾
  `http://alzzl.spaces.live.com/blog/cns!C1FF77E38059705D!281.entry`

里面还有不少 `WOW` 的文章值得细读。

我们简单起见，直接拿别人已经做好的 `MH` 分析。偶分析的是 `Bmap`。

以下所有的都是 `offset`，由于 `game.dll` 基址可变，一般用 `offset` 比较稳定。

```
1.patch  showunitsingame
0x3A04AB

2.patch  showunitsmap
0x36087C

3.  patch  clickableunits
0x28464C

4.patch  clickableunits 2
0x284662

5.patch  revealillu
0x281F1C

6.  patch  removefogingame
0x73B949

7.patch  pingsignal
0x42F836

8.patch  showinvisiblemap
0x360C91

9.patch  showinvisiblemap
0x3558FE

10.bypass  DotA -ah command
0x3C3F02
0x3BF63C

11.enable  viewable items
0x3F75AA
0x3A03EB

12.enable  colored invisibles
exe:
0x4559EC
0x455A60
0x455A65
0x455A6A
dll:
0x398E09

13.enable  view skills
0x201B9C
0x34E6E8

14.enable  view cooldowns
0x28D68E
0x34E726

15.enable  click invisible units
0x284662

16.  patch  tradeamount
0x34D1E2
0x34D1EB

17.patch  view all icons
0x36FFD1
0x36FFD7

18.patch  view enemy icons
0x36FFD1
0x36FFD7
```

基本上国内的几个 `MH`，比如 `warmh`、`lynnmh`、`ayssmh` 基本上都是这 `patch` 点，只是 `patch` 方式有所差别而已。文件的没什么，内存的就各有千秋了。标准写 `writeprocessmemory`、`lpk`、纯 `asm` 代码等各种方法均有。

这些都是为了避免各种游戏平台的本地检测的。对于游戏而言，没用什么实际检测。当然这是局域网模式的特点，连接 `BN` 战网又是一回事。国内偶没看到多少人能连 `BN` 游戏的，当然偶也没 `Z` 版 `cdkey`，自然也不可能测试了。

### 游戏 Bug 与限制

关于游戏 bug，确实存在的，有几种 bug 情况。一种是利用游戏 bug 出错，另外几种是交换 bug 造成 `CollDown` 时间免除等 bug，不过一般作用不大。偶也不喜欢去利用 bug 刷积分。偶们写辅助工具是为了玩的舒服一点，不是为了刷积分。其实要刷积分直接欺骗平台快多了，根本不需要游戏，哈哈。

另外这里是简单提一下，不能直接修改 `金钱`和 `攻击力`，`锁定生命`也是不行的。

简单起见，我们只关心感兴趣的几个功能：
1. 去除迷雾
2. 显示隐身单位
3. 识别幻象分身

其余的功能个人认为作用不大。其实功能 2 和 3 对于其他的英雄没什么大威胁。只是对于 `剑圣`来说，把 2 个主功能给消除了，就明显被动了不少。那些没隐身和分身的单位基本上作用不大的了。

## 4. Hook 引擎与检测规避

好啦，关于修改代码的几个问题，上面的 `Bmap` 是直接打 `nop` 或者 `jmp` 解决的。

直接修改文件肯定不用考虑了，这是任何效验都会发现的。这种办法是肯定被游戏平台封杀了。偶们需要使用一个 `hook` 引擎来把这一段代码整个的搬出去运行，这样可以躲避内存效验（看各种平台具体情况）。

有的平台喜欢做内存 `md5`，那么我们就直接欺骗这个 `md5` 的算法函数，可以让其返回特定数值。更好的办法是内存重定位就是我们把原始的复制出去一份，让效验的去检测拿原始的，这个在 `netroc` 大侠的文章有介绍。

不过偶也不想用 `hook` `int0e`，那样成本太高了。玩个游戏还写驱动，划不来啊！[](img/f43c1e514327696dccbb0da997496b2a.jpg "扮鬼脸")

### Hook 引擎选择

关于 `hook` 引擎的选择。其实偶根本不怎么懂得引擎的，只是随便说说，不当之处欢迎大家指正。

首先，偶不喜欢那种复杂的引擎。偶们是懒人，要求使用简单效果好。

那么要求是这样，设函数为 `gamefunc`：
- 偶们要直接输出 `gamefunc`、`mygamefunc`、`oldgamefunc`
- `gamefunc` - 原始游戏内部函数
- `mygamefunc` - 我们的替代函数
- `oldgamefunc` - 如果要在 `mygamefunc` 里面 `call` 原始函数，用这个

以下这种方式偶这种懒人是受不了的：
`http://www.xfocus.net/articles/200511/832.html` 这种方法

```c
__declspec(naked) my_function_detour_ntdeviceiocontrolfile()
{
    __asm
    {
        // exec missing instructions
        push ebp
            mov  ebp, esp
            push 0x01
            push dword ptr [ebp+0x2C]
            // jump to re-entry location in hooked function
            // this gets 'stamped' with the correct address
            // at runtime.
            //
            // we need to hard-code a far jmp, but the assembler
            // that comes with the DDK will not poop this out
            // for us, so we code it manually
            // jmp FAR 0x08:0xAAAAAAAA
            _emit 0xEA
            _emit 0xAA
            _emit 0xAA
            _emit 0xAA
            _emit 0xAA
            _emit 0x08
            _emit 0x00
    }
}
```

还要自己处理堆栈帧，偶很懒，不想用。

稍稍改进一点的是：
`http://www.whitecell.org/forums/viewthread.php?tid=360`

这个，虽然多定义一个 `jmp` 函数，但是起码我们不用自己动堆栈帧了。

不过也复杂了一点，我们还是使用成型的 `hooklib` 也许更加方便。

`lib` 的要求是一定要有反汇编引擎或者长度识别，确保 `hook` 任意地方都能成功运行无需人工干预。

偶比较喜欢的类型是 `Ms-Rem` 的 `lib`，直接输出三函数就可以了。虽然老 `V` 说这个 `jmp` 比较诡异，不过偶实际测试，没发现 `bug`。

`fengyue` 的 `lib`，确实是功能强大不假，但是使用起来有些复杂：

```c
DWORD WINAPI HookProc(DWORD RetAddr ,__pfnXXXX pfnXXXX, ...);

RetAddr  //调用 api 的返回地址
pfnXXX   //类型为__pfnXXXX，待 hook 的 api 的声明类型，用于调用未被 hook 的 api
```

感觉不是那么方便。

再来不得不灌水 1 下的是 `cooldiyer` 的 `hooklib`。这个就实在是有点不过关了，他的 `GetPatchSize` 在计算偶的 `patch` 点的长度发生了错误。偶不懂引擎，不知道错在哪里。他是基于 `Ms-Rem` 的没错，但是奇异的是 `Ms-Rem` 的 `lib` 计算正常，`cooldiyer` 的计算出错？不明白。

### 技术细节

还有就是一个奇怪的事情，当然偶也不明白。

偶感觉 `jmp far`(7 字节)和 `jmp near`(5 字节)在 `win` 里面似乎没什么差别？似乎+了一个段表示符没什么作用？或者是 `win` 不使用段来寻址？

偶只能猜测 `cooldiyer` 是为了避免这种 `jmp near` 的计算相对地址的麻烦。不过有另外一办法也可以不需要计算相对地址，那就是 `push xxx, ret`。![](img/f43c1e514327696dccbb0da997496b2a.jpg "扮鬼脸")

当然这个属于吹毛求疵了。实际上没什么影响的。只要正常工作，具体用什么指令无所谓的。

## 5. 平台检测措施

接下来就要分析分析各个平台具体的检测措施了。

- **GG 平台**：据说用了驱动 `hook` 了 `ntopenprocess`，不过没事，解决的办法太多了。不过偶这儿无法用 `GG`，算了。
- **浩方**：似乎他检测不怎么严格，再说在偶机器上经常看不见别人，原因未知。不过浩方据说随便你怎么 `MH`，都不会封号，所以可以放心测试。
- **VS**：这个据说是检测最严格的家伙把，而且有封号的风险。他有定期的内存 `hash` 检测(`md5`)，而且还会扫描其他进程的内存特征。如果你开过 `360` 保险箱，那么应该会提示有一个 `dll` 会注入的。这个 `dll` 中也可能包含有一些检测，需要具体分析。

下篇偶再来抓个软柿子捏捏把。今天就灌水这这里咯。![](img/022c916b9660e467a8663bd57a4ae6d1.jpg "狂笑")
