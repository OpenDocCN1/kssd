
## FindClassMethod 分析

``` 
/ArtMethod* FindClassMethod(std::string_view name, std::string_view signature, PointerSize pointer_size) {
return FindClassMethodWithSignature(this, name, signature, pointer_size);
template <typename SignatureType>
static inline ArtMethod* FindClassMethodWithSignature(ObjPtr<Class> this_klass, std::string_view name, const SignatureType& signature, PointerSize pointer_size)
REQUIRES_SHARED(Locks::mutator_lock_) {
for (ArtMethod& method : this_klass->GetDeclaredMethodsSlice(pointer_size)) {
ArtMethod* np_method = method.GetInterfaceMethodIfProxy(pointer_size);
if (np_method->GetName() == name && np_method->GetSignature() == signature) {
return &method;
}
}
ObjPtr<Class> klass = this_klass->GetSuperClass();
ArtMethod* uninherited_method = nullptr;
for (; klass != nullptr; klass = klass->GetSuperClass()) {
DCHECK(!klass->IsProxyClass());
for (ArtMethod& method : klass->GetDeclaredMethodsSlice(pointer_size)) {
if (method.GetName() == name && method.GetSignature() == signature) {
if (IsInheritedMethod(this_klass, klass, method)) {
return &method;
}
uninherited_method = &method;
break;
}
}
if (uninherited_method != nullptr) {
break;
}
}
ObjPtr<Class> end_klass = klass;
DCHECK_EQ(uninherited_method != nullptr, end_klass != nullptr);
klass = this_klass;
...
for (; klass != end_klass; klass = klass->GetSuperClass()) {
DCHECK(!klass->IsProxyClass());
for (ArtMethod& method : klass->GetCopiedMethodsSlice(pointer_size)) {
if (method.GetName() == name && method.GetSignature() == signature) {
return &method;
// No further check needed, copied methods are inherited by definition.
}
}
}
return uninherited_method;
// Return the `uninherited_method` if any.
}
}
```

显然，`FindClassMethod` 这个流程里并没有调用 `ShouldDenyAccessToMember` 函数，因此没有访问限制。

这个函数的主要流程是从类对象 `mirror::Class` 以及其父类中遍历所有 `ArtMethod` 指针，查找与目标 `name` 和 `signture` 匹配的 `ArtMethod` 指针。

# 柳暗花明

既然 `mirror::Class::FindClassMethod` 这个函数中没有对非 SDK 接口进行限制的逻辑，那我们何不直接调用这个函数呢?

若要调用一个非公开的 native 函数，需满足以下两个条件：

> 1. 可以获取到这个函数的地址；
> 2. 能够构造出调用函数需要传递的各个参数；

先看第一个条件，`FindClassMethod` 是一个非 `inline` 函数，并且在系统的 `libart.so` 文件符号表中，可以查找到这个函数对应符号为：

> `_ZN3art6mirror5Class15FindClassMethodENSt3__117basic_string_viewIcNS2_11char_traitsIcEEEES6_NS_11PointerSizeE`

因此，可以使用 Linux 动态库 `dl` 接口轻松获取到这个函数的地址，代码如下：

```
const char *func_name = "_ZN3art6mirror5Class15FindClassMethodENSt3__117basic_string_viewIcNS2_11char_traitsIcEEEES6_NS_11PointerSizeE";
void *art_so_handle = dlopen("libart.so", RTLD_NOW);
void* address = dlsym(art_so_handle, func_name);
auto findClassMethod = reinterpret_cast<void *( *)(void *, std::string_view, std::string_view, size_t)>(address);
```

不过，这里需要注意的是，从 Android 7.0 开始，Android 系统限制了 App 使用 `dlopen` 打开系统动态库。不过，笔者之前开发了一个库可以轻松绕过这种限制。

源码：[bypass_dlfunctions](https://bbs.kanxue.com/elink@d53K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6i4K9h3&6V1P5g2y4Z5j5g2)9J5c8X3u0&6M7r3q4K6M7#2)9#2k6X3c8D9k6Y4g2F1j5%4c8A6L8$3&6K6)

实现原理：[另一种绕过 Android 系统库访问限制的方法](https://bbs.kanxue.com/elink@a96K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4K9h3&6V1P5i4y4Z5j5g2)9J5k6h3N6A6N6r3S2#2j5W2)9J5k6h3W2G2i4K6u0r3x3U0l9J5x3g2)9J5c8U0l9#2i4K6u0r3x3U0k6Q4x3V1k6Q4x3U0g2q4y4g2)9J5y4e0S2r3i4K6t1#2b7e0k6Q4x3U0g2q4y4q4)9J5y4f1t1^5i4K6t1#2z5o6m8Q4x3U0g2q4y4#2)9J5y4f1p5%4i4K6t1#2z5p5c8Q4x3U0g2q4y4#2)9J5y4f1u0n7i4K6t1#2z5e0g2Q4x3U0g2q4z5q4)9J5y4f1u0r3i4K6t1#2z5o6N6m8L8X3c8J5L8$3W2V1i4K6t1#2c8e0N6Q4x3U0g2n7x3#2)9J5y4f1u0n7i4K6t1#2c8e0N6Q4x3U0g2n7b7W2)9J5y4e0W2r3i4K6t1#2c8e0g2Q4x3U0g2n7b7g2)9J5y4e0V1K6i4K6t1#2c8e0S2Q4x3U0g2m8c8g2)9J5y4f1u0r3i4K6t1#2c8e0W2Q4x3U0f1&6y4#2)9J5y4f1q4q4i4K6t1#2c8e0W2Q4x3U0f1&6z5g2)9J5y4e0V1H3i4K6t1#2c8e0g2Q4x3U0f1^5z5q4)9J5y4f1t1$3i4K6t1#2c8e0N6Q4x3U0f1&6b7g2)9J5y4e0R3@1i4K6t1#2c8e0k6Q4x3U0f1&6y4W2)9J5y4f1t1&6i4K6t1#2c8e0k6Q4x3U0g2n7x3#2)9J5y4e0V1#2i4K6u0r3)

再看第二个条件，这个函数需要传递四个参数，第一个是这个函数所在类 `mirror::Class` 的指针, 第二个参数是方法的 `name`，第三个参数是方法的签名，第四个参数是指针的大小。显然，除了第一个对象，其他几个都是已知的。

难点是第一个参数，回到上面，看看 `FindMethodJNI` 这个函数是如何获取到 `mirror::Class` 指针：

```
ObjPtr<mirror::Class> c = soa.Decode<mirror::Class>(jni_class);
```

`Decode` 函数源码如下：

```
// art/runtime/scoped_thread_state_change-inl.h
template<typename T>
inline ObjPtr<T> ScopedObjectAccessAlreadyRunnable::Decode(jobject obj) const {
Locks::mutator_lock_->AssertSharedHeld(Self());
DCHECK(IsRunnable());
// Don't work with raw objects in non-runnable states.
return ObjPtr<T>::DownCast(Self()->DecodeJObject(obj));
}
```

最终调用到了 `Thread::DecodeJObject` 函数，这个函数的逻辑比较清晰，就是从 `jobject` 对应的引用类型的 `table` 中查找对应的 `mirror::Object` 对象:

```
// art/runtime/thread.cc
ObjPtr<mirror::Object> Thread::DecodeJObject(jobject obj) const {
...
IndirectRef ref = reinterpret_cast<IndirectRef>(obj);
IndirectRefKind kind = IndirectReferenceTable::GetIndirectRefKind(ref);
ObjPtr<mirror::Object> result;
bool expect_null = false;
if (kind == kLocal) {
IndirectReferenceTable& locals = tlsPtr_.jni_env->locals_;
result = locals.Get<kWithoutReadBarrier>(ref);
}
else if (kind == kJniTransitionOrInvalid) {
result = reinterpret_cast<mirror::CompressedReference<mirror::Object>*>(obj)->AsMirrorPtr();
VerifyObject(result);
}
else if (kind == kGlobal) {
result = tlsPtr_.jni_env->vm_->DecodeGlobal(ref);
}
else {
result = tlsPtr_.jni_env->vm_->DecodeWeakGlobal(const_cast<Thread*>(this), ref);
if (Runtime::Current()->IsClearedJniWeakGlobal(result)) {
expect_null = true;
result = nullptr;
}
}
...
return result;
}
```

查看 `libart.so` 的符号表，发现 `DecodeJObject` 这个函数也是导出函数，对应的符号为：

> `_ZNK3art6Thread13DecodeJObjectEP8_jobject`

这个函数的第一个参数是当前线程 native `Thread` 的指针。这个指针可以有两种方法获取到，第一种方式是反射获取 `Thread.java` 的 `nativePeer` 成员变量，这个值里存的就是 native `Thread` 的指针，反射过程代码如下在:

```
jclass thread_class = env->FindClass("java/lang/Thread");
jmethodID currentThread_id = env->GetStaticMethodID(thread_class, "currentThread", "()Ljava/lang/Thread;");
jobject current_thread = env->CallStaticObjectMethod(thread_class, currentThread_id);
jfieldID nativePeer_id = env->GetFieldID(thread_class, "nativePeer", "J");
jlong native_thread = env->GetLongField(current_thread, nativePeer_id);
```

使用这种方式需要反射获取 `Thread.java` 中的私有成员变量 `nativePeer`，目前这个变量被列入了 `greylist` 中，`greylist` 中的 `api` 可以被调用，但在未来更高的 `TargetSDK` 版本可能会将其列入黑名单中。

```
Accessing hidden field Ljava/lang/Thread;->nativePeer:J (greylist, JNI, allowed)
```

为此，我们可以使用另外一种方法获取 native `Thread` 指针。

在 `JNIEnv` 结构体中，第一个位置是虚函数表，第二个位置就是 native `Thread` 指针。每个线程中 `JNIEnv` 都是已知的。因此，可以通过下面方法简单得到 `Thread` 指针，和上面的方法得到的结果刚好一致。

```
auto* fakeEnv = reinterpret_cast<FakeJNIEnv*>(jni_env);
void* native_thread = fakeEnv->self_;

struct FakeJNIEnv {
void* vtb_;
void *const self_; // Link to Thread::Current().
void *const vm_; // The invocation interface JavaVM.
};
```
