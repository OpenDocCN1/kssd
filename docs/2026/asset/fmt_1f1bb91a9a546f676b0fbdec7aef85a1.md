
`continue;`

...

`callFunc(env, callArg, &zipFile, entry, ...);`

这段代码来自于`NativeLibraryHelper`文件中的`iterateOverNativeFiles`函数，它遍历`zip`文件中的每个入口。对于每个文件，会调用一个函数来处理这个入口相关的一些事情，比如看它是否是需要解压出来的兼容库（在这种情况下，函数是`copyFileIfChanged`）。

看一下[`ZipFileRO.cpp`](https://bbs.kanxue.com/elink@588K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2L8X3c8J5L8$3W2V1i4K6u0W2k6$3!0G2k6$3I4W2M7$3!0#2M7X3y4W2i4K6u0W2j5$3!0E0i4K6u0r3M7r3I4S2N6r3k6G2M7X3#2Q4x3V1k6X3M7X3q4E0k6i4N6G2M7X3E0K6i4K6u0r3L8X3q4@1K9i4k6W2i4K6u0W2k6$3W2@1i4K6u0r3i4K6u0n7i4K6u0r3j5h3&6V1M7X3!0A6k6q4)9J5k6o6c8Q4x3X3f1J5i4K6u0W2x3W2)9#2k6Y4t1I4i4K6u0r3L8r3W2T1M7#2)9J5c8Y4g2@1K9h3I4K6i4K6u0r3h3X3W2H3c8X3W2D9k6g2u0a6i4K6u0W2j5%4m8H3)，我们可以看到，`findEntryByIndex`函数是从`hash table`的顶层开始查找使用过的槽（`slots`）（是的，算法复杂度为`O(N^2)`）。不是吧，这并不是遍历一个`zip`文件内容的合理的方法，:P（Hendy注：这是个表情）。Java代码里面的有关这个`O(N^2)`算法的bug已经被提出来了，这里的bug将来可能也会被修复。

```cpp
ZipEntryRO ZipFileRO::findEntryByIndex(int idx) const {
    if (idx < 0 || idx >= mNumEntries)
        return NULL;
    for (int ent = 0; ent < mHashTableSize; ent++)
        if (mHashTable[ent].name != NULL)
            if (idx-- == 0)
                return (ZipEntryRO) (ent + kZipEntryAdj);
    return NULL;
}
```

不幸的是，这个代码释放出来的库文件会替换之前释放出来的。因此，`hash table`中后面的入口会覆盖前面的。虽然可以使`hash table`重叠（将后面的入口移到数组的最顶层），但很难控制并且不太可能（由于空间预留——`over-provisioning`）。

## 替换 AndroidManifest.xml

Android应用程序包里的其它的文件类型包括资源（`assets and resources`）（技术上来说，`resources`也是存在`assets`中的）。资源是由`AssetManager`来管理的，它使用了与原生库解压逻辑相同的基于`hash table`的C++解压实现。

特别地，每个包都包含一个`AndroidManifest.xml`文件，这个文件描述了包的内容。值得注意的是，这个`bug`的一个`POC`已经在GitHub上提交了一个问题，[`Research exactly what can be modified`](https://bbs.kanxue.com/elink@c75K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6r3N6i4A6A6L8$3^5J5y4q4)9J5c8V1q4F1k6s2u0G2K9h3c8y4j5i4y4@1k6i4u0w2k6i4W2K6i4K6u0r3K9i4y4K6N6h3g2K6i4K6u0r3y4l9`.`.)。注意这句表述：当`AndroidManifest`被破坏时，会出现奇怪的事情。

这个文件很少包含既被Java层的[`PackageParser`](https://bbs.kanxue.com/elink@d18K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2L8X3c8J5L8$3W2V1i4K6u0W2k6$3!0G2k6$3I4W2M7$3!0#2M7X3y4W2i4K6u0W2j5$3!0E0i4K6u0r3M7r3I4S2N6r3k6G2M7X3#2Q4x3V1k6X3M7X3q4E0k6i4N6G2M7X3E0K6i4K6u0r3j5X3q4K6k6g2)9J5k6h3N6A6N6q4)9J5c8W2)9J5b7W2)9J5c8X3q4F1k6s2u0G2K9h3c8Q4x3X3b7@1i4K6u0W2x3W2)9J5k6e0u0Q4y4h3k6J5x3g2)9J5c8X3y4G2M7X3g2Q4x3V1k6B7j5i4k6S2i4K6u0r3j5h3&6V1M7X3!0A6k6q4)9J5c8X3y4G2L8Y4c8W2L8Y4c8Q4x3V1k6H3L8g2)9J5c8W2m8S2j5$3E0S2k6$3g2b7j5i4u0K6k6i4u0Q4x3X3g2B7j5i4k6S2)使用，又被C++层的`AssetManager`使用的属性。因此，我们需要对每个属性的用法进行研究来看看有没有对我们有用的（这听起来比实际上更有趣一些，我正在尝试把这件事情做完——对不起。Hendy注：作者也在研究，并没有搞完）。

```java
private static final String ANDROID_MANIFEST_FILENAME
    = "AndroidManifest.xml";

if ((flags & PARSE_IS_SYSTEM) != 0) {
    JarEntry jarEntry = jarFile.getJarEntry(
        ANDROID_MANIFEST_FILENAME);
    certs = loadCertificates(jarFile, jarEntry, ...);
    ...
} else {
    Enumeration<JarEntry> entries = jarFile.entries();
    while (entries.hasMoreElements()) {
        final JarEntry je = entries.nextElement();
        final String name = je.getName();
        if (ANDROID_MANIFEST_FILENAME.equals(name)) {
            final Attributes attributes =
                manifest.getAttributes(name);
            pkg.manifestDigest =
                ManifestDigest.fromAttributes(attributes);
        }
        ...
    }
}
```

这段从`PackageParser`中提取出来的代码，是用来校验签名的。在此过程中，当遇到`AndroidManifest`时，将它的摘要保存起来以备后用。这个字段在文档注释中可以看到。我们仍然需要看代码以确认它有没有会导致问题的用法。

```java
/**
 * Digest suitable for comparing whether this
 * package's manifest is the same as another.
 */
public ManifestDigest manifestDigest;
```

由于`PackageParser`看到的`AndroidManifest.xml`入口总是从系统中来的，而且我们所关注的`apk`也很可能是已经在系统中安装的。这意味着`manifestDigest`字段会被设置成设备上已经安装的系统包同样的值。

我能想到的唯一使用此字段的地方是包安装进程自身校验安装包文件在`pm`前端工具与后端`PackageManager`服务间传递时在安装进程之外没有改变。

```java
if (!args.manifestDigest.equals(pkg.manifestDigest)) {
    res.returnCode = PackageManager.INSTALL_FAILED_PACKAGE_CHANGED;
    return;
}
```

如果安装包在此过程中修改了，安装进程会失败并返回`INSTALL_FAILED_PACKAGE_CHANGED`。这对于我们来说不是个问题。因为解析并翻译`manifest`来校验都需要`ManifestDigest`对象，而它们都是从Java层获得的。

## 转发锁定公共文件

Hendy注：这部分作者也没搞清楚，可以看我另外翻译的有关转发锁的文章

另外一个使用`java zip`代码访问`AndroidManifest.xml`的来自于[`PackageHelper.java`](https://bbs.kanxue.com/elink@405K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2L8X3c8J5L8$3W2V1i4K6u0W2k6$3!0G2k6$3I4W2M7$3!0#2M7X3y4W2i4K6u0W2j5$3!0E0i4K6u0r3M7r3I4S2N6r3k6G2M7X3#2Q4x3V1k6X3M7X3q4E0k6i4N6G2M7X3E0K6i4K6u0r3j5X3q4K6k6g2)9J5k6h3N6A6N6q4)9J5c8W2)9J5b7W2)9J5c8X3q4F1k6s2u0G2K9h3c8Q4x3X3b7@1i4K6u0W2x3W2)9J5k6e0u0Q4y4h3k6J5x3g2)9J5c8X3y4G2M7X3g2Q4x3V1k6B7j5i4k6S2i4K6u0r3j5$3!0E0i4K6u0r3j5h3&6V1M7X3!0A6k6q4)9J5c8X3W2F1N6r3g2J5L8X3q4D9i4K6u0r3j5$3!0F1N6r3g2F1N6q4)9J5c8W2m8S2j5$3E0S2k6$3g2t1k6h3I4H3k6i4u0Q4x3X3g2B7j5i4k6S2)。，它位于`extractPublicFiles`方法中。这个方法只在具有“转发锁”（Android提供拷贝保护的术语）的包时才被[`PackageManagerService`](https://bbs.kanxue.com/elink@743K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2L8X3c8J5L8$3W2V1i4K6u0W2k6$3!0G2k6$3I4W2M7$3!0#2M7X3y4W2i4K6u0W2j5$3!0E0i4K6u0r3M7r3I4S2N6r3k6G2M7X3#2Q4x3V1k6X3M7X3q4E0k6i4N6G2M7X3E0K6i4K6u0r3j5X3q4K6k6g2)9J5k6h3N6A6N6q4)9J5c8W2)9J5b7W2)9J5c8X3q4F1k6s2u0G2K9h3c8Q4x3X3b7@1i4K6u0W2x3W2)9J5k6e0u0Q4y4h3k6J5x3g2)9J5c8Y4y4W2M7Y4k6A6j5$3g2K6i4K6u0r3K9X3q4$3j5g2)9J5c8X3y4G2L8g2)9J5c8X3q4F1k6s2u0G2K9h3c8Q4x3V1k6K6k6i4u0$3k6i4u0Q4x3V1k6H3L8g2)9J5c8W2m8S2j5$3E0S2k6$3g2y4j5h3&6S2k6$3g2J5f1$3g2J5N6X3W2U0k6g2)9J5k6h3A6S2N6X3p5`.)调用。

```java
// Copy manifest, resources.arsc and res directory to public zip
for (final ZipEntry zipEntry : Collections.list(privateZip.entries())) {
    final String zipEntryName = zipEntry.getName();
    if ("AndroidManifest.xml".equals(zipEntryName)
            || "resources.arsc".equals(zipEntryName)
            || zipEntryName.startsWith("res/")) {
        size += zipEntry.getSize();
        if (publicZipFile != null)
            copyZipEntry(zipEntry, privateZip, publicZipOutStream);
    }
}
```

```java
if (isFwdLocked()) {
    final File destResourceFile =
        new File(getResourcePath());
    PackageHelper.extractPublicFiles(
        codeFileName, destResourceFile);
}
```

往回追溯一个包是如何变成`isFwdLocked`的。它很大程度来自于在调用`pm`前端安装包时是否指定了`-l`参数。我也跟踪了另外一条路径，它包的设置中将其标记为特殊的转发锁。

```java
while ((opt = nextOption()) != null) {
    if (opt.equals("-l")) {
        installFlags |= PackageManager.INSTALL_FORWARD_LOCK;
}
```

```java
if (isForwardLocked(pkg)) {
    currFlags |= PackageManager.INSTALL_FORWARD_LOCK;
    newFlags |= PackageManager.INSTALL_FORWARD_LOCK;
}
```

```java
private static boolean isForwardLocked(PackageParser.Package pkg) {
    return (pkg.applicationInfo.flags & ApplicationInfo.FLAG_FORWARD_LOCK) != 0;
}
```

```java
private boolean isForwardLocked(PackageSetting ps) {
    return (ps.pkgFlags & ApplicationInfo.FLAG_FORWARD_LOCK) != 0;
}
```

```java
/* Set the global "forward lock" flag */
if ((flags & PARSE_FORWARD_LOCK) != 0)
    pkg.applicationInfo.flags |=
        ApplicationInfo.FLAG_FORWARD_LOCK;
```

老实说，当我试图找出如果设定`PARSE_FORWARD_LOCK`时，我发现当安装包的老版本设定了`INSTALL_FORWARD_LOCK`标记时，现在安装会失败（这会导致不停地往前一个版本递归），看起来就成死循环了。

只要不尝试创建一个有此bug的转发锁的应用，你就不应该会有问题。如果有方法可以得到我找不到或没有考虑的带转发锁的应用，读者应该避免使用这种机制。

## 研究 AssetManager

接下来对`manifest`的使用就是`AssetManager`（或`assmgr`）了。`Asset`系统使用`cookie`来跟踪文件（Hendy注：可以参看我写的关于Android资源的笔记），它其实就是文件数组的偏移值：文件被加到`asset path`中，就可以得到一个`cookie`。当要查找一个`asset`时，可以传入`cookie`和`asset`的文件名。

```java
AssetManager assmgr = null;
assmgr = new AssetManager();
int cookie = assmgr.addAssetPath(mArchiveSourcePath);
assmgr.openXmlResourceParser(cookie,
ANDROID_MANIFEST_FILENAME);
```
