
思路中断，不妨查看一下启动时的`so`时候的`ELF`文件，该部分执行程序启动时候的函数指针（`JNI`）。

![image-20231024110656618](https://bbs.kanxue.com/upload/attach/202310/940967_6R4R7NGJRUPPFXW.png)

![image-20231024111830602](https://bbs.kanxue.com/upload/attach/202310/940967_8B5TZ6RMUPUEPXC.png)

其中的`.init_array`里面`sub_31B0`引起了注意，点击进入。

![image-20231024110705756](https://bbs.kanxue.com/upload/attach/202310/940967_GV232UCGFVQX3R2.png)

### 反编译

![image-20231024110715246](https://bbs.kanxue.com/upload/attach/202310/940967_X37JHEXFREE6YUR.png)

大概意思是`sub_30D0`创建了一个线程，线程调用了`sub_30D0`来检测里面是否有`frida`和`xpose`检测。

![image-20231024110731235](https://bbs.kanxue.com/upload/attach/202310/940967_N9QQ2GK39D7CN3K.png)

绕过这个函数可以通过`hook pthread_create`来阻止进程的创建来进行绕过，也可以通过`hook sub_30D0`进行绕过。

在这里选择后者。

这里先贴一下相关知识点。

> `linker64`是 Android 64 位系统中的动态链接器，它负责加载和链接共享库，以及解析符号引用，使得程序能够正确执行。在 Android 系统中，`linker64` 是一个重要的组件，它在应用程序启动时被调用，负责加载应用程序所需的共享库，并建立共享库之间的依赖关系。
>
> `linker64`的主要功能包括：
>
> - 加载共享库：根据程序指定的共享库路径，加载共享库到内存中。
> - 符号解析：解析程序中对共享库的符号引用，找到符号在内存中的地址，以便正确执行程序。
> - 重定位：对共享库中的重定位表进行处理，将符号引用重定位到正确的地址。
> - 初始化：执行共享库中的`init_array`段，调用初始化函数进行必要的初始化操作。
> - 启动程序：最后，`linker64`调用程序的入口函数（一般是`main`函数），启动应用程序的执行。
>
> 在 Android 64 位系统中，`init_array`一般会被`linker64`中的`call_array`函数调用。`call_array`函数负责遍历共享库中的`init_array`段，并依次执行其中的函数指针所指向的初始化函数。它会在动态链接器加载共享库时被调用，确保在程序启动时执行这些初始化函数。

由于反调试函数（`sub_30D0`）启动线程是在`init_array`中，而`so`层的`init_array`都是被`linker64`中的`call_array`调用，所以我们直接去**Hook**`linker64 call_array`。在这之前需要动态调试`so`获得`linker64 call_array`的地址。

### 3.2.1 动态调试`so`获取`linker64.so call_array`的函数地址

#### 3.2.1.1 使用`adb`命令打开手机上的`android_server64`

1. 搜索自己 IDA 下面的`android_server64`。

![image-20231024110741053](https://bbs.kanxue.com/upload/attach/202310/940967_KDVKQDZCMNZTQSB.png)

2. 把`android_server64`使用`adb push`命令发送到自己的手机文件夹`/data/local/tmp`下面。
3. 启动`android_server64`。

![image-20231024110751007](https://bbs.kanxue.com/upload/attach/202310/940967_XCHKZGD62GYYGX4.png)

#### 3.2.1.2 使用 WINDOWS 端的 IDA 连接手机端

1. 打开`ida`，不要`new`新工程，直接进入，选择`Remote ARM Linux/Android debugger`。

![image-20231024110838726](https://bbs.kanxue.com/upload/attach/202310/940967_MAUUYSCZFDSNZV4.png)

2. 操作如下。

![image-20231024110827803](https://bbs.kanxue.com/upload/attach/202310/940967_ZHCTGK5C27ET6SY.png)

点击 OK。

![image-20231024110845019](https://bbs.kanxue.com/upload/attach/202310/940967_6KKBMN4N83MG95K.png)

弹出一个列表框，`search`搜索`app`的包名。

![image-20231024110850905](https://bbs.kanxue.com/upload/attach/202310/940967_TGUZPSZT9QMT9V3.png)

#### 3.2.1.3 下面会有个坑

我们在右边的`Module`板块搜索我们在`IDA`之前找到的重要`call`或者`so`的关键字，会发现有时候根本搜索不到。

![image-20231024110859179](https://bbs.kanxue.com/upload/attach/202310/940967_75Z2V4DTHFMM7HB.png)

因为没有`F9`加载。

原因如下：

> IDA Pro 附加成功之后，会先调到其他`so`，比如`libc.so`这些。这个时候，需要先`F9`跳过，等没得跳之后，才需要执行`jdb`。
>
> 按`F9`加载就行。

搜索`Module`：`linker64`。

![image-20231024110904940](https://bbs.kanxue.com/upload/attach/202310/940967_NKTXZ2PGQ9RJZ7S.png)

在`Module`：`linker64`下搜索`call_array`。

![image-20231024110924940](https://bbs.kanxue.com/upload/attach/202310/940967_QFZF6TKNCAA6WXN.png)

记录一下`call_array`地址：`0000007D68B58764`减去`linker64`的基地址（`7D68B38000`）=`00020764`（偏移地址）。

![image-20231024110932115](https://bbs.kanxue.com/upload/attach/202310/940967_3RVRGVTUZNGMR4U.png)

开始编写`frida`的`Hook`脚本：

```javascript
// 获取linker64模块的基地址
var linker64_module = Module.getBaseAddress("linker64");

// 使用拦截器附加linker64模块的偏移地址
// 7D68B58764 - 7D68B38000 = 0x20764
Interceptor.attach(linker64_module.add(0x20764), {
  // 进入函数，检查参数args[3]指向的字符串是否匹配libfoo.so
  onEnter: function (args) {
    if (args[3].readCString().match("libfoo.so")) {
      // 获取libfoo.so的基地址
      var libfoo_module = Module.findBaseAddress("libfoo.so");
      console.log("获取libfoo.so的基地址==" + libfoo_module);

      Interceptor.replace(
        libfoo_module.add(0x30D0),
        new NativeCallback(
          function () {
            return;
          },
          "void",
          []
        )
      );
    }
  },
  onLeave: function (result) {}
});
```

运行之后发现仍然存在检测，继续往下分析`native`层的`init`函数：

1. 调用`sub_323C(JNIenv, JClass);`
2. 保存字符串`"pizzapizzapizzapizzapizz"`。

![image-20231024110942230](https://bbs.kanxue.com/upload/attach/202310/940967_6VNEDM8Q8DXG9FU.png)

我们查看`sub_323C`的调用次数，以便把握`Hook`时机（选中函数后按下`x`查看引用次数）。

![image-20231024110948552](https://bbs.kanxue.com/upload/attach/202310/940967_W9SWPFPKT9F2DWB.png)

只调用了一次。由于`sub_323C`只在`init`函数中，`init`函数在`apk`启动（`oncreate`）时才被调用，所以我们选择在`libfool.so`加载的时候进行`Hook`。`libfoo.so`是`apk`在`java`层调用`System.loadLibrary("foo")`进行加载，调用的底层逻辑是通过`libandroid_runtime.so`的`android_dlopen_ext`来加载的`so`。

> 具体的调用逻辑如下：
>
> 1. 应用程序通过 JNI 接口调用`libandroid_runtime.so`中的`android_dlopen_ext`函数。
> 2. `android_dlopen_ext`函数接收一个参数，即`libfoo.so`的路径。它会根据路径加载`libfoo.so`动态链接库。
> 3. 在加载`libfoo.so`之前，`android_dlopen_ext`函数会检查是否已经加载过该库。如果已经加载过，则直接返回之前加载的库的句柄。
> 4. 如果`libfoo.so`还没有被加载过，则`android_dlopen_ext`函数会通过系统调用打开`libfoo.so`文件，并获取到动态链接库的句柄。
> 5. `android_dlopen_ext`函数会将`libfoo.so`的句柄保存在一个全局的缓存中，以便在后续的调用中能够直接使用。
> 6. `android_dlopen_ext`函数返回`libfoo.so`的句柄给应用程序。
> 7. 应用程序可以通过句柄来调用`libfoo.so`中的函数，实现底层逻辑。
>
> 需要注意的是，`android_dlopen_ext`函数是`Android Runtime`中的一个函数，用于加载动态链接库。它是通过 JNI 接口提供给应用程序使用的。在调用`android_dlopen_ext`函数之前，应用程序需要先加载`libandroid_runtime.so`，并通过 JNI 接口导入`android_dlopen_ext`函数。这样才能在`Java`层调用`android_dlopen_ext`函数，并实现加载`libfoo.so`的功能。

根据这些信息，继续编写`Frida`脚本：

```javascript
Java.perform(function () {
  // 获取linker64模块的基地址
  var linker64_module = Module.getBaseAddress("linker64");

  // 使用拦截器附加linker64模块的偏移地址
  // 7D68B58764 - 7D68B38000 = 0x20764
  Interceptor.attach(linker64_module.add(0x20764), {
    // 进入函数，检查参数args[3]指向的字符串是否匹配libfoo.so
    onEnter: function (args) {
      if (args[3].readCString().match("libfoo.so")) {
        // 获取libfoo.so的基地址
        var libfoo_module = Module.findBaseAddress("libfoo.so");
        console.log("获取libfoo.so的基地址==" + libfoo_module);

        Interceptor.replace(
          libfoo_module.add(0x30D0),
          new NativeCallback(
            function () {
              return;
            },
            "void",
            []
          )
        );
      }
    },
    onLeave: function (result) {}
  });

  // hook android_dlopen_ext
  var libfoo_loaded_flag = 0;

  // 定义一个变量用于标记是否已加载libfoo.so
  // 获取libandroid_runtime.so中的android_dlopen_ext函数的地址
  var android_dlopen_ext_addr = Module.getExportByName(
    "libandroid_runtime.so",
    "android_dlopen_ext"
  );

  // 拦截android_dlopen_ext函数
  Interceptor.attach(android_dlopen_ext_addr, {
    // 进入android_dlopen_ext函数时执行的操作
    onEnter: function (args) {
      // 判断传入的参数中是否包含"libfoo.so"的字符串
      if (-1 != args[0].readCString().indexOf("libfoo.so")) {
        // 如果包含，则将libfoo_loaded_flag标记为1，表示已加载libfoo.so
        libfoo_loaded_flag = 1;
      }
    },

    // 离开android_dlopen_ext函数时执行的操作
    onLeave: function (result) {
      // 如果libfoo_loaded_flag为1，表示已加载libfoo.so
      if (libfoo_loaded_flag == 1) {
        // 通过Interceptor.replace方法替换libfoo.so中偏移为0x323C的函数
        var libfoo_module = Module.findBaseAddress("libfoo.so");

        Interceptor.replace(
          libfoo_module.add(0x323C),
          new NativeCallback(
            function () {
              console.log("获取反调试地址sub_323c==" + libfoo_module);
              return;
            },
            "void",
            []
          )
        );

        libfoo_loaded_flag = 0;
      }
    }
  });

  // var RootDetection = Java.use("sg.vantagepoint.util.RootDetection");
  // RootDetection["checkRoot1"].implementation = function () {
  //   console.log("checkRoot1 is called");
  //   var ret = this.checkRoot1();
  //   console.log("checkRoot1 ret value is " + ret);
  //   return false;
  // };
});

// 立即开始执行函数
setImmediate(main);
```

#### 3.2.1.4 回到 JAVA 层

开始过`java`层的`root`检测。

![image-20231024111001233](https://bbs.kanxue.com/upload/attach/202310/940967_FDA5P28VPGPUCXM.png)

直接选择`checkRoot1`，右键获取`Frida`脚本。

![image-20231024111012120](https://bbs.kanxue.com/upload/attach/202310/940967_VXU6ZGFNXS3733M.png)

添加到之前的脚本上，返回`false`。

