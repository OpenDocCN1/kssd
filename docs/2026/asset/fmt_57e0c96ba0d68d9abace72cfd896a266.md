
`static jobject Method_invoke(JNIEnv * env, jobject javaMethod, jobject javaReceiver, jobjectArray javaArgs) {`
`ScopedFastNativeObjectAccess soa(env);`
`return InvokeMethod<kRuntimePointerSize>(soa, javaMethod, javaReceiver, javaArgs);`
`}`

`InvokeMethod` 定义在 `art/runtime/reflection.cc`，其实现的核心代码如下:

```
template <PointerSize kPointerSize>
jobject InvokeMethod(const ScopedObjectAccessAlreadyRunnable& soa, jobject javaMethod,
jobject javaReceiver, jobject javaArgs, size_t num_frames) {
ObjPtr<mirror::Executable> executable = soa.Decode<mirror::Executable>(javaMethod);
const bool accessible = executable->IsAccessible();
ArtMethod * m = executable->GetArtMethod();

if (UNLIKELY(!declaring_class->IsVisiblyInitialized())) {
Thread * self = soa.Self();
Runtime::Current()->GetClassLinker()->EnsureInitialized(
self, h_class,
/*can_init_fields=*/ true,
/*can_init_parents=*/ true)
}

if (!m->IsStatic()) {
if (declaring_class->IsStringClass() && m->IsConstructor()) {
m = WellKnownClasses::StringInitToStringFactory(m);
} else {
m = receiver->GetClass()->FindVirtualMethodForVirtualOrInterface(m, kPointerSize);
}
}

if (!accessible && !VerifyAccess(/*...*/)) {
ThrowIllegalAccessException(
StringPrintf("Class %s cannot access %s method %s of class %s", ...));
}

InvokeMethodImpl(soa, m, np_method, receiver, objects, &shorty, &result);
}
```

上面省略了许多细节，主要是做了一些调用前的检查和预处理工作，流程可以概况为:

1. 判断方法所属的类是否已经初始化过，如果没有则进行初始化；
2. 将 `String.<init>` 构造函数调用替换为对应的工厂 `StringFactory` 方法调用；
3. 如果是虚函数调用，替换为运行时实际的函数；
4. 判断方法是否可以访问，如果不能访问则抛出异常；
5. 调用函数；

值得注意的是，`jobject` 类型的 `javaMethod` 可以转换为 `ArtMethod` 指针，该结构体是 ART 虚拟机中对于具体方法的描述。之后经过一系列调用:

- `InvokeMethodImpl`
- `InvokeWithArgArray`
- `method->Invoke()`

最终进入 `ArtMethod::Invoke` 函数，还是只看核心代码:

```
void ArtMethod::Invoke(Thread * self, uint32_t * args, uint32_t args_size, JValue * result,
const char * shorty) {
Runtime * runtime = Runtime::Current();
if (UNLIKELY(!runtime->IsStarted() ||
(self->IsForceInterpreter() && !IsNative() && !IsProxyMethod() && IsInvokable()))) {
art::interpreter::EnterInterpreterFromInvoke(...);
} else {
bool have_quick_code = GetEntryPointFromQuickCompiledCode() != nullptr;
if (LIKELY(have_quick_code)) {
if (!IsStatic()) {
(*art_quick_invoke_stub)(this, args, args_size, self, result, shorty);
} else {
(*art_quick_invoke_static_stub)(this, args, args_size, self, result, shorty);
}
} else {
LOG(INFO) << "Not invoking '" << PrettyMethod() << "' code=null";
}
}
self->PopManagedStackFragment(fragment);
}
```

ART 对于 Java 方法实现了两种执行模式，一种是像 Dalvik 虚拟机一样解释执行字节码，姑且称为解释模式；另一种是快速模式，即直接调用通过 OAT 编译后的本地代码。

> 在 ART 早期指定本地代码还细分为 Portable 和 Quick 两种模式，但由于对极致速度的追求以及随着 Quick 模式的不断优化，Portable 也逐渐退出了历史舞台。

阅读上述代码可以得知，当 ART 运行时尚未启动或者指定强制使用解释执行时，虚拟机执行函数使用的是解释模式，ART 可以在启动时指定 `-Xint` 参数强制使用解释执行，但即便指定了使用解释执行模式，还是有一些情况无法使用解释执行，比如:

1. 当所执行的方法是 Native 方法时，这时只有二进制代码，不存在字节码，自然无法解释执行；
2. 当所执行的方法无法调用，比如 `access_flag` 判定无法访问或者当前方法是抽象方法时；
3. 当所执行的方式是代理方法时，ART 对于代理方法有单独的本地调用方式；

### 解释执行

