
```cpp
jsi::Value HermesRuntimeImpl::evaluateJavaScript(
const std::shared_ptr<const jsi::Buffer> &buffer,
const std::string &sourceURL) {
return evaluateJavaScriptWithSourceMap(buffer, nullptr, sourceURL);
}
jsi::Value HermesRuntime::evaluateJavaScriptWithSourceMap(
const std::shared_ptr<const jsi::Buffer> &buffer,
const std::shared_ptr<const jsi::Buffer> &sourceMapBuf,
const std::string &sourceURL) {
return impl(this)->evaluatePreparedJavaScript(
impl(this)->prepareJavaScriptWithSourceMap(
buffer, sourceMapBuf, sourceURL));
}
```

`evaluateJavaScriptWithSourceMap`函数中调用了`prepareJavaScriptWithSourceMap`函数，继续调用`isHermesBytecode`函数。

```cpp
HermesRuntimeImpl::prepareJavaScriptWithSourceMap(
const std::shared_ptr<const jsi::Buffer> &jsiBuffer,
const std::shared_ptr<const jsi::Buffer> &sourceMapBuf,
std::string sourceURL) {
std::pair<std::unique_ptr<hbc::BCProvider>, std::string> bcErr{};
auto buffer = std::make_unique<BufferAdapter>(std::move(jsiBuffer));
vm::RuntimeModuleFlags runtimeFlags{};
runtimeFlags.persistent = true;
bool isBytecode = isHermesBytecode(buffer->data(), buffer->size());
```

`isHermesBytecode`用来判断是否是Hermes的Bytecode。

```cpp
bool HermesRuntime::isHermesBytecode(const uint8_t *data, size_t len) {
return hbc::BCProviderFromBuffer::isBytecodeStream(
llvh::ArrayRef<uint8_t>(data, len));
}
static bool isBytecodeStream(llvh::ArrayRef<uint8_t> aref) {
const auto *header =
reinterpret_cast<const hbc::BytecodeFileHeader *>(aref.data());
return (
aref.size() >= sizeof(hbc::BytecodeFileHeader) &&
header->magic == hbc::MAGIC);
}
const static uint64_t MAGIC = 0x1F1903C103BC1FC6;
```

`MAGIC`和`index.android.bunddle`的前8个字节是一直的，至此已经知道`index.android.bundle`并不是被应用gl加密了，而是被Hermes编译成了`bytecode`。

![](upload/attach/202207/548459\_BEZBPEGJ4YFAYV2.png)

```cpp
BytecodeFileHeader(
uint64_t magic, //0x1F1903C103BC1FC6
uint32_t version, //0x54 表示是84版本，commit 0bac657c61ac47a3a9537d982921f5f6c9630d41 (HEAD, tag: v0.8.1)
......
}
hermes/include/hermes/BCGen/HBC/BytecodeVersion.h路径中有定义BYTECODE_VERSION
const static uint32_t BYTECODE_VERSION = 84;
```

#### 3.3 反编译Hermes bytecode与回编译

前面的分析过程，已经知道了`index.android.bundle`是Hermes的`bytecode`，没办法直接查看源码，需要借助其他工具进行反编译。

Hermes官方提供了`hbcdump`工具，可以进行反编译，但使用起来比较麻烦，有另一个开源工具`hbctool`可以对`hermesbytecode`进行反编译与回编译。

