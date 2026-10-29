
# 使用 Frida 绕过 Android 应用检测

以下内容展示了如何使用 `Frida` 脚本对 Android 应用进行动态分析，绕过 `native` 层和 `Java` 层的检测逻辑。

## Native 层分析与初步 Hook

首先，获取 `linker64` 模块的基地址，并通过 `Interceptor.attach` 附加其偏移地址，以在加载 `libfoo.so` 时进行拦截。

```javascript
// 获取linker64模块的基地址
var linker64_module = Module.getBaseAddress("linker64");
//使用拦截器附加linker64模块的偏移地址
// 7D68B58764 - 7D68B38000 = 0x20764
Interceptor.attach(linker64_module.add(0x20764),{
    // 进入函数，代码检查参数args[3]指向的字符串是否匹配libfoo.s
    onEnter:function(args) {
        if(args[3].readCString().match("libfoo.so")) {
            // 获取libfoo.so的基地址
            var libfoo_module = Module.findBaseAddress('libfoo.so');
            console.log("获取libfoo.so的基地址==>"+libfoo_module)
            Interceptor.replace(libfoo_module.add(0x30D0),new NativeCallback(function(){
                return;
            },'void',[]));
        }
    },onLeave:function(result){}
})
```

运行之后发现仍然存在检测，继续往下分析 `native` 层的 `init` 函数。

1. 调用`sub_323C(JNIenv, JClass);`
2. 保存字符串 "`pizzapizzapizzapizzapizz`"

![image-20231024110942230](upload/attach/202310/940967_6VNEDM8Q8DXG9FU.png)

为了把握 `Hook` 时机，我们查看 `sub_323C` 的调用次数（选中函数后按下 `x` 查看引用次数）。

![image-20231024110948552](upload/attach/202310/940967_W9SWPFPKT9F2DWB.png)

`sub_323C` 只调用了一次，且由于它只在 `init` 函数中，而 `init` 函数在 `apk` 启动（`onCreate`）时才被调用，所以我们选择在 `libfoo.so` 加载的时候进行 `Hook`。`libfoo.so` 是 `apk` 在 `Java` 层调用 `System.loadLibrary("foo")` 进行加载的，其底层逻辑是通过 `libandroid_runtime.so` 中的 `android_dlopen_ext` 来加载的 `so`。

> **具体的调用逻辑如下：**
>
> 1. 应用程序通过 `JNI` 接口调用 `libandroid_runtime.so` 中的 `android_dlopen_ext` 函数。
> 2. `android_dlopen_ext` 函数接收一个参数，即 `libfoo.so` 的路径。它会根据路径加载 `libfoo.so` 动态链接库。
> 3. 在加载 `libfoo.so` 之前，`android_dlopen_ext` 函数会检查是否已经加载过该库。如果已经加载过，则直接返回之前加载的库的句柄。
> 4. 如果 `libfoo.so` 还没有被加载过，则 `android_dlopen_ext` 函数会通过系统调用打开 `libfoo.so` 文件，并获取到动态链接库的句柄。
> 5. `android_dlopen_ext` 函数会将 `libfoo.so` 的句柄保存在一个全局的缓存中，以便在后续的调用中能够直接使用。
> 6. `android_dlopen_ext` 函数返回 `libfoo.so` 的句柄给应用程序。
> 7. 应用程序可以通过句柄来调用 `libfoo.so` 中的函数，实现底层逻辑。

需要注意的是，`android_dlopen_ext` 函数是 `Android Runtime` 中的一个函数，用于加载动态链接库。它是通过 `JNI` 接口提供给应用程序使用的。在调用 `android_dlopen_ext` 函数之前，应用程序需要先加载 `libandroid_runtime.so`，并通过 `JNI` 接口导入 `android_dlopen_ext` 函数。这样才能在 `Java` 层调用 `android_dlopen_ext` 函数，并实现加载 `libfoo.so` 的功能。

根据这些信息，继续编写 `Frida` 脚本。

