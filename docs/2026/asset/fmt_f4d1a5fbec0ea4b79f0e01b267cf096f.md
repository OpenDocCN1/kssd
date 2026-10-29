
```java
if (appClassloader instanceof BaseDexClassLoader) {
} else if (appClassloader instanceof ClassLoader) {
    List<String> nameList = new ArrayList<>();
    BufferedReader br = null;
    try {
        br = new BufferedReader(new InputStreamReader(new FileInputStream(new File(Environment.getExternalStorageDirectory() + "/8958236_classlist.txt")),
                "UTF-8"));
        String lineTxt = null;
        while ((lineTxt = br.readLine()) != null) {
            if (!TextUtils.isEmpty(lineTxt)) {
                String name = lineTxt.replace("L", "").replaceAll("/", ".").replace(";", "");
                Log.e("hook1", "after replace name:" + name);
                nameList.add(name);
            }
        }
        br.close();
    } catch (Exception e) {
        Log.e("hook1", Log.getStackTraceString(e));
    }
    for (String name : nameList) {
        try {
            Log.e("Hook1", "===================>loadClass:" + name);
            appClassloader.loadClass(name);
        } catch (Exception e) {
            Log.e("Hook1", Log.getStackTraceString(e));
        }
    }
    return;
}
```

因此这里我们直接遍历文件，然后调用`loadClass()`即可。

接下来我们对 Dex 方法体进行批量恢复后，可以看到，函数都恢复了。

![图片描述](upload/attach/202011/777497_7AVBCFZ8UGW82JM.png)

少部分没有恢复的函数，是因为这个`classloader`没有找到类，报 class not found。接下来我们可以将所有`classloader`都跑一遍这个`list`即可。

## 方法二

其实 FART 也是解析`DexFile`从而拿到`classlist`的。

既然有了整个`dexFile`文件，那么有啥不能解析的呢。

```c
if (size == 8958236) {
    u4 classDefSize = dex_file.pHeader->classDefsSize;
    u4 classDefsOff = dex_file.pHeader->classDefsOff;
    __android_log_print(5, "hookso", "base:%p", (void *) base);
    __android_log_print(5, "hookso", "classDefSize:%ld classDefsOff:%p", classDefSize,
                        (void *) classDefsOff);
    u4 typeIdsOff = dex_file.pHeader->typeIdsOff;
    u4 stringIdsOff = dex_file.pHeader->stringIdsOff;
    __android_log_print(5, "hookso", "typeIdsOff:%p stringIdsOff:%p", (void *) typeIdsOff,
                        (void *) stringIdsOff);
    int i;
    for (i = 0; i < classDefSize; i++) {
        long currClassAddr = (long) classDefsOff + i * 32 + (long) base;
        __android_log_print(5, "hookso", "currClass ptr:%p", (void *) currClassAddr);
        int *idx = (int *) (currClassAddr);
        __android_log_print(5, "hookso", "currClassIdx:%i", *idx);
        int tmpIdx = *idx;
        long currTypeIdAddr = ((long) typeIdsOff + 4 * tmpIdx + (long) base);
        int *currTypeIdx = (int *) currTypeIdAddr;
        __android_log_print(5, "hookso", "currTypeIdx:%ld", *currTypeIdx);
        tmpIdx = *currTypeIdx;
        long currStringOffAddr = ((long) stringIdsOff + 4 * tmpIdx + (long) base);
        int *currStringOff = (int *) currStringOffAddr;
        __android_log_print(5, "hookso", "currStringOff:%ld", *currStringOff);
        tmpIdx = *currStringOff;
        long off = (long) base + tmpIdx;
        __android_log_print(5, "hookso", "string off:%p", (void *) off);
        const uint8_t *strPtr = (uint8_t *) off;
        DecodeUnsignedLeb128(&strPtr);
        char *classname = (char *) strPtr;
        __android_log_print(5, "hookso", "classname:%s", classname);
    }
}
```

在这里我们可以一层一层的不断获取不断遍历，就可以拿到我们想要的`classname`。

![图片描述](upload/attach/202011/777497_JHQTP55FZQVMBKG.png)

拿到`classname`后我们就可以用这个去遍历`loadClass()`后再 dump dex 即可。

可以看到，这个题目最后想要说的就是自定义`ClassLoader`如何进行函数抽取还原，那么这里我们需要熟悉`DexFile`的文件结构以及了解`ClassLoader`如何使用。

题目出自 2W 班 7 月第三题

**题目要求：基于 Frida 实现的 FART 的一个版本是通过对`ClassLink`类中的`LoadMethod()`函数进行 hook 实现对函数粒度的脱壳的。请编写基于 Xposed 实现的 FART 版本插件，能够实现对函数粒度的脱壳。**

根据题目要求，可以知道，我们通过 hook`LoadMethod()`即可得到`DexFile`，进而得到`base`和`size`，即可 dump 出 dex。如果有函数抽取，即遍历所有的类得到`codeitem`后还原即可。

