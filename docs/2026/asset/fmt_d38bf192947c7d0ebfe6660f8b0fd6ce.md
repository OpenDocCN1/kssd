
编写hook模块的第一步就是判断当前的进程名字，如果是要hook的进程就hook，不是则返回，所以修改模块后，要将模块重新load zygote里面必须重启zygote，要想zygote重启就要重启手机了。

解决办法：

所以修改的逻辑是不把模块load到zygote里面，而是load到自己想要hook的进程里面，这样修改模块后只需重启该进程即可

步骤：

（1）将上面`XposedInit.loadModules()`注释掉即可
（2）在`2`处修改代码

```
if (isZygote) {
    XposedHelpers.findAndHookMethod("com.android.internal.os.ZygoteConnection", BOOTCLASSLOADER,
        "handleChildProc",
        "com.android.internal.os.ZygoteConnection.Arguments", FileDescriptor[].class, FileDescriptor.class,
        PrintStream.class, new XC_MethodHook() {
            @Override
            protected void afterHookedMethod(MethodHookParam param) throws Throwable {
                // TODO Auto-generated method stub
                super.afterHookedMethod(param);
                String processName = (String) XposedHelpers.getObjectField(param.args[0], "niceName");
                String coperationAppName = "指定进程名称如：com.android.settings";
                if(processName != null){
                    if(processName.startsWith(coperationAppName)){
                        log("--------Begin Load Module-------");
                        XposedInit.loadModules();
                    }
                }
            }
        });
    ZygoteInit.main(args);
} else {
    RuntimeInit.main(args);
}
```

我们只需要将我们的模块进程指定，这样就不用每次开机都重启那

#### （2）取消操作Installer APP

通过阅读源码，我们发现读Install App的源码发现其实勾选hook模块其实app就是把模块的apk位置写到一个文件里，等load模块时会读取这个文件，从这个文件中的apk路径下把apk load到进程中

`loadmodules`的源码：

