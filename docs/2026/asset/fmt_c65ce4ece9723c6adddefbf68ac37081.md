
但还是要注意一个 APK 文件中只会有一个 `R` 类，虽说每个库模块都会生成一个对应的 `R` 类，用于引用该模块中的资源。但是这些 `R` 类会在编译后合并成一个 `R` 类，最终打包到 APK 文件中。这个合并的 `R` 类包含了所有模块中的资源 ID，可以在应用程序的代码中引用这些资源。

现在了解完了库模块，那我们继续讲，现在已经定义好了资源 ID 吗？还没有，除了这里，我们还需要在一处地方定义我们布局文件的资源 ID，那就是 `values\public.xml` 文件中。

`values\public.xml` 是一个用于指定资源 ID 的 XML 文件。该文件中包含了一系列的 `<public>` 标签，每个标签定义了一个资源的名称、类型和 ID。这些资源可以是应用程序内部使用的资源，也可以是外部库或框架中定义的资源。

反编译后 `values` 和 `values-xx` 文件夹会在 `res` 文件夹下，实则 `values` 这些文件以及文件夹就是未反编译时的 `resources.arsc` 文件，这里应该是 `apktool` 反编译之后把它们放在了一起。

但我去那一看，发现一个问题：`values\public.xml` 文件中定义的资源 ID 数量比 `R$layout` 中定义的资源 ID 多得多。

![](upload/attach/202307/926486_9FZX492J2BEBGYR.png)

看这密密麻麻类型为 `layout` 的 `<public>`，还有好多类型为 `layout` 的 `<public>` 没法截下来，可见数量之多，但这不应该呀！`R$layout` 中定义的资源 ID 数量应该与 `public.xml` 文件中定义的资源 ID 数量一致呀！我找了会，我找到了这些：

![](upload/attach/202307/926486_JHEJA8JVNMG8NDV.png)

![](upload/attach/202307/926486_EHJ9UKE8JTPAJ3M.png)

我发现除了 `com.zj.wuaipojie` 下的 `R` 类，在 `com` 下的其他包里也有 `R` 类，那么根据前面所讲的库模块，`public.xml` 中定义的资源 ID 数量是这些 `R` 类定义的资源 ID 数量之和。

那么我们就先去 `public.xml` 文件给我们写的布局文件定义资源 ID

![](upload/attach/202307/926486_YD7MHA8PNRHTGFX.png)

开始还看错了，给了个 `0x7f0b003f`，后面去修改 `R$layout.smali` 时才发现这个资源 ID 应该已经存在了，然后在这文本文件中一查确实已经存在，就赶忙修改成了 `0x7f0b007e`，我们自己给资源声明资源 ID 一定要注意唯一性。

我们下一步需要将 `R$layout.smali` 中的资源 ID 和 `MyActivity` 中使用的资源 ID 给修改了。

![](upload/attach/202307/926486_NXDADTHWNK2EWDR.png)

![](upload/attach/202307/926486_ZJ5B4AMXR589DQ8.png)

修改完成，我们就完成第二步了！我们的布局文件已经放好并且定义好了资源 ID，但是我们的 `Activity` 只放到了反编译文件夹中却没有去声明它，那我们下一步就是去声明 `Activity` 并设为默认启动。

### 第三步：声明 Activity 并设为默认启动

我们来看一下配置信息：

