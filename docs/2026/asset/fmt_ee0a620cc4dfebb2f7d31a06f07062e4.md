
```c
// libxposed\_dalvik.cpp
void XposedBridge\_hookMethodNative(JNIEnv\* env, jclass clazz, jobject reflectedMethodIndirect,
jobject declaredClassIndirect, jint slot, jobject additionalInfoIndirect) {
// Usage errors?
if (declaredClassIndirect == NULL || reflectedMethodIndirect == NULL) {
dvmThrowIllegalArgumentException("method and declaredClass must not be null");
return;
}
// Find the internal representation of the method
// dvmDecodeIndirectRef 将间接引用jobject转换为对象引用Object\*
ClassObject\* declaredClass = (ClassObject\*) dvmDecodeIndirectRef(dvmThreadSelf(), declaredClassIndirect);
// dvmSlotToMethod 根据偏移量，从ClassLoader中获取函数指针
Method\* method = dvmSlotToMethod(declaredClass, slot);
if (method == NULL) {
dvmThrowNoSuchMethodError("Could not get internal representation for method");
return;
}
if (isMethodHooked(method)) {
// already hooked
return;
}
// Save a copy of the original method and other hook info
// hookInfo属性配置
XposedHookInfo\* hookInfo = (XposedHookInfo\*) calloc(1, sizeof(XposedHookInfo));
memcpy(hookInfo, method, sizeof(hookInfo->originalMethodStruct));
hookInfo->reflectedMethod = dvmDecodeIndirectRef(dvmThreadSelf(), env->NewGlobalRef(reflectedMethodIndirect));
hookInfo->additionalInfo = dvmDecodeIndirectRef(dvmThreadSelf(), env->NewGlobalRef(additionalInfoIndirect));
// Replace method with our own code
// 修改method变成native方法
SET\_METHOD\_FLAG(method, ACC\_NATIVE);
// 赋予method对象的native方法为hookedMethodCallback
method->nativeFunc = &hookedMethodCallback;
// method的insns数组保存method信息
method->insns = (const u2\*) hookInfo;
method->registersSize = method->insSize;
method->outsSize = 0;
if (PTR\_gDvmJit != NULL) {
// reset JIT cache
char currentValue = \*((char\*)PTR\_gDvmJit + MEMBER\_OFFSET\_VAR(DvmJitGlobals,codeCacheFull));
if (currentValue == 0 || currentValue == 1) {
MEMBER\_VAL(PTR\_gDvmJit, DvmJitGlobals, codeCacheFull) = true;
} else {
ALOGE("Unexpected current value for codeCacheFull: %d", currentValue);
}
}
}
void hookedMethodCallback(const u4\* args, JValue\* pResult, const Method\* method, ::Thread\* self) {
if (!isMethodHooked(method)) {
dvmThrowNoSuchMethodError("Could not find Xposed original method - how did you even get here?");
return;
}
// 获取属性
XposedHookInfo\* hookInfo = (XposedHookInfo\*) method->insns;
Method\* original = (Method\*) hookInfo;
Object\* originalReflected = hookInfo->reflectedMethod;
Object\* additionalInfo = hookInfo->additionalInfo;
// convert/box arguments
const char\* desc = &method->shorty[1]; // [0] is the return type.
Object\* thisObject = NULL;
size\_t srcIndex = 0;
size\_t dstIndex = 0;
// for non-static methods determine the "this" pointer
if (!dvmIsStaticMethod(original)) {
thisObject = (Object\*) args[0];
srcIndex++;
}
ArrayObject\* argsArray = dvmAllocArrayByClass(objectArrayClass, strlen(method->shorty) - 1, ALLOC\_DEFAULT);
if (argsArray == NULL) {
return;
}
while (\*desc != '\0') {
char descChar = \*(desc++);
JValue value;
Object\* obj;
switch (descChar) {
case 'Z':
case 'C':
case 'F':
case 'B':
case 'S':
case 'I':
value.i = args[srcIndex++];
obj = (Object\*) dvmBoxPrimitive(value, dvmFindPrimitiveClass(descChar));
dvmReleaseTrackedAlloc(obj, self);
break;
case 'D':
case 'J':
value.j = dvmGetArgLong(args, srcIndex);
srcIndex += 2;
obj = (Object\*) dvmBoxPrimitive(value, dvmFindPrimitiveClass(descChar));
dvmReleaseTrackedAlloc(obj, self);
break;
case '[':
case 'L':
obj = (Object\*) args[srcIndex++];
break;
default:
ALOGE("Unknown method signature description character: %c", descChar);
obj = NULL;
srcIndex++;
}
setObjectArrayElement(argsArray, dstIndex++, obj);
}
// call the Java handler function
// JValue result;
// dvmCallMethod调用函数methodXposedBridgeHandleHookedMethod
// methodXposedBridgeHandleHookedMethod在initXposedBridge时候已经被赋值了，值为XposedBridge.handleHookedMethod函数
dvmCallMethod(self, (Method\*) methodXposedBridgeHandleHookedMethod, NULL, &result,
originalReflected, (int) original, additionalInfo, thisObject, argsArray);
dvmReleaseTrackedAlloc(argsArray, self);
// exceptions are thrown to the caller
if (dvmCheckException(self)) {
return;
}
// return result with proper type
ClassObject\* returnType = dvmGetBoxedReturnType(method);
if (returnType->primitiveType == PRIM\_VOID) {
// ignored
} else if (result.l == NULL) {
if (dvmIsPrimitiveClass(returnType)) {
dvmThrowNullPointerException("null result when primitive expected");
}
pResult->l = NULL;
} else {
if (!dvmUnboxPrimitive(result.l, returnType, pResult)) {
dvmThrowClassCastException(result.l->clazz, returnType);
}
}
}
private static Object handleHookedMethod(Member method, int originalMethodId, Object additionalInfoObj,
Object thisObject, Object[] args) throws Throwable {
AdditionalHookInfo additionalInfo = (AdditionalHookInfo) additionalInfoObj;
if (disableHooks) {
try {
return invokeOriginalMethodNative(method, originalMethodId, additionalInfo.parameterTypes,
additionalInfo.returnType, thisObject, args);
} catch (InvocationTargetException e) {
throw e.getCause();
}
}
// 获取callback函数的副本
Object[] callbacksSnapshot = additionalInfo.callbacks.getSnapshot();
final int callbacksLength = callbacksSnapshot.length;
// 没有回调函数直接直接原始函数
if (callbacksLength == 0) {
try {
return invokeOriginalMethodNative(method, originalMethodId, additionalInfo.parameterTypes,
additionalInfo.returnType, thisObject, args);
} catch (InvocationTargetException e) {
throw e.getCause();
}
}
// 获取method的参数
MethodHookParam param = new MethodHookParam();
param.method = method;
param.thisObject = thisObject;
param.args = args;
// 执行@beforeHookedMethod的函数
// call "before method" callbacks
int beforeIdx = 0;
do {
try {
((XC\_MethodHook) callbacksSnapshot[beforeIdx]).beforeHookedMethod(param);
} catch (Throwable t) {
XposedBridge.log(t);
// reset result (ignoring what the unexpectedly exiting callback did)
param.setResult(null);
param.returnEarly = false;
continue;
}
if (param.returnEarly) {
// skip remaining "before" callbacks and corresponding "after" callbacks
beforeIdx++;
break;
}
} while (++beforeIdx < callbacksLength);
// call original method if not requested otherwise
if (!param.returnEarly) {
try {
// invokeOriginalMethodNative也是native函数
param.setResult(invokeOriginalMethodNative(method, originalMethodId,
additionalInfo.parameterTypes, additionalInfo.returnType, param.thisObject, param.args));
} catch (InvocationTargetException e) {
param.setThrowable(e.getCause());
}
}
// 执行@afterHookedMethod的函数
// call "after method" callbacks
int afterIdx = beforeIdx - 1;
do {
Object lastResult = param.getResult();
Throwable lastThrowable = param.getThrowable();
try {
((XC\_MethodHook) callbacksSnapshot[afterIdx]).afterHookedMethod(param);
} catch (Throwable t) {
XposedBridge.log(t);
// reset to last result (ignoring what the unexpectedly exiting callback did)
if (lastThrowable == null)
param.setResult(lastResult);
else
param.setThrowable(lastThrowable);
}
} while (--afterIdx >= 0);
// return
if (param.hasThrowable())
throw param.getThrowable();
else
return param.getResult();
}
```

`invokeOriginalMethodNative`是`native`函数，想想它是在什么时候变成`native`函数的呢？

在`onVmCreated`函数执行的时候，回顾一下

```java
Method\* xposedInvokeOriginalMethodNative = (Method\*) env->GetStaticMethodID(classXposedBridge, "invokeOriginalMethodNative",
"(Ljava/lang/reflect/Member;I[Ljava/lang/Class;Ljava/lang/Class;Ljava/lang/Object;[Ljava/lang/Object;)Ljava/lang/Object;");
if (xposedInvokeOriginalMethodNative == NULL) {
ALOGE("ERROR: could not find method %s.invokeOriginalMethodNative(Member, int, Class[], Class, Object, Object[])", CLASS\_XPOSED\_BRIDGE);
dvmLogExceptionStackTrace();
env->ExceptionClear();
return false;
}
dvmSetNativeFunc(xposedInvokeOriginalMethodNative, XposedBridge\_invokeOriginalMethodNative, NULL);
```

通过`dvmSetNativeFunc`将`xposedInvokeOriginalMethodNative`的真正执行函数变为`xposedInvokeOriginalMethodNative`
