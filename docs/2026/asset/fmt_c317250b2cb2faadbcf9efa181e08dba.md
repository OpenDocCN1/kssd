
```java
// java\_lang\_reflect\_Method.cc
static jobject Method\_invoke(JNIEnv\* env, jobject javaMethod, jobject javaReceiver,
jobject javaArgs) {
ScopedFastNativeObjectAccess soa(env);
return InvokeMethod(soa, javaMethod, javaReceiver, javaArgs);
}
// reflection.cc
jobject InvokeMethod(const ScopedObjectAccessAlreadyRunnable& soa, jobject javaMethod,
jobject javaReceiver, jobject javaArgs, size\_t num\_frames) {
......
// 比较了参数个数
if (arg\_count != classes\_size) {
ThrowIllegalArgumentException(StringPrintf("Wrong number of arguments; expected %d, got %d",
classes\_size, arg\_count).c\_str());
return nullptr;
}
......
// 调用了这里
InvokeWithArgArray(soa, m, &arg\_array, &result, shorty);
......
}
// reflection.cc
static void InvokeWithArgArray(const ScopedObjectAccessAlreadyRunnable& soa,
ArtMethod\* method, ArgArray\* arg\_array, JValue\* result,
const char\* shorty)
REQUIRES\_SHARED(Locks::mutator\_lock\_) {
uint32\_t\* args = arg\_array->GetArray();
if (UNLIKELY(soa.Env()->check\_jni)) {
CheckMethodArguments(soa.Vm(), method->GetInterfaceMethodIfProxy(kRuntimePointerSize), args);
}
// 调用了这里
method->Invoke(soa.Self(), args, arg\_array->GetNumBytes(), result, shorty);
}
// art\_method.cc
void ArtMethod::Invoke(Thread\* self, uint32\_t\* args, uint32\_t args\_size, JValue\* result,
const char\* shorty) {
// 自己觉得这里dump可以了，后来老师指点了一下，还是需要深入的。
if (UNLIKELY(\_\_builtin\_frame\_address(0) < self->GetStackEnd())) {
ThrowStackOverflowError(self);
return;
}
......
if (!IsStatic()) {
(\*art\_quick\_invoke\_stub)(this, args, args\_size, self, result, shorty);
} else {
(\*art\_quick\_invoke\_static\_stub)(this, args, args\_size, self, result, shorty);
}
......
}
// https://www.jianshu.com/p/2ff1b63f686b
// https://www.cnblogs.com/lanrenxinxin/p/5207174.html
// 加固的app都是用Interpreter解释执行的，所以我们看看这个函数
void EnterInterpreterFromInvoke(Thread\* self,
ArtMethod\* method,
ObjPtr<mirror::Object> receiver,
uint32\_t\* args,
JValue\* result,
bool stay\_in\_interpreter) {
......
if (LIKELY(!method->IsNative())) {
JValue r = Execute(self, code\_item, \*shadow\_frame, JValue(), stay\_in\_interpreter);
if (result != nullptr) {
\*result = r;
}
......
}
// Execute函数
static inline JValue Execute(
Thread\* self,
const DexFile::CodeItem\* code\_item,
ShadowFrame& shadow\_frame,
JValue result\_register,
bool stay\_in\_interpreter = false) REQUIRES\_SHARED(Locks::mutator\_lock\_) {
DCHECK(!shadow\_frame.GetMethod()->IsAbstract());
DCHECK(!shadow\_frame.GetMethod()->IsNative());
if (LIKELY(shadow\_frame.GetDexPC() == 0)) {
// 看一下android的解释器：
// 在android8.1下面，解释器为两种：Switch解释器与汇编解释器：
// art/runtime/interpreter/interpreter.cc 默认配置的是汇编解释器，我们如果要对方法级别的保护做dump的话，需要在这里修改，然后dump opcode，如果是arm的话，修改起来比较难，我们可以修改为switch解释器，然后再尝试修改dump opcode。
enum InterpreterImplKind {
kSwitchImplKind, // Switch-based interpreter implementation.
kMterpImplKind // Assembly interpreter
};
- static constexpr InterpreterImplKind kInterpreterImplKind = kMterpImplKind;
+ static constexpr InterpreterImplKind kInterpreterImplKind = kSwitchImplKind;
// 然后看/art/runtime/interpreter/interpreter\_switch\_impl.cc这个文件
// 其中的参数self表示当前线程，shadow\_frame类似于java虚拟机中的stack frame,用于存储方法中的本地变量表、操作栈方法出口等信息。
template<bool do\_access\_check, bool transaction\_active>
JValue ExecuteSwitchImpl(Thread\* self, const DexFile::CodeItem\* code\_item,
ShadowFrame& shadow\_frame, JValue result\_register,
bool interpret\_one\_instruction) {
......
ArtMethod\* method = shadow\_frame.GetMethod();
//在这里dump opcode，抄一下老师的dumpDexFileByExecute方法。
jit::Jit\* jit = Runtime::Current()->GetJit();
......
}
```

分析到这里，基本就可以修改函数，炒一炒老师的代码，完成脱壳了，这部分就需要我修改源码了，慢慢改。

7. 附上我写的完整的frida脚本，没有拖出来完整的dex。
