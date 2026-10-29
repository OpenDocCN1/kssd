
# [原创]海卓apn去广告破vip

声明：本文章仅做技术研究，请勿用于非法用途。

这个软件虽然小，功能还是做得不错的。

就是时不时的弹出广告来。。。　比较不爽。

于是今天想研究一番。

## 软件信息

> 更新日期：2012年1月4日
> 当前版本：2.3.6
> 所需的 Android 版本：1.6 及更高版本
> 类别：工具

首先帖几个分析要用到的 `opcode` 知识：

```
array-length vx,vy	Calculates the number of elements of the array referenced by vy and puts the length value into vx.
const/4 vx,lit4	Puts the 4 bit constant into vx
	1221 - const/4 v1, #int2
 Moves literal 2 into v1. The destination register is in the lower 4 bit in the second byte, the literal 2 is in the higher 4 bit.
aget-object vx,vy,vz	Gets an object reference value of an object reference array into vx. The array is referenced by vy and is indexed by vz.	
4602 0200 - aget-object v2, v2, v0
 Gets an object reference array element. The array is referenced by v2 and the element is indexed by v0. The element will be put into v2.
invoke-virtual { parameters }, methodtocall	Invokes a virtual method with parameters.
6E53 0600 0421 - invoke-virtual { v4, v0, v1, v2, v3}, Test2.method5:(IIII)V // method@0006
 Invokes the 6th method in the method table with the following arguments: 
v4 is the "this" instance, v0, v1, v2, and v3 are the method parameters. The method has 5 arguments (4 MSB bits of the second byte)5.
move-result-wide vx	Move the long/double result value of the previous method invocation into vx,vx+1.
	0B02 - move-result-wide v2
 Move the long/double result value of the previous method invocation into v2,v3.
iput vx,vy, field_id	Puts vx into an instance field. The instance is referenced by vy.
iput-wide vx,vy, field_id	Puts the wide value located in vx and vx+1 registers into an instance field. The instance is referenced by vy.
sget-object vx,field_id	Reads the object reference field identified by the field_id into vx.
goto/16 target	Unconditional jump by 16 bit offset2.
```

先用 `adb` 把程序从手机里面拖出来，再用 `apktool` 反编译。

首先打算采取暴破的方式，发现它验证采取的是从网络。

验证关键方法（函数）：

```
  iget-object v2, v2, Lnet/hidroid/common/user/a;->f:Ljava/lang/String;

    invoke-virtual {v3, v2}, Ljava/lang/StringBuilder;->append(Ljava/lang/String;)Ljava/lang/StringBuilder;

    const-string v2, "dd7K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8W2!0q4z5q4!0n7c8W2)9&6z5g2!0q4z5g2)9^5y4#2)9^5b7#2!0q4y4#2!0n7c8q4)9&6x3g2!0q4y4g2)9&6c8q4)9^5x3q4!0q4y4g2!0n7x3g2)9^5c8W2!0q4z5q4)9&6y4q4!0n7c8q4!0q4y4W2)9^5c8g2)9^5z5g2)9J5c8X3q4U0N6r3W2$3j5i4c8W2i4K6g2X3j5i4g2@1K9q4)9J5k6i4m8Z5M7l9`.`."

    invoke-virtual {v1}, Ljava/lang/StringBuilder;->toString()Ljava/lang/String;

    move-result-object v1

    invoke-static {p0, v2, v1}, Lnet/hidroid/common/b/f;->a(Landroid/content/Context;Ljava/lang/String;Ljava/lang/String;)Ljava/lang/String;

    move-result-object v1

    if-eqz v1, :cond_0

    const-string v2, ","

    invoke-virtual {v1, v2}, Ljava/lang/String;->split(Ljava/lang/String;)[Ljava/lang/String;

    move-result-object v2

    if-eqz v2, :cond_2

    array-length v3, v2

    const/4 v4, 0x2

    if-eq v3, v4, :cond_3
```

提交数据到 `active_auth.php`，把从服务器返回的数据以逗号分隔为数组，
取数组第1个元素 转为整形送给 `Ljava/lang/Integer;->parseInt(Ljava/lang/String;)I`
取返回结果的第2个元素，把结果存入 `Lnet/hidroid/common/user/c;->a:I`
最后返回一用户对象用于进一验证。分析到这里，没有找到它那个无条件跳转的位置　（`goto/16 :goto_0`）

于是接着看下去，

如果点击购买，会显示 `order.php` 的内容，
要求填写：

购买软件:
用户名
密码
应用编号
IMEI

提交这些信息后，服务器会返回数据：

```
    const-string v2, "005K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8W2!0q4z5q4!0n7c8W2)9&6z5g2!0q4z5g2)9^5y4#2)9^5b7#2!0q4y4g2)9^5y4g2!0n7y4#2!0q4y4q4!0n7c8q4)9&6x3#2!0q4y4#2!0n7c8q4)9&6x3g2!0q4y4g2!0n7c8q4)9^5x3q4!0q4y4g2!0n7x3g2)9^5c8W2!0q4z5q4)9&6y4q4!0n7c8q4)9J5c8X3!0J5k6r3g2J5i4K6u0W2M7r3S2H3"

    invoke-virtual {v1}, Ljava/lang/StringBuilder;->toString()Ljava/lang/String;

    move-result-object v1

    invoke-static {p0, v2, v1}, Lnet/hidroid/common/b/f;->a(Landroid/content/Context;Ljava/lang/String;Ljava/lang/String;)Ljava/lang/String;

    move-result-object v1

    if-eqz v1, :cond_1

    const-string v2, ","
否则以,分隔返回结果为数组
    invoke-virtual {v1, v2}, Ljava/lang/String;->split(Ljava/lang/String;)[Ljava/lang/String;
保存函数执行结果到v2
    move-result-object v2

如果返回结果是0(注册失败)，直接跳到cond_1
    if-eqz v2, :cond_0

    array-length v3, v2

    const/4 v4, 0x6
要求返回的结果转换为数组后，该数组要有6个元素
    if-eq v3, v4, :cond_2
```

