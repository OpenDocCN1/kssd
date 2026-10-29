
# [原创]【Code_Review】Tweak开发：给调音量增加震动反馈

### 看雪iOS安全研究群：288567296

# 目录

* [Volbrate（tweak PreferenceBundle）](#volbratetweakpreferencebundle)
* [tweak](#tweak)
* [plist文件](#plist文件)
* [xm文件](#xm文件)
* [震动实现](#震动实现)
* [preferenceBundle](#preferencebundle)
* [新建PreferenceBundle工程](#新建preferencebundle工程)
* [PreferenceBundle文件](#preferencebundle文件)
* [加载PreferenceBundle](#加载preferencebundle)
* [makefile](#makefile)
* [编译](#编译)
* [总结](#总结)
* [参考链接](#参考链接)

# Volbrate（tweak+PreferenceBundle）

最近分析一个开源越狱插件--`Volbrate`，这个插件是给iPhone音量键增加震动的，功能很简单。亮点在于，这个插件提供了在手机设置中做一些功能修改和设定

如图所示：

<img src="upload/attach/201712/695268_dzsgvfn42q9n0je.png" width = "300" height = "550" alt="图片名称" align=center />

通过搜索知道是利用了PreferenceBundle，theos第9个模板--`preference_bundle_modern`。只写过tweak，对PreferenceBundle不了解，所以就边分析边学怎么写PreferenceBundle。

## tweak

有关tweak的编写，资料好多，就不再说了。推荐看狗神的《iOS应用逆向工程》第二版

### plist文件

![plist][3]

可知，其作用于`springboard`

### xm文件

简单来说，就是hook音量键，然后添加震动功能。

关键hook代码：

```mm
%hook VolumeControl
-(void)increaseVolume {
if(volVibrationOptions == 2) {
[FeedbackCall vibrateDevice];
} if(volVibrationOptions == 1 && volMax == YES){
[FeedbackCall vibrateDevice];
} else {
%orig;
}
}
-(void)decreaseVolume {
if(volVibrationOptions == 2) {
[FeedbackCall vibrateDevice];
} if(volVibrationOptions == 1 && volMin == YES){
[FeedbackCall vibrateDevice];
} else {
%orig;
}
}
-(float)volume {
x = %orig;
if(x == 0){
volMin = YES;
} if(x == 1){
volMax = YES;
} if(x > 0 && x < 1) {
volMin = NO;
volMax = NO;
}
return %orig;
}
%end
```

由代码可知，hook `VolumeControl`类里面的三个函数。

`-(float)volume` 获取音量值

`-(void)increaseVolume` 增加一格音量

`-(void)decreaseVolume` 减少一格音量

### 震动实现

tweak中震动的实现是 调用一个private API ：`AudioServicesPlaySystemSoundWithVibration`

Apple官方并没有这个函数的文档。搜索这个函数，出来最多就是`are there apis for custom vibrations in ios`[4]。翻译部分内容：

其函数原型:

`void AudioServicesPlaySystemSoundWithVibration(SystemSoundID inSystemSoundID, id arg, NSDictionary* vibratePattern)`

- 第一个参数：`inSystemSoundID`类型为`SystemSoundID`，就像调用`[AudioServicesPlaySystemSound:]`[5]函数一样，赋值为`kSystemSoundID_Vibrate`可以产生短暂的震动
- 第二个参数：不重要，传参数为`nil`就可
- 第三个参数：`vibratePattern`类型为`NSDictionary`, 根据源代码有两个key, 一个为设置震动时长与连续震动之间的间隙。一个为设置震动的强度。

```mm
+ (void)vibrateDeviceForTimeLengthIntensity:(CGFloat)timeLength vibrationIntensity:(CGFloat)vibeIntensity {
NSMutableDictionary* dict = [NSMutableDictionary dictionary];
NSMutableArray* arr = [NSMutableArray array];
[arr addObject:[NSNumber numberWithBool: YES]]; //vibrate for time length
[arr addObject:[NSNumber numberWithInt: timeLength*1000]];
[arr addObject:[NSNumber numberWithBool: NO]];
[arr addObject:[NSNumber numberWithInt: 50]];
[dict setObject: arr forKey:@"VibePattern"];
[dict setObject:[NSNumber numberWithFloat: vibeIntensity] forKey:@"Intensity"];
AudioServicesPlaySystemSoundWithVibration(kSystemSoundID_Vibrate, nil, dict);
}
```

## preferenceBundle

有意思的是怎么才能写PreferenceBundle，在设置中随时修改某些参数。达到修改tweak功能的作用。比如可以做一个 是否启用hook的开关...

Preference Bundles可以作为iPhone设置中的扩展程序，开发者能编写自己想要的bundles，安装后位于手机`/Library/PreferenceBundles/`目录下。

- Theos的PreferenceBundles模板结合了MobileSubstrate的PreferenceLoader

PreferenceLoaders是MobileSubstrate其中的一个工具，可以把tweak扩展PreferenceBundles注入到iOS的设置中

- `Tweak.xm`不能直接调用PreferenceBundle来获取一些修改后的变量值，而是通过另一种方式，比如从某个`plist`文件读取，变量的`plist`文件位于`/User/Library/Preferences`目录。

### 新建PreferenceBundle工程

PreferenceBundles作为tweak的扩展，**直接就在tweak的工程目录**新建PreferenceBundles工程

利用theos `nic.pl`来创建新建PreferenceBunldes工程

![][7]

如图依次填入，工程名，BundleID，工程前缀

最后的目录结构如图：

![][8]

这时候tweak的`makefile`会自动增加一些东西

![][9]

### PreferenceBundle文件

- `entry.plist`

确定在设置应用中的入口图标，文字等。

![][10]

- `Info.plist`

这个文件保存工程信息，不需做什么修改

- `Root.plist`

`Root.plist`可以看做是PreferenceBundle的UI布局文件。其中要跟tweak交互的变量就声明在这里，比如：

```xml
<key>key</key>
<string>varName</stirng>
```

**`entry.plist` 和 `Root.plist`文件的所用的键值详细内容，可参考`Preferences specifier plist`[11]**

`Info.plist`和`Root.plist`都在`Resources`文件夹里。且工程所用到图片，图标文件也保存在`Resources`目录下。

- 代码文件

模板会新建两个文件，`XXXRootListController.h` 和 `XXXRootListController.m`，`XXX`就是之前设置的工程前缀。

`XXXRootListController`必须继承`PSListController`或者`PSViewController`，且必须实现`- (id)specifiers`方法，因为`PSListController`依赖`_specifiers`来获得metadata和group。

`iphonedevwiki`[6]的示例代码：

```mm
- (id)specifiers {
if (!\_specifiers){
\_specifiers = [[self loadSpecifiersFromPlistName: kNameOfPreferencesPlist target: self] retain];
}
return \_specifiers;
}
```

`kNameOfPreferencePlist`指的就是`Root.plist`。

这些theos都已经替我们做好了。其他逻辑代码就写在`XXXRootListController.m`里，可以有多个`.m`文件。

如何在PreferenceBundle中设置和读取要交互的变量，具体方法可参考`iphonedevwiki`[6]

### 加载PreferenceBundle

在tweak的`constructor`（`%ctor`）中完成PreferenceBundle的加载，

`Preferences`[12]的示例代码：

```mm
@interface NSUserDefaults (Tweak_Category)
- (id)objectForKey:(NSString *)key inDomain:(NSString *)domain;
- (void)setObject:(id)value forKey:(NSString *)key inDomain:(NSString *)domain;
@end
static NSString *nsDomainString = @"com.your.tweak";
static NSString *nsNotificationString = @"com.your.tweak/preferences.changed";
static BOOL enabled;
static void notificationCallback(CFNotificationCenterRef center, void *observer, CFStringRef name, const void *object, CFDictionaryRef userInfo) {
NSNumber *n = (NSNumber *)[[NSUserDefaults standardUserDefaults] objectForKey:@"enabled" inDomain:nsDomainString];
enabled = (n)? [n boolValue]:YES;
}
%ctor {
// Set variables on start up
notificationCallback(NULL, NULL, NULL, NULL, NULL);
// Register for 'PostNotification' notifications
CFNotificationCenterAddObserver(CFNotificationCenterGetDarwinNotifyCenter(),
NULL,
notificationCallback,
(CFStringRef)nsNotificationString,
NULL,
CFNotificationSuspensionBehaviorCoalesce);
// Add any personal initializations
}
```

`notificationCallback()`获取需要的变量值。

`CFNotificationCenterAddObserver()`是`CoreFoundation/CFNotificationCenter.h`中的函数，注册监听事件，设置监听相关变量改变后所做的事。

- 第一个参数：`CFNotificationCenterGetDarwinNotifyCenter()`，不变
- 第二个参数：`NULL`
- 第三个参数：监听到消息后做所的事，一般是重新获取变量值。所以填`notificationCallback`
- 第四个参数：Darwin消息字符串，在`Root.plist`文件中设置相关变量的消息字符串，然后在`Tweak.xm`文件中要写上对于的消息字符串。`key`为`PostNotification`。当对应的变量改变时，就会发送这个消息字符串。然后监听事件就会接受到消息。

比如在`Root.plist`文件中：

```xml
<key>key</key>
<string>varName</stirng>
<key>PostNotification</key>
<string>thisVarDarwinNotification</string>
```

- 第五个参数：`NULL`
- 第六个参数：`CFNotificationSuspensionBehaviorCoalesce`，不变

### makefile

`makefile`的编写跟`tweak`差不多

### 编译

编译是跟`tweak`一起编译，不用做其他操作。

# 总结

如何写PreferenceBundle的中文资料没找到，所以参考的都是英文资料，所以有些地方翻译的不好，大致意思应该明白。写的较为简单，算是基本了解怎么写PreferenceBundle吧。

# 参考链接

<mark class="encrypted">606K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6K6N6r3q4U0K9$3!0$3k6i4u0X3L8r3!0%4i4K6u0W2j5$3!0E0i4K6u0r3M7i4g2W2M7%4c8A6L8$3&6K6i4K6u0r3x3e0t1&6y4U0j5@1y4U0N6Q4x3V1k6S2M7X3g2Q4x3X3c8@1K9r3g2J5k6g2)9J5k6r3q4H3K9i4y4Q4x3X3c8X3L8%4u0Q4x3X3c8U0N6i4y4@1L8$3#2Q4x3X3c8$3K9h3u0J5j5i4c8A6L8$3&6K6i4K6u0V1K9h3&6Q4x3X3c8A6L8%4x3`.</mark>

<mark class="encrypted">b9aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6V1k6i4u0$3z5o6u0Q4x3V1k6q4P5r3y4Z5j5h3&6Y4k6h3&6@1i4K6u0r3N6$3W2C8K9g2)9J5c8W2m8S2M7Y4c8Q4x3X3b7$3i4K6y4m8i4K6u0V1f1s2u0W2k6X3g2J5k6h3&6U0k6i4y4Q4x3V1y4Q4x3X3c8b7M7X3g2X3k6i4u0W2L8X3y4W2M7#2)9J5b7#2)9J5k6r3q4Q4x3X3c8D9K9i4c8@1L8r3g2Q4x3X3c8f1N6$3g2S2K9#2)9J5b7#2)9J5k6r3q4F1k6q4)9J5k6p5S2W2j5i4m8K6i4K6u0V1L8$3k6Q4x3X3c8y4L8%4u0W2i4K6u0V1f1s2u0W2k6X3g2J5k6h3&6U0k6i4x3`.</mark>

<mark class="encrypted">f91K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4y4Z5j5i4u0W2k6r3W2F1M7%4c8S2L8X3y4W2i4K6u0W2L8X3g2@1i4K6u0r3x3U0l9I4y4g2)9J5c8U0l9J5i4K6u0r3M7$3g2@1N6r3W2F1k6%4y4Q4x3X3c8@1K9r3g2Q4x3X3c8J5K9h3N6Z5N6q4)9J5k6s2N6S2P5g2)9J5k6s2u0W2k6s2g2^5i4K6u0r3</mark>

<mark class="encrypted">be7K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3W2H3K9r3!0F1k6h3c8W2N6Y4N6A6K9$3W2Q4x3X3g2F1k6i4c8Q4x3V1k6A6L8X3c8W2P5q4)9J5k6i4m8Z5M7q4)9J5c8W2m8J5k6h3k6W2M7X3g2F1j5$3g2n7N6h3&6V1L8r3g2K6</mark>

<mark class="encrypted">a6aK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3W2H3K9r3!0F1k6h3c8W2N6Y4N6A6K9$3W2Q4x3X3g2F1k6i4c8Q4x3V1k6A6L8X3c8W2P5q4)9J5k6i4m8Z5M7q4)9J5c8W2m8J5k6h3k6W2M7X3g2F1j5$3g2x3L8$3q4V1k6i4t1`.</mark>
