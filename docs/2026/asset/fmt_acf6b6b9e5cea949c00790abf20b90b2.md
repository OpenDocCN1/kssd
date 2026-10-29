
`continueWrite` 方法的实现比较复杂，因为其中还支持传入小于 `mDataSize` 的参数去缩小 `Parcel` 内存，不过在我们这里的上下文中仅用于增长内存，因此实际上最终只是通过 `realloc` 或者 `malloc` 去分配更多的内存：

```cpp
if (desired > mDataCapacity) {
    uint8_t* data = reallocZeroFree(mData, mDataCapacity, desired, mDeallocZero);
    if (data) {
        LOG_ALLOC("Parcel %p: continue from %zu to %zu capacity", this, mDataCapacity, desired);
        gParcelGlobalAllocSize += desired;
        gParcelGlobalAllocSize -= mDataCapacity;
        mData = data;
        mDataCapacity = desired;
    } else {
        mError = NO_MEMORY;
        return NO_MEMORY;
    }
}
```

这部分目前只需要了解即可，我们关注的还是前面写入数据的逻辑。回顾一下，这是在 `writeAligned` 方法中直接通过 `memcpy` 去写入的数据，因此对于基础数字类型是没有额外开销的，且序列化的字节序就是当前机器的字节序。这也是为什么 `Parcel` 只适合在同一设备中实现 IPC，如果在不同设备中使用，可能会出现字节序的问题。

### String

说完了 `Int`，我们接着看常用的 `String` 类型，JNI 的定义和实现如下：

```cpp
static void android_os_Parcel_writeString16(JNIEnv* env, jclass clazz, jlong nativePtr, jstring val) {
    Parcel* parcel = reinterpret_cast<Parcel*>(nativePtr);
    if (parcel != nullptr) {
        status_t err = NO_ERROR;
        if (val) {
            // NOTE: Keep this logic in sync with Parcel.cpp
            const size_t len = env->GetStringLength(val);
            const size_t allocLen = len * sizeof(char16_t);
            err = parcel->writeInt32(len);
            char* data = reinterpret_cast<char*>(parcel->writeInplace(allocLen + sizeof(char16_t)));
            if (data != nullptr) {
                env->GetStringRegion(val, 0, len, reinterpret_cast<jchar*>(data));
                *reinterpret_cast<char16_t*>(data + allocLen) = 0;
            } else {
                err = NO_MEMORY;
            }
        } else {
            err = parcel->writeString16(nullptr, 0);
        }
        if (err != NO_ERROR) {
            signalExceptionForError(env, clazz, err);
        }
    }
}
```

和之前差别不大，值得注意的是，`Parcel::writeInplace` 返回的是待写入的内存地址，直接用了 `JNIEnv::GetStringRegion` 去进行写入。如果 Java 传入的字符串是 `null`，则使用 `writeString16(nullptr)` 去写入，其内部实现是写入特殊的整数 `-1`。