![image-20211003152312993](https://bbs.kanxue.com/upload/attach/202110/905443_MRFGVRB95DGVNXN.png)

apk配置文件就是installer app文件路径下的`conf/modules.list`这个文件`data/data/de.robv.android.xposed.installer/conf/modules.list`或者`data/user_de/0/de.robv.android.xposed.installer/conf/modules.list`

所以我们勾选一个文件，实际是将其写到`conf/modules.list`文件下，此时我们发现Xposed中还有一个方法`loadModule`

![image-20211003152549464](https://bbs.kanxue.com/upload/attach/202110/905443_VZW9RBC368RNDVZ.png)

这个方法可以根据具体的路径和类加载器，直接导入模块，所以只要我们在上面代码中修改一些，就可以直接导入，不需要勾选那，我们确定apk路径：`pathclass = "/data/local/tmp/module.apk"`和类加载器为根类加载器

```
if (isZygote) {
    XposedHelpers.findAndHookMethod("com.android.internal.os.ZygoteConnection", BOOTCLASSLOADER,
        "handleChildProc",
        "com.android.internal.os.ZygoteConnection.Arguments", FileDescriptor[].class, FileDescriptor.class,
        PrintStream.class, new XC_MethodHook() {
            @Override
            protected void afterHookedMethod(MethodHookParam param) throws Throwable {
                // TODO Auto-generated method stub
                super.afterHookedMethod(param);
                String processName = (String) XposedHelpers.getObjectField(param.args[0], "niceName");
                String coperationAppName = "指定进程名称如：com.android.settings";
                if(processName != null){
                    if(processName.startsWith(coperationAppName)){
                        log("--------Begin Load Module-------");
                        String pathclass = "/data/local/tmp/module.apk";
                        //注意这里是loadModule方法,类加载器是我们的根类加载器
                        XposedInit.loadModule(pathclass,BOOTCLASSLOADER);
                    }
                }
            }
        });
    ZygoteInit.main(args);
} else {
    RuntimeInit.main(args);
}
```

## 四、Xposed框架特征

本节参考世界美景大佬[定制Xposed框架](https://bbs.pediy.com/thread-255836.htm)和肉丝大佬[来自高纬的对抗：魔改XPOSED过框架检测(下)](https://bbs.kanxue.com/elink@71bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0M7q4)9J5k6i4N6W2K9i4S2A6L8W2)9J5k6i4q4I4i4K6u0W2j5$3!0E0i4K6u0r3M7#2)9J5c8W2W2m8e0f1y4J5f1g2y4A6x3p5I4r3d9V1N6z5d9i4N6n7z5i4q4t1c8p5p5`.)，经过我们上文的Android运行机制和Xposed框架运行机制讲解，相信大家对Xposed的框架已经有了进一步的认识，这样我们再来看这篇帖子里的修改的特征，就显得十分清晰了

![image-20211003152549464](https://bbs.kanxue.com/upload/attach/202110/905443_NYZ9Y2CYQZ4FK6U.png)

## 五、Xposed特征修改

### 1. `XposedInstaller`

我们下载`XposedInstaller`的工程代码加载到AndroidStudio中，让`XposedInstaller`配置环境，错误提示解决见上文[源码编译（2）——Xopsed源码编译详解](https://bbs.pediy.com/thread-269616.htm)

修改点：

（1）修改整体包名
（2）修改`xposed.prop`

#### （1）修改整体包名

先来改下整体的包名，首先将目录折叠给取消掉

![image-20211003152549464](https://bbs.kanxue.com/upload/attach/202110/905443_R2CYDHPKU3RZPAB.png)

然后我们在包名路径中，将`xposed`改成`xppsed`，这样可以保证包名长度是一样，同时`xposed`特征消失不见，选择Refactor→Rename

![image-20211003152549464](https://bbs.kanxue.com/upload/attach/202110/905443_N5TSSY3G76CATWQ.png)

![image-20211003152549464](https://bbs.kanxue.com/upload/attach/202110/905443_MARHR259GEVABDE.png)

我们直接点击Refactor，就可以替换成功，点击Preview，则需要再点击 Do Refactor

![image-20211003152549464](https://bbs.kanxue.com/upload/attach/202110/905443_VX9ERNTJ4CU5HA5.png)

这时候我们可以发现程序下的包名都改变了

![image-20211003152549464](https://bbs.kanxue.com/upload/attach/202110/905443_4MN63CEF67UZNBY.png)

接下来就是在整个项目的根文件夹下，进行整体的包名替换，因为还有很多编译配置、或者路径配置等等，需要进行包名的更换

在app文件夹右击，选择Replace in Path

![image-20211003152549464](https://bbs.kanxue.com/upload/attach/202110/905443_P9SBZBQJ4VZ2MT5.png)

把所有的`de.robv.android.xposed.installe`都改成`de.robv.android.xpsed.installer`

![image-20211003152549464](https://bbs.kanxue.com/upload/attach/202110/905443_Q7AGM3YFYW7GRZB.png)

搜出来匹配的地方只有5个文件中合计的7处地方，并不多，直接replace All替换即可

#### （2）`xposed.prop`改成`xpsed.prop`

就是把如下图处的`xposed.prop`改成`xppsed.prop`即可

![image-20211003152549464](https://bbs.kanxue.com/upload/attach/202110/905443_7DZDD9RXPDFJRG2.png)

接下来就是编译了。编译时先Build→Clean一下，然后再Build→Make Project，这样就直接编译通过了。可以连接到手机，刷到手机上去，App会被装在手机上，但是无法自动启动，得手动点开

![image-20211003174013862](https://bbs.kanxue.com/upload/attach/202110/905443_PCXG52K9UV22P6S.png)

### 2. `XposedBridge`

#### （1）修改整体包名

首先是改包名，方法与上文一模一样，也是首先将`xposed`进行重构，改成`xppsed`

![image-20211003152549464](https://bbs.kanxue.com/upload/attach/202110/905443_PS7CNF4T4TEQTPP.png)

注：我们发现很多类型需要手动修改包名，我们依次将包名修改，这里我们发现重构后没反应，很可能是Android Studio的问题，换一个版本或重启

然后也是一样的在项目根目录下，执行Replace in Path，将所有的`de.robv.android.xposed.installer`都改成`de.robv.android.xppsed.installer`

这里就修改完成

#### （2）生成文件

`XposedBridge.jar`:

先Make Clean一下，然后编译，将编译出来的文件复制一份，命名为`XppsedBridge.jar`即可

![image-20211003152549464](https://bbs.kanxue.com/upload/attach/202110/905443_KYEBXRUS685QD9X.png)

`api.jar`:

然后我们在Gradle->others->generate API中生成`api.jar`，保存在`build/api`下

![image-20211003152549464](https://bbs.kanxue.com/upload/attach/202110/905443_8EV6BRM48Q2D6US.png)

### 3. Xposed

Xposed中的文件需要修改的地方不少：

![image-20211003152549464](https://bbs.kanxue.com/upload/attach/202110/905443_DFMD3CNUETK3VY6.png)

#### （1）`libxposed_common.h`

修改之前：

![image-20211003152549464](https://bbs.kanxue.com/upload/attach/202110/905443_JC7SWW7RBWK9HQU.png)

修改之后：

![image-20211003152549464](https://bbs.kanxue.com/upload/attach/202110/905443_CPY7ZVCB4XXUAZG.png)

#### （2）`Xposed.h`

修改之前：

```
#define XPOSED_PROP_FILE "/system/xposed.prop"
#define XPOSED_LIB_ART XPOSED_LIB_DIR "libxposed_art.so"
#define XPOSED_JAR "/system/framework/XposedBridge.jar"
#define XPOSED_CLASS_DOTS_ZYGOTE "de.robv.android.xposed.XposedBridge"
#define XPOSED_CLASS_DOTS_TOOLS "de.robv.android.xposed.XposedBridge$ToolEntryPoint"
```

修改之后：

```
#define XPOSED_PROP_FILE "/system/xppsed.prop"
#define XPOSED_LIB_ART XPOSED_LIB_DIR "libxppsed_art.so"
#define XPOSED_JAR "/system/framework/XppsedBridge.jar"
#define XPOSED_CLASS_DOTS_ZYGOTE "de.robv.android.xppsed.XposedBridge"
#define XPOSED_CLASS_DOTS_TOOLS "de.robv.android.xppsed.XposedBridge$ToolEntryPoint"
```

![image-20211003152549464](https://bbs.kanxue.com/upload/attach/202110/905443_B43DDS6BWF6XVNQ.png)

#### （3）`xposed_service.cpp`

修改之前：

```
IMPLEMENT_META_INTERFACE(XposedService, "de.robv.android.xposed.IXposedService");
```

修改之后：

```
IMPLEMENT_META_INTERFACE(XposedService, "de.robv.android.xppsed.IXposedService");
```

#### （4）`xposed_shared.h`

修改之前：

