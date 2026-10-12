
# B-Puzzle Version 5.0

> 原文：[`www.pediy.com/kssd/pediy04/kanxue261.htm`](https://www.pediy.com/kssd/pediy04/kanxue261.htm)

**破解 B-Puzzle Version 5.0**

**作者**：同心是我  
**破解时间**：2001-11-14  
**软件简介**：一个旋转排序的益智游戏，数字、图片都可以，玩起来非常过瘾！不错！  
**下载**：`http://www.adcsoft.com`  
**限制**：试玩 10 天，游戏画面上显示“Unregistered Version of B-Puzzle”。  
**注册费用**：$10  
**破解工具**：`TRW 1.23`  
**难度**：易

◆◆◆◆◇◇◇◇◆◆◆◆◇◇◇◇◆◆◆◆◇◇◇◇◆◆◆◆◇◇◇◇◆◆◆◆◇◇◇◇◆◆◆◆

早就听说 `ADC-Soft` 的游戏非常适合新手操练，于是就奔 `http://www.adcsoft.com` 拉回来了四个游戏，一一加以开刀。昨天破完了 2 个，今天接着破，谁让我不破就手痒呢？这几个软件真是练习破解的精品。

我刚开始学破解没多长时间，希望各位高手看后别笑我水平太菜，请多多给我帮助和指导，谢谢！

运行 `B-Puzzle`，接下来又是漫长而又短暂的 3 秒钟，点击“`Enter reg.code`”按钮，进入注册提示窗口，填写：

`User name:tongxinshiwo Registraiton code:780822`

按 `Ctrl+N` 激活 `TRW`。

输入 `bpx hmemcpy` ←…… 下断点  
输入 `g` ←…… 继续运行

返回注册窗口，点击 `OK`。

又返回 `TRW`，你会在屏幕下方看到：

`Break on BP1` ←…… 拦截成功

输入 `bc *` ←…… 清除断点  
输入 `pmodule` ←…… 等于按 `F12` 若干次

这时，在屏幕中间你能看到绿色的那一行：

`015F:00458AD5 MOV [EBX+0C],EAX`

直接按 62 下 `F10` 到:

`015F:0040B9CF MOV EDX,02`

输入 `d eax`  
输入 `e` ←…… 光标跳至屏幕上部的内存窗口，按↑键向上查看。

一直到 `0030:0070EC5C` 处，看吧，注册码就在眼前。←…… 在屏幕右上方 `PROT32` 下面会有个注册码。

我的是：`63259863`

按 `F5` 返回，会提示“Invalid user name or registration code.”，点击“确定”按钮，再点击“Enter reg.code”按钮，进入注册窗口，重新输入 `User Name` 和 `Registration Code`，点击“`OK`”按钮，你就可以看到“Thank youfor you registering!”。哈哈，我已经是注册用户了！

`User name: tongxinshiwo`  
`Registraiton code: 63259863`

以同样的方法得到中文注册名的注册码。

`User name: 同心是我`  
`Registraiton code: 1470`

**破解后的思考：**

(1) 这个软件到期后，会提示“Your evaluation copy has been expired.”。如果这时你点击“Register Later”按钮，程序会自动退出，如果你点击右上角“×”按钮就会看到游戏画面了。看来 10 天的限制还是不彻底。下个版本作者要继续努力哟，嘻嘻……

(2) 英文名字的注册码是 8 个数字，中文的是 4 个数字，在内存中找到 `1470` 这个注册码后，我还想这是个假注册码呢，没想到一注册到也成功了。在破解以前未曾想到两个注册码的位数竟相差 4 位。

这是我的第五篇破解手记，不足之处请大家多多指出，谢谢！

同心是我 `372.net@371.net`

★破解很容易学的，只要你有信心和毅力再加上适当的破解工具软件，相信你也会成为破解高手的，让我们一起进步吧！★

◆◆◆◆◇◇◇◇◆◆◆◆◇◇◇◇THE END◇◇◇◇◆◆◆◆◇◇◇◇◆◆◆◆ `4dd466d0e1e0cc123afbe504da2462ca`
