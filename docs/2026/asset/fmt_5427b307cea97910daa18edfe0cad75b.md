
## 属性分类

上面提到了属性的几种分类，下面看一下分类的不同含义:

1. `ro`开头的只读属性，只能被设置一次
2. `ctl`开头的控制属性,它只能是如下的值:
   a) `ctl.sigstop_on`
   b) `ctl.sigstop_off`
   c) `ctl.start`
   d) `ctl.stop`
   e) `ctl.restart`
   f) `ctl.interface_start`
   g) `ctl.interface_stop`
   h) `ctl.interface_restart`
   `ctl`开头的属性用来向`init`进程中的`service`发命令用的，属性值为`service`的名称/接口名称，比如`"ctl.start zygote"`就会调用名称为`zygote`的`service`的`Start`函数。
3. `persist`开头的持久化属性，属性值一般都是非持久化的，重启以后又会重新的初始化。而持久化属性在重启以后仍然会保留原来的值，持久化属性保存在`/data/property/persistent_properties`文件中。
4. `selinux.restorecon_recursive`属性，这个属性是用来恢复指定`value`文件的`selinux`上下文的，它的特殊之处在于`init`进程会`fork`一个子进程来调用`selinux_android_restorecon`函数，目前只在`vold`中发现使用这个属性。
5. 其他属性。

## 属性同步

由于属性是进程间共享的资源，而不同的进程可以读写属性，因此属性需要一种同步机制。属性同步依赖于`futex`系统调用,`futex`系统调用的第一个参数需要利用`mmap`机制在各个进程间共享，在属性实现中有如下几个变量提供这个参数:

1. `/dev/__properties__/properties_serial`文件内存映射成的`prop_area`结构的成员变量`serial_`。
2. 每个属性所对应`prop_info`结构的成员变量`serial`。
`serial`类型都为`atomic_uint_least32_t`，原子变量。
`futex`系统调用不仅可以实现多线程的事件通知机制，也可以实现互斥锁机制，`bionic`库封装了这个系统调用，比如`__futex_wait`,`__futex_wake`。
对于第1种情况`/dev/__properties__/properties_serial`文件中的`serial_`，初始值为0，当有属性被添加/更新时，`serial_`会自增1，并且通过`__futex_wake`唤醒所有等待着的进程，这些等待的进程是通过调用`__system_property_wait`函数(第一个参数为`nullptr`就可以等待这个`serial_`)从而调用`__futex_wait`进行等待的。由于属性被添加/更新时`serial_`会变化，所以`__system_property_wait`函数可以实现等待任意属性添加/更新的功能。`serial_`就是"global serial"。
对于第2种情况`prop_info`结构的成员变量`serial`初始值为属性值长度<<24,当更新了某个属性值的时候先将`serial`值|=1通知其他进程正在修改此属性，修改完属性以后将`serial`的值设置为`(len << 24) | ((serial + 1) & 0xffffff)`，然后通知其他所有等待`serial`值发生变化的进程。

假设一个进程正在修改某个属性，而另外一个进程要读取这个属性会发生什么？
看一下`system_properties.cpp`文件中`SystemProperties::Get`函数对同步的处理:
1. 首先调用`prop_info* SystemProperties::Find(const char* name)`函数根据名称查询`prop_info`对象。
2. 接着调用`int SystemProperties::Read(const prop_info* pi, char* name, char* value)`函数来获取属性的值，输入参数为`value`。在这个函数中首先进入一个循环，调用`Serial`函数等待`serial`值&1不为1，如果为1表示有进程正在修改此属性，此时等待修改进程更新完属性以后就读取到更新后的属性。如果读取进程判断没有进程修改，其他进程才开始修改，读取进程还会在读取属性以后再判断当前属性值是否发生了变化，如果发生了变化说明读取到的是旧属性值，它就重新进入循环再尝试读取新的属性值。所以如果有多个进程尝试读取更新属性值，读取进程有可能读取到旧属性，也有可能读取到更新后的属性。由于`init`进程通过`socket`接收更新属性的请求是串行的，所以不会有多个进程更新属性的情况。

# 二. 属性改机检测

上面说完了属性的原理，下面谈一下我对属性改机的理解，属性改机最好是只针对某些`app`,只有这些`app`看到的属性是修改过的，不影响全局的属性，全局的属性如果修改了很可能系统工作就不正常了。且只针对某些`app`修改属性,修改属性的控制逻辑也比较好设计，可以更方便的实现一键改机，不需要重启机器。

