
```java
public static Object addressToObject64(Class objectType, long address) {
if (objectType == null)
return null;
if (objectType.isPrimitive()) {
if (objectType == int.class) {
return (int)address;
} else if (objectType == long.class) {
return address;
} else if (objectType == short.class) {
return (short)address;
} else if (objectType == byte.class) {
return (byte)address;
} else if (objectType == char.class) {
return (char)address;
} else if (objectType == boolean.class) {
return address != 0;
} else {
throw new RuntimeException("unknown type: " + objectType.toString());
}
} else {
return SandHook.getObject(address);
}
}
```

----

## 对象与地址互转

- 一切的基础在于，当一个对象在栈上时，是不会 Moving GC 的，保证了对象地址在传递中的有效性

- 地址 -> 对象, 使用 ART 内部的 `AddWeakGlobalRef`

- 对象 -> 地址, 直接使用 Java 层的 `Unsafe` 类

----

### 结果

- 如此，几乎 9 成以上的函数 Hook 都走内部 Stub 的方式，Hook 耗时在 1ms 以内

- DexMaker 方式第一次大约需要 80ms 左右，后面直接加载约为 3 - 5ms，其实也能接受

---

## Inline 处理

- ART Inline 优化

- 阻止 JIT Inline

- 阻止 dex2oat Inline(Profile)

- 系统类中 Inline 的如何处理

----

### ART Inline 优化

ART 的 inline 类似其他语言的编译器优化，在 Runtime(JIT) 或者 dex2oat 期间, ART 将 “invoke 字节码指令” 替换成 callee 的方法体。

往往被 inline 的都是较为简单的方法。

----

### 阻止 JIT 期间的 Inline

观察 JIT Inline 的条件：

```cpp
const CompilerOptions& compiler\_options = compiler\_driver\_->GetCompilerOptions();
if ((compiler\_options.GetInlineDepthLimit() == 0)
|| (compiler\_options.GetInlineMaxCodeUnits() == 0)) {
return;
}
bool should\_inline = (compiler\_options.GetInlineDepthLimit() > 0)
&& (compiler\_options.GetInlineMaxCodeUnits() > 0);
if (!should\_inline) {
return;
}
size\_t inline\_max\_code\_units = compiler\_driver\_->GetCompilerOptions().GetInlineMaxCodeUnits();
if (code\_item->insns\_size\_in\_code\_units\_ > inline\_max\_code\_units) {
VLOG(compiler) << "Method " << PrettyMethod(method)
<< " is too big to inline: "
<< code\_item->insns\_size\_in\_code\_units\_
<< " > "
<< inline\_max\_code\_units;
return false;
}
```

----

当被 inline 方法的 code units 大于设置的阈值的时候，方法 Inline 失败。

这个阈值是 `CompilerOptions -> inline_max_code_units_`

----

经过搜索，`CompilerOptions` 一般与 `JitCompiler` 绑定：

```cpp
class JitCompiler {
public:
static JitCompiler\* Create();
virtual ~JitCompiler();
..............
private:
std::unique\_ptr<CompilerOptions> compiler\_options\_;
............
};
} // namespace jit
} // namespace art
```

----

#### 结论

ok，那么我们就得到了 “`static void\* jit\_compiler\_handle\_`” 的 C++ 符号 “`\_ZN3art3jit3Jit20jit\_compiler\_handle\_E`“

最后修改里面的值就可以了。

`SandHook.disableVMInline()`

----

### 阻止 dex2oat Inline

- Android N 以上默认的 ART 编译策略为 speed-profile

- 除了 JIT 期间的内联，系统在空闲时会根据这个所谓的 profile 进行 speed 模式的 dex2oat

- speed 模式包含 Inline 优化

- 现象是 App 多打开几次发现 Hook 不到了

----

#### 如何阻止

