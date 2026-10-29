

# 基于 `lldb` 的 trace 脚本<辅助算法分析，算法还原，以及算法验证>

由于工作的原因，在逆向分析中，经常遇到强混淆，vm 虚拟化等加固方案。在分析中，痛不欲生。为了能在逆向过程中，还能安心的喝口咖啡，同时还能分析/还原各种高混淆/vm 虚拟化的代码。参考函数追踪的原理，弄了个指令级的 `lldb-trace` 脚本。

> **前瞻：**
>
> 平时分析过程中，难免遇到 c/c++ 函数，以及 objc 函数。而 objc 函数中，又包含了 `retain`、`release`... 等函数。故而，在 trace 过程中，objc 函数形如 `retain`、`release`... 等等都直接忽略。而对于 c++ 函数，类的构造函数、析构函数、系统函数等，也可以做过滤处理。对于 objc 函数 `objc_msgsend`，我们可以做重点分析：在于你需不需要分析这个函数。而我不需要，所以，我只需要知道 `objc_msgsend` 函数的函数即可，而对应的函数实现，我就不 trace 了。

### 框架分析：

> 1. 对不同的平台而言，做不同的配置
> 2. 忽略的函数，都放在忽略的函数列表中
> 3. `objc_msgsend` 函数需要特殊处理，故放在受保护的函数列表中
> 4. trace 过程中，需要读取寄存器的值，所以，用正则匹配出上一条指令所有的寄存器

### 更新：

> 1. `trace` 指令参数的优化，绝大部分参数都有默认值
> 2. tracing 中，结束地址可以有多个（在某些混淆情况下，不确定结束地址在哪，可以多设置几个结束地址，用 ";" 分割）
> 3. 增加了暂停其他线程的可选参数
> 4. 增加了只 trace 本模块的可选参数
> 5. 增加了进度信息（防止以为脚本卡死…等的不耐心…从而关闭了 `lldb`）
> 6. 对 `msg_send` 函数的参数解开发中…
> 7. 增加了对还原的算法检测脚本

## 脚本怎么使用：

> 1. 在准备追踪的地方下断点：（我的断点，从 `breakpoint` 函数 `si` 进入到 `a` 函数的第一行）
>
> ![](upload/attach/202107/748611_8JXMMAMNGHUY3RQ.png)

> 2. 导入 `lldbTrace.py` 脚本。（你可以设置，默认的 log 文件路径。如果不设置，默认和脚本同位置）
>
> ![](upload/attach/202107/748611_CWY5Y29QTD7T65B.png)

> 3. 设置一个停止追踪的地址：（当前 `a` 函数，我把结束地址设为最后地址，和 `ret` 地址。为了查看 debug 信息，我把 log 类型设置成了 debug）
>
> ![](upload/attach/202107/748611_9VNXU5JG99Q5HK9.png)

> 4. 设置好，直接回车，结果如下：
>
> ![](upload/attach/202107/748611_W4NWEK25E95S2P7.png)

### 脚本在 git 上：

[基于 lldb 的汇编指令级 trace 脚本](<elink@da4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6&6j5h3&6Y4P5i4y4K6i4K6u0r3L8r3I4V1j5W2)9J5k6s2c8J5j5h3y4W2>)

> 脚本还有很多不完善的地方，需要慢慢优化。

不过利用 trace 结果，能还原手写的算法，以及强混淆或者某些 vm 虚拟机。

### 某手撸 AES 算法的 trace 结果：

![](upload/attach/202107/748611_RZNWTHPYTY83E48.png)

## 基于 trace 脚本的基础上，对 trace 结果，以及实际分析中，做了一个自动检测还原的函数的脚本。利用脚本，不用每次都去动态调试，对照比较结果。

### 算法还原检测脚本简介：

```python
#!/usr/bin/python3
# -*- encoding: utf-8 -*-
# @File : test.py
# @Time : 2021/07/29 15:15:38
# @Author : wt
from wtpytracer import *
####################
## command :
## python3 test.py ~/Desktop/1627533881instrace.log
## 定义需要检测的变量 : flag + '_' + 地址 + '_' + 寄存器
Check_0x10000393c_w8 = 'Check_0x10000393c_w8'
### 翻译的测试代码
def f(x):
    ret = 0
    for index in range(x):
        ret = ret + index
    check_value(ret, Check_0x10000393c_w8) # check ret 和 0x10000393c 的 w8 的寄存器值
    return ret + x
if __name__ == '__main__':
    import sys
    args_list = sys.argv
    if len(args_list) != 2:
        exit()
    file_name = args_list[1]
    try:
        set_trace_data(Check_0x10000393c_w8)
        parser_trace_log_file(file_name)
        enable(CheckFunctionTracer())
        f(5)
    finally:
        disable()
```

### 具体操作如下：

> **a.** 在 IDA 中，找到需要还原的算法（可以是汇编，也可以是伪代码），翻译成对应的 Python 代码
>
> ![](upload/attach/202107/748611_R5MCHG82KJE8KZ5.png)

> **b.** 利用 `lldbTrace.py` 脚本，把需要翻译的函数 trace 一哈。结果如下：
>
> ![](upload/attach/202107/748611_8TCBSVU8FJF5EG4.png)

> **c.** 在 IDA 伪代码中，切换到对应的汇编，对照 trace 结果，确定具体检测的地址。因为 trace 当前打印的是上一句代码执行后的寄存器值。所以，我们把 `check` 的地址定义为 `0x10000393c`，寄存器为 `w8`
>
> ![](upload/attach/202107/748611_AZEE9BVXST3KVXF.png)

> **d.** 定义检测的变量 `Check_0x10000393c_w8 = 'Check_0x10000393c_w8'`。在翻译的代码中，添加检测函数 `check_value(ret, Check_0x10000393c_w8)`。在解析 `tracelog` 文件之前，设置 `check` 的相关信息 `set_trace_data(Check_0x10000393c_w8)`

> **e.** 用 Python 调用此脚本，并传入 `tracelog` 文件的路径。结果如下：
>
> ![](upload/attach/202107/748611_2GXP7CR349V2WJC.png)

### 模块代码：

