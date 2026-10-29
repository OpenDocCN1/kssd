
### 3. 先去下载Android studio，之前下载过sdk，这里就不用再下载了，找到`bin`目录下`idea.properties`文件，末尾加一句`disable.android.first.run=true`，表示首次启动不检测sdk，进入软件后，依次指定jdk，sdk安装目录，下载必须的组件后就可以正常使用了。

[fc2K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2S2L8X3c8J5L8$3W2V1k6r3g2$3N6r3!0G2L8s2y4Q4x3X3g2U0L8W2)9J5c8R3`](https://bbs.kanxue.com/elink@46eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2S2L8X3c8J5L8$3W2V1k6r3g2$3N6r3!0G2L8s2y4Q4x3X3g2U0L8W2)9J5c8R3`.`.)

### 4. 新建一个空白Android项目（java语言），切换到`protect file`，然后为模块编写做准备：

#### 1）第一步：修改`build.gradle`文件添加依赖（打开`app/build.gradle`文件，找到`dependencies`在里面添加，添加完后右上边点一下`sync now`同步一下，会自动下载一些依赖包）

|  |  |
| --- | --- |
| 1  2 | `compileOnly` `'de.robv.android.xposed:api:82'`  `compileOnly` `'de.robv.android.xposed:api:82:sources'` |

![](https://bbs.kanxue.com/upload/attach/202202/859914_NZBKB6H7UZ433NQ.jpg)

#### 2）第二步：修改`app/src/main/andoirmanifest.xml`文件 在`aplication`中添加以下属性

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9 | `<meta-data`  `android:name="xposedmodule"`  `//xposed模块`  `android:value="true"` `/>` `//值为true，当我们安装xposed框架的时候就会认为这是一个xposed模块。`  `<meta-data`  `android:name="xposeddescription"`  `//关于这个xposed模块的描述`  `android:value="description"` `/>` `//这个值我们可以自己改`  `<meta-data`  `android:name="xposedminversion"`  `//要求的最低版本，这里默认就好了`  `android:value="82"` `/>` |

![](https://bbs.kanxue.com/upload/attach/202202/859914_6SV4T6SARDGWXH7.jpg)

#### 3）第三步：新建hook类

![](https://bbs.kanxue.com/upload/attach/202202/859914_HYE6J9W5279C22N.jpg)

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24 | `import` `de.robv.android.xposed.IXposedHookLoadPackage;`  `import` `de.robv.android.xposed.XC_MethodHook;`  `import` `de.robv.android.xposed.XposedBridge;`  `import` `de.robv.android.xposed.XposedHelpers;`  `import` `de.robv.android.xposed.callbacks.XC_LoadPackage;`    `//MyHook类名可以修改，但一定要和新建的类名一致`  `public` `class` `MyHook implements IXposedHookLoadPackage {`    `public void handleLoadPackage(XC_LoadPackage.LoadPackageParam lpparam) throws Throwable {`  `if`(lpparam.packageName.equals(`"包名"`)) {`  `XposedHelpers.findAndHookMethod(`"类名"`, lpparam.classLoader,`"函数名"`, new XC_MethodHook() {`  `@Override`  `protected void afterHookedMethod(MethodHookParam param)`  `throws Throwable {`      `Boolean result` `=` `true;` `//默认返回true`  `param.setResult(result);`  `}`  `});`  `};`  `}`  `}` |

将上面代码中的包名、类名、函数名替换成我们要hook的包名`com.kanxue.xposed`、类名`com.kanxue.xposed.SignCheck`、函数名`check`，默认返回值`result`是`true`，可以根据实际情况修改。

#### 4）第四步：修改入口类

在`main`下新建`assets`目录，在`assetss`目录下新建`xposed_init`文件，添加入口类 `com.example.hooksig.MyHook` 根据实际情况添加入口类

![](https://bbs.kanxue.com/upload/attach/202202/859914_WXTTJ9WUASSC5E4.jpg)

### 5. 准备工作完成，就可以点击`build`，`build apk`，生成`apk`文件，报错。

|  |  |
| --- | --- |
| 1  2 | `> Could` `not` `resolve` `all` `files` `for` `configuration` `':app:debugCompileClasspath'.`  `> Could` `not` `find de.robv.android.xposed:api:82.` |

### 6. 经过查询，依赖的第三方仓库已经改成了这个域名 `api.xposed.info`，修改全局的`settings.gradle`文件，然后重新点`build apk`，生成`apk`文件。

```
allprojects {
  repositories {
    google()
    mavenCentral()
    // 添加此库
    maven { url "e57K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2M7r3W2Q4x3X3g2^5M7r3!0K6k6h3c8Q4x3X3g2A6L8X3k6G2i4K6u0r3" }
  }
}
```

### 7. 安装生成的`apk`模块，提示已安装，未激活，打开`xposed`框架，左上角点开，点模块，勾选激活，然后重启模拟器。

![](https://bbs.kanxue.com/upload/attach/202202/859914_BTRVB7JJ9WEFKYU.jpg)

### 8. 打开目标`apk`，发现签名校验已经绕过去了，还有`root`检测。

![](https://bbs.kanxue.com/upload/attach/202202/859914_74AXGWRYVVUAY9H.jpg)

### 9. 使用前面的方法，找到`root`检测的类和函数，继续修改`hook`代码：

![](https://bbs.kanxue.com/upload/attach/202202/859914_8S54DA48G5HM3UG.jpg)

### 10. 使用前面的方法，找到`flag`判断的类和函数，继续修改`hook`代码：

![](https://bbs.kanxue.com/upload/attach/202202/859914_G89R4TXUNMCBW9D.jpg)

## 0x5 总结

### 1. 按照步骤来，生成`apk`一直报错，刚开始看到文章说要用本地的`jar`包，可是网上`xposed:api:82.jar`包都是收费的，最后找了一个54版本的`jar`，虽然成功编译了，但是达不到`hook`效果，一直没`hook`成功。

### 2. 最后又看到说修改全局`build.gradle`文件，添加第三方依赖库，还是报错，说是要修改`settings.gradle`文件，最后终于成功了。

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
