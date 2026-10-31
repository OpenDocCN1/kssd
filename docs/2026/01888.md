
# 谈谈关于如何破解`OicqPassOver.exe` Ver1.14的问题

(4千字)

## 一、如何注册`OicqPassOver.exe` Ver1.14

### 1. 如果你对这个软件没什么兴趣的话，按如下方法注册即可

在`HKEY_CURRENT_USER\Software\Intel`新建字符串值`speed`内容为`102717715`，再新建字符串值`cpu`内容为`BB901AC1`，其中`cpu`的内容会随`speed`的内容不同而变化。

### 2. 如果你对他很感兴趣的话，分析一下他的注册流程

简要方法：

```
017f:00451d41 lea eax,[ebp-04]
017f:00451d44 call 00407b9c
017f:00451d49 mov edx,[ebp-04]
017f:00451d4c pop eax      ------>d eax
017f:00451d4d call 00403c3c
017f:00451d52 jnz 004551ee0
```

首先选注册，输入`78787878`，后按`ctrl+d`进入`softice`，执行`s 30:0 l ffffffff '78787878'`，后`bpm`搜到的地址，按`f5`返回，点确定后被`softice`拦到，下`bd *`，按数次`f12`进入`OicqPassOver`的领空，按`f10`跟踪到上面所写的地方，当程序执行到`00451d4c`处时，用`d eax`，上下翻页看一下内存的话，`speed`值，`cpu`值，密码值写在一起，很容易找到，当程序执行到`00451d52`时，如果密码正确不跳，否则跳，跟踪完毕。

注册的话，填正确的密码，强制不跳，都能注册，但程序会在`windows\system`下放一个叫`kbd.386`的文件，据说这个玩艺会使使用者当机，我没碰到过当机，不知是怎么回事:(，可是不管你怎么注册，他都会放的，我只有继续跟踪了。

## 二、如何跳过程序的“暗柱”

我们继续跟踪到

```
017F:00451DB5 MOV EDX,00451E78
017f:00451dba mov edx,esi
017f:00451dbc mov ecx,[eax]
017f:00451dbe call [eax+34]
017f:00451dc1 dec ebx
017f:00451dc2 jnz 00451d85  ->jmp 00451e0f
```

这个`jnz`会在刚才`d eax`所显示内存处的下面连续写`hahaha.....!`，就是`kbd.386`的内容，我没有仔细分析，直接跳到`00451e0f`的话会提示注册成功，且不会在`windows\system`下放`kbd.386`（`jmp 00451e0f`是试出来的，在此省略测试若干，由于我没细分析，有毛病的话，就应该是`jmp`到哪的问题了）

## 三、如何跳过"路漫漫"的问题

这个问题我的能力解决不了，不过我参考了一下

`iCEbIRD [CCG]`，[CCG] stands for cHINA cRACKING gROUP

`http://nemc.yeah.net/`

`icebird_crack@21cn.com`

的破`OICQ密码终结者 1.13`的关于消除`nag screen`的部分，得到完满的解决。

好我们开始(参考部分原文)

因为`soft-ice`无法查找中文，用`winhex`查找"路漫漫"（他用的是`ultra-edit`，我习惯用`winhex`），把他改为`cracking`，然后执行`OicqPassOver.exe`，进入后在`softice`中下`bpx destroywindow`，点确定，被拦到，按`f12`到`OicqPassOver`的领空后，下`s 30:0 l ffffffff 'cracking'`，`bpm`搜到的地址，退出`OicqPassOver`，再次执行`OicqPassOver.exe`，被拦到，下`bd *`，按`f12`直到进入`OicqPassOver`的领空，按住`f10`直到回到`OicqPassOver`中，点确定后又被拦到，看一下是哪一个`call`调的

```
017f:0044b14d call [edx+cc]
```

由于这是一个公共的`call`，我们不能直接把它给`NOP`掉，所以在程序中加入一段子程序，在`44b14d`处首先`CALL`我的子程序进行判断，看是出现那个有诗句的`nag screen`还是其它。那么我的`code`要放在哪里呢？因为程序的入口在`.CODE section`，所以用`winhex`打开文件，在`.CODE section`中找可以存放我的`code`的地方，我找到了这里：(这一段几乎抄的原文,下面是关键)

```
000562e3 e8 bb f5 fe ff 5b e8 a1 c8 fa ff 90 00 00 00  <- end of .CODE
section
000562f2 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00  <- here save
my code
```

解释一下我理解的上面的查找方法:抄下`softice`跟踪到的机器码,比如`017f:00451d52 jnz 004551ee0`的机器码是`0f85c8000000`,在`winhex`中查找之,找到后,按`pagedown`向下翻页直到又连续的`000000000000000000000......`处(在`hex values`中),最明显的标志是`text`栏中`runtime error`的上面(我的表达能力太差了:(,sorry)

我决定在`562f2`处放我的`code`，所以先在这里做一个`mark`，键入`mycode`.

再次运行程序，这次在`44b14d`处拦下后就下`s 30:0 l ffffffff 'mycode'`，找到了内存地址`456ee3`，

此时还要记下两样数据：`[EDX+CC]==>443608`，`EDI==>0`

现在可以开始改了，下`A 44b14d`，然后下`CALL 456ee3`，下`nop`，回车,程序变成了这样

```
017f:00448125 0f8450bcee78  call 456ee3
017f:00448126 90            nop
```

接下来就下`G 456ee3`，到了`456ee3`后就下`A`，然后下这些指令：

```
xxxx:xxxxxxxx 83ff00        CMP EDI,00
xxxx:xxxxxxxx 0f846742ffff    JZ        0044b153
xxxx:xxxxxxxx e917c7feff    JMP      00443608
```

完成.

## 四、做成release版的`OicqPassOver.exe`密码终结者

### 1. 注册号随便填

- 查找：`0f85c8000000b201`
- 改为：`909090909090b201`

### 2. 跳过程序的“暗柱”

- 查找：`ff51344b75f1`
- 改为：`ff51344beb4b`

### 3. 跳过"路漫漫....."

- 查找：`8b10ff92cc0000008945f8`
- 改为：`8b10e891bd0000908945f8`

- 查找：`6D79636F6465????????????????`
- 改为：`83ff000f846742ffffe917c7feff`

(只有填`mycode`时,这样找,或直接用`winhex`在`000562f2`处写入`83ff000f846742ffffe917c7feff`)

完成.

注：

1. 没有经过安全测试,有什么副作用未知:)
2. 我用的是`winhex9.52`,如果你没有注册号的话,按如下方法改可以免注册:)

## 附：`winhex` ver9.52免注册

`*[45759d]` 在这强制置1的话,程序会认为你已经注册

```
017f:00450041 jnz 00450064 -> nop
017f:00450062 jz 00450068  -> jmp 00450068
017f:0043e31b jl 0043e372  -> nop
017f:0043e36b jz 0043e37d  -> jmp 0043e37d
7521B85C524500 -> 9090B85C524500
3B027404 -> 3B02eb04
7C55A13C764500 -> 9090A13C764500
741083F954 -> eb1083F954
```

cracked by 电神魔鬼

thanks for your supports
