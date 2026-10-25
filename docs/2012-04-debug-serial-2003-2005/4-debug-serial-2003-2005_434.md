
# 流光 4.5

> 原文：[`www.pediy.com/kssd/pediy05/pediy50255.htm`](https://www.pediy.com/kssd/pediy05/pediy50255.htm)

**注**：新版的 `4.7` 已经加壳，此文仅供初学者参考。

`www.sandflee.net` 2002-3-1 灰色轨迹

**流光 4.5 完全破解(补充)**

**前言**：对不起，各位，上次写了一文，发现高级扫描不能用。所以这次写了一个补充的，呵呵。

**作者**：`oo0o`

**真实姓名**：`hang feilu`

**地址**：`yn_zt_x school_` 高 200

**破解日期**：`2002.2.13`

**使用时间**：`20 m`

**目标**：`release iv build 3116 expired 2002/12/31`

**级别**：低

**分析**：
1. 出于测试的原因，小榕并没有加壳，给破解带来一定的方便，但是有数字验证。
2. 对国内 `ip` 限制。
3. 为了方便，不破解 `key` 认证，而是暴破。

**使用工具**：`w32dasm 8.93` , `hiew`

## 1. 准备工作

将流光目录 `fluxay45.exe` 用 `w32dasm` 反汇编，并将反汇编内容存为 `fluxay45.txt` (方便测试与修改)

将 `fluxay45.exe` 备份到其它目录

**需要了解的基础**：

**工具**：
*   `w32dasm` 用于静态反汇编
*   `hiew` 用于修改程序

**汇编命令**：
*   `test` 测试。(两操作数作与运算，仅修改标志位，不回送结果)。
*   `je` / `jz` 等于转移。
*   `jmp` 无条件转移指令。
*   `jle` / `jng` 小于或等于转移。

## 2. 去除数字验证

记事本打开 `fluxay45.txt`，查找 `"数字验证失败"`，会找到：

```
* referenced by a (u)nconditional or (c)onditional jump at address:

|:00406e29(u)

|

:00406e30 85c0 test eax, eax
:00406e32 7438 je 00406e6c ;比较,相等就跳,否则就显示数字验证失败,
;改为 jmps:)具体步骤见下
:00406e34 a1a4f15800 mov eax, dword ptr [0058f1a4]
:00406e39 51 push ecx
:00406e3a 85c0 test eax, eax
:00406e3c 8bcc mov ecx, esp
:00406e3e 740b je 00406e4b
:00406e40 89642428 mov dword ptr [esp+28], esp

* possible stringdata ref from data obj ->"数字验证失败，可能被病毒感染或者捆绑了恶意程序"

->"。"

|

:00406e44 6804b25200 push 0052b204
:00406e49 eb09 jmp 00406e54
```

打开 `hiew`，选择流光目录下的 `fluxay45.exe`

按 `f4` 选择 `decode`，按 `f7` 在 `hex` 处填入 `85 c0 74 38 a1 a4`

(填多点，一次性找到，不然可能找到很多重复代码 :)

按 `f3` 修改 `7438` 为 `eb38`，按 `f9` 更新文件，按 `f10` 退出 < - 注意是按 `f10`，不是 `esc`，不然会不更新文件。

执行 `fluxay45.exe` 试一试，没有提示，说明成功！退出 `fluxay45.exe`

## 3. 去除国内 ip 限制

### 1) pop3/nt/ftp/sql 扫描破解

先试一下 `fluxay45.exe`，按 `ctrl+r` 填入国内 `ip` 网段 `211.x.x.1` - `211.x.x.255`。扫描

可看到 `"对此 ip 保留, 忽略..."` 字样

打开 `fluxay45.txt`，查找 `"对此 ip 保留"`，找到如下：

```
* referenced by a (u)nconditional or (c)onditional jump at address:

|:00473f13(c)

|

.... 处理数据部份,省略:)...........以上好复杂~~~~~~~~~:)

:00473fdb 85c0 test eax, eax
:00473fdd 746d je 0047404c ;比较,不同则跳,就是改这儿了!具体见下
:00473fdf 68d4070000 push 000007d4
:00473fe4 e866430600 call 004d834f
:00473fe9 8b0da4f15800 mov ecx, dword ptr [0058f1a4]
:00473fef 33db xor ebx, ebx
:00473ff1 83c404 add esp, 00000004
:00473ff4 3bcb cmp ecx, ebx
:00473ff6 a344045900 mov dword ptr [00590444], eax
:00473ffb 7407 je 00474004

* possible stringdata ref from data obj ->"对此 ip 保留, 忽略..."

|

:00473ffd 68908d5400 push 00548d90
:00474002 eb05 jmp 00474009
```

打开 `hiew`，选择流光目录下的 `fluxay45.exe`

按 `f4` 选择 `decode`，按 `f7` 在 `hex` 处填入 `85 c0 74 6d 68 d4` (填多点，一次性找到 :)

按 `f3` 修改 `746d` 为 `eb6d`，按 `f9` 更新文件，按 `f10` 退出。

执行 `fluxay45.exe`，按 `ctrl+r` 试一试。`211.x.x.1` - `211.x.x.25`

嘿嘿 `c_c` 成功了 ~~~~~~~

### 2) 高级扫描破解

先试一下 `fluxay45.exe`，按 `ctrl+a` 填入国内 `ip` 网段 `211.x.x.1` - `211.x.x.25`。扫描

可看到 `"不在此版本的许可的 ip 范围内，忽略."` 字样 < === 注意： 与 `pop3/nt/ftp/sql` 扫描不一样哦 :)

