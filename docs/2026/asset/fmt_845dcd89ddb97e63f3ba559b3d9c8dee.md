
# [分享]hook框架-xposed安装使用

## 0x1 xposed简介

### 1.介绍

由于frida和objection每次都需要我们注入到app来对程序进行修改，有了xposed hook框架，我们可以编写xposed模块,实现对app的永久修改。

## 0x2 xposed原理

### 1.原理

所有的app都是通过zygote fork出来的，xposed通过替换zygote的进程文件`app_process`, 就实现了对所有app进程的控制。

## 0x3 xposed的使用

### 1.安装`XposedInstaller_3.1.5.apk`

1）打开模拟器，安装`XposedInstaller_3.1.5.apk`，运行，提示xposed框架未安装。

![upload/attach/202202/859914_GM25PJESDQSX7CC.jpg](upload/attach/202202/859914_GM25PJESDQSX7CC.jpg)

2）去<a href="elink@895K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8q4)9J5k6s2S2V1j5g2)9J5k6i4S2H3L8%4y4W2k6q4)9J5k6h3W2F1k6X3!0Q4x3V1k6X3M7X3q4E0k6i4N6G2M7X3E0Q4x3V1j5`."><mark class="encrypted">014K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8q4)9J5k6s2S2V1j5g2)9J5k6i4S2H3L8%4y4W2k6q4)9J5k6h3W2F1k6X3!0Q4x3V1k6X3M7X3q4E0k6i4N6G2M7X3E0Q4x3V1j5`.</mark></a>下载SDK，我们看到上一步的图片中API是25，所以这里下载x86版本的sdk25。

![upload/attach/202202/859914_VD5EEV4XX6474WM.jpg](upload/attach/202202/859914_VD5EEV4XX6474WM.jpg)

3）`adb push` `/sdcard/Android/data/de.robv.android.xposed.installer/cache/downloads/framework`，但是后面两个目录默认没有，需要我们手动创建一下。

```
mount -o rw,remount /
cd /sdcard/Android/data/de.robv.android.xposed.installer/cache
mkdir downloads
cd downloads
mkdir framework
C:\Users\LENOVO\Desktop>adb push xposed-v89-sdk25-x86.zip /sdcard/Android/data/de.robv.android.xposed.installer/cache/downloads/framework
xposed-v89-sdk25-x86.zip: 1 file pushe...d. 27.3 MB/s (5206379 bytes in 0.182s)
```

4）如图所示，重启xposed，然后进行勾选，出现了安装/更新，右键instanll安装，然后会出现读取root权限的提示，这里选择允许，永久记住选择。

![upload/attach/202202/859914_76UR5H3EMKNBJKK.jpg](upload/attach/202202/859914_76UR5H3EMKNBJKK.jpg)

5）重启模拟器，显示已激活。

![upload/attach/202202/859914_894MPQAB8G2NEVA.jpg](upload/attach/202202/859914_894MPQAB8G2NEVA.jpg)

### 2.找到需要hook的函数

### 3.编写hook模块

## 0x4 实战

### 1.安装`看雪KCTF-xposed.apk`文件，打开，有签名校验。

![upload/attach/202202/859914_9NQQ3VH2KTER88Y.jpg](upload/attach/202202/859914_9NQQ3VH2KTER88Y.jpg)

### 2.还是通过Androidkiller反编译，找到函数入口点`com.kanxue.xposed.MainActivity`，然后找到签名类，找到函数`method public setRealCer(Ljava/lang/String;)V`。

### 3.先去下载Android studio，之前下载过sdk，这里就不用再下载了，找到`bin`目录下`idea.properties`文件，末尾加一句`disable.android.first.run=true`，表示首次启动不检测sdk，进入软件后，依次指定jdk，sdk安装目录，下载必须的组件后就可以正常使用了。

<a href="elink@a08K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2S2L8X3c8J5L8$3W2V1k6r3g2$3N6r3!0G2L8s2y4Q4x3X3g2U0L8W2)9J5c8R3`.`."><mark class="encrypted">bd1K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2S2L8X3c8J5L8$3W2V1k6r3g2$3N6r3!0G2L8s2y4Q4x3X3g2U0L8W2)9J5c8R3`.`.</mark></a>

### 4.新建一个空白Android项目（java语言），切换到protect file，然后为模块编写做准备：

#### 第一步：修改`build.gradle`文件添加依赖（打开`app/build.gradle`文件，找到`dependencies`在里面添加，添加完后右上边点一下`sync now`同步一下，会自动下载一些依赖包)

```
compileOnly 'de.robv.android.xposed:api:82'
compileOnly 'de.robv.android.xposed:api:82:sources'
```

![upload/attach/202202/859914_NZBKB6H7UZ433NQ.jpg](upload/attach/202202/859914_NZBKB6H7UZ433NQ.jpg)

#### 第二步：修改`app/src/main/andoirmanifest.xml`文件 在`aplication`中添加以下属性

```
<meta-data
android:name="xposedmodule" //xposed模块
android:value="true" /> //值为true，当我们安装xposed框架的时候就会认为这是一个xposed模块。
<meta-data
android:name="xposeddescription" //关于这个xposed模块的描述
android:value="description" /> //这个值我们可以自己改
<meta-data
android:name="xposedminversion" //要求的最低版本，这里默认就好了
android:value="82" />
```

![upload/attach/202202/859914_6SV4T6SARDGWXH7.jpg](upload/attach/202202/859914_6SV4T6SARDGWXH7.jpg)

#### 第三步：新建hook类

![upload/attach/202202/859914_HYE6J9W5279C22N.jpg](upload/attach/202202/859914_HYE6J9W5279C22N.jpg)

```
import de.robv.android.xposed.IXposedHookLoadPackage;
import de.robv.android.xposed.XC_MethodHook;
import de.robv.android.xposed.XposedBridge;
import de.robv.android.xposed.XposedHelpers;
import de.robv.android.xposed.callbacks.XC_LoadPackage;
//MyHook类名可以修改，但一定要和新建的类名一致
public class MyHook implements IXposedHookLoadPackage {
public void handleLoadPackage(XC_LoadPackage.LoadPackageParam lpparam) throws Throwable {
if(lpparam.packageName.equals("包名")) {
XposedHelpers.findAndHookMethod("类名", lpparam.classLoader,"函数名", new XC_MethodHook() {
@Override
protected void afterHookedMethod(MethodHookParam param)
throws Throwable {
Boolean result = true;//默认返回true
param.setResult(result);
}
});
};
}
}
```

将上面代码中的包名、类名、函数名替换成我们要hook的包名`com.kanxue.xposed`、类名`com.kanxue.xposed.SignCheck`、函数名`check`，默认返回值`result`是`true`，可以根据实际情况修改。

#### 第四步：修改入口类

在`main`下新建`assets`目录，在`assetss`目录下新建`xposed_init`文件，添加入口类`com.example.hooksig.MyHook` 根据实际情况添加入口类

![upload/attach/202202/859914_WXTTJ9WUASSC5E4.jpg](upload/attach/202202/859914_WXTTJ9WUASSC5E4.jpg)

### 5.准备工作完成，就可以点击build，build apk，生成apk文件，报错。

```
> Could not resolve all files for configuration ':app:debugCompileClasspath'.
> Could not find de.robv.android.xposed:api:82.
```

### 6.经过查询，依赖的第三方仓库已经改成了这个域名`api.xposed.info`，修改全局的`settings.gradle`文件，然后重新点build apk，生成apk文件。

```
allprojects {
repositories {
google()
mavenCentral()
// 添加此库
maven { url "https://api.xposed.info/" }
}
}
```

### 7.安装生成的apk模块，提示已安装，未激活，打开xposed框架，左上角点开，点模块，勾选激活，然后重启模拟器。

![upload/attach/202202/859914_BTRVB7JJ9WEFKYU.jpg](upload/attach/202202/859914_BTRVB7JJ9WEFKYU.jpg)

### 8.打开目标apk，发现签名校验已经绕过去了，还有root检测。

![upload/attach/202202/859914_74AXGWRYVVUAY9H.jpg](upload/attach/202202/859914_74AXGWRYVVUAY9H.jpg)

### 9.使用前面的方法，找到root检测的类和函数，继续修改hook代码：

![upload/attach/202202/859914_8S54DA48G5HM3UG.jpg](upload/attach/202202/859914_8S54DA48G5HM3UG.jpg)

### 10.使用前面的方法，找到flag判断的类和函数，继续修改hook代码：

![upload/attach/202202/859914_G89R4TXUNMCBW9D.jpg](upload/attach/202202/859914_G89R4TXUNMCBW9D.jpg)

## 0x5 总结

### 1.按照步骤来，生成apk一直报错，刚开始看到文章说要用本地的jar包，可是网上`xposed:api:82.jar`包都是收费的，最后找了一个54版本的jar，虽然成功编译了，但是达不到hook效果，一直没hook成功。

### 2.最后又看到说修改全局`build.gradle`文件，添加第三方依赖库，还是报错，说是要修改`settings.gradle`文件，最后终于成功了。