<mark class="encrypted">ba0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6T1L8$3&6Y4N6s2u0G2M7q4)9J5c8X3S2T1j5%4c8G2L8$3H3`. 提供了59，62，74，76版本的反编译。

<mark class="encrypted">e06K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6F1K9h3!0K6k6h3N6S2i4K6u0r3K9r3u0U0N6r3!0G2L8q4)9J5c8Y4c8J5k6h3g2Q4x3V1k6V1M7X3q4X3N6q4)9J5c8X3S2T1j5#2)9J5k6s2j5^5y4l9`.` 对84版本的Hermes进行补充。

安装可以反编译84版本的`hbctool`：

```bash
pip install https://github.com/niosega/hbctool/archive/ac6fabb69a7229ed9764997d153d4f703d1381aa.zip
```

反编译：

```bash
hbctool disasm index.android.bundle bundle
```

回编译：

```bash
hbctool asm bundle index.android.bundle.re
```

`bytecode`反编译出来的代码如下：搜索两处能md5相关的代码。

![](upload/attach/202207/548459\_NZ3ZQGDRCTD3QEY.png)

![](upload/attach/202207/548459\_NEDX7FGKS767SXG.png)

并不需要深入`bytecode`指令详情，了解几个简单指令即可。

只需要写一个打印日志的函数，变成`bytecode`，反编译后替换掉这两个函数，就能看到这两个md5函数的输入参数。

#### 3.4 js代码编译成Hermes bytecode

先编译Hermes源码，会得到`./build/bin/hermes`工具。

`test.js`：

```js
function hook_log(a, b) {
window.nativeLoggingHook(a + " " + b, 1);
return " ";
}
function test() {
hook_log("333", "555");
}
```

写了一些简单的js代码，把`test.js`编译成`bytecode`：

```bash
./build/bin/hermes -emit-binary -out test.hbc test.js
```

010editor打开`test.hbc`可以看到`MAGIC`。

![](upload/attach/202207/548459\_9HYEVHHHP9F36A4.png)

反编译`test.hbc`：

```bash
hbctool disasm test.hbc testdir
```

得到：

```text
Function<hook_log>1(3 params, 14 registers, 0 symbols):
GetGlobalObject Reg8:0 ; 先获取全局变量 Reg8:0
TryGetById Reg8:4, Reg8:0, UInt8:1, UInt16:7 ; 通过window字符串从Reg8:0全局变量中拿到window变量，存放在Reg8:4
; Oper[3]: String(7) 'window'
GetByIdShort Reg8:3, Reg8:4, UInt8:2, UInt8:5 ; 从windows变量中拿nativeLoggingHook，存放在Reg8:3
; Oper[3]: String(5) 'nativeLoggingHook'
LoadParam Reg8:1, UInt8:1
LoadConstString Reg8:0, UInt16:0
; Oper[1]: String(0) ' '
Add Reg8:2, Reg8:1, Reg8:0
LoadParam Reg8:1, UInt8:2
Add Reg8:2, Reg8:2, Reg8:1
LoadConstUInt8 Reg8:1, UInt8:1
Call3 Reg8:1, Reg8:3, Reg8:4, Reg8:2, Reg8:1 ;调用window.nativeLoggingHook函数
Ret Reg8:0
EndFunction
```

#### 3.5 替换bytecode，得到signcode算法

把反编译出来的`bytecode`替换到md5函数中。

需要对字符串的id进行修改，字符串对应`hbctool`反编译出来的`string.json`中的字符串id。

```json
{
"id": 253,
"isUTF16": false,
"value": "window"
},
{
"id": 33087,
"isUTF16": false,
"value": "nativeLoggingHook"
},
{
"id": 18880,
"isUTF16": false,
"value": " "
},
```

```text
Function<md5>13508(3 params, 14 registers, 0 symbols):
GetGlobalObject Reg8:0
TryGetById Reg8:4, Reg8:0, UInt8:1, UInt16:253 ; 修改字符串id UInt16:7-> UInt16:253
; Oper[3]: String(254) 'window'
GetById Reg8:3, Reg8:4, UInt8:2, UInt16:33087 ;修改字符串id UInt8:5 -> UInt16:33087, 从UInt8改成UInt16, 需要把GetByIdShort修改成GetById指令
; Oper[3]: String(4397) 'nativeLoggingHook'
LoadParam Reg8:1, UInt8:1
LoadConstString Reg8:0, UInt16:18880 ; 修改字符串id UInt16:0 -> UInt16:18880
; Oper[1]: String(674) ' '
Add Reg8:2, Reg8:1, Reg8:0
LoadParam Reg8:1, UInt8:2
Add Reg8:2, Reg8:2, Reg8:1
LoadConstUInt8 Reg8:1, UInt8:1
Call3 Reg8:1, Reg8:3, Reg8:4, Reg8:2, Reg8:1
Ret Reg8:0
EndFunction
```

再次回编译，得到`index.android.bundle.re`：

```bash
hbctool asm bundle index.android.bundle.re
md5sum index.android.bundle.re
7eba6c20134268a82e5a0af948ca550b index.android.bundle.re
```

现在替换app目录下的`index.android.bundle`：

```bash
adb push index.android.bundle.re /data/local/tmp/index.android.bundle.re
adb shell
su
cd /data/data/com.pcakge.name/files/RN/rnbundle/
cp index.android.bundle index.android.bundle.bak # 对原来的文件备份
cp /data/local/tmp/index.android.bundle.re index.android.bundle
```

结束app，运行`adb logcat "*:S ReactNative:V ReactNativeJS:V"`，再次运行app就可以得到md5函数的输入参数：

```text
ReactNativeJS: XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXAPPKEYGL_RN_D02F48E720CACLIENTANDROIDDEVICEIDBBC6AD7E-1561-4702-8C1E-3AAC682B4D8BENCRYPT1PASSWORD6666666666TIMESTAMPUINFO666666666VERSION3.1.0XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX undefined
XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX脱敏
```

算法分析完成。

## 参考链接

<mark class="encrypted">4ffK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6X3j5h3y4W2j5X3!0G2K9#2)9J5c8Y4u0W2j5h3y4@1i4K6u0V1L8X3q4@1K9i4k6W2i4K6u0W2k6$3W2@1</mark>

<mark class="encrypted">302K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6D9j5i4y4@1K9h3&6Y4i4K6u0V1P5h3q4F1k6#2)9J5c8X3k6J5K9h3c8S2i4K6g2X3K9r3!0G2K9#2)9#2k6X3I4A6j5X3q4J5N6l9`.`</mark>

<mark class="encrypted">174K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6X3j5h3y4W2j5X3!0G2K9#2)9J5c8X3S2W2M7X3#2W2M7H3`.`</mark>

<mark class="encrypted">a4eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6T1L8$3&6Y4N6s2u0G2M7q4)9J5c8X3S2T1j5%4c8G2L8$3H3`</mark>

