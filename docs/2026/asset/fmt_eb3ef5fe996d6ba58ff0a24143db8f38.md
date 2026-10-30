
# [原创] 看雪.TSRC 2017CTF秋季赛第二题——只缘身在此山中，不识庐山真面目

写在前面：最近特忙，不太可能每题都有空参加，能做一题是一题了，文章争取写清楚，论坛有些文字乱码我也不知道怎么办。

下载地址：[84eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0N6r3k6Q4x3X3g2H3k6h3c8A6P5g2)9J5k6h3y4G2L8g2)9J5c8X3q4@1N6r3q4U0K9q4)9J5k6r3c8G2N6$3&6D9L8$3q4V1i4K6u0V1z5o6k6Q4x3X3g2Z5N6r3@1`.](https://bbs.kanxue.com/elink@99fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6U0N6r3k6Q4x3X3g2H3k6h3c8A6P5g2)9J5k6h3y4G2L8g2)9J5c8X3q4@1N6r3q4U0K9q4)9J5k6r3c8G2N6$3&6D9L8$3q4V1i4K6u0V1z5o6k6Q4x3X3g2Z5N6r3@1`.)

## 一、侦查

无壳 Microsoft Visual C/C++(6.0)[libc]，有明确提示成功和失败信息。

## 二、出坑

OD加载，查找字符，定位到：“You get it!”

```
地址       反汇编                                    文本字符串

00401000   PUSH ctf2017_.0041B06C                    \n Crackme for CTF2017 @Pediy.\n

0040102F   PUSH ctf2017_.0041B05C                    You get it!\n

0040103F   PUSH ctf2017_.0041B038                    Bad register-code, keep trying.\n

00401053   PUSH ctf2017_.0041B0AC                     Coded by Fpc.\n\n

00401060   PUSH ctf2017_.0041B090                     Please input your code:

00401072   PUSH ctf2017_.0041B08C                    %s

```

```
00401000  /$  68 6CB04100   PUSH ctf2017_.0041B06C                   ;  \n Crackme for CTF2017 @Pediy.\n

00401005  |.  E8 382D0100   CALL ctf2017_.00413D42

0040100A  |.  83C4 04       ADD ESP,0x4

0040100D  |.  C705 34B04100>MOV DWORD PTR DS:[0x41B034],0x2          ;  2

00401017  |.  E8 34000000   CALL ctf2017_.00401050                   ;  input

0040101C  |.  E8 6F000000   CALL ctf2017_.00401090                   ;  check1

00401021  |.  E8 BA000000   CALL ctf2017_.004010E0                   ;  check2

00401026  |.  A1 34B04100   MOV EAX,DWORD PTR DS:[0x41B034]          ;  0

0040102B  |.  85C0          TEST EAX,EAX

0040102D  |.  75 10         JNZ Xctf2017_.0040103F

0040102F  |.  68 5CB04100   PUSH ctf2017_.0041B05C                   ;  You get it!\n

00401034  |.  E8 092D0100   CALL ctf2017_.00413D42

00401039  |.  83C4 04       ADD ESP,0x4

0040103C  |.  33C0          XOR EAX,EAX

0040103E  |.  C3            RETN

0040103F  |>  68 38B04100   PUSH ctf2017_.0041B038                   ;  Bad register-code, keep trying.\n

00401044  |.  E8 F92C0100   CALL ctf2017_.00413D42

00401049  |.  83C4 04       ADD ESP,0x4

0040104C  |.  33C0          XOR EAX,EAX

0040104E  \.  C3            RETN

0040104F      90            NOP

00401050  /$  83EC 0C       SUB ESP,0xC

00401053  |.  68 ACB04100   PUSH ctf2017_.0041B0AC                   ;   Coded by Fpc.\n\n

00401058  |.  E8 E52C0100   CALL ctf2017_.00413D42

0040105D  |.  83C4 04       ADD ESP,0x4

00401060  |.  68 90B04100   PUSH ctf2017_.0041B090                   ;   Please input your code:

00401065  |.  E8 D82C0100   CALL ctf2017_.00413D42

0040106A  |.  83C4 04       ADD ESP,0x4

0040106D  |.  8D4424 00     LEA EAX,DWORD PTR SS:[ESP]

00401071  |.  50            PUSH EAX

00401072  |.  68 8CB04100   PUSH ctf2017_.0041B08C                   ;  %s

00401077  |.  E8 F72C0100   CALL ctf2017_.00413D73

0040107C  |.  8D4424 08     LEA EAX,DWORD PTR SS:[ESP+0x8]

00401080  |.  83C4 14       ADD ESP,0x14

00401083  \.  C3            RETN

```

