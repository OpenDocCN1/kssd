
# android 版 foxit 去水印去搜索栏修改教程

> 原文：[`www.pediy.com/kssd/pediy12/137609.html`](https://www.pediy.com/kssd/pediy12/137609.html)

咳咳，罪过罪过，又 XX 了一个安卓的软件了。。

有不太明白清楚的地方可以参考论坛之前的一个帖子：http://bbs.pediy.com/showthread.php?t=135323

又或者看我在蓝魔之家发过的另外一个修改 DOCUMENT TO GO 的帖子：

http://www.ramosfans.com/thread-8004-1-1.html

前几天无意中发现 `foxit pdf` 有安卓版，于是下载试用两下。打开的速度挺快的，而且还是缓冲一页的，不像 `ezpdf` 和 `document to go` 那样是半页缓冲的。

随便打开一本书。。擦。。毛啊。。偌大的 `foxit copyright(c) by foxit corporation` 挺直的竖立在头顶上捏。。

![`res.yruan.com/softpics/pics/2011-06/yruan_com_34101308926576.jpg`](http://res.yruan.com/softpics/pics/2011-06/yruan_com_34101308926576.jpg)

上 `apktool`，解包。。在 `res\drawable` 下可以看到水印的图片躺在那儿。。

## 去水印方法有二

一，修改水印 view 的 xml 里面定义的 weight,length 大小，修改如下：`android:layout_width="0.0dip"` `android:layout_height="0.0dip"`（文件为 `watermark_view.xml`）

二，修改图片，`png` 貌似是可透明的图片，实际上只要把水印的图片换成同样大小 `dip` 的透明图片即可。

## 去搜索栏也同样有两个方法

一，修改搜索栏 view 的 xml 里面定义的 weight,length 大小，方法同上。（文件为 `doc_search.xml`）

二，修改 `classes.dex` 里面的代码。把 `classes.dex` 逆成 jar 包之后，就可以使用 `jde` 来看代码了。定位到 `com.Foxit.Reader` 的 `DocReaderActivity`。在安卓里面，我的理解，Activity 就相当于对话框之类的，所以，基本上在找代码的时候，看这么一些个带有关键字的类就可以了。我们看一下这个函数 `ToDecideWhatToShowAndWhatToHideTheFuckUI()`

```java
private void ToDecideWhatToShowAndWhatToHideTheFuckUI()
{
  FoxitApplication localFoxitApplication = (FoxitApplication)getApplication();
  boolean bool1 = this.mOnSearchMode;
  boolean bool2 = localFoxitApplication.getReflowState();
  int i = this.mAnnotState;
  int j = getResources().getConfiguration().orientation;
  if (bool2)
  {
    this.mReflowBack.setVisibility(0);
    this.mReflowZoomIn.setVisibility(0);
    this.mReflowZoomOut.setVisibility(0);
    if (this.mToolbarLayout != null)
      this.mToolbarLayout.setVisibility(8);
    if (this.mToolBarFlipper != null)
      this.mToolBarFlipper.setVisibility(8);
    this.mSearchLayout.setVisibility(8);
    this.mPencilSettingFlipper.setVisibility(8);
    this.mSoftMenu.setVisibility(8);
    this.mZoomInButton.setVisibility(8);
    this.mZoomOutButton.setVisibility(8);
    this.mGotoSeekBar.setVisibility(8);
    this.mOnSearchMode = 0;
    this.mAnnotState = 0;
    return;
  }
  this.mReflowBack.setVisibility(8);
  this.mReflowZoomIn.setVisibility(8);
  this.mReflowZoomOut.setVisibility(8);
  this.mSoftMenu.setVisibility(0);
  this.mZoomInButton.setVisibility(0);
  this.mZoomOutButton.setVisibility(0);
  this.mGotoSeekBar.setVisibility(0);
  if (this.mToolbarLayout != null)
    this.mToolbarLayout.setVisibility(0);
  if (this.mToolBarFlipper != null)
    this.mToolBarFlipper.setVisibility(0);
  if (bool1)
  {
    this.mSearchLayout.setVisibility(8);
    this.mPencilSettingFlipper.setVisibility(8);
    if (this.mToolbarLayout != null)
      this.mToolbarLayout.setVisibility(8);
    if (this.mToolBarFlipper != null)
      this.mToolBarFlipper.setVisibility(8);
    this.mAnnotState = 0;
    return;
  }
  this.mSearchLayout.setVisibility(8);
  switch (i)
  {
  default:
    return;
  case 0:
    ViewFlipper localViewFlipper1 = this.mPencilSettingFlipper;
    Runnable localRunnable1 = this.mAnnotSettingDelayRunnable;
    boolean bool3 = localViewFlipper1.removeCallbacks(localRunnable1);
    this.mPencilSettingFlipper.setVisibility(8);
    this.mPencilTool.setImageResource(2130837640);
    this.mEraserTool.setImageResource(2130837630);
    return;
  case 1:
    this.mPencilSettingFlipper.setVisibility(0);
    ViewFlipper localViewFlipper2 = this.mPencilSettingFlipper;
    Animation localAnimation = AnimationUtils.loadAnimation(this.mContext, 2130968577);
    localViewFlipper2.startAnimation(localAnimation);
    if (j == 2)
    {
      this.mSoftMenu.setVisibility(8);
      this.mZoomInButton.setVisibility(8);
      this.mZoomOutButton.setVisibility(8);
      this.mGotoSeekBar.setVisibility(8);
    }
    while (true)
    {
      this.mPencilTool.setImageResource(2130837644);
      this.mEraserTool.setImageResource(2130837630);
      return;
      if (this.mToolBarFlipper.getCurrentView().getId() != 2131296297)
        continue;
      this.mToolBarFlipper.showNext();
    }
  case 2:
  }
  this.mPencilSettingFlipper.setVisibility(8);
  ViewFlipper localViewFlipper3 = this.mPencilSettingFlipper;
  Runnable localRunnable2 = this.mAnnotSettingDelayRunnable;
  boolean bool4 = localViewFlipper3.removeCallbacks(localRunnable2);
  this.mPencilTool.setImageResource(2130837640);
  this.mEraserTool.setImageResource(2130837633);
  if (j == 2)
  {
    this.mSoftMenu.setVisibility(8);
    this.mZoomInButton.setVisibility(8);
    this.mZoomOutButton.setVisibility(8);
    this.mGotoSeekBar.setVisibility(8);
    return;
  }
  if (this.mToolBarFlipper.getCurrentView().getId() != 2131296297)
    return;
  this.mToolBarFlipper.showNext();
}
```

从函数的名字就可以猜到是干什么用的了。再详细看代码，果真是这里。

`boolean bool1 = this.mOnSearchMode;` 这里获取是否为搜索模式。

```java
if (bool1)
{
  this.mSearchLayout.setVisibility(8);
  this.mPencilSettingFlipper.setVisibility(8);
  if (this.mToolbarLayout != null)
    this.mToolbarLayout.setVisibility(8);
  if (this.mToolBarFlipper != null)
    this.mToolBarFlipper.setVisibility(8);
  this.mAnnotState = 0;
  return;
}
```

是搜索模式则显示相应的 widget。

最后修改好相应的 xml，重新签名就可以了。
