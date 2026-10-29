
两种实现在处理 ZIP 文件中多个同样名字的文件时是不相同的。Java 实现的处理方式是遍历 ZIP 文件的中心目录（central directory），并且把每个入口加入一个`LinkedHashMap`中。入口的键值使用的是文件名。

```java
private final LinkedHashMap<String, ZipEntry> mEntries

    = new LinkedHashMap<String, ZipEntry>();

int numEntries = it.readShort() & 0xffff;

for (int i = 0; i < numEntries; ++i) {

    ZipEntry newEntry = new ZipEntry(hdrBuf, bin);

    mEntries.put(newEntry.getName(), newEntry);

}
```

之后，`PackageParser`会遍历 ZIP 文件的每个入口，校验文件的签名。这里，代码会遍历`LinkedHashMap`。这样做的结果是，只有指定名字的最后一个入口会被用来做签名校验——之前所有重复的入口都会被忽略。

```java
JarFile jarFile = new JarFile(mArchiveSourcePath);

Enumeration<JarEntry> entries = jarFile.entries();

final Manifest manifest = jarFile.getManifest();

while (entries.hasMoreElements()) {

    final JarEntry je = entries.nextElement();

    final Certificate[] localCerts =

        loadCertificates(jarFile, je, readBuffer);

    if (localCerts == null) {

        Slog.e(TAG, "Package " + pkg.packageName

                + " has no certificates at entry "

                + je.getName() + "; ignoring!");
```

## 替换 Dalvik 字节码

到目前为止，所有利用此 bug 的破解都直接遵循了 2013 年美国黑帽大会上发表的原始理论中“APK 代码修改”章节：目标是替换 VM 中由`JarFile`实现加载的 APK 中存储的字节码。它的实现由 C 实现的`unzip`完成。

```c
int numEntries = pArchive->mNumEntries;

pArchive->mHashTableSize =

    dexRoundUpPower2(1 + (numEntries * 4) / 3);

pArchive->mHashTable = calloc(pArchive->mHashTableSize,

    sizeof(ZipHashEntry));

int i;

for (i = 0; i < numEntries; i++) {

    hash = computeHash(ptr + kCDELen, fileNameLen);

    addToHash(pArchive, ptr + kCDELen, fileNameLen, hash);

    ptr += kCDELen + fileNameLen + extraLen + commentLen;

}

///////////////////////////////////////////////////////////////////////////////////////////////

static void addToHash(ZipArchive *pArchive,

    const char *str, int strLen, unsigned int hash)

{

    const int hashTableSize = pArchive->mHashTableSize;

    int ent = hash & (hashTableSize - 1);

    while (pArchive->mHashTable[ent].name != NULL)

        ent = (ent + 1) & (hashTableSize-1);

    pArchive->mHashTable[ent].name = str;

    pArchive->mHashTable[ent].nameLen = strLen;

}
```

这个算法是一个线性探查（linear probing）的 unchained hash 表（不受约束的 hash 表？），不会替换。这样做的结果就是，原始 ZIP 文件中的每个入口都会被存到表里面。查找入口的算法与添加时一样：向前遍历找到一个匹配的。这意味着，会使用先添加的入口，而不是后加入的入口。

因此，如果你往一个现有的 APK 里添加一个新的`classes.dex`入口，并且将其加到 APK 中已经存在的`classes.dex`之前，它就会被校验成功（因为校验时会使用第二个入口），而修改的文件会被 VM 加载（因为它在前面）。

## 经过优化的 dex 文件

这个说起来比做起来简单多了：那些你想利用的 APK 文件通常来自设备里已经存在的制造商文件。然而，由于节省空间和启动速度的原因，这些文件实际上并不包含字节码：里面的`classes.dex`已经在外部优化过了。

我手边闲置的一台 Google Nexus 设备的情况是，没有一个系统包包含有可以被替换的字节码。一个叫 Pau Oliva（他给出了此 bug 的 POC）的人在 android-security 讨论邮件列表里面提到了这个问题：[Info on Android bug 8219321](https://bbs.kanxue.com/elink@be6K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4M7X3!0#2M7s2y4Q4x3X3g2Y4L8$3!0Y4L8r3g2Q4x3X3g2U0L8$3#2Q4x3V1k6X3L8%4u0#2L8g2)9J5c8W2)9J5x3#2)9J5x3h3#2K6k6#2)9J5c8X3q4F1k6s2u0G2K9h3c8Q4x3X3c8K6k6h3y4#2M7X3W2@1P5g2)9J5k6r3c8A6M7$3y4#2M7%4y4Q4x3V1k6a6N6i4u0v1f1e0g2v1P5W2S2G2M7#2)9J5c8U0q4j5N6o6y4Q4y4h3k6B7y4@1S2k6P5o6c8v1)？

是的，但是实际上没有这么简单：系统 App 都是经过 odex 的，并且里面没有`classes.dex`。因此，你可以通过修改资源/XML，但是你不能修改实际的程序，因为它是在 APK 外面的 odex 文件。

因此，看起来大部分的开发都在找制造商发布的 APK 升级。因为升级包必须包含要发布的代码的新版本。这些 APK 是可以修改的。然而，找到这些是可遇不可求的，因此也就不容易做到自动化。

除了`#8219321` bug，还有一个现在已知的签名 bug：`#9695860`。这个是根据所有人都盯着的、等待发布`#8219321`时 AOSP 发布的一个补丁](https://bbs.kanxue.com/elink@3dcK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2L8X3c8J5L8$3W2V1i4K6u0W2k6$3!0G2k6$3I4W2M7$3!0#2M7X3y4W2i4K6u0W2j5$3!0E0i4K6u0r3M7r3I4S2N6r3k6G2M7X3#2Q4x3V1k6D9K9h3u0U0L8%4u0W2i4K6u0r3i4K6u0n7i4K6u0r3z5h3g2V1k6U0b7K6k6r3k6U0j5K6x3#2j5K6M7$3x3h3b7&6y4$3g2T1z5e0p5#2y4X3q4U0y4o6t1#2y4o6p5#2x3X3c8V1j5X3x3#2y4b7`.)，它使我们当中的很多人都知道了这个 bug。

这个 bug 与`#8219321`是不同的。人们想利用它来破解要比`#8219321`困难得多。特别是，人们发现你可以使用它来交换小于 64K（这因此会使你有所限制）的文件内容。

当所有人都在关注`classes.dex`时，Pau Oliva（`#8219321` bug 的漏洞 POC 的开发者）在 Twitter 上说：[很难找到一个使用系统签名的、包含 classes.dex 的 APK，现在变成了找到那些 classes.dex 大小小于 64K 的 APK 文件](https://bbs.kanxue.com/elink@dc8K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3q4F1k6s2u0G2K9h3c8Q4x3X3c8G2k6W2)9J5k6r3q4F1L8%4c8Z5k6i4u0Q4x3X3c8S2L8X3c8J5L8$3W2V1k6g2)9J5k6s2k6W2M7X3W2X3K9h3y4S2N6r3W2G2L8W2)9J5k6r3u0&6M7r3q4K6M7#2)9J5c8R3`.`.)。之后，他表示确实[找到了 Motorola 的一个这样的文件](https://bbs.kanxue.com/elink@f35K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3q4F1k6s2u0G2K9h3c8Q4x3X3c8G2k6W2)9J5k6r3q4F1L8%4c8Z5k6i4u0Q4x3X3c8S2L8X3c8J5L8$3W2V1k6g2)9J5k6s2k6W2M7X3W2X3K9h3y4S2N6r3W2G2L8W2)9J5c8R3`.`.)。

要了解目前为止人们利用`#9695860` bug 的更多信息，建议看一下 [Android Security Squad（安卓安全小分队）](https://bbs.kanxue.com/elink@f1eK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3u0D9L8$3N6Q4x3X3g2K6K9h3&6S2i4K6u0W2j5$3!0E0i4K6u0W2j5$3&6Q4x3V1k6K6i4K6u0r3j5X3I4G2k6#2)9#2k6X3u0W2y4X3c8S2j5$3q4W2x3o6p5H3x3h3u0C8M7$3#2Q4x3X3g2Z5N6r3#2D9)的描述，以及（除非你能看懂中文，如果你会，那篇就足够了）[H-Online](https://bbs.kanxue.com/elink@f14K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3S2Q4x3X3c8G2L8X3I4A6L8X3g2Q4x3X3g2U0L8$3#2Q4x3V1k6G2M7r3g2F1i4K6u0r3L8X3g2%4M7#2)9J5c8X3W2@1k6h3#2Q4x3V1k6e0k6h3y4G2L8X3c8Q4x3X3c8m8L8X3c8J5L8$3W2V1i4K6u0V1M7$3W2Y4L8X3q4@1N6i4u0W2i4K6u0V1j5i4c8@1j5h3y4C8i4K6u0V1k6r3W2K6j5$3I4G2M7$3g2V1i4K6u0V1x3e0V1I4z5o6l9$3x3g2)9J5k6h3S2@1L8h3H3`.)和 [Sophos](https://bbs.kanxue.com/elink@f84K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3&6S2K9$3g2V1M7$3g2U0N6i4u0A6N6s2W2Q4x3X3g2K6L8%4m8Z5L8%4y4Q4x3X3g2U0L8$3#2Q4x3V1j5J5x3o6p5K6i4K6u0r3x3o6N6Q4x3V1j5I4y4#2)9J5c8X3q4F1j5i4c8G2L8i4W2Q4x3X3c8G2k6W2)9J5k6r3q4F1L8%4c8Z5k6i4u0Q4x3X3c8S2L8X3c8J5L8$3W2V1i4K6u0V1K9r3!0D9k6g2)9J5k6r3y4Z5K9h3&6W2M7$3g2Q4x3X3c8J5k6i4y4W2j5i4u0U0K9r3g2J5M7#2)9J5k6r3y4D9j5h3W2E0i4K6u0V1L8X3g2%4i4K6u0V1j5$3!0V1k6g2)9J5k6s2k6W2M7X3W2X3K9h3y4S2N6r3W2G2L8W2)9J5k6r3u0&6M7r3q4K6M7#2)9J5c8R3`.`.)上的英文描述。

## 替换原生库

另外一个可能性是替换包中的原生库。这些文件是在包安装或系统启动的时候由 Package Manager 解压出来的。实现此目的的 C++ 代码与 Dalvik 中的 C 代码非常相似，但改动挺大的。

```cpp
int numEntries = mNumEntries;

mHashTableSize = roundUpPower2(1 + (numEntries * 4) / 3);

mHashTable = calloc(mHashTableSize, sizeof(HashEntry));

for (int i = 0; i < numEntries; i++) {

    hash = computeHash(ptr + kCDELen, fileNameLen);

    addToHash(ptr + kCDELen, fileNameLen, hash);

    ptr += kCDELen + fileNameLen + extraLen + commentLen;

}
```

```cpp
void ZipFileRO::addToHash(

    const char *str, int strLen, unsigned int hash

) {

    int ent = hash & (mHashTableSize-1);

    while (mHashTable[ent].name != NULL)

        ent = (ent + 1) & (mHashTableSize-1);

    mHashTable[ent].name = str;

    mHashTable[ent].nameLen = strLen;

}
```

再次，我们可以看到使用了 hash table。与 Dalvik 中的实现一样，从 ZIP 文件中读出的入口保存时没有被替换。然而，与 Dalvik 中不一样的是，使用 ZIP 文件的代码并没有使用 hash table 来通过名字（比如`classes.dex`）查找文件：它是通过遍历整个 table。

```cpp
const int N = zipFile.getNumEntries();

for (int i = 0; i < N; i++) {

    const ZipEntryRO entry = zipFile.findEntryByIndex(i);

    if (strncmp(fileName, APK_LIB, APK_LIB_LEN))

        continue;

    ...

callFunc(env, callArg, &zipFile, entry, ...);
```

这段代码来自于 NativeLibraryHelper 文件中的`iterateOverNativeFiles`函数，它遍历 ZIP 文件中的每个入口。对于每个文件，会调用一个函数来处理这个入口相关的一些事情，比如看它是否是需要解压出来的兼容库（在这种情况下，函数是`copyFileIfChanged`）。
