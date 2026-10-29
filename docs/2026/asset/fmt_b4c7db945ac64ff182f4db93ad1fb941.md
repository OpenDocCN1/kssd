
- 那我们回过头继续看看，看看是aes的input又在是啥
- ase的input
- ![img_34.png](upload/attach/202312/93_G4979Y986ZM5GF4.png)
- 在这里有被传入！
- ![img_35.png](upload/attach/202312/93_BNQ7JVZ8UY8Z7KB.png)
- 看着像是sha512（bytes）+bytes的拼接！

### 中间段
- ![img_36.png](upload/attach/202312/93_A5H5RVJMCWJ4PV3.png)
- 恰好我们的中间段也有在别的地方引用诶
- ![img_37.png](upload/attach/202312/93_ZW3MBV6J2EYQ8EM.png)
- 刚好是在这里随机生成的诶！

#### 到这里我们就剩下最后一个问题了了，aes的key/iv如何生成？
- ![img_38.png](upload/attach/202312/93_3GB5TTZTFV6PDCP.png)
- 也是有引用的诶！！！看看在哪！
- ![img_39.png](upload/attach/202312/93_XMKCRC78V653224.png)
- 是sha512的值诶
- 那看看sha512的输入是啥
- ![img_40.png](upload/attach/202312/93_ZSEZDMNJJQ899DZ.png)
- 又有引用诶，再看看这个引用又是啥？
- ![img_41.png](upload/attach/202312/93_T6W8GTAPM32A4YU.png)
- 看着这个： 5C A5 E7 F7 3C 2B 01 CE CA BB DE C0 5C 8E A2 B9有点熟悉，是不是刚才出现过的！？？？
- 正是上面的随机数值诶！
- 那生成的还有一半的值呢？
- ![img_42.png](upload/attach/202312/93_638YSUZSHN9RJPY.png)

```text
0040: 4D D4 C2 E6 B8 31 62 09 0E 52 B3 C7 A6 73 3B A4 M....1b..R...s;.
0050: 1C B2 46 2B 82 9A B5 8A 19 6B 39 DB 57 17 75 24 ..F+.....k9.W.u$
0060: F4 9B AF 7F 08 E8 D6 8D 26 A7 2E 37 C1 A9 5A 2F ........&..7..Z/
0070: 1F 05 A5 18 92 AE F2 94 97 32 B6 2A 38 AA DD 58 .........2.\*8..X
```
- 经过多次的测试，发现是固定值！

## 最后总结![img.png](upload/attach/202312/93_S4V3FNZ2JJR87ZQ.png)
- 分析到这里基本整个流程都分析完毕了。
- 最终的逻辑代码我也没写，我也不会写。因为我没有这个业务的需求，如果需要的话，那么对着写就完事了，毕竟整个流程也是一目了然了。分析花了四五个小时，把整个过程水一遍写成文章也花费了四个小时。截至现在已经是2023年12月20日04点01分了。
- 害。狗命重要。明天再发上去了。。。
- 另外大佬们有好玩的类抽取壳，可以发样本给小弟我玩玩，最近捣鼓了一小玩具，测试的样本也有好几十个了，想继续完善一下。希望大佬们可以发一些有趣的类抽取样本，特别是那种不能回填到`code_item`上去的那种！
- 如果哪里写的不对，请大佬们多多赐教了，谢谢啦

## 2023年12月20日10点34分更新
- 通过大佬的反馈，那并不是固定值的！而是我们漏掉的！之所以为固定的是因为我们输入的固定！通过日志我们可以发现我们还少处理一个方法！这里备注为：查表的那个就是了~
- 那就把这个漏网之鱼抓捕回来看看它究竟是长的咋样子呢！

