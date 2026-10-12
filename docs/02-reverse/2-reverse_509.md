
# 浅谈 sar 指令在整数除法中的优化

> 原文：[`www.pediy.com/kssd/pediy11/106975.html`](https://www.pediy.com/kssd/pediy11/106975.html)

初学《加密与解密 3》看到第四章里的编译器的算法优化时的一点困惑，然后动手实践了一下得到一点心得体会，拿来跟大家分享！

ps.本人汇编基础非常差，`sar` 指令也是刚刚学会的，另外脑袋转的慢，别人用 5 分钟能解决的问题，我得用 10 分钟，因此这篇文章我花了 6 个多小时才完成。

如果有错误欢迎指出，以免误导他人！

## 浅谈 Sar 指令在整数除法中的优化

还不懂 `sar` 指令的同学马上去 `baidu` 啊，`Google` 啊查去，我刚查完！

首先来看一段 `vc` 代码

```
int get()
{
  return -8;
}

int main()
{
  int a = get()/2;
  printf("%d",a);
  return 0;
}
```

汇编代码（本文里反汇编码都是指 `Release` 版本下的 `OD` 返回编码）如下

```
00401000  /$  B8 F8FFFFFF   mov     eax, -8   ; get 子程序 总是返回-8
00401005  \.  C3            retn
00401006      90            nop
00401010  /$  E8 EBFFFFFF   call    00401000  ;得到返回值 eax = -8
00401015  |.  99            cdq               ;将 eax 的值符合扩展到 edx
00401016  |.  2BC2         sub     eax, edx   ;等价于 eax = eax + 符号位（eax 为正数或者 0 时符号位为 0，负数时为 1）
00401018  |.  D1F8         sar     eax, 1     ;带符合右移一位
0040101A  |.  50            push    eax
0040101B  |.  68 30704000   push    00407030    ;  ASCII "%d"
00401020  |.  E8 0B000000   call    00401030     ;printf 函数
00401025  |.  83C4 08       add     esp, 8
00401028  |.  33C0         xor     eax, eax
0040102A  \.  C3            retn
0040102B      90            nop
```

简单的伪代码表示如下

```
Mov eax, idata             ;idata 表示一个整数
Add eax, eax 的符号位
Sar eax, 1
```

上面是一个整数/2 的情况

如果一个整数/2^n(这里 2^n 表示 2 的 n 次方) 应该如何表示呢

我们将上面的 `VC` 代码修改这么一句

`int a = get()/2; => int a = get()/16;`

通过反汇编自己翻译成伪代码就是这样的

```
Mov eax, idata             ;idata 表示一个整数
Add eax, (eax 的符号位)*(2^n - 1) ;这里 n=4
Sar eax, n
```

上面这个就是一般的除数是 2^n 一般的公式了，正负整数都管用

粗略的用这样一个数学公式来表示：

`Idata/(2^n) = YWn(idata+2^n - 1)`

注意 `YWn` 表示将其后面的数以 `sar` 方式移动 n 位 即 `sar (数),n`

再变换一下，将 `idata` 换成 `(idata 2^n + 1)`

`(Idata 2^n+1)/(2^n) = YWn(idata)             -----------最终公式`

Ps.上面的公式是我 `OD` 反汇编总结出来的，缺少数学的严格推理证明。我不会证，谁来证明下啊？感激不尽！

下面来解释一个难点，也是写本文的初衷

看一段 `VC` 代码：

```
int get()
{
  int i=rand()%2;
   if(i)
  return rand();
   else
   return rand()*(-1);
}

int main()
{
  int a = get()/9;
  printf("%d",a);
  return 0;
}
```

反汇编码：

```
00401010  /$  E8 EBFFFFFF   call    00401000   ;调用 get 函数 eax=随即数
00401015  |.  8BC8          mov     ecx, eax  ; ecx = eax
00401017  |.  B8 398EE338   mov     eax, 38E38E39 ;这是一个编译器预先设定好的 MagicNumber
0040101C  |.  F7E9          imul    ecx ;带符合的乘法，不多说
0040101E  |.  D1FA          sar     edx, 1 ;将乘法的高 32 位结果算数右移一位
00401020  |.  8BC2          mov     eax, edx ;eax = edx
00401022  |.  C1E8 1F       shr     eax, 1F   ;逻辑右移 31 位
00401025  |.  03D0          add     edx, eax ;以上这 3 句的意思就是  edx = edx+edx 的符号位，本文最重要的目的就是解释他为什么加 1
00401027  |.  52            push    edx
00401028  |.  68 30704000   push    00407030          ;  ASCII "%d"
0040102D  |.  E8 0E000000   call    00401040  ;printf edx
00401032  |.  83C4 08       add     esp, 8
00401035  |.  33C0         xor     eax, eax
00401037  \.  C3            retn
```

关于 `MagicNumber` 这里给你几个连接自己看，不多介绍

[`bbs.pediy.com/showthread.php?t=68849`](http://bbs.pediy.com/showthread.php?t=68849)

[`bbs.pediy.com/showthread.php?t=81456`](http://bbs.pediy.com/showthread.php?t=81456)

我们把上面的反汇编代码写成伪代码，如下

```
idata 表示一个 32 位有符号数，这里一定是有符号的，否则逻辑错误
magic 表示立即数 0x38E38E39  他是一个正数
FH(idata) 表示 idata 的最高位，即 FH(idtat) = (idata & 0x80000000)? 1:0
\[edx,eax\]表示 edx 和 eax 组成的 64 位的寄存器

Mov \[edx,eax\], magic*idata
Sar \[edx,eax\],33       ;由于移位的操作此时 eax 里面的值已经没有意义了
Add edx + idata 的符号位  ;因为 magic 是个正数，所以等价于 magic*idata 的符号位
```

而我们的本意是 `idata/9`

因此得到如下数学公式，我们就是要证明他的正确性

`YW33(magic*idata) + FH(idata) = idata/9           ------------ (黄金公式)`

①.  当 `FH(idata) == 0` 时

上述黄金公式即

`Magic*idata/2³³ = idata/9`

即

`Magic*9 = 2³³`

我们将 `magic = 0x0x38E38E39` 带入  然后用科学计算器验证了下结果如下

`Magic*9 = 0x200000001`

`2³³ = 0x200000000`

这种优化的除法本身就是一种近似，这里不多讨论为什么存在一个 1 的差异

证毕。

②.  当 `FH(idata) == 1` 时

由 `最终公式(Idata 2^n+1)/(2^n) = YWn(idata)`

得到

`YW33(magic*idata) = (magic*idata 2³³ + 1)/(2³³)`

将右边带入黄金公式即：

`(magic*idata 2³³ + 1)/(2³³) + 1 = idata/9`

即

`Magic*idata/2³³  1 + 1 = idata/9          (;1/2³³ ≈ 0 因此忽略掉)`

即

`Magic*idata/2³³ = idata/9`

即

`Magic*9 = 2³³`

证毕。

## 上传的附件

| ![](img/58e0728fcb44eb7297cf87d68539ed4e.jpg) || 浅谈 Sar 指令在整数除法中的优化.doc  

 |

遗漏说明一下 ：最终公式只对负数有效

正数是这个样子：`Idata/(2^n) = YWn(idata)`
