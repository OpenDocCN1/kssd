
```diff
diff --git a/BUILD.bazel b/BUILD.bazel
index 3d37f45cede..584701ef478 100644
--- a/BUILD.bazel
+++ b/BUILD.bazel
@@ -1921,6 +1921,8 @@ filegroup(
"src/sandbox/external-pointer.h",
"src/sandbox/external-pointer-table.cc",
"src/sandbox/external-pointer-table.h",
+ "src/sandbox/testing.cc",
+ "src/sandbox/testing.h",
"src/sandbox/sandbox.cc",
"src/sandbox/sandbox.h",
"src/sandbox/sandboxed-pointer-inl.h",
diff --git a/BUILD.gn b/BUILD.gn
index 7ef8c1f2e06..d0538db38c3 100644
--- a/BUILD.gn
+++ b/BUILD.gn
@@ -304,18 +304,18 @@ declare\_args() {
# Enable the experimental V8 sandbox.
# Sets -DV8\_SANDBOX.
- v8\_enable\_sandbox = false
+ v8\_enable\_sandbox = true
# Enable external pointer sandboxing. Requires v8\_enable\_sandbox.
# Sets -DV8\_SANDBOXED\_EXTERNAL\_POINRTERS.
- v8\_enable\_sandboxed\_external\_pointers = false
+ v8\_enable\_sandboxed\_external\_pointers = true
# Enable sandboxed pointers. Requires v8\_enable\_sandbox.
# Sets -DV8\_SANDBOXED\_POINTERS.
- v8\_enable\_sandboxed\_pointers = false
+ v8\_enable\_sandboxed\_pointers = true
# Enable all available sandbox features. Implies v8\_enable\_sandbox.
- v8\_enable\_sandbox\_future = false
+ v8\_enable\_sandbox\_future = true
# Experimental feature for collecting per-class zone memory stats.
# Requires use\_rtti = true
@@ -3332,6 +3332,7 @@ v8\_header\_set("v8\_internal\_headers") {
"src/sandbox/sandbox.h",
"src/sandbox/sandboxed-pointer-inl.h",
"src/sandbox/sandboxed-pointer.h",
+ "src/sandbox/testing.h",
"src/snapshot/code-serializer.h",
"src/snapshot/context-deserializer.h",
"src/snapshot/context-serializer.h",
@@ -4353,6 +4354,7 @@ v8\_source\_set("v8\_base\_without\_compiler") {
"src/runtime/runtime.cc",
"src/sandbox/external-pointer-table.cc",
"src/sandbox/sandbox.cc",
+ "src/sandbox/testing.cc",
"src/snapshot/code-serializer.cc",
"src/snapshot/context-deserializer.cc",
"src/snapshot/context-serializer.cc",
diff --git a/src/d8/d8.cc b/src/d8/d8.cc
index 050cbdc78df..061379666a8 100644
--- a/src/d8/d8.cc
+++ b/src/d8/d8.cc
@@ -2860,7 +2860,7 @@ Local<FunctionTemplate> Shell::CreateNodeTemplates(Isolate\* isolate) {
Local<ObjectTemplate> Shell::CreateGlobalTemplate(Isolate\* isolate) {
Local<ObjectTemplate> global\_template = ObjectTemplate::New(isolate);
- global\_template->Set(Symbol::GetToStringTag(isolate),
+/\* global\_template->Set(Symbol::GetToStringTag(isolate),
String::NewFromUtf8Literal(isolate, "global"));
global\_template->Set(isolate, "version",
FunctionTemplate::New(isolate, Version));
@@ -2877,13 +2877,13 @@ Local<ObjectTemplate> Shell::CreateGlobalTemplate(Isolate\* isolate) {
global\_template->Set(isolate, "readline",
FunctionTemplate::New(isolate, ReadLine));
global\_template->Set(isolate, "load",
- FunctionTemplate::New(isolate, ExecuteFile));
+ FunctionTemplate::New(isolate, ExecuteFile));\*/
global\_template->Set(isolate, "setTimeout",
FunctionTemplate::New(isolate, SetTimeout));
// Some Emscripten-generated code tries to call 'quit', which in turn would
// call C's exit(). This would lead to memory leaks, because there is no way
// we can terminate cleanly then, so we need a way to hide 'quit'.
- if (!options.omit\_quit) {
+/\* if (!options.omit\_quit) {
global\_template->Set(isolate, "quit", FunctionTemplate::New(isolate, Quit));
}
global\_template->Set(isolate, "testRunner",
@@ -2909,7 +2909,7 @@ Local<ObjectTemplate> Shell::CreateGlobalTemplate(Isolate\* isolate) {
if (i::FLAG\_expose\_async\_hooks) {
global\_template->Set(isolate, "async\_hooks",
Shell::CreateAsyncHookTemplate(isolate));
- }
+ }\*/
return global\_template;
}
diff --git a/src/init/bootstrapper.cc b/src/init/bootstrapper.cc
index 16015435073..ecd1fbb4116 100644
--- a/src/init/bootstrapper.cc
+++ b/src/init/bootstrapper.cc
@@ -24,6 +24,7 @@
#include "src/logging/runtime-call-stats-scope.h"
#include "src/objects/instance-type.h"
#include "src/objects/objects.h"
+#include "src/sandbox/testing.h"
#ifdef ENABLE\_VTUNE\_TRACEMARK
#include "src/extensions/vtunedomain-support-extension.h"
#endif // ENABLE\_VTUNE\_TRACEMARK
@@ -5694,6 +5695,10 @@ bool Genesis::InstallSpecialObjects(Isolate\* isolate,
}
#endif // V8\_ENABLE\_WEBASSEMBLY
+ if (GetProcessWideSandbox()->is\_initialized()) {
+ MemoryCorruptionApi::Install(isolate);
+ }
+
return true;
}
diff --git a/src/sandbox/testing.cc b/src/sandbox/testing.cc
new file mode 100644
index 00000000000..327fd33588d
--- /dev/null
+++ b/src/sandbox/testing.cc
@@ -0,0 +1,194 @@
+// Copyright 2022 the V8 project authors. All rights reserved.
+// Use of this source code is governed by a BSD-style license that can be
+// found in the LICENSE file.
+
+#include "src/sandbox/testing.h"
+
+#include "src/api/api-inl.h"
+#include "src/api/api-natives.h"
+#include "src/common/globals.h"
+#include "src/execution/isolate-inl.h"
+#include "src/heap/factory.h"
+#include "src/objects/backing-store.h"
+#include "src/objects/js-objects.h"
+#include "src/objects/templates.h"
+#include "src/sandbox/sandbox.h"
+
+namespace v8 {
+namespace internal {
+
+//#ifdef V8\_EXPOSE\_MEMORY\_CORRUPTION\_API
+
+namespace {
+
+// Sandbox.byteLength
+void SandboxGetByteLength(const v8::FunctionCallbackInfo<v8::Value>& args) {
+ v8::Isolate\* isolate = args.GetIsolate();
+ double sandbox\_size = GetProcessWideSandbox()->size();
+ args.GetReturnValue().Set(v8::Number::New(isolate, sandbox\_size));
+}
+
+// new Sandbox.MemoryView(args) -> Sandbox.MemoryView
+void SandboxMemoryView(const v8::FunctionCallbackInfo<v8::Value>& args) {
+ v8::Isolate\* isolate = args.GetIsolate();
+ Local<v8::Context> context = isolate->GetCurrentContext();
+
+ if (!args.IsConstructCall()) {
+ isolate->ThrowError("Sandbox.MemoryView must be invoked with 'new'");
+ return;
+ }
+
+ Local<v8::Integer> arg1, arg2;
+ if (!args[0]->ToInteger(context).ToLocal(&arg1) ||
+ !args[1]->ToInteger(context).ToLocal(&arg2)) {
+ isolate->ThrowError("Expects two number arguments (start offset and size)");
+ return;
+ }
+
+ Sandbox\* sandbox = GetProcessWideSandbox();
+ CHECK\_LE(sandbox->size(), kMaxSafeIntegerUint64);
+
+ uint64\_t offset = arg1->Value();
+ uint64\_t size = arg2->Value();
+ if (offset > sandbox->size() || size > sandbox->size() ||
+ (offset + size) > sandbox->size()) {
+ isolate->ThrowError(
+ "The MemoryView must be entirely contained within the sandbox");
+ return;
+ }
+
+ Factory\* factory = reinterpret\_cast<Isolate\*>(isolate)->factory();
+ std::unique\_ptr<BackingStore> memory = BackingStore::WrapAllocation(
+ reinterpret\_cast<void\*>(sandbox->base() + offset), size,
+ v8::BackingStore::EmptyDeleter, nullptr, SharedFlag::kNotShared);
+ if (!memory) {
+ isolate->ThrowError("Out of memory: MemoryView backing store");
+ return;
+ }
+ Handle<JSArrayBuffer> buffer = factory->NewJSArrayBuffer(std::move(memory));
+ args.GetReturnValue().Set(Utils::ToLocal(buffer));
+}
+
+// Sandbox.getAddressOf(object) -> Number
+void SandboxGetAddressOf(const v8::FunctionCallbackInfo<v8::Value>& args) {
+ v8::Isolate\* isolate = args.GetIsolate();
+
+ if (args.Length() == 0) {
+ isolate->ThrowError("First argument must be provided");
+ return;
+ }
+
+ Handle<Object> arg = Utils::OpenHandle(\*args[0]);
+ if (!arg->IsHeapObject()) {
+ isolate->ThrowError("First argument must be a HeapObject");
+ return;
+ }
+
+ // HeapObjects must be allocated inside the pointer compression cage so their
+ // address relative to the start of the sandbox can be obtained simply by
+ // taking the lowest 32 bits of the absolute address.
+ uint32\_t address = static\_cast<uint32\_t>(HeapObject::cast(\*arg).address());
+ args.GetReturnValue().Set(v8::Integer::NewFromUnsigned(isolate, address));
+}
+
+// Sandbox.getSizeOf(object) -> Number
+void SandboxGetSizeOf(const v8::FunctionCallbackInfo<v8::Value>& args) {
+ v8::Isolate\* isolate = args.GetIsolate();
+
+ if (args.Length() == 0) {
+ isolate->ThrowError("First argument must be provided");
+ return;
+ }
+
+ Handle<Object> arg = Utils::OpenHandle(\*args[0]);
+ if (!arg->IsHeapObject()) {
+ isolate->ThrowError("First argument must be a HeapObject");
+ return;
+ }
+
+ int size = HeapObject::cast(\*arg).Size();
+ args.GetReturnValue().Set(v8::Integer::New(isolate, size));
+}
+
+Handle<FunctionTemplateInfo> NewFunctionTemplate(
+ Isolate\* isolate, FunctionCallback func,
+ ConstructorBehavior constructor\_behavior) {
+ // Use the API functions here as they are more convenient to use.
+ v8::Isolate\* api\_isolate = reinterpret\_cast<v8::Isolate\*>(isolate);
+ Local<FunctionTemplate> function\_template =
+ FunctionTemplate::New(api\_isolate, func, {}, {}, 0, constructor\_behavior,
+ SideEffectType::kHasSideEffect);
+ return v8::Utils::OpenHandle(\*function\_template);
+}
+
+Handle<JSFunction> CreateFunc(Isolate\* isolate, FunctionCallback func,
+ Handle<String> name, bool is\_constructor) {
+ ConstructorBehavior constructor\_behavior = is\_constructor
+ ? ConstructorBehavior::kAllow
+ : ConstructorBehavior::kThrow;
+ Handle<FunctionTemplateInfo> function\_template =
+ NewFunctionTemplate(isolate, func, constructor\_behavior);
+ return ApiNatives::InstantiateFunction(function\_template, name)
+ .ToHandleChecked();
+}
+
+void InstallFunc(Isolate\* isolate, Handle<JSObject> holder,
+ FunctionCallback func, const char\* name, int num\_parameters,
+ bool is\_constructor) {
+ Factory\* factory = isolate->factory();
+ Handle<String> function\_name = factory->NewStringFromAsciiChecked(name);
+ Handle<JSFunction> function =
+ CreateFunc(isolate, func, function\_name, is\_constructor);
+ function->shared().set\_length(num\_parameters);
+ JSObject::AddProperty(isolate, holder, function\_name, function, NONE);
+}
+
+void InstallGetter(Isolate\* isolate, Handle<JSObject> object,
+ FunctionCallback func, const char\* name) {
+ Factory\* factory = isolate->factory();
+ Handle<String> property\_name = factory->NewStringFromAsciiChecked(name);
+ Handle<JSFunction> getter = CreateFunc(isolate, func, property\_name, false);
+ Handle<Object> setter = factory->null\_value();
+ JSObject::DefineAccessor(object, property\_name, getter, setter, FROZEN);
+}
+
+void InstallFunction(Isolate\* isolate, Handle<JSObject> holder,
+ FunctionCallback func, const char\* name,
+ int num\_parameters) {
+ InstallFunc(isolate, holder, func, name, num\_parameters, false);
+}
+
+void InstallConstructor(Isolate\* isolate, Handle<JSObject> holder,
+ FunctionCallback func, const char\* name,
+ int num\_parameters) {
+ InstallFunc(isolate, holder, func, name, num\_parameters, true);
+}
+
+} // namespace
+
+// static
+void MemoryCorruptionApi::Install(Isolate\* isolate) {
+ CHECK(GetProcessWideSandbox()->is\_initialized());
+
+ Factory\* factory = isolate->factory();
+
+ // Create the special Sandbox object that provides read/write access to the
+ // sandbox address space alongside other miscellaneous functionality.
+ Handle<JSObject> sandbox =
+ factory->NewJSObject(isolate->object\_function(), AllocationType::kOld);
+
+ InstallGetter(isolate, sandbox, SandboxGetByteLength, "byteLength");
+ InstallConstructor(isolate, sandbox, SandboxMemoryView, "MemoryView", 2);
+ InstallFunction(isolate, sandbox, SandboxGetAddressOf, "getAddressOf", 1);
+ InstallFunction(isolate, sandbox, SandboxGetSizeOf, "getSizeOf", 1);
+
+ // Install the Sandbox object as property on the global object.
+ Handle<JSGlobalObject> global = isolate->global\_object();
+ Handle<String> name = factory->NewStringFromAsciiChecked("Sandbox");
+ JSObject::AddProperty(isolate, global, name, sandbox, DONT\_ENUM);
+}
+
+//#endif // V8\_EXPOSE\_MEMORY\_CORRUPTION\_API
+
+} // namespace internal
+} // namespace v8
diff --git a/src/sandbox/testing.h b/src/sandbox/testing.h
new file mode 100644
index 00000000000..0c30397c3c5
--- /dev/null
+++ b/src/sandbox/testing.h
@@ -0,0 +1,28 @@
+// Copyright 2022 the V8 project authors. All rights reserved.
+// Use of this source code is governed by a BSD-style license that can be
+// found in the LICENSE file.
+
+#ifndef V8\_SANDBOX\_TESTING\_H\_
+#define V8\_SANDBOX\_TESTING\_H\_
+
+#include "src/common/globals.h"
+
+namespace v8 {
+namespace internal {
+
+//#ifdef V8\_EXPOSE\_MEMORY\_CORRUPTION\_API
+// A JavaScript API that emulates typical exploit primitives.
+//
+// This can be used for testing the sandbox, for example to write regression
+// tests for bugs in the sandbox or to develop fuzzers.
+class MemoryCorruptionApi {
+ public:
+ V8\_EXPORT\_PRIVATE static void Install(Isolate\* isolate);
+};
+
+//#endif // V8\_EXPOSE\_MEMORY\_CORRUPTION\_API
+
+} // namespace internal
+} // namespace v8
+
+#endif // V8\_SANDBOX\_TESTING\_H\_
```