> [GetStringRegion](https://bbs.kanxue.com/elink@652K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1k6h3y4Z5L8X3!0@1k6i4y4Q4x3V1k6Y4N6h3W2V1k6i4y4Q4x3V1k6B7L8X3W2Q4x3V1k6K6M7r3g2U0i4K6u0r3k6Y4g2F1j5%4c8A6L8$3&6K6i4K6u0W2K9s2c8E0L8l9`.`.) 返回的是 Unicode 字符，因此每个字符占 2 字节。

```cpp
status_t Parcel::writeString16(const char16_t* str, size_t len) {
    if (str == nullptr)
        return writeInt32(-1);
    // ...
}
```

因此对于字符串结构，`Parcel` 的序列化也是无开销顺序写入的。

### Array

数组也是个常用的数据类型，但不同的数组传输格式有所不同。对于 `char`、`int`、`long` 等原始类型而言，传输数组实际上就是逐个写入每个元素，并且在前面写入数组的大小：

```java
public final void writeIntArray(@Nullable int[] val) {
    if (val != null) {
        int N = val.length;
        writeInt(N);
        for (int i = 0; i < N; i++) {
            writeInt(val[i]);
        }
    } else {
        writeInt(-1);
    }
}
```

但是 `wirteByteArray` 有所优化：

```java
public final void writeByteArray(@Nullable byte[] b, int offset, int len) {
    if (b == null) {
        writeInt(-1);
        return;
    }
    ArrayUtils.throwsIfOutOfBounds(b.length, offset, len);
    nativeWriteByteArray(mNativePtr, b, offset, len);
}
```

JNI 中直接使用 `memcpy` 去写入：

```cpp
static void android_os_Parcel_writeByteArray(
        JNIEnv* env, jclass clazz, jlong nativePtr,
        jobject data, jint offset, jint length) {
    Parcel* parcel = reinterpret_cast<Parcel*>(nativePtr);
    parcel->writeInt32(length);
    void* dest = parcel->writeInplace(length);
    jbyte* ar = (jbyte*)env->GetPrimitiveArrayCritical((jarray)data, 0);
    if (ar) {
        memcpy(dest, ar + offset, length);
        env->ReleasePrimitiveArrayCritical((jarray)data, ar, 0);
    }
}
```

### Parcelable

对于 `Parcelable` 类型的数据，使用 `writeParcelable` 方法进行写入：

```java
public final void writeParcelable(@Nullable Parcelable p, int parcelableFlags) {
    if (p == null) {
        writeString(null);
        return;
    }
    writeParcelableCreator(p);
    p.writeToParcel(this, parcelableFlags);
}

public final void writeParcelableCreator(@NonNull Parcelable p) {
    String name = p.getClass().getName();
    writeString(name);
}
```

其中需要注意的是，在调用 `Parcelable.writeToParcel` 之前，会先获取 `Parcelable` 实际的类名，并以字符串的方式写入。

### FileDescriptor

Android IPC 的一个特点是可以支持传输文件句柄，其 JNI 实现如下：

```cpp
static void android_os_Parcel_writeFileDescriptor(
        JNIEnv* env, jclass clazz, jlong nativePtr, jobject object) {
    Parcel* parcel = reinterpret_cast<Parcel*>(nativePtr);
    if (parcel != NULL) {
        const status_t err = parcel->writeDupFileDescriptor(
                jniGetFDFromFileDescriptor(env, object));
        if (err != NO_ERROR) {
            signalExceptionForError(env, clazz, err);
        }
    }
}
```

`object` 为 `FileDescriptor` 对象，这里的实现就是获取 `FileDescriptor.fd`，即 `open(2)` 返回的、`int` 格式的文件描述符，进行 `fcntl(oldFd, F_DUPFD_CLOEXEC, 0)` 复制后进行写入。

Binder 的系统调用本身就支持传输文件类型的数据，因此这里 `Parcel` 的实现只需要做好上层的封装：

```cpp
status_t Parcel::writeFileDescriptor(int fd, bool takeOwnership) {
    // ...
    switch (rpcFields->mSession->getFileDescriptorTransportMode()) {
    case RpcSession::FileDescriptorTransportMode::UNIX:
    case RpcSession::FileDescriptorTransportMode::TRUSTY: {
        if (status_t err = writeInt32(RpcFields::TYPE_NATIVE_FILE_DESCRIPTOR); err != OK) {
            return err;
        }
        if (status_t err = writeInt32(rpcFields->mFds->size()); err != OK) {
            return err;
        }
    }
    }

    flat_binder_object obj;
    obj.hdr.type = BINDER_TYPE_FD;
    obj.flags = 0;
    obj.binder = 0;
    /* Don't pass uninitialized stack data to a remote process */
    obj.handle = fd;
    obj.cookie = takeOwnership ? 1 : 0;
    return writeObject(obj, true);
}
```

值得一提的是，`writeObject` 中写入数据除了更新上面提到的 `mData` 等字段，还需要更新 `kernelFields`，如 `mObjects` 字段，作为内核参数的记录。

### Binder

在 Android IPC 中，一个重要的参数类型就是回调，即客户端发送一个 `IBinder` 类型的对象给服务端，然后服务端可以调用其 `onTransact` 方法实现反向异步的数据传输。对于这种类型的数据，主要通过 `writeStrongBinder` 方法：

```java
public final void writeStrongBinder(IBinder val) {
    nativeWriteStrongBinder(mNativePtr, val);
}
```

JNI 实现和传输文件类似。
