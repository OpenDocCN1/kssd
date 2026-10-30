
```go
func Debug(s string, args []string) {
    // runtime.LockOSThread()

    // open process
    cmd := exec.Command(s, args...)
    cmd.Stderr = os.Stderr
    cmd.SysProcAttr = &sys.SysProcAttr{Ptrace: true}
    if err := cmd.Start(); err != nil {
        log.Fatal(err)
    }

    pid := cmd.Process.Pid
    // get rip
    var regs sys.PtraceRegs
    if err := sys.PtraceGetRegs(pid, &regs); err != nil {
        log.Panic(err)
    }
    log.Printf("Value of rip: %x\n", regs.Rip)

    // set breakpoint1 before loop
    addr := uintptr(regs.Rip + bpOffset[0])
    data, err := SetBreakPoint(pid, addr)
    log.Printf("Set breakpoint at %x\n", addr)
    if err != nil {
        log.Panic(err)
    }

    // Continue
    if err := sys.PtraceCont(pid, 0); err != nil {
        log.Panic(err)
    }
    var ws sys.WaitStatus
    if _, err := sys.Wait4(pid, &ws, sys.WALL, nil); err != nil {
        log.Fatal(err)
    }

    log.Println("Hit bp1")

    // Cancel breakpoint
    if err := CancelBreakPoint(pid, addr, data); err != nil {
        log.Panic(err)
    }

    // need to do:
    // make snapshot

    // Set breakpoint before end
    bpAddr = uintptr(entrypoint + bpOffset[1])
    _, err = SetBreakPoint(pid, bpAddr)
    if err != nil {
        log.Panic(err)
    }

    bpAddr = uintptr(entrypoint + bpOffset[2])
    _, err = SetBreakPoint(pid, bpAddr)
    if err != nil {
        log.Panic(err)
    }

    // Continue
    if err = sys.PtraceCont(pid, 0); err != nil {
        log.Panic(err)
    }
    if _, err = sys.Wait4(pid, &ws, sys.WALL, nil); err != nil {
        log.Fatal(err)
    }

    log.Println("Hit bp2")

    // need to do
    // restore snapshot
}
```

输出为：

```text
└─$ go test -v
=== RUN   TestDebug
2022/01/10 01:43:54 Value of rip: 45cb60
2022/01/10 01:43:54 Set breakpoint at 48d8e0
2022/01/10 01:43:54 Hit bp1
2022/01/10 01:43:54 Hit bp2
--- PASS: TestDebug (0.00s)
PASS
ok      fuzz/debugger 0.003s
```

现在只需要替换掉测试的图片数据

### 4.3.7 相关位置确定

我不清楚原文是怎么确定的断点地址，从源码看断点是硬编码到代码中的，并且在启动子进程之前取消了ASLR。

我考虑先通过静态分析确定断点代码距离程序起始位置的偏移，然后在程序起始位置中断，获取`RIP`的地址，再加上偏移，就可以得到断点地址了。

上面已经确定了`RIP`的值是`0x45cb60`。

一开始我打算在循环开始之前设置断点，后来发现这样设置不太方便替换测试图像数据。

在IDA中找到函数`check`的调用位置，发现由于这个函数太短，已经被优化掉了：

![图片描述](https://bbs.kanxue.com/upload/attach/202201/600394_5G4GBKAP52TMTUN.png)

注意到在最终的`cmp`语句中，其中的一个比较对象`sil`来自于`main_checkVal`再加上偏移值，它对应的就是程序中的`checkVal` slice数组。

因此`bl`中保存的就是图像数据，向上溯源，寄存器`rdx`中保存的就应该是测试图像数据的起始地址。

如果继续向上溯源，`rdx ← [rsp+88h+var_30] ← rax ← ReadFile`。

因此最终第一个断点选在`ReadFile`函数调用结束之后，即`0x48D888`。

接下来选择第二个断点，因为程序有两个中断点，一个是测试没通过到达的`os.Exit()`函数，一个是全部测试通过到达的`main_vul`函数，因此我选择设置两个断点，分别在这两个函数调用之前。断点位置为：`0x48da0c`和`0x48da1b`

得到断点位置距离程序起始位置的偏移分别为`0x30d28`、`0x30eac`和`0x30ebb`。

在到达第一个断点后，检查一下寄存器`eax`指向的地址中保存的数据是不是测试图像：

```go
if err := sys.PtraceGetRegs(pid, &regs); err != nil {
    log.Panic(err)
}
dataAddr := regs.Rax
data := make([]byte, 8)
sys.PtracePeekData(pid, uintptr(dataAddr), data)
log.Printf("%x\n", data)
```

得到输出：

```text
2022/01/10 02:52:37 ffd8ffe000104a46
```

发现确实是测试图像的前八个字节。

---

然后需要确定可写内存的地址。使用文章中提到的方法，每次启动得到的内存块地址不同（不知道是不是ASLR的问题，因为我没有处理ASLR）。

我决定直接在程序中获取可读写内存块的地址:
