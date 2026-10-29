
`cookie`通过`ConvertJavaArrayToDexFiles(env, cookie, /*out*/ dex_files, /*out*/ oat_file)`转换成`DexFile*`实例。

再通过`class_linker->DefineClass`从`DexFile*`里加载类。

## 5. 替换mCookie

### 5.1. 关键点

可以看到`DexFile_defineClassNative`里的`ConvertJavaArrayToDexFiles`函数把`cookie`和`dex_files`、`oat_file`关联了起来。

> `/art/runtime/native/dalvik_system_DexFile.cc`

```
static jclass DexFile_defineClassNative(JNIEnv* env, jclass, jstring javaName, jobject javaLoader, jobject cookie, jobject dexFile) {
std::vector<const DexFile*> dex_files;
const OatFile* oat_file;
if (!ConvertJavaArrayToDexFiles(env, cookie, /*out*/ dex_files, /*out*/ oat_file)) {
VLOG(class_linker) << "Failed to find dex_file";
DCHECK(env->ExceptionCheck());
return nullptr;
}
// ...省略，chinamima
}
```

从这里可以猜测得出，`cookie`和`dex_files`、`oat_file`是一对一关系的。

因此，只需要把`cookie`替换成另一个dex的值，就可以加载另一个dex的类。

### 5.2. 思路

替换`cookie`值在native层比较难操作，因此我们把目光放到java层。

> `java/dalvik/system/DexFile.java`

```
public final class DexFile {
private Object mCookie;

DexFile(String fileName, ClassLoader loader, DexPathList.Element[] elements) throws IOException {
mCookie = openDexFile(fileName, null, 0, loader, elements);
mInternalCookie = mCookie;
mFileName = fileName;
}
// ...省略，chinamima
}
```

在`DexFile`里也有一个`mCookie`，通过`defineClassNative`传参到`DexFile_defineClassNative`的。

也就是替换`mCookie`即可达成目的--加载其他dex。

### 5.3. 实现

根据`DexFile`的构造函数可知。

调用`DexFile`的`openDexFile`得到一个新的`cookie`，用于替换原始的`mCookie`。

### 5.4. 思考

- `openDexFile`需要一个dex的路径，也就是dex需要作为文件释放到某处，容易被截获。
- art模式下，`openDexFile`会调用`dex2oat`生成oat文件，造成一定的性能损耗。

## 6. 替换ClassLoader

### 6.1. 覆盖ClassLoader成员变量

![RePlugin的全局只hook一处的方案](https://bbs.kanxue.com/upload/attach/202104/633423_GCSUEZXVFJVSA5K.png)

### 6.2. 替换LoadedApk里ClassLoader

由`初始化ClassLoader`的调用链路可知，`LoadedApk`保存了`ClassLoader`的实例，实际上这个实例就是`findClass`时用到的`PathClassLoader`。

因此，新new一个`ClassLoader`并替换掉`LoadedApk`里的`mClassLoader`即能实现替换dex效果。

## 7. 添加到DexPathList

```
package dalvik.system;
public class BaseDexClassLoader extends ClassLoader {
private final DexPathList pathList;

protected Class<?> findClass(String name) throws ClassNotFoundException {
Class c = pathList.findClass(name, suppressedExceptions);
if (c == null) {
// ...省略，chinamima
}
return c;
}
}
```

```
package dalvik.system;
final class DexPathList {
private Element[] dexElements;

public Class<?> findClass(String name, List<Throwable> suppressed) {
for (Element element : dexElements) {
Class<?> clazz = element.findClass(name, definingContext, suppressed);
if (clazz != null) {
return clazz;
}
}
// ...省略，chinamima
return null;
}
}
```

从上面代码可知，`ClassLoader`是从`DexPathList pathList`里加载class的。

而`DexPathList`是由`Element[] dexElements`组成的，在`findClass`最先从数组最前面取`Element`。

因此，我们可以在`DexPathList.dexElements`的前面插入隐藏dex的`Element`，从而实现加载隐藏dex。

## 8. 直接加载dex byte

```
public final class DexFile {
private Object mCookie;

DexFile(ByteBuffer buf) throws IOException {
mCookie = openInMemoryDexFile(buf);
mInternalCookie = mCookie;
mFileName = null;
}

private static Object openInMemoryDexFile(ByteBuffer buf) throws IOException {
if (buf.isDirect()) {
return createCookieWithDirectBuffer(buf, buf.position(), buf.limit());
} else {
return createCookieWithArray(buf.array(), buf.position(), buf.limit());
}
}
}
```

发现`DexFile.openInMemoryDexFile`可以直接加载buffer，因此可以从内存中加载dex。

### 8.1. 从内存中加载dex，避免生成oat

#### 8.1.1. 调用链路

```
DexFile.openInMemoryDexFile
├─ DexFile_createCookieWithDirectBuffer
│  └─ CreateSingleDexFileCookie
│      └─ CreateDexFile
│          └─ ArtDexFileLoader::open
│              └─ ArtDexFileLoader::OpenCommon
└──────── ConvertDexFilesToJavaArray
```

关键点在于`ArtDexFileLoader::open`，此处传递了一个`kNoOatDexFile`参数，明显是无`oat`文件的意思。

#### 8.1.2. 详细源码

> `dalvik_system_DexFile.cc`

```
static jobject DexFile_createCookieWithDirectBuffer(JNIEnv* env, jclass, jobject buffer, jint start, jint end) {
// ...省略，chinamima
std::unique_ptr<MemMap> dex_mem_map(AllocateDexMemoryMap(env, start, end));
// ...省略，chinamima
// 创建单个DexFile实例，并转成cookie
return CreateSingleDexFileCookie(env, std::move(dex_mem_map));
}
```

```
static jobject CreateSingleDexFileCookie(JNIEnv* env, std::unique_ptr<MemMap> data) {
// 创建DexFile
std::unique_ptr<const DexFile> dex_file(CreateDexFile(env, std::move(data)));
if (dex_file.get() == nullptr) {
DCHECK(env->ExceptionCheck());
return nullptr;
}
std::vector<std::unique_ptr<const DexFile>> dex_files;
dex_files.push_back(std::move(dex_file));
// 转成cookie
return ConvertDexFilesToJavaArray(env, nullptr, dex_files);
}
```
