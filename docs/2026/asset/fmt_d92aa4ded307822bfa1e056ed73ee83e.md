
## Parcel 中 IBinder 的写入

| 1  2  3  4  5  6  7  8  9  10 |
| --- |
| `static void android_os_Parcel_writeStrongBinder(JNIEnv* env, jclass clazz, jlong nativePtr, jobject object) {`<br>`    Parcel* parcel = reinterpret_cast<Parcel*>(nativePtr);`<br>`    if (parcel != NULL) {`<br>`        const status_t err = parcel->writeStrongBinder(ibinderForJavaObject(env, object));`<br>`        if (err != NO_ERROR) {`<br>`            signalExceptionForError(env, clazz, err);`<br>`        }`<br>`    }`<br>`}` |

主要分为两步，第一步取出 IBinder Java 对象的 Native 指针，即 `IBinder.mObject`，然后将其转为 C++ 的 IBinder 指针传入：

```cpp
status_t Parcel::writeStrongBinder(const sp<IBinder>& val) {
    return flattenBinder(val);
}
```

传输 IBinder 类型的数据同样需要更新 `mObjects` 缓存。

### 其他

除了上面介绍的这些，Parcel 实现中还有许多值得关注的细节，比如 `writeBlob` 同样也是写入 `byte[]`，但对于过大的数据会选择用共享内存的方式去进行传输。

但根据上面的简单分析，我们也能大致看出 Parcel 的一些问题，即序列化和反序列化纯属手工操作，并且在某些操作中没有严格的类型检查等，下面我们就来逐一探讨。

# Bundle

在 Android 中，Bundle 类是一个类似 HashMap 的数据结构，但其是为了在 Parcel 序列化/反序列化的使用中而高度优化的。因此理解 Bundle 本身的序列化过程对我们理解后面的内容也至关重要。

## 序列化

Bundle 的序列化过程调用链路如下：

- `Bundle.writeToParcel`
- `BaseBundle.writeToParcelInner`
- `Parcel.writeArrayMapInternal`

`writeToParcelInner` 中主要负责写入 Bundle 相关的头部字段：

```java
void writeToParcelInner(Parcel parcel, int flags) {
    // Keep implementation in sync with writeToParcel() in
    // frameworks/native/libs/binder/PersistableBundle.cpp.
    final ArrayMap<String, Object> map;
    synchronized (this) {
        // ...
    }

    // Special case for empty bundles.
    if (map == null || map.size() <= 0) {
        parcel.writeInt(0);
        return;
    }

    int lengthPos = parcel.dataPosition();
    parcel.writeInt(-1); // dummy, will hold length
    parcel.writeInt(BUNDLE_MAGIC);

    int startPos = parcel.dataPosition();
    parcel.writeArrayMapInternal(map);
    int endPos = parcel.dataPosition();

    // Backpatch length
    parcel.setDataPosition(lengthPos);
    int length = endPos - startPos;
    parcel.writeInt(length);
    parcel.setDataPosition(endPos);
}
```

`writeArrayMapInternal` 则主要实现 Bundle 内部字典数据的写入：

```java
void writeArrayMapInternal(@Nullable ArrayMap<String, Object> val) {
    if (val == null) {
        writeInt(-1);
        return;
    }

    // Keep the format of this Parcel in sync with writeToParcelInner() in
    // frameworks/native/libs/binder/PersistableBundle.cpp.
    final int N = val.size();
    writeInt(N);
    int startPos;
    for (int i = 0; i < N; i++) {
        writeString(val.keyAt(i));
        writeValue(val.valueAt(i));
    }
}
```

可以看到序列化 Bundle 的过程是和序列化 ArrayMap 一致的，即先写入整型的 size，然后依次写入每个 key 和 value。这里值得注意的是 `writeValue` 的实现：

```java
public final void writeValue(@Nullable Object v) {
    if (v == null) {
        writeInt(VAL_NULL);
    } else if (v instanceof String) {
        writeInt(VAL_STRING);
        writeString((String) v);
    } else if (v instanceof Integer) {
        writeInt(VAL_INTEGER);
        writeInt((Integer) v);
    } else if (v instanceof Map) {
        writeInt(VAL_MAP);
        writeMap((Map) v);
    }
    // ....
    else {
        Class<?> clazz = v.getClass();
        if (clazz.isArray() && clazz.getComponentType() == Object.class) {
            // Only pure Object[] are written here, Other arrays of non-primitive types are
            // handled by serialization as this does not record the component type.
            writeInt(VAL_OBJECTARRAY);
            writeArray((Object[]) v);
        } else if (v instanceof Serializable) {
            // Must be last
            writeInt(VAL_SERIALIZABLE);
            writeSerializable((Serializable) v);
        } else {
            throw new RuntimeException("Parcel: unable to marshal value " + v);
        }
    }
}
```

