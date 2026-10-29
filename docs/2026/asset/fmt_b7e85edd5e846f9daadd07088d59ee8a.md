
# [原创]使用unidbg还原标准ollvm的fla控制流程平坦化

最近网上查了查还原平坦化相关的资料。看了无名侠大佬的文章。由于本人太菜了。好多地方看的有点懵懵懂懂的。最终还是自己摸索着尝试还原平坦化。写的哪里不对。还请大佬指正。

先贴上几位大佬的文章

[ARM64 OLLVM反混淆][https://bbs.pediy.com/thread-252321.htm]

[利用符号执行去除控制流平坦化][<mark class="encrypted">23cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6k6h3y4#2M7X3W2@1P5g2)9J5k6i4c8W2L8X3y4W2L8Y4c8Q4x3X3g2U0L8$3#2Q4x3V1k6A6L8X3c8W2P5q4)9J5k6i4m8Z5M7q4)9J5c8X3u0D9L8$3N6Q4x3V1k6E0M7$3N6Q4x3V1j5I4x3e0u0Q4y4f1b7`.</mark>

翻阅过大佬的文章后。本菜菜得到两个简单的结论。

**一、fla主要功能是将if else这类的逻辑给转换成while+switch组合来处理。达到将简单的逻辑给复杂化，增加静态分析难度的目的。**

**二、模拟执行找出真实块之间的关联，patch修改相应的代码。直接将真实块关联起来，从而实现反混淆**

接下来。我将采用先射箭后画靶的方式。来逐步的还原一个标准的ollvm fla的案例。

首先是准备我们的案例，如下是未混淆前的源码

```c++
std::string calcKey(std::string data){

if (data.length()>10){

data="ceshi";

}else if(data.length()>30){

data="ollvm";

}else{

data="fla";

}

return data;

}

extern "C" JNIEXPORT jstring JNICALL

Java\_com\_example\_ollvmdemo2\_MainActivity\_stringFromJNI(

JNIEnv \*env,

jobject /\* this \*/) {

std::string hello = "Hello from C++";

hello=calcKey(hello);

return env->NewStringUTF(hello.c\_str());

}
```

然后是混淆的参数配置是`add\_definitions("-mllvm -fla -mllvm -split -mllvm -split\_num=3")`

贴上混淆后的样本：链接: <mark class="encrypted">1ffK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6H3j5h3&6Q4x3X3g2T1j5h3W2V1N6g2)9J5k6h3y4G2L8g2)9J5c8Y4y4Q4x3V1j5I4c8r3b7H3g2o6b7&6P5r3S2K6K9X3N6i4M7V1A6%4y4#2m8e0g2q4q4r3b7b7`.`.</mark> 密码: rrem

然后贴上ida解析出来的代码

```c++
unsigned \_\_int64 \_\_usercall calcKey@<X0>(unsigned \_\_int64 result@<X0>, \_\_int64 a2@<X8>)

{

signed int v2; // w8

signed int v3; // w8

unsigned \_\_int64 v4; // [xsp+10h] [xbp-40h]

\_\_int64 v5; // [xsp+18h] [xbp-38h]

signed int v6; // [xsp+24h] [xbp-2Ch]

bool v7; // [xsp+3Fh] [xbp-11h]

unsigned \_\_int64 v8; // [xsp+40h] [xbp-10h]

bool v9; // [xsp+4Fh] [xbp-1h]

v6 = 910266824;

v5 = a2;

v4 = result;

while ( v6 != -1929161090 )

{

switch ( v6 )

{

case -1578842400:

v6 = 56578246;

break;

case -1521928633:

v9 = v8 > 0x1E;

v6 = -124633339;

break;

case -124633339:

if ( v9 )

v3 = 1872828810;

else

v3 = 1744272424;

v6 = v3;

break;

case 56578246:

v6 = 1089662144;

break;

case 256858465:

if ( v7 )

v2 = 1938015978;

else

v2 = 1363893773;

v6 = v2;

break;

case 910266824:

v6 = 2056492010;

break;

case 1089662144:

result = sub\_F8E4(v5, v4);

v6 = -1929161090;

break;

case 1363893773:

result = sub\_F77C(v4);

v8 = result;

v6 = -1521928633;

break;

case 1493239651:

v6 = 1089662144;

break;

case 1697837166:

v6 = 56578246;

break;

case 1744272424:

result = sub\_F830(v4, "fla");

v6 = 1697837166;

break;

case 1872828810:

result = sub\_F830(v4, "ollvm");

v6 = -1578842400;

break;

case 1938015978:

result = sub\_F830(v4, "ceshi");

v6 = 1493239651;

break;

case 2056492010:

result = sub\_F77C(v4);

v7 = result > 0xA;

v6 = 256858465;

break;

}

}

return result;

}
```