打开 `fluxay45.txt`，查找 `"不在此版本的许可的 ip 范围内"`，找到了两处 :)

**先看第一处**

```
* referenced by a (u)nconditional or (c)onditional jump at address:

|:00466feb(c)

|

..............................略

:00467064 83c404 add esp, 00000004
:00467067 85c0 test eax, eax ;:)大家是不是觉得有点眼熟?
:00467069 746b je 004670d6 ;看来就是改这个了:)
:0046706b 8b1568165500 mov edx, dword ptr [00551668]
:00467071 8955dc mov dword ptr [ebp-24], edx
:00467074 a1a4f15800 mov eax, dword ptr [0058f1a4]
:00467079 c645fc7e mov [ebp-04], 7e
:0046707d 3bc7 cmp eax, edi
:0046707f 7412 je 00467093
:00467081 8b857cffffff mov eax, dword ptr [ebp+ffffff7c]
:00467087 8d4ddc lea ecx, dword ptr [ebp-24]
:0046708a 50 push eax

* possible stringdata ref from data obj ->"%s 不在此版本的许可的 ip 范围内，忽略。"

|

:0046708b 68dc715400 push 005471dc
:00467090 51 push ecx
:00467091 eb10 jmp 004670a3
```

用 `hiew` 改 `746b` 为 `eb6b` :)

改完后发现不行哦！！！

**看下一处：**

```
* referenced by a (u)nconditional or (c)onditional jump at address:

|:004b1bae(u)

|

:004b1afe 8b542418 mov edx, dword ptr [esp+18]
:004b1b02 8d8c2480110000 lea ecx, dword ptr [esp+00001180]
:004b1b09 52 push edx
:004b1b0a e8c133ffff call 004a4ed0
:004b1b0f 85c0 test eax, eax ;:)就只有它了.呵
:004b1b11 7450 je 004b1b63 ;改 74 为 eb
:004b1b13 a1a4f15800 mov eax, dword ptr [0058f1a4]
:004b1b18 85c0 test eax, eax
:004b1b1a 7411 je 004b1b2d
:004b1b1c 8b442418 mov eax, dword ptr [esp+18]
:004b1b20 8d4c2414 lea ecx, dword ptr [esp+14]
:004b1b24 50 push eax

* possible stringdata ref from data obj ->"%s 不在此版本的许可的 ip 范围内，忽略。"

|

:004b1b25 68dc715400 push 005471dc
:004b1b2a 51 push ecx
:004b1b2b eb0f jmp 004b1b3c
```

打开 `hiew`，选择流光目录下的 `fluxay45.exe`

按 `f4` 选择 `decode`，按 `f7` 在 `hex` 处填入 `85 c0 74 50 a1 a4 f1` (填多点，一次性找到 :)

按 `f3` 修改 `7450` 为 `eb50`，按 `f9` 更新文件，按 `f10` 退出。

执行 `fluxay45.exe`，按 `ctrl+a` 试一试。`211.x.x.1` - `211.x.x.25`

`wow.` 提示选择流光主机 :) `yeah` 成功。第一处改了之后，大家不用改回来 :) 也许是以前版本的流光核心。

当然如果不能用还是改回来 ^_^

### 3) ipc$ 国内 ip 限制

先试一下 `fluxay45.exe`，添加一个 `ipc$` 主机 :) `x.x.x.x` < === 感谢 `"&╦快乐王子"` 提供

点右键 -> 探测 -> 探测 `ipc$` 用户列表。

