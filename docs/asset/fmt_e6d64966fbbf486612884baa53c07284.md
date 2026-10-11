
# 深思 3 乱弹(五)

> 原文：[`www.pediy.com/kssd/pediy05/pediy50263.htm`](https://www.pediy.com/kssd/pediy05/pediy50263.htm)

**深思 3 乱弹(五）**

作者：紫竹

转载：大老

我也知道这些问题比较“擦边”，不过，希望能有好人出来指点一二了。；-）

## 问题一

紫竹在《深思 3 乱弹》一文中写到：深思 3 的数据段是可以完全读出来的。可是，我想了很久，不知道要如何才能读出数据段来，通过标准的 `Sense3()`？可是这个函数没有读取数据的功能，只有执行程序的功能呀~！

## 问题二

猜出代码段的内容——这里头有什么技巧没有？是不是还要自己编程序去读一下狗，才能猜出来的。

## 问题三

`Sense3.dll` 中，除开 `Sense3()` 以外，还有 `Sense3Ex()` 函数，据我分析，该函数属于开发级的调用接口，可是我搞不清楚它的入口参数的意义，有哪位朋友研究出来过么？

这三个问题实在太大了，这里先回答第一个问题，另外两个问题，以后找时间再答。

### 问题一解答

读深思 3 的数据，依赖于狗内的代码。

说到读，就不能不说写，因为读写是互逆的操作。

因为大部分加密者很懒，都用自动生成的代码，所以，我们先来看看深思 3 自动生成的代码是什么样的：

#### 1、简单随机度：

```
;  (C) Copyright 1998 SenseLock Technologies Inc.
developerpass\*,\*,\*    ；开发口令
userpass111,222,333  ；用户口令
casesensitive1
code1segmentcode              ；代码段开始
Encrypt:subbx1,ax1  ；数据变换
xchgax0,bx1
rorax0,8
xchgax0,bx1
addax1,bx1
xchgax0,ax1
rorax0,3
xchgax0,ax1
addbx1,ax1
xchgax0,bx1
rolax0,8
xchgax0,bx1
subax1,bx1
xchgax0,ax1
rolax0,8
xchgax0,ax1
ret4
WriteMemory:xchgbx1,\[ax1\]  ；写数据
ret4
ReadMemory:movbx1,\[ax1\]  ；读数据
ret4
Decrypt:xchgax0,ax1    ；数据逆变换
rorax0,8
xchgax0,ax1
addax1,bx1
xchgax0,bx1
rorax0,8
xchgax0,bx1
subbx1,ax1
xchgax0,ax1
rolax0,3
xchgax0,ax1
subax1,bx1
xchgax0,bx1
rolax0,8
xchgax0,bx1
addbx1,ax1
ret4
code1ends                   ；代码段结束
data1segmentdata                ；数据段开始
dw0
dw1
dw2
data1ends                   ；数据段结束
```

我们看到，整个代码分为四段，有两个互为逆运算的数据变换段，一个读数据段，一个写数据段。我们要研究的是读数据，所以，只看读、写两个段：

```
ReadMemory:movbx1,\[ax1\]  ；读数据
ret4
WriteMemory:xchgbx1,\[ax1\]  ；写数据
ret4
```

他们共同的特点是：只有两行，并且第二行都是返回。

先看读数据，需要输入两个数据，`bx1,ax1`，把地址为 `ax1` 的数据区的数据给 `bx1`，返回的数据中，`ax1` 不变，`bx1` 变。`bx1` 就是地址为 `ax1` 的数据的值。

再看写数据，需要输入两个数据，`bx1,ax1`，把地址为 `ax1` 的数据区的数据与 `bx1` 的值交换，返回的数据中，`bx1,ax1` 交换了位置，并且 `bx1` 的数值变化了。

如果我们手里有一个软件和它的狗，如何找到读数据的代码？

用静态反汇编工具软件，把软件反汇编，找到所有对狗操作的地方，可以把所有狗内的代码段地址找到，其中代码只有两行的代码段，就是读、写代码段了。然后再来区分读、写。

进入动态跟踪调试工具（如 `TRW2000` 等），跟踪这两个代码段，看返回数据的变化，按前面的分析，就可以区分出读、写了。

找到了读数据的代码地址，就可以利用原软件或者自己编一个软件来读出狗内的数据了。

#### 2、普通随机度：

```
;  (C) Copyright 1998 SenseLock Technologies Inc.
developerpass\*,\*,\*      ；开发口令
userpass111,222,123    ；用户口令
casesensitive1
code1segmentcode             ；代码段开始
ret4
Encrypt:subax1,bx1    ；数据变换
addbx1,ax1
subbx1,ax1
addax1,bx1
jmpLabel1
ReadMemory:movax1,\[bx1\]  ；读数据
Label1:jmpLabel2
Decrypt:subax1,bx1    ；数据逆变换
addbx1,ax1
subbx1,ax1
addax1,bx1
Label2:jmpLabel3
WriteMemory:xchgax1,\[bx1\]  ；写数据
Label3:movbx0,rand             ；随机干扰
xchgax0,bx0
clrbx0
subbx1,ax0                   ；通讯变换入口
xchgax1,bx1
addbx1,ax0
xchgbx1,ax1
subax0,bx1
xchgax0,ax1
rolax0,3
xchgax1,ax0
xchgax1,bx1
addbx1,ax0
xchgbx1,ax1
xchgax0,bx1
rolax0,4
xchgbx1,ax0
xorbx1,ax0
xchgax0,bx1
xorbx1,ax1
xchgbx1,ax0
subbx1,ax1
xchgax0,bx1
xorbx1,ax1
xchgbx1,ax0
xorax1,bx1
jmpbx0
code1ends                       ；代码段结束
data1segmentdata               ；数据段开始
dw64dup(0)
data1ends                       ；数据段结束
```

我们看到，整个代码分为五段，比简单随机度多了一个通讯变换段（通讯变换段前面还有三行随机干扰，不算一个单独的段），有两个互为逆运算的数据变换段，一个读数据段，一个写数据段。我们要研究的是读数据，所以，只看读、写两个段：

```
WriteMemory:xchgax1,\[bx1\]  ；写数据
ReadMemory:movax1,\[bx1\]  ；读数据
Label1:jmpLabel2
```

他们的共同特点是，一般有两行组成，如果是在通讯变换前面的，则只有一行。第二行是转移语句。第一行与前面介绍的简单随机度相同，所以，这里就不在细说了。

开头说了，一般的加密者很懒，往往用自动生成的代码。如果不用自动生成的代码，又如何呢？

例如，代码段是这样的：

```
mov ax0, bx1
mov bx0, ax1
mov ax1, [bx0]
mov bx1, [ax0]
add ax1, bx1
add bx1, ax1
ret 4
```

这实际是每次读两个数据，然后再进行简单计算，然后返回。这样，就不可能直接读出数据区的数据了。这就需要猜出代码来，如果知道代码，其实不是很简单嘛？至于如何猜到狗内的代码，我准备在下一个问题里回答。

---

oicq:79234668

dalao@qdcnc.com

http://dalao2002.yeah.net  4dd466d0e1e0cc123afbe504da2462ca
