
# 怎么拍

## 3. 需要相机：`lldb`

注：其实没看到`seelflower`的文章时，相机一开始选择的是`frida`去`dump`内存，但是研究了一段时间之后，发现`frida`有太多不足了，首先无法获取特殊状态寄存器的值，没有断点机制（`frida`运行时似乎是单线程），通过`mprotct`修改内存`Readable`容易发生`crash`。也有可能是我能力不够，`frida`脚本放在附件中，可以尝试用`frida`实现，最大的难点是无法实现断点，中断`app`状态。

为了能让`Unidbg`成功载入快照，拍摄内容：

1. `App`运行内存
2. 内存段的地址、权限信息
3. 断点中断时的寄存器信息
4. `SO`符号表（包括中断时地址）

有了目标，现在开始实现。

### 第一步：装载`lldb`（相机）

```bash
# 获取lldb android 服务端
$NDK_PATH/toolchains/llvm/prebuilt/linux-x86_64/lib64/clang/9.0.8/lib/linux/aarch64/lldb-server

# ADB导入设备、赋予权限
adb push lldb-server /data/local/tmp/
adb shell
cd /data/local/tmp
chmod 755 lldb-server

# 开启服务
./lldb-server p --server --listen unix-abstract://data/local/tmp/debug.sock

# pc端连接android
$ lldb
platform select remote-android
platform connect unix-abstract-connect://data/local/tmp/debug.sock
```

### 第二步：编写`dump`脚本

内存段（`Segment`）、寄存器（`Register`）的`lldb Dump`脚本
