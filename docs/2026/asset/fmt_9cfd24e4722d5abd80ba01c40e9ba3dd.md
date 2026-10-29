
```c
static jobject DexFile_openDexFileNative(JNIEnv* env,
jclass,
jstring javaSourceName,
jstring javaOutputName ATTRIBUTE_UNUSED,
jint flags ATTRIBUTE_UNUSED,
jobject class_loader,
jobjectArray dex_elements) {
ScopedUtfChars sourceName(env, javaSourceName);
if (sourceName.c_str() == nullptr) {
return 0;
}
Runtime* const runtime = Runtime::Current();
ClassLinker* linker = runtime->GetClassLinker();
std::vector<std::unique_ptr<const DexFile>> dex_files;
std::vector<std::string> error_msgs;
const OatFile* oat_file = nullptr;
dex_files = runtime->GetOatFileManager().OpenDexFilesFromOat(sourceName.c_str(),
class_loader,
dex_elements,
/*out*/ &oat_file,
/*out*/ &error_msgs);
if (!dex_files.empty()) {
jlongArray array = ConvertDexFilesToJavaArray(env, oat_file, dex_files);
...
return array;
} else {
...
return nullptr;
}
}
```

这样就完成了dex的加载过程，而`BaseDexClassLoader`派生出两个子类加载器：`PathClassLoader`和`DexClassLoader`

![image-20211010132844821](upload/attach/202202/905443_UMC87QBA2UBNZZY.png)

Android中如果`parent`类加载器加载不到类，最终还是会调用`ClassLoader`对象自己的`findClass()`方法

**`loadClass()`加载：**

```java
public abstract class ClassLoader {
public Class<?> loadClass(String className) throws ClassNotFoundException {
return loadClass(className, false);
}
protected Class<?> loadClass(String className, boolean resolve) throws ClassNotFoundException {
//判断当前类加载器是否已经加载过指定类，若已加载则直接返回
Class<?> clazz = findLoadedClass(className);
if (clazz == null) {
//如果没有加载过，则调用parent的类加载递归加载该类，若已加载则直接返回
clazz = parent.loadClass(className, false);
if (clazz == null) {
//还没加载，则调用当前类加载器来加载
clazz = findClass(className);
}
}
return clazz;
}
}
```

该方法的加载流程如下：

1. 判断当前类加载器是否已经加载过指定类，若已加载则直接返回，否则继续执行；
2. 调用`parent`的类加载递归加载该类，检测是否加载，若已加载则直接返回，否则继续执行；
3. 调用当前类加载器，通过`findClass`加载。

**`findLoadedClass`：**

[-> `ClassLoader.java`]

```java
protected final Class<?> findLoadedClass(String name) {
ClassLoader loader;
if (this == BootClassLoader.getInstance())
loader = null;
else
loader = this;
return VMClassLoader.findLoadedClass(loader, name);
}
```

**`findClass`：**

[-> `BaseDexClassLoader.java`]

```java
public class BaseDexClassLoader extends ClassLoader {
protected Class<?> findClass(String name) throws ClassNotFoundException {
Class c = pathList.findClass(name, suppressedExceptions);
...
return c;
}
}
```

**`DexPathList.findClass`：**

```java
public Class findClass(String name, List<Throwable> suppressed) {
for (Element element : dexElements) {
DexFile dex = element.dexFile;
if (dex != null) {
//找到目标类，则直接返回
Class clazz = dex.loadClassBinaryName(name, definingContext, suppressed);
if (clazz != null) {
return clazz;
}
}
}
return null;
}
```

代码解释：

一个`Classloader`可以包含多个dex文件，每个dex文件被封装到一个`Element`对象，这些`Element`对象排列成有序的数组 `dexElements`。当查找某个类时，会遍历所有的dex文件，如果找到则直接返回，不再继续遍历`dexElements`。也就是说当两个类不同的dex中出现，会优先处理排在前面的dex文件，这便是热修复的核心精髓，将需要修复的类所打包的dex文件插入到`dexElements`前面

热修复原理：

现在很多热修复技术就是把修复的dex文件放在`DexPathList`中`Element[]`数组的前面，这样就实现了修复后的`Class`抢先加载了，达到了修改bug的目的

**`DexFile.loadClassBinaryName`：**

```java
public final class DexFile {
public Class loadClassBinaryName(String name, ClassLoader loader, List<Throwable> suppressed) {
return defineClass(name, loader, mCookie, suppressed);
}
private static Class defineClass(String name, ClassLoader loader, Object cookie, List<Throwable> suppressed) {
Class result = null;
try {
result = defineClassNative(name, loader, cookie);
} catch (NoClassDefFoundError e) {
if (suppressed != null) {
suppressed.add(e);
}
} catch (ClassNotFoundException e) {
if (suppressed != null) {
suppressed.add(e);
}
}
return result;
}
}
```

`defineClassNative()`这是native方法。

**`defineClassNative`：**

[-> `dalvik_system_DexFile.cc`]

