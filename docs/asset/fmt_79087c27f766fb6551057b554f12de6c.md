
# Crunch v1.1 的 ProcDump 脚本

> 原文：[`www.pediy.com/kssd/pediy03/forum350.htm`](https://www.pediy.com/kssd/pediy03/forum350.htm)

国庆节大家好！送个小甜饼给大家 ----- 有关 Crunch v1.1 加壳程序的 ProcDump 脱壳 Script

【声明】

我写文章以交流为主，希望大家在转载时能保持文章的完整性。

【前言】

由于我现在还没有破解版的 Crunch v1.1，所以就拿未注册版的 Crunch 来测试。好象未注册版的 Crunch 加壳的程序的名字有限制，其他的我就不清楚了。由于我 `DOWN` 的这个文件是根据看雪论坛的一个贴子提供的地址下载的。连帮助文件也没有。:-)

目标：     脱壳

作者：     ljttt

写作日期： 2000-10-02

以下是我写的 `ProcDump` 用的 Script，把其加入 `ProcDump` 的 `Script.ini` 文件就可以了。我测试的结果可以对一些 Crunch 加壳的程序自动脱壳，在使用时最好选中 `Option/Rebuild new import table`。不过我用 `C++Builder3` 编写的程序用此自动脱壳测试时有问题。:-)

另外，在使用时最好不要在启动时加载 `SoftICE` 等调试器。本来我想在 Script 中跳过这些反跟踪的代码的，但是不知道有 Script 中有哪个指令可以直接设置 `ESI` 等寄存器的值，如 `ESI=10`。哪位大虾知道，还请麻烦你告诉我一声。我先多谢了。

如果哪位大虾已经把 `Crunch.EXE` 脱壳了，能否通知我一声，或者简单说说脱壳的方法。毕竟我自己从头摸索又要花费大把大把的时间。小弟先作揖了。

```
[INDEX]

P24=My Crunch 1.1

[My Crunch 1.1]

L1=OBJR

L2=LOOK F3,A4

L3=ADD 2

L4=BP

L5=WALK

L6=OBJR

L7=LOOK FF,D2,CC,CC

L8=BP

L9=MOVE 9

LA=WALK

LB=WALK

LC=LOOK 83,FB,00,7F,F0

LD=ADD 3

LE=BP

LF=WALK

L10=WALK

L11=LOOK 03,C5,FF,E0,FF,A5

L12=ADD 4

L13=BP

L14=WALK

L15=OBJR

L16=LOOK 8B,85,??,??,??,??,01,85,??,??,??,??,FF,A5

L17=ADD C

L18=BP

L19=WALK

L1A=OBJR

L1B=LOOK 61,5D,8B,85,??,??,??,??,5D,FF,E0

L1C=ADD 9

L1D=BP

L1E=STEP

OPTL1=00000000

OPTL2=01010001

OPTL3=01010001

OPTL4=00030000

OPTL5=00000000
```

【后记】

如果大家在使用时有什么问题或者有什么建议，可以给跟帖子说说。 `4dd466d0e1e0cc123afbe504da2462ca`
