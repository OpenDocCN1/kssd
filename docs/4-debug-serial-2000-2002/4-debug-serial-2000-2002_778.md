
# CTris 2000 Version2.0

> 原文：[`www.pediy.com/kssd/pediy04/kanxue259.htm`](https://www.pediy.com/kssd/pediy04/kanxue259.htm)

## 破解 CTris 2000 Version2.0

**作者**：同心是我

**破解时间**：2001-11-13

**软件简介**：一个有七种方块、四种面板的增强版俄罗斯方块游戏，可玩性很强！

**下载**：`http://www.adcsoft.com`

**限制**：只能试玩 10 天。

**注册费用**：$15

**破解工具**：`TRW` 1.23

**难度**：易

---

早就听说 `ADC-Soft` 的游戏非常适合新手操练，于是就奔 `http://www.adcsoft.com` 拉回来了四个游戏，一一加以开刀。真是不错，我刚刚破了那个增强版的扫雷，接下来今天就破解 `CTris 2000 Version2.0` 吧！

我刚开始学破解没多长时间，希望各位高手看后别笑我水平太菜，请多多给我帮助和指导，谢谢！

## 破解过程

运行 `CTris 2000`，又是漫长而又短暂的 3 秒钟，点击“Enter reg.code”按钮，进入注册提示窗口，填写：

```
User name: tongxinshiwo 
Registraiton code: 780822
```

按 `Ctrl+N` 激活 `TRW`，

```
输入  bpx hmemcpy  ←…… 下断点

输入  g            ←…… 继续运行
```

返回注册窗口，点击 `OK`

又返回 `TRW`，你会在屏幕下方看到：

```
Break on BP1       ←…… 拦截成功
```

```
输入  bc *         ←…… 清除断点

输入  pmodule      ←…… 等于按 F12 若干次
```

这时，在屏幕中间你能看到绿色的那一行：

```
015F:0042CC6D  MOV  [EBX+0C],EAX
```

按 7 下 `F12` 一直到下面:

```
015F:004291AE  RET
```

再按 22 下 `F10` 到:

```
015F:0046DA61  MOV  EAX,[EBP-0C]       ←……下 D EDX
```

看吧，注册码就在眼前。←…… 在屏幕右上方 `PROT32` 下面会有个注册码。

我的是：`CT87975921`

按 `F5` 返回，会提示“Invalid user name or registration code!”，点击“确定”按钮，再点击“Enter reg.code”按钮，进入注册窗口，重新输入 `User Name` 和 `Registration Code`，点击“OK”按钮，你就可以看到“Thank you for you registering.”。怎么样，注册用户已经是你了！

```
User name:          tongxinshiwo 

Registraiton code:  CT87975921
```

以同样的方法可得到中文注册名的注册码。

```
User name:          同心是我 

Registraiton code:  CT54278840
```

## 破解后的思考

这个软件到期后，会提示“Your evaluation copy has been expired.”

如果这时你点击“Continue”按钮，程序会自动退出，如果你点击右上角“×”按钮则程序会继续运行，再点一次“×”按钮，就会看到游戏画面了。

看来 10 天的限制还不彻底呀。下个版本作者要继续努力哟，嘻嘻……

这是我写的第四篇破解手记，不足之处请大家多多指出，谢谢！

`同心是我` `372.net@371.net`

★破解很容易学的，只要你有信心和毅力再加上适当的破解工具软件，相信你也会成为破解高手的，让我们一起进步吧！★

◆◆◆◆◇◇◇◇◆◆◆◆◇◇◇◇THE END◇◇◇◇◆◆◆◆◇◇◇◇◆◆◆◆ `4dd466d0e1e0cc123afbe504da2462ca`
