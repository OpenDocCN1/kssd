
# [原创]flutter逆向 ACTF native app

## 前言

算了一下好长时间没打过CTF了,前两天看到ACTF逆向有道flutter逆向题就过来玩玩啦,花了一个下午做完了.说来也巧,我给DASCTF十月赛出的逆向题其中一道也是flutter,不过那题我难度降的相当之低啦,不知道有多少人做出来了呢~

## 还原函数名

flutter逆向的一大难点就是不知道`libapp.so`的函数名,虽然有工具`reflutter`可以帮助我们得到其中的符号,但是我个人认为基于对`libflutter.so`源码插桩后重编译再重打包apk的方式具有极大的不可预料性,极有可能导致apk闪退,这一题便出现了这种情况,所以接下来我将介绍的工具`blutter`是纯静态分析来还原函数名,更令人惊喜的是它提供了`IDApython`脚本来让我们可以在IDA中对函数进行重命名,而这个项目中提供的其他文件也相当好用

### blutter的编译及使用

blutter项目地址

```text
https://github.com/worawit/blutter
```

在各个平台如何编译在这个项目的`README.md`中写的已经相当详细了,这里我就简单介绍一下Windows上的编译过程吧,注意一下这些命令需要全程运行在代理环境否则会导致无法下载

首先clone项目

```shell
git clone https://github.com/worawit/blutter --depth=1
```

随后运行初始化脚本

```shell
cd .\blutter\
python .\scripts\init_env_win.py
```

请注意,接下来我们需要打开`x64 Native Tools Command Prompt`,它可以在`Visual Studio`文件夹中找到

![image-20231031182720564](upload/attach/202310/963320_MFVX569Z367GMZN.png)

然后运行`blutter.py`并提供`libapp.so`和`libflutter.so`的文件夹路径以及输出文件夹路径

```shell
python .\blutter.py ..\chall\lib\arm64-v8a\ .\output
```

![image-20231031183050383](upload/attach/202310/963320_8SE9Q6XPXPQNKRG.png)

输出文件夹目录如下

![image-20231031183135453](upload/attach/202310/963320_EXQ7X5P6KS763RX.png)

随后我们用ida反编译`libapp.so`,并运行输出文件夹中的IDApython脚本`ida_script/addNames.py`,符号就被全部恢复出来啦

![image-20231031183441477](upload/attach/202310/963320_2YZJDCGNUYWEFE9.png)

## hook关键函数 获取函数参数

这里我们需要关注的函数是`flutter_application_1_main__LongPressDemoState::_onTap`,因为在flutter的开发中,`onTap`函数是按钮点击之后的响应函数

![image-20231031183702090](upload/attach/202310/963320_7FHYGYEEZYMWY6Y.png)

随后我们进入`sub_1DE500`,在该函数中双击`sub_1DE59C`进入

![image-20231031183814522](upload/attach/202310/963320_2NGTJ6XQQ6V2PWP.png)

在这个函数中我们发现了`256`,`%`,`^`这些特征,合理猜测一下算法可能是RC4

![image-20231031185252569](upload/attach/202310/963320_BQ4VDSEVN49PGKA.png)

![image-20231031185414233](upload/attach/202310/963320_GM8G96YGJ2H8SBG.png)

![image-20231031185438467](upload/attach/202310/963320_MC624UWY8EDGAN6.png)

接下来我们使用输出文件夹中的`blutter_frida.js`hook一下`sub_1DE59C`看看情况

![image-20231031185604019](upload/attach/202310/963320_F32MESU23H6G8ZB.png)

```shell
PS D:\hgame\ACTF\native app\work\blutter> frida -U -f "com.example.flutter_application_1" -l .\output\blutter_frida.js
[Pixel 3::com.example.flutter_application_1 ]->
Unhandle class id: 46, TypeArguments
GrowableList@6d00488c29 = [
188676,
0,
{
"key": "Unhandle class id: 46, TypeArguments"
},
34,
{
"key": [
184,
132,
137,
215,
146,
65,
86,
157,
123,
100,
179,
131,
112,
170,
97,
210,
163,
179,
17,
171,
245,
30,
194,
144,
37,
41,
235,
121,
146,
210,
174,
92,
204,
22
]
},
0,
0,
0
]
```

