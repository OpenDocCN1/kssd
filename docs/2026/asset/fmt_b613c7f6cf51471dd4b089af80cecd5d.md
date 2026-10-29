
```cpp
status_t Parcel::growData(size_t len)
{
if (len > INT32_MAX) {
// don't accept size_t values which may have come from an
// inadvertent conversion from a negative int.
return BAD_VALUE;
}
if (len > SIZE_MAX - mDataSize) return NO_MEMORY; // overflow
if (mDataSize + len > SIZE_MAX / 3) return NO_MEMORY; // overflow
size_t newSize = ((mDataSize+len)*3)/2;
return (newSize <= mDataSize)
? (status_t) NO_MEMORY
: continueWrite(std::max(newSize, (size_t) 128));
}
```

`continueWrite` 方法的实现比较复杂，因为其中还支持传入小于 `mDataSize` 的参数去缩小 `Parcel` 内存，不过在我们这里的上下文中仅用于增长内存，因此实际上最终只是通过 `realloc` 或者 `malloc` 去分配更多的内存:

```cpp
if (desired > mDataCapacity) {
uint8_t* data = reallocZeroFree(mData, mDataCapacity, desired, mDeallocZero);
if (data) {
LOG_ALLOC("Parcel %p: continue from %zu to %zu capacity", this, mDataCapacity,
desired);
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

这部分目前只需要了解即可，我们关注的还是前面写入数据的逻辑，回顾一下是在 `writeAligned` 方法中，直接通过 `memcpy` 去写入的数据，因此对于基础数字类型是没有额外开销的，且序列化的字节序就是当前机器的字节序。这也是为什么 `Parcel` 只适合在同一设备中实现 IPC，如果对于不同设备中可能会出现字节序的问题。

### String

说完了 `Int` 我们接着看常用的 `String` 类型，JNI 的定义和实现如下:

```cpp
static void android_os_Parcel_writeString16(JNIEnv *env, jclass clazz, jlong nativePtr,
jstring val) {
Parcel* parcel = reinterpret_cast<Parcel*>(nativePtr);
if (parcel != nullptr) {
status_t err = NO_ERROR;
if (val) {
// NOTE: Keep this logic in sync with Parcel.cpp
const size_t len = env->GetStringLength(val);
const size_t allocLen = len * sizeof(char16_t);
err = parcel->writeInt32(len);
char *data = reinterpret_cast<char*>(parcel->writeInplace(allocLen + sizeof(char16_t)));
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

和之前差别不大，值得注意的是 `Parcel::writeInplace` 返回的是待写入的内存地址，直接用了 `JNIEnv::GetStringRegion` 去进行写入。如果 Java 传入的字符串是 `null`，则使用 `writeString16(nullptr)` 去写入，其内部实现是写入特殊的整数 `-1`:

> `GetStringRegion` 返回的是 Unicode 字符，因此每个字符占 2 字节；

```cpp
status_t Parcel::writeString16(const char16_t* str, size_t len)
{
if (str == nullptr) return writeInt32(-1);
// ...
}
```

因此对于字符串结构，`Parcel` 的序列化也是无开销顺序写入的。

### Array

数组也是个常用的数据类型，但不同的数组传输格式有所不同。对于 `char`/`int`/`long` 等原始类型而言，传输数组实际上就是逐个写入每个元素，并且在前面写入数组的大小:

```java
public final void writeIntArray(@Nullable int[] val) {
if (val != null) {
int N = val.length;
writeInt(N);
for (int i=0; i<N; i++) {
writeInt(val[i]);
}
} else {
writeInt(-1);
}
}
```

但是 `writeByteArray` 有所优化:

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

JNI 中直接使用 `memcpy` 去写入:

```cpp
static void android_os_Parcel_writeByteArray(JNIEnv* env, jclass clazz, jlong nativePtr,
jobject data, jint offset, jint length)
{
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

对于 `Parcelable` 类型的数据，使用 `writeParcelable` 方法进行写入:

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

其中需要注意的是在调用 `Parcelable.writeToParcel` 之前，会先获取 `Parcelable` 实际的类名，并以字符串的方式写入。

### FileDescriptor

Andorid IPC 的一个特点是可以支持传输文件句柄，其 JNI 实现如下:

```cpp
static void android_os_Parcel_writeFileDescriptor(JNIEnv* env, jclass clazz, jlong nativePtr, jobject object)
{
Parcel* parcel = reinterpret_cast<Parcel*>(nativePtr);
if (parcel != NULL) {
const status_t err =
parcel->writeDupFileDescriptor(jniGetFDFromFileDescriptor(env, object));
if (err != NO_ERROR) {
signalExceptionForError(env, clazz, err);
}
}
}
```

`object` 为 `FileDescriptor` 对象，这里的实现就是获取 `FileDescriptor.fd`，即 `open(2)` 返回的，`int` 格式的文件描述符，进行 `fcntl(oldFd, F_DUPFD_CLOEXEC, 0)` 复制后进行写入。

`Binder` 的系统调用本身就支持传输文件类型的数据，因此这里 `Parcel` 的实现只需要做好上层的封装:

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
obj.binder = 0; /* Don't pass uninitialized stack data to a remote process */
obj.handle = fd;
obj.cookie = takeOwnership ? 1 : 0;
return writeObject(obj, true);
}
```

值得一提的是，`writeObject` 中写入数据除了更新上面提到的 `mData` 等字段，还需要更新 `kernelFields`，如 `mObjects` 字段，作为内核参数的记录。

### Binder

在 Android IPC 中一个重要的参数类型就是回调，即客户端发送一个 `IBinder` 类型的对象给服务端，然后服务端可以调用其 `onTransact` 方法实现反向异步的数据传输。对于这种类型的数据，主要通过 `writeStrongBinder` 方法:

```java
public final void writeStrongBinder(IBinder val) {
nativeWriteStrongBinder(mNativePtr, val);
}
```

JNI 实现和传输文件类似:

```cpp
static void android_os_Parcel_writeStrongBinder(JNIEnv* env, jclass clazz, jlong nativePtr, jobject object)
{
Parcel* parcel = reinterpret_cast<Parcel*>(nativePtr);
if (parcel != NULL) {
const status_t err = parcel->writeStrongBinder(ibinderForJavaObject(env, object));
if (err != NO_ERROR) {
signalExceptionForError(env, clazz, err);
}
}
}
```

主要分为两步，第一步取出 `IBinder` Java 对象的 Native 指针，即 `IBinder.mObject`，然后将其转为 C++ 的 `IBinder` 指针传入:

```cpp
status_t Parcel::writeStrongBinder(const sp<IBinder>& val)
{
return flattenBinder(val);
}
```

传输 `IBinder` 类型的数据同样需要更新 `mObjects` 缓存。

### 其他

除了上面介绍的这些，`Parcel` 实现中还有许多值得关注的细节，比如 `writeBlob` 同样也是写入 `byte[]`，但对于过大的数据会选择用共享内存的方式去进行传输。但根据上面的简单分析，我们也能大致看出 `Parcel` 的一些问题，即序列化和反序列化纯属手工操作，并且在某些操作中没有严格的类型检查等，下面我们就来逐一探讨。

# Bundle

在 Andorid 中，`Bundle` 类是一个类似 `HashMap` 的数据结构，但其是为了在 `Parcel` 序列化/反序列化的使用中而高度优化的。因此理解 `Bundle` 本身的序列化过程对我们理解后面的内容也至关重要。

## 序列化

`Bundle` 的序列化过程调用链路如下:

- `Bundle.writeToParcel`
- `BaseBundle.writeToParcelInner`
- `parcel.writeArrayMapInternal`

`writeToParcelInner` 中主要负责写入 `Bundle` 相关的头部字段:

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

`writeArrayMapInternal` 则主要实现 `Bundle` 内部字典数据的写入:
