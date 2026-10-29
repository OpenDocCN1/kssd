
```cpp
// Android 9 ~ 11
const void* ArtMethod::RegisterNative(const void* native_method) {
  CHECK(IsNative()) << PrettyMethod();
  CHECK(native_method != nullptr) << PrettyMethod();
  void* new_native_method = nullptr;
  Runtime::Current()->GetRuntimeCallbacks()->RegisterNativeMethod(this,
      native_method, /*out*/&new_native_method);
  SetEntryPointFromJni(new_native_method);
  return new_native_method;
}
// Android 9 以下
const void* ArtMethod::RegisterNative(const void* native_method, bool is_fast) {
  CHECK(IsNative()) << PrettyMethod();
  // 多了一个 FastNative 检测,在已经注册为 FastNative 后不能再次注册
  CHECK(!IsFastNative()) << PrettyMethod();
  CHECK(native_method != nullptr) << PrettyMethod();
  if (is_fast) {
    AddAccessFlags(kAccFastNative);
  }
  void* new_native_method = nullptr;
  Runtime::Current()->GetRuntimeCallbacks()->RegisterNativeMethod(this,
      native_method, /*out*/&new_native_method);
  SetEntryPointFromJni(new_native_method);
  return new_native_method;
}
```

这里可以看到在 `Android 9` 以上直接调用即可覆盖掉，`Android 9` 以下需要清理 `FastNative` 标志

## Java Native 代码实现

