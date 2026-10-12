
# Flash 相关

> 原文：[`www.pediy.com/kssd/pediy04/kanxue373.htm`](https://www.pediy.com/kssd/pediy04/kanxue373.htm)

终于考完试了，从网上下了一大堆的 `flash`，好好放松放松，其中包括 `showgood` 的大话三国系列等等。但当我想打开“浪花一朵朵”的动画时，却出现提示：`This movie can only be Viewed at www.showgood.com`，我晕@!#$%@!$#!@#$.....我可是 `56K` 的破猫，怎能忍受线上观看呢？！只有想办法破解了。

我先想用我的 `flash 5` 打开，提示被保护，下载了 `Flash Deprotector` 解除保护也不能编辑，我倒~~~~怎么办呢？这又不是 `exe` 文件或 `dll` 文件，又不能反汇编，还真给我出了个难题。

不过，`Cracker` 的智慧是无穷的^\_^经过 `N(N<=15)`分钟的思考和尝试，俺利用最原始的武器------`UEDIT` 解决了问题，具体步骤如下：

用 `Uedit` 打开“浪花一朵朵.swf”文件，在里面查找“`showgood`”，可以找到几个地方，其中 `000062c0h` 这个地方引起了我的注意，右边显示的“`bbq movie.showgood.com indexOf bbq1 210.78.159.210 noline \_root gotoAndStop`”。幸亏俺学过几天 `flash`，看得懂一点点脚本语言，这段代码就是检测是否在 `showgood.com` 上观看的关键部分，后面的 `gotoAndStop` 就相当于我们平时破解普通软件时的关键跳。把 `gotoAndStop` 所对应的 `16` 进制码都改成 `00` 就可以了。嘿嘿，剩下的就不用我说了吧，把 `gotoAndStop` 解决掉就可以了。

一点点经验，与大家共享，不知道还有没有更好的方法^\_^ `4dd466d0e1e0cc123afbe504da2462ca`
