
# frida源码阅读之frida-java

```js
function replaceDalvikImplementation (fn) {
if (fn === null && dalvikOriginalMethod === null) {
return;
}
//保存原Method结构
if (dalvikOriginalMethod === null) {
dalvikOriginalMethod = Memory.dup(methodId, DVM\_METHOD\_SIZE);
dalvikTargetMethodId = Memory.dup(methodId, DVM\_METHOD\_SIZE);
}
if (fn !== null) {
implementation = implement(f, fn); //由js代码生成对应的native函数
let argsSize = argTypes.reduce((acc, t) => (acc + t.size), 0);
if (type === INSTANCE\_METHOD) {
argsSize++;
}
/\*
\* make method native (with kAccNative)
\* insSize and registersSize are set to arguments size
\*/
const accessFlags = (Memory.readU32(methodId.add(DVM\_METHOD\_OFFSET\_ACCESS\_FLAGS)) | kAccNative) >>> 0;
const registersSize = argsSize;
const outsSize = 0;
const insSize = argsSize;
Memory.writeU32(methodId.add(DVM\_METHOD\_OFFSET\_ACCESS\_FLAGS), accessFlags);
Memory.writeU16(methodId.add(DVM\_METHOD\_OFFSET\_REGISTERS\_SIZE), registersSize);
Memory.writeU16(methodId.add(DVM\_METHOD\_OFFSET\_OUTS\_SIZE), outsSize);
Memory.writeU16(methodId.add(DVM\_METHOD\_OFFSET\_INS\_SIZE), insSize);
Memory.writeU32(methodId.add(DVM\_METHOD\_OFFSET\_JNI\_ARG\_INFO), computeDalvikJniArgInfo(methodId));
//利用 dvmUSeJNIBridge完成替换Method结构的insns和NativeFunc
api.dvmUseJNIBridge(methodId, implementation);
patchedMethods.add(f);
} else {
patchedMethods.delete(f);
Memory.copy(methodId, dalvikOriginalMethod, DVM\_METHOD\_SIZE);
implementation = null;
}
}
```

hook后执行的`native`函数就是用`implement`函数实现的，该函数时最终调用`frida-node`的`new NativeCallback`接口实现将`js`函数转换为`native`函数。

## frida-java ART Hook原理

常见的ART Hook方法为：替换方法的入口点，即`ArtMethod`的`entry_point_from_quick_compiled_code_`，并将原方法的信息备份到`entry_point_from_jni_`。替换后的入口点，会重新准备栈和寄存器，执行hook的方法。

`frida-java`采用的hook方法，我在其他地方并未遇见过，其原理为：首先将方法native化，然后将`ArtMethod`的`entry_point_from_jni_`替换为hook的方法，并将`entry_point_from_quick_compiled_code_`替换为`art_quick_generic_jni_trampoline`。当调用被hook的方法时，首先会跳转到`art_quick_generic_jni_trampoline`，该函数会做一些`jni`调用的准备，然后跳转到`ArtMethod`结构的`entry_point_from_jni_`所指向的hook方法，这样就完成了一次hook。完成art hook的源码如下：

```js
function replaceArtImplementation (fn) {
if (fn === null && artOriginalMethodInfo === null) {
return;
}
const artMethodSpec = getArtMethodSpec(vm);
const artMethodOffset = artMethodSpec.offset; //获取ArtMethod各个字段的偏移
if (artOriginalMethodInfo === null) {
artOriginalMethodInfo = fetchMethod(methodId); //保存原方法信息
}
if (fn !== null) {
implementation = implement(f, fn);
// kAccFastNative so that the VM doesn't get suspended while executing JNI
// (so that we can modify the ArtMethod on the fly)
patchMethod(methodId, {
//替换entry\_point\_from\_jni\_为hook的方法;
'jniCode': implementation,
//native化
'accessFlags': (Memory.readU32(methodId.add(artMethodOffset.accessFlags)) | kAccNative | kAccFastNative) >>> 0,
//替换entry\_point\_from\_quick\_compiled\_code\_为art\_quick\_generic\_jni\_trampoline;
'quickCode': api.artQuickGenericJniTrampoline,
//entry\_point\_from\_interpreter\_;
'interpreterCode': api.artInterpreterToCompiledCodeBridge
});
patchedMethods.add(f);
} else {
patchedMethods.delete(f);
patchMethod(methodId, artOriginalMethodInfo);
implementation = null;
}
}
```

然后看下`art_quick_generic_jni_trampoline`源码，`art_quick_generic_jni_trampoline`主要负责`jni`调用的准备，包括堆栈的设置，参数的设置等。个人能力有限只能大概看看流程。

```asm
ENTRY art\_quick\_generic\_jni\_trampoline
SETUP\_REFS\_AND\_ARGS\_CALLEE\_SAVE\_FRAME\_WITH\_METHOD\_IN\_R0
// Save rSELF
mov r11, rSELF
// Save SP , so we can have static CFI info. r10 is saved in ref\_and\_args.
mov r10, sp
.cfi\_def\_cfa\_register r10
sub sp, sp, #5120
// prepare for artQuickGenericJniTrampoline call, 转为为c调用约定
// (Thread\*, SP)
// r0 r1 <= C calling convention
// rSELF r10 <= where they are
mov r0, rSELF // Thread\*
mov r1, r10 // ArtMethod\*\*
//调用c的 artQuickGenericJniTrampoline
blx artQuickGenericJniTrampoline // (Thread\*, sp)
// The C call will have registered the complete save-frame on success.
// The result of the call is:
// r0: pointer to native code, 0 on error.
// r1: pointer to the bottom of the used area of the alloca, can restore stack till there.
// Check for error = 0.
cbz r0, .Lexception\_in\_native
// Release part of the alloca.
mov sp, r1
// Save the code pointer
mov r12, r0
// Load parameters from frame into registers.
pop {r0-r3}
```