所有代码都可以在我的 [fake-linker](https://bbs.kanxue.com/elink@776K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6K6j5h3&6X3k6h3&6Y4b7h3&6V1M7X3!0A6k6q4)9J5c8X3k6S2K9$3g2Q4x3X3c8D9K9h3&6C8k6i4u0Q4x3V1k6T1L8r3!0T1i4K6u0r3L8h3q4A6L8W2)9J5c8Y4y4J5j5#2)9J5c8X3#2S2K9h3&6Q4x3V1k6U0M7s2m8Q4x3V1k6D9K9h3&6C8k6i4u0Q4x3V1k6S2M7Y4c8Q4x3V1k6Z5L8$3!0C8i4K6g2X3K9X3&6A6i4K6g2X3L8X3q4@1K9i4k6W2i4K6g2X3K9h3&6@1k6i4u0X3j5h3y4W2i4K6g2X3K9h3#2H3L8q4)9J5k6h3y4H3M7l9`.`.) 中找到

经过上面分析要 `Hook` 则只需要调用 `RegisterNatives` 方法，而我们还要备份原方法方便后面可以调用，而原方法的地址在 `ArtMethod` 对象中的 `jni` 入口指针中保存，因此需要查找 `ArtMethod`，为了简单适配不同我们自己手动注册一个函数，然后再拿这个地址跟 `ArtMethod` 对象中去比较获取偏移量。

1. 获取方法的 `ArtMethod` 指针，在 `Android 11` 以下 `jmethodID` 就是实际的 `ArtMethod` 指针，`Android 11` 以上不返回真实的 `ArtMethod` 指针，但在 Java Method 对象中有一个 `private long artMethod` 保存着 `ArtMethod` 指针，其它 `Hook` 框架都有介绍就不分析了，因此获取 `ArtMethod` 指针如下

```cpp
static void* GetArtMethod(JNIEnv* env, jclass clazz, jmethodID methodId) {
#if __ANDROID_API__ >= __ANDROID_API_R__
  if (IsIndexId(methodId)){
    jobject method = env->ToReflectedMethod(clazz, methodId, true);
    return reinterpret_cast<void*>(env->GetLongField(method, field_art_method));
  }
#endif
  return methodId;
}
```

2. 查找 `jni` 入口指针偏移，指针对齐

```cpp
uintptr_t* artMethod = static_cast<uintptr_t*>(GetArtMethod(env, clazz, methodId));
bool success = false;
for (int i = 0; i < 30; ++i) {
  // 直接与我们自己注册的 native 方法地址相比较
  if (reinterpret_cast<void*>(artMethod[i]) == native) {
    jni_offset = i;
    success = true;
    LOGD("found art method entrypoint jni offset: %d", i);
    break;
  }
}
```

3. 读取方法原注册地址

```cpp
static void* GetOriginalNativeFunction(const uintptr_t* art_method) {
  if (__predict_false(art_method == nullptr)) {
    return nullptr;
  }
  return (void*) art_method[jni_offset];
}
```

在 `Android 9` 以下如果原方法是 `FastNative` 类型则还需要清除标志，因此还要查找 `uint32_t accessFlags` 成员的偏移，这里我们又采用查找的方式，来确定偏移，使用 `uint32_t` 对齐，我这里选择的是 `0x109` 标志也就是 `public static native`，这里最好要有 `public` 标志 `0x1`，因为正常指针都是 4/8 字节对齐的，这样避免误判

```cpp
if (api >= __ANDROID_API_Q__){
  // 非内部隐藏类
  flags |= 0x10000000;
}
char* start = reinterpret_cast<char*>(artMethod);
for (int i = 1; i < 18; ++i) {
  uint32_t value = *(uint32_t*) (start + i * 4);
  if (value == flags) {
    access_flags_art_method_offset = i * 4;
    LOGD("found art method match access flags offset: %d", i * 4);
    success &= true;
    break;
  }
}
if (access_flags_art_method_offset < 0){
  if (api >= __ANDROID_API_N__) {
    access_flags_art_method_offset = 4;
  } else if (api == __ANDROID_API_M__){
    access_flags_art_method_offset = 12;
  } else if (api == __ANDROID_API_L_MR1__){
    access_flags_art_method_offset = 20;
  } else if (api == __ANDROID_API_L__){
    access_flags_art_method_offset = 56;
  }
}
```

准备工作完成了就该实现具体的 `Hook`

```cpp
int RegisterNativeAgain(JNIEnv* env, jclass clazz, HookRegisterNativeUnit* items, size_t len) {
  if (clazz == nullptr || items == nullptr || len < 1) {
    LOGE("Registration class or method cannot be empty");
    return -1;
  }
  int success = 0;
  JNINativeMethod methods[1];
  for (int i = 0; i < len; ++i) {
    JNINativeMethod hook = items[i].hook_method;
    const char* sign = hook.signature;
    if (sign[0] == '!') {
      sign++;
    }
    // 第一步：查找方法的 jmethodID，这里要去除 FastNative标志的签名，否者查找不成功
    jmethodID methodId = items[i].is_static ? env->GetStaticMethodID(clazz, hook.name, sign) : env->GetMethodID(clazz, hook.name, sign);
    if (methodId == nullptr) {
      LOGE("Find method failed, name: %s, signature: %s, is static: %d", hook.name, hook.signature, items[i].is_static);
      JNIHelper::PrintAndClearException(env);
      continue;
    }
    // 第二步：获取方法对应的 ArtMethod 指针
    void* artMethod = GetArtMethod(env, clazz, methodId);
    // 第三步：备份原方法
    void* backup = GetOriginalNativeFunction(static_cast<uintptr_t*>(artMethod));
    if (backup == hook.fnPtr) {
      LOGE("The same native method has been registered, name: %s, signature: %s, address: %p, is static: %d",
          hook.name, hook.signature, hook.fnPtr, items[i].is_static);
      continue;
    }
    if (items[i].backup_method != nullptr) {
      *(items[i].backup_method) = backup;
    }
    if (!HasAccessFlag(reinterpret_cast<char*>(artMethod), kAccNative)) {
      LOGE("You are hooking a non-native method, name: %s, signature: %s, is static: %d", hook.name, hook.signature, items[i].is_static);
      continue;
    }
    // 第四步：Android 9 以下如果有 FastNative 标志则还需要清除
    bool restore = ClearFastNativeFlag(reinterpret_cast<char*>(artMethod));
    if (api >= __ANDROID_API_O__) {
      hook.signature = sign;
    }
    methods[0] = hook;
    // 第五步：重新注册为我们的 Hook 方法
    if (env->RegisterNatives(clazz, methods, 1) == JNI_OK) {
      success++;
      /*
      * Android 8.0,8.1 必须清除 FastNative 标志才能注册成功,所以如果原来包含 FastNative 标志还得恢复,
      * 否者调用原方法可能会出现问题
      */
      // 第六步：如果需要则恢复原函数的 FastNative 标志
      if (restore && (api == __ANDROID_API_O__ || api == __ANDROID_API_O_MR1__)) {
        AddAccessFlag(reinterpret_cast<char*>(artMethod), kAccFastNative);
      }
    } else {
      LOGE("register native function failed, method name: %s, sign: %s, is static: %d", hook.name, hook.signature, items[i].is_static);
      JNIHelper::PrintAndClearException(env);
      if (restore) {
        AddAccessFlag(reinterpret_cast<char*>(artMethod), kAccFastNative);
      }
      if (items[i].backup_method != nullptr) {
        *(items[i].backup_method) = nullptr;
      }
    }
  }
  return success;
}
```

## 实际应用
