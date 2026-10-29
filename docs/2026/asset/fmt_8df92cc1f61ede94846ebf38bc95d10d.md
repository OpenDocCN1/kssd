
## `CatalystInstanceImpl::jniLoadScriptFromFile` 实现

```cpp
void CatalystInstanceImpl::jniLoadScriptFromFile(
const std::string &fileName,
const std::string &sourceURL,
bool loadSynchronously) {
auto reactInstance = instance_;
if (!reactInstance) {
return;
}
switch (getScriptTagFromFile(fileName.c_str())) { //根据文件头判断脚本类型，
case ScriptTag::MetroHBCBundle: {
std::unique_ptr<const JSBigFileString> script;
RecoverableError::runRethrowingAsRecoverable<std::system_error>(
[&fileName, &script]() {
script = JSBigFileString::fromPath(fileName);
});
const char *buffer = script->c_str();
uint32_t bufferLength = (uint32_t)script->size();
uint32_t offset = 8;
while (offset < bufferLength) {
uint32_t segment = offset + 4;
uint32_t moduleLength =
bufferLength < segment ? 0 : *((uint32_t *)buffer) + offset / 4);
reactInstance->loadScriptFromString(
std::make_unique<const JSBigStdString>(
std::string(buffer + segment, buffer + moduleLength + segment)),
sourceURL,
false);
offset += ((moduleLength + 3) & ~3) + 4;
}
break;
}
case ScriptTag::RAMBundle:
instance_->loadRAMBundleFromFile(fileName, sourceURL, loadSynchronously);
break;
case ScriptTag::String:
default: {
std::unique_ptr<const JSBigFileString> script;
RecoverableError::runRethrowingAsRecoverable<std::system_error>(
[&fileName, &script]() {
script = JSBigFileString::fromPath(fileName);
});
instance_->loadScriptFromString(
std::move(script), sourceURL, loadSynchronously);
}
}
}
```

`loadScriptFromString`

```cpp
void Instance::loadScriptFromString(
std::unique_ptr<const JSBigString> string,
std::string sourceURL,
bool loadSynchronously) {
SystraceSection s("Instance::loadScriptFromString", "sourceURL", sourceURL);
if (loadSynchronously) {
loadBundleSync(nullptr, std::move(string), std::move(sourceURL));
} else {
loadBundle(nullptr, std::move(string), std::move(sourceURL));
}
}
```

```javascript
let loadScriptFromString = base_libreactnativejni.add(0xA0BF8 + 1);
Interceptor.attach(loadScriptFromString, {
onEnter(args) {
console.log("loadScriptFromString:", (args[1]), ptr(args[2]).add(Process.pointerSize * 2).readPointer().readCString(), args[3])
}
})
```

`hook loadScriptFromString` 后，可以知道 `loadSynchronously` 为 0，继续分析 `loadBundle`

```cpp
void Instance::loadBundle(
std::unique_ptr<RAMBundleRegistry> bundleRegistry,
std::unique_ptr<const JSBigString> string,
std::string sourceURL) {
callback_->incrementPendingJSCalls();
SystraceSection s("Instance::loadBundle", "sourceURL", sourceURL);
nativeToJsBridge_->loadBundle(
std::move(bundleRegistry), std::move(string), std::move(sourceURL));
}
//Instance::loadBundle调用了nativeToJsBridge_->loadBundle，
void NativeToJsBridge::loadBundle(
std::unique_ptr<RAMBundleRegistry> bundleRegistry,
std::unique_ptr<const JSBigString> startupScript,
std::string startupScriptSourceURL) {
runOnExecutorQueue(
[this,
bundleRegistryWrap = folly::makeMoveWrapper(std::move(bundleRegistry)),
startupScript = folly::makeMoveWrapper(std::move(startupScript)),
startupScriptSourceURL =
std::move(startupScriptSourceURL)](JSExecutor *executor) mutable {
auto bundleRegistry = bundleRegistryWrap.move();
if (bundleRegistry) {
executor->setBundleRegistry(std::move(bundleRegistry));
}
try {
executor->loadBundle(
std::move(*startupScript), std::move(startupScriptSourceURL));
} catch (...) {
m_applicationScriptHasFailure = true;
throw;
}
});
}
//NativeToJsBridge::loadBundle里面调用了NativeToJsBridge::runOnExecutorQueue
```

`runOnExecutorQueue` 的参数是一个 `Lambda` 表达式，它的回调函数里面调用了 `loadBundle`，在源码中搜索 `loadBundle`，找到了很多处实现代码，这时候我们酒不知道 `loadBundle` 具体实现在哪里，继续写 `hook_loadBundle`

```javascript
function hook_loadBundle() {
//根据前面分析RN的一些so，可以发现大部分函数符号都还在so里面，那么我们通过DebugSymbol的findFunctionsMatching，找到所有loadBundle函数，并进行hook
DebugSymbol.findFunctionsMatching("*loadBundle*").map(addr => {
Interceptor.attach(addr, {
onEnter(args) {
console.log("loadBundle:", DebugSymbol.fromAddress(addr));
}
})
})
}
```