简单跟踪几步发现`401090`、`4010E0`为显性算法比较，分析`401090`、`4010E0`：

```
void sub_401090()

{
  int l32; // [esp+4h] [ebp-8h]

  int h32; // [esp+8h] [ebp-4h]



  if ( h32 && l32 && h32 != l32 && 5 * (h32 - l32) + h32 == 0x8F503A42 && 0xD * (h32 - l32) + l32 == 0xEF503A42 )

    --dword_41B034;

}



void sub_4010E0()

{
  int l32; // [esp+4h] [ebp-8h]

  int h32; // [esp+8h] [ebp-4h]



  if ( h32 && l32 && h32 != l32 && 0x11 * (h32 - l32) + h32 == 0xF3A94883 && 7 * (h32 - l32) + l32 == 0x33A94883 )

    --dword_41B034;

}

```

把输入每4个字符ASCII作为十六进制赋值计算，假设为`a`、`b`则：

⑴：`0x5 * ( a - b ) + a == 0x8F503A42`

⑵：`0xD * ( a - b ) + b == 0xEF503A42`

⑶：`0x11 * ( a - b ) + a == 0xF3A94883`

⑷：`0x7 * ( a - b ) + b == 0x33A94883`

考虑该显性算法未随机打乱数值，低位特征依然继承到计算结果，因此先尝试求解低位，再逐个往高位求解，避免穷举范围过大：

```
def solve_false():

	start_time = time.clock()

	for a1 in range(0x30,0x7b):

		for b1 in range(0x30,0x7b):

			if (0x5 * (a1 - b1) + a1) & 0xff == 0x42:

				if (0xd * (a1 - b1) + b1) & 0xff == 0x42:

					if (0x11 *(a1 - b1) + a1) & 0xff == 0x83:

						if (0x7 * (a1 - b1) + b1) & 0xff == 0x83:

							print ("found sn a1 : %x" % a1)

							print ("found sn b1 : %x" % b1)

	print ('use time: %.3f second' % (time.clock()-start_time))

	return

```

迅速识别该方程组无合法答案，无疑是作者埋的坑，立即寻找其他途径。

## 三、溢出

测试不同输入长度，发现当输入15个字符时就已经可以覆盖`CALL 00401050`的返回地址，将断点设在`401083`观察：

比如输入：123456781111AAA

```
0018FF3C  34333231

0018FF40  38373635

0018FF44  31313131

0018FF48  00414141  ctf2017_.00414141

esp：

0018FF48   00414141  ctf2017_.00414141
```

返回地址覆盖为：`0x414141`“AAA”，因此作者设置了此处溢出可以根据输入字符转移流程，达到隐蔽算法目的。这里我们可以测试（复制到验证界面）：

`123456781111/@`

可以显示成功，但因比赛规则要求防守方和攻击方都得遵守合法字符的要求，因此出现特殊字符不被认可，我们继续分析。

## 四、破解

上文已经做好了一切破解的准备，因合法字符要求，现在我们寻找`0x413030-0x417a7a`之间的可疑过程或函数入口，删除OD模块分析：

很快就看到了`0x413131`这个位置很可疑，花指令一大堆，此处无银三百两嘛，对应溢出字符“11A”，现在我们输入“Just0for0fun11A”（为避免啰嗦，直接用正确注册码阐述）:

```
a：0x7473754A "Just"

b：0x726F6630 "0for"

c：0x6E756630 "0fun"

```

取消前文所有断点，在`0x413131`位置设置新断点，回车断下，忽略花指令，仅摘录算法相关代码：