```xml
<!-- <?xml version="1.0" encoding="utf-8" standalone="no"?> 表示这个XML文件使用的XML版本是1.0，编码格式为UTF-8，依赖于外部文档 -->
<?xml version="1.0" encoding="utf-8" standalone="no"?>
<!-- 定义应用程序的清单文件 -->
<!-- <manifest> 标签是清单文件的根标签，xmlns:android="http://schemas.android.com/apk/res/android"它定义了Android命名空间，命名空间用于标识 Android 框架定义的元素和属性，以及应用程序定义的元素和属性。而"http://schemas.android.com/apk/res/android"这个URL将作为Android命名空间用于存储XML元素和属性，并通过命名空间绑定和前缀来简化代码。如果不知道是什么意思，那就理解为这是一个导入库，当使用 Android 标签时，需要在标签前加上 android: 前缀。 -->
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
android:compileSdkVersion="32"
android:compileSdkVersionCodename="12"
package="com.zj.wuaipojie"
platformBuildVersionCode="32"
platformBuildVersionName="12">
<!-- 定义应用程序的权限 -->
<uses-permission android:name="android.permission.INTERNET"/>
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
<!-- 定义应用程序的组件 -->
<application
android:allowBackup="true"
android:appComponentFactory="androidx.core.app.CoreComponentFactory"
android:dataExtractionRules="@xml/data_extraction_rules"
android:extractNativeLibs="false"
android:fullBackupContent="@xml/backup_rules"
android:icon="@mipmap/ic_launcher"
android:label="@string/app_name"
android:networkSecurityConfig="@xml/network_config"
android:supportsRtl="true"
android:theme="@style/Theme.Wuaipojie">
<!-- 定义应用程序的活动 -->
<activity android:exported="true" android:name="com.zj.wuaipojie.ui.ChallengeFifth"/>
<activity android:exported="true" android:name="com.zj.wuaipojie.ui.ChallengeFourth"/>
<activity android:exported="false" android:name="com.zj.wuaipojie.ui.ChallengeThird"/>
<activity android:exported="false" android:name="com.zj.wuaipojie.ui.ChallengeSecond"/>
<activity android:name="com.zj.wuaipojie.ui.AdActivity"/>
<activity android:exported="true" android:label="@string/app_name" android:name="com.zj.wuaipojie.ui.MainActivity">
<intent-filter>
<action android:name="android.intent.action.MAIN"/>
<category android:name="android.intent.category.LAUNCHER"/>
</intent-filter>
</activity>
<activity android:name="com.zj.wuaipojie.ui.ChallengeFirst"/>
</application>
</manifest>
```

我们看上面的代码，我们需要去声明我们的 `Activity` 并设为默认启动，那就需要将 `com.zj.wuaipojie.ui.MainActivity` 替换成我们的 `Activity`，但这样会出现问题的，在《安卓逆向这档事》四、恭喜你获得广告&弹窗静默卡中提到过，一般在启动 `Activity` 的时候会预先加载一些数据，这样修改有很大概率会因为某些数据没有加载上从而导致应用出现闪退或者其他奇奇怪怪的问题。

然后我想使用这个方法——Acitivity切换定位，修改 `Intent` 的 `Activity` 类名，可是我看到 `MainActivity` 没有人调用它，暂时只能是另寻他法了。

![](upload/attach/202307/926486_N3NRHQ6STQ4DVG7.png)

现在我能想到的方法就是将我们自己的 `Activity` 代替 `MainActivity` 的位置，再为 `MainActivity` 重新进行声明，我想或许正己大佬讲的预加载数据未加载完全的问题是因为将之后启动的 `Activity` 作为主 `Activity` 进行默认启动，这样有些之后才会加载的数据在启动时因为未被加载，而之后启动的 `Activity` 却先加载了，这样就导致有些数据没有被加载从而出现奇奇怪怪的问题。而我只是将我自己写的 `Activity` 进行默认启动，这样貌似没有数据加载不全一说呢！只不过我能力有限，也不知道我的猜想是否正确，但现在别无他法，只能硬着头皮试试看了。

```xml
<?xml version="1.0" encoding="utf-8" standalone="no"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
android:compileSdkVersion="32"
android:compileSdkVersionCodename="12"
package="com.zj.wuaipojie"
platformBuildVersionCode="32"
platformBuildVersionName="12">
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
<application
android:allowBackup="true"
android:appComponentFactory="androidx.core.app.CoreComponentFactory"
android:dataExtractionRules="@xml/data_extraction_rules"
android:extractNativeLibs="false"
android:fullBackupContent="@xml/backup_rules"
android:icon="@mipmap/ic_launcher"
android:label="@string/app_name"
android:networkSecurityConfig="@xml/network_config"
android:supportsRtl="true"
android:theme="@style/Theme.Wuaipojie">
<activity android:exported="true" android:name="com.zj.wuaipojie.ui.ChallengeFifth" />
<activity android:exported="true" android:name="com.zj.wuaipojie.ui.ChallengeFourth" />
<activity android:exported="false" android:name="com.zj.wuaipojie.ui.ChallengeThird" />
<activity android:exported="false" android:name="com.zj.wuaipojie.ui.ChallengeSecond" />
<activity android:name="com.zj.wuaipojie.ui.AdActivity" />
<activity
android:exported="true"
android:label="@string/app_name"
android:name="com.zj.wuaipojie.ui.MainActivity" />
<activity android:exported="true" android:name="com.zj.wuaipojie.ui.MyActivity">
<intent-filter>
<action android:name="android.intent.action.MAIN" />
<category android:name="android.intent.category.LAUNCHER" />
</intent-filter>
</activity>
<activity android:name="com.zj.wuaipojie.ui.ChallengeFirst" />
</application>
</manifest>
```

