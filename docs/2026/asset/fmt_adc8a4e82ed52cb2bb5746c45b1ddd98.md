
go/src/internal/fuzz/`mutator.go` 实现了对初始文件的变异功能，其核心代码如下：

```go
func (m *mutator) mutate(vals []any, maxBytes int) {
maxPerVal := maxBytes/len(vals) - 100
i := m.rand(len(vals))
switch v := vals[i].(type) {
case int:
vals[i] = int(m.mutateInt(int64(v), maxInt))
case int8:
vals[i] = int8(m.mutateInt(int64(v), math.MaxInt8))
case int16:
vals[i] = int16(m.mutateInt(int64(v), math.MaxInt16))
case int64:
vals[i] = m.mutateInt(v, maxInt)
case uint:
vals[i] = uint(m.mutateUInt(uint64(v), maxUint))
case uint16:
vals[i] = uint16(m.mutateUInt(uint64(v), math.MaxUint16))
case uint32:
vals[i] = uint32(m.mutateUInt(uint64(v), math.MaxUint32))
case uint64:
vals[i] = m.mutateUInt(uint64(v), maxUint)
case float32:
vals[i] = float32(m.mutateFloat(float64(v), math.MaxFloat32))
case float64:
vals[i] = m.mutateFloat(v, math.MaxFloat64)
case bool:
if m.rand(2) == 1 {
vals[i] = !v // 50% chance of flipping the bool
}
case rune: // int32
vals[i] = rune(m.mutateInt(int64(v), math.MaxInt32))
case byte: // uint8
vals[i] = byte(m.mutateUInt(uint64(v), math.MaxUint8))
case string:
...
case []byte:
...
default:
panic(fmt.Sprintf("type not supported for mutating: %T", vals[i]))
}
}
```

`gofuzz` 目前支持的类型有：`string`, `[]byte`、`int`, `int8`, `int16`, `int32`/`rune`, `int64`、`uint`, `uint8`/`byte`, `uint16`, `uint32`, `uint64`、`float32`, `float64`、`bool`。以`int`型为例，可以看到`go-native-fuzz`对该型的变异方式还比较单一，加上或减去一个随机数，并判断其变异后的返回值不能超高`int`支持的最大范围。

```go
func (m *mutator) mutateInt(v, maxValue int64) int64 {
var max int64
for {
max = 100
switch m.rand(2) {
case 0:
// Add a random number
if v >= maxValue {
continue
}
if v > 0 && maxValue-v < max {
// Don't let v exceed maxValue
max = maxValue - v
}
v += int64(1 + m.rand(int(max)))
return v
case 1:
// Subtract a random number
if v <= -maxValue {
continue
}
if v < 0 && maxValue+v < max {
// Don't let v drop below -maxValue
max = maxValue + v
}
v -= int64(1 + m.rand(int(max)))
return v
}
}
}
```

而对于`byte[]`型的值变异会比较多样化，比如增、删、改、插、交换、亦或等等。

```go
var byteSliceMutators = []byteSliceMutator{
byteSliceRemoveBytes,
byteSliceInsertRandomBytes,
byteSliceDuplicateBytes,
byteSliceOverwriteBytes,
byteSliceBitFlip,
byteSliceXORByte,
byteSliceSwapByte,
byteSliceArithmeticUint8,
byteSliceArithmeticUint16,
byteSliceArithmeticUint32,
byteSliceArithmeticUint64,
byteSliceOverwriteInterestingUint8,
byteSliceOverwriteInterestingUint16,
byteSliceOverwriteInterestingUint32,
byteSliceInsertConstantBytes,
byteSliceOverwriteConstantBytes,
byteSliceShuffleBytes,
byteSliceSwapBytes,
}
func (m *mutator) mutateBytes(ptrB *[]byte)
func (m *mutator) mutateFloat(v, maxValue float64) float64
func (m *mutator) mutateUInt(v, maxValue uint64) uint64
func (m *mutator) mutateInt(v, maxValue int64) int64
...
```

# 项目实战：yaml

项目地址：`430K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6Y4L8#2)9J5k6s2W2S2L8h3I4Q4x3V1k6&6j5h3#2D9`

该项目是作为 `juju` 项目的一部分在 `Canonical` 中开发的，基于著名的 `libyaml` C 库的纯 Go 端口，可以快速可靠地解析和生成 YAML 数据 ，使 Go 程序能够轻松地编码和解码 YAML 值 。

`yaml.Unmarshal()` 函数解码在字节切片中找到的第一个文档，并将解码后的值赋给输出值，十分适合作为我们测试的对象。

首先将项目下载的本地，然后 `git checkout` 切换到分支 `v3`，之后编写 `fuzzing` 函数，创建文件 `fuzz_test.go`。

```go
package yaml_test
import (
"testing"
"gopkg.in/yaml.v3"
)
func FuzzUnmarshal(f *testing.F){
f.Add( []byte{1})
f.Fuzz(func(t *testing.T,num []byte){
var v interface{}
_ = yaml.Unmarshal([]byte(num),&v)
})
}
```

`go test -fuzz =Fuzz` 开始 `fuzz`

