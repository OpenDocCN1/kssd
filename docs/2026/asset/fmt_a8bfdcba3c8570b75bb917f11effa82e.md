
# [原创]ASPACK 2.2 破解手记

【文章标题】: ASPACK 2.2 破解手记

【文章作者】: 啊cr/FTSTT

【软件名称】: 47cK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3q4K6M7r3q4U0K9#2)9J5k6h3y4G2L8b7`.`.

【软件大小】: 675 KB (691,771 字节)

【下载地址】: 3ecK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3q4K6M7r3q4U0K9#2)9J5k6h3y4G2L8b7`.`.

【加壳方式】: Aspr

【保护方式】: Aspr+key解码

【编写语言】: Delphi 5

【使用工具】: OD

【操作平台】: Win xp Sp3

【软件介绍】: A Win32 executable file compressor.

【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

--------------------------------------------------------------------------------

> 今天看到RegKiller放了一个2.2的破解，说明中提到他这个破解 右键菜单调用压缩的时候有一个自动退出的bug，想起自己4月份放的那个忘记测试这里了，测试之下发现比他这个bug还严重 都弹出出错对话框了 呵呵，fix it。重新弄一下 记录一下完整的破解过程。

准备工作：`OD`一个，`VolX`脚本一个，`ASPACK 22`安装文件一只，`ASPACK 212r`注册版一套（用来抄代码）。

开始`XX`：

## 第一 comodore

### 第一步，观察

根据经验这个版本共有三个限制 配置保存 配置应用 时间限制。