```javascript
Java.perform(function(){
    // 获取linker64模块的基地址
    var linker64_module = Module.getBaseAddress("linker64");
    //使用拦截器附加linker64模块的偏移地址
    // 7D68B58764 - 7D68B38000 = 0x20764
    Interceptor.attach(linker64_module.add(0x20764),{
        // 进入函数，代码检查参数args[3]指向的字符串是否匹配libfoo.s
        onEnter:function(args) {
            if(args[3].readCString().match("libfoo.so")) {
                // 获取libfoo.so的基地址
                var libfoo_module = Module.findBaseAddress('libfoo.so');
                console.log("获取libfoo.so的基地址==>"+libfoo_module)
                Interceptor.replace(libfoo_module.add(0x30D0),new NativeCallback(function(){
                    return;
                },'void',[]));
            }
        },onLeave:function(result){}
    })
    // hook android_dlopen_ext
    var libfoo_loaded_flag = 0; //定义一个变量用于标记是否已加载libfoo.so
    // 获取libandroid_runtime.so中的android_dlopen_ext函数的地址
    var android_dlopen_ext_addr = Module.getExportByName("libandroid_runtime.so", "android_dlopen_ext");
    // 拦截android_dlopen_ext函数
    Interceptor.attach(android_dlopen_ext_addr, {
        // 进入android_dlopen_ext函数时执行的操作
        onEnter:function(args){
            // 判断传入的参数中是否包含"libfoo.so"的字符串
            if(-1 != args[0].readCString().indexOf("libfoo.so")){
                // 如果包含，则将libfoo_loaded_flag标记为1，表示已加载libfoo.so
                libfoo_loaded_flag = 1;
            }
        },
        // 离开android_dlopen_ext函数时执行的操作
        onLeave:function(result){
            // 如果libfoo_loaded_flag为1，表示已加载libfoo.so
            if(libfoo_loaded_flag == 1){
                // 通过Interceptor.replace方法替换libfoo.so中偏移为0x323C的函数
                var libfoo_module = Module.findBaseAddress("libfoo.so");
                Interceptor.replace(libfoo_module.add(0x323C), new NativeCallback(function(){
                    console.log("获取反调试地址sub_323c==>"+libfoo_module)
                    return;
                }, 'void', []));
                libfoo_loaded_flag = 0;
            }
        }
    })
    // var RootDetection = Java.use("sg.vantagepoint.util.RootDetection");
    // RootDetection["checkRoot1"].implementation = function () {
    //     console.log('checkRoot1 is called');
    //     var ret = this.checkRoot1();
    //     console.log('checkRoot1 ret value is ' + ret);
    //     return false;
    // };
})
// 立即开始执行函数
setImmediate(main);
```

## 回到 Java 层处理 Root 检测

现在，我们来处理 `Java` 层的 `root` 检测。

![image-20231024111001233](upload/attach/202310/940967_FDA5P28VPGPUCXM.png)

直接选择 `checkRoot1` 右键获取 `Frida` 脚本。

![image-20231024111012120](upload/attach/202310/940967_VXU6ZGFNXS3733M.png)

将其添加到之前的脚本上，并返回 `false`。

```javascript
Java.perform(function(){
    // 获取linker64模块的基地址
    var linker64_module = Module.getBaseAddress("linker64");
    //使用拦截器附加linker64模块的偏移地址
    // 7D68B58764 - 7D68B38000 = 0x20764
    Interceptor.attach(linker64_module.add(0x20764),{
        // 进入函数，代码检查参数args[3]指向的字符串是否匹配libfoo.s
        onEnter:function(args) {
            if(args[3].readCString().match("libfoo.so")) {
                // 获取libfoo.so的基地址
                var libfoo_module = Module.findBaseAddress('libfoo.so');
                console.log("获取libfoo.so的基地址==>"+libfoo_module)
                Interceptor.replace(libfoo_module.add(0x30D0),new NativeCallback(function(){
                    return;
                },'void',[]));
            }
        },onLeave:function(result){}
    })
    // hook android_dlopen_ext
    var libfoo_loaded_flag = 0; //定义一个变量用于标记是否已加载libfoo.so
    // 获取libandroid_runtime.so中的android_dlopen_ext函数的地址
    var android_dlopen_ext_addr = Module.getExportByName("libandroid_runtime.so", "android_dlopen_ext");
    // 拦截android_dlopen_ext函数
    Interceptor.attach(android_dlopen_ext_addr, {
        // 进入android_dlopen_ext函数时执行的操作
        onEnter:function(args){
            // 判断传入的参数中是否包含"libfoo.so"的字符串
            if(-1 != args[0].readCString().indexOf("libfoo.so")){
                // 如果包含，则将libfoo_loaded_flag标记为1，表示已加载libfoo.so
                libfoo_loaded_flag = 1;
            }
        },
        // 离开android_dlopen_ext函数时执行的操作
        onLeave:function(result){
            // 如果libfoo_loaded_flag为1，表示已加载libfoo.so
            if(libfoo_loaded_flag == 1){
                // 通过Interceptor.replace方法替换libfoo.so中偏移为0x323C的函数
                var libfoo_module = Module.findBaseAddress("libfoo.so");
                Interceptor.replace(libfoo_module.add(0x323C), new NativeCallback(function(){
                    console.log("获取反调试地址sub_323c==>"+libfoo_module)
                    return;
                }, 'void', []));
                libfoo_loaded_flag = 0;
            }
        }
    })
    var RootDetection = Java.use("sg.vantagepoint.util.RootDetection");
    RootDetection["checkRoot1"].implementation = function () {
        console.log('checkRoot1 is called');
        var ret = this.checkRoot1();
        console.log('checkRoot1 ret value is ' + ret);
        return false;
    };
})
// 立即开始执行函数
setImmediate(main);
```

## 再次来到 Native 层分析 bar 判断函数

![image-20231024111034939](upload/attach/202310/940967_TSYESFE99SSQ4G8.png)

简单概括一下，`bar` 函数通过生成 `key1` 和字符串 `pizzapizzapizzapizzapizz` 进行异或来获取新的字符串。我们的思路就是 `hook` 生成 `key1` 的函数（`v6 = sub_10E0(v9);`）得到 `key1`，再和 `"pizzapizzapizzapizzapizz"` 进行异或。

完整脚本如下：
