
```
public static Method findMethodExact(Class<?> clazz, String methodName, Class<?>... parameterTypes) {

    ...

    Method method = clazz.getDeclaredMethod(methodName, parameterTypes);//获取方法声明

    method.setAccessible(true);

    methodCache.put(fullMethodName, method);

    return method;
}
```

完整代码：

```
public static Method findMethodExact(Class<?> clazz, String methodName, Class<?>... parameterTypes) {

    String fullMethodName = clazz.getName() + '#' + methodName + getParametersString(parameterTypes) + "#exact";

    if (methodCache.containsKey(fullMethodName)) {
        Method method = methodCache.get(fullMethodName);
        if (method == null)
            throw new NoSuchMethodError(fullMethodName);
        return method;
    }

    try {
        Method method = clazz.getDeclaredMethod(methodName, parameterTypes);//获取方法声明
        method.setAccessible(true);
        methodCache.put(fullMethodName, method);
        return method;
    } catch (NoSuchMethodException e) {
        methodCache.put(fullMethodName, null);
        throw new NoSuchMethodError(fullMethodName);
    }
}
```

让我们继续跟进 `hookMethod`。

### `hookMethod`

功能：把方法的参数类型、返回类型、回调函数记录到 `AdditionalHookInfo` 类里，并通过 `hookMethodNative` 方法进入 Native 层进行 Hook。

在这个方法里面差不多都是关键代码，也就不省略，全部贴出。

完整代码：

```
    public static XC_MethodHook.Unhook hookMethod(Member hookMethod, XC_MethodHook callback) {

        /*检测是否是支持的Method类型*/

        if (!(hookMethod instanceof Method) && !(hookMethod instanceof Constructor<?>)) {
            throw new IllegalArgumentException("Only methods and constructors can be hooked: " + hookMethod.toString());
        } else if (hookMethod.getDeclaringClass().isInterface()) {
            throw new IllegalArgumentException("Cannot hook interfaces: " + hookMethod.toString());
        } else if (Modifier.isAbstract(hookMethod.getModifiers())) {
            throw new IllegalArgumentException("Cannot hook abstract methods: " + hookMethod.toString());
        }

        boolean newMethod = false;
        CopyOnWriteSortedSet<XC_MethodHook> callbacks;
        synchronized (sHookedMethodCallbacks) {
            callbacks = sHookedMethodCallbacks.get(hookMethod);
            if (callbacks == null) {//如果为null则函数没有被Hook,否则已经Hook
                callbacks = new CopyOnWriteSortedSet<>();
                sHookedMethodCallbacks.put(hookMethod, callbacks);
                newMethod = true;
            }
        }
        callbacks.add(callback);//记录回调函数

        if (newMethod) {//建立新Hook
            Class<?> declaringClass = hookMethod.getDeclaringClass();//获取类声明
            int slot;
            Class<?>[] parameterTypes;
            Class<?> returnType;
            if (runtime == RUNTIME_ART) {//判断是否是ART模式
                slot = 0;
                parameterTypes = null;
                returnType = null;
            } else if (hookMethod instanceof Method) {//实例方法
                slot = getIntField(hookMethod, "slot");//获取slot
                parameterTypes = ((Method) hookMethod).getParameterTypes();//获取参数类型Class<?>数组 和前面相同
                returnType = ((Method) hookMethod).getReturnType();//获取返回类型
            } else {//构造函数
                slot = getIntField(hookMethod, "slot");//获取slot
                parameterTypes = ((Constructor<?>) hookMethod).getParameterTypes();//获取参数类型Class<?>数组 和前面相同
                returnType = null;//构造参数返回类型为null
            }
            //把回调函数,参数类型,返回类型 记录在一个AdditionalHookInfo类里
            AdditionalHookInfo additionalInfo = new AdditionalHookInfo(callbacks, parameterTypes, returnType);
            //Native方法
            hookMethodNative(hookMethod, declaringClass, slot, additionalInfo);
        }

        return callback.new Unhook(hookMethod);
    }
```

看一下 `AdditionalHookInfo` 构造函数：

```
private AdditionalHookInfo(CopyOnWriteSortedSet<XC_MethodHook> callbacks, Class<?>[] parameterTypes, Class<?> returnType) {
    this.callbacks = callbacks;
    this.parameterTypes = parameterTypes;
    this.returnType = returnType;
}
```

## Native 层分析

### `hookMethodNative`

```
void XposedBridge_hookMethodNative(JNIEnv* env, jclass clazz, jobject reflectedMethodIndirect,
            jobject declaredClassIndirect, jint slot, jobject additionalInfoIndirect) {
    // Usage errors?
    if (declaredClassIndirect == NULL || reflectedMethodIndirect == NULL) {
        dvmThrowIllegalArgumentException("method and declaredClass must not be null");
        return;
    }

    // Find the internal representation of the method
    ClassObject* declaredClass = (ClassObject*) dvmDecodeIndirectRef(dvmThreadSelf(), declaredClassIndirect);
    Method* method = dvmSlotToMethod(declaredClass, slot);
    if (method == NULL) {
        dvmThrowNoSuchMethodError("Could not get internal representation for method");
        return;
    }

    if (isMethodHooked(method)) {
        // already hooked
        return;
    }

    // Save a copy of the original method and other hook info
    XposedHookInfo* hookInfo = (XposedHookInfo*) calloc(1, sizeof(XposedHookInfo));
    memcpy(hookInfo, method, sizeof(hookInfo->originalMethodStruct));
    hookInfo->reflectedMethod = dvmDecodeIndirectRef(dvmThreadSelf(), env->NewGlobalRef(reflectedMethodIndirect));
    hookInfo->additionalInfo = dvmDecodeIndirectRef(dvmThreadSelf(), env->NewGlobalRef(additionalInfoIndirect));

    // Replace method with our own code
    SET_METHOD_FLAG(method, ACC_NATIVE);
    method->nativeFunc = &hookedMethodCallback;
    method->insns = (const u2*) hookInfo;
    method->registersSize = method->insSize;
    method->outsSize = 0;

    if (PTR_gDvmJit != NULL) {
        // reset JIT cache
        char currentValue = *((char*)PTR_gDvmJit + MEMBER_OFFSET_VAR(DvmJitGlobals,codeCacheFull));
        if (currentValue == 0 || currentValue == 1) {
            MEMBER_VAL(PTR_gDvmJit, DvmJitGlobals, codeCacheFull) = true;
        } else {
            ALOGE("Unexpected current value for codeCacheFull: %d", currentValue);
        }
    }
}
```

详细分析：

1. 得到 Native 层的 `Method`

   ```
   ClassObject* declaredClass = (ClassObject*) dvmDecodeIndirectRef(dvmThreadSelf(), declaredClassIndirect);

   Method* method = dvmSlotToMethod(declaredClass, slot);
   ```

   从 Android 源代码的 Dalvik 虚拟机代码里找到 `dvmSlotToMethod` 实现如下：

   实现非常简单，所以 `slot` 相等于记录 `Method` 的下标。

   ```
   Method* dvmSlotToMethod(ClassObject* clazz, int slot)
   {
       if (slot < 0) {
           slot = -(slot+1);
           assert(slot < clazz->directMethodCount);
           return &clazz->directMethods[slot];
       } else {
           assert(slot < clazz->virtualMethodCount);
           return &clazz->virtualMethods[slot];
       }
   }
   ```

### `Method` 结构
