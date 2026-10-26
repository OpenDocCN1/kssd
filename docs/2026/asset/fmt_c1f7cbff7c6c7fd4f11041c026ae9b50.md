
# [原创]Cheat Engine gtutorial-i386闯关记 第二关

进入第二关后会看到如下界面：

![](https://bbs.kanxue.com/upload/attach/201902/399589_U5N9GSMDYM55EWY.png)

第二关简直丧心病狂，除了要1vs2，电脑攻高血厚(攻击力和血都是玩家的2倍)；更要命的玩家一次只能攻击一个目标，但是会同时受到2个目标的还击，躲都躲不了。换句话说，除非改代码，否则根本无法过关。如果Freeze玩家HP，侥幸干掉一个目标，剩下的一个目标瞬间能灭掉玩家，不管玩家有多少血。估计是作者直接把玩家的HP清零了。（顺带一提，如果读者自己扫描几次游戏，会发现玩家的HP是100，2个电脑玩家的HP是200；玩家的每次攻击造成电脑-1HP，而电脑造成玩家-2HP）。这一关的提示让我想起Cheat Engine官网上的一篇教程：Find the team id in the player structure

这一关玩家的HP有精确的数值，所以`Scan Type`改成`Exact Value`即可。玩家挨几次打，马上能定位存放玩家血量的变量，将该变量加入Cheat Table后，右键`Find out what writes to this address`，然后切换进游戏，让玩家再被打一顿，就会显示修改玩家HP的代码了：

![](https://bbs.kanxue.com/upload/attach/201902/399589_3DJWC2YJU26JDJX.png)

![](https://bbs.kanxue.com/upload/attach/201902/399589_S5WSJKTSXAK2TD5.png)

在教程 Find the team id in the player structure 中间接的提到，有些游戏可能会设计如下形式的扣血函数：

``` 
int DecreaseHP(RoleObj*);
```

当玩家(假设为`PlayerObj`，继承自`RoleObj`)或者电脑(假设为`CPUObj`，同样继承自`RoleObj`)掉血时，会把`PlayObj`和`RoleObj`传递给`DecreaseHP`函数，由该函数完成扣血功能。借助这个思路，让我假设指令

```
gtutorial-i386.exe+34B70 - 29 50 50              - sub [eax+50],edx
```

就是`DecreaseHP`函数中完成扣血的指令。根据这样的假设，当电脑收到攻击后也会执行这条指令，在此设置断点，我们不就可以查看并比对`PlayObj`和`CPUObj`的内存布局了？CE提供了较为简便的方法实现上诉目的：

1). 在`Memory Viewer`窗口指令`gtutorial-i386.exe+34B70 - 29 50 50              - sub [eax+50],edx`处右键--选择`Find out what address this instruction access`，点击该菜单项后会弹出`Changed Address by xxxx`窗口：

![](https://bbs.kanxue.com/upload/attach/201902/399589_UGUZTM6Y43U4BN4.png)

![](https://bbs.kanxue.com/upload/attach/201902/399589_GU9FMNSP4AHK2QE.png)

2). 返回到游戏，让玩家和2个CPU都掉血，然后查看窗口中的内存项，由于在游戏中我们修改了3个对象的血量，所以窗口中有3个记录项。全选这3个记录项，右键选择`Open dissect data with selected address`，这时CE会问一堆问题，一路勾选确定最终打开数据解析窗口：

![](https://bbs.kanxue.com/upload/attach/201902/399589_UTDMSWYG68SWGD9.png)

![](https://bbs.kanxue.com/upload/attach/201902/399589_GV6PFGU2DWCQ75B.png)

![](https://bbs.kanxue.com/upload/attach/201902/399589_8NDUAVRQB5QCBWW.png)

大家注意到偏移窗口中偏移`0x50`和`0x54`这两行数据没？这是整个窗口中为数不多的看着有点意义的数值。结合反汇编代码，我们可以断定对象首地址的偏移为`0x50`处为角色当前HP值，而偏移`0x54`处为角色MAX HP值。有了MAX HP这个依据，我们就可以在`AA`(`Auto assemble`)窗口中区分当前修改的是玩家还是CPU。

回到`Memory Viewer`窗口，准备编写注入脚本，选中`sub [eax+50],edx`指令，点击菜单项 `Tools--"Auto assemble"--Template--"AOB injection"`，下面的代码片是我的注入脚本：

```
define(symbol, 29 50 50 C3 00 00)

[ENABLE]

aobscanmodule(INJECT,gtutorial-i386.exe,29 50 50 C3 00 00) // should be unique

assert(INJECT, symbol)

alloc(newmem,$1000)

label(cmpEnd)

label(code)

label(return)

newmem:
  cmp [eax+54],(int)100
  jnz CPURole
  mov edx, (int)0
  jmp cmpEnd

CPURole:
  cmp [eax+54],(int)200
  jnz cmpEnd
  mov edx (int)200
cmpEnd:

code:
  sub [eax+50],edx
  ret 
  add [eax],al
  jmp return
```

保存脚本，然后点击AA窗口 `File--"Assign to current cheat table"`，将脚本分配到Cheat Table中，然后激活脚本，返回到游戏中，验证我们能不能击毁目标：

第一击：

![](https://bbs.kanxue.com/upload/attach/201902/399589_YAYNCFRZQHFZNPV.png)

第二击，通关：

![](https://bbs.kanxue.com/upload/attach/201902/399589_W9HEFY767GEH9A3.png)

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
