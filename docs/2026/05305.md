
# [原创]那些年我曾今玩过的魔兽外挂（一）

## 0x0 序言

本系列文章的发布顺序并非按照外挂功能复杂程度排序，这点请各位看官注意。

另外，亲爱的大手子们，你以为不封你们真的是因为没检测到你们吗？

那么，接下来要说的这个，应该是我目前玩过的在思路上最巧妙的，涉及到的点也是个检测的盲点，在长达10年（毫不夸张）的对抗过程中被双方都忽略点。

文件名：`jasstools.zip`

包含文件：`jasstools.m3d`、`globals.txt`、`function.txt`、`main.txt`、`Game.dll`

MD5：

`jasstools.m3d: f55f15eb7a6c5b962df6a32c40d09372`

`game.dll: 267861a0dfd416dbad13e7ee3ec7794a`

## 0x1 分析

OK，依照惯例从`DllMain()`开始：

![](https://bbs.kanxue.com/upload/attach/202002/615441_RT367VSR5P3QMC2.png)

`DllMain()`非常清晰没什么好看的，当`ul_reason_for_call == DLL_PROCESS_ATTACH`，即dll挂靠的时候执行`InitHack()`函数。

显然这个`InitHack()`函数才是我们要关注的重点。

![](https://bbs.kanxue.com/upload/attach/202002/615441_NRKZVKYHVYTYBRE.png)

就一份外挂来说，这个流程是写得非常符合“war3的规范的”，甚至比一些商业代码都要规范，当然还有更规范的外挂代码。其中有一句代码是我要吐槽的：

```
if ( strstr(&v_file_name, "War3.exe") || (v_ret_val = strstr(&v_file_name, "war3.exe")) != 0 )
```

这种代码在这份显得“非常正规”的代码中简直是神一样的存在，因为正常做法大概是：

```
std::transform(name.begin(), name.end(), name.begin(), ::tolower);
```

虽然这样要多写一点代码，但是看起来更符合这份代码的风格不是吗？

当然这句话我在很多易语言源码中见过比如：`game.dll`、`Game.dll`、`GAME.DLL`比较三次的，实际上根本不用比较。

其流程图大致如下：

![](https://bbs.kanxue.com/upload/attach/202002/615441_66PCU3PRSP36U59.png)

吐槽完毕，显然通过观察流程我们发现样本做了一个非常关键的操作，即hook `g_game_module_handle + 0x1FAB34` 也就是`compline_jass_script()`这个函数。

不得了啦！偷梁换柱嘞！不被发现的那种~~~

好，那么我们看下这个回调函数`DetourGameComplineJassScript()`

![](https://bbs.kanxue.com/upload/attach/202002/615441_T35FUEY4TXYHE7U.png)

就截图了上半部分，下半部分不是关键，所以省略了。

那么这一大串东西，到底做了些什么？其实很简单，看以下伪代码：

```
while（编译的脚本 != nullptr）
{
    if（v_当前编译的脚本名称 == "war3map.j"）
    {
        ......
        v_修改后的脚本大小 == 向原地图脚本中插入作弊脚本();
        将脚本使用暴雪字符串处理规范进行处理();
        覆盖原脚本Buffer();
        ......
    }
}
调用原ComplineJassScript();
```

看以下流程图：

![](https://bbs.kanxue.com/upload/attach/201903/615441_SRJ8U37XAW6JHQE.png)

所以，这个外挂的核心功能流程已经被我们分析清晰了，当然这个关键函数里还有一个关键点：`InsertCheatScriptToMapOrginJassScript()`

这个函数负责向内存中的原地图脚本插入作弊脚本，那么我们跟进去看看它是怎么实现这个操作的：

![](https://bbs.kanxue.com/upload/attach/201903/615441_WWDK3B3GKCAH7UD.png)

![](https://bbs.kanxue.com/upload/attach/201903/615441_GSSF6RNF6EN96PK.png)

![](https://bbs.kanxue.com/upload/attach/201903/615441_W5MUCBKS6JSNSEW.png)

哇！这密密麻麻的一坨是啥？看着好恐怖啊！实际上一点都不恐怖，你只需要关注我画红框的地方。这些就是这个函数的关键，你只要关心它们就好了。

在给出流程图之前，我们要引入一个前置知识，Jass2语言：

![](https://bbs.kanxue.com/upload/attach/201903/615441_P6BK2DJXZ6MW7ZR.png)

当然，百科上说的太抽象了，实际上你可以这么理解，jass2是一种脚本语言，运行在一个名为“jass虚拟机”的解释器上，因此jass2 ≈ java，jass2 ≈ C#，jass2 ≈ lua等等，这样是不是就好理解了？

为了理解这个外挂为什么要这么做，以及这么做能达到什么效果，我们来实际操作一下帮助大家理解一下。

首先新建一个地图，编写一个示例喊话call，其效果如下所示：

![](https://bbs.kanxue.com/upload/attach/201903/615441_GNN3FT368Q44D6H.png)

然后。解压出它的脚本文件`war3map.j`，为其添加，外挂作者的喊话call函数：

![](https://bbs.kanxue.com/upload/attach/201903/615441_56SB2B4RDMSVTX9.png)

![](https://bbs.kanxue.com/upload/attach/201903/615441_MN3RENMUPVYRJ59.png)

这里再讲到一个概念：就是触发器模式。

这个模式已经被广泛应用到unity、虚幻等引擎中，教科书般得阐述了“事件驱动”编程这个思想。

那么jass2中的一个触发器的结构是什么样子的？

它由三个部分组成，触发器变量`trigger xxx`，一个自定义条件或者动作函数，当然条件或者动作函数只要有一种就好，两者都有就是一个标准的触发器，当然如果都没有那么这个触发器就没有任何意义了。

那么它是如何工作的？如上图所示，首先要对触发器变量进行初始化，即`CreateTrigger()`，当然在其定义的时候也可以一次性初始化，可以少写一行代码。然后为其绑定条件和动作（这里不再展开其中的区别），最后在`main`函数对这个触发器进行“注册”，即在`main`函数中调用初始化函数。

到这里，相信你也应该明白了它的工作原理，可以做如下猜测`globals`区域以`endglobals`为结尾（对应`globals.txt`），从下一行开始为自定义函数区域（对应`functions.txt`），直到`main`函数为止，`main`函数类似于c语言中的`main`函数（对应`main.txt`）。

我们将修改过的`war3map.j`文件替换回原地图，其效果如下：

![](https://bbs.kanxue.com/upload/attach/201903/615441_QYDS7B5TQ9GX73M.png)

所以，`InsertCheatScriptToMapOrginJassScript`核心流程如下所示：

![](https://bbs.kanxue.com/upload/attach/201903/615441_H5XCA98DA7WNNU9.png)

怎么样，是不是能看懂了？

至此，这个外挂样本的功能已经基本逆向完毕。

现在回过头来说说它的“Anti检测功能”。我们知道Themida会处理几个函数，分别是：`DbgBreakPoint`，`DbgUiRemoteBreakin`，`DbgUserBreakPoint`，这个外挂处理的方法也很暴力，通过`GetProcAddress`获得其地址以后，直接恢复原来的字节试图达到恢复hook的目的。

![](https://bbs.kanxue.com/upload/attach/201903/615441_JYBEFGH7WE8YK5B.png)

注意，我说的是“试图”，你以为恢复了hook就能调试了？太年轻了，小伙子。

他还处理了硬断检测使用了`ClearDrRegisters`，不过，单hook不会触发这个检测，就不多讲了，流程就是暂停线程->获取线程上下文->清空调试寄存器->恢复线程。

至此，整个样本分析完毕。

## 0x2 检测

好了，知道它干了哪些坏事，我们就可以针对他进行一波检测。

1.还记得这句代码吗？

```
g_int_game_compline_jass_script = (int)(g_game_module_handle + 0x1FAB34);
```

我们可以检测这个hook点，当然，由于反作弊系统的加载要远早于外挂的加载，所以，在初始化的时候检测是不可行的，因此可以把这个检测的时机延后到`jass_vm_entry()`。这个时候外挂已经加载，且因为他没有清理和卸载工作，所以hook不会恢复，必定能检测到。

2.拦截模块加载，这里采用白名单模式，IAThook `mss32.LoadLibrary`来过滤魔兽的自加载插件，不在白名单内的全部砍了。

3.文件CRC，时机同样在`jass_vm_entry()`。这里的CRC有个技巧:先从服务器上得到地图j文件的md5保证其未被篡改，然后从地图中提取它并映射到内存，然后运算其CRC32值，然后与内存中已经加载的j文件的CRC32做比较，如不同，就判定为作弊。

4.函数列表检测，从jass虚拟机中取出所有的函数名称以及数量，与原始文件进行比较，如有差异则判定为作弊（当然这里要对dota特殊处理，将自己添加的函数也加入原文件的函数列表中，其数量亦增加相应数值。）

5.CallStack检测，栈帧回溯，没啥用，放弃这个方案。

## 0x2 总结

这个外挂样本为我们展现了静态替换的升级版本，动态替换，直接攻击了游戏的脚本编译函数，切中了反作弊系统的忙点，有一定的研究价值，也提醒我们在与外挂对抗的过程中，不要忽略任何一个可能被利用的点，道高一尺，魔高一丈，不能用老眼光、老思路看待作弊手段。

## 0x3 补充

那位说，“该编译函数一共只调用了4次，所以我们可以检测它的调用次数，如果超过4次就判定为作弊。”的同学，你的思路没错，就是没考虑完全。外挂加载的时机要远远晚于反作弊模块，且它是inlinehook不会额外增加引用计数、其次它的hook的会覆盖你的inlinehook，所以你就永远检测不到它，这也是这个样本为什么能在线上活跃两年之久的原因。希望你看完这篇文章以后能更新一下代码，让这款外挂从线上消失。

那么，大家一起加油吧！

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于 2019-3-6 12:35 被黑洛编辑，原因：

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)

上传的附件：

- [jasstool.rar](javascript:void(0);)

  （4.33MB，279次下载）