这是经过我修改后的配置文件，为什么要这么改呢？我来讲一下主 `activity`：

```xml
<activity android:exported="true" android:name="com.zj.wuaipojie.ui.MyActivity">
<intent-filter>
<action android:name="android.intent.action.MAIN" />
<category android:name="android.intent.category.LAUNCHER" />
</intent-filter>
</activity>
```

- `android:exported="true"`：指定该 `Activity` 是否可以被其他应用程序调用。如果设置为 `true`，则表示该 `Activity` 可以被其他应用程序调用；如果设置为 `false`，则表示该 `Activity` 只能被当前应用程序调用。
- `android:name="com.zj.wuaipojie.ui.MyActivity"`：指定该 `Activity` 的完整类名。
- `action android:name="android.intent.action.MAIN"`：表示 `activity` 作为一个什么动作启动，`android.intent.action.MAIN` 表示作为主 `activity` 启动
- `category android:name="android.intent.category.LAUNCHER"`：表示这个 `activity` 为当前应用程序优先级最高的 `Activity`

现在对主 `Activity` 有了解了，但是你现在应该还有一个问题，那就是为什么将自己的 `Activity` 代替掉原先的主 `Activity`，而原先的主 `Activity` 不需要重新进行声明呢？

这是因为你将自己写的 `Activity` 设为主 `Activity` 启动且优先级最高，那么你只是将自己的 `Activity` 替换了 `MainActivity` 在 `AndroidManifest.xml` 文件中的位置。这样，当应用程序启动时，系统将启动 `MyActivity`，而不是 `MainActivity`，但是 `MainActivity` 仍然存在于 `AndroidManifest.xml` 文件中，可以在应用程序中使用。

### 第四步：用 apktool 的打包功能重新打包反编译的 apk 文件

在需要重新打包反编译的 `apk` 文件所处的文件夹中打开 `powershell` 并且输入 `cmd`；

输入命令：`apktool.bat b` 需要打包的文件夹 `-o` 重新打包反编译的 `apk` 文件存放的路径

我输入命令后出现了一个报错：

![](upload/attach/202307/926486_3NC2F22A33U7BYV.png)

我去查了一下，这个错误提示是由 `APKTOOL` 处理 `APK` 时出现的，提示资源目录名无效。具体原因可能是因为你的资源目录名包含了非法字符，比如空格或其他特殊字符，导致 `APKTOOL` 无法处理它们。

这个错误信息中显示了 `res` 文件夹下的一个资源目录名为 `“navigation”`，可能是导致错误的原因之一，我们去看看该文件夹：

![](upload/attach/202307/926486_K86EWNB9P62TM4T.png)

但是这个文件没法删除，因为 `R` 类中有子类对应着这个资源文件夹，我不管怎么改文件名字都出了问题，我只能另寻他法，我查到除了资源文件夹名称无效还有可能是由以下原因导致的：

1. 目录不存在：请检查命令中指定的资源目录路径是否正确，如果路径不正确请修改为正确的路径。如果路径正确但目录不存在，请创建该目录。
2. 目录为空：请检查命令中指定的资源目录中是否包含有效的资源文件。如果目录为空，请将需要的资源文件放入该目录。
3. 权限问题：请确保您具有足够的权限访问命令中指定的目录。如果您不确定是否具有足够的权限，请使用管理员权限运行命令。
4. 其他原因：
   - 重新安装或更新 `apktool`：如果您的 `apktool` 版本过旧或者出现了其他问题，可以尝试重新安装或更新 `apktool`。
   - 使用其他工具进行反编译和回编译：如果 `apktool` 无法解决问题，您可以尝试使用其他工具进行反编译和回编译，例如 `dex2jar` 和 `jd-gui` 等。
   - 使用 `MT` 管理器进行打包时是没有任何问题的，如果不是我的操作问题那就应该大概是 `apktool` 回编译执行的方法流程和需要回编译的应用程序的代码之间发生了某种冲突导致出现此问题，想要解决这种情况引发的问题需要通过修改代码来解决这个问题。

我对以上原因进行检查，排除掉了第一、第二、第三和 `apktool` 版本过旧这些原因，现在只剩下了两种情况，一是需要使用其他工具，二是 `apktool` 和需要回编译的应用程序的代码之间发生了某种冲突，这两种情况我更倾向于第二种。