```
================================
提示:

fluxay 4.5>正在连接主机 202.x.x.x...

此 ip 地址不提供 ipc$探测。

没有用户。

所有用户列表线程成功退出。
=================================
```

找到四处!!!

**第一处** ^_^

```
* referenced by a (u)nconditional or (c)onditional jump at address:

|:0047eca2(u)

..................处理数据部份,省略

:0047ed42 85c0 test eax, eax ;有点象哦.
:0047ed44 0f84df000000 je 0047ee29 ;改为 jmp
:0047ed4a 391da4f15800 cmp dword ptr [0058f1a4], ebx ;比较是什么语言代码
:0047ed50 740c je 0047ed5e ;是 e 文就跳到下面显示"this ip range is disabled."
:0047ed52 8d4c2410 lea ecx, dword ptr [esp+10]

* possible stringdata ref from data obj ->"此 ip 地址不提供 ipc$探测。"

|

:0047ed56 6874965400 push 00549674
:0047ed5b 51 push ecx
:0047ed5c eb0a jmp 0047ed68

* referenced by a (u)nconditional or (c)onditional jump at address:

|:0047ed50(c)

|

:0047ed5e 8d542410 lea edx, dword ptr [esp+10]

* possible stringdata ref from data obj ->"this ip range is disabled."

|

:0047ed62 6858965400 push 00549658
:0047ed67 52 push edx
```

用 `hiew` 按 `f7` 查找上面的那串代码 (注意上下文代码是否一至 :) 我找了好几次才找到，小心一点！！！死了别怪我 :)

按 `f3` 再按回车改 `je 0047ee29` 为 `jmp 0047ee29`

启动流光，试一下 :) 不行哦！！！

**看第二处** !_!

```
* referenced by a (u)nconditional or (c)onditional jump at address:

|:00480f81(u)

|

..........................处理数据部份,省略

:00480ffd 85c0 test eax, eax ;:)呵呵呵.一定是
:00480fff 7440 je 00481041 ;改为 eb
:00481001 393da4f15800 cmp dword ptr [0058f1a4], edi
:00481007 740c je 00481015
:00481009 8d542410 lea edx, dword ptr [esp+10]

* possible stringdata ref from data obj ->"此 ip 地址不提供 ipc$探测。"

|

:0048100d 6874965400 push 00549674
:00481012 52 push edx
:00481013 eb0a jmp 0048101f
```

打开 `hiew`，选择流光目录下的 `fluxay45.exe`

按 `f4` 选择 `decode`，按 `f7` 在 `hex` 处填入 `85 c0 74 40 39 3d a4` (填多点，一次性找到 :)

按 `f3` 修改 `7450` 为 `eb50`，按 `f9` 更新文件，按 `f10` 退出。

执行 `fluxay45.exe`，添加一个 `ipc$` 主机 :) `x.x.x.x`

点右键 -> 探测 -> 探测 `ipc$` 用户列表。

```
====================================
显示:

fluxay 4.5>正在连接主机 202.x.x.x...

和主机 202.x.x.x 空连接建立成功。

正在读取主机 202.x.x.x 的用户列表，请等待...

主机 202.x.x.x 的用户 00: administrator* (管理计算机(域)的内置帐户)

主机 202.x.x.x 的用户 01: guest* (供来宾访问计算机或访问域的内置帐户)

读取主机 202.x.x.x 的用户列表完成，2 个用户。

fluxay 4.5>正在连接主机 202.x.x.x...
```



此 `ip` 地址不提供 `ipc$` 探测。？咦？这个是探测什么的。是不是还有呀。kao 再试

所有用户列表线程成功退出。

`fluxay 4.5>` 准备探测主机 `202.x.x.x`...

所有用户列表线程成功退出。

探测 `202.x.x.x`: `guest`............【空】

....................省了一串:)

探测 `202.x.x.x`: `administrator`............`12345`

探测结束，得到主机 `202.x.x.x` 的 0 个帐号。

所有远程登陆线程成功退出。

======================================

-----------------------------------------------------------------------------------------------

第三处 @\_@

\* referenced by a (u)nconditional or (c)onditional jump at address:

\|:004849a6(u)

...................省略：）

```
:00484a19 85c0 test eax, eax ;又是。kao！管它是干什么的。

:00484a1b 7440 je 00484a5d ;改：）请大家注意备份。

:00484a1d 391da4f15800 cmp dword ptr \[0058f1a4\], ebx

:00484a23 740c je 00484a31

:00484a25 8d542410 lea edx, dword ptr \[esp+10\]

\* possible stringdata ref from data obj ->"此 ip 地址不提供 ipc$探测。"

|

:00484a29 6874965400 push 00549674

:00484a2e 52 push edx

:00484a2f eb0a jmp 00484a3b
```

