
# 英雄无敌 III

> 原文：[`www.pediy.com/kssd/pediy06/pediy6596.htm`](https://www.pediy.com/kssd/pediy06/pediy6596.htm)

偶有一个非常喜欢的游戏，英雄无敌 III，99 年出的游戏，现在连她的发行公司都破产了，这样老掉牙的东西当然没人更新乐（虽然现在有一批俄罗斯发烧友在做 MOD）。但是能不能自己给她加入新的乐趣呢？

怎么不行泥~WE ARE CRACKERS ！下面是偶给这个游戏做的一个补丁的部分功能的过程。想想当初学 CRACK 的时候就是看着各位大大的文章长大的，现在有空也写点什么吧，但是水平很屁，纯属抛砖引玉：）

英雄无敌这个游戏有 5 个难度，但是作为一个骨灰玩家，这第 5 个难度已经不能满足偶的被虐待欲了，于是想到为这个做游戏增加难度的补丁。

加大难度最直接的方法当然就是给电脑更多的钱和资源，和电脑英雄更多的经验。于是制作这个补丁就分为 3 个步骤：

1. 找出程序中加资源和战斗后加经验的地方
2. 写自己的增加资源和经验的函数，并做成 DLL
3. 修改游戏 EXE 文件，在游戏中调用自己写的在 DLL 中的函数

LET'S GO！第一步很简单，运行游戏，在内存中找出存放钱和经验的的地址，然后叫出 SOFTICE，在找到的内存地址上设下断点。切换到游戏，度日（游戏中每过一日就会根据你占据的领地加钱和资源），SOFTICE 跳出：

```asm
:0054ECD7 cmp dword ptr [ebp-20], 00000007
:0054ECDB jge 0054ECFA
:0054ECDD mov edx, dword ptr [ebp-20]
:0054ECE0 mov eax, dword ptr [ebp-1C]
:0054ECE3 mov ecx, dword ptr [eax+4*edx]
:0054ECE6 mov edx, dword ptr [ebp-20]
:0054ECE9 mov eax, dword ptr [ebp-18]
:0054ECEC add ecx, dword ptr [eax+4*edx]
:0054ECEF mov edx, dword ptr [ebp-20]
:0054ECF2 mov eax, dword ptr [ebp-1C]
:0054ECF5 mov dword ptr [eax+4*edx], ecx
:0054ECF8 jmp 0054ECCE
:0054ECFA jmp 0054EC79
```

这里就是加资源和钱的循环，由于有 6 种资源和钱，所以这个循环要循环 7 次，每次增加一种资源。可以通过修改这里的代码来让电脑获得更多的资源，但是必须要知道当前写的内存单位是不是代表电脑的资源，如果不管就会导致连自己的资源也增加了，这样起不到增强电脑的效果。

于是必须要找到游戏中数据的绝对寻址（虚拟地址）。资源这种东西在程序中肯定会为它分配一个永久的空间，而不会是在局部栈中分配，也不会在堆中分配。

因此这是一个全局变量。（即使 C++中类把它的成员封装了，但是编译时还是会通过 NAME-MANGLING 生成一个全局唯一的标识符）

```asm
:00458098 imul edx, 00000168
:0045809E mov eax, dword ptr [00826D40]
:004580A3 lea ecx, dword ptr [eax+edx+00020AD0]
:004580AA mov edx, dword ptr [ebp-30]
:004580AD mov eax, dword ptr [ebp-34]
:004580B0 mov esi, dword ptr [ebp-30]
:004580B3 mov eax, dword ptr [eax+4*edx+0000009C]
```

从这里知道资源的寻址：

`[82B0B4] * 168 + [826D40] + 20AD0 + 9C + 资源号 * 4`

可以看出，这个寻址是在编译后就确定下来的。

然后找出判别是否为电脑的标志：

`[826D40] + 20AD0 + [82B0B4] * 168 + E2`

这里可以看出一点面向对象的影子，这个游戏听说是用 VC 做的。`[826D40] + 20AD0` 是一个类的所有对象基地址，应该是每一个势力就是这个类一个对象，并且编译器为每个对象分配的数据空间为 168H 个字节。`[82B0B4]`就是势力的编号。知道了这些之后就可以进入第二步了。

写一个函数，作用是先判断是不是电脑，如果是，就为它加上额外的资源。

入口参数为当前的势力编号好了：

`DLLIMPORT void AddResource(int iPlayerID);`

函数中把通过上面两个寻址完成判断和增加资源的任务。同样加经验的函数也是如此。是不是很简单：）

写完所有函数之后，就编译成一个 DLL

头文件如下

```c
#ifndef _PACKDll_H_
#define _PACKDll_H_

#if BUILDING_DLL
# define DLLIMPORT __declspec(dllexport)
#else /* Not BUILDING_DLL */
# define DLLIMPORT __declspec(dllimport)
#endif /* Not BUILDING_DLL */

DLLIMPORT void Initialization();
DLLIMPORT int MakeMonster(int );
DLLIMPORT int ChangMntNmbToID(int );
DLLIMPORT void AddResource(int );
DLLIMPORT int AddExp(int );
DLLIMPORT int Remark(int );

#endif /* _DLL_H_ */
```

可以看出有 6 个出口函数。

完成第二步进入第三步，在游戏的 EXE 文件中加入引用自己写的 DLL 中函数的代码。首先是修改 EXE 的引入表，增入自己的 DLL，这样程序在运行的开始才会连接偶写的 DLL。

用 PE 编辑工具 `LordPE` 打开 `HEROES3.EXE`，选择引入表，回发现程序所有 DLL 和引入函数。增加一个 DLL，输入 DLL 中的所有引出函数名，`LordPE` 就帮你把引入表做好了，是不是太简单鸟？糊里糊涂就搞定了？`THUNKRVA` 是什么？`THUNKVAULE` 又是什么东东？那看看真正的引入表是怎样的吧：

```
43 66 50 61 63 6B 2E 64 6C 6C 00 00 00 49 6E 69
74 69 61 6C 69 7A 61 74 69 6F 6E 00 00 01 4D 61
6B 65 4D 6F 6E 73 74 65 72 00 00 02 43 68 61 6E
67 4D 6E 74 4E 6D 62 54 6F 49 44 00 03 00 41 64
64 52 65 73 6F 75 72 63 65 00 04 00 41 64 64 45
78 70 00 05 00 52 65 6D 61 72 6B 00 0B 10 47 00
1C 10 47 00 2A 10 47 00 3C 10 47 00 4A 10 47 00
53 10 47 00
```

第一行 `43 66 50 61 63 6B 2E 64 6C 6C` 是这个 DLL 的名字，也就是 `CfPack.dll`

然后有一个 `00`，是名字的结束标志。接下来的 `0000` 就是标志第一个函数的开始

也是这个函数在 DLL 中的序号。`49 6E 69 74 69 61 6C 69 7A 61 74 69 6F 6E` 就是第一个引入函数的名字：`Initialization` 接下来是结束附 `00`，然后又是下一个函数。一共 6 个函数，到第 6 行的 `6B`（红色）结束。

下来是什么呢？就是每个函数的 `THUNKVALUE` 也就是每个出口函数的符号在引入表中的地址。如 `6B` 接下来的 `0B 10 47`，就是第一个函数的符号的地址。也就是第一行中红色的 `00` 的地址 `0047100B`，下来 `00` 是分隔符，然后是第二个引入函数符号的地址 `0047101C`，指向第二行的红字部分。这样一共 6 个地址，分别指向 6 个引入函数的名字。

搞定引入表后，就可以该原代码了。

首先，在 `HEROES3.EXE` 中找到空白的部分，如

```asm
:007AB425 int 03
:007AB426 int 03
:007AB427 int 03
:007AB428 int 03
:007AB429 int 03
:007AB42A int 03
:007AB42B int 03
:007AB42C int 03
:007AB42D int 03
:007AB42E int 03
:007AB42F int 03
...
```

这里有一大段空白的部分

然后跳到加资源的地方：

```asm
:0054ECD7 cmp dword ptr [ebp-20], 00000007
:0054ECDB jge 0054ECFA
:0054ECDD mov edx, dword ptr [ebp-20]
:0054ECE0 mov eax, dword ptr [ebp-1C]
:0054ECE3 mov ecx, dword ptr [eax+4*edx]
:0054ECE6 mov edx, dword ptr [ebp-20]
:0054ECE9 mov eax, dword ptr [ebp-18]
:0054ECEC add ecx, dword ptr [eax+4*edx]
:0054ECEF mov edx, dword ptr [ebp-20]
:0054ECF2 mov eax, dword ptr [ebp-1C]
:0054ECF5 mov dword ptr [eax+4*edx], ecx
:0054ECF8 jmp 007AB46D
```

兰色的部分被改了，跳到了空白的地方，然后，呵呵，怎么搞都可以鸟！

把原来的 `int 03` 改成以下代码

```asm
:007AB46D 8B4DF8                  mov ecx, dword ptr [ebp-08]
:007AB470 51                      push ecx
* Reference To: CfPack.AddResource, Ord:0003h
:007AB471 FF1568108700             Call dword ptr [00871068]
:007AB477 83C404                  add esp, 00000004
:007AB47A E9FA37DAFF              jmp 0054EC79
:007AB47F CC                      int 03
```

调用 `CfPack.AddResource` 后跳会去，神不知，鬼不觉就运行给电脑加资源的过程，呵呵：）

这里的 `Call dword ptr [00871068]` 调用的 `CfPack.AddResource` 地址是怎么知道啊？`[00871068]` 就是这个函数的地址了，这个地址是由引入表中的 `THUNKRVA`：`471068`（系统在装入 DLL 之后填写的该函数接口的具体地址）加上程序装如地址 `400000H` 得到的。

同理，其他功能也可以由类似方法完成。

当然了，还要写一个图形界面用来设置新的难度等级，把设置写在一个文件中，然后在 DLL 中加入读取设置的函数 `Initialization()` 放在游戏运行开始的地方运行就 OK 了。

呵呵，新的难度下又终于可以被电脑虐待了~~-\_-!!

PS：这个补丁在英雄世界上可以下载。

[`www.heroworld.net/heroes4site/L06_download/Files/H3-AI.rar`](http://www.heroworld.net/heroes4site/L06_download/Files/H3-AI.rar)

`4dd466d0e1e0cc123afbe504da2462ca`
