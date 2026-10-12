
# 南极星 2.23

> 原文：[`www.pediy.com/kssd/pediy03/forum328.htm`](https://www.pediy.com/kssd/pediy03/forum328.htm)

查找过程很简单：

首先进入南极星，点 `register`，下 `bpm lockmytask`，点 `ok`，被拦到，按 `f12` 直到回到 `njcom` 的领空，往回倒到 `cmp byte ptr [06f4],00`，`[06f4]`为 1 时，表示已注册。关掉南极星，在 `s-ice` 中下 `bp hmemcpy`，启动 `njcom`，被拦到，按 `f12` 直到 `njcom` 领空，下 `bpm ds:06f4`，按 `f5` 被拦到：

```
cs:3bfc 32c0 ->b001  xor al,al ->mov al,01

cs:3bfe a2f506      mov [06f5],al

cs:3c01 a2f406      mov [06f4],al
```

好相应找到 `32c0` 改为 `b001`，但并不这么简单。用 `ws32dasm` 反汇编，把对应的地址改成如下的就搞定了 :)

```
0001.428 ff7408      push word ptr [si+08]

0001.42b e95701      jmp 0585

.

.

.

0001.585 1e          push ds

0001.585 84c9        test cl,cl

0001.588 751a        jne 05a4

0001.58a 8e5eb8      mov ds,[bp-48]

0001.58d 3ec706fc3bb001 mov word ptr ds:[3cfc],01b0 在这注册

0001.594 3ec70607479090 mov word ptr ds:[4707],9090 在这跳出死循环

0001.59b 3ec7060c47eb1e mov word ptr ds:[470c],1eeb 在这跳出死循环

0001.5a2 90          nop

0001.5a3 90          nop

0001.5a4 1f          pop ds

0001.5a5 ff76b8      push word ptr [bp-48]

0001.5a8 e983fe      jmp 042e
```

用 `winhex` 整理一下

查找：`7408ff76b86a01`

改成：`7408e957016a01`

查找：`c8000000c9ca020000436f707972696768742028632920313939332054616c204e65766f2e00`

改成：`1e84c9751a8e5eb83ec706fc3bb0013ec706074790903ec7060c47eb1e90901fff76b8e983fe`

接着在 `C:\WINDOWS\NJCOM.INI` 加上以下语句

```
[NJCOM Registration]

User_Name=你的名字

Company_Name=你的公司

Serial_Number=随便
```

完活

应该没什么问题了，有问题的话告诉我

cracked by 电神魔鬼

oicq:1789655

thanks for your supports

做完后，可正常使用，但不要向后调系统时钟，会过期的。解决方法是：打开 `c:\windows\njcom.ini`

把对应的设置改成如下的，`DaysInstalled=369` 时，`njcom` 就会无条件过期的，改完之后别忘了把 `njcom.ini` 加只读属性。

```
[Usage]

DaysInstalled=0

LastUsage=11099
```

可以无限使用了 hahahaha...... `4dd466d0e1e0cc123afbe504da2462ca`