打开 `hiew`，选择流光目录下的 `fluxay45.exe`

按 `f4` 选择 `decode`，按 `f7` 在 `hex` 处填入 `85 c0 74 40 39 1d a4` (填多点，一次性找到:)

按 `f3` 修改 `7450` 为 `eb50`，按 `f9` 更新文件，按 `f10` 退出.

执行 `fluxay45.exe`，删除刚才用户列表，再试

点右键->探测->探测 `ipc$` 用户列表.

与第二显示一样，kao

-----------------------------------------------------------------------------------------------

第四处 \*\_\*

referenced by a (u)nconditional or (c)onditional jump at address:

\|:004935af(u)

\|....................略

```
:00493631 85c0 test eax, eax ;有备份的举手。：）

:00493633 0f84b7000000 je 004936f0 ;改！>:|

:00493639 391da4f15800 cmp dword ptr \[0058f1a4\], ebx

:0049363f 740c je 0049364d

:00493641 8d442410 lea eax, dword ptr \[esp+10\]

\* possible stringdata ref from data obj ->"此 ip 地址不提供 ipc$探测。"

|

:00493645 6874965400 push 00549674

:0049364a 50 push eax

:0049364b eb0a jmp 00493657
```

用 `hiew` 改，不写了.shit!(噢，英语老师最不让说的词\^\\_^\^下次不说了)

算了，说两句

是按 `f3` 再按回车改 `je 004936f0` 为 `jmp 004936f0`

执行 `fluxay45.exe`，删除刚才的用户列表

点右键->探测->探测 `ipc$` 用户列表.

===================

显示:

`fluxay 4.5>` 正在连接主机 `202.x.x.x`...

和主机 `202.x.x.x` 空连接建立成功。

正在读取主机 `202.x.x.x` 的用户列表，请等待...

主机 `202.x.x.x` 的用户 00: `administrator\*` (管理计算机(域)的内置帐户)

主机 `202.x.x.x` 的用户 01: `guest\*` (供来宾访问计算机或访问域的内置帐户)

读取主机 `202.x.x.x` 的用户列表完成，2 个用户。

`fluxay 4.5>` 正在连接主机 `202.x.x.x`...

和主机 `202.x.x.x` 空连接建立成功。

正在读取主机 `202.x.x.x` 的共享列表，请等待...

读取主机 `202.x.x.x` 的共享列表失败(错误号: 259)， ;看看，哦是读 `share` 列表。怎么能不改这个功能哩:)

对方主机不允许。

所有用户列表线程成功退出。

`fluxay 4.5>` 准备探测主机 `202.x.x.x`...

探测 `202.x.x.x`: `guest`............【空】

....................省略若干

探测 `202.x.x.x`: `administrator`............`12345`

探测结束，得到主机 `202.x.x.x` 的 0 个帐号。

所有远程登陆线程成功退出。

`fluxay 4.5>` 保存项目....ok.

`fluxay 4.5>` 打开项目....ok，已经探测 0 个密码。

========================================================

`ipc$` 大大大功告成了。:) 好复杂。由于太多，可能有些地方会写错，有错 `mail` me :`oo0o@ynmail.com`

不过我是可以用：）呵。不要找我要，好象 `www.xzero.org` 上有破解的。我写这文是为了让大家了解破解。

4.对了，好象还有日期破解：？5........写吧。写完了，我再也不上网了，我要好好学习了：）

请把各位的日期调到 2003 年，执行 `fluxay45.exe`

提示"流光已经到期"云云

打开 `fluxay45.txt` 按 `ctrl+home` 到文本最开头，以"到期"为关键字查找。又是四处！！！

怎么会有四处？：）试一下

第一处:

\* referenced by a (u)nconditional or (c)onditional jump at address:

\|:004f050c(u)

\|.......................略

```
:00406605 3d3e4f123e cmp eax, 3e124f3e

:0040660a 7e4b jle 00406657

:0040660c 6a00 push 00000000

:0040660e 8d4c2434 lea ecx, dword ptr \[esp+34\]

:00406612 e899a40000 call 00410ab0

:00406617 a1a4f15800 mov eax, dword ptr \[0058f1a4\]

:0040661c 51 push ecx

:0040661d 85c0 test eax, eax

:0040661f c784245807000000000000 mov dword ptr \[esp+00000758\], 00000000

:0040662a 8bcc mov ecx, esp

:0040662c 740b je 00406639 ;看着它就想睡觉:) 改！（先备份）

:0040662e 8964240c mov dword ptr \[esp+0c\], esp

\* possible stringdata ref from data obj ->"【流光 4.5】已经到期，请到 http://www.netxeyes.c"

\->"om 下载新版本。"

|

:00406632 68a0b15200 push 0052b1a0

:00406637 eb09 jmp 00406642
```