`hook` 结果如下：
`loadBundle: 0xbb70699d libreactnativejni.so!_ZN8facebook5react8Instance10loadBundleENSt6__ndk110unique_ptrINS0_17RAMBundleRegistryENS2_14default_deleteIS4_EEEENS3_IKNS0_11JSBigStringENS5_IS9_EEEENS2_12basic_stringIcNS2_11char_traitsIcEENS2_9allocatorIcEEEE`
`loadBundle: 0xbb70a845 libreactnativejni.so!_ZN8facebook5react16NativeToJsBridge10loadBundleENSt6__ndk110unique_ptrINS0_17RAMBundleRegistryENS2_14default_deleteIS4_EEEENS3_IKNS0_11JSBigStringENS5_IS9_EEEENS2_12basic_stringIcNS2_11char_traitsIcEENS2_9allocatorIcEEEE`
`loadBundle: 0xbb9a5ec5 libhermes-executor-release.so!_ZN8facebook5react11JSIExecutor10loadBundleENSt6__ndk110unique_ptrIKNS0_11JSBigStringENS2_14default_deleteIS5_EEEENS2_12basic_stringIcNS2_11char_traitsIcEENS2_9allocatorIcEEEE`

最终找到了 `libhermes-executor-release.so` 的 `facebook::react::JSIExecutor::loadBundle` 函数

## `facebook::react::JSIExecutor::loadBundle`

`facebook::react::JSIExecutor::loadBundle` 函数中调用了 `evaluateJavaScript`

```cpp
void JSIExecutor::loadBundle(
std::unique_ptr<const JSBigString> script,
std::string sourceURL) {
SystraceSection s("JSIExecutor::loadBundle");
bool hasLogger(ReactMarker::logTaggedMarker);
std::string scriptName = simpleBasename(sourceURL);
if (hasLogger) {
ReactMarker::logTaggedMarker(
ReactMarker::RUN_JS_BUNDLE_START, scriptName.c_str());
}
runtime_->evaluateJavaScript(
std::make_unique<BigStringBuffer>(std::move(script)), sourceURL);
flush();
if (hasLogger) {
ReactMarker::logTaggedMarker(
ReactMarker::RUN_JS_BUNDLE_STOP, scriptName.c_str());
}
}
```

`hook "*evaluateJavaScript*"` 

```javascript
function hook_evaluateJavaScript() {
DebugSymbol.findFunctionsMatching("*evaluateJavaScript*").map(addr => {
Interceptor.attach(addr, {
onEnter(args) {
console.log("evaluateJavaScript:", DebugSymbol.fromAddress(addr), "\r\n", print_native_stack("evaluateJavaScript", this));
}
})
})
}
```

并没有找到 `evaluateJavaScript` 函数，只找到了一个 `evaluateJavaScriptWithSourceMap` 函数
`evaluateJavaScript: 0xbb866775 libhermes.so!_ZN8facebook6hermes13HermesRuntime31evaluateJavaScriptWithSourceMapERKNSt6__ndk110shared_ptrIKNS_3jsi6BufferEEES9_RKNS2_12basic_stringIcNS2_11char_traitsIcEENS2_9allocatorIcEEEE`
`[evaluateJavaScript] called from:`
`0xbb867431 libhermes.so!0x11431`
`0xbb6605b9 libhermes-executor-common-release.so!0x1c5b9`
`0xbb727051 libhermes-executor-release.so!_ZN8facebook5react11JSIExecutor10loadBundleENSt6__ndk110unique_ptrIKNS0_11JSBigStringENS2_14default_deleteIS5_EEEENS2_12basic_stringIcNS2_11char_traitsIcEENS2_9allocatorIcEEEE+0x18c`
`0xbb4b94e7 libreactnativejni.so!0xa54e7`
`0xbb4ba133 libreactnativejni.so!_ZNKSt6__ndk18functionIFvPN8facebook5react10JSExecutorEEEclES4_+0x16`
`0xbb4a5ae1 libreactnativejni.so!0x91ae1`
`0xbb4961d3 libreactnativejni.so!_ZN8facebook3jni6detail13MethodWrapperIMNS_5react15JNativeRunnableEFvvEXadL_ZNS4_3runEvEES4_vJEE8dispatchENS0_9alias_refIPNS1_8JTypeForINS0_11HybridClassIS4_NS3_8RunnableEE8JavaPartESB_vE11_javaobjectEEE+0xe`
`0xbb49617f libreactnativejni.so!_ZN8facebook3jni6detail15FunctionWrapperIPFvNS0_9alias_refIPNS1_8JTypeForINS0_11HybridClassINS_5react15JNativeRunnableENS6_8RunnableEE8JavaPartES8_vE11_javaobjectEEEESD_vJEE4callEP7_JNIEnvP8_jobjectSG_+0x26`
`0xee6708df libart.so!art_quick_generic_jni_trampoline+0x2e`
`0x71915663 boot-framework.oat!0x54e663`

`evaluateJavaScriptWithSourceMap` 函数 在 React Native 源码中没有搜到，在另一个 hermes 项目中有所实现。

`evaluateJavaScript` 函数对应 so 的地址 `0xbb867431 libhermes.so!0x11431`
