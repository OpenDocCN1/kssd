
这个方法提供了一个`[addr,addr+offset]`的内存任意读写的能力，但是范围限定在了沙箱内，从这段代码可以看出来`(offset > sandbox->size() || size > sandbox->size() || (offset + size) > sandbox->size())`

### Sandbox.MemoryView

```c++
// new Sandbox.MemoryView(args) -> Sandbox.MemoryView
void SandboxMemoryView(const v8::FunctionCallbackInfo<v8::Value>& args) {
v8::Isolate* isolate = args.GetIsolate();
Local<v8::Context> context = isolate->GetCurrentContext();
if (!args.IsConstructCall()) {
isolate->ThrowError("Sandbox.MemoryView must be invoked with 'new'");
return;
}
Local<v8::Integer> arg1, arg2;
if (!args[0]->ToInteger(context).ToLocal(&arg1) ||
!args[1]->ToInteger(context).ToLocal(&arg2)) {
isolate->ThrowError("Expects two number arguments (start offset and size)");
return;
}
Sandbox* sandbox = GetProcessWideSandbox();
CHECK_LE(sandbox->size(), kMaxSafeIntegerUint64);
uint64_t offset = arg1->Value();
uint64_t size = arg2->Value();
if (offset > sandbox->size() || size > sandbox->size() ||
(offset + size) > sandbox->size()) {
isolate->ThrowError(
"The MemoryView must be entirely contained within the sandbox");
return;
}
Factory* factory = reinterpret_cast<Isolate*>(isolate)->factory();
std::unique_ptr<BackingStore> memory = BackingStore::WrapAllocation(
reinterpret_cast<void*>(sandbox->base() + offset), size,
v8::BackingStore::EmptyDeleter, nullptr, SharedFlag::kNotShared);
if (!memory) {
isolate->ThrowError("Out of memory: MemoryView backing store");
return;
}
Handle<JSArrayBuffer> buffer = factory->NewJSArrayBuffer(std::move(memory));
args.GetReturnValue().Set(Utils::ToLocal(buffer));
}
```

### 获取对象地址

这个就是获取`obj`的地址

```c++
// Sandbox.getAddressOf(object) -> Number
void SandboxGetAddressOf(const v8::FunctionCallbackInfo<v8::Value>& args) {
v8::Isolate* isolate = args.GetIsolate();
if (args.Length() == 0) {
isolate->ThrowError("First argument must be provided");
return;
}
Handle<Object> arg = Utils::OpenHandle(*args[0]);
if (!arg->IsHeapObject()) {
isolate->ThrowError("First argument must be a HeapObject");
return;
}
// HeapObjects must be allocated inside the pointer compression cage so their
// address relative to the start of the sandbox can be obtained simply by
// taking the lowest 32 bits of the absolute address.
uint32_t address = static_cast<uint32_t>(HeapObject::cast(*arg).address());
args.GetReturnValue().Set(v8::Integer::NewFromUnsigned(isolate, address));
}
```

### 获取对象大小

下面是获取`obj`的`size`

```c++
// Sandbox.getSizeOf(object) -> Number
void SandboxGetSizeOf(const v8::FunctionCallbackInfo<v8::Value>& args) {
v8::Isolate* isolate = args.GetIsolate();
if (args.Length() == 0) {
isolate->ThrowError("First argument must be provided");
return;
}
Handle<Object> arg = Utils::OpenHandle(*args[0]);
if (!arg->IsHeapObject()) {
isolate->ThrowError("First argument must be a HeapObject");
return;
}
int size = HeapObject::cast(*arg).Size();
args.GetReturnValue().Set(v8::Integer::New(isolate, size));
}
```

完整的`patch`内容