改 `740b` 为 `eb0b` ok 执行，？不行？

大家注意到这儿了么？

```
:00406605 3d3e4f123e cmp eax, 3e124f3e

:0040660a 7e4b jle 00406657

jle/jng 小于或等于转移.会不会是小于某个日期就跳呢？：）改
```

结果：不行。呵。不过一定是改了一部份。（猜）

第二处:

```
:00407ac5 3d3e4f123e cmp eax, 3e124f3e

:00407aca 7e4b jle 00407b17 ;改 7e4b 为 eb4b

:00407acc 6a00 push 00000000

:00407ace 8d4c2434 lea ecx, dword ptr \[esp+34\]

:00407ad2 e8d98f0000 call 00410ab0

:00407ad7 a1a4f15800 mov eax, dword ptr \[0058f1a4\]

:00407adc 51 push ecx

:00407add 85c0 test eax, eax

:00407adf c784245807000000000000 mov dword ptr \[esp+00000758\], 00000000

:00407aea 8bcc mov ecx, esp

:00407aec 740b je 00407af9

:00407aee 8964240c mov dword ptr \[esp+0c\], esp

\* possible stringdata ref from data obj ->"【流光 4.5】已经到期，请到 http://www.netxeyes.c"

\->"om 下载新版本。"

|

:00407af2 68a0b15200 push 0052b1a0

:00407af7 eb09 jmp 00407b02
```

大家用 `hiew` 改 `7e4b` 为 `eb4b` （不用我写了吧：）

执行，？可以了？：）呵呵呵。大功告成。\^\\_^\^

省下的几处我也难得试了，也许某些地方还有限制，有兴趣的自己试吧。

结束语：流光 4.5 的破解其实已经很简单了，大家也不要再为难小榕了：） 大家也许不知道流光 2001

的破解要难得多，流光连上几个网站下载过期数据。：）这个似乎已经去除？

还有那个调查表也比这个难（有兴趣的找来破破：）用 `softice` 跟踪吧。静态反汇编比较麻烦）

本人菜鸟一只，在网上没找到流光 4.5，也没有 `cracker` 写它的破解方法，(也许是我找不到吧：）

所以写此文，有错的地方大家请指点\^\\_^\^

如果还有哪不行，请 `mail` me:`oo0o@ynmail.com` 我不想在 `qq` 上聊 <低级问题一概不答，没时间，sorry>:)

附：有几个地方也许是以前版本的流光内核，改不改都行，有兴趣的试一下。不过我不想改回来了：）

懒！如果死了之类，请改回来。有备份？\^\\_^\^ 请大家不要入侵国内主机，而是帮他们找洞。我不是 `cracker`

也不是 `hack`，甚至不是 `script kids`。我只是一个菜鸟。\^\\_^\^ 88

补充:也许小榕以后会对文件加壳，所以先说了以后也不再写了.:)

使用工具:`typ` ,`upx1.2` for `win` (`ddcrack.yeah.net` 有下)以流光 4 2915 为例

把 `fluxay4.exe` copy 到期 `d:\typ`

再 `d:\typ\typ3` `fluxay4.exe`

可以看到(如果是 `xp` 会有错误提示.按忽略即可)

```
■ extended \[32\] exe "pe" starting at $0108

■ common object file format - (intel 386)

■ target sytem windows gui

■ upx / markus f.x.j. oberhumer & lászló molnár \[0.90 win32,rtm32/pe\]
```

看到了吗? `upx` 0.9 压缩?:)

用 `upx1.2` 中文版的图形界面(不用我讲怎么用了吧:) 解压，就可得到正确的反汇编代码.呵

`mail:` `oo0o@ynmail.com`

呵呵！贴了这篇文章，真的很惭愧，因为我也想到了，可是，也只有在这里拾人牙慧了！

呵呵！有待学习。

给我曾经深的 `dodo`。

相爱的时光太短，遗忘的过程太长。也许我们只是人生轨迹的一个交点

`boyhacker`

`qq;43711234`

原作者：`boyhacker`

来 源：中国奇兵 `4dd466d0e1e0cc123afbe504da2462ca`