解释执行的入口是 `art::interpreter::EnterInterpreterFromInvoke`，该函数定义在 `art/runtime/interpreter/interpreter.cc`，关键代码如下:

```
void EnterInterpreterFromInvoke(Thread * self,
ArtMethod * method,
ObjPtr<mirror::Object> receiver,
uint32_t * args,
JValue * result,
bool stay_in_interpreter) {
CodeItemDataAccessor accessor(method->DexInstructionData());
if (accessor.HasCodeItem()) {
num_regs = accessor.RegistersSize();
num_ins = accessor.InsSize();
}
// 初始化栈帧 ......
if (LIKELY(!method->IsNative())) {
JValue r = Execute(self, accessor, *shadow_frame, JValue(), stay_in_interpreter);
if (result != nullptr) {
*result = r;
}
}
}
```

其中的 `CodeItem` 就是 DEX 文件中对应方法的字节码，还是老样子，直接看简化的调用链路:

| method | file |
| --- | --- |
| Execute | `art/runtime/interpreter/interpreter.cc` |
| ExecuteSwitch | ... |
| ExecuteSwitchImpl | `art/runtime/interpreter/interpreter_switch_impl.h` |
| ExecuteSwitchImplAsm | ... |
| ExecuteSwitchImplAsm | `art/runtime/arch/arm64/quick_entrypoints_arm64.S` |
| ExecuteSwitchImplCpp | `art/runtime/interpreter/interpreter_switch_impl-inl.h` |

`ExecuteSwitchImplAsm` 为了速度直接使用汇编实现，在 ARM64 平台中的定义如下:

```
// Wrap ExecuteSwitchImpl in assembly method which specifies DEX PC for unwinding.
// Argument 0: x0: The context pointer for ExecuteSwitchImpl.
// Argument 1: x1: Pointer to the templated ExecuteSwitchImpl to call.
// Argument 2: x2: The value of DEX PC (memory address of the methods bytecode).
ENTRY ExecuteSwitchImplAsm
SAVE_TWO_REGS_INCREASE_FRAME x19, xLR, 16
mov x19, x2
// x19 = DEX PC
CFI_DEFINE_DEX_PC_WITH_OFFSET(0 /* x0 */, 19 /* x19 */, 0)
blr x1
// Call the wrapped method.
RESTORE_TWO_REGS_DECREASE_FRAME x19, xLR, 16
ret
END ExecuteSwitchImplAsm
```

本质上是调用保存在 x1 寄存器的第二个参数，调用处的代码片段如下:

```
template<bool do_access_check, bool transaction_active>
ALWAYS_INLINE JValue ExecuteSwitchImpl() {
/...
void * impl = reinterpret_cast<void *>(&ExecuteSwitchImplCpp<do_access_check, transaction_active>);
const uint16_t * dex_pc = ctx.accessor.Insns();
ExecuteSwitchImplAsm(&ctx, impl, dex_pc);
}
```

即调用了 `ExecuteSwitchImplCpp`，在该函数中，可以看见典型的解释执行代码:

```
template<bool do_access_check, bool transaction_active>
void ExecuteSwitchImplCpp(SwitchImplContext * ctx) {
Thread * self = ctx->self;
const CodeItemDataAccessor& accessor = ctx->accessor;
ShadowFrame& shadow_frame = ctx->shadow_frame;
self->VerifyStack();

uint32_t dex_pc = shadow_frame.GetDexPC();
const auto * const instrumentation = Runtime::Current()->GetInstrumentation();
const uint16_t * const insns = accessor.Insns();
const Instruction * next = Instruction::At(insns + dex_pc);

while (true) {
const Instruction * const inst = next;
dex_pc = inst->GetDexPc(insns);
shadow_frame.SetDexPC(dex_pc);
TraceExecution(shadow_frame, inst, dex_pc);
uint16_t inst_data = inst->Fetch16(0);
// 一条指令 4 字节

if (InstructionHandler(...).Preamble()) {
switch (inst->Opcode(inst_data)) {
case xxx: ...;
case yyy: ...;
...
}
}
}
}
```

在当前版本中 (Android 12)，实际上是通过宏展开去定义了所有 `op_code` 的处理分支，不同版本实现都略有不同，但解释执行的核心思路从 Android 2.x 版本到现在都是一致的，因为字节码的定义并没有太多改变。

### 快速执行

再回到 `ArtMethod` 真正调用之前，如果不使用解释模式执行，则通过 `art_quick_invoke_stub` 去调用。stub 是一小段中间代码，用于跳转到实际的 native 执行，该符号使用汇编实现，在 ARM64 中的定义在 `art/runtime/arch/arm64/quick_entrypoints_arm64.S`，核心代码如下:

```
```
