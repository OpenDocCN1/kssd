
# 梦幻屏保 v1.1

> 原文：[`www.pediy.com/kssd/pediy05/pediy50509.htm`](https://www.pediy.com/kssd/pediy05/pediy50509.htm)

本文系 `jney2` 第一篇破文，错误之处，敬请指正。版本归论坛和作者所有。

软件名称：`梦幻屏保 V1.1`  
软件大小： `866 KB`  
软件语言： `简体中文`  
软件类别： `国产软件 / 共享版 / 桌面制作`  
应用平台： `Win9x/NT/2000/XP`  
加入时间： `2003-02-22 21:27:25`  
下载次数： `1132`  
开 发 商： [`remotion.myetang.com/`](http://remotion.myetang.com/)

软件介绍：  

梦幻屏保是一个操作简单表现出色的屏幕保护制作工具。您只须简单的单击鼠标，几分钟内就可创建美轮美奂的 Windows 屏幕保护。软件特点:1.程序操作简便；2.支持添加任意张 Bmp、Jpg,Jpeg 格式图片；3.支持添加任意个媒体文件，包括 mp3,wav,mid 格式；4.拥有多达 128 中图片切换特效；5 支持顺序或是随机方式播放图片和媒体文件。您还可以利用 梦幻屏保 制作后缀为 EXE 的独立运行的电子贺卡。还想要什么呢？这就是您的选择！ 

下载页面：`http://www.skycn.net/soft/10998.html`

加密方式：注册码  
功能限制：次数限制  
PJ 工具：主要是 `WIN32Dasm 9.0` 汉化版。  
PJ 日期：`2003-03-25`  

作者 `jney2` 申明：只是学习，请不用于商业用途或是将本文方法制作的注册机任意传播，造成后果，本人一概不负。

1，安装完成后，对主文件 `SDD.EXE` 用 `PEiD` 扫描，发现用 `UPX` 打包，于是用 `UPX124W` 解包。得到文件 `2.exe`。

2，用 `J.Ping W32Dasm 9.0` 反汇编 `2.exe`,保存汇编工程。

3，点字符串数据参考按钮即可看到下列字符串：（有删节）

"错误,梦幻屏保 注册认证失败"

"梦幻屏保 1.1(注册版)"  

"梦幻屏保 注册成功,谢谢您对本软件的支持,您的认?

"请输入项目名称！"  

"请输入注册认证码!"  

"请至少添加两张图片！"  

"请重新启动本软件,以完成软件的注册"  

"软件试用次数已到,是否马上注册本软件?"

"未注册版本限用 40 次,您已经使用了"  

"未注册版本限用 40 次,您已经使用了 1 次"  

"未注册版本限用 40 次,您已经使用了 40 次"

"注册码输入有误,请重新输入!"

4，喜欢爆破的可作以下修改：  

（1），修改方试用次数。  

在字符串数据参考清单中双击 "未注册版本限用 40 次,您已经使用了 40 次" 即可看到如下反汇编代码：

```
:004CDF8B BA34E24C00                 mov edx, 004CE234  
:004CDF90 8BC3                       mov eax, ebx  
:004CDF92 8B38                       mov edi, dword ptr [eax]  
:004CDF94 FF5708                     call [edi+08]  
:004CDF97 83F828                     cmp eax, 00000028；28H=40  
:004CDF9A 0F8EAE000000               jle 004CE04E；不大于则继续试用 (jney2 注)  
:004CDFA0 8B8614030000               mov eax, dword ptr [esi+00000314]  
:004CDFA6 8B4068                     mov eax, dword ptr [eax+68]  
:004CDFA9 BAFF000000                 mov edx, 000000FF  
:004CDFAE E8F961F5FF                 call 004241AC

* Possible StringData Ref from Code Obj ->"未注册版本限用 40 次,您已经使用了 40 次"  
                                   |  
:004CDFB3 BA88E24C00                 mov edx, 004CE288  
:004CDFB8 8B8614030000               mov eax, dword ptr [esi+00000314]  
:004CDFBE E8F953F7FF                 call 004433BC
```

稍懂汇编就知该怎么修改了，是改次数还是把 `JLE` 改为 `JMP` 你看着办吧！

（2），直接修改为注册版。

在字符串数据参考清单中双击 "梦幻屏保 1.1(注册版)" 即可看到如下反汇编代码：

```
* Possible StringData Ref from Code Obj ->"Project"  
                                   |  
:004CDF42 BA34E24C00                 mov edx, 004CE234  
:004CDF47 8BC3                       mov eax, ebx  
:004CDF49 8B38                       mov edi, dword ptr [eax]  
:004CDF4B FF5708                     call [edi+08]  
:004CDF4E 5A                         pop edx  
:004CDF4F 3BD0                       cmp edx, eax；判断是否注册？  
:004CDF51 7531                       jne 004CDF84；未注册则跳。(jney2 注)  
:004CDF53 A1E4644D00                 mov eax, dword ptr [004D64E4]  
:004CDF58 8B00                       mov eax, dword ptr [eax]

* Possible StringData Ref from Code Obj ->"梦幻屏保 1.1(注册版)"  
                                   |  
:004CDF5A BA58E24C00                 mov edx, 004CE258  
:004CDF5F E85854F7FF                 call 004433BC
```

把 `7531` 改为 `7431` 或 `9090` 就 `OK` 了。（当然用 `HEX` 编辑器）

5，找到属于自己的注册码。  

首先声明我是一个级别为 0 的菜鸟。分析该程序结构清晰，找注册码应该不难。  

我的电脑两个硬盘三个系统。两个 `XP`，一个 `WINDOWS2000SERVER`，我绝大部分时间用 `XP`。所以用不了 `TRW2000` 和 `SOFTICE`。就用 `WIN32Dasm` 吧。我是这样找到注册码的。

1），在字符串数据参考清单中双击 "注册码输入有误,请重新输入!" 即可看到如下反汇编代码：

```
:004CE398 8B4DFC                     mov ecx, dword ptr [ebp-04]  
:004CE39B 8A4C01FF                   mov cl, byte ptr [ecx+eax-01]  
:004CE39F 80F930                     cmp cl, 30  
:004CE3A2 7208                       jb 004CE3AC  
:004CE3A4 8B5DFC                     mov ebx, dword ptr [ebp-04]  
:004CE3A7 80F939                     cmp cl, 39  
:004CE3AA 761F                       jbe 004CE3CB；判断是否为 0-9 的数字，是则跳。(jney2 注)

* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:004CE3A2(C)  
|

* Possible StringData Ref from Code Obj ->"注册码输入有误,请重新输入!"  
                                   |  
:004CE3AC B8F8E44C00                 mov eax, 004CE4F8  
:004CE3B1 E892E2F6FF                 call 0043C648  
:004CE3B6 8B9610030000               mov edx, dword ptr [esi+00000310]  
:004CE3BC A154824D00                 mov eax, dword ptr [004D8254]  
:004CE3C1 E83A01F9FF                 call 0045E500  
:004CE3C6 E9C1000000                 jmp 004CE48C

* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:004CE3AA(C)  
|  
:004CE3CB 40                         inc eax  
:004CE3CC 4A                         dec edx  
:004CE3CD 75C9                       jne 004CE398

* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:004CE391(C)  
|  
:004CE3CF 8D55EC                     lea edx, dword ptr [ebp-14]  
:004CE3D2 8B860C030000               mov eax, dword ptr [esi+0000030C]  
:004CE3D8 E8AF4FF7FF                 call 0044338C  
:004CE3DD 8B45EC                     mov eax, dword ptr [ebp-14]  
:004CE3E0 E873ADF3FF                 call 00409158  
:004CE3E5 E82EF8FFFF                 call 004CDC18  
:004CE3EA 8BD8                       mov ebx, eax  
:004CE3EC 8B45FC                     mov eax, dword ptr [ebp-04]  
:004CE3EF E864ADF3FF                 call 00409158  
:004CE3F4 3BD8                       cmp ebx, eax  
:004CE3F6 0F8586000000               jne 004CE482  
:004CE3FC 8D55F8                     lea edx, dword ptr [ebp-08]  
:004CE3FF A1E4644D00                 mov eax, dword ptr [004D64E4]  
:004CE404 8B00                       mov eax, dword ptr [eax]  
:004CE406 E8FD3B0000                 call 004D2008  
:004CE40B 8D45E8                     lea eax, dword ptr [ebp-18]

* Possible StringData Ref from Code Obj ->"\\DSS.ini"  
                                   |  
:004CE40E B91CE54C00                 mov ecx, 004CE51C  
:004CE413 8B55F8                     mov edx, dword ptr [ebp-08]  
:004CE416 E82967F3FF                 call 00404B44  
:004CE41B 8B4DE8                     mov ecx, dword ptr [ebp-18]  
:004CE41E B201                       mov dl, 01

* Possible StringData Ref from Code Obj ->"|軫"  
                                   |  
:004CE420 A19CD34600                 mov eax, dword ptr [0046D39C]  
:004CE425 E822F0F9FF                 call 0046D44C  
:004CE42A 8BD8                       mov ebx, eax  
:004CE42C 8D55E4                     lea edx, dword ptr [ebp-1C]  
:004CE42F 8B860C030000               mov eax, dword ptr [esi+0000030C]  
:004CE435 E8524FF7FF                 call 0044338C  
:004CE43A 8B45E4                     mov eax, dword ptr [ebp-1C]  
:004CE43D E816ADF3FF                 call 00409158  
:004CE442 E8D1F7FFFF                 call 004CDC18  
:004CE447 50                         push eax

* Possible StringData Ref from Code Obj ->"registecode"  
                                   |  
:004CE448 B930E54C00                 mov ecx, 004CE530

* Possible StringData Ref from Code Obj ->"Project"  
                                   |  
:004CE44D BA44E54C00                 mov edx, 004CE544  
:004CE452 8BC3                       mov eax, ebx  
:004CE454 8B30                       mov esi, dword ptr [eax]  
:004CE456 FF560C                     call [esi+0C]  
:004CE459 8BC3                       mov eax, ebx  
:004CE45B E8F055F3FF                 call 00403A50

* Possible StringData Ref from Code Obj ->"梦幻屏保 注册成功,谢谢您对本软件的支持,您的认?  
                                      ->"墒俏易畲蟮亩?"  
                                   |  
:004CE460 B854E54C00                 mov eax, 004CE554  
:004CE465 E8DEE1F6FF                 call 0043C648

* Possible StringData Ref from Code Obj ->"请重新启动本软件,以完成软件的注册"  
                                   |  
:004CE46A B89CE54C00                 mov eax, 004CE59C  
:004CE46F E8D4E1F6FF                 call 0043C648  
:004CE474 A1FC664D00                 mov eax, dword ptr [004D66FC]  
:004CE479 8B00                       mov eax, dword ptr [eax]  
:004CE47B E89050F9FF                 call 00463510  
:004CE480 EB0A                       jmp 004CE48C

* Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:004CE3F6(C)  
```



\* Possible StringData Ref from Code Obj -> "错误,梦幻屏保 注册认证失败"

```
:004CE482 B8C8E54C00                 mov eax, 004CE5C8
:004CE487 E8BCE1F6FF                 call 0043C648
```

分析以上代码可以看出，关键语句在：

```
:004CE3CF 8D55EC                     lea edx, dword ptr [ebp-14]；1
:004CE3D2 8B860C030000               mov eax, dword ptr [esi+0000030C]；2
:004CE3D8 E8AF4FF7FF                 call 0044338C；3
:004CE3DD 8B45EC                     mov eax, dword ptr [ebp-14]；4
:004CE3E0 E873ADF3FF                 call 00409158；5
:004CE3E5 E82EF8FFFF                 call 004CDC18；6
:004CE3EA 8BD8                       mov ebx, eax；7
:004CE3EC 8B45FC                     mov eax, dword ptr [ebp-04]；8
:004CE3EF E864ADF3FF                 call 00409158；9
:004CE3F4 3BD8                       cmp ebx, eax；比较注册码是否正确？等于则 OK！
:004CE3F6 0F8586000000               jne 004CE482
```

2），在“调试”菜单里选择“载入过程”，则会调入 `2.exe`。
3），找到以上语句，在语句 `1` 这一行按 `F2` 设断点。
4），按 `F9` 运行。则启动程序。点“软件注册”，你会看到你的机器码，我的是 `59858453`，在注册码中填入试炼码 `23456789`。点“注册”。
5），程序会在断点处停住。按 `F8`，执行完语句 `4` 后，你会发现 `EAX` 所指的内存是我的机器码 `59858453`。
6），语句 `5` 子程序作用是将字符转为十六进制。关键代码如下：

...(有删）

```
:0040330F 8A1E                       mov bl, byte ptr [esi]
:00403311 46                         inc esi
```

...(有删）

```
:0040334E 80EB30                     sub bl, 30；字符转十进制
:00403351 80FB09                     cmp bl, 09
:00403354 7725                       ja 0040337B
:00403356 39F8                       cmp eax, edi
:00403358 7721                       ja 0040337B
:0040335A 8D0480                     lea eax, dword ptr [eax+4*eax]
:0040335D 01C0                       add eax, eax
:0040335F 01D8                       add eax, ebx；十进制转十六进制
:00403361 8A1E                       mov bl, byte ptr [esi]
:00403363 46                         inc esi
:00403364 84DB                       test bl, bl
:00403366 75E6                       jne 0040334E
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```
:00403346(C)
```

```
:00403368 FECD                       dec ch
:0040336A 7409                       je 00403375
:0040336C 85C0                       test eax, eax
:0040336E 7D4E                       jge 004033BE
:00403370 EB09                       jmp 0040337B
```

7），所以按 `F8` 执行完语句 `6` 后，`EAX` 的值便是注册码的十六进制。

`EAX=72391CA4H=1916345508` 这才是我要找的真正的注册码。（用计算器算呀！）

8），按 `F8` 执行完语句 `8` 后，你会发现 `EAX` 所指的内存是你输入的试炼码 `23456789`。
9），同样的语句 `9` 将 `23456789` 转为十六制。
10），中止进程，重新启动 `DSS.EXE`,输入得到注册码，OK！:)!

6,利用刘健英的 `KERMAKER1.73` 可以很快地做出内存注册机。方法不再详解。

以上有不当之处请高手指正！谢谢！。`jney2` 的信箱是：`jney2@21cn.com`

`4dd466d0e1e0cc123afbe504da2462ca`