我们对比看下混淆前和混淆后的代码。原本我们只需要看if条件就知道意义的。但是现在就需要不停的顺着条件。进入case一步步的分析每一步。如果是这么简单的例子。我们还是可以静态分析看的出来真正的条件。但是代码量过于庞大时就很难跟踪分析了。

## 思路：先射箭后画靶

先简单说下如何先射箭后画靶，上面这个例子的逻辑比较简单。初学的情况，我们可以直接静态分析下，找出所有的真实块。然后用ida的keypatch插件来手动将真实块关联起来。再用ida解析看看结果是否和我们的混淆前的一致。结果一致后。我们就可以开始写unidbg代码来根据特征匹配真实块。特征匹配的结果可以和我们静态分析的结果对比看是否一致。最后只要unidbg实现出我们手动关联的效果，就完成了自动化反混淆了。下面我就先手动的还原。

## 一、手动还原

1、找出所有真实块以及对应的汇编地址，标准的ollvm虚假块中一般只有简单的修改v6的值，其他的基本都是真实块

```c++
case -1521928633: //0xF694
v9 = v8 > 0x1E;
v6 = -124633339;
break;

case -124633339: //0xF6BC
if ( v9 )
v3 = 1872828810;
else
v3 = 1744272424;
v6 = v3;
break;

case 256858465: //0xF624
if ( v7 )
v2 = 1938015978;
else
v2 = 1363893773;
v6 = v2;
break;

case 1089662144: //0xF750
result = sub\_F8E4(v5, v4);
v6 = -1929161090;
break;

case 1363893773: //0xF678
result = sub\_F77C(v4);
v8 = result;
v6 = -1521928633;
break;

case 1744272424: //0xF710
result = sub\_F830(v4, "fla");
v6 = 1697837166;
break;

case 1872828810: //0xF6E0
result = sub\_F830(v4, "ollvm");
v6 = -1578842400;
break;

case 1938015978: //0xF648
result = sub\_F830(v4, "ceshi");
v6 = 1493239651;
break;

case 2056492010: //0xF5F8
result = sub\_F77C(v4);
v7 = result > 0xA;
v6 = 256858465;
break;
```

找出所有真实块的地址后。接着就是顺着逻辑将他们全部串联起来。这里我举两个比较典型的例子。先从函数开始的地方开始。下面简单整理下第一个真实块的关联

根据`v6=910266824`第一次赋值。我们锁定到下面的case分支

```c++
unsigned \_\_int64 \_\_usercall calcKey@<X0>(unsigned \_\_int64 result@<X0>, \_\_int64 a2@<X8>)

{

v6 = 910266824;

v5 = a2;

v4 = result;

while ( v6 != -1929161090 )

{

switch ( v6 )

{

....
case 910266824:
v6 = 2056492010;
break;
....
case 2056492010: //0xF5F8 第一个真实块
result = sub\_F77C(v4);
v7 = result > 0xA;
v6 = 256858465;
break;
....
case 256858465: //0xF624 第二个真实块
if ( v7 )
v2 = 1938015978;
else
v2 = 1363893773;
v6 = v2;
}
}
return result;
}
```

根据上面的代码。我们第一个真实块。应该是直接跳转到`0xF5F8`这个位置。而第一个跳转的位置。应该是在while循环前。先贴上第一个block的汇编代码

```
.text:000000000000F470 loc\_F470
.text:000000000000F470 LDR W8, [SP,#0x50+var\_2C]
.text:000000000000F474 MOV W9, #0x567E
.text:000000000000F478 MOVK W9, #0x8D03,LSL#16
.text:000000000000F47C CMP W8, W9
.text:000000000000F480 STR W8, [SP,#0x50+var\_44]
.text:000000000000F484 B.EQ loc\_F76C
.text:000000000000F488 B loc\_F48C
```

我们可以直接第一句就跳转到真实指令。因为这里使用的while是fla混淆的所以这些数据我们并不需要去执行。修改代码如下