> 注意，为了便于按照时间线理解历史漏洞，这里代码使用的是 [Android 8.0 中的版本](https://bbs.kanxue.com/elink@28cK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3q4F1k6s2u0G2K9h3c8^5M7X3g2X3i4K6u0W2j5$3!0E0i4K6u0r3z5q4)9J5k6e0m8Q4x3X3f1H3i4K6g2X3M7U0c8Q4x3V1k6^5M7X3g2X3i4K6u0r3k6Y4u0S2L8h3g2%4L8%4u0C8M7#2)9J5c8X3u0S2M7$3g2Q4x3V1k6U0L8%4u0W2i4K6u0r3K9X3q4$3j5g2)9J5c8X3q4F1k6s2u0G2K9h3c8Q4x3V1k6G2M7#2)9J5c8W2m8S2M7X3y4W2L8q4)9J5k6h3A6S2N6X3q4Q4x3U0y4%4M7X3W2@1k6g2k6S2L8s2g2W2)。

`writeValue` 会根据对象类型分别写入一个代表类型的整数以及具体的数据。所支持的类型如下所示：

```java
// Keep in sync with frameworks/native/include/private/binder/ParcelValTypes.h.
private static final int VAL_NULL = -1;
private static final int VAL_STRING = 0;
private static final int VAL_INTEGER = 1;
private static final int VAL_MAP = 2;
private static final int VAL_BUNDLE = 3;
private static final int VAL_PARCELABLE = 4;
private static final int VAL_SHORT = 5;
private static final int VAL_LONG = 6;
private static final int VAL_FLOAT = 7;
private static final int VAL_DOUBLE = 8;
private static final int VAL_BOOLEAN = 9;
private static final int VAL_CHARSEQUENCE = 10;
private static final int VAL_LIST = 11;
private static final int VAL_SPARSEARRAY = 12;
private static final int VAL_BYTEARRAY = 13;
private static final int VAL_STRINGARRAY = 14;
private static final int VAL_IBINDER = 15;
private static final int VAL_PARCELABLEARRAY = 16;
private static final int VAL_OBJECTARRAY = 17;
private static final int VAL_INTARRAY = 18;
private static final int VAL_LONGARRAY = 19;
private static final int VAL_BYTE = 20;
private static final int VAL_SERIALIZABLE = 21;
private static final int VAL_SPARSEBOOLEANARRAY = 22;
private static final int VAL_BOOLEANARRAY = 23;
private static final int VAL_CHARSEQUENCEARRAY = 24;
private static final int VAL_PERSISTABLEBUNDLE = 25;
private static final int VAL_SIZE = 26;
private static final int VAL_SIZEF = 27;
private static final int VAL_DOUBLEARRAY = 28;
```

可以看到其中支持大部分基础类型以及 Parcelable、IBinder 等 Parcel 本身所支持的类型。

综上所述，我们可以大致得出 Bundle 的内存布局：

| size | type | value |
| --- | --- | --- |
| 4 | Int | length |
| 4 | Int | `BUNDLE_MAGIC (0x4C444E42)` |
| 4 | Int | `map.size()` |
| xxx | String | `key[0]` |
| xxx | Dynamic | `val[0]` |
| ... | ... | ... |
| xxx | String | `key[map.size() - 1]` |
| xxx | Dynamic | `val[map.size() - 1]` |

其中 key 因为是 String 类型，因此内部是长度 + String16 的布局，而 value 则根据类型不同使用不同的结构。

`key`：

- `length Int32`
- `data String16`

`value`：

- `VAL_XXX Int32`
- `data writeInt/writeString/...`

下面是在一个 Bundle 序列化的示例，原始内容为：

```java
Bundle A = Bundle();
A.putInt("key1", 0x1337);
A.putString("key2", "hello");
A.putLong("key33", 0x0123456789);
```

使用 Parcel 序列化后的二进制数据如下：

![bin.png](https://bbs.kanxue.com/upload/attach/202303/844554_K2X5MST8XX25V3W.jpg "Bundle 序列化")

其中有几个需要注意的细节：

- `length` 字段是不包括 `length + magic` 部分的，因此实际上序列化数据的总大小会比头部的结果大 8 字节即 100；
- String 数据是 4 字节对齐的（如 `key33`）；如果本身已经对齐，也会在后面进行特殊拓展，如（`key1`/`key2`）；特殊拓展的值为 `Parcel::writeInplace` 中的 `padMask`，对于已经 4 字节对齐的 mask 正好是 0。

## 存储

Bundle 虽然内部是 ArrayMap 结构，但实际存储的时候还有一点优化。在平时开发中细心的朋友可能会发现，有时候在获取远程的 Bundle，比如 `CotentResolver.call` 的结果时候，直接打印输出会是 `Bundle[mParcelledData.dataSize=xxx]` 的结果。但是对于自己构建的 Bundle，却可以打印出完整的 Map 元素。

这其实可以从 Bundle 的源码中看出来：

|  |  |
| --- | --- |
