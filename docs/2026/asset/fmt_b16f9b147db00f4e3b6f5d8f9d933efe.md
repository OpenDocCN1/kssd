
在手机中安装`magisk.apk`，依次点击“安装”→“选项”→“下一步”→“方式”→“选择修补一个文件”→“选择刚刚存放在手机中的`boot.img`文件”→“开始”。等待执行结束，你会发现在`boot.img`所在的目录中多了一个文件（有时候这个文件在电脑中看不见，在手机中重命名后就能看见了，不知道为啥）。将这个文件拷贝到刷机包`boot.img`所在的目录，将刷机包原本的`boot.img`重命名为`boot.img.bak`，将`magisk`生成的这个文件重命名为`boot.img`。此时，刷机包中的`boot.img`就被`magisk`生成的`boot.img`替换了。

## 刷入`boot.img`

有两种方式刷入修改后的`boot.img`，我喜欢偷懒直接刷机，毕竟点击鼠标更简单哈。

- 因为已经将刷机包中的`boot.img`进行了替换，所以可以简单地再重新刷一下机就可以了。
- 使用`fastboot flash boot boot.img`命令仅刷入`boot.img`。

## 安装`edxposed`

### `nexus5`

笔者最开始使用的设备是`nexus5`，笔者先后经历了：

- 安卓 4.4 系统无法安装新版`magisk`（安装旧版解决）。
- 安装`riru`提示 SDK 版本过低（手机升级安卓 6.0 解决）。
- 安装新版`riru`模块需要 Android 8.0（安装旧版`riru`）。
- 找不到旧版本的`edxposed`（百度了一下午的帖子，找了资源）。
- `edxposed`和`riru`的版本不匹配导致安装失败（又翻了一天的帖子，找了一大堆资源挨个试）。
- 好不容易`edxposed`和`riru`都安装上了，手机重启无法开机。
- 这个安装过程大概历时三天，此时我的心态已然崩溃（因为试岗七天已经过了 4 天，买设备也来不及），直接开始躺平，这种状态一直持续到试岗失败，退出群聊。

![](upload/attach/202304/950902_HACDA9236KZ5F9B.png)

### 小米 6X

事情的转机来自于我老妈说她的小米 6X 电池不太行了。此时的我转念一想，换个手机我手里不就有个安卓 9 的手机了么，就这样，小米 6X 就变成了我的 Android 逆向工程机。

小米 6X 的`edxposed`安装依然遇到版本问题，这里我总结一下使用的版本：

- `Magisk-v23.0.apk`
- `riru-v25.4.4-release.zip`
- `EdXposed-v0.5.2.2_4683-master-release.zip`
- `EdXposedManager-4.6.2-46200-org.meowcat.edxposed.manager-release.apk`

安装 Xposed 插件后，可以看到此时已经成功 Hook（`nexus5`坑我不浅！！！）。

![](upload/attach/202304/950902_ECWY8MZPXDR86JP.png)

## Xposed 插件开发

此时我们先考虑在改界面添加一个`TextView`，那么问题就变成了获取`Context`的问题。根据之前学习的经验，可以通过`findAndHookConstructor`来解决，下面上代码：

```java
package com.example.xposeddemo;
import android.app.Activity;
import android.content.ContentResolver;
import android.content.Context;
import android.database.Cursor;
import android.net.Uri;
import android.os.Bundle;
import android.util.Log;
import android.view.Gravity;
import android.view.ViewGroup;
import android.widget.FrameLayout;
import android.widget.TextView;
import android.widget.Toast;
import android.content.ContentResolver;
import java.lang.reflect.Method;
import java.util.ArrayList;
import java.util.List;
import de.robv.android.xposed.IXposedHookLoadPackage;
import de.robv.android.xposed.XC_MethodHook;
import de.robv.android.xposed.XposedHelpers;
import de.robv.android.xposed.callbacks.XC_LoadPackage;
public class Main implements IXposedHookLoadPackage {
private String packageName = "com.whatsapp";
private String className = packageName + ".contact.picker.ContactPicker";
Context context;
@Override
public void handleLoadPackage(XC_LoadPackage.LoadPackageParam loadPackageParam) throws Throwable {
hookMainAcivityInit(loadPackageParam);
XposedHelpers.findAndHookMethod(className,
loadPackageParam.classLoader, "onCreate", android.os.Bundle.class, new XC_MethodHook() {
@Override
protected void afterHookedMethod(MethodHookParam param) throws Throwable {
super.afterHookedMethod(param);
Log.d("lxz", "hook start");
//获取界面
final Activity mActivity = (Activity) param.thisObject;
//创建一个 TextView
TextView textView = new TextView(context);
// 创建布局，设置参数
FrameLayout.LayoutParams params = new FrameLayout.LayoutParams(
ViewGroup.LayoutParams.WRAP_CONTENT, ViewGroup.LayoutParams.WRAP_CONTENT);
// 设置控件到底端的距离
params.bottomMargin = 100;
// 设置控件的位置
params.gravity = Gravity.BOTTOM | Gravity.CENTER_HORIZONTAL;
// 设置控件文本
textView.setText("hook text");
// 添加 TextView 到 Activity 中
mActivity.addContentView(textView,params);
}
});
}
private void hookMainAcivityInit(XC_LoadPackage.LoadPackageParam loadPackageParam)
{
String packageName = loadPackageParam.packageName;
if(!packageName.equals(packageName))
return;
Class hookClass = XposedHelpers.findClass(
className,loadPackageParam.classLoader);
XposedHelpers.findAndHookConstructor(
hookClass,
new XC_MethodHook() {
@Override
protected void beforeHookedMethod(MethodHookParam param) throws Throwable {
super.beforeHookedMethod(param);
context = (Context) param.thisObject;
}
}
);
}
}
```

重启手机后也如预期般地显示了`hook text`字样。

![](upload/attach/202304/950902_D6MBD2MGGY6W8PS.png)

接下来我们只要遍历通讯录后把内容设置到`textView`上就可以了。这部分的内容在之前的 Android 安全笔记中也有提过，读写系统应用通讯录的`ContentProvider`，其重点在于以下几点：

- 读写系统应用通讯录的`ContentProvider`需要权限，分别为`android.permission.READ_CONTACTS`和`android.permission.READ_CONTACTS`。
- 数据库中直接看到的`mimetype_id`项并不存在，该项为多表查询，真实字段为`mimetype`，可以通过在代码中遍历列名观察到。
- `mimetype`是`String`类型，而不是在数据库中看到的`int`类型。
- 添加联系人时，应先在`raw_contacts`中添加一个空项，然后再在`data`中添加各种数据。
- 注意：虽然添加了读写通讯录的权限，但依然要在手机中手动配置应用读写通讯录的权限，这里我踩过坑！！！

如果不会的话请移步我之前的笔记中`ContentProvider`部分，这里我们就不啰嗦了，直接上完整代码。