![](https://bbs.kanxue.com/upload/attach/200905/3847_ob8jyif4dw5rqio.gif)

### 第二步，脱壳

感谢`VolX`，轻松秒掉，时间限制同时也秒掉了。

![](https://bbs.kanxue.com/upload/attach/200905/3847_84dav0xf6je69gc.gif)

版本区别不大，尝试抄袭`212r`中的代码。

### 第三步，调戏212r，把需要抄袭的代码都找到。

注册`212r`，`OD`载入。

回忆一下限制，不能保存选项，有可能是是退出不保存或者启动的时候不读取，假设是启动不读取配置，看看初始化代码吧。

用资源工具查看主窗体叫`TfrmMain`，`F9`运行程序，`G 401000` 二进制搜索"`TfrmMain`" ，找到这里：

```
0043F120   .  54 66 72 6D 4>ASCII "TfrmMain"
```

向下翻看到这里

```
0043F5DC . FF15 08494400 CALL DWORD PTR DS:[444908]

0043F5E2 . E9 83000000 JMP ASPack.0043F66A
```

典型的`Aspr`解码，记下`CALL DWORD PTR DS:[444908]`，搜索此命令

```
0043F5DC . FF15 08494400 CALL DWORD PTR DS:[444908]

0044152C . FF15 08494400 CALL DWORD PTR DS:[444908]

00441CEA . FF15 08494400 CALL DWORD PTR DS:[444908]

00442257 |> \FF15 08494400 CALL DWORD PTR DS:[444908]
```

共四处解码。

在`22`里同样方法

找到

```
004651A2 . FF15 F0AC4600 CALL DWORD PTR DS:[46ACF0]

004651A8 . E9 83000000 JMP Doit_ASP.00465230
```

记下`CALL DWORD PTR DS:[46ACF0]`，并搜此命令

```
004651A2 . FF15 F0AC4600 CALL DWORD PTR DS:[46ACF0]

0046716C . FF15 F0AC4600 CALL DWORD PTR DS:[46ACF0]

00467951 . FF15 F0AC4600 CALL DWORD PTR DS:[46ACF0]

00467EEB > \FF15 F0AC4600 CALL DWORD PTR DS:[46ACF0]
```

巧啊，也是四处，假设他们对应的功能是完全相同的，事实证明正是如此。

分析`212r`中相关代码的功能，方便移植后的修复。

### 第一处：`0043F5DC`

重新启动`212r`，在`0043F5DC`出下内存访问断点，

第`N`次后断在

```
0043F5DC   .  FF15 08494400 CALL DWORD PTR DS:[444908]
```

`F8`走过这句，可以看到下面解码了，`Ctrl+A`分析一下

```
0043F5DC . FF15 08494400 CALL DWORD PTR DS:[444908] ; 解码

0043F5E2 . E9 01000000 JMP ASPack.0043F5E8

0043F5E7 29 DB 29 ; 小花

0043F5E8 > 8B45 FC MOV EAX,DWORD PTR SS:[EBP-4]

0043F5EB . 8B80 E0010000 MOV EAX,DWORD PTR DS:[EAX+1E0] ; @控件1: LabelTrial

0043F5F1 . 8B40 44 MOV EAX,DWORD PTR DS:[EAX+44]

0043F5F4 . 33D2 XOR EDX,EDX

0043F5F6 . E8 2993FDFF CALL ASPack.00418924 ; @CALL_1:

0043F5FB . 8D55 D4 LEA EDX,DWORD PTR SS:[EBP-2C]

0043F5FE . A1 BC564400 MOV EAX,DWORD PTR DS:[4456BC] ;@指针 1:

0043F603 . E8 586DFCFF CALL ASPack.00406360 ; @CALL_2:

0043F608 . 8B55 D4 MOV EDX,DWORD PTR SS:[EBP-2C]

0043F60B . 8B45 FC MOV EAX,DWORD PTR SS:[EBP-4]

0043F60E . 8B80 E0010000 MOV EAX,DWORD PTR DS:[EAX+1E0] ; @控件1: LabelTrial

0043F614 . E8 073BFDFF CALL ASPack.00413120 ; @CALL_3:

0043F619 . 8B45 FC MOV EAX,DWORD PTR SS:[EBP-4]

0043F61C . 8B80 6C020000 MOV EAX,DWORD PTR DS:[EAX+26C] ; @控件2: LabelUnregistered

0043F622 . E8 E53EFDFF CALL ASPack.0041350C ; @CALL_4:

0043F627 . B2 01 MOV DL,1

0043F629 . 8B45 FC MOV EAX,DWORD PTR SS:[EBP-4]

0043F62C . 8B80 7C020000 MOV EAX,DWORD PTR DS:[EAX+27C] ; @控件3: EditSection

0043F632 . E8 513AFDFF CALL ASPack.00413088 ; @CALL_5:

0043F637 . B2 01 MOV DL,1

0043F639 . 8B45 FC MOV EAX,DWORD PTR SS:[EBP-4]

0043F63C . 8B80 78020000 MOV EAX,DWORD PTR DS:[EAX+278] ; @控件4: CBExit

0043F642 . E8 413AFDFF CALL ASPack.00413088 ; @CALL_5:

0043F647 . B2 01 MOV DL,1

0043F649 . 8B45 FC MOV EAX,DWORD PTR SS:[EBP-4]

0043F64C . 8B80 EC020000 MOV EAX,DWORD PTR DS:[EAX+2EC] ; @控件5: CBMaxCompression

0043F652 . E8 313AFDFF CALL ASPack.00413088 ; @CALL_5:

0043F657 . 8B45 FC MOV EAX,DWORD PTR SS:[EBP-4]

0043F65A . 8B80 E4020000 MOV EAX,DWORD PTR DS:[EAX+2E4] ; @控件6: FS

0043F660 . C640 20 01 MOV BYTE PTR DS:[EAX+20],1 ; @标志位 1:

0043F664 . EB 04 JMP SHORT ASPack.0043F66A

0043F666 . 46 DB 46 ; 小花

0043F667 . 8C DB 8C ; 小花

0043F668 . DD DB DD ; 小花

0043F669 . 94 DB 94 ; 小花

0043F66A > FF15 0C494400 CALL DWORD PTR DS:[44490C] ; 编码
```

这段代码从解码开始到编码结束，其中小花若干,nop小花后`0043F5E2`——`0043F669`二进制数据如下

```
E9 01 00 00 00 90 8B 45 FC 8B 80 E0 01 00 00 8B 40 44 33 D2 E8 29 93 FD FF 8D 55 D4 A1 BC 56 44

00 E8 58 6D FC FF 8B 55 D4 8B 45 FC 8B 80 E0 01 00 00 E8 07 3B FD FF 8B 45 FC 8B 80 6C 02 00 00

E8 E5 3E FD FF B2 01 8B 45 FC 8B 80 7C 02 00 00 E8 51 3A FD FF B2 01 8B 45 FC 8B 80 78 02 00 00

E8 41 3A FD FF B2 01 8B 45 FC 8B 80 EC 02 00 00 E8 31 3A FD FF 8B 45 FC 8B 80 E4 02 00 00 C6 40

20 01 EB 04 90 90 90 90
```

新开一个`OD`载入脱壳后的`22`，定位到对应的位置`004651A8`，二进制粘贴上面的代码，看一下结尾 长度刚好一致。

```
004651A2   .  FF15 F0AC4600 CALL DWORD PTR DS:[46ACF0]

00465230   > \FF15 F4AC4600 CALL DWORD PTR DS:[46ACF4]
```

这两个已经被强大的脚本自动修复了，指向`ret`等于`nop`了，如果空间不足可以`nop`掉这两句腾出空间补代码。

这段代码一共涉及`6`个控件，`5`个`CALL`一个指针，一个标志，若干局部变量。

这些值复制到新版中可能需要修正，下面记录他们的特征。

在`212r`中

控件名称可以算是空间的特征，查找控件名：

例如：

```
0043F5EB   .  8B80 E0010000 MOV EAX,DWORD PTR DS:[EAX+1E0]
```

控件标号是`1E0`，在`code`段二进制搜索（区分大小写） `E001`，第一次找到的是代码，第二次找到

```
0043EA70      E0            DB E0

0043EA71      01            DB 01

0043EA72      00            DB 00

0043EA73      00            DB 00

0043EA74      03            DB 03

0043EA75      00            DB 00

0043EA76   .  0A            DB 0A

0043EA77   .  4C 61 62 65 6>ASCII "LabelTrial"
```

这里"`LabelTrial`"就是控件的名字了，同样方法查找其他控件,在新程序里搜索控件名就可以找到编号了。

在`22`中

在`code`段二进制搜索（区分大小写） `LabelTrial`，找到

```
00464620      04            DB 04

00464621      03            DB 03

00464622      00            DB 00

00464623      00            DB 00

00464624      03            DB 03

00464625      00            DB 00

00464626   .  0A            DB 0A

00464627   .  4C 61 62 65 6>ASCII "LabelTrial"
```

标号为`304`

修正代码

```
004651B1      8B80 04030000 MOV EAX,DWORD PTR DS:[EAX+304]
```

其他几个用同样的方法找到。

指针的特征，

在`212r`中

例如：

```
0043F5FE   .  A1 BC564400   MOV EAX,DWORD PTR DS:[4456BC]
```

搜索常数`4456BC`

找到

```
00442BC4  /.  55            PUSH EBP

00442BC5  |.  8BEC          MOV EBP,ESP

00442BC7  |.  8B45 08       MOV EAX,DWORD PTR SS:[EBP+8]

00442BCA  |.  A3 BC564400   MOV DWORD PTR DS:[4456BC],EAX

00442BCF  |.  5D            POP EBP                                  ;  0012FE50

00442BD0  \.  C2 0400       RETN 4
```

，看看前后代码，熟悉`aspr`的都知道怎么找这个地址了，`DWORD PTR DS:[4456BC]=注册名的指针`，属于`pre-dip`可以轻松找到，他的下一段代码是时间限制，然后是过期提示等，

其中

```
00442C0D  |.  68 402C4400   PUSH ASPack.00442C40                     ;  ASCII "Expired"
```

这句有字符串很明显。

在`22`中

查找字符串 `Expired`，找到

```
00468255  |.  68 88824600   PUSH Doit_ASP.00468288                   ; |Title = "Expired"
```

向上

```
004681E0  /.  55            PUSH EBP

004681E1  |.  8BEC          MOV EBP,ESP

004681E3  |.  8B45 08       MOV EAX,DWORD PTR SS:[EBP+8]

004681E6  |.  A3 E8AC4600   MOV DWORD PTR DS:[46ACE8],EAX

004681EB  |.  8B45 0C       MOV EAX,DWORD PTR SS:[EBP+C]             ;  Doit_ASP.00441F8D

004681EE  |.  A3 ECAC4600   MOV DWORD PTR DS:[46ACEC],EAX

004681F3  |.  5D            POP EBP                                  ;  0012FE1C

004681F4  \.  C2 0800       RETN 8
```

时间限制，继续向上，找到了

```
004681D6  |.  A3 24C94600   MOV DWORD PTR DS:[46C924],EAX
```

感谢强大的脚本，这里面已经写好"`VolX`"的指针了，悄悄地改成自己的字串。

标志位：

在`212r`中

例如：

```
0043F65A   .  8B80 E4020000 MOV EAX,DWORD PTR DS:[EAX+2E4]

0043F660   .  C640 20 01    MOV BYTE PTR DS:[EAX+20],1
```

这个标志位和`2E4`控件有直接关系，细想一下应该是控件禁用，搜索常数 `2E4`，成功找到另一处类似代码：

```
0043F5C2   .  8B80 E4020000 MOV EAX,DWORD PTR DS:[EAX+2E4]

0043F5C8   .  C640 20 00    MOV BYTE PTR DS:[EAX+20],0
```

很巧这段代码就在解码`call`的前面。

在`22`中

之前修复了

```
0043F5C2   .  8B80 E4020000 MOV EAX,DWORD PTR DS:[EAX+2E4]
```

对应的

```
00465220      8B80 08040000 MOV EAX,DWORD PTR DS:[EAX+408]
```

好，搜索常数`408`，找到

```
00465188   .  8B80 08040000 MOV EAX,DWORD PTR DS:[EAX+408]

0046518E   .  C640 24 00    MOV BYTE PTR DS:[EAX+24],0
```

修正代码

```
00465226      C640 24 01    MOV BYTE PTR DS:[EAX+24],1
```
