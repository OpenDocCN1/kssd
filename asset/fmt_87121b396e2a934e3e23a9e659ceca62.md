
# Idyle Tasks Manager 98

> 原文：[`www.pediy.com/kssd/pediy03/forum923.htm`](https://www.pediy.com/kssd/pediy03/forum923.htm)

《爆破 Idyle Tasks Manager 98》糊理糊涂版

目标：`Idyle Tasks Manager 98 V1.0 for Windows 95/98`

该软件不仅是一个系统信息工具，而且可以监视当前正在运行的所有程序并强行中断以释放内存。

工具：`TRW2000 V1.22` & `W32DSM89` & `ULTRAEDIT-32`

作者：`RATARICE`

简录：

一、运行程序，发现其是 20 天试用，注册吧！却发现它要发 E-MAIL 才行！那怎么办？

二、将时间向后调一个月，再运行程序，提示“the version has expired...”

确定后，又弹出两个提示框，程序结束。记下上面的话了吗？

三、用 `W32DSM89` 反汇编它，找到上面的话。

四、省略无数次尝试... ...

五、改来改去，无果。只好找出 `TRW2000`。

运行 `trw`, `ctrl+N` 下 `bpx getsystemtime` `ctrl+N`；

运行程序。被拦。下 `pmodule`，按几次 `F10` 就弹出错误提示。

五、省略无数次尝试... ...

六、盛怒之下，决定将弹出错误的 `call` `Nop` 掉。

七、改好后，运行竟然成功！

`Tasks Manager98 1.12` 破解过程。

下载地址：`http://www.newhua.com.cn/down/taskmanager98.zip`

工具：`TRW20001`。`22` 和一个 16 进制编辑工具。

首先安装完软件（废话），然后把系统日期跳后 1 年。运行后出现到期提示。好了，开始：

运行 `TRW2000`，用 `MESSAGEBOXA` 下断，再次运行，断到这里：

```
* Reference To: user32.MessageBoxA, Ord:0000h

:0048488F E83421F8FF            Call 004069C8------------我们在这回到程序领空
:00484894 33C0                  xor eax, eax
:00484896 5A                    pop edx
:00484897 59                    pop ecx
:00484898 59                    pop ecx
:00484899 648910                mov dword ptr fs:[eax], edx
:0048489C 68B6484800            push 004848B6
:004848A1 8D45F8                lea eax, dword ptr [ebp-08]
:004848A4 BA02000000            mov edx, 00000002
:004848A9 E822F2F7FF            call 00403AD0
:004848AE C3                    ret
```

向上看有没有跳转语句。。。。。没有，好按 `F10` 出来这个 `CALL`。

```
:004923F7 E84C24FFFF            call 00484848-------------这个就是出错

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00492396(U)
|
:004923FC 8B0DC04E4900          mov ecx, dword ptr [00494EC0]--我们在这里。
:00492402 8B09                  mov ecx, dword ptr [ecx]
:00492404 B201                  mov dl, 01
:00492406 A178754800            mov eax, dword ptr [00487578]
:0049240B E874ACF9FF            call 0042D084
```

这回再向上找跳转语句。。。。。有了：

```
:0049234F 7347                  jnb 00492398
```

改了试试。。。。。没用。。。没关系，我们再向上走。。。。。

```
:00492335 0F8653010000          jbe 0049248E
```

改这个试试。。。。好了，没有时间提示了。

还没有完，软件还是老出注册提示，我们也来把它搞掉。

先想想，软件注册了就没有时间提示了，那是不是判断注册的语句在判断时间的语句之前呢？？

再向上，就是这里：

```
:004922FC 740D                  je 0049230B                  -------这个是第一个
:004922FE 80BBF805000000        cmp byte ptr [ebx+000005F8], 00
:00492305 0F8510020000          jne 0049251B                 -------这个是第二个，完全跳出注册的窗体。
```

改掉之后。。。。。。就是个没有麻烦的软件了。哈哈。。。。。（家里人喊到：大半夜的不睡觉笑什么 ！！！！！！！！）

我是在 `TRW2000` 下动态调试通过。

`LL`

`China Cracking Group`

`2001.5.4` `4dd466d0e1e0cc123afbe504da2462ca`
