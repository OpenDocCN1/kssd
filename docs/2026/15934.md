
# [原创]Frida持久化方案(Xcube)之方案一——基于Magisk和Riru

上一篇介绍了基于 Xposed 的 Frida 持久化方案，这次带来基于 Magisk 和 Riru 的持久化方案。方案原理其实简单的很，开发过程却是非常坑，比如把手机刷成砖，比如每次出错都得重启手机加载插件。还好这些坑已经克服，现在只要编写靠谱的 JS 脚本，自己过掉 App 的各种检测了！

## 原理

### Riru 插件

我们知道 Riru 给出了在 App 进程 fork 出来的时候的几个回调函数：

- `forkAndSpecializePre`
- `forkAndSpecializePost`
- `specializeAppProcessPre`
- `specializeAppProcessPost`
- `forkSystemServerPre`
- `forkSystemServerPost`

这里选择了在如下方法处调用 `frida-gumjs` 引擎库执行 JS 脚本：

```cpp
static void forkAndSpecializePost(JNIEnv *env, jclass clazz, jint res) {
    if (res == 0) {
        // in app process
        enable_hack = rirutest(env, *_appDataDir);
        if (enable_hack) {
            gumjsHook();
        }
    } else {
        // in zygote process, res is child pid
        // don't print log here, see https://github.com/RikkaApps/Riru/blob/77adfd6a4a6a81bfd20569c910bc4854f2f84f5e/riru-core/jni/main/jni_native_method.cpp#L55-L66
    }
}
```

### `frida-gumjs.a`

`frida-gumjs.a` 是 Frida 封装的 hook 库和 QuickJS 引擎，上一篇里也提到过，官方有下载，我们这里不细说。

### 配置

同样使用配置文件来管理哪些 package 需要 hook，配置文件固定放在：

```c
const char *filepath = "/data/local/tmp/myscript.js";
```

要 hook 的包名每行一个：

```
org.xtgo.xcube.base2
org.xtgo.xcube.base
```

咱这个插件使用的 Frida 脚本路径也是写死的：

```c
const char *confpath = "/data/local/tmp/pkg.conf";
```

## 使用

1. 安装 Magisk 和 Magisk Manager，安装 `riru-core` 核心组件。
2. Frida 官网下载 `frida-gumjs.a`，放到 `/module/src/main/cpp/` 对应的 ABI 下。
3. 执行 `build`，`out/` 目录会生成 zip 插件包。
4. Magisk Manager 安装插件，重启手机。
5. 在 `/data/local/tmp/pkg.conf` 中添加要 hook 的包名。
6. 将 Frida 脚本 push 到 `/data/local/tmp/myscript.js`。
7. 在 `logcat` 中可以看到 Frida 脚本中输出的日志。

## 源码

`8d5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6K6N6X3g2F1k6$3!0F1k6#2)9J5c8Y4S2U0N6h3u0W2j5X3q4K6k6g2)9#2k6Y4u0A6M7Y4g2Q4x3X3g2Y4K9i4b7`。

`386K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6K6N6X3g2F1k6$3!0F1k6#2)9J5c8Y4S2U0N6h3u0W2j5X3q4K6k6g2)9#2k6Y4u0A6M7Y4g2Q4x3X3g2Y4K9i4b7`。

## 相关链接

- [冰与火的战歌：Windows 内核攻防实战高级班！从零到实战，融合 AI 与 Windows 内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
- [#逆向分析](https://bbs.kanxue.com/forum-161-1-118.htm)
- [#NDK分析](https://bbs.kanxue.com/forum-161-1-119.htm)
- [#混淆加固](https://bbs.kanxue.com/forum-161-1-121.htm)
- [#脱壳反混淆](https://bbs.kanxue.com/forum-161-1-122.htm)
- [#HOOK注入](https://bbs.kanxue.com/forum-161-1-125.htm)
- [#工具脚本](https://bbs.kanxue.com/forum-161-1-128.htm)
