
# 压缩壳和低强度加密壳的脱壳小结及简易脱壳法

> 原文：[`www.pediy.com/kssd/pediy10/61077.html`](https://www.pediy.com/kssd/pediy10/61077.html)

不知道看雪的精华帖从什么时候开始算起，记得很早的时候已经有几篇精华了，现在居然不让在调试和保护版块发帖，郁闷....... 望版主赏脸给个精

坛主的建议很中肯，愉快接受，谢谢！

## 一、脱壳的一般方法

脱壳方法现在能找到的资料很多，总结起来主要有这么几种方法：

1. `单步跟踪脱壳`
2. `ESP 定律脱壳`
3. `特征代码脱壳`
4. `内存镜像脱壳`
5. `最后一次异常脱壳`
6. `ollydbg SFX 自动跟踪脱壳`

这些方法可以去网上找一篇帖子“普通壳的脱壳方法和脱壳技巧，叫新手少走弯路！”，这篇贴子讲得比较详细,这里不再赘述。

## 二、OEP 内存断点万能脱壳法

对于压缩壳和低强度加密壳经过一段时间的研究发现有一种万能的脱壳方法－－OEP 内存断点万能脱壳法。什么是压缩壳和低强度加密壳呢？所谓压缩壳和低强度加密壳是指能用 `PEiD` 的查找 `OEP` 插件找到 `OEP` 的壳，具体的种类有很多，经本人试验有以下这些 `upx`、`aspack`、`fsg`、`Aspack Scrambler`、`ExeStealth`、`ID Application Protector`、`V2Packer`、`WWPack32`、`XComp0.98`、`bambam`、`BeRoEXEPacker`、`dePACK`、`ExeShield Protector`、`KByS`、`NsPacK`、`tElock`、`yoda's Protector`、`nPack`、`!EP(ExE Pack)`、`ASDPack`、`eXPressor`、`PECompact`.......等很多种，事实上只要能用 `PEiD` 侦测到 `OEP` 的壳都可以用这种方法。

`OEP` 内存断点法具体操作步骤如下：

1. `PEiD` 侦测壳的种类和 `OEP`（壳的种类未知也可以的，但 `OEP` 必须要能够侦测到）
2. `ollydbg` 载入目标程序，停在壳的入口
3. 数据窗口 `Ctrl+G`，填入 `PEiD` 侦测到 `OEP`，确定
4. 在数据窗口 `OEP` 处设置内存访问断点，`F9` 运行
5. 观察内存数据窗口 `OEP` 处数据变化
6. 直到出现 6 种语言的特征代码
   - `Borland Delphi 6.0-7.0`：`55 8B EC 83`
   - `Microsoft Visual C++ 6.0`：`55 8B EC 6A`
   - `Microsoft Visual C++ 6.0 [Overlay] E 语言`：`55 8B EC 6A`
   - `Microsoft Visual Basic 5.0-6.0`：`68 D0 ** ** ** E8`
   - `Dasm`：`6A 00 E8 C5`
   - `BC++`：`EB 10`
7. 取消内存断点，反汇编窗口 `Ctrl+G` 填入 `PEiD` 侦测到 `OEP`，在 `OEP` 处下断，`F9` 运行，到达 `OEP`
8. `Dump it` 运行，出错的话用 `ImportREC` 修复即可，一般都可修复的。

## 三、OEP 内存断点万能脱壳法脱壳机理

我们知道的，壳如果要把原来加密或压缩的代码运行起来的话就必须要解压和解密原来的代码,这一个过程我们可以将他看做是代码的内存写入，当我们用 `ollydbg` 载入目标程序，数据窗口 `Ctrl+G` 转到 `OEP` 地址时往往看到的数据全部是 `0`，这说明 `OEP` 处的代码被压缩或加密了。（有些时候可以看到有代码，但这些代码不是 6 种常见语言的入口特征代码，这说明 `OEP` 处肯定是被加密了，被壳的加密代码替换了。）基于这样的理解我们可以在数据窗口 `OEP` 处设置内存访问断点，`F9` 运行，观察内存数据窗口 `OEP` 处数据变化，内存数据变化说明 `OEP` 处代码在被解密，当我们看到 6 种常见语言的入口特征代码时说明 `OEP` 处已经被解压或解密，可以取消内存断点，直接在 `OEP` 处下断，运行到 `OEP` 脱壳。

## 四、OEP 内存断点万能脱壳法的简化变形

了解了 `OEP` 内存断点万能脱壳法脱壳机理以后，对于一些压缩壳这种方法还可以进行简化，对于压缩壳 `OEP` 处的代码在未解压前是不会被执行的，只有在解压完全之后才会被执行，（加密壳不一定是这样的，因为在 `OEP` 处的代码可能被壳的代码替换，是有可能被执行到的，执行的目的是解密原程序。）基于这样的理解，我们可以直接在在数据窗口 `OEP` 处设置硬件执行断点，当 `OEP` 处代码被执行时进行中断，而这时也正好是我们脱壳的最佳时机。

## 五、OEP 内存断点万能脱壳法简化变形的 OD 脚本

`OEP` 内存断点万能脱壳法经过简化变形后其操作变得非常简单机械，我们可以用一个很简单的 `ollydbg` 脚本来使其自动执行脱壳，脚本如下：

```
/*
//////////////////////////////////////////////////
        All Compress packer Unpacking script
        Author:        sxssl
        Email : sxssl@163.com
        OS    : WinXP sp2,Ollydbg 1.1,OllyScript v0.92
        Date  : 2008-1-20
        Config: Ignore all exceptions
        Note  : If you have one or more question, email me please,thank you!
//////////////////////////////////////////////////
*/
input:
ask "Please Enter OEP VA"
cmp $RESULT,400000
jb message
cmp $RESULT,400000
ja bphw
bphw:
bphws $RESULT, "x"     
run 
bphwc $RESULT
jmp end
message:
msg "OEP VA Is False! Please Enter True OEP VA!"
jmp input
end:
msg "Script by sxssl,Thank you for using my script!"
ret
```

## 附件

|  |  |
| :--- | :--- |
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) | `试炼.rar` `[解压密码：PEDIY]` |