```java
debugger.addBreakPoint(module, 0x4500);
```
- ![img_1.png](upload/attach/202312/93_7ZWCDQUQAT92Q4B.png)
- 看着也像是一个指针，那就看看指针指向的内容吧！
- ![img.png](upload/attach/202312/93_8YQ7QJ8PUAK6VAT.png)
- 好像有引用，那就看看他在哪里被用到了
- ![img_2.png](upload/attach/202312/93_AEHD38HN9YRZ8JK.png)
- sha512的结果！
- 那就看看sha512的输入文本是啥呢？
- ![img_3.png](upload/attach/202312/93_M967AAZK3VE6GQ2.png)
- 嘿嘿又有引用
- ![img_4.png](upload/attach/202312/93_MUTS3TSAK7YBXVP.png)
- 还是随机值，那我们就知道他的输入为随机值的sha512的结果了
- 就简单分析一下这个方法，先看传递的参数只有一个result，然后又从result中获取三个值，那么这三个值就是：

```text
0-->80 60 35 40
1-->00 00 00 00
2-->40 00 00 00
v1 = result[2];
v2 = \*result;
v3 = result[1];
就可以直到v1=0x40，v2为0x40356080，v3为0
v1 =
```
- ![img_5.png](upload/attach/202312/93_WSG4Q65S75MY9JR.png)
- 这样子看的话，第一个if if ( v1 )是true的，第二个if ( v1 < 0x20 )就为false了！
- 再往下看只有这里对地址写入内容了，那我们调试一下看看？
- ![img_6.png](upload/attach/202312/93_F9KCWRTX3UT8GUD.png)
- 汇编对应的就是
- ![img_7.png](upload/attach/202312/93_FKDAUEY9RCP7GAV.png)
- 那我们下一个断点看一下！

```shell
b0x4560
```
- ![img_8.png](upload/attach/202312/93_7WYR5XFQMG59XV4.png)

```text
>>> q0=0xa43b73a6c7b3520e096231b8e6c2d44d(1.8056244479338115E-263, -3.7768713601031674E-134) q1=0x24751757db396b198ab59a822b46b21c(-4.496232995265719E-257, 4.642831181428702E-133)
```
- 这两个q0跟q1是什么鬼，那我们咦好像就是我们上面所说的定值的了诶~
- 那再执行一下看看？
- ![img_9.png](upload/attach/202312/93_93WCSV2HHDZAEGC.png)

```text
q0=0x2f5aa9c1372ea7268dd6e8087faf9bf4(-5.367597597728704E-242, 1.4054386263309782E-80) q1=0x58ddaa382ab6329794f2ae9218a5051f(-9.09210480120694E-208, 1.1969099196948154E120)
```
- 让我们执行到底再看看？再结合一下我们上面所说的定值

```text
0040: 4D D4 C2 E6 B8 31 62 09 0E 52 B3 C7 A6 73 3B A4 M....1b..R...s;.
0050: 1C B2 46 2B 82 9A B5 8A 19 6B 39 DB 57 17 75 24 ..F+.....k9.W.u$
0060: F4 9B AF 7F 08 E8 D6 8D 26 A7 2E 37 C1 A9 5A 2F ........&..7..Z/
0070: 1F 05 A5 18 92 AE F2 94 97 32 B6 2A 38 AA DD 58 .........2.\*8..X
```
- 好像就完全的对的上了诶
- 嘿嘿，其实就是在这里来的。。。成功把这条漏网之鱼给补抓上了啦

## 捣鼓背景
## 其实就是今晚无聊整个乐子打发一下时间了吧？？？
- 
- 
- 害，真实的原因是发现身边的大佬不是已经分析完某x了，就是在分析的路上了。再不跟上大佬的步伐，估计以后车尾灯都看不到了，再想抱紧大佬大腿当个挂件都难了，
- 既然是刚开始分析的，那就找点简单的吧！
- 大佬们自行跳过
- 大佬们自行跳过
- 大佬们自行跳过

## 因为这是今晚无聊分析的一个小样本，大佬们自行跳过就好啦！
- 
- 