- 提前主动调用 dex2oat，附带 `--inline-max-code-units=0` 参数，要求 Dex 是动态加载的 (`ArtDexOptimizer`)

- 如果上述条件无法满足，则需要干掉 profile 文件以阻止空闲时 dex2oat，当然会对性能造成一定影响 (`SandHook.tryDisableProfile`)

----

#### profile

- 首先明确一点是 profile 是安装时创建的，并非由 ART 创建，ART 只负责读取以及写入

- profile 存储了方法的热度，以指导 dex2oat 进程编译高热度方法

- 如果你从 Google Play 下载某个热门 App，是有可能带有一份已经有内容的 profile 的

- 综上，我们只需要删除这个文件，如果不保险，则可以修改权限使之不可写

----

#### profile path

`/data/misc/profiles/cur/" + userId + "/" + selfPackageName + "/primary.prof"`

```cpp
bool ProfileCompilationInfo::Load(const std::string& filename, bool clear\_if\_invalid) {
ScopedTrace trace(\_\_PRETTY\_FUNCTION\_\_);
std::string error;
if (!IsEmpty()) {
return kProfileLoadWouldOverwiteData;
}
int flags = O\_RDWR | O\_NOFOLLOW | O\_CLOEXEC;
ScopedFlock profile\_file = LockedFile::Open(filename.c\_str(), flags,
/\*block\*/false, &error);
if (profile\_file.get() == nullptr) {
LOG(WARNING) << "Couldn't lock the profile file " << filename << ": " << error;
return false;
}
```

----

### 系统类中被 Inline 的

- 理论上非 root 的环境是没什么好的办法的

- 只能特例一个个解决

- `dexOptimize Caller`

----

#### dexOptimize

- 我们 `dexOptimize Caller` 使 `Caller` 重返解释执行，那么就可以顺利的调入我们的 Hook 方法

- `dexOptimize` 其实就是将 `ArtMethod` 的 `CodeEntry` 重新设置成解释 bridge

- 依然是上面提到的 `art_quick_to_interpreter_bridge/art_quick_generic_jni_trampoline`

----

#### 跳板获取

- 这两个跳板不在动态链接表中，`(fake)dlsym` 无法搜索到

- 但是这个符号是被保留的，在 `SHT_SYMTAB` 中，这个表存储了所有的符号，所以我重新实现了符号搜索

- 除此之外也可以从从未调用的方法中获取，但是可能遇到被强制 dex2oat 的情况

<mark class="encrypted">df9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6Y4j5h3&6&6j5h3)9I4x3e0c8Q4x3V1k6m8L8X3c8J5L8$3W2V1c8f1I4r3</mark>

---

## Android Q

- AccessFlag

- Hidden API

----

### AccessFlag

这个比较好解决，Android Q 上也是因为 Hidden Api 机制为每个方法增加了一个 Flag，导致我使用预测 Flag 值在 `ArtMethod` 中搜索 Offset 未能搜到。

`kAccPublicApi = 0x10000000` 代表此方法/Field 为公开 API

----

```cpp
uint32\_t accessFlag = getIntFromJava(jniEnv, "com/swift/sandhook/SandHook",
"testAccessFlag");
if (accessFlag == 0) {
accessFlag = 524313;
//kAccPublicApi
if (SDK\_INT >= ANDROID\_Q) {
accessFlag |= 0x10000000;
}
}
int offset = findOffset(p, getParentSize(), 2, accessFlag);
```

----

### hidden api

这个是从 Android P 就开始引入的反射限制机制。

目前来说有几种方案：

- Hook 法，Hook 某些判断函数，修改 ART 对限制 API 的判断流程

- Hidden API 在内部抽象为 `Policy`，修改全局的 `Policy` 策略，一般是 `Runtime` 中的某个 Flag

- 系统函数调 Hidden API 是 OK 的，想办法让 ART 误以为是系统函数调用 API，一般是修改 `ClassLoader`

----

### P 与 Q 的区别