主要是将调用约定转换为c的调用约定，将参数准备好，然后调用`artQuickGenericJniTrampoline`，该函数源码如下：

```c
extern "C" TwoWordReturn artQuickGenericJniTrampoline(Thread\* self, ArtMethod\*\* sp)
SHARED\_LOCKS\_REQUIRED(Locks::mutator\_lock\_) {
ArtMethod\* called = \*sp;
DCHECK(called->IsNative()) << PrettyMethod(called, true);
uint32\_t shorty\_len = 0;
const char\* shorty = called->GetShorty(&shorty\_len);
// Run the visitor and update sp.
BuildGenericJniFrameVisitor visitor(self, called->IsStatic(), shorty, shorty\_len, &sp);
visitor.VisitArguments();
visitor.FinalizeHandleScope(self);
// Fix up managed-stack things in Thread.
self->SetTopOfStack(sp);
self->VerifyStack();
// Start JNI, save the cookie.
uint32\_t cookie;
if (called->IsSynchronized()) {
cookie = JniMethodStartSynchronized(visitor.GetFirstHandleScopeJObject(), self);
if (self->IsExceptionPending()) {
self->PopHandleScope();
// A negative value denotes an error.
return GetTwoWordFailureValue();
}
} else {
cookie = JniMethodStart(self);
}
uint32\_t\* sp32 = reinterpret\_cast<uint32\_t\*>(sp);
\*(sp32 - 1) = cookie;
// Retrieve the stored native code.
//获取到ArtMethod结构的entry\_point\_from\_jni\_
void\* nativeCode = called->GetEntryPointFromJni();
// There are two cases for the content of nativeCode:
// 1) Pointer to the native function.
// 2) Pointer to the trampoline for native code binding.
// In the second case, we need to execute the binding and continue with the actual native function
// pointer.
DCHECK(nativeCode != nullptr);
if (nativeCode == GetJniDlsymLookupStub()) {
#if defined(\_\_arm\_\_) || defined(\_\_aarch64\_\_)
nativeCode = artFindNativeMethod();
#else
nativeCode = artFindNativeMethod(self);
#endif
if (nativeCode == nullptr) {
DCHECK(self->IsExceptionPending()); // There should be an exception pending now.
DCHECK(self->IsExceptionPending()); // There should be an exception pending now.
// End JNI, as the assembly will move to deliver the exception.
jobject lock = called->IsSynchronized() ? visitor.GetFirstHandleScopeJObject() : nullptr;
if (shorty[0] == 'L') {
artQuickGenericJniEndJNIRef(self, cookie, nullptr, lock);
} else {
artQuickGenericJniEndJNINonRef(self, cookie, lock);
}
return GetTwoWordFailureValue();
}
// Note that the native code pointer will be automatically set by artFindNativeMethod().
}
// Return native code addr(lo) and bottom of alloca address(hi).
return GetTwoWordSuccessValue(reinterpret\_cast<uintptr\_t>(visitor.GetBottomOfUsedArea()),
reinterpret\_cast<uintptr\_t>(nativeCode));
}
```

### 总结

笔记分析的还是很粗糙的，发出来是希望可以抛砖引玉，可以在论坛看到更多frida的文章，如有错误，恳请大佬指出。

## frida-java简介

frida的JavaScript API按功能划分了许多模块，`frida-java`具体实现了其中的`Java`模块，提供了`Java Runtime`相关的API。我们知道`JNI`连接了native世界和java世界，而`frida-java`相当于实现了一个js世界到java世界的单向通道。利用`frida-java`，我们可以使用js代码实现：调用java方法、创建java对象、对java函数进行hook等操作。

`frida-java`源码结构如下：

- `index.js`: 封装了JavaScript API中`Java`模块的api
- `lib`
  - `android.js`: 封装了一些Android虚拟机的api
  - `api.js`
  - `class-factory.js`: java类的一些处理函数
  - `env.js`: js中实现了`JNIEnv`的一个代理
  - `mkdex.js`: dex文件的一些处理函数
  - `result.js`: 检测jni调用是否产生异常
  - `vm.js`: js中实现了`JavaVM`的一个代理

接下来通过下面两个问题来分析`frida-java`源码：

1. `frida-java`是如何连通到java世界？
2. `frida-java`如何实现java方法hook？

## frida-java是如何连通到java世界？

总的来说`frida-java`通过两步实现js世界到java世界的单向通道，首先利用`frida-gum`提供的js接口操作native世界，然后再基于jni连通到java世界。

![](https://bbs.kanxue.com/upload/attach/201806/657395_KUK5RJUFFQG7YWN.jpg)

主要步骤如下：

1. 连通native世界，获得一些虚拟机的接口，如`JNI_GetCreatedJavaVMs`
2. 获得`JavaVM`
3. 获得`JNIEnv`
4. 获得java类的class引用
5. 操作该java类，如创建对象，调用方法等。

### 连通native世界

利用`frida-gum`中提供的`Module`、`Memory`、`NativeFunction`等模块，可以实现查找、调用、hook导出函数；读写、分配内存等操作。如下面例子所示，可以在`js`代码中调用`native`函数。
