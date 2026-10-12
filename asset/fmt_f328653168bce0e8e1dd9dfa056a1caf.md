
# Ghost 多媒体视频点播系统

> 原文：[`www.pediy.com/kssd/pediy05/pediy50213.htm`](https://www.pediy.com/kssd/pediy05/pediy50213.htm)

**文章题目**：破解 `Ghost` 多媒体视频点播系统全过程

**撰写**：仙剑太郎

**网页**：`http://cnokweb.yeah.net`

**声明**：若需转载本文，请保持文章的完整性。商业用途请联系仙剑太郎（`dqjun@21cn.com`）。

今天收到一位网友的 `QQ` 消息，要我破解 `Ghost` 多媒体视频点播系统 `V1.0`，还说这个 `2M` 多的程序（包括运行库和控件）的注册费竟索要 `2000` 大洋！！！我晕~~立即 `down` 下来看个究竟，安装后一看，我倒！！！什么点播系统嘛，其实就是个 `HTTP` 服务器，再搞点 `HTML` 自动生成罢了，刚学编程的也能做出这样的东东！竟然要我们给他`￥2000`注册费，真是太过份了！不行，一定要搞定它，好出出气！（我也很体谅作者的辛苦成果，不过`￥2000`的巨额注册费实在有点让人费解，这其实只是个几百`K`的软件嘛，我以前做的软件一个也不要钱的！）

好了，废话不说太多，下面开始啦！

**准备工具**：`SmartCheck 6.03`、`Ghost` 多媒体视频点播系统（下载地址：`http://download.tzinfo.net/it/2002418185913b.exe`，`2,867KB`，安装完后它会提示重启，但可以不重启直接运行）

开工啰！老方法，首先用`FileInfo`查看一下`GhostVOD.exe`的壳和编译器的相关信息，唔，很好，没有加壳，用`VB6`写的，这里大伙儿肯定想到了调试`VB`的老大哥`SmartCheck`了吧！因为`VB`是解释执行的语言，用普通反汇编和动态调试的方法难以应付复杂算法的`VB`程序，不说了，这些东西不是主要的，我们先要设置好`SmartCheck`，照看雪教程的那篇关于`SmartCheck`的文章设置，然后我们打开`Ghost`多媒体视频点播系统的主程序`GhostVOD.exe`，然后点击`start`或按`F5`开始载入程序，稍等片刻，主界面弹出了，点击“帮助”->“注册”，然后随便输入用户`ID`和序列号，例如`ID`为`cnokweb.yeah.net`序列号为`78787878`。接着点击“确定”，提示说“序列号非法！”，理解一下这句话，会不会序列号有一定格式顺序的呢？带着这个疑问，我们回到`SmartCheck`界面，下拉滚动条，展开`\_Click`，我们再展开下一层的`\_Click`，可以看到：

```
\_Click
    Trim
    Mid$
    Mid$
    MsgBox
```

其中第一个`Mid$`的内容为：

```
String  str = 004B0D4C
      = "78787878"
Long  start = 10 0x0000000A
length (variant)
    Integer  .iVal = 1 0x0001
```

表示从字符串“`78787878`”中取第 `10` 位起的一个字符。

第二个`Mid$`内容为：

```
String  str = 004AC5E8
      = "78787878"
Long  start = 5 0x00000005
length (variant)
    Integer  .iVal = 1 0x0001
```

表示从字符串“`78787878`”中取第 `5` 位起的一个字符。

看到这里我们要想想为什么要取 `5` 位和 `10` 的那个字符呢？接下来我们要解开迷底了，先把光标停在`MsgBox`里，然后我们点击`SmartCheck`的“View”->“Show All Events”显示所有事件。稍等片刻，这时多了很多信息，我们从`MsgBox`向上看，来到下面时，你在右边窗口看到了什么？

```
      Mid$
      \_\_vbaStrMove returns DWORD:4B427C
      \_\_vbaStrCmp returns DWORD:FFFFFFFF  //就是这里
      Mid$
      \_\_vbaStrMove returns DWORD:4976D4
      \_\_vbaStrCmp returns DWORD:1        //还有这里
      \_\_vbaFreeStrList returns DWORD:1C
      \_\_vbaFreeObjList
      \_\_vbaVarDup returns DWORD:6BE990
      MsgBox
```

我们点击上面的`\_\_vbaStrCmp returns DWORD:FFFFFFFF`和`\_\_vbaStrCmp returns DWORD:1`，我们可以在右边看到类似这样的信息：

```
 unsigned short \* string1 = 0040BF8C
      = "-"
 unsigned short \* string2 = 004976D4
      = "7"
```

这下明白了吗，原来前面所说的第 `5` 和 `10` 个字符原来是判别是否是“-”的，明白了这点，我们来研究一下序列号的排列顺序：

程序判断第 `5` 位为“-”，第 `10` 位为“-”那么中间必然是“`xxxx-xxxx-`”，不过看起来好象不怎么顺眼，因为一般来说注册码都不会以“-”结尾，到这里我们可以想到后面应该还有一串数字，会不会和前面两个一样是四位的呢？即“`xxxx-xxxx-xxxx`”呢？我们随便编造一个可以试一试，用户`ID`依然是填`cnokweb.yeah.net`序列号改填：`abcd-0123-4567`，点确定，提示“序列号非法”。看到这里我们想到了什么？序列号格式依然不对耶，回到`SmartCheck`，我们看到了和上面不同的情况：

```
            \_Click
                Trim
                Mid$
                Mid$
                OnError
                Len returns LONG:14  //多了这一句
                MsgBox returns Integer:1
```

点击上面多了的那一句，看看右边窗口出现什么？出现了我们刚才输入的序列号！那一句的意思为计算我们输入序列号的字符数。发现问题了吧，原来是要检测序列号长度的。我们要猜想一下究竟多少个字符才附合序列号格式，我们先在序列号后多加一位数，变为“`xxxx-xxxx-xxxxx`”。我们再试一试，在序列号里填入`abcd-0123-45678`，点击确定，提示说“用户`ID`与序列号非法！”，咦，这次怎么不同了！`Yeah`！我们之前猜的序列号格式对了，现在只是`ID`与序列号不匹配，也就是进入比较阶段了，注册码离我们不远了。。。。

`OK`，现在回到`SmartCheck`里，和上面一样展开`\_Click`，这次我们看到的信息比之前多了吧，下面一句句解释，让我们完全了解作者的注册码算法：

```
            \_Click
                Trim
                Mid$                          //判断第 10 位是否为“-”
                Mid$                          //判断第 5 位是否为“-”
                OnError
                Len returns LONG:15           //计算输入序列号长度，这里返回值为 15
                Right                         //取序列号最后 5 位
                Sqr returns double:213.724 (displayed as single-precision floating point)
                                             //上面一句是将最后 5 位转换成双精度浮点型数据
                Double (1.82461e+007) --> Long (18246056) //将双精度型+7 后转为长整型
                Long (18246056) --> String ("18246056")   //再转成字符串，这时我们得到了一个字符串“18246056”，这和注册码有什么关系呢？继续看下去。
                Len returns LONG:8            //计算“18246056”长度为 8
                Len returns LONG:8            //同上
                Long (8) --> Integer (8)      //将上面的长度 8 转为整数
                Mid$                          //取“18246056”第 8 位的 1 个字符，即取 6
                String ("6") --> Integer (6)
                Mid$                          //取“18246056”第 7 位的 1 个字符，即取 5
                String ("5") --> Integer (5)
                Mid$                          //取“18246056”第 6 位的 1 个字符，即取 0
                String ("0") --> Integer (0)
                Mid$                          //....同上类推....
                String ("6") --> Integer (6)
                Mid$                          //....同上类推....
                String ("4") --> Integer (4)
                Mid$                          //....同上类推....
                String ("2") --> Integer (2)   //....同上类推....
                Mid$
                String ("8") --> Integer (8)
                Mid$                          //...同上类推...直到最后一位“1”
                String ("1") --> Integer (1)
                Integer (5) --> String ("5")   //为什么有“5”呢？
```

看懂上面的序列号计算过程了吗？其实算法非常的简单，不过有一点不知大家有没有留意，一般软件计算注册码都是通过用户名计算注册码，然后和你输入的注册码比较，相等则注册码成功。不过现在从上面看来，它并没有用到我们输入的用户`ID`：`cnokweb.yeah.net`，哈哈，好奇怪吧。其实这和普通注册码算法差不多，只是它倒过来罢了，这里是先通过运算用户输入的序列号，然后与用户输入的`ID`作比较。现在想明白了吗，这就是说，上面那段字符串“`18246056`”有可能是通过我们编造出来的“假”序列号运算出来的用户`ID`！不过先别开心，我们还没完全找到注册码（其实是用户`ID`），不信你把它用作用户`ID`注册，也会告诉你非法。现在要深入想一想了，上面为什么会出现一个“`5`”呢？为了详细了解，我们点击`SmartCheck`的“View”->“Show All Events”显示所有事件。稍等片刻，我们由`MsgBox`向上，来到如下地方：

```
                Integer (5) --> String ("5")            //神秘的“5”
                \_\_vbaStrMove returns DWORD:4B427C
                \_\_vbaStrCat returns DWORD:4976D4       //关键！
                \_\_vbaStrMove returns DWORD:4976D4
                \_\_vbaStrCopy returns DWORD:4AC5C0
                      ...........省略............
                MultiByteToWideChar returns INT:17
                SysAllocStringLen returns LPVOID:4CB10C
                MultiByteToWideChar returns INT:17
                HeapFree returns BOOL:1
                \_\_vbaStrCmp returns DWORD:FFFFFFFF       //用户 ID 的比较就在这儿！
                \_\_vbaFreeStr returns DWORD:30
                \_\_vbaFreeObj
                \_\_vbaVarDup returns DWORD:6BE990
                MsgBox
```

上面出现神秘的“`5`”之后，出现一系列字符串的操作，这里应该内有乾坤了。我们来到`\_\_vbaStrCat returns DWORD:4976D4`时，右边窗口出现如下：

```
    unsigned short \* string1 = 004B427C
          = "5"
    unsigned short \* string2 = 004B0D4C
          = "18246056"
```

意思是说，把“`5`”与“`18246056`”连接起来，再到下一行`\_\_vbaStrCopy returns DWORD:4AC5C0`就可以看到连接后变成的字符串：“`182460565`”，继续往下看，我们到了`\_\_vbaStrCmp returns DWORD:FFFFFFFF`关键比较的地方，在右边窗口我们也看到它和我们之前输入的用户`ID`：`cnokweb.yeah.net`进行比较：

```
    unsigned short \* string1 = 004CB10C
          = "cnokweb.yeah.net"
    unsigned short \* string2 = 004AC5C0
          = "182460565"
```

到这里，我们终于弄出序列了，序列号就是我们前面所编造的 `15` 位数：`abcd-0123-45678`，用户`ID`就是上面那串数字：`182460565`，好了，终于搞定了！可以喝杯茶，坐下来，体息一下了~~

虽然上面转了很多弯，而且有些是靠猜的，做`Cracker`就要有这样的不怕错误精神，错了至多再来一次！现在要坐下来总结一下了，这个程序应该说是比较新鲜类型的倒转比较型，其实最终还是一样的。程序的算法也很简单，流程用`VB`代码表示大致如下：

`text1.text`为用户`ID`，`text2.text`为序列号：

```
if mid(text2.text,10,1)="-" and mid(text2.text,5,1)="-" then
    if len(text2.text)=15 then
        sn=right(text2.text,5)
        **转换 sn 为双精度+7，最后转为 string
        num=len(sn)
        **中间一连串的字符转移过程（好象没什么用）
        x=cstr(5)  神秘的 5，也许是其它，如 6
        **把 sn 与 5 或 6 等连接起来，形成用户 ID
        if sn=text1.text then
            msgbox "注册成功！"
        else
            msgbox "用户 ID 与序列号非法！"
    else
        msgbox "序列号非法！"
    end if
    msgbox "序列号非法！"
end if
```

总的来说，我们自己编造的序列号前 `10` 位除了“-”外，其余可以是任意英文或数字，最后 `5` 位才与用户`ID`有直接关系。另外，那神秘的`5`我现在也搞不懂是怎么冒出来的，有时不是`5`而是`6`等等，哪位能回答这个疑问？