P 上，判断函数较为集中，`Policy` 的 Flag 也较为好搜索，然而到了 Q 上就多了，至于在 `Runtime` 中搜索 Flag，由于 `Runtime` 是个巨大的结构体，这并不是一个健壮的方法。。。

----

### 另外一种解决办法

- 最后是想办法让 ART 误以为是系统函数调用 API，还有一种办法是双重反射，即用反射调用 `Class.getDeclareMethod` 之类的 API 去使用反射，也能达到目的。(这个方法还是在贴吧偶然看到的，OpenJDK 也有这个问题)

- 后面就简单了，依据此法找到 Hidden API 的开关方法，调用即可。

----

#### 实现

```java
static {
try {
getMethodMethod = Class.class.getDeclaredMethod("getDeclaredMethod", String.class, Class[].class);
forNameMethod = Class.class.getDeclaredMethod("forName", String.class);
vmRuntimeClass = (Class) forNameMethod.invoke(null, "dalvik.system.VMRuntime");
addWhiteListMethod = (Method) getMethodMethod.invoke(vmRuntimeClass, "setHiddenApiExemptions", new Class[]{String[].class});
Method getVMRuntimeMethod = (Method) getMethodMethod.invoke(vmRuntimeClass, "getRuntime", null);
vmRuntime = getVMRuntimeMethod.invoke(null);
} catch (Exception e) {
Log.e("ReflectionUtils", "error get methods", e);
}
}
public static boolean passApiCheck() {
try {
addReflectionWhiteList("Landroid/", "Lcom/android/");
return true;
} catch (Throwable throwable) {
throwable.printStackTrace();
return false;
}
}
```

---

![sandhook\_arch.png](upload/attach/202001/675677\_X69WEGRSA8KGTUV.jpg)

---

## 进程注入

到上面为止，Hook 的大部分细节已经介绍完了，但是本进程的 Hook 不是我们想要的。我们想要将 Hook 作用于其他进程则必须将 Hook 逻辑注入到目标进程。

- Root 注入

- 非 Root 注入，沙箱环境

- 非 Root 插件加载

- 代码植入

----

### Root 注入

Root 注入基本分为：

- 全局注入，一般选择注入 Zygote 进程

- 单进程注入，`ptrace`

----

#### Zygote 注入

- 原版 Xposed 修改了 `app_process` 的源码，在其中加入加载 Xposed 框架的逻辑

- Edxp 依赖 “Riru”，Riru 伪造了 `libmemtrack.so`，在其中加入了加载 “模块” 逻辑，`libmemtrack` 是 Zygote 的必备库，这个库比较简单，所以成为了目标

- 使用 Zygote 的注入，所有 android 进程都将附带 Xposed 的 lib，所以为 “全局注入”

----

#### 单进程注入

- 使用 `ptrace`，核心在于找到 `mmap/dlopen/dlsym` 等函数在目标进程的地址

- 依据本进程计算偏移即可

- 要注意 N 以上对 `dlopen` 的限制

- 问题在于无法绕过目标进程的反调试保护，另外容易错过 Hook 时机

- 另外这种方法也可以注入 Zygote

----

### 非 Root 注入

- 需要沙箱环境，核心在于利用同 UID 免 Root 使用 `ptrace`，步骤和上面相同

- 类似 GG 助手

<mark class="encrypted">833K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6Y4j5h3&6&6j5h3)9I4x3e0c8Q4x3V1k6e0j5h3&6V1b7X3!0^5d9r3!0G2K9#2m8D9N6h3N6A6L8R3`.`.</mark>

----

### 非 Root 插件加载

- 同样需要沙箱环境，不同的是需要沙箱在加载内部 app 进程时主动加载 Xposed 插件

- 这样不会错过 Hook 时机，也不存在反调试问题

- 然而稳定性取决于沙箱本身的稳定性

- 类似 VirtualXposed
