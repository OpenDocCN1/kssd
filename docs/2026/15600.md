
# [原创]android逆向--胆儿肥而美 （unity3d游戏初探）第一篇

## unity3d初探

ps：此demo是好久之前搞的，今天发一下，因为网上大多数关于android逆向unity3d的文章几乎都的修改金币，当初也看过几篇相关的，都是搜索`Currency`等关键字直接就可以定位到，但是我在做的过程中并没有这么容易，还是有一些坑，所以把心路历程再次记录下来。

所需工具：【.NET Reflector(查看)】【Reflexil(是netReflector的一个插件，用于修改并保存)】

先上游戏的图，应该很多人玩过。在此不修改金币，而是删除与游戏无关的控件，因为如果不小心触碰到会有弹框，以及支付（其实在为后面的胆儿肥而美做准备）

![](https://bbs.kanxue.com/upload/attach/202004/880781_3ZYAMM6XJP53B7R.jpg)

反编译app，发现是基于unity3d引擎开发的（`Assembly-CSharp.dll`是特征，同时一般关键逻辑都写在这个dll里面）

![](https://bbs.kanxue.com/upload/attach/202004/880781_J6A25EQ2B72QTYH.jpg)

打开.NET Reflector，把`Assembly-CSharp.dll`拖入(注意只能在原位置拖入，还不错没有加密)，并搜索`setting`（为什么是`setting`不是其他的？看第一附图中的设置按钮）。

定位到`DisableSettingButton()`方法：

```
public void DisableSettingButton()

{

    this.settingsButton.DOFade(0f, 0.25f);

    this.SettingsBG.DOFade(0f, 0.25f);

    this.AudioButton.DOFade(0f, 0.25f);

    this.VibrationButton.DOFade(0f, 0.25f);

    this.RestorePurchaseButton.DOFade(0f, 0.25f);

    this.GDPRButton.DOFade(0f, 0.25f);

    base.Invoke("DisableSettings", 0.5f);

}
```

为什么是`DisableSettingButton()`方法？（因为我在人群中看了你一样，久久无法忘记你的容颜）

因为`settingsButton` `SettingsBG` `AudioButton` `VibrationButton` `RestorePurchaseButton` `GDPRButton`这几个关键字，和游戏里面的刚好11对应，于是我赌你的枪里没有子弹。

![](https://bbs.kanxue.com/upload/attach/202004/880781_GBGD8846U2GA9RM.jpg)

需要删除的是`RestorePurchaseButton`和`GDPRButton`按钮（他们有烦人的支付弹窗）

点击Tools>Reflexil 使用reflexil修改il代码，il代码又是什么？搞不懂啊，没接触过呀，没事儿，我们先到人群中去看一眼。

我们发现语法其实很简单，都是11对应的，每条语句以`pop`指令隔开，这就简单了。

找到：

```
this.RestorePurchaseButton.DOFade(0f, 0.25f);
this.GDPRButton.DOFade(0f, 0.25f);
```

这两条语句块删除，保存，重打包，安装，搞定！上面的一系列操作速度要快，动作一定要帅。然后会发现，什么？没有效果，按钮依旧存在？没关系，这是我们搞逆向的常规操作，一次就成功的时候是极少的。

![](https://bbs.kanxue.com/upload/attach/202004/880781_7G8BZFPH6XUC26C.jpg)

重新分析dll：通过分析`RestorePurchaseButton`和`GDPRButton`关键字的调用关系，找到方法`OnSettingsButtonPressed()`，方法体和游戏里面设置按钮对应，于是我又赌你的枪里没有子弹。

```
public void OnSettingsButtonPressed()

{

	if (!this.animmationRunning)

	{

		AudioManager.Instance.playSFX("Button", 0.5f);

		if (!this.CloseOrOpen)

		{

			this.animmationRunning = true;

			ShortcutExtensions.DORotate(this.settingsButton.transform, new Vector3(0f, 0f, -90f), 0.5f, RotateMode.Fast);

			ShortcutExtensions.DOScaleX(this.SettingsBG.transform, 1f, 0.5f).SetEase<Tweener>(Ease.OutBack);

			float delay = 0f;

			ShortcutExtensions.DOScale(this.AudioButton.transform, 1f, 0.2f).SetEase<Tweener>(Ease.OutBack).SetDelay<Tweener>(delay);

			delay += 0.1f;

			ShortcutExtensions.DOScale(this.VibrationButton.transform, 1f, 0.2f).SetEase<Tweener>(Ease.OutBack).SetDelay<Tweener>(delay);

			delay += 0.1f;

			ShortcutExtensions.DOScale(this.RestorePurchaseButton.transform, 1f, 0.2f).SetEase<Tweener>(Ease.OutBack).SetDelay<Tweener>(delay);

			delay += 0.1f;

			ShortcutExtensions.DOScale(this.GDPRButton.transform, 1f, 0.2f).SetEase<Tweener>(Ease.OutBack).SetDelay<Tweener>(delay).OnComplete<Tweener>(() => this.animmationRunning = false);

		}

		else

		{

			ShortcutExtensions.DORotate(this.settingsButton.transform, new Vector3(0f, 0f, 0f), 0.5f, RotateMode.Fast);

			ShortcutExtensions.DOScaleX(this.SettingsBG.transform, 0f, 0.5f).SetEase<Tweener>(Ease.InBack);

			float num2 = 0f;

			ShortcutExtensions.DOScale(this.GDPRButton.transform, 0f, 0.2f).SetEase<Tweener>(Ease.InBack).SetDelay<Tweener>(num2);

			num2 += 0.1f;

			ShortcutExtensions.DOScale(this.RestorePurchaseButton.transform, 0f, 0.2f).SetEase<Tweener>(Ease.InBack).SetDelay<Tweener>(num2);

			num2 += 0.1f;

			ShortcutExtensions.DOScale(this.VibrationButton.transform, 0f, 0.2f).SetEase<Tweener>(Ease.InBack).SetDelay<Tweener>(num2);

			num2 += 0.1f;

			ShortcutExtensions.DOScale(this.AudioButton.transform, 0f, 0.2f).SetEase<Tweener>(Ease.InBack).SetDelay<Tweener>(num2).OnComplete<Tweener>(() => this.animmationRunning = false);

		}

		this.CloseOrOpen = !this.CloseOrOpen;

	}

}
```

重新修改dll：删除下面四条代码，删除方式同上

```
ShortcutExtensions.DOScale(this.RestorePurchaseButton.transform, 1f, 0.2f).SetEase<Tweener>(Ease.OutBack).SetDelay<Tweener>(delay);
ShortcutExtensions.DOScale(this.GDPRButton.transform, 1f, 0.2f).SetEase<Tweener>(Ease.OutBack).SetDelay<Tweener>(delay).OnComplete<Tweener>(() => this.animmationRunning = false);
ShortcutExtensions.DOScale(this.RestorePurchaseButton.transform, 0f, 0.2f).SetEase<Tweener>(Ease.InBack).SetDelay<Tweener>(num2);
ShortcutExtensions.DOScale(this.GDPRButton.transform, 0f, 0.2f).SetEase<Tweener>(Ease.InBack).SetDelay<Tweener>(num2);
```

保存，重打包，安装！搞定！

![](https://bbs.kanxue.com/upload/attach/202004/880781_NUEFPDMTWQZVYJ3.jpg)

现在就还剩屏幕两边的四个按钮了，搞起。

分析dll：因为四个按钮中有一个的名字叫做【VIP Access】，以此作为关键字搜索，找到方法`CheckForButtonState()`，通过查找`CheckForButtonState()`的调用关系找到方法`EnableHomePanel()`。为什么确定是这里？和上面一样，代码和实际控件对应，所以我猜你抢里没有子弹。

![](https://bbs.kanxue.com/upload/attach/202004/880781_YRUM3JQM4S2P5RD.jpg)

删除整个方法体。为什么是删除整个方法体？你都11对应，不删你删谁呢？（注意需要保留最后的`ret`指令）

![](https://bbs.kanxue.com/upload/attach/202004/880781_24XVSTW7NR8E26K.jpg)

打包，安装，搞定！

![](https://bbs.kanxue.com/upload/attach/202004/880781_YYP6K27EKGRGR87.jpg)

好了第一部分结束了，现在我们来聊聊第二部分的话题。

## 胆儿肥而美

好了，我们重打包了一个app，并取消了他的一些限制，我们还能做些什么？

有人会说 我们还能继续改，加金币，加能力。或许把。

但是据我所知的，一些人是可以让它产生一定收入的，这个就需要胆儿肥了。但是的确是有人正在这样做（反正我是不敢的）。往后本系列就分析这样做需要那些要求（仅在技术层面），并可能加以实现。

1.广告方式。添加广告接口到重打包的应用，通过千展和点击盈利。

往往重打包的apk已经有广告了，如本apk好像就是facebookads和googleads，如果需添加自己的广告，最好的方式就是破解广告sdk，让广告sdk认为我是源apk，还能拿给别人用，爽歪歪。（一些原因，非本系列重点）

2.内容付费。什么内容可以让我付费观看？反正我也不知道。

用上游戏app举例，因为已经去除了他本身的限制，或许可以添加一些我们自己定义的玩法。

如：每天免费玩`n`次，第`n`次开始需要付费才能继续玩。本系列往后就以此为模板，相关app的分析工作。

总的来说

按目前使用习惯会牵涉以下接口：

1.两个大厂指定金额收款二维码即时生成

2.交易记录的获取

3.转账。

好了，本篇完！拜了个拜。

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