```
.text:000000000000F470 loc\_F470
.text:000000000000F470 LDR W8, [SP,#0x50+var\_2C]
.text:000000000000F474 B loc\_F5F8 ; Keypatch modified this from:
.text:000000000000F474 ; MOV W9, #0x567E
.text:000000000000F478 ; ---------------------------------------------------------------------------
.text:000000000000F478 MOVK W9, #0x8D03,LSL#16
.text:000000000000F47C CMP W8, W9
.text:000000000000F480 STR W8, [SP,#0x50+var\_44]
.text:000000000000F484 B.EQ loc\_F76C
.text:000000000000F488 B loc\_F48C
```

这样第一个块就关联好了。然后继续关联第二个真实块`0xF624`。下面先列出第一个真实块的汇编

```
.text:000000000000F5F8 loc\_F5F8
.text:000000000000F5F8

.text:000000000000F5F8 LDR X0, [SP,#0x50+var\_40]
.text:000000000000F5FC BL sub\_F77C
.text:000000000000F600 CMP X0, #0xA
.text:000000000000F604 CSET W8, HI
.text:000000000000F608 MOV W9, #1
.text:000000000000F60C AND W8, W8, W9
.text:000000000000F610 STURB W8, [X29,#var\_11]
.text:000000000000F614 MOV W8, #0x5961
.text:000000000000F618 MOVK W8, #0xF4F,LSL#16
.text:000000000000F61C STR W8, [SP,#0x50+var\_2C]
.text:000000000000F620 B loc\_F778
```

汇编的最后是跳转到了`0xf778`。这里就是又进入控制分发器了。我们直接让他跳转到第二个真实块。修改最后一行汇编如下

```
.text:000000000000F618 MOVK W8, #0xF4F,LSL#16
.text:000000000000F61C STR W8, [SP,#0x50+var\_2C]
.text:000000000000F620 B loc\_F624 ; Keypatch modified this from:
.text:000000000000F620 ; B loc\_F778
```

第二个真实块这里就有点特殊了。这是一个分支让我们的第三个真实块可能是`0xF648`位置也可能是`0xF678`位置。我们先看看c++的部分

```c++
case 256858465:
if ( v7 ) //根据条件让第三个真实块不固定了
v2 = 1938015978;
else
v2 = 1363893773;
v6 = v2;
```

那么这里我们就不能简单的b跳转哪个真实块了。这里我的预想是把这块的代码修改成如下

```c++
case 256858465:
if ( v7 ) //根据条件让第三个真实块不固定了
//直接跳转到0xF648
else
//直接跳转到0xF678
v6 = v2;
```

但是我们肯定不能修改c++的代码。所以看看这个真实块的汇编部分

```
.text:000000000000F624 loc\_F624
.text:000000000000F624

.text:000000000000F624 LDURB W8, [X29,#var\_11]
.text:000000000000F628 MOV W9, #0xC6EA
.text:000000000000F62C MOVK W9, #0x7383,LSL#16
.text:000000000000F630 MOV W10, #0x5E0D
.text:000000000000F634 MOVK W10, #0x514B,LSL#16
.text:000000000000F638 TST W8, #1
.text:000000000000F63C CSEL W8, W9, W10, NE
.text:000000000000F640 STR W8, [SP,#0x50+var\_2C]
.text:000000000000F644 B loc\_F778
```

这里可以看到那个`v7`的值就是`w8`。所以我们修改成判断`w8=1`。就跳转到`0xf648`。否则跳转到`0xf678`。下面贴上修改后的结果

```
.text:000000000000F624 loc\_F624
.text:000000000000F624

.text:000000000000F624 LDURB W8, [X29,#var\_11]
.text:000000000000F628 MOV W9, #0xC6EA
.text:000000000000F62C MOVK W9, #0x7383,LSL#16
.text:000000000000F630 MOV W10, #0x5E0D
.text:000000000000F634 MOVK W10, #0x514B,LSL#16
.text:000000000000F638 CMP W8, #1 ; Keypatch modified this from:
.text:000000000000F638 ; TST W8, #1
.text:000000000000F63C B.EQ loc\_F648 ; Keypatch modified this from:
.text:000000000000F63C ; CSEL W8, W9, W10, NE
.text:000000000000F640 B loc\_F678 ; Keypatch modified this from:
