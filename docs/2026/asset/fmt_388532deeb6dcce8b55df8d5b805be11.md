
# [原创]android版foxit去水印去搜索栏修改教程

咳咳，罪过罪过，又XX了一个安卓的软件了。 。

有不太明白清楚的地方可以参考论坛之前的一个帖子：http://bbs.pediy.com/showthread.php?t=135323

又或者看我在蓝魔之家发过的另外一个修改DOCUMENT TO GO的帖子：

848K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4u0S2L8h3!0K6k6X3q4F1M7#2)9J5k6h3y4G2L8g2)9J5c8Y4c8Z5M7X3g2S2k6q4)9J5k6o6R3H3x3o6c8Q4x3X3b7I4i4K6u0V1x3g2)9J5k6h3S2@1L8h3H3`.

前几天无意中发现foxit pdf有安卓版，于是下载试用两下。打开的速度挺快的，而且还是缓冲一页的，不像ezpdf和document to go那样是半页缓冲的。

随便打开一本书。。擦。。毛啊。。偌大的foxit copyright(c) by foxit corporation挺直的竖立在头顶上捏。 。

![](http://res.yruan.com/softpics/pics/2011-06/yruan_com_34101308926576.jpg)

上`apktool`，解包。。在`res\drawable`下可以看到水印的图片躺在那儿。 。

## 去水印方法有二

一，修改水印view的xml里面定义的weight,length大小，修改如下：`android:layout_width="0.0dip"` `android:layout_height="0.0dip"`（文件为`watermark_view.xml`）

二，修改图片，png貌似是可透明的图片，实际上只要把水印的图片换成同样大小dip的透明图片即可。

## 去搜索栏也同样有两个方法

一，修改搜索栏view的xml里面定义的weight,length大小，方法同上。（文件为`doc_search.xml`）

二，修改`classes.dex`里面的代码。把`classes.dex`逆成`jar`包之后，就可以使用`jde`来看代码了。定位到`com.Foxit.Reader`的`DocReaderActivity`。在安卓里面，我的理解，`Activity`就相当于对话框之类的，所以，基本上在找代码的时候，看这么一些个带有关键字的类就可以了。我们看一下这个函数`ToDecideWhatToShowAndWhatToHideTheFuckUI()`

```
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

`boolean bool1 = this.mOnSearchMode;`这里获取是否为搜索模式。

```
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

是搜索模式则显示相应的widget。

最后修改好相应的xml，重新签名就可以了。

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