先来看一下属性改机检测，主要分为两方面:
1. 检测属性实现机制有没有问题
2. 检测得到的属性值有没有可疑的比如`ro.debuggable=1`
这里说的主要是第一个方面:检测属性实现机制有没有问题，主要检测点有以下:
1. 检测属性实现所依赖的`selinux`是否正确开启
2. 多方面获取属性，检测获取到的属性值是否一致
3. 改属性会用到IO重定向，检测是否有IO重定向操作
其他的检测比如是否`root`，是否有`frida`,`xposed`等等和属性检测没有直接关系就不再提及了。

先来看第1条检测`selinux`,厂商开启了`selinux`以后`android`系统部分一般都会遵守谷歌制定的策略，先调用`getenforce`判断`selinux`开启状态，然后去访问公开的被`selinux`策略禁止的文件，如果可以访问就表示`selinux`实现有问题，比如`/dev/socket/property_service`这个`socket`文件`app`是没办法`open`的，有一些属性文件`selinux`策略也是禁止访问的，系统中还有很多的文件可以用来做这种检测。读取`/proc/self/mounts`判断`selinuxfs`有没有挂载到`/sys/fs/selinux`。

### 第2条检测的逻辑如下:

1. 先调用`getprop`获取所有的属性,以此为基准对比其他途径获取的属性，一旦有差异基本上就代表着属性的实现有问题，为可疑设备。
2. 对比通过`android.os.SystemProperties`的`get`方法获取的属性，这个方法需要用反射来调用。
3. 对比`android.os.Build`类获取的属性，这个类在`zygote` `preload class`的时候就已经被初始化了，获取属性的时机很早，里边的属性有可能是原始的属性。
4. 对比通过`jni`调用的`__system_property_get`函数。
5. 对比通过`jni`调用的`__system_property_find`和`__system_property_read_callback`函数，引入这个两个函数需要加上`#if __ANDROID_API__ >= 26`。
6. 对比通过`jni`调用的遍历属性的函数`__system_property_foreach`和`__system_property_read_callback`，引入这个两个函数需要加上`#if __ANDROID_API__ >= 26`。
7. 对比调用"`getprop+prop`名称"获取的属性。
8. 自己解析`/system/etc/selinux/plat_property_contexts`等文件看获取到的属性上下文是否匹配。
9. 自己写代码解析`/dev/__properties__`下的文件，不使用`libc`的接口去获取属性并对比。

最后一种方式可以把系统和属性相关的代码拷贝到`jni`中，自己生成接口去获取属性，完全不依赖于系统的运行时。看下图:

![](upload/attach/202208/607812_ACC8NPPNC869NEG.png)

### 第3条的检测逻辑如下:

有很多改属性的逻辑都会将`/dev/__properties__`下的文件进行IO重定向，你读取`/dev/__properties__`下的文件，偷梁换柱让你读取到的是其他目录下的文件。IO重定向有很多种实现方式，这里检测其中的一些。

1. 有些IO重定向会直接修改内核`do_sys_open`函数，调用`do_filp_open`函数的时候传递另外一个`filename`过去，虽然可以实现IO重定向，但会在`/proc/pid/fd`下面暴露自己，打开文件的`fd`会指向被重定向的文件，检测手段:

```c
static std::string get_selfpath(int fd) {
char path[PATH_MAX];
snprintf(path, PATH_MAX, "%s/%d", "/proc/self/fd/", fd);
char buff[PATH_MAX];
ssize_t len = ::readlink(path, buff, sizeof(buff) - 1);
if (len != -1) {
buff[len] = '\0';
return std::string(buff);
}
return "";
}
```

```c
static bool checkFileFDLinkName(std::string filename) {
int fd = open(filename.c_str(), O_RDONLY);
__android_log_print(ANDROID_LOG_INFO, LOG_TAG, "filename:%s,fd:%d", filename.c_str(), fd);
std::string link_path = get_selfpath(fd);
__android_log_print(ANDROID_LOG_INFO, LOG_TAG, "link path: %s", link_path.c_str());
return filename == link_path;
}
```

然后调用

```c
checkFileFDLinkName("/dev/__properties__/property_info")
&& checkFileFDLinkName("/proc/cpuinfo")
&& checkFileFDLinkName("/proc/meminfo")
&& checkFileFDLinkName("/dev/__properties__/u:object_r:default_prop:s0")
```

逻辑是:存在这种IO重定向时，当你打开`/proc/cpuinfo`的时候，给你重定向到`/xxx`文件，假设`open("/proc/cpuinfo")`得到的`fd`为5，此时调用`readlink("/proc/pid/fd/5")`如果得到的路径是`/xxx`而不是`/proc/cpuinfo`，表示有问题。

