
```
2023-04-01 23:51:02.119 22893-22970/? I/Zhenxi: [文件名]strcmp() arg1 -> /storage/emulated/0/DCIM/.tmfs/.turing.dat arg2-> /storage/emulated/0/.turing.dat
2023-04-01 23:51:02.119 22893-22970/? I/Zhenxi: [文件名]strcmp() arg1 -> /storage/emulated/0/DCIM/.tmfs/.turing.dat arg2-> /storage/emulated/0/DCIM/.tmfs/.turing.dat
2023-04-01 23:51:02.119 22893-22970/? I/Zhenxi: [文件名]strcmp() arg1 -> /storage/emulated/0/.turing.dat arg2-> /storage/emulated/0/.turing.dat
2023-04-01 23:51:02.120 22893-22970/? I/Zhenxi: [文件名]strcmp() arg1 -> /data/user/0/com.xxxxxx.vvvv/app\_turingdfp/1/.turing.dat arg2-> /storage/emulated/0/.turing.dat
2023-04-01 23:51:02.120 22893-22970/? I/Zhenxi: [文件名]strcmp() arg1 -> /data/user/0/com.xxxxxx.vvvv/app\_turingdfp/1/.turing.dat arg2-> /storage/emulated/0/DCIM/.tmfs/.turing.dat
2023-04-01 23:51:02.136 23013-23041/? I/Zhenxi: [文件名]strcpy() arg1 -> android.os.Handler$MessengerImpl arg2-> android.os.Handler$MessengerImplresult -> android.os.Handler$MessengerImpl
2023-04-01 23:51:02.136 23013-23041/? I/Zhenxi: [文件名]strcmp() arg1 -> org.chromium.android\_webview.AwContents arg2-> android.os.Handler$MessengerImpl
2023-04-01 23:51:02.136 23013-23041/? I/Zhenxi: [文件名]strcmp() arg1 -> android.app.ActivityThread$ApplicationThread arg2-> android.os.Handler$MessengerImpl
2023-04-01 23:51:02.136 23013-23041/? I/Zhenxi: [文件名]strcpy() arg1 -> android.os.IMessenger$Stub arg2-> android.os.IMessenger$Stubresult -> android.os.IMessenger$Stub
2023-04-01 23:51:02.137 23013-23041/? I/Zhenxi: [文件名]strcmp() arg1 -> org.chromium.android\_webview.AwContents arg2-> android.os.IMessenger$Stub
2023-04-01 23:51:02.137 23013-23041/? I/Zhenxi: [文件名]strcmp() arg1 -> android.app.ActivityThread$ApplicationThread arg2-> android.os.IMessenger$Stub
2023-04-01 23:51:02.137 23013-23041/? I/Zhenxi: [文件名]strcpy() arg1 -> android.os.Binder arg2-> android.os.Binderresult -> android.os.Binder
2023-04-01 23:51:02.137 23013-23041/? I/Zhenxi: [文件名]strcmp() arg1 -> org.chromium.android\_webview.AwContents arg2-> android.os.Binder
2023-04-01 23:51:02.138 23013-23041/? I/Zhenxi: [文件名]strcmp() arg1 -> android.app.ActivityThread$ApplicationThread arg2-> android.os.Binder
....
```

### JNIEnvRegisterNative注册监听：

这个实现也很简单，直接 hook `artmethod` 里面的 `RegisterNative`，然后调用 `prettyMethod` 函数指针打印 `artmethod` 信息。

我这边在回调里面打印了，**方法基础签名信息 ，绝对地址，相对地址，所属efl文件** 。这个方法里面没做判断，会打印注册全部的信息。

```
HOOK\_DEF(void\*, RegisterNative, void \*thiz, void \*native\_method) {
string basicString = invokePrintf\_org\_PrettyMethodSym(thiz, true);
if (isSave) {
\*invokeOs << basicString.append("\n");
}
Dl\_info info;
dladdr(native\_method, &info);
size\_t relative\_offset =
reinterpret\_cast<size\_t>(native\_method) - reinterpret\_cast<size\_t>(info.dli\_fbase);
LOG(INFO) <<"REGISTER\_NATIVE " << basicString.c\_str() << " absolute address(内存地址) -> "
<< native\_method << " relative offset(相对地址) "<<(void\*)relative\_offset
<<"所属ELF文件 ["<<getFileNameForPath(info.dli\_fname)+"]";
return orig\_RegisterNative(thiz, native\_method);
}
```

