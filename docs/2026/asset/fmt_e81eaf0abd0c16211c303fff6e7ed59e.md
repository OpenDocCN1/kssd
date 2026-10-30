
`null@ubuntu:~/gowork/src/github.com/yaml$ go test -fuzz=Fuzz`
`OK: 45 passed`
`fuzz: elapsed: 0s, gathering baseline coverage: 0/1 completed`
`fuzz: elapsed: 0s, gathering baseline coverage: 1/1 completed, now fuzzing with 2 workers`
`fuzz: elapsed: 3s, execs: 61041 (20341/sec), new interesting: 129 (total: 130)`
`fuzz: elapsed: 6s, execs: 142873 (27284/sec), new interesting: 199 (total: 200)`
`fuzz: elapsed: 9s, execs: 212708 (23280/sec), new interesting: 239 (total: 240)`
`fuzz: elapsed: 12s, execs: 274044 (20439/sec), new interesting: 271 (total: 272)`
`fuzz: elapsed: 15s, execs: 320924 (15631/sec), new interesting: 298 (total: 299)`
`fuzz: elapsed: 18s, execs: 389403 (22820/sec), new interesting: 317 (total: 318)`
`fuzz: elapsed: 21s, execs: 423864 (11490/sec), new interesting: 334 (total: 335)`
`fuzz: elapsed: 24s, execs: 444293 (6809/sec), new interesting: 344 (total: 345)`
`......`

### 可选参数

- `-fuzztime`: fuzz 目标在退出前将执行的总时间或迭代次数，默认为无限期。
- `-fuzzminimizetime`：在每次最小化尝试期间执行模糊目标的时间或迭代次数，默认为 60 秒。
- `-parallel`：一次运行的模糊测试进程的数量，默认值 `$GOMAXPROCS`。目前，在 fuzzing 期间设置 `-cpu` 无效。

第一行表示在开始模糊测试之前收集了“基线覆盖率”。

- `elapsed`：自进程开始以来经过的时间量
- `execs`：针对模糊目标运行的输入总数（自最后一个日志行以来的平均 execs/sec）
- `new interesting`：已添加到生成的语料库中的“有趣”输入的总数（与整个语料库的总大小）

出现 `panic` 之后立马返回，停止 fuzz。

```
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

```
null@ubuntu:~/gowork/src/github.com/yaml$ go test -run=FuzzUnmarshal/b27ab1d6a899fb4f0607968de5b80e930b49a0e279bd4341c515ebf9bd7e7c78
--- FAIL: FuzzUnmarshal (0.00s)
--- FAIL: FuzzUnmarshal/b27ab1d6a899fb4f0607968de5b80e930b49a0e279bd4341c515ebf9bd7e7c78 (0.00s)
panic: internal error: attempted to parse unknown event (please report): none [recovered]
panic: internal error: attempted to parse unknown event (please report): none [recovered]
panic: internal error: attempted to parse unknown event (please report): none
```

查看崩溃字符

```
null@ubuntu:~/gowork/src/github.com/yaml$ cat testdata/fuzz/FuzzUnmarshal/b27ab1d6a899fb4f0607968de5b80e930b49a0e279bd4341c515ebf9bd7e7c78
go test fuzz v1
[]byte(":   \xf0")
```

其中第一行显示了模糊引擎文件的编码版本，下面的是构成语料库条目的值，即导致程序崩溃的输入。

再次运行 `go test` 单元测试时，会将该输入当作默认输入并引发崩溃：