2. 检测通过`open`和`openat`得到的文件内容是否相同，有些重定向判断某个文件是否需要重定向是通过检查内核里边`do_sys_open`的`filename`参数来实现的而忽略了`dfd`参数，此时将路径分隔开调用`openat`，如果得到的内容和调用`open`得到的内容不同则表明有问题。

```c
static bool checkOpenAt(std::string filename, std::string dir, std::string pathname) {
int openfd = open(filename.c_str(), O_RDONLY);
std::string open_content;
ReadFdToString(openfd, &open_content);
int dirfd = open(dir.c_str(), O_RDONLY);
int openat_fd = openat(dirfd, pathname.c_str(), O_RDONLY);
std::string openat_content;
ReadFdToString(openat_fd, &openat_content);
return open_content == openat_content;
}
```

调用: `checkOpenAt("/proc/cpuinfo", "/proc", "cpuinfo")`

上面的检查可以很大程度上检测出属性是否被改机。

# 三.定制源码反检测

那么如何实现自己的一套改属性的方案呢?

需要实现的目标如下:
1. 可以绕过上面的所有检测
2. 可以只针对某些`app`生效
3. 不需要重启手机
4. 不要改动原来的属性
5. 效率不能太差

在这里说一下我的实现方案:
要想绕过上面的所有检测，那么这个属性必须做的和真实的属性一样:`selinux`上下文是匹配的，属性区域也是存在并且匹配的，所有接口结果一致，除了里边内容是假的所有行为都和系统属性没什么差别。

先准备一份属性文件以及`/system/etc/selinux/plat_property_contexts，/vendor/etc/selinux/vendor_property_contexts`文件，这三个文件可以从真实的手机比如华为，`oppo`手机中获取。后面这两个文件需要先用脚本处理一下将厂商定义的`selinux`上下文修改为改机手机上已存在的`selinux`上下文，不然就得修改`selinux`策略文件添加这些上下文。

需要构建一个新的属性目录，假设为`/dev/xxx/__properties__`, `init`进程既初始化`/dev/__properties__`区域也初始化`/dev/xxx/__properties__`区域

`<sys/system_properties.h>`和`<sys/_system_properties.h>`头文件需要准备两份拷贝，我这里命名为`system_properties_ext.h`和`_system_properties_ext.h`,里边所有的函数都添加`_ext`后缀。

有了接口以后还需要准备一份接口实现文件`system_property_api_ext.cpp`,实现类为`static SystemProperties system_properties_ext;`这个类的实现依赖路径从`/dev/__properties__`修改为`/dev/xxx/__properties__`。

有了接口和实现类以后，`init`进程就可以利用新的接口初始化新区域:

```c
#define EXT_ROOT_DIR /dev/xxx

void property_init_ext() {
mkdir(EXT_ROOT_DIR, S_IRWXU | S_IXGRP);
mkdir(EXT_ROOT_DIR "/__properties__", S_IRWXU | S_IXGRP | S_IXOTH);
CreateSerializedPropertyInfoExt();//读取的文件修改为上面准备好的plat_property_contexts和vendor_property_contexts文件
if (__system_property_area_init_ext()) {
LOG(FATAL) << "Failed to initialize property area";
}
if (!property_info_area_ext.LoadDefaultPathExt()) {
LOG(FATAL) << "Failed to load serialized property info file";
}
init_ext_properties();
}
```

`CreateSerializedPropertyInfoExt`函数，`__system_property_area_init_ext`函数都已经切换为`/dev/xxx/__properties__`区域。

`init_ext_properties`函数用来设置属性的值，实现为解析上面准备的属性文件，调用`InitPropertySetExt`函数设置属性:

```c
void init_ext_properties() {
auto file_contents = std::string();
const std::string filename = "/xxx/xxx/my_prop";
if (!ReadFileToString(filename, &file_contents)) {
PLOG(ERROR) << "Could not read properties from '" << filename << "'";
return;
}
for (const auto& line : Split(file_contents, "\n")) {
auto trimmed_line = Trim(line);
if (trimmed_line.empty() || StartsWith(trimmed_line, "#")) {
continue;
}
auto tokenizer = SpaceTokenizer(line);
auto key = tokenizer.GetNext();
auto value = tokenizer.GetRemaining();
if (key.empty() || value.empty()) {
continue;
}
InitPropertySetExt(key.substr(1, key.size() - 2), value.substr(1, value.size() - 2));
}
}
```
