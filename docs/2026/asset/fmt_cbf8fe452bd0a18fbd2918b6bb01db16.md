
通常情况下可以使用 Android Studio 中的 `java2smali` 插件，但是我不想安装这么臃肿的软件。下面介绍一种比较轻便的生成 `smali` 方法，结合 VS Code 的自动补全功能非常好用。此方法非常小巧，所用工具不到 `50 MB`。

![](https://bbs.kanxue.com/upload/attach/201912/617776_ZC6ZXAWRAARDVAE.png)

原理很简单：`javac` 将 `.java` 编译为 `.class`，之后用 `dx.jar` 将 `.class` 整合为 `classes.dex` 文件，再用 `baksmali.jar` 反编译 `.dex` 文件，得到需要的 `smali`。这个步骤需要 `android.jar` 库函数，还需要将原来程序的 `classes.dex` 转为 `classes.jar`（为了 `extends` 继承），即 `javac -cp` 中的依赖项。

写成的批处理脚本为：

> `make_smali.bat`

```bat
@echo off

set jarlib=android_api21.jar;classes.jar

set dx=dx_r29.0.2.jar

set baksmali=baksmali-2.3.4.jar



for /f %%i in ('dir /s /b "%~f1"') do (

    ::echo %%i

    if %%~xi EQU .java (

       echo processing %%~fi 

       javac -cp  %jarlib% "%%~fi")

)

java -jar %dx% --dex --output="%~dp1classes.dex" "%~f1"

java -jar %baksmali% d "%~dp1classes.dex"

pause
```

## 3.2 VS Code

先放一张 VS Code 的效果图：

![Vscode MakeSmali demo](https://bbs.kanxue.com/upload/attach/202001/617776_SR9GYSCR6EDWDMC.jpg)

在 VS Code 中使用 `Ctrl+Shift+P`，选择 `Java: Create Java Project` 创建 Java 环境，配置 `.classpath` 的 `dependencies`（需要使用 `Java: Refresh` 才能更新），之后即可获得类似于 Android Studio 的自动补全功能。配置 `Build Task` 可以实现自动化。

### `.classpath`

```xml
<?xml version="1.0" encoding="UTF-8"?>

<classpath>

    <classpathentry kind="con" path="org.eclipse.jdt.launching.JRE_CONTAINER/org.eclipse.jdt.internal.debug.ui.launcher.StandardVMType/JavaSE-1.8"/>

    <classpathentry kind="src" path="src"/>

    <classpathentry kind="output" path="bin"/>

    <classpathentry kind="lib" path="MiNE3.v1.5_classes.jar"/>

    <classpathentry kind="lib" path="D:\\AppExtend\\ReverseToolAndroid\\scripts\\android_api21.jar"/>

</classpath>
```

### `tasks.json`

```json
{
    // See dd3K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4L8#2)9J5k6h3#2A6j5%4u0G2M7$3!0X3N6q4)9J5k6h3y4G2L8g2)9J5c8X3k6%4L8r3W2F1K9#2)9J5c8W2)9K6c8V1I4A6L8X3E0u0k6q4)9K6c8o6M7K6x3K6f1#2z5l9`.`.

    // for the documentation about the tasks.json format

    "version": "2.0.0",

    "tasks": [
        {
            "label": "MakeSmali",

            "type": "shell",

            "command": ".\\make_smali.cmd src",

            "group": {
                "kind": "build",

                "isDefault": true
            }
        }
    ]
}
```

## 4. 编写新的入口 `smali`

### 方法一：`extends` 原来程序的入口 `class`

> `start.java`

```java
package cn.natdon.onscripterv2yuri;

import android.os.Bundle;



public class start extends cn.natdon.onscripterv2.start {

    public void onCreate(Bundle savedInstanceState) {

        super.onCreate(savedInstanceState);

    }

}
```

> `start.smali`

```smali
.class public Lcn/natdon/onscripterv2yuri/start;

.super Lcn/natdon/onscripterv2/start;

.source "start.java"





# direct methods

.method public constructor <init>()V

    .registers 1



    .prologue

    .line 4

    invoke-direct {p0}, Lcn/natdon/onscripterv2/start;-><init>()V



    return-void

.end method





# virtual methods

.method public onCreate(Landroid/os/Bundle;)V

    .registers 2



    .prologue

    .line 6

    invoke-super {p0, p1}, Lcn/natdon/onscripterv2/start;->onCreate(Landroid/os/Bundle;)V



    .line 7

    return-void

.end method
```

### 方法二：使用 `startActivity` 调用原入口 `Activity`

> `start2.java`

```java
package cn.natdon.onscripterv2yuri;

import android.app.Activity;

import android.content.Intent;

import android.os.Bundle;

import cn.natdon.onscripterv2.start;



public class start2 extends Activity{

    public void onCreate(Bundle savedInstanceState){

        Intent intent = new Intent(this, start.class);

        startActivity(intent);

    }

}
```

> `start2.smali`

```smali
.class public Lcn/natdon/onscripterv2yuri/start2;

.super Landroid/app/Activity;

.source "start2.java"





# direct methods

.method public constructor <init>()V

    .registers 1



    .prologue

    .line 7

    invoke-direct {p0}, Landroid/app/Activity;-><init>()V



    return-void

.end method





# virtual methods

.method public onCreate(Landroid/os/Bundle;)V

    .registers 4



    .prologue

    .line 9

    new-instance v0, Landroid/content/Intent;



    const-class v1, Lcn/natdon/onscripterv2/start;



    invoke-direct {v0, p0, v1}, Landroid/content/Intent;-><init>(Landroid/content/Context;Ljava/lang/Class;)V



    .line 10

    invoke-virtual {p0, v0}, Lcn/natdon/onscripterv2yuri/start2;->startActivity(Landroid/content/Intent;)V



    .line 11

    return-void

.end method
```

## 5. 将修改完的 App 编译并签名

```bash
keytool -genkey -alias abc.keystore -keyalg RSA -validity 20000 -keystore abc.keystore

java -jar apktool_2.4.0.jar b "%~1" -o "%~1_rebuild.apk"

jarsigner -verbose -keystore abc.keystore -signedjar "%~1_rebuild_signed.apk" "%~1_rebuild.apk" -storepass "password" abc.keystore
```

编译好的 APK 即可安装，只要原 App 没有校验，基本上重编译的都能运行。

[[招生]科锐逆向工程师培训（2026年11月13日实地，远程教学同时开班，第57期）](https://bbs.kanxue.com/thread-51839.htm)

最后于 2020-1-10 22:35 被 `devseed` 编辑，原因：添加内容
