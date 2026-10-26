
# [原创]利用U3D引擎特性快递定位数组

作为当代不争气的青年代表，在放弃挣扎前奋笔写下最后一篇文章，之后不问江湖事。

![](https://bbs.kanxue.com/upload/attach/202108/826671_3TMM3UXR5APBRXA.jpg)

`IL2CPP`我就不介绍了，显得水文字。搞过U3D的男人都懂，`IL2CPP`模式打包的游戏，游戏下面会存在一个名为`global-metadata.dat`的文件，这个东西解包之后会得到一个`dump.cs`和Mono模式下的所有DLL。那么综合网上大部分的教程来说，他们只会用`dump.cs`来看偏移，虽然直接用`dnspy`打开`dll`看也是可以的，但是`dump.cs`更偏向于sdk文件，可以直接把它的内容当作头文件来用，更加方便。接下来我将给大家带来一套组合拳，教大家如何正确的使用这个玩意来快速找到一个游戏的数组。这里插个题外话，标题虽然这次讲解的是针对`IL2CPP`,但是相对来说，Mono也是可以的，只不过在偏移和一些结构上面略显差异。

\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_

老早时期，精力旺盛，在一次乱翻内存之中发现了`u3d`的一个惊天大秘密，然后经过一系列的操作和研究后就总结出了一套自己独有的使用方式。是这样的，`u3d`里的对象，假设对象地址是`0x1000000000`,那么在这个对象的某一个地方是会存着这个对象的类名，并且寻址的偏移还是固定的，之前查了一会，这玩意好像叫做反射。那么，这个有什么用捏？我这里先模拟一下传统的找数组方法：**首先`CE`搜索人物的某项数据，比如坐标，然后一层一层往上跟到本人地址，然后通过访问本人地址或者扫指针，最后找出数组。**这些看起来都是非常常规的方法，但是如果放到`u3d`游戏上来说是非常恶心的。众所周知，`u3d`有用的偏移大部分都是10层起步，像这种偏移量，先不说你有没有那个耐心用`CE`跟出来，就算你扫指针，那也是吃不消的，况且你还要去做数据对比。所以说整个流程下来，即使你找到数组，那也是累得半死的。这个时候，类名就派上用场了。

![](https://bbs.kanxue.com/upload/attach/202108/826671_DXJYESQFRJTUTTZ.jpg)

我们知道凡是U3D打包的游戏，源代码基本都在`Assembly-CSharp.dll`这个`DLL`里，那么我们可不可以大胆猜测一下，**游戏有一个主管理器类，里面存放着很多数据，其中就包括玩家数组，模型数组之类的，然后我们通过用`DnSpy`这个工具来找到这个类在游戏的实际名字，最后通过搜索类名的方法，反推到对象名**。上面我提到过了，对象到类名的偏移是固定的，那么找到类的实例对象就相当于做小学加减法。存在既合理，下面开干。这里用的是**枪火重生**这个游戏，我们用`il2cppDumper`把`GM`文件解包之后用`DnSpy`打开`DLL`。

![](https://bbs.kanxue.com/upload/attach/202108/826671_7QQYM9E8K68MC3V.jpg)

既然是找有玩家数组的类，我们第一步就是去找玩家类，玩家类可以通过玩家的属性去找，常见的是血量，对应的关键词大部分是`HP,Health`，这个游戏用的是`HP`。

![](https://bbs.kanxue.com/upload/attach/202108/826671_PEJ5VK3RMPS959P.jpg)

**左边表示成员，右边表示所在的类**。放眼看去，只有`PlayerProp`符合我们的要求，我们双击后，左边的列表将会定位到该类，我们点击展开按钮查看内部的内容。

![](https://bbs.kanxue.com/upload/attach/202108/826671_4UJYKWEZMZ22NYV.jpg)

发现里面都是一些属性，从类名也可以判断出这个是存放玩家属性的类而不是玩家类，并且应该是属于玩家类的一个成员，所以我们只需要查找那个类引用了`PlayerProp`这个类。对这个类右键点分析，在右下角点开被使用。

![](https://bbs.kanxue.com/upload/attach/202108/826671_2Q5Y562FEXHWSPB.jpg)

根据类名可以发现有两个可能是玩家类，经过两者内容对比，最终确认`NewPlayerObject`为玩家类。

![](https://bbs.kanxue.com/upload/attach/202108/826671_E4S2RHTSHDBBQ4C.jpg)

![](https://bbs.kanxue.com/upload/attach/202108/826671_XDHE39CN9JEDF3E.jpg)

既然已经找到了玩家类，那么数找组就很容易了。同样的操作，对玩家类右键点分析，然后点击被使用，通过根据类名进行了一系列筛选后，判定`NewPlayerManager`为**主管理类，而且为静态类。**

![](https://bbs.kanxue.com/upload/attach/202108/826671_VSSNCPCY5DM8CUB.jpg)

![](https://bbs.kanxue.com/upload/attach/202108/YHP6WFHSKV24CQX.jpg)

并且在该类下面也发现了很多数组。

![](https://bbs.kanxue.com/upload/attach/202108/826671_VBR7HUNCJZVFYWD.jpg)

到这里，我们基本工作就已经完成了。到时候我们直接到游戏找到这个类然后通过`DnSpy`显示的偏移或`Dump.cs`显示的偏移拿到对应数组即可。

上面我提到过，U3D的对象都会在自身下的某的地方存着该对象的类型，而且该偏移是固定的。那么这个偏移怎么找？我也不知道。

![](https://bbs.kanxue.com/upload/attach/202108/826671_B95AFMKXNX7ABA9.jpg)

因为是当时偶然间在`CE`内存翻到的，然后去对比了其他游戏，发现偏移是固定的。目前只知道这个玩意叫做`runtimeType`.

![](https://bbs.kanxue.com/upload/attach/202108/826671_PAB3SYSCUQG2MPC.jpg)![](https://bbs.kanxue.com/upload/attach/202108/826671_AA7FYRXWC9U7U2C.jpg)

由于万物继承`object`，所以直接在头部翻就能找到了。

```
Il2cpp：

        obj+0x10]=className

        obj+0x18]=spaceName

Mono：

        obj+0x0]+0x48]=className

        obj+0x0]+0x50]=spaceName
```

这里说一个细节，在`il2cpp`中，`className`的内存属性是只读，Mono中是可读可写，然后就是`className`的后面紧接的字符串是该类的成员函数名和成员变量名，可以通过这个细节来筛选其他数据。

进入游戏**(这里建议一定要进入游戏，因为进入游戏后才会有对象，有了对象才能方便我们查看数组对象)**，打开`CE`搜索我们找到的类`NewPlayerManager`。

![](https://bbs.kanxue.com/upload/attach/202108/826671_HXJQTXMZTJ3P9CD.jpg)

这里只有一个结果，那么通过查看内存数据再对比`DnSpy`,发现该地址是正确的，然后开始做加减法。将该地址放到`CE`搜索**（记得恢复查找的内存属性）。**

![](https://bbs.kanxue.com/upload/attach/202108/826671_NV3HS9N69Z239CB.jpg)

发现这里有两个，这个时候不要慌，都拿下来减`0x10`.

![](https://bbs.kanxue.com/upload/attach/202108/826671_Q9YMERUVGEDGCW7.jpg)

第二个为`0`，`u3d`的对象头部不可能为`0`，这是经验之谈。这个时候`0x 2E99B5E6060`，实际上已经是`NewPlayerManager`的实例了，现在可以通过跟地址或者扫指针去找基地址，但是因为这个类是静态类所以这里直接搜索一下看看有没有基地址存着这个地址。有一个，那么这个就可以用了。

![](https://bbs.kanxue.com/upload/attach/202108/826671_6Q467BFNAGBCCMV.jpg)

数据都找好了，现在就来看看数组吧，我们打开`dump.cs`，然后讲这个地址添加到`ce`的数据结构工具。

![](https://bbs.kanxue.com/upload/attach/202108/826671_MSGSE26DYD8Y7H5.jpg)

此时会存在一个问题，`dump`中显示`0x10`是一个字典，但是当先的`0x10`却是类名，那这就很尴尬。。这里在说一个点，也算是经验之谈。当前我们这个地址`0x 2E99B5E6060，`只能算是一个假体，真正实体在`0xB8`的位置，至于为什么，我也不懂，分析多了，见多了就默认了。我们进入`0xB8`的位置.

![](https://bbs.kanxue.com/upload/attach/202108/826671_363EA8CXRSSY9AB.jpg)

发现现在数据相似度已经很高了。我们直接去看看怪物的数组**（`0x48`）**

![](https://bbs.kanxue.com/upload/attach/202108/826671_KCPW25Z3Z84AMCD.jpg)

进去后`0x10`是数组指针，`0x18`为数量，这是`List`固定的，然后进入`0x10`。

![](https://bbs.kanxue.com/upload/attach/202108/826671_NFVRWBHEQTHVTSV.jpg)

同样数组是从`0x20`开始的，也是固定的。这些其实不是说我在瞎说，因为U3D的`List`他内存结构就是这样的。![](https://bbs.kanxue.com/upload/attach/202108/826671_Z5RZUUMFN3JPTA2.jpg)

当前敌人也只有一个，然后`0x20`的指针，类名也是`NewPlayerObject`。

![](https://bbs.kanxue.com/upload/attach/202108/826671_BS62N7FZQNPHMAV.jpg)

然后剩下的，就是查看`dump.cs`去拿到对应偏移即可。

## 补充一下

在`NewPlayerManager`类中由于该类直接是继承`object`，所以该类的`0x10`直接是类名。

![](https://bbs.kanxue.com/upload/attach/202108/826671_AX8GTWMG65MRHQ4.jpg)

但是像`NewPlayerObject`类基类并不是`object`，所以需要在`0x0`的地方读一层之后`0x10`的位置才是类名。

![](https://bbs.kanxue.com/upload/attach/202108/826671_JBHN4GBXCRGJ74Q.jpg)

因为是文字叙述，无法将所有细节给描述出来。所以大部分东西直接一笔带过，至于为什么，多去实践就知道了。这次针对的是`il2cpp`，所以在其他也是`il2cpp`的游戏中可能操作起来的相似度较高，Mono上可能有一些差异，但并不碍事，我只是想说有那么个方法可以快速拿到数据而已，至于能不能做到看自己的造化。文章中难免出现先一些错误，各位就留点情面吧。

这大概也是我最后一次写这种类型的技术文了。。留个纪念吧。

![](https://bbs.kanxue.com/upload/attach/202108/826671_DVUPXJS793U6AHN.jpg)

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于  2021-8-18 15:13

被PlaneJun编辑

，原因：

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)

[# .NET平台](https://bbs.kanxue.com/forum-4-1-7.htm)
