
# [原创]安卓U3D逆向从`Assembly-CSharp`到il2cpp

随着unity技术的发展及厂商对于脚本源码的保护，很大一部分U3D应用的scripting backend已经由mono转为了il2cpp，本文从unity简单应用的制作讲起，介绍U3D应用脚本的`Assembly-CSharp.dll`的逆向及`il2cpp.so`的逆向分析。

目录如下：

- 0x1.U3D应用制作 → 输出两个apk，分别为mono和il2cpp
- 0x2.mono后台apk的逆向分析 → `Assembly-CSharp.dll` 逆向修改
- 0x3.il2cpp后台apk的逆向分析 → `il2cpp.so`逆向修改

## 0x1.U3D应用制作

工具：Unity最新版本 2017.2.1f1

### 1.1 创建Project

新建一个Unity3D Project，Project name为`UnityTest`，如下图

![](https://bbs.kanxue.com/upload/attach/201712/654083_qj6c86kddkuywzg.jpg)

### 1.2 创建script定义界面元素

创建`GameObject`和C# Script，均命名为`Sample`，同时创建两个UI Text，分别命名为`HP`和`Attack`。打开C# Script，进行界面元素的义，创建三个矩形按钮分别为`HP`，`ATK`和`LevelUP`，设置点击`HP`显示100，点击`ATK`显示50，点击`LevelUP`后`HP`和`ATK`的值都翻倍。

代码如下：

```
using UnityEngine;

using UnityEngine.UI;

public class Sample : MonoBehaviour

{

        public Text _hp;

        public Text _attack;



        private Rect _healRect = new Rect(10f, 20f, 300f, 100f);

        private Rect _attackRect = new Rect(10f, 170f, 300f, 100f);

        private Rect _LevelUpRect = new Rect(10f, 320f, 300f, 100f);



        public void SetHp(int hp)

	{

	    _hp.text = hp.ToString();

	}



	public void SetAttack(int attack)

	{

	    _attack.text = attack.ToString();

	}



	public void OnGUI()

	{

        int hp_value = 100;

        int attack_value = 50;

        if (GUI.Button(_healRect, "HP"))

            {

                SetHp(hp_value);

            }

        if (GUI.Button(_attackRect, "ATK"))

            {

                SetAttack(attack_value);

            }

        if (GUI.Button(_LevelUpRect, "LevelUp"))

            {

                SetHp(hp_value * 2);

                SetAttack(attack_value * 2);

            }

        }

}
```

代码和界面完成后，我们设置包名（`com.test.test`）进行打包，分别输出mono后台和il2cpp后台的apk。

![](https://bbs.kanxue.com/upload/attach/201712/654083_sdh9c9lfit23d95.jpg)

点击Player Setting进行后台设置，可点击Scripting Backend选择mono或者il2cpp。

![](https://bbs.kanxue.com/upload/attach/201712/654083_jhtqnrz0nknkq2t.jpg)

得到的apk我们分别命名为`Sample-mono.apk`和`Sample-il2cpp.apk`。

安装测试正常，界面如下：

初始界面

![](https://bbs.kanxue.com/upload/attach/201712/654083_pe7fvtzn3k95gaf.jpg)

点击`HP`和`ATK`后界面

![](https://bbs.kanxue.com/upload/attach/201712/654083_buoaak7c0luageu.jpg)

点击`LevelUp`后界面

![](https://bbs.kanxue.com/upload/attach/201712/654083_bppcft7fqew6nq5.jpg)

至此，我们完成了两种后台的apk的创建，接下来先对mono后台的apk进行逆向分析。

## 0x2.mono后台apk的逆向分析

工具：Android Killer，DnSpy

思路：将`Sample-il2cpp.apk`拖拽进Android Killer进行反编译，获取到脚本文件`/Assets/bin/Data/Managed/Assembly-CSharp.dll`，dll拖进DnSpy进行反编译分析，修改关键method后，保存并重新打包运行。

Android Killer部分不再上图，`Assembly-CSharp.dll`拖拽进DnSpy，我们可以很轻易的找到我们需要的关键函数，如图：

![](https://bbs.kanxue.com/upload/attach/201712/654083_mk5eav9dx8ctnug.jpg)

根据代码信息，我们可以确定，`num`为初始`HP`，`num2`为初始`ATK`，我们在此通过编辑IL指令将他们的值分别改为1000和500，如图：

![](https://bbs.kanxue.com/upload/attach/201712/654083_gxbi838fobs6ibm.jpg)

确认后C#编码变更如下：

![](https://bbs.kanxue.com/upload/attach/201712/654083_ijwjany9qs2mwor.jpg)

保存后，重新打包apk安装测试结果如下：

![](https://bbs.kanxue.com/upload/attach/201712/654083_wai7xo2fxo1pskb.jpg)

至此，我们的mono后台apk修改已经完成，接下来我们进行il2cpp后台apk的修改。

## 0x3.il2cpp后台apk的逆向分析

工具：Android Killer，`il2cppDumper`，IDA，010Editor

思路：利用`il2cppDumper`获取到关键函数的offset后，使用IDA找到关键函数，分析汇编语言逻辑后，利用010Editor进行16进制文件编辑来实现修改。

Android Killer相关操作再次不做赘述，反编译后我们得到的Project中，`lib`目录如图：

![](https://bbs.kanxue.com/upload/attach/201712/654083_rhoomos1pqsud3r.jpg)

在此我们只分析`armeabi-v7a`文件夹下的`libil2cpp.so`，`x86`为intel架构，暂不做分析处理。

将`il2cpp.so`连同`assets\bin\Data\Managed\Metadata`目录下的`global-metadata.dat`文件一起放进`il2cppDumper`目录。

![](https://bbs.kanxue.com/upload/attach/201712/654083_x6dqsg3yu321wtt.jpg)

运行`il2cppDumper`，第一步，选择`il2cpp.so`

![](https://bbs.kanxue.com/upload/attach/201712/654083_uz0u8yl6tokgvw3.jpg)

第二步，选择`global-metadata.dat`

![](https://bbs.kanxue.com/upload/attach/201712/654083_t6s1n97cibuinkc.jpg)

进入到`il2cppDumper`控制台界面

![](https://bbs.kanxue.com/upload/attach/201712/654083_7utda9j4sq4wzxj.jpg)

选择mode后可以得到`Dump.cs`，包含关键函数的偏移信息

![](https://bbs.kanxue.com/upload/attach/201712/654083_37qnym2jo86avk2.jpg)

![](https://bbs.kanxue.com/upload/attach/201712/654083_1d9z7wodqg32ib7.jpg)

打开`Dump.cs`，找到关键函数的偏移信息

![](https://bbs.kanxue.com/upload/attach/201712/654083_dpvsp5uon7faoig.jpg)

用IDA打开`libil2cpp.so`文件，跳转到此偏移地址`0x7CE514`

![](https://bbs.kanxue.com/upload/attach/201712/654083_sn7v47ybzsaqa6z.jpg)

定位到关键函数后，我们可以看汇编，也可以F5插件看伪代码，为了便于理解我们这里F5处理一下

![](https://bbs.kanxue.com/upload/attach/201712/654083_j93vtdk39p1j6ef.jpg)

这次我们从`LevelUp`入手，我们在编写源码的时候，是进行了乘以2的处理，为什么汇编语言中是直接将翻倍后的数值进行传参呢，因为汇编不会完全跟我们源码的逻辑相同，但是结果一定是一样的，`LevelUp`的汇编代码如下

![](https://bbs.kanxue.com/upload/attach/201712/654083_trfrpccteax8irr.jpg)

切换到对应的地址查看16进制

![](https://bbs.kanxue.com/upload/attach/201712/654083_jv4lv1f1fcdnhvh.jpg)

`0xC8`即为200，我们使用010Editor将其地址`7CE6B4`的值修改为`FF`（十进制为255）

![](https://bbs.kanxue.com/upload/attach/201712/654083_pu6j2p01h0vwg4i.jpg)

保存后替换`lib/armeabi-v7a`目录下的`libil2cpp.so`文件，打包运行，结果如下

![](https://bbs.kanxue.com/upload/attach/201712/654083_086alc2o93k17hr.jpg)

至此，il2cpp后台的apk已修改完毕。

## 附件

源码见附件，下载后请先open scence再进行building，scence路径`Assets/Scences/Sample.unity`

APK请网盘下载，如下：

链接: [`Sample-mono.apk`](https://bbs.kanxue.com/elink@e4cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3j5h3&6Q4x3X3g2T1j5h3W2V1N6g2)9J5k6h3y4G2L8g2)9J5c8Y4y4Q4x3V1j5I4K9e0g2q4k6#2g2o6f1l9`.`.) 密码: fua5

链接: [`Sample-il2cpp.apk`](https://bbs.kanxue.com/elink@34bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3j5h3&6Q4x3X3g2T1j5h3W2V1N6g2)9J5k6h3y4G2L8g2)9J5c8Y4y4Q4x3V1j5I4K9e0g2a6M7h3!0j5N6R3`.`.) 密码: qpav

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- [`Sample源码.zip`](javascript:void(0);) （32.25kb，182次下载）
- [`Il2CppDumper-v2.1.6.zip`](javascript:void(0);) （49.35kb，631次下载）