打印效果如下：

```
2023-04-01 23:53:12.617 24017-24247/? I/Zhenxi: REGISTER\_NATIVE int com.xxxxxx.bbbb.core.downloadproxy.jni.TPDownloadProxyNative.clearCache(java.lang.String, java.lang.String, int)
absolute address(内存地址) -> 0x77787377fc relative offset(相对地址) 0x3357fc所属ELF文件 [mmmm.so]
2023-04-01 23:53:12.617 24017-24247/? I/Zhenxi: REGISTER\_NATIVE long com.xxxxxx.bbbb.core.downloadproxy.jni.TPDownloadProxyNative.verifyOfflineCacheSync(java.lang.String, int, java.lang.String, java.lang.String)
absolute address(内存地址) -> 0x7778737908 relative offset(相对地址) 0x335908所属ELF文件 [mmmm.so]
2023-04-01 23:53:12.617 24017-24247/? I/Zhenxi: REGISTER\_NATIVE void com.xxxxxx.bbbb.core.downloadproxy.jni.TPDownloadProxyNative.setPlayerState(int, int)
absolute address(内存地址) -> 0x7778737a68 relative offset(相对地址) 0x335a68所属ELF文件 [mmmm.so]
2023-04-01 23:53:12.617 24017-24247/? I/Zhenxi: REGISTER\_NATIVE void com.xxxxxx.bbbb.core.downloadproxy.jni.TPDownloadProxyNative.updateTaskInfo(int, java.lang.String, java.lang.String)
absolute address(内存地址) -> 0x7778737a74 relative offset(相对地址) 0x335a74所属ELF文件 [mmmm.so]
2023-04-01 23:53:12.617 24017-24247/? I/Zhenxi: REGISTER\_NATIVE void com.xxxxxx.bbbb.core.downloadproxy.jni.TPDownloadProxyNative.updatePlayerPlayMsg(int, int, int, int)
absolute address(内存地址) -> 0x7778737b78 relative offset(相对地址) 0x335b78所属ELF文件 [mmmm.so]
2023-04-01 23:53:12.617 24017-24247/? I/Zhenxi: REGISTER\_NATIVE boolean com.xxxxxx.bbbb.core.downloadproxy.jni.TPDownloadProxyNative.isNativeReadyForWork()
absolute address(内存地址) -> 0x7778737b8c relative offset(相对地址) 0x335b8c所属ELF文件 [mmmm.so]
...
```

### LinkerCallBack回调:

这个方法里面主要是 Hook 了 `linker` 底层 `open()` 的方法，在 `Linker` 刚刚将内存加载到内存里还没有进行初始化的时候，得到一个回调。

也是详细打印了各种信息 ，比如**SO开始地址，结束地址，ELF的长度**。可以在这块进行 SO 的 dump 和保存，这个时机点还有一个作用就是做监听和资源文件。

举个栗子，在对游戏源码进行脱壳和修复的时候，比如 LUA 文件的 dump 修复，是需要先 Hook `buffloader` 函数的 ，也就是在这这个时机点进行 Hook 。So 刚刚加载到内存里面，还没有进行源码的加载，即刻进行 Hook ，这么一来他加载的文件都会被拦截。实际太早或者太晚，都可能导致 dump 的不全。

这块也是暴露出来一个时机点，方便 Hook。

```
void onSoLoadedAfter(const char \*filename,void \*ret){
auto mapInfo = getSoBaseAddress(filename);
char buffer[PATH\_MAX];
sprintf(buffer, "linker load %s start-> 0x%zx end-> 0x%zx size -> %lu",
filename, mapInfo.start, mapInfo.end, (mapInfo.end - mapInfo.start));
if (isSave) {
if (hookStrHandlerOs != nullptr) {
(\*hookStrHandlerOs) << buffer;
}
}
LOGI("%s ", buffer);
}
```

打印效果如下：

