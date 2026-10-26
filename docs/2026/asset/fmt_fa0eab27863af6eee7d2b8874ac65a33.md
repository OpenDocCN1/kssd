
**17.然后再快速单步很快就能发现刚才所下的第二个断点，发现`rand`下面有一个`memcpy`拷贝内存的函数，走一遍注意观察`OD`中右上角寄存器的变化，我们刚刚看到这个函数里面，是将`ECX`传递给了`ESI`,而`0x41CAFC`地址处对`ESI`进行了访问，走一步之后我们发现`EAX`的值是`0x12BB50`,那么选中它，右键->数据窗口跟随 ，查看`0x13BB50`地址里面有些什么数据。**

![](https://bbs.kanxue.com/upload/attach/201911/850153_8589WQGMJECDHQM.png)

![](https://bbs.kanxue.com/upload/attach/201911/850153_G5HBVUEEJKNDGYE.png)

**18.然后单步走到`memcpy`下一步处，观察`0x12BB50`地址里的数据，多来几次，发现除了前8个字节没变，后面都被填充成某种规律的010100...猜测`0x12BB50`可能是游戏的地图数组基地址，先不管，继续单步，当走到`0x41CB15`时，`0x12BB50`处数据又被刷新了。所以`0x12BB50`极有可能是我们要找的地图数组基地址。**

![](https://bbs.kanxue.com/upload/attach/201911/850153_DTYNZ6V9KEE448C.png)

![](https://bbs.kanxue.com/upload/attach/201911/850153_FVT3P3GT3CJMMK5.png)

**19.为了验证是否正确，按`F9`让游戏运行起来，观察地图与该内存数据是否有联系，多来几次，发现地图和`OD`中的内存非常相似。**

![](https://bbs.kanxue.com/upload/attach/201911/850153_85WE94YKJ3BG8YR.png)

**20.为了再次确认，我们把游戏中的物品点击消除掉在观察内存发现被清0了，那么我们就可以确定`0x12BB50`就是地图数组基地址了，并且得到了`0x12BB58`就是地图数据的起始位置了。**

![](https://bbs.kanxue.com/upload/attach/201911/850153_V955TW5RGVEZ77A.png)

![](https://bbs.kanxue.com/upload/attach/201911/850153_TN5NUPB3XRQKK58.png)

**21.现在，我们来捋一捋思路，先将地图数组变成有规律的0与1，然后再将1上的位置随机分配各种代表图案的数值，而前一步并没有循环随机函数，只随机了一次，大胆假设它应该存在一个保存各个地图基本形状的数组，随机函数是随机使用那个地图，经过分析我在`rand`函数上面最近的一个`CALL`证实了我这个假设**

![](https://bbs.kanxue.com/upload/attach/201911/850153_8Y9JA6TE3EMDQD6.png)

**`22.localkyodaimap.map`就保存着我们所有地图的基本形状（有图案还是没有图案），至于是什么图案，是在`0x41CB15`地址处的`CALL`里面随机分配的，这里就不细说了**

![](https://bbs.kanxue.com/upload/attach/201911/850153_QA7G7DMZ3DNVWGG.png)

![](https://bbs.kanxue.com/upload/attach/201911/850153_SEY7PV7T23ZVVWR.png)

![](https://bbs.kanxue.com/upload/attach/201911/850153_EPJ7QJFFHS2NNRH.png)

**23.接下来就是分析道具了，先从指南针开始。指南针道具是帮助玩家找到可以消除的2个相同图案，而要实现其功能必须要访问地图数组，所以我们可以在`0x12BB58`地址处下一个内存访问断点。**

![](https://bbs.kanxue.com/upload/attach/201911/850153_FDBKVHVZ7HNA82B.png)

**24.先在`OD`中按`ALT+B`来到断点模块，把其它断点禁止或者删掉，然后运行游戏，点击指南针，程序就会停下来，点击`K`进行栈回溯分析。发现有5个上层调用，在5个位置处都设置上断点，把刚刚设置的内存访问断点删除掉。**

![](https://bbs.kanxue.com/upload/attach/201911/850153_WJ34Z268RPBV3HT.png)

**25.再按`F9`运行`OD`发现运行多次都停在`0x4292A5`处，那么这肯定不是我们要找的函数，去掉其断点。再运行，发现只要点击游戏界面，就会在`0x40CACA`处断下，所以这个也不是我们要找的，去掉其断点，继续运行，同样的`0x41AF11`只要点击游戏界面，就会在该地址处断下，`pass`掉，再按`F9`运行起来。**

**剩下的`0x41E76C`与`0x41DE5C`处的函数可能是我们要找的关键函数，现在想要写代码完成指南针的功能，只需要找到这两处地址的函数调用时所需要的参数，那就可以模拟出指南针的功能。**

**重新运行起来，点击指南针，会在`0x41DE5C`处断下，选中这一行按`Enter`键进入这个函数一直往下拉找到末尾`RETN`观察发现`RETN 0xC`也就是说`0x41DE5C`处的函数调用需要3个参数（当然这里不考虑通过`ECX`寄存器传递的`this`指针），通过观察堆栈得出参数是`0`，`0`，`F0`。**

![](https://bbs.kanxue.com/upload/attach/201911/850153_Z6YW27D3XERU62N.png)

![](https://bbs.kanxue.com/upload/attach/201911/850153_PZPPH6DVTDQ7VD8.png)

**26.为了确认这3个参数是不是可变的，多操作几次，发现参数没有改变，那现在就只要确认`ECX`的值，就可以用这个函数模拟指南针功能了。暂时先不管它，那么我们再次运行，使其段在`0x41E76C`处，按照上面分析方法，这个函数有2个参数，是两个地址，数据窗口跟随，然后单步，看看两处地址的数据有啥变化。**

![](https://bbs.kanxue.com/upload/attach/201911/850153_QSZ8NVZX97M2TJ7.png)

**27.指南针的作用是找出两个可以相消除的图案，也就是要找到这两个图案的坐标，通过观察发现这个`CALL`有可能是获取两个可以相消除的图案的坐标。然后我们多次将内存中的数值与找到的两个图案相比较，证实了此猜想成立。**

![](https://bbs.kanxue.com/upload/attach/201911/850153_BJ86YHG2XHDBA8G.png)

![](https://bbs.kanxue.com/upload/attach/201911/850153_7QJE4YP24DHBQN9.png)

**28.所以现在只剩下我们刚刚找到的处于`0x41DE5C`的`CALL`,只要找到`ECX`的值就可以了。往上找`ECX`的值，需要往上层函数分析，这里我们换一种方法分析，使用`CE`附加程序，搜索`ESI`的值，发现有几个绿色的基址。**

![](https://bbs.kanxue.com/upload/attach/201911/850153_CAB8F8T5X2KAFM6.png)

![](https://bbs.kanxue.com/upload/attach/201911/850153_PKQJFV5NXTGKZXM.png)

**29.我们把这几个基址在`OD`反汇编窗口右键->查找->所有常量，排除后还剩下`0x45DEBC`与`0x7793CDD8`,这里先用`0x45DEBC`试试。**

![](https://bbs.kanxue.com/upload/attach/201911/850153_S6KKQSZ9YYWMQDV.png)

![](https://bbs.kanxue.com/upload/attach/201911/850153_K8FDMK6ZG2RWMEG.png)

**30.相同的原理我们找出其它道具的调用`CALL`,这里就不细说了。我们发现，都会调用`0x41DE5C`处的`CALL`,只是参数不同，对比下面参数**

**调用指南针：`0`，`0`，`F0`**

**调用炸弹：`0`，`0`，`F4`**

**调用重列：`0`，`0`，`F1`**

**得出结论第3个参数是道具类型，而`0x41DE5C`处的函数功能应该是使用道具。**

![](https://bbs.kanxue.com/upload/attach/201911/850153_DA268QH3CMHCZW5.png)

![](https://bbs.kanxue.com/upload/attach/201911/850153_EMVD8PVBW9XCCVE.png)

**31.找消除`CALL`,要消除，肯定要对地图数组改写，所以我们可以在相应位置设置内存写入断点，然后点击这两个位置消除，`OD`就会断下，取消内存写入断点，然后通过栈回溯分析。**

![](https://bbs.kanxue.com/upload/attach/201911/850153_VERSHQNR2Z7XK4X.png)

![](https://bbs.kanxue.com/upload/attach/201911/850153_68X7P53Y7MK6DMK.png)

**32.通过工具栏上的`K`跳转到栈回溯窗口，发现有5个，按照之前的套路进行分析。最后还有3个`CALL`有可能是我们要找的，通过分析，发现第一个`CALL`有7个参数，有点多，而且其中有参数的值不确定，`pass`,第三个`CALL`没有传坐标，应该不是，也`pass`,那么我们就只剩下第2个`CALL`了，它有6个参数，经过分析，第一个参数为0，第二个参数为地图数组基址（这个前面已经找到），第3，4个参数为两个坐标，我们可以通过上面指南针中找到的获取两个可以消除的点的函数获得。**

![](https://bbs.kanxue.com/upload/attach/201911/850153_76P7MKQXKW2Y22U.png)

![](https://bbs.kanxue.com/upload/attach/201911/850153_Z73ERK8RRBCRPP9.png)

![](https://bbs.kanxue.com/upload/attach/201911/850153_K5243N8CDJP8T42.png)

![](https://bbs.kanxue.com/upload/attach/201911/850153_U47R946EB2S3DRH.png)

**33.往上找到（这里需要往上一层找，第5，6参数是上层函数的参数），第5，6个参数的值是在这赋值的，`ECX`是`ESI+0x1E84`的值，而`ESI`和上面我们找到的`ESI`一样，这就不说了。**

![](https://bbs.kanxue.com/upload/attach/201911/850153_YUAQD72RDD425V2.png)

![](https://bbs.kanxue.com/upload/attach/201911/850153_UR46NR28AT5A7EY.png)

## 编写辅助工具：

**下面就是编写辅助了，单消只需要调用获取两个能相消的坐标的函数，再调用消除`CALL`就可以了,当然使用炸弹道具也可以完成，秒杀就是循环使用单消就行，这里有点，就是如果没有能够相消除的两个点的话，是会返回两个（0，0）坐标，所以，可以根据此退出循环。**

**1.用`VS2017`首先创建一个`MFC`的`DLL`程序，选择在静态库中使用`MFC`,这样兼容性更好，这个选项也可以在属性->常规->`MFC`的使用里面更改。**

![](https://bbs.kanxue.com/upload/attach/201911/850153_R55RKKZZ4BDYVJV.png)

![](https://bbs.kanxue.com/upload/attach/201911/850153_8E5R895MZ7VZDC3.png)

![](https://bbs.kanxue.com/upload/attach/201911/850153_H3DQ227AW6HTCPS.png)

**2.通过`Spy++`获得游戏的窗口名与类名，用于`FindWindow`函数获取游戏窗口句柄。**

![](https://bbs.kanxue.com/upload/attach/202002/850153_SU7QNJ5RC6WAVF3.png)

**3.使用`__asm`内联汇编调用我们找到的关键函数，说明下，消息采用自定义消息。**
