
# [原创]解决手机设备 TaintDroid 运行程序崩溃

作 者: 蟑螂一号  
时 间: 2013-09-24,21:44:41

最近在研究 TaintDroid 并编译运行刷入到 Nexus s 手机设备运行。结果运行 lbe、高德地图等含有第三方 lib 库时候，程序直接崩溃。通过 log 查看，原来 TaintDroid 在虚拟机中限制了应用加载的库，关键代码可以查看源码 `davilk/vm/Native.cpp` 源文件中的 `dvmLoadNativeCode()` 方法，该方法中有如下代码：

```c
#ifdef WITH_TAINT_TRACKING
// PJG: TODO: factor out this check
if (strncmp(pathName, "/system", sizeof("/system")-1) != 0 && strcmp(pathName, "libjavacore.so") !=0 && strcmp(pathName, "libnativehelper.so") !=0) {
    ALOGW("Denying lib %s (not \"/system\" prefix)\n", pathName);
    return false;
}
if (strstr(pathName, "/../") != NULL) {
    ALOGW("Denying lib %s (contains \"/../\")\n", pathName);
    return false;
}
#endif
```

该代码判断如果应用加载的是应用自己实现的 lib 库，直接返回，导致应用运行崩溃。

## 解决办法

直接把上面代码注释掉就 ok 咯。

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