<mark class="encrypted">7ebK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6F1K9h3!0K6k6h3N6S2i4K6u0r3K9r3u0U0N6r3!0G2L8q4)9J5c8Y4c8J5k6h3g2Q4x3V1k6V1M7X3q4X3N6q4)9J5c8X3S2T1j5#2)9J5k6s2j5^5y4l9`.`</mark>

## 0. 前言

Android应用gl，使用了加固i，老版本的应用gl是js源码，新版本更新后，刚开始以为是加密了源码，分析后才知道是使用了Hermes优化引擎。

## 1. Hermes简介

Hermes是Facebook为React Native开源的一款javascript优化引擎。

原来的`index.android.bundle`是javascript的，被Hermes优化后变成了`Bytecode`。

Hermes优化的优化效果如下：

优化前：

![](https://bbs.kanxue.com/upload/attach/202207/548459_CATSKCPAMETAHCR.png)

优化后：

![](https://bbs.kanxue.com/upload/attach/202207/548459_XP3HYZCXFSM3TMA.png)

## 2. 前期准备

#### 2.1 绕过root检测

应用gl使用了加固i，在被root的手机上运行，闪退。

绕过方法也很简单，在magisk的设置中，开启遵守排除列表，然后在配置排除列表中选择应用gl。

![](https://bbs.kanxue.com/upload/attach/202207/548459_DT5A4CRSRU75Z4K.png)

应用被添加到排除列表后，将没办法使用lspoed，只能使用frida。

#### 2.2 绕过ssl pinning

|  |  |

| --- | --- |