```bash
null@ubuntu:~/gowork/src/github.com/yaml$ go test -fuzz=Fuzz
OK: 45 passed
fuzz: elapsed: 0s, gathering baseline coverage: 0/1 completed
fuzz: elapsed: 0s, gathering baseline coverage: 1/1 completed, now fuzzing with 2 workers
fuzz: elapsed: 3s, execs: 61041 (20341/sec), new interesting: 129 (total: 130)
fuzz: elapsed: 6s, execs: 142873 (27284/sec), new interesting: 199 (total: 200)
fuzz: elapsed: 9s, execs: 212708 (23280/sec), new interesting: 239 (total: 240)
fuzz: elapsed: 12s, execs: 274044 (20439/sec), new interesting: 271 (total: 272)
fuzz: elapsed: 15s, execs: 320924 (15631/sec), new interesting: 298 (total: 299)
fuzz: elapsed: 18s, execs: 389403 (22820/sec), new interesting: 317 (total: 318)
fuzz: elapsed: 21s, execs: 423864 (11490/sec), new interesting: 334 (total: 335)
fuzz: elapsed: 24s, execs: 444293 (6809/sec), new interesting: 344 (total: 345)
......
```

可选参数：

- `-fuzztime`：`fuzz` 目标在退出前将执行的总时间或迭代次数，默认为无限期。
- `-fuzzminimizetime`：在每次最小化尝试期间执行模糊目标的时间或迭代次数，默认为 60 秒。
- `-parallel`：一次运行的模糊测试进程的数量，默认值 `$GOMAXPROCS`。目前，在 `fuzzing` 期间设置 `-cpu` 无效。

第一行表示在开始模糊测试之前收集了“基线覆盖率”。

- `elapsed`：自进程开始以来经过的时间量
- `execs`：针对模糊目标运行的输入总数（自最后一个日志行以来的平均 `execs/sec`）
- `new interesting`：已添加到生成的语料库中的“有趣”输入的总数（与整个语料库的总大小）

出现 `panic` 之后立马返回，停止 `fuzz`。

```bash
--- FAIL: FuzzUnmarshal (362.91s)
--- FAIL: FuzzUnmarshal (0.00s)
testing.go:1349: panic: internal error: attempted to parse unknown event (please report): none
goroutine 1422926 [running]:
runtime/debug.Stack()
/usr/local/go/src/runtime/debug/stack.go:24 +0x90
testing.tRunner.func1()
/usr/local/go/src/testing/testing.go:1349 +0x1f2
panic({0x6be7a0, 0xc00d77d900})
/usr/local/go/src/runtime/panic.go:838 +0x207
gopkg.in/yaml%2ev3.handleErr(0xc0013a36e0)
/home/null/gowork/src/github.com/yaml/yaml.go:294 +0xc5
panic({0x6be7a0, 0xc00d77d900})
/usr/local/go/src/runtime/panic.go:838 +0x207
gopkg.in/yaml%2ev3.(*parser).parse(0xc00d80b800)
/home/null/gowork/src/github.com/yaml/decode.go:163 +0x2d9
gopkg.in/yaml%2ev3.unmarshal({0xc00d653b50, 0x5, 0x10}, {0x6b5ce0?, 0xc00d77d8f0}, 0x0?)
/home/null/gowork/src/github.com/yaml/yaml.go:161 +0x373
gopkg.in/yaml%2ev3.Unmarshal(...)
/home/null/gowork/src/github.com/yaml/yaml.go:89
gopkg.in/yaml%2ev3_test.FuzzUnmarshal.func1(0x0?, {0xc00d653b50, 0x5, 0x10})
/home/null/gowork/src/github.com/yaml/fuzz_test.go:11 +0x55
reflect.Value.call({0x6c14a0?, 0x718e40?, 0x13?}, {0x7041e6, 0x4}, {0xc00d8086c0, 0x2, 0x2?})
/usr/local/go/src/reflect/value.go:556 +0x845
reflect.Value.Call({0x6c14a0?, 0x718e40?, 0x514?}, {0xc00d8086c0, 0x2, 0x2})
/usr/local/go/src/reflect/value.go:339 +0xbf
testing.(*F).Fuzz.func1.1(0x0?)
/usr/local/go/src/testing/fuzz.go:337 +0x231
testing.tRunner(0xc00d7ff040, 0xc00d80e000)
/usr/local/go/src/testing/testing.go:1439 +0x102
created by testing.(*F).Fuzz.func1
/usr/local/go/src/testing/fuzz.go:324 +0x5b8
Failing input written to testdata/fuzz/FuzzUnmarshal/b27ab1d6a899fb4f0607968de5b80e930b49a0e279bd4341c515ebf9bd7e7c78
To re-run:
go test -run=FuzzUnmarshal/b27ab1d6a899fb4f0607968de5b80e930b49a0e279bd4341c515ebf9bd7e7c78
FAIL
exit status 1
```

出现崩溃后，模糊引擎会将导致崩溃的输入写入该测试的种子语料库中，而且此输入会当作执行 `go test` 命令时的默认输入。根据奔溃提示，执行 `go test run=FuzzUnmarshal/b27ab1d6a899fb4f0607968de5b80e930b49a0e279bd4341c515ebf9bd7e7c78` 复现。
