
# movie play 2.51

> 原文：[`www.pediy.com/kssd/pediy02/forum253.htm`](https://www.pediy.com/kssd/pediy02/forum253.htm)

工具：`softice`、`offset converter`、`ultraedit`、`procdump`。

`offset converter` 的 URL: `http://kontum.vietmedia.com/protools/files/utilities/oc.zip`。

程式：`movie play 2.51`。

`http://www.titanic.cc/movplay/moveplay.htm`。

## 前言

看了那篇用 `self modifying code` 的技巧，我又试了一下其他的软体，以前拿不掉的 `nag screen` 现在可以掉了，真是太棒了。

再讲一下什么是 `self modifying code`，就是找一段原来的程式、所没有用到的程式码，加入你自己的 code，而且还能修改他的 code。我想也可以用这个来设变数吧。当我学到了这个技巧，我只能说，杰克，这真是太神奇了。

但是这个程式呢？我只是拿掉他的 `nag screen` 而已。其他的限制呢？我想对大家来说也不是问题吧，所以就不写了。

ok, go!!

## 步骤

### 1. 拦截 Dialog Box

程式一执行就出现一个 `nag screen`，叫你赶快去注册。的确讨厌，而今天所说的就是要拿掉它。

如何拿掉呢？

先离开 `movie play`，按 `ctrl-d` 进入 `softice`。

下 `bpx createdialogindirectparama`，这是在拦截 `dialogbox` 的。当然不只这个 API 啦，不过 `movie play` 是用这个。

按 `F5`，回到 `windows`，执行 `movie play`，立刻被拦住，按一下 `F12`，就回到 `movie play` 的领空了。

在这边：

```
CS:00420C33    CALL 00420897
CS:00420C38    CMP EAX,EDI <--- 出来光棒停在这边
CS:00420C3A    JZ 420C8E
```

此时 `eax = 1`，`edi = 0`，下意识的下 `r eax 0`。

将 `eax` 改成 `0`，按 `F5`，竟然 `nag screen` 不见了。

那我想大家一定想说，那还不简单，把 `jz` 改成 `jmp` 就好了啊。

可惜事情没有这麽简单，因为下 `bpx 00420c38`，会发现 `cs:00420c38` 会走两次。第一次，若 `eax = 0`，则 `nag` 会不见；第二次，若 `eax = 0`，则主视窗也离开了。

所以啦，不能这样改。不过 `cs:00420c33` 好像是关键喔。

在一次执行 `movie play` 被 `softice` 拦住，按一下 `F12`，就先下 `bd*`，在下 `bpx 00420c33`，按 `F5`，回到 `windows`，结束 `movie play`。

### 2. Trace 内部逻辑

执行 `movie play`，被 `softice` 拦住，停在 `cs:00420c33`。

按 `F8` 进去 trace，开始按数下 `F10`。

到了：

```
CS:00420AA5 SETNZ AL
CS:00420AA8 MOV ECX,[EBP-C]
CS:00420AAB POP EDI
CS:00420AAC POP ESI
```

当然，在 `CS:00420AA5` 这一行，若令 `AL = 1`，`nag screen` 就会出来；若令 `AL = 0`，`nag` 就不会出来了。

可是也不能直接修改，因为这边会走两次。若改这边，主程式也出不来了。

### 3. 寻找空间

那应该如何是好呢？就用 `self modifying code` 的技巧，加入自己的 code。

既然要加入自己的 code，首先就要找一段原来程式没有用到的程式码，而且又要足够空间可容纳我们所要加进去的程式码...那应该找哪里呢？

都好，只要不妨碍原来程式运作就行了。

### 4. 定位未使用的代码

用 `symbol loader` 载入 `movie play`。

一开始在进去都是一些 `invalid`，按一下 `F8`。

在这边：

```
CS:0041392E MOV EBP,ESP
CS:00413930 PUSH FF
CS:00413932 PUSH 00431A60
```

看一下上面的 CODE，好像一些上面的 code 没走到。

如：

```
CS:00413900 MOV EDI,[EBP+14]
CS:00413903 TEST EDI,EDI
```

下 `bpx 413900`，运行一下，试一下里面所有的功能，拨放一下。

真的没走到这边耶，那就准备改这边的值了，离开 `movie play`。

### 5. 注入跳板

执行 `movie play`。

到了：

```
CS:00420AA5 SETNZ AL
CS:00420AA8 MOV ECX,[EBP-C]
CS:00420AAB POP EDI
```

光棒停在 `cs:00420aa5` 时，此时 `AL=0`，但是只要一走完 `cs:00420aa5`，`AL=1` 了。

我们希望第一次走到这边 `AL=0`，第二次走到这边 `AL=1`，这是我们希望的，因为这样 `nag screen` 就会不见了。

先把附近的机械码抄起来，是 `0f95c08b4df45f5e`。

现在想把 `setnz al` 改成 `jmp 413900`。

所以下 `a`，就是组译了，在下 `jmp 413900`，按两次 `enter`，ok，改好了，抄一下机械码，是 `E9562EFFFF`。

现在按一下 `F10`，就跳到 `CS:413900` 了。

### 6. 编写修复代码

到了这边 `CS:413900` 当然也要改啊，要改成什麽呢？

首先要把 `CS:00420AA5` 恢复成 `SETNZ AL`，不然会有麻烦，但是要如何恢复呢？

下 `a`，在下：

```
mov dword ptr [420aa5],8b0c950f
mov byte ptr [420aa9],4d
jmp 00420aa8
```

按一下 `enter`（前两句各按一次，最后一句按两次）。

这就是我们所加入的码了，现在来分析一下，为什麽要下 `8b0c950f` 呢？仔细看一下原本刚刚抄下的机械码，`CS:00420AA5 SETNZ AL` 的机械码是 `0f95c08b`。

现在要恢复，当然是让他复原了，不过这个顺序要反过来，所以变成了 `8b0c950f`。

但是刚刚在 `CS:00420AA5` 把他改成 `jmp 413900` 是 5 byte，而 `mov dword ptr [420aa5],8b0c950f` 只是 4 byte 而已，所以还要再下一行 `mov byte ptr [420aa9],4d`，这样就恢复了。

再来就跳回去原来的地方，继续执行了。

而我改的这几行，的机械码也抄起来吧，满长的喔：

```
c705a50a
42000f95
c08bc605
a90a4200
4de992d1
0000
```

按一下 `F5`，就没问题了...已经去掉 `nag screen` 了。

### 7. 修改二进制文件

所以了，先关掉 `movie play`，拿起 `ultra edit` 去改了。

找 `0f95c08b4df45f5e`，改 `E9562EFFFF`。

再来呢？我好像是忽略了什麽了，糟了，我忘记抄下 `cs:413900` 附近的机械码了。当然可以再执行一次，再去抄下来。

不过这边我介绍另一个 tool，`offset converter`。

执行他，去开启 `movie play`，输入 `413900`，他就跑出来一个值了，是 `13900`，当然是 16 进位值。

所以用 `ultra edit` 去找到这个值，拉着卷轴，拉到了 `13900`，就可以开始改了。

改成：

```
c705a50a
42000f95
c08bc605
a90a4200
4de992d1
0000
```

这样就可以了。

### 8. 结语

不过这个软体有使用期限，这样做只是去掉 `nag screen` 而已，没有去掉其他限制，不过这也不是问题，这个就请大家自行去试了。

因为此篇只要讲的是 `self modifying code`。还是那句老话，觉得此软体好就去注册，不好用就杀掉吧，才能有更好的软体，只有注册，才能有更好的软体。

`4dd466d0e1e0cc123afbe504da2462ca`