```c
static jclass DexFile_defineClassNative(JNIEnv* env, jclass, jstring javaName, jobject javaLoader,
jobject cookie) {
std::unique_ptr<std::vector<const DexFile*>> dex_files = ConvertJavaArrayToNative(env, cookie);
if (dex_files.get() == nullptr) {
return nullptr; //dex文件为空, 则直接返回
}
ScopedUtfChars class_name(env, javaName);
if (class_name.c_str() == nullptr) {
return nullptr; //类名为空, 则直接返回
}
const std::string descriptor(DotToDescriptor(class_name.c_str()));
const size_t hash(ComputeModifiedUtf8Hash(descriptor.c_str())); //将类名转换为hash码
for (auto& dex_file : *dex_files) {
const DexFile::ClassDef* dex_class_def = dex_file->FindClassDef(descriptor.c_str(), hash);
if (dex_class_def != nullptr) {
ScopedObjectAccess soa(env);
ClassLinker* class_linker = Runtime::Current()->GetClassLinker();
class_linker->RegisterDexFile(*dex_file);
StackHandleScope<1> hs(soa.Self());
Handle<mirror::ClassLoader> class_loader(
hs.NewHandle(soa.Decode<mirror::ClassLoader*>(javaLoader)));
//获取目标类
mirror::Class* result = class_linker->DefineClass(soa.Self(), descriptor.c_str(), hash,
class_loader, *dex_file, *dex_class_def);
if (result != nullptr) {
// 找到目标对象
return soa.AddLocalReference<jclass>(result);
}
}
}
return nullptr; //没有找到目标类
}
```

在native层创建目标类的对象并添加到虚拟机列表

![image-20211010135119533](upload/attach/202202/905443_VVZREZMVSGHFN86.png)

![image-20211010140620506](upload/attach/202202/905443_3YCRSJDHT277MAD.png)

![image-20211010150633152](upload/attach/202202/905443_F7YZRU2RZJNR2A3.png)

我们继续分析Native层可以发现：

`DexFile.defineClassNative()` 的实现在 `/art/runtime/native/dalvik_system_DexFile.cc`，最终由 `ClassLinker.DefineClass()` 实现

`Class.classForName()` 的实现在 `/art/runtime/native/java_lang_Class.cc`，最终由 `ClassLinker.FindClass()` 实现

`ClassLinker`核心原理：

先从已加载类的 `class_table` 中查询，若找到则直接返回；若找不到则说明该类是第一次加载，则执行加载流程，其中可能需要穿插加载依赖的类，加载完成后将其缓存到 `class_table` 中。

在 `ClassLinker` 中，会维护两类 `class_table`，一类针对基本类，一类针对其它的类。`class_table` 是作为缓存已经加载过的类的缓冲池。不管以什么样的方式去加载类，都需要先从 `class_table` 中先进行查询以提高加载性能。

`ClassLinker` 在加载类的时候遇到该类依赖的类，进行穿插加载依赖类：

![image-20211010150149783](upload/attach/202202/905443_2WAMWN6VJDBV3KR.png)

我们总结`BaseDexClassLoader`初始化和加载原理：

![image-20211010150449575](upload/attach/202202/905443_53XYBNC4YSPB4SX.png)

Android类加载详细流程：

![image-20211010150449575](upload/attach/202202/905443_4BHWCR3HWU5CMYZ.png)

### 3.案例

#### （1）验证类加载器

我们验证App中的`MainActivity`类加载器和系统类`String`类的类加载器：

```java
ClassLoader thisclassloader = MainActivity.class.getClassLoader();
ClassLoader StringClassloader = String.class.getClassLoader();
Log.e("ClassLoader1","MainActivity is in" + thisclassloader.toString());
Log.e("ClassLoader1","String is in" + StringClassloader.toString());
```

![image-20211010155539808](upload/attach/202202/905443_WQUKDJDVHYU8JJZ.png)

我们可以明显发现`PathClassLoader`加载已安装的`APK`类加载器，而`BootClassLoader`加载系统预安装的类

#### （2）遍历父类加载器

```java
public static void printClassLoader(ClassLoader classLoader) {
Log.e("printClassLoader","this->"+ classLoader.toString());
ClassLoader parent = classLoader.getParent();
while (parent!=null){
Log.i("printClassLoader","parent->"+parent.toString());
parent = parent.getParent();
}
}
```

#### （3）验证双亲委派机制

```java
try {
Class StringClass = thisclassloader.loadClass("java.lang.String");
Log.e("ClassLoader1","load StringClass!"+thisclassloader.toString());
} catch (ClassNotFoundException e) {
e.printStackTrace();
Log.e("ClassLoader1","load MainActivity fail!"+thisclassloader.toString());
}
```

我们使用`PathClassLoader`去加载 `String.class`类，还是可以加载成功，因为双亲委派的机制

#### （4）动态加载

这里我借用网上寒冰大佬动态加载的案例，来进一步讲述使用`DexClassLoader`类实现简单的动态加载插件dex，并验证`ClassLoader`的继承关系

我们先编写一个测试类文件，然后生成dex文件

![image-20220218192716863](upload/attach/202202/905443_TEJSDF8ZQYXWT8K.png)

我们先将dex文件放到模拟器的`sdcard/`下

![image-20211010150449575](upload/attach/202202/905443_TQTC2S7ZF9XG8S9.png)

我们新建一个程序，然后编写主程序的代码，并授权sd读取权限

```java
Context appContext = this.getApplication();
testDexClassLoader(appContext,"/sdcard/classes.dex");
```

```xml
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
```

然后我们编写类加载器代码