本次实践分为两个部分，一个即是原始思路，照葫芦画瓢参照 FART 的脱壳思路去编写代码；遇到困难暂时没法解决的时候，转变思路二，改用 FART 配合 Xposed 去实现。

### 解题过程

![图片描述](https://bbs.kanxue.com/upload/attach/202011/777497_SP86CZETMFBZHDT.png)

![图片描述](https://bbs.kanxue.com/upload/attach/202011/777497_G9VDZDEKFX5N7F4.png)

![图片描述](https://bbs.kanxue.com/upload/attach/202011/777497_684KYSNZVCPCZQW.png)

粗略分析一下就是：

1. 拿到`DexFile`，就可以拿到`base`、`size`
2. 拿到`artmethod`，就可以拿到`codeitem`的 offset 和 method idx
3. 计算`codeitem`的长度
4. dump 出来

那么我们在 so 层也可以画葫芦试试。

测试安卓版本为 8.1。

部分代码如下：

```cpp
void *pVoid = old_loadmethod3(thiz, thread, dex_file, it, klass, artmethod);
__android_log_print(5, "hookso", "pVoid ptr:%p", pVoid);

// 获取 base 和 size
const DexHeader *base = dex_file.pHeader;
size_t size = dex_file.pHeader->fileSize;

// 获取 code item offset 和 method idx
uint32_t codeItemOffset = artmethod->dex_code_item_offset_;
uint32_t idx = artmethod->dex_method_index_;

// hook prettyMethod 方法，主动调用获得方法名
const std::string &string = prettyMethodFunction(artmethod, artmethod, true);

// 通过偏移可以拿到 codeitem
long codeItemAddr = (long) base + codeItemOffset;
CodeItem *codeItem = (CodeItem *) codeItemAddr;

// 这部分代码可以直接 dump dex 出来
int pid = getpid();
char dexFilePath[100] = {0};
sprintf(dexFilePath, "/sdcard/xxxxx/%p %d LoadMethod.dex", base, size);
mkdir("/sdcard/xxxxx", 0777);

int fd = open(dexFilePath, O_CREAT | O_RDWR, 666);
if (fd > 0) {
    ssize_t i = write(fd, base, size);
    if (i > 0) {
        close(fd);
    }
}
...
```

上述看起来一步一步的确是可以拿得到`codeitem`，然后进一步拿得到`ins`的。但是有一个问题我一直没法解决，就是某些方法会报 access violation 异常。我通过搜索发现这个是底层发出来的异常，软件层貌似没法 catch。

通过使用 Frida-FART，我发现 Frida 版本的也有这样的问题，但是 Frida 这边可以通过`try catch`捕捉，让程序继续行走。

![图片描述](https://bbs.kanxue.com/upload/attach/202011/777497_VBSAG8GCZ3Y58VP.png)

因为暂时无法解决这个问题，所以我放弃这个方向，尝试用 FART 进行 dump。

这里偷懒，就直接用寒冰大佬的 Xposed + FART ROM 继续 dump，就没有自己编译源码了。这里环境为 Android 6.0。

那么这时候思路就转变了，通过 hook`loadMethod()`方法可以拿到`artmethod`，而根据 FART 代码，`dumpArtMethod()`这个方法，传入`artmethod`即可 dump。

```cpp
// 先执行原来的 loadMethod 逻辑
void *pVoid = old_loadmethod3(thiz, thread, dex_file, it, klass, artmethod);
__android_log_print(5, "hookso", "pVoid ptr:%p", pVoid);

// 然后通过 so 层 hook dumpArtMethod 函数，并将参数传入
try {
    dumpArtMethodFunction(artmethod);
} catch (...) {
}
```

上述通过 so 层调用 FART 的`dumpArtMethodFunction()`方法，可以 dump 出 dex，以及`classlist`。这是 FART 自带的功能。

而在 Java 层，我们需要遍历所有类。

首先 hook 掉`DexClassLoader`和`PathClassLoader`的构造函数，并将`classloader`存起来。

因为是自定义的 Xposed，所以名字为`XcustomBridge`。其实这里就是 Xposed，仅供参考，不可照抄。

```java
XcustomBridge.hookAllConstructors(DexClassLoader.class, new XC_MethodHook() {
    @Override
    protected void afterHookedMethod(MethodHookParam param) throws Throwable {
        super.afterHookedMethod(param);

        final ClassLoader classLoader = (ClassLoader) param.thisObject;
        XcustomBridge.log("DexClassLoader:" + classLoader.toString());

        mClassLoaders.add(classLoader);
    }
});

XcustomBridge.hookAllConstructors(PathClassLoader.class, new XC_MethodHook() {
    @Override
    protected void afterHookedMethod(MethodHookParam param) throws Throwable {
        super.afterHookedMethod(param);

        final ClassLoader classLoader = (ClassLoader) param.thisObject;
        XcustomBridge.log("PathClassLoader:" + classLoader.toString());

        mClassLoaders.add(classLoader);
    }
});
```

紧接着加载我们的 so，并遍历所有的`classloader`，执行`loadClass()`操作。
