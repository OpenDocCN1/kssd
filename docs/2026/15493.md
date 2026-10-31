
# [原创]破解7notes with mazec体验版的10天限制

声明：本文章仅做技术研究，请勿用于非法用途。

这个软件是日文手写输入法，对于工作当中经常要用日文进行书写的人来说还是很不错的，但是这个版本是体验版，有10天的限制，10天过后会弹出提示不能继续使用，如下图：

![](https://bbs.kanxue.com/upload/attach/201211/150432_z1avynq8yk6etay.png)

用到的工具：`APKTOOL`、`JD-GUI`、`Notepad++`

## 1、定位过期判断逻辑

用`APKTOOL`反编译该程序，由弹出的过期提示，在smali文件中搜索关键字："Expired"，如下图：

![](https://bbs.kanxue.com/upload/attach/201211/150432_vhnyb74jz2md32s.png)

其中`isTrialExpired()`方法值得我们注意，由字面意思可以推测，这个方法应该是判断是否过期。因此，我们再次搜索关键字：`isTrialExpired`，如下图：

![](https://bbs.kanxue.com/upload/attach/201211/150432_qhawief36yggle5.png)

其中有对该方法的定义和调用该方法的地方，我们不妨先看看这个方法的定义，在`smali\com\sevenknowledge\mazec\MazecIms.smali`文件中的定义：

```
.method public isTrialExpired()Z

    .locals 7



    .prologue

    const-wide/16 v5, 0x0



    const/4 v2, 0x0



    .line 207

    sget-object v3, Lcom/sevenknowledge/common/BuildOption;->PRODUCT_TYPE:Lcom/sevenknowledge/common/BuildOption$ProductType;



    sget-object v4, Lcom/sevenknowledge/common/BuildOption$ProductType;->TRIAL:Lcom/sevenknowledge/common/BuildOption$ProductType;



    if-ne v3, v4, :cond_3



    .line 209

    iget-wide v3, p0, Lcom/sevenknowledge/mazec/MazecIms;->mExpireBase:J



    cmp-long v3, v3, v5



    if-ltz v3, :cond_0



    iget-wide v3, p0, Lcom/sevenknowledge/mazec/MazecIms;->mExpireTime:J



    cmp-long v3, v3, v5



    if-gez v3, :cond_1



    .line 210

    :cond_0

    invoke-direct {p0}, Lcom/sevenknowledge/mazec/MazecIms;->initExpireDate()V



    .line 212

    :cond_1

    invoke-static {}, Ljava/lang/System;->currentTimeMillis()J



    move-result-wide v0



    .line 213

    .local v0, t:J

    iget-wide v3, p0, Lcom/sevenknowledge/mazec/MazecIms;->mExpireBase:J



    cmp-long v3, v0, v3



    if-ltz v3, :cond_2



    iget-wide v3, p0, Lcom/sevenknowledge/mazec/MazecIms;->mExpireTime:J



    cmp-long v3, v3, v0



    if-gez v3, :cond_3



    :cond_2

    const/4 v2, 0x1



    .line 216

    .end local v0           #t:J

    :cond_3

    return v2

.end method

```

由于smali语句不太方便阅读，因此我们用`JD-GUI`工具来阅读对应的JAR文件，上面这一段代码对应的java语句：

```
 public boolean isTrialExpired()

  {

    int i = 0;

    if (BuildOption.PRODUCT_TYPE == BuildOption.ProductType.TRIAL)

    {

      if ((this.mExpireBase < 0L) || (this.mExpireTime < 0L))

        initExpireDate();

      long l = System.currentTimeMillis();

      if ((l < this.mExpireBase) || (this.mExpireTime < l))

        i = 1;

    }

    return i;

  }


```

很明显，返回值决定了是否过期，返回值`i=0`时，没有过期，`i=1`时，过期。所以，我们在这里可以改变返回值，让`i`始终=0，就可以永不过期。

## 2、修改smali代码使返回值始终为0

在smali文件中，我们在该方法的末尾如此修改：

```
   if-gez v3, :cond_3



    :cond_2

    const/4 v2, 0x1



    .line 216

    .end local v0           #t:J

    :cond_3

    [COLOR="Red"]const/4 v2, 0x0[/COLOR]

    return v2

.end method
```

## 3、修改ProductType的初始化值

还有一种改法，我们看到，`if (BuildOption.PRODUCT_TYPE == BuildOption.ProductType.TRIAL)`，`i`的值才为1，那么`BuildOption.PRODUCT_TYPE`的值都有哪些呢？

通过`jd-gui`，我们找到在`BulidOption.class`中定义了`ProductType`类型有哪些：

```
 public static enum ProductType

  {

    static

    {

      FREE = new ProductType("FREE", 2);

      ProductType[] arrayOfProductType = new ProductType[3];

      arrayOfProductType[0] = PREMIUM;

      arrayOfProductType[1] = TRIAL;

      arrayOfProductType[2] = FREE;

      $VALUES = arrayOfProductType;

    }

  }
```

可以看到`ProductType`有三种类型：`PREMIUM`、`TRIAL`、`FREE`。而在类的成员变量当中就已经初始化了`ProductType`的值：

```
public class BuildOption

{

  public static final String APP_PACKAGE_NAME = "com.sevenknowledge.sevennotestrial";

  public static final boolean DEVELOP_MODE = false;

  public static final DeviceName DEVICE_NAME;

  public static final DeviceType DEVICE_TYPE;

  public static final DeviceVendor DEVICE_VENDOR;

  public static final boolean KEYBOARD_CONVERSION_MODE = false;

  public static final MarketType MARKET_TYPE;

  public static final String PRODUCT_NAME = "AndroidS-Edit-J-T";

[COLOR="red"]  public static final ProductType PRODUCT_TYPE = ProductType.TRIAL;[/COLOR]

  public static final boolean VALID_ENGLISH_MODE;
```

因此我们可以修改初始化的值为`ProductType.PREMIUM`：

`public static final ProductType PRODUCT_TYPE = ProductType.PREMIUM`

对应到smali文件中的修改：

```
# direct methods

.method static constructor <clinit>()V

    .locals 1



    .prologue

    .line 32

    sget-object v0, Lcom/sevenknowledge/common/BuildOption$ProductType;->[COLOR="red"]PREMIUM[/COLOR]:Lcom/sevenknowledge/common/BuildOption$ProductType;



    sput-object v0, Lcom/sevenknowledge/common/BuildOption;->PRODUCT_TYPE:Lcom/sevenknowledge/common/BuildOption$ProductType;
```

## 4、重建并测试

最后重建APK，签名，测试，更改系统时间超过10天，不再弹出限制，可以继续使用。

APK附件太大，无法上传，给出网盘链接

原版：

023K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4m8S2L8W2)9J5k6h3u0S2K9h3c8#2i4K6u0W2j5$3!0E0i4K6u0r3M7$3S2S2M7X3g2Q4x3V1k6D9K9h3&6C8i4K6y4r3M7$3S2S2M7X3g2A6k6q4)9K6c8o6p5K6y4e0b7H3z5q4)9J5y4Y4g2C8i4K6y4p5y4U0l9@1y4o6j5H3x3U0t1J5

修改后：

ea3K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4m8S2L8W2)9J5k6h3u0S2K9h3c8#2i4K6u0W2j5$3!0E0i4K6u0r3M7$3S2S2M7X3g2Q4x3V1k6D9K9h3&6C8i4K6y4r3M7$3S2S2M7X3g2A6k6q4)9K6c8o6p5K6y4e0b7H3z5g2)9J5y4Y4g2C8i4K6y4p5y4U0l9@1y4o6j5H3x3U0t1J5

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

上传的附件：

- [过期提示.png](javascript:void(0);)

  （34.52kb，4次下载）

- [搜索EXPIRED.png](javascript:void(0);)

  （15.45kb，5次下载）

- [搜索isTrialExpired.png](javascript:void(0);)

  （15.37kb，10次下载）
