
# [原创]Cheat Engine进阶教程：`gtutorial-i386`闯关记 第三关 [完结撒花]

作者延续前一关变态作风，gtutorial第三关依然是不修改游戏无法过关（至少我不能！）：左下角那个白色小人是玩家，画面上3个刺猬是移动障碍物，玩家触之即死。游戏提示为：将所有红色平台变绿即可开门。

整个游戏有2个难点：难点1，如果不修改游戏，玩家无法从平台A跳上平台B，更何况地面上还有刺猬捣乱，影响发挥！难点2，门开启的瞬间，这3个刺猬飞向门边，摆出副上门讨债的架势把门堵死了，玩家碰到刺猬又会过关失败，所以玩家根本无法进门。

![](https://bbs.kanxue.com/upload/attach/201902/399589_6TUY53CC5VBJE5C.png)

现在进入正题，说说怎么过关，受这篇Creating a cheat table - Coordinates文章的启发，我的想法是修改玩家的坐标，让它可以到达画面的任意位置。由于这关有2个难点，所以本文分两部分介绍。

## 难点1：开门

先说难点1，怎么开门：在整个游戏中，除了玩家可以上下运动（跳跃），玩家和障碍物都可以横向运动，所以定位玩家的Y坐标要比定位X坐标简单。最开始，我在几个比较低的平台上来回跳跃来改变及定位玩家的Y坐标，但是由于刺猬的影响，Y坐标还没确定游戏就结束了。所以我重新审视了一下游戏：作者说要让所有平台变绿才能开门，我算了下连同玩家脚下的地面一共有12块平台，作者有没有可能再程序中设置了全局变量记录变绿的平台数？让我们试一试。游戏开始时，地面已经变色，所以，在CE的`value`编辑框中输入1。`Scan Type`选择`Exact Value`；`Value Type`选择`4 Byte`，然后按`first scan`做初次扫描。接着再跳平台-扫描，重复几次，马上能定位存储着变色的平台的变量：

![](https://bbs.kanxue.com/upload/attach/201902/399589_CU97N4BT4WH2XV7.png)

定位变量后就简单了，直接将变量值改为12，画面右边的门马上会开，但同时，游戏的后半部分情节马上开始（看看小人的表情，分明是再说WTF）：

![](https://bbs.kanxue.com/upload/attach/201902/399589_TAQSDPJ9FVU8ER3.png)

## 难点2：坐标定位

2.刺猬堵门后，由于没有干扰，搜索玩家的坐标反而变得一马平川了。先搜索X坐标，让玩家回到画面左下角，在CE上点击`New Scan`，`Scan Type`设为`Unknown initial value`，`Value Type`设为`Float`，然后点击`First Scan`。之后，控制玩家慢慢向右移动，每次移动-暂停时，在CE中设置`Scan Type`为`Increased value`并点击`Next Scan`，反复几次即可定位x坐标。最终，读者会发现x坐标在[-1,1]区间内变动：

![](https://bbs.kanxue.com/upload/attach/201902/399589_H9G88G76PWQHTNK.png)

有了存储x坐标的变量的地址，估摸着存储y坐标的变量的地址肯定在附近。由于`sizeof(float)=4`，所以y坐标可能距离x坐标4字节。我们需要从Cheat table中复制黏贴2次`x-coor`项。第一次设置`Adjust address by`的值为4（`Adjust address by`意为，复制项的地址距离源项的偏移），并设置该新添加的项在CT表中的`Description`为`y-coor(+4)`；第二次设置`Adjust address by`的值为-4，并设置该新添加的项在CT表中的`Description`为`y-coor(-4)`：

![](https://bbs.kanxue.com/upload/attach/201902/399589_DMBJPXPHF3GB7W6.png)

当然刚才添加的两个CT项并不一定就是y坐标，需要通过上下跳跃来验证其正确性。在游戏中按下`p`键即可让游戏暂定，我们在玩家起跳时记录下原始数据，当玩家起跳后再次记录数据，落地后第三次记录数据。通过以上步骤可以找到x坐标的地址+`sizeof(float)`=y坐标的地址。有了这两个坐标变量就可以随意改变玩家的位置，比如浮空（要达到浮空的效果，记得把x坐标和y坐标两个变量`Freeze`）如下图：

![](https://bbs.kanxue.com/upload/attach/201902/399589_PDP2EZHV7D2YV2Y.png)

至此，过关不成问题了。但是，总觉得少了点什么？对，没做外挂！恩，这关的外挂做什么呢？好像整个游戏中定位玩家坐标的过程最为复杂，那就做个定位外挂，用来定位玩家坐标的基址~

## 编写外挂：定位玩家坐标基址

首先定位访问y坐标的指令，在CT表中选中`y-coor`项，右键`Find out what write to this address`

![](https://bbs.kanxue.com/upload/attach/201902/399589_TXTVCJ7ADBTF69Q.png)

回到游戏中移动玩家，这时CE的`The following opcodes write to`窗口会列出修改y坐标的指令以及存储玩家坐标的结构体的基址（`Extra Info`中的 `The value of the pointer needed to find this address is probabyly 01905980`）：

![](https://bbs.kanxue.com/upload/attach/201902/399589_J47TYZHFDYN5PK8.png)

`Extra Info`窗口中显示，访问坐标基址的指针存放在`edx`中，好，那我们就写个AA脚本，将`edx`的写到全局可被访问的内存中，脚本如下：

```text
[ENABLE]

aobscanmodule(INJECT,gtutorial-i386.exe,D9 5A 1C 8B 45 D4 8B) // should be unique

alloc(newmem,$1000)
alloc(coorbase,4)

label(code)
label(return)

newmem:
  mov [coorbase],edx

code:
  fstp dword ptr [edx+1C]
  mov eax,[ebp-2C]
  jmp return

INJECT:
  jmp newmem
  nop

return:

registersymbol(INJECT)
registersymbol(coorbase) //声明符号，这样就可以在CE中访问coorbase

[DISABLE]

INJECT:
  db D9 5A 1C 8B 45 D4

unregistersymbol(INJECT)
unregistersymbol(coorbase)

dealloc(newmem)
dealloc(coorbase)
```

激活脚本后，需要在CE的CT表中创建`coorbase`项——点击`Add address Manually`并填写`Add address`窗口中的内容：

![](https://bbs.kanxue.com/upload/attach/201902/399589_TZZZTFRRCNU677X.png)

`coorbase`存放的是指针，我们最终是要通过这个指针分别访问x/y坐标，所以还要分别手头添加2个地址，并把这几项拖到`Step3`下形成一组。并在`Step3`项上右键菜单`Group config`-`Hide children when deactived`：

![](https://bbs.kanxue.com/upload/attach/201902/399589_NVMMNRUF22N2Q4V.png)

选这个选项的原因是`coorbase`是在`Step3`脚本中创建的，当`disable`这个脚本时`coorbase`被释放，再从被释放的地址去读玩家坐标显得不合理，故如此处理。

这是最终CT表中的表项：

![](https://bbs.kanxue.com/upload/attach/201902/399589_RQFH3V52ZM35DV4.png)

当再次进入这个关卡并应用这个脚本马上能获得玩家的坐标~

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
