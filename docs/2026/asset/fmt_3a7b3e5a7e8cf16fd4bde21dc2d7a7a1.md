
![](https://bbs.kanxue.com/upload/attach/201312/444052_j1l3io3tpvqcpiq.png)
![](https://bbs.kanxue.com/upload/attach/201312/444052_yxf2vxwaas0cp5u.png)
![](https://bbs.kanxue.com/upload/attach/201312/444052_vkoa7nh1t5cx2yi.png)
![](https://bbs.kanxue.com/upload/attach/201312/444052_eakcfmhlscjy8zi.png)

第 7-9 行对属性键值对名称的长度做检查，`PROP_NAME_MAX`和`PROP_VALUE_MAX`定义在`/bionic/libc/include/sys/system_properties.h`中，其值分别为 32 和 92。第 10 行调用`__system_property_find`查询系统中是否已经存在该属性。`__system_property_find`定义在`/bionic/libc/bionc/system_properties.c`文件中，定义如下：

![](https://bbs.kanxue.com/upload/attach/201312/444052_9g2u9ia6fm9l50j.png)

第 3 行将`__system_property_area__`值赋给局部变量`pa`，此处`__system_property_area__`又是什么时候初始化的呢？原来在`property_service.c`源文件中引用了这个变量(158 行)，同时在`init_property_area`中为此变量赋值，可以参考前文关于`init_property_area`函数的分析。4-5 行获取`prop_info`的个数以及第一个`toc`的起始地址。9-13 行根据`toc`来查找`prop_info`。第 10 行使用`TOC_NAME_LEN(toc)`宏来根据`toc`计算存储的`prop_info`的`name`长度是否相符，`TOC_NAME_LEN(toc)`和`TOC_TO_INFO(area, toc)`宏定义于`/bionic/libc/include/sys/_system_properties.h`头文件中，定义如下：

![](https://bbs.kanxue.com/upload/attach/201312/444052_byhdm5f6yx79axj.png)

在前文对`prop_area`结构体分析时，已经说明了`toc`高 8 位表示`prop_info`的`name`的长度，低 24 位表示`prop_info`相对`prop_area`的偏移地址。所以`TOC_NAME_LEN`宏可以计算`prop_info`的`name`长度，而`TOC_TO_INFO`可以获取对应`prop_info`的地址。第 11 行获取`prop_info`的起始地址。第 12 行通过比较`name`值是否相同来判断是否找到需要的`prop_info`。若找到则返回`prop_info`的起始地址，否则返回 0。

继续回到`property_set`函数。对于开始设置属性，`__system_property_find`返回值当然是 0 了，所以进入`else`分支。18 行将`__system_property_area__`赋值给局部变量`pa`。19 行判断`prop_info`的个数是否达到上限，若达到则返回。20 行获取新的`prop_info`的起始地址，21-23 行为该`prop_info`中的成员设置相应的值，包括`serial`的值，我们可以发现`serial`的高 8 位被设置为`name`的长度。24-27 行则完成`prop_area`中成员相应值得更新，增加一个`toc`，更新`count`和`serial`值。这样就完成了一个新属性在内存空间的设置。

那么如果不是一个新的`prop_info`，而是需要更新一个`prop_info`，又会如何操作呢？分析下`property_set`函数的`if`分支不就清楚了。第 12 行判断需要更新的属性名是否以`"ro."`开头，若是，则返回，也就是说所有属性名称类似`ro.*`的属性，一旦设置就不能再更改。14 行调用`update_prop_info`更新相应的`prop_info`，当然就是更新属性的值了，`update_prop_info`的函数如下，主要完成属性值以及`prop_info`中`serial`成员更新。

![](https://bbs.kanxue.com/upload/attach/201312/444052_b4kwvn9yikwl8qa.png)

`property_set`函数在完成`prop_info`的更新后，会相应更新`prop_area`中`serial`值。

以上分析了 Android 属性系统初始化的过程，因为属性系统最终是通过服务的形式提供查询、更新的，在分析该服务之前，先做总结如图 4 所示流程图。

![](https://bbs.kanxue.com/upload/attach/201312/444052_bm5aos4xjd2jx30.png)

## Android 属性系统服务

Android 属性系统服务是如何启动的呢？在`init.c`的`main`函数中，调用`queue_builtin_action`函数除了将`property_init_action`添加到`action_queue`队列中外，还将`property_service_init_action`添加到了`action_queue`队列中(`init.c`的 735 行)。代码如下：

![](https://bbs.kanxue.com/upload/attach/201312/444052_ja92b1k5dzyipvg.png)

`queue_builtin_action`函数在前文已经分析过了，名为`property_service_init_action`的函数会在`execute_one_command`调用中执行。`property_service_init_action`函数代码如下：

![](https://bbs.kanxue.com/upload/attach/201312/444052_e8au3s715vp60p9.png)

从注释中可以看出在读取属性文件时会触发属性服务运行。该函数最终会调用`start_property_service`运行属性服务，主要是创建一个 socket 并监听。`start_property_service`定义在`/system/core/init/property_service.c`文件中，代码如下：

![](https://bbs.kanxue.com/upload/attach/201312/444052_mhjrd8an3jewd8b.png)

4-6 行调用`load_properties_from_file`分别从 3 个文件中加载属性。`PROP_PATH_SYSTEM_BUILD`、`PROP_PATH_SYSTEM_DEFAULT`和`PROP_PATH_LOCAL_OVERRIDE`都定义在`/bionic/libc/include/sys/_system_properties.h`头文件中，值分别为`/system/build.prop`、`/system/default.prop`和`/data/local.prop`。在加载完上述属性后，第 7 行调用`load_persistent_properties`加载持久化属性，这些属性存在于`/data/property`目录下，该目录下存放了以`persist`开头的属性文件，如系统语言、国家编码等。第 8 行创建一个 socket，`create_socket`函数定义于`/system/core/init/util.c`文件中，其中 0666 表示权限，两个 0 表示`uid`和`gid`的值都为 0，也就是 root 用户和组，`PROP_SERVICE_NAME`值为`"property_service"`，对应的 socket 文件为`"/dev/socket/property_service"`，可参考`util.c`源文件的第 96 行。12 行监听创建的 socket，此处就不再继续跟踪这个`listen`函数了，有兴趣的可以继续分析，`listen`函数定义于`/bionic/libc/unistd/socketcalls.c`文件中。13 行将`property_set_fd`赋值为创建的 socket 文件描述符。

综上分析可知，Android 属性系统服务实际上是由`init`进程维护的，通过创建一个 socket 来监听相应的属性获取和设置的请求，最终需要由`init`进程来处理，至于是如何通过 socket 来获取和设置属性，后面会继续分析。

在继续分析之前，再来看下另一个与属性设置相关的 action，这个 action 在`init.c`文件的 722 行处添加到`action_queue`中的，代码如下：

![](https://bbs.kanxue.com/upload/attach/201312/444052_641kz9rv2p95v5z.png)

`set_init_properties_action`主要是设置了一些以`ro`开头的属性，包括基带信息，启动模式等，代码如下：

![](https://bbs.kanxue.com/upload/attach/201312/444052_57tmrq54dp2rqf9.png)
![](https://bbs.kanxue.com/upload/attach/201312/444052_iufbkz2hgraveeu.png)

## 系统属性内存空间共享

通过前文分析可知，Android 系统属性是通过在内核空间中开辟的一个共享内存来存储属性信息的，那么在整个系统中，其他进程是如何读取这块内存并映射到当前进程空间中的呢？研究下`init.c`文件中的`service_start`函数，该函数部分代码如下：

![](https://bbs.kanxue.com/upload/attach/201312/444052_jzyt4oql7no4u16.png)
![](https://bbs.kanxue.com/upload/attach/201312/444052_kd142cr06yp9cdl.png)

第 4 行`fork`出子进程，第 5 行判断是否是子进程，若是，进入`if`分支。第 10 行通过`properties_inited`函数判断属性区域是否已经初始化，11 行通过`get_property_workspace`获取`prop_area`起始地址以及大小，代码如下：

![](https://bbs.kanxue.com/upload/attach/201312/444052_5jh5juyj6bzk6w0.png)

`pa_workspace`在`init_workspace`函数中初始化，`fd`为`/dev/__properties__`的文件描述符，`size`表示`prop_area`内存区域的大小，该值固定为 32768。

`service_start`第 13 行调用`add_environment`将获取的属性区域信息添加到环境变量中。但是此处的环境变量实际上是定义在`init.c`中的长度为 32 的静态指针数组`static const char *ENV[32]`。`add_environment`函数代码如下：

![](https://bbs.kanxue.com/upload/attach/201312/444052_mboy8og9jao7o9r.png)

从上述代码易知，该函数首先从`ENV`中找到一个未填充的入口，然后将内存中分配用来存储环境变量值的起始地址复制给该未填充的入口。

继续回到`service_start`函数，在完成其他环境变量的设置以及`gid`、`uid`等的设置后，会调用`execve`执行相应的 service 进程，如下代码所示：

![](https://bbs.kanxue.com/upload/attach/201312/444052_pxa7wiihhd84ssm.png)

一直没有找到`execve`的函数源码 (汗！)，从网上查了些资料，该函数最终会执行到`__libc_init_common`函数(源码位于`/bionic/libc/bionic/libc_init_common.c`)，部分代码如下：

![](https://bbs.kanxue.com/upload/attach/201312/444052_7r21lgkity0xa6h.png)

第 5 行调用`__system_properties_init`函数，该函数位于`/bionic/libc/bionic/system_properties.c`文件中，部分代码如下：

![](https://bbs.kanxue.com/upload/attach/201312/444052_45b4lshzb2n53io.png)
![](https://bbs.kanxue.com/upload/attach/201312/444052_osvk7xh2sve3paa.png)

第 8 行获取名为`ANDROID_PROPERTY_WORKSPACE`的环境变量值，10 行从该环境变量中提取出`prop_area`的起始地址，11-13 行获取`prop_area`内存区域大小，14 行完成内存映射并将起始地址返回给局部变量`pa`，注意此处调用`mmap`映射内存是只读的。16 行将`pa`又赋值给全局变量`__system_property_area__`。

以上分析了 Android 属性系统区域内存的共享，主要是在`init`进程中将`prop_area`信息存储到环境变量中，在进程中获取该环境变量并解析，调用`mmap`进行内存空间映射，由于此内存空间是只读的，所以所有对属性系统区域的更改最终需要通过`init`进程来完成。由于每个进程都会执行这个函数，`__system_property_area__`是一个`static`变量，所以每个进程都有一个队里的对系统属性的引用，可以通过这个静态变量来访问系统属性区域内存信息。

## Android 系统属性获取

在 Android 应用程序开发中，可以通过`android.os.Build`类来访问一些属性信息，如设备品牌，设备名称，CPU 信息等。我们以访问`Build.BRAND`来分析 Android 系统中是如何获取属性信息的。

`android.os.Build.java`源码位于`/frameworks/base/core/java/android/os/Build.java`，`BRAND`的定义如下：

![](https://bbs.kanxue.com/upload/attach/201312/444052_9qedi09xzpgqwfy.png)

从上述代码易知，`BRAND`值是通过调用`getString("ro.product.brand")`来获取的，`getString`代码如下：

![](https://bbs.kanxue.com/upload/attach/201312/444052_2vb6p69unb1hy61.png)

`getString`通过`SystemProperties`类(`/frameworks/base/core/java/android/os/SystemProperties.java`)的`get(String key, String def)`获取对应属性信息，该函数代码如下：

![](https://bbs.kanxue.com/upload/attach/201312/444052_py47aaol4sx306f.png)
![](https://bbs.kanxue.com/upload/attach/201312/444052_ssby73chskzb5zh.png)

第 2 行首先检查属性名称长度是否超过预定义的最大长度，`PROP_NAME_MAX`值为 31，这与`/bionic/libc/include/sys/system_properties.h`中`PROP_NAME_MAX=32`其实一致的，因为一个是通过`>`比较，一个是通过`>=`比较。第 5 行调用`native_get`函数，从函数名可知该函数是 Native Code，在`SystemProperties.java`第 32 行有相应的声明，如下：

![](https://bbs.kanxue.com/upload/attach/201312/444052_bgi1ofzx0ohu2rs.png)

一般来说调用 Native 函数，需要在`static`块中加载对应的库，即调用`System.loadLibrary`，但是在`SystemProperties`类中并没有显示的加载对应的库。原来在`Zygote`启动过程中会调用`startReg`来注册 JNI 函数，`native_get`这个 JNI 函数就是在此时注册的。关于注册过程可以简单分析如下。

在`app_main.cpp`(`/frameworks/base/cmds/app_process/app_main.cpp`)的`main`函数中启动`Zygote`，如下代码所示：

![](https://bbs.kanxue.com/upload/attach/201312/444052_9iimcyix6fk8h8o.png)