## 分析总结
花了几个小时快速分析的，所以并未能的完全的分析他的整个流程。只能是通过一些快速分析的手段进行快速的定位以及验证！
到最后才发现他的整个基本都是这个样子比如一开始的`JNI_OnLoad`方法，有空还想把那个指令的执行也分析一下，看一下他是如何入参出参以及执行方法！
看完整篇文章可以收获为：
- unidbg的简单调试
- so的方法简单修正
- aes的key初始化流程
然后大佬们就可以判断是否有必要继续看下去了

## 目标选取
既然都是用unidbg分析，，，那就肯定绕不开凯神的小demo了
那就是：
- [unidbg-android/src/test/java/com/bytedance/frameworks/core/encrypt/TTEncrypt.java](https://bbs.kanxue.com/elink@91aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6*7K9r3E0D9x3o6t1J5z5q4)9J5c8Y4g2F1K9h3c8T1k6#2)9J5c8X3u0D9L8$3u0Q4x3V1k6E0j5i4y4@1k6i4u0Q4x3V1k6#2L8X3W2V1j5X3N6Q4x3X3c8S2L8X3c8J5L8$3W2V1i4K6u0r3M7%4u0U0i4K6u0r3N6r3g2K6N6q4)9J5c8X3A6S2N6X3q4Q4x3V1k6U0L8$3#2Q4x3V1k6T1P5i4c8W2k6r3q4F1j5$3g2Q4x3V1k6X3M7X3q4E0k6i4N6G2M7X3E0K6i4K6u0r3j5$3!0J5k6g2)9J5c8X3g2F1j5%4u0&6M7s2c8Q4x3V1k6f1g2p5g2F1j5%4u0&6M7s2c8Q4x3X3g2B7j5i4k6S2)
刚入门捣鼓的时候，学习unidbg的各种hook脚本的时候，就是通过捣鼓这几个小demo进行学习的了，那这次就当作交了拖了很久很久的作业了
既然目标已经确定了，那就开始吧！

## 开发环境
- 开发环境：Unidbg，对，就这个就够了
- 电脑环境：Windows11
- app板本：26.1.0（是大佬发给我的，所以没得下载链接）

## 模拟执行
既然是交作业第一步就是先把凯神的复制过来改改先，然后把apk配置为我们目标的板本！
由于分析的版本是64的，加载方式是通过apk加载的，所以第一步将`for32Bit()`改成`for64Bit()`；然后再向`createDalvikVM`传入apk的当前路径；接着修改一下`loadLibrary`的传递参数为：`ttEncrypt`，顺便将第二个参数设置为：`true`；让他执行一下so的初始化方法！
他的有一些hook的demo跟ida pro调试代码我们目前用不着，那就先删除！
改完就成这个样子了！
- ![](https://bbs.kanxue.com/upload/attach/202312/93_2JRC2E3FESW992K.png)
- 然后跑起来！
- ![](https://bbs.kanxue.com/upload/attach/202312/93_M3FV27PHS69ZX26.png)
- 紧接着就翻车了~~害，按照提示我们去看一下apk中是否存在这个依赖了~~
- ![](https://bbs.kanxue.com/upload/attach/202312/93_E8MHGXMB6QB5DPF.png)
- 确实没有我们要加载名称为：`ttEncrypt`的库，那我们就去手机中看看吧！
- 既然没有这个依赖库，那我们通过凯神的代码得知是调用`ttEncrypt([BI)[B`，那我们就试试这个吧！试试手机中是否有这个导出的符号了来碰碰运气了！
- ![](https://bbs.kanxue.com/upload/attach/202312/93_J8R55RBXYP97USG.png)
- 瞎猫碰着死耗子！嘿嘿，还不用去分析apk了，今天的运气还不错，继续冲冲冲！
- 那我们就把依赖库的名称从“ttEncrypt”改成“Encryptor”再试试！
- ![](https://bbs.kanxue.com/upload/attach/202312/93_3UTDJFBUN85VDR9.png)
- 这次还不赖，发现是找不到“ttEncrypt([BI)[B”方法，既然我们可以通过符号定位到so，那必定是存在这个方法的！
- 那为什么找不到呢？通过观察上面的前三条可以发现，现在的类名已经变更为：`com/bytedance/frameworks/encryptor/EncryptorUtil`，而不再是：`com/bytedance/frameworks/core/encrypt/TTEncryptUtils`！
- 那我们修正一下再试试！
- 参数为第一个参数为0x10个byte！第二个参数byte[]的长度！
- ![](https://bbs.kanxue.com/upload/attach/202312/93_QDEZP3RVPABFB7S.png)
- 竟然就可以跑起来了~
- 还有结果了呢~
- 那就关机睡觉！
- 那肯定不会就到此结束的啦~还得把算法移植出来呢！

## 算法移植

### 算法入口
- 说的算法移植，那第一步必定就是：原神启动！
- 哈哈哈，走错片场了呢！
- 是ida pro启动，打开我们的女神软件ida pro！
- ![](https://bbs.kanxue.com/upload/attach/202312/93_Q6DDQURCA7YVSV8.png)
- 然后把so拖入进行分析！
- 按照惯例，我们先看一下导出符号都有啥
- ![](https://bbs.kanxue.com/upload/attach/202312/93_STEKKEVESFFZA9Z.png)
- 只有两个导出函数，`JNI_OnLoad`跟`start`
- 我们都知道`JNI_OnLoad`是动态注册方法的一个时机，`start`是作为直接启动so的一个入口。那我们就看看`JNI_OnLoad`中都有啥吧！
- ![](https://bbs.kanxue.com/upload/attach/202312/93_BTHDSY97FZRYHEK.png)]
- 啥都没，那我们进入一下`sub_da0`吧
- ![](https://bbs.kanxue.com/upload/attach/202312/93_XFJBRYJ65BNJ4UR.png)
- 看不懂！所以我们简单看一下`sub_da0`的程序流程图
- ![](https://bbs.kanxue.com/upload/attach/202312/93_PVA25H6K4XWVB23.png)]
- 发现都是密密麻麻的块，直接劝退我这种密集恐惧症患者了！直接关闭了！
- 既然我们不想通过分析`JNI_OnLoad`找到他动态注册代码的地方，我们就通过unidbg的log直接得知他的运行入口！那就是：

|  |  |
| --- | --- |
| 1 | `Find native function Java_com_bytedance_frameworks_encryptor_EncryptorUtil_ttEncrypt => RX@0x40007a38[libEncryptor.so]0x7a38` |
- 通过这个告诉我们有时候不知道该咋办的时候可以多看看log！看看有没有一些蛛丝马迹的！
- 那我现在就知道他的代码入口点为：`0x7a38`

### 算法分析

#### so方法修正
- ![](https://bbs.kanxue.com/upload/attach/202312/93_HX5AC2N8P6Z3GEV.png)
- 哦豁，打开之后全部爆红，而且不能按f5，那可咋办了？？？
- 别着急，那我们就作为一个简单的修正即可！
- ![](https://bbs.kanxue.com/upload/attach/202312/93_9J8C37A2EYJYVGQ.png)
- 我们先从`0x7a38`开始往下选选择一块区域，选择一些就可以了，然后按一下p即可！（这种为强制将选择的区域代码创建方法）
- ![](https://bbs.kanxue.com/upload/attach/202312/93_FHPDGAWM38RE63D.png)
- 然后我们发现下面还有一些红色的块，此时别着急。我们往下拉直到找到RET这个指令的上一个红色快的最后一条代码。也就是`0x7B58`处，我们在这里按一下e即可！
- ![](https://bbs.kanxue.com/upload/attach/202312/93_2W5W5BS5EMGDMJH.png)
- 然后就正常啦！
- ![](https://bbs.kanxue.com/upload/attach/202312/93_QA9BCT6YZS947QC.png)
- 但是我们往上滑动的时候发现`0x7B04`也是有一块红色的！
- ![](https://bbs.kanxue.com/upload/attach/202312/93_WTYX5UPCYUQQXR6.png)
- 那我们也按照上面的方式，就在这一行按一下e即可！修正后就如图所示啦~