这里我们只hook到一个数组的值,另一个数组的类型是`TypeArguments`,研究了一下`blutter_frida.js`后发现作者还没有对这种数据类型格式提供hook支持

![image-20231031185952753](upload/attach/202310/963320_NCJGBA8PY5RKHD3.png)

### IDA动态调试libapp.so

现在我们得到了一个数组,我们就暂时认为它就是flag经过加密之后得到的结果,接下来我们在IDA中对`sub_1DE59C`下断点动态调试来更加深入的研究一下

首先我们需要将IDA文件夹中的`dbgsrv/android_server64` push到手机上面,然后运行一下并且指定端口

```shell
blueline:/data/local/tmp # ./as64 -p 11112
IDA Android 64-bit remote debug server(ST) v7.7.27. Hex-Rays (c) 2004-2022
Listening on 0.0.0.0:11112...
```

随后端口转发一下

```shell
PS C:\Users\oacia> adb forward tcp:11112 tcp:11112
11112
```

在IDA中选择调试器为`Android debugger`

![image-20231031190502580](upload/attach/202310/963320_CWU2U7SEGDTUXEV.png)

随后点击`Debugger->Debugger options...`选择如下配置

![image-20231031190621711](upload/attach/202310/963320_3WDWY3VANC7XDVA.png)

点击`Debugger->Process options...`,`Hostname`修改为`127.0.0.1`,`Port`修改为`11112`

![image-20231031190653343](upload/attach/202310/963320_F5ZXSQ9S9F8PRHS.png)

然后点击`Debugger->Attach to process...`,附加到我们目标包名的进程上面

![image-20231031190932527](upload/attach/202310/963320_T5PNZXHPZ2V2K83.png)

弹出该弹窗选择Same即可

![image-20231031191001330](upload/attach/202310/963320_6DSHQUM3QAC9W9H.png)

在手机上点击按钮,然后在IDA中点击这个绿色的剪头,就可以动态调试啦

![image-20231031191049306](upload/attach/202310/963320_RYU2BUUNECY5R5J.png)

![image-20231031191123141](upload/attach/202310/963320_EJZA7BBN43NHQRT.png)

在动态调试之后,未知的变量也逐渐浮现了出来,这里我们发现了`v28>=256`,那么很有可能就是RC4了哦

![image-20231031191544680](upload/attach/202310/963320_EDRUMJUKRPN897U.png)

既然这样,那么直接在这里唯一的异或的地方用IDA去trace一下,把异或的数组dump下来不就行了:)

![image-20231031192301154](upload/attach/202310/963320_6F573XUGZBVAAEV.png)

于是我们得到了被异或的数组了

![image-20231031193001825](upload/attach/202310/963320_B6TPMVQNHB8SVQ2.png)

但是在异或运算的地方下断点之后,我输入的数全都是`1`,这里被异或的数也全是`0xce`

![image-20231031194118997](upload/attach/202310/963320_VAG2DFT5HDT9V4E.png)

所以莫非不是RC4?让`0xce`和`0x31`异或一下看看,竟然是`0xff`这么有意义的数字

![image-20231031194218233](upload/attach/202310/963320_J7YU5M4WFBEMB2P.png)

所以exp也就能写出来啦~

```python
final = [184, 132, 137, 215, 146, 65, 86, 157, 123, 100, 179, 131, 112, 170, 97, 210, 163, 179, 17, 171, 245, 30, 194,
144, 37, 41, 235, 121, 146, 210, 174, 92, 204, 22]
xor = [14, 14, 68, 80, 29, 201, 241, 46, 197, 208, 123, 79, 187, 55, 234, 104, 40, 117, 133, 12, 67, 137, 91, 31, 136,
177, 64, 234, 24, 27, 26, 214, 122, 217]
flag = [chr(xor[i]^final[i]^0xff) for i in range(len(final))]
print(''.join(flag))
# Iu2xpwXLAK734btEt9kXIhfpRgTlu6KuI0
```
