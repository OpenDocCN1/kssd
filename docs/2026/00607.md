
# x64dbg---使用经验

一些杂乱的x64dbg使用经验

## 指令提示和帮助

在一条指令上右键

显示指令提示 --- 在右侧显示每条指令的基本含义

指令帮助 --- 在日志界面输出指令的各种用法解释

## 一些快捷键

`Ctrl + A` 分析模块，可自动分析出常见模式的函数区块

`Ctrl + Home` 跳转到当前函数开头

`Ctrl + End` 跳转到当前函数结尾

`x` 查看调用当前函数方(也就是相互引用，需要在函数开头按)

## 看门狗

可用于监测特定位置的数据变化

![看门狗](upload/attach/202308/736120_DT7ZKJBQRJCS5RY.png)

## 断点日志

x64dbg的断点记录比较丰富，log命令参数建议用双引号包裹

<mark class="encrypted">c70K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Z5k6h3I4H3i4K6u0W2P5o6j5@1k6r3u0Y4i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3V1k6D9j5i4c8W2M7%4c8Q4x3V1k6U0L8$3#2E0j5h3&6V1M7#2)9J5c8Y4y4U0M7X3W2H3N6q4)9J5c8X3I4G2k6#2)9J5k6h3S2@1L8h3H3`.</mark>

这里是一些参数格式和示例：

<mark class="encrypted">422K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Z5k6h3I4H3i4K6u0W2P5o6j5@1k6r3u0Y4i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3V1k6D9j5i4c8W2M7%4c8Q4x3V1k6A6L8Y4c8J5L8$3c8#2j5%4c8A6L8$3&6Q4x3V1k6r3L8%4u0E0j5i4c8@1K9h3&6Y4i4K6u0W2K9s2c8E0L8l9`.`.</mark>

一个输出举例，可以很方便看到一些输出字符串之类的

```
{utf16@[esp+4]}
{utf16@arg.get(0)}
```

断点命中次数和寄存器所指地址的字符串

```
{$breakpointcounter}:{ascii@rcx}
{$breakpointcounter}:{utf16@rcx}
```

指定内存的指定大小数据

`{mem;size@address}` will print the size bytes starting at address in hex.

举例，`0032B378`地址的4个字节

```
{mem;4@0032B378}
```

模块名、标签、注释

```
{modname@address}
{label@address}
{comment@address}
```

## 暂停条件或日志条件

`$breakpointcounter >= 20`

注意：这里的20是16进制

## 断点命令

断下后可以执行各种各样的命令

<mark class="encrypted">746K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Z5k6h3I4H3i4K6u0W2P5o6j5@1k6r3u0Y4i4K6u0W2j5$3!0E0i4K6u0r3k6h3&6Q4x3V1k6D9j5i4c8W2M7%4c8Q4x3V1k6U0L8$3#2E0j5h3&6V1M7#2)9J5c8X3W2F1k6r3g2^5i4K6u0W2K9s2c8E0L8l9`.`.</mark>

下方命令为: 将`bl`的值设为`dl`的值, 然后继续运行

`set bl, dl; go`

![断点命令](upload/attach/202308/736120_PTGC7ECDB5BDVQZ.png)

勾选"不输出日志"可以不输出断点被触发时的提示信息

还可以直接改`eip`的值，然后继续运行

`set eip, 401313; go`

`log`记录某值到日志

`log {ascii@ecx}`

设置变量用于数量控制

`set $num 0; set $num, $num+1`

休眠，16进制，单位为毫秒

`doSleep 3e8`

注意：断点命令为多条时，想运行一段时间再执行另外一条是不行的

如：

```
StepOut; log "{{label@eax}}"
# 原意是想运行到ret指令，然后输出eax的标签
# 这里的问题是后面的命令不会等待前面执行到ret再运行，所以想输出eax的标签就很随机而且很可能出错
```

## 给调用某一个api的所有位置下断点

x64dbg没有类似OD的`bpx`命令(有，但是功能不一样)

可以 右键 --> 搜索 --> 当前模块 --> 跨模块调用

![api断点](upload/attach/202308/736120_V84ZWQAC466SAZA.png)

按"反汇编"排序，找到相应的api，右键选择"在所有对***的调用上设置断点"

![所有调用设置断点](upload/attach/202308/736120_ZUUTDN62MT2KFQV.png)

## 修改程序

空格修改汇编代码

文件 --> 补丁 --> 修补文件 即可

## 导出小范围数据

选中数据，右键

二进制编辑 --> 编辑 --> 复制数据 即可

---

2020/7/13
