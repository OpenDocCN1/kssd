
## 所以看一下 `java_lang_Class` 中的 `GetHiddenapiAccessContextFunction`

```cpp
const AccessContext caller_context = fn_get_access_context(); /* 2. 获取caller的上下文 */
```

```cpp
// java_lang_Class.cc
static std::function<hiddenapi::AccessContext()> GetHiddenapiAccessContextFunction(Thread* self) {
  return [=]() REQUIRES_SHARED(Locks::mutator_lock_) {
    return GetReflectionCaller(self);
  };
}
```

### `GetReflectionCaller` 函数，返回 caller 的 `AccessContext`

```cpp
// Walks the stack, finds the caller of this reflective call and returns
// a hiddenapi AccessContext formed from its declaring class.
static hiddenapi::AccessContext GetReflectionCaller(Thread* self)
    REQUIRES_SHARED(Locks::mutator_lock_) {
  // Walk the stack and find the first frame not from java.lang.Class,
  // java.lang.invoke or java.lang.reflect. This is very expensive.
  // Save this till the last.
  struct FirstExternalCallerVisitor : public StackVisitor {
    explicit FirstExternalCallerVisitor(Thread* thread)
        : StackVisitor(thread, nullptr, StackVisitor::StackWalkKind::kIncludeInlinedFrames),
          caller(nullptr) {
    }

    bool VisitFrame() override REQUIRES_SHARED(Locks::mutator_lock_) { /*获取caller的关键函数*/
      ArtMethod* m = GetMethod();
      if (m == nullptr) {
        // Attached native thread. Assume this is *not* boot class path.
        caller = nullptr;
        return false;
      } else if (m->IsRuntimeMethod()) {
        // Internal runtime method, continue walking the stack.
        return true;
      }

      ObjPtr<mirror::Class> declaring_class = m->GetDeclaringClass();
      if (declaring_class->IsBootStrapClassLoaded()) {
        if (declaring_class->IsClassClass()) {
          return true;
        }
        // Check classes in the java.lang.invoke package. At the time of writing, the
        // classes of interest are MethodHandles and MethodHandles.Lookup, but this
        // is subject to change so conservatively cover the entire package.
        // NB Static initializers within java.lang.invoke are permitted and do not
        // need further stack inspection.
        ObjPtr<mirror::Class> lookup_class = GetClassRoot<mirror::MethodHandlesLookup>();
        if ((declaring_class == lookup_class || declaring_class->IsInSamePackage(lookup_class))
            && !m->IsClassInitializer()) {
          return true;
        }
        // Check for classes in the java.lang.reflect package, except for java.lang.reflect.Proxy.
        // java.lang.reflect.Proxy does its own hidden api checks (https://r.android.com/915496),
        // and walking over this frame would cause a null pointer dereference
        // (e.g. in 691-hiddenapi-proxy).
        ObjPtr<mirror::Class> proxy_class = GetClassRoot<mirror::Proxy>();
        if (declaring_class->IsInSamePackage(proxy_class) && declaring_class != proxy_class) {
          if (Runtime::Current()->isChangeEnabled(kPreventMetaReflectionBlacklistAccess)) {
            return true;
          }
        }
      }

      caller = m;
      return false;
    }

    ArtMethod* caller;
  };

  FirstExternalCallerVisitor visitor(self);
  visitor.WalkStack();

  // Construct AccessContext from the calling class found on the stack.
  // If the calling class cannot be determined, e.g. unattached threads,
  // we conservatively assume the caller is trusted.
  ObjPtr<mirror::Class> caller = (visitor.caller == nullptr)
      ? nullptr : visitor.caller->GetDeclaringClass();
  return caller.IsNull() ? hiddenapi::AccessContext(/* is_trusted= */ true)
                         : hiddenapi::AccessContext(caller);
}
```

#### 最后一行 `caller.IsNull()` 的时候上下文传入了 `true`

```cpp
// art/runtime/hidden_api.h
// Represents the API domain of a caller/callee.
class AccessContext {
 public:
  // Initialize to either the fully-trusted or fully-untrusted domain.
  explicit AccessContext(bool is_trusted)
      : klass_(nullptr),
        dex_file_(nullptr),
        domain_(ComputeDomain(is_trusted)) {}
```

在此，`domain_` 通过 `ComputeDomain` 初始化：

```cpp
static Domain ComputeDomain(bool is_trusted) {
  return is_trusted ? Domain::kCorePlatform : Domain::kApplication;
}
```

#### 当 `is_trusted` 为 true 的时候，确实会获取级别最高的 hiddenapi 访问权限

### 再来看看 `caller` 不为 `null` 时

```cpp
// Initialize from Class.
explicit AccessContext(ObjPtr<mirror::Class> klass)
    REQUIRES_SHARED(Locks::mutator_lock_)
    : klass_(klass),
      dex_file_(GetDexFileFromDexCache(klass->GetDexCache())),
      domain_(ComputeDomain(klass, dex_file_)) {}
```

#### 通过 `klass` 拿到 `dex_file`，然后调用 `ComputeDomain` 计算该 `dex_file` 的 `domain`

```cpp
static Domain ComputeDomain(ObjPtr<mirror::Class> klass, const DexFile* dex_file)
    REQUIRES_SHARED(Locks::mutator_lock_) {
  // Check other aspects of the context.
  Domain domain = ComputeDomain(klass->GetClassLoader(), dex_file);

  if (domain == Domain::kApplication &&
      klass->ShouldSkipHiddenApiChecks() &&
      Runtime::Current()->IsJavaDebuggable()) { /*只有debugable的包才会走if中的逻辑*/
    // Class is known, it is marked trusted and we are in debuggable mode.
    domain = ComputeDomain(/* is_trusted= */ true);
  }

  return domain;
}
```

#### 最终 `dex_file` 的 `domain` 值是通过 `GetHiddenapiDomain()` 获取的

```cpp
static Domain ComputeDomain(ObjPtr<mirror::ClassLoader> class_loader, const DexFile* dex_file) {
  if (dex_file == nullptr) {
    return ComputeDomain(/* is_trusted= */ class_loader.IsNull());
  }

  return dex_file->GetHiddenapiDomain();
}
```

```cpp
hiddenapi::Domain GetHiddenapiDomain() const { return hiddenapi_domain_; }
void SetHiddenapiDomain(hiddenapi::Domain value) const { hiddenapi_domain_ = value; }
```

#### 接下来看看 `dex_file` 的 `domain` 值是如何设置的：直接搜索

```text
liuzhuangzhuang.das@virtualbox:~/work/aosp/art$ cgrep SetHiddenapiDomain
./runtime/native/dalvik_system_DexFile.cc:899:    const_cast<DexFile*>(dex_file)->SetHiddenapiDomain(hiddenapi::Domain::kCorePlatform);
./runtime/hidden_api.cc:125:    dex_file.SetHiddenapiDomain(dex_domain);==============//真正初始化dex_file的domain值是在这里
./libdexfile/dex/dex_file.h:767:  void SetHiddenapiDomain(hiddenapi::Domain value) const { hiddenapi_domain_ = value; }
./test/674-hiddenapi/hiddenapi.cc:49:    const_cast<DexFile*>(dex_file.get())->SetHiddenapiDomain(
./openjdkjvmti/fixed_up_dex_file.cc:151:  new_dex_file->SetHiddenapiDomain(original.GetHiddenapiDomain());
```