```
2023-04-01 23:53:12.023 24017-24429/? I/Zhenxi: linker load /data/app/~~KJajvMQT0WLC5kpaiv75pA==/baoming-ZPiDMFjwVCA12Ot9z\_btog==/lib/arm64/yyyy.so start-> 0x77a275b000 end-> 0x77a27ad000 size -> 335872
2023-04-01 23:53:12.140 24017-24247/? I/Zhenxi: linker load /data/app/~~KJajvMQT0WLC5kpaiv75pA==/baoming-ZPiDMFjwVCA12Ot9z\_btog==/lib/arm64/kkkk.so start-> 0x778f68b000 end-> 0x779054c000 size -> 15470592
2023-04-01 23:53:12.172 24017-24424/? I/Zhenxi: linker load libnetd\_client.so start-> 0x78ff86c000 end-> 0x78ff875000 size -> 36864
2023-04-01 23:53:12.362 24017-24429/? I/Zhenxi: linker load /data/app/~~KJajvMQT0WLC5kpaiv75pA==/baoming-ZPiDMFjwVCA12Ot9z\_btog==/lib/arm64/libckeygeneratorV2.so start-> 0x7783a48000 end-> 0x7783aeb000 size -> 667648
2023-04-01 23:53:12.410 24017-24429/? I/Zhenxi: linker load /data/app/~~KJajvMQT0WLC5kpaiv75pA==/baoming-ZPiDMFjwVCA12Ot9z\_btog==/lib/arm64/libxps\_ws.so start-> 0x778159a000 end-> 0x77816ef000 size -> 1396736
2023-04-01 23:53:12.611 24017-24247/? I/Zhenxi: linker load /data/app/~~KJajvMQT0WLC5kpaiv75pA==/baoming-ZPiDMFjwVCA12Ot9z\_btog==/lib/arm64/libDownloadProxy.so start-> 0x7778402000 end-> 0x7778d73000 size -> 9900032
2023-04-01 23:53:12.700 24017-24247/? I/Zhenxi: linker load libc.so start-> 0x7900557000 end-> 0x7900653000 size -> 1032192
2023-04-01 23:53:12.729 23903-23903/? I/Zhenxi: linker load /data/app/~~KJajvMQT0WLC5kpaiv75pA==/baoming-ZPiDMFjwVCA12Ot9z\_btog==/lib/arm64/kkkk.so start-> 0x772d186000 end-> 0x772e047000 size -> 15470592
...
```

### Java 调用方法监听：

主要是 Hook 了 `artmethod` 的 `invoke()` 方法，一切的 Java 方法底层都会走这个方法，包括脱壳也是修改的这个方法获取被抽取的指令，然后对 `Dex` 进行重构。

我想尝试在这个方法里面对参数进行打印，但是失败了，在源码里面很好修改，但是通过 Hook 的话很多函数拿不到 ，加上一些核心的方法被编译器 `inline` 了，很不好操作和处理，就没继续关注了 。如果你有想法和思路可以提 `issues` 或者提交代码。这个方法不建议开启，个人用的很少，主要打印量太大了，一秒几千条日志。他会打印系统的一些 Java 方法，所以很卡顿，不过如果你想做监听和记录，分析一些隐藏 `API` 很有用。`callback` 实现如下。

```
HOOK\_DEF(void\*, invoke, void \*thiz, void \*self, uint32\_t \*args, uint32\_t args\_size, void \*result,
const char \*shorty) {
string basicString = invokePrintf\_org\_PrettyMethodSym(thiz, true);
LOG(INFO) << "invoke method info -> " << basicString;
if (isSave) {
\*invokeOs << basicString.append("\n");
}
return orig\_invoke(thiz, self, args, args\_size, result, shorty);
}
```

### nativeLib :

这个是一个我自己封装的一个 `native` 库，里面有很多常用的方法，都进行了封装 ，删除了一些改机和没用的模块，留了一些常用的模块。

主要是下面几个比较常用的 ，也方便后续开发和维护。

- `hook` 模块
- 符号查找模块（支持非导出函数）
- 常用工具类模块
- `raw_syscall` 模块
- `libpath`

比如常见的 `hook` 操作。如何 `inline hook` 少于四个字节的方法，如何插装 `hook` 和如何异常 `hook`。这些都不需要关注，导入头文件以后直接。

```
HookUtils::Hooker(xxx,(void\*) xxx,(void\*\*) &new\_xxx);
```

一键 `hook` 即可。底层封装分三步实现，先 `dobby hook`，失败了则使用异常 `hook`，最后 `dobby` 插装。代码如下