下面再看它对这6个元素是怎么处理的：

```
    :cond_2
    const/4 v1, 0x0

    aget-object v1, v2, v1

    invoke-static {v1}, Ljava/lang/Integer;->parseInt(Ljava/lang/String;)I

    move-result v1

    iput v1, v0, Lnet/hidroid/common/user/c;->a:I

    const/4 v1, 0x1

    aget-object v1, v2, v1

    invoke-static {v1}, Ljava/lang/Double;->parseDouble(Ljava/lang/String;)D

    move-result-wide v3

    iput-wide v3, v0, Lnet/hidroid/common/user/c;->c:D

    const/4 v1, 0x2

    aget-object v1, v2, v1

    invoke-static {v1}, Ljava/lang/Double;->parseDouble(Ljava/lang/String;)D

    move-result-wide v3

    iput-wide v3, v0, Lnet/hidroid/common/user/c;->d:D

    const/4 v1, 0x3

    aget-object v1, v2, v1

    invoke-static {v1}, Ljava/lang/Double;->parseDouble(Ljava/lang/String;)D

    move-result-wide v3

    iput-wide v3, v0, Lnet/hidroid/common/user/c;->e:D

    const/4 v1, 0x4

    aget-object v1, v2, v1

    iput-object v1, v0, Lnet/hidroid/common/user/c;->f:Ljava/lang/String;

    const/4 v1, 0x5

    aget-object v1, v2, v1

    iput-object v1, v0, Lnet/hidroid/common/user/c;->b:Ljava/lang/String;
    :try_end_0
    .catch Ljava/lang/Exception; {:try_start_0 .. :try_end_0} :catch_0

    goto :goto_0

    :catch_0
    move-exception v1

    sget-object v2, Lnet/hidroid/common/user/b;->a:Ljava/lang/String;

    const-string v3, "network error"

    invoke-static {v2, v3, v1}, Lnet/hidroid/common/b/b;->a(Ljava/lang/String;Ljava/lang/String;Ljava/lang/Throwable;)V

    goto :goto_0
.end method
```

从这里可以看出，根据其返回数据的结果，可以伪造 `dns`，即修改 `hosts` 指向它的购买和验证服务器，然后，在伪造的服务器上面放几个相应的页面用于返回数据给应用。

不过这样做太麻烦了。

于是继续往下看：

```
.method public static a(Landroid/content/Context;Ljava/lang/String;)V
    .locals 4

    invoke-static {p0}, Landroid/preference/PreferenceManager;->getDefaultSharedPreferences(Landroid/content/Context;)Landroid/content/SharedPreferences;

    move-result-object v0

    invoke-interface {v0}, Landroid/content/SharedPreferences;->edit()Landroid/content/SharedPreferences$Editor;

    move-result-object v0

    const-string v1, "KEY_IS_VIP"

    const/4 v2, 0x1

    invoke-interface {v0, v1, v2}, Landroid/content/SharedPreferences$Editor;->putBoolean(Ljava/lang/String;Z)Landroid/content/SharedPreferences$Editor;

    const-string v1, "KEY_VIP_USERNAME"

    invoke-interface {v0, v1, p1}, Landroid/content/SharedPreferences$Editor;->putString(Ljava/lang/String;Ljava/lang/String;)Landroid/content/SharedPreferences$Editor;

    invoke-interface {v0}, Landroid/content/SharedPreferences$Editor;->commit()Z

    invoke-static {}, Landroid/webkit/CacheManager;->getCacheFileBaseDir()Ljava/io/File;

    move-result-object v0

    if-eqz v0, :cond_0

    sget-object v1, Lnet/hidroid/common/user/b;->a:Ljava/lang/String;

    const-string v2, "cacheFileBaseDir exists"

    invoke-static {v1, v2}, Lnet/hidroid/common/b/b;->a(Ljava/lang/String;Ljava/lang/String;)V

    invoke-virtual {v0}, Ljava/io/File;->listFiles()[Ljava/io/File;

    move-result-object v1

    if-eqz v1, :cond_0

    array-length v2, v1

    const/4 v0, 0x0
    :goto_0
    if-lt v0, v2, :cond_1

    :cond_0
    return-void

    :cond_1
    aget-object v3, v1, v0

    invoke-virtual {v3}, Ljava/io/File;->delete()Z

    add-int/lit8 v0, v0, 0x1

    goto :goto_0
.end method
```

这下人品好了。。。看到 `KEY_IS_VIP` 了没？

`Landroid/content/SharedPreferences$Editor;->putString(Ljava/lang/String;Ljava/lang/String;)Landroid/content/SharedPreferences$Editor;` 是 `android` 系统 `api` 里面用于写入应用配置的。

`android/preference/PreferenceManager;->getDefaultSharedPreferences` 获取应用的默认配置对象并送给 `p0`

`const-string v1, "KEY_IS_VIP"`
