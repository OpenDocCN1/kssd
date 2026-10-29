
# [原创]RC4、Base64魔改看雪CTF-变形金刚学习笔记

## 前言

最近在研究某某app的数据库，发现自己在so层的调试比较薄弱，专门找了看雪的CTF-变形金刚来学习。希望在用ida调试so方面有所突破。

利用国庆期间整理成笔记。技术不成熟或许描述的不够清晰请大伙见谅。

也拜读了几位大佬的文章。

## 工具准备

- `ida7.0`
- `Transformers.apk`
- `idea`
- `jadx`
- `frida`
- 小米4root手机（android6.0）

## app分析目标

为了熟悉该app，先介绍一下大神分析后的结果。

输入错误密码截图

比如输入密码是：`12345678`，会提示错误信息：`Transformers:error`

![](https://bbs.kanxue.com/upload/attach/202010/746297_TT69MPF7UA9ZN6X.jpg)

输入正确密码截图

输入密码(长度16个字符)是：`fu0kzHp2aqtZAuY6`，会提示错误信息：`Transformers:flag{android4-9}`

![](https://bbs.kanxue.com/upload/attach/202010/746297_A7GVHA9JMZP8TFT.jpg)

## 静态分析-java

把`Transformers.apk`拖入`jadx`

障眼法1:该app设计作者采用了障眼法，很容易欺骗分析人员，让我们认为`OnClick`的回调处理逻辑在`MainActivity`中。

障眼法2：在`MainActivity`的`OnCreate`中欺骗代码注册`OnClick`回调。主要原理是利用`Activity`的`onStart`事件会晚与`OnCreate`事件，从而实现在`OnStart`中注册的`OnClick`回调覆盖了`OnCreate`中注册的回调。

![](https://bbs.kanxue.com/upload/attach/202010/746297_RGH94EXXY42XPC9.jpg)

![](https://bbs.kanxue.com/upload/attach/202010/746297_YRWY4JRCW77HGFM.jpg)

`Activity`基类的`onStart`才是真正注册`OnClick`回调的地方

![](https://bbs.kanxue.com/upload/attach/202010/746297_XSW35RU7YWFSQP7.jpg)

## Base64解码图

为了方便理解`eq`中涉及的魔改的Base64编码，这里先贴上一直Base64解码的草稿图

下图是Base64编码4个字符`'{6\*`的解码草稿图，画的不是很好，主要是方便我自己理解写解码Base64的逻辑代码。

![](https://bbs.kanxue.com/upload/attach/202010/746297_U72KXFCZTN3Q2GF.jpg)

它是用64个可打印字符表示二进制所有数据方法。由于2的6次方等于64，所以可以用每6个位元为一个单元，对应某个可打印字符。我们知道三个字节有24个位元，就可以刚好对应于4个Base64单元，即3个字节需要用4个Base64的可打印字符来表示。

![](https://bbs.kanxue.com/upload/attach/202010/746297_599QHJDPREFYZRB.jpg)

## 静态分析与动态调试跟踪liboo000oo.so

这里将分3个关键步骤：

- 1：`JNI_OnLoad`分析动态注册`eq`对应的函数`sub_784`，稍后再详细介绍。
- 2：`.fini_array`部分跟踪，主要是`eq`函数用到的一下基础数据，比如Base64编码表、`RC4`加密算法用到的`key`初始值，之后它会通过一系列异或处理算出真正的密钥`key`。也稍后再详细介绍。
- 3：`eq`函数的算法跟踪分析。

这里先列一下分析结果

```
JNI_onLoad部分==========

	eq是由sub_784动态注册

	

ini_array部分==========



	#解码36长度字符串byte_4020：

	byte_4020 =650f909c-7217-3647-9331-c82df8b98e98+0x00（结束符）





	#解码Base64为的64+1个编码字符byte_4050= 

	base64Chars = byte_4050 =!:#$%&()+-*/`~_[]{}?<>,.@^abcdefghijklmnopqrstuvwxyz0123456789\';+0x00（结束符）



	#app伪装java类的名称byte_40A0：

	className_Sign= android/support/v7/app/AppCompiatActivity +0x00（结束符）





sub_784部分============

	

	获取RC4的密钥算法============：

		v5=算法01 = 删除(byte_4040中的"-") =650f909c721736479331c82df8b98e98

		v4=算法02 =删除byte_4020中的”-“后，特殊运算又加上”-“符号=89e89b8f-d28c-1339-7463-7127c909f056

	

	自定义算法03==========

	v6=v17=自定义算法03 = 特殊算法处理后 =36f36b3c-a03e-4996-8759-8408e626c215	

		算法参数01：v4

		算法参数02：unk_23DE(10位长度) =2409715836

		算法参数03：unk_23D8（16位长度）=dbeafc2409715836

	

	RC4魔改部分======	（已经逆向算法）

		RC4第1步：v48:拷贝&unk_B4D863E8中的256个字符到v48(应视是RC4算法的魔改数据)

		RC4第2步：v49:计算RC4 的临时256自己T向量，公式：iK[i]=(byte)aKey.charAt((i % aKey.length()));

		RC4第3步：V48状态向量S进行置换操作V48

		RC4第4步：RC4产生密钥流iOutputChar,然后return new String(iOutputChar)

			v27 =33 ,即v6[3];

			

		RC4第5步：产生密钥流--其他还结合魔改Base64运算

			分配输入密码对应的base64字节的存储空间

			Base64算法的字典

			

	Base64魔改部分======（已经逆向算法）

		Base64的字典部分-64个字符+1个结束符

		每4个字符中第0个与0x6异或，第0个与0xF异或

		     switch (i%4){

                case 0:

                    base64char = (char) (iAscii ^0x07);

                    break;



                case 2:

                    base64char = (char) (iAscii ^0xF);

                    break;

                default:

                    base64char =encodeChars[i];



            }
```

## JNI_OnLoad 代码逻辑分析RegisterNatives

1：搜索`JNI_OnLoad`，双击进入函数体

2：导入`Jni.h`文件。菜单路径：`ida/File/Load file/Parse C header file`

3：修改参数类型为`_JavaVM`

4：按g快捷键直接跳转到`0x4014`![](https://bbs.kanxue.com/upload/attach/202010/746297_B544NRQ677MBJMN.jpg)

![](https://bbs.kanxue.com/upload/attach/202010/746297_YCVNNNNSUZ5DM53.jpg)

`off_4010`即可看到注册的函数

找到`eq`注册对应的`sub_784`函数

![](https://bbs.kanxue.com/upload/attach/202010/746297_H44NTQ9MRRV7JM7.jpg)

当然`frida`的`hook_art_so_register.js`脚本快速定位`eq`对应`sub_784`

该脚本通过hook `art.so` 的`register`函数，打印动态注册的地址

```
//调用方式==测试ok

frida -U --no-pause -f package_name -l hook_art_so_register.js
```

或直接在`fini_array`初始化数据中可以看到

## 跟踪.init_array（其实这步可以跳过）

另外根据`so`文件的加载流程应该是先加载`init_array`，然后是`JNI_OnLoad`

`ida` 中`ctrl+s` 进入`init_array`，会先执行`.datadiv_decode5009363700628197108`，其实就是通过一些运算得到一些初始化数据。

![](https://bbs.kanxue.com/upload/attach/202010/746297_QWTBBRNP7HWEPWY.jpg)

`init_array`

按`f5` 采用伪代码方式查看`datadiv_decode5009363700628197108`，根据得到一些数据。

1：解码36长度字符串`byte_4020`：`650f909c-7217-3647-9331-c82df8b98e98+0x00`（结束符）

2：魔改Base64编码的table表，为的64+1个编码字符`byte_4050`= `!:#$%&()+-*/`~_[]{}?<>,.@^abcdefghijklmnopqrstuvwxyz0123456789\';+0x00`（结束符）

![](https://bbs.kanxue.com/upload/attach/202010/746297_8ZNZXMUH8PFQE4Z.jpg)

## eq函数的分析（即sub_784函数）

`ida`定位到`sub_784`，修改参数类型，并做了动态调试后，梳理`sub_784`的逻辑。

![](https://bbs.kanxue.com/upload/attach/202010/746297_24BVJMM69NVN2KZ.jpg)

`unk_23E8`字符串（`RC4`魔改算法的初始化256个字符S状态向量）

![](https://bbs.kanxue.com/upload/attach/202010/746297_ECVCR6QFVGSQ477.jpg)

`unk_23E8`字符串Base64的table 拷贝

![](https://bbs.kanxue.com/upload/attach/202010/746297_E2D3TUJ382FWGUV.jpg)

![](https://bbs.kanxue.com/upload/attach/202010/746297_AH6D2J5EGDJ7545.jpg)

```
unk_23E8=Sbox = [0xD7,0xDF,0x02,0xD4,0xFE,0x6F,0x53,0x3C,0x25,0x6C,0x99,0x97,0x06,0x56,0x8F,0xDE,0x40,0x11,0x64,0x07,0x36,0x15,0x70,0xCA,0x18,0x17,0x7D,0x6A,0xDB,0x13,0x30,0x37,0x29,0x60,0xE1,0x23,0x28,0x8A,0x50,0x8C,0xAC,0x2F,0x88,0x20,0x27,0x0F,0x7C,0x52,0xA2,0xAB,0xFC,0xA1,0xCC,0x21,0x14,0x1F,0xC2,0xB2,0x8B,0x2C,0xB0,0x3A,0x66,0x46,0x3D,0xBB,0x42,0xA5,0x0C,0x75,0x22,0xD8,0xC3,0x76,0x1E,0x83,0x74,0xF0,0xF6,0x1C,0x26,0xD1,0x4F,0x0B,0xFF,0x4C,0x4D,0xC1,0x87,0x03,0x5A,0xEE,0xA4,0x5D,0x9E,0xF4,0xC8,0x0D,0x62,0x63,0x3E,0x44,0x7B,0xA3,0x68,0x32,0x1B,0xAA,0x2D,0x05,0xF3,0xF7,0x16,0x61,0x94,0xE0,0xD0,0xD3,0x98,0x69,0x78,0xE9,0x0A,0x65,0x91,0x8E,0x35,0x85,0x7A,0x51,0x86,0x10,0x3F,0x7F,0x82,0xDD,0xB5,0x1A,0x95,0xE7,0x43,0xFD,0x9B,0x24,0x45,0xEF,0x92,0x5C,0xE4,0x96,0xA9,0x9C,0x55,0x89,0x9A,0xEA,0xF9,0x90,0x5F,0xB8,0x04,0x84,0xCF,0x67,0x93,0x00,0xA6,0x39,0xA8,0x4E,0x59,0x31,0x6B,0xAD,0x5E,0x5B,0x77,0xB1,0x54,0xDC,0x38,0x41,0xB6,0x47,0x9F,0x73,0xBA,0xF8,0xAE,0xC4,0xBE,0x34,0x01,0x4B,0x2A,0x8D,0xBD,0xC5,0xC6,0xE8,0xAF,0xC9,0xF5,0xCB,0xFB,0xCD,0x79,0xCE,0x12,0x71,0xD2,0xFA,0x09,0xD5,0xBC,0x58,0x19,0x80,0xDA,0x49,0x1D,0xE6,0x2E,0xE3,0x7E,0xB7,0x3B,0xB3,0xA0,0xB9,0xE5,0x57,0x6E,0xD9,0x08,0xEB,0xC7,0xED,0x81,0xF1,0xF2,0xBF,0xC0,0xA7,0x4A,0xD6,0x2B,0xB4,0x72,0x9D,0x0E,0x6D,0xEC,0x48,0xE2,0x33]
```

`byte_24E8`指向的内容

![](https://bbs.kanxue.com/upload/attach/202010/746297_QUXW4JS3T2KE2A4.jpg)

**函数return前的逻辑**

**![](https://bbs.kanxue.com/upload/attach/202010/746297_G6Q4SMBCVKRNVQE.jpg)**

`v49`字符串-`RC4`魔改的临时向量T

![](https://bbs.kanxue.com/upload/attach/202010/746297_322WACTUT7MC8AG.jpg)

最终是算出来Base64编码的`v28_new_base64`与`v41`的字符比较（`{9\*8ga\*l!Tn?@#fj'j$\g;;`）

![](https://bbs.kanxue.com/upload/attach/202010/746297_FU79F26NAKZYZ45.jpg)

把`sub_784`业务逻辑直接采用在`ida`中注解的方式修订如下
