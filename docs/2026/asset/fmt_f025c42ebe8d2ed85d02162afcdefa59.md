
脚本：`uEmu.py` 代码太长 `d41K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6S2L8r3g2^5K9s2g2V1k6g2)9J5c8Y4g2q4L8i4g2Q4c8f1k6Q4b7V1y4Q4z5o6S2Q4c8e0k6Q4z5o6S2Q4z5e0k6Q4c8e0S2Q4z5o6m8Q4z5o6g2Q4c8e0g2Q4b7V1g2Q4b7f1g2Q4c8e0c8Q4b7V1k6Q4b7e0q4Q4c8e0g2Q4z5o6g2Q4b7f1y4Q4c8e0c8Q4b7V1y4Q4z5e0N6Q4c8e0g2Q4z5p5k6Q4b7U0N6Q4c8e0g2Q4z5f1u0Q4z5f1g2Q4c8e0g2Q4b7e0c8Q4z5p5c8Q4c8f1k6Q4b7V1y4Q4z5f1q4a6L8r3I4$3L8g2!0q4c8W2!0n7b7#2)9^5b7#2!0q4y4W2)9^5z5g2)9^5x3q4!0q4y4W2)9&6b7#2)9^5z5g2!0q4z5q4)9^5y4q4)9&6b7g2!0q4y4W2)9&6b7#2!0m8b7#2!0q4z5q4)9^5y4#2!0m8b7g2!0q4y4g2)9^5c8W2)9&6y4W2!0q4c8W2!0n7b7#2)9^5z5b7.`

脚本导入操作：复制`uEmu.py`到`IDA_Pro_7.5\plugins\`下，重启 IDA。

首先我们找到加密函数，设置起始和结束断点：

![image-20220922175139399](https://bbs.kanxue.com/upload/attach/202210/905443_5JR52FD7XCSXRF7.png)

![image-20220922175209878](https://bbs.kanxue.com/upload/attach/202210/905443_KWEPXNKVQNQRPCW.png)

![image-20220922175235173](https://bbs.kanxue.com/upload/attach/202210/905443_A8NJBRD4EK25XZG.png)

然后我们右键，此时出现`uEmu`：

![image-20220922175347552](https://bbs.kanxue.com/upload/attach/202210/905443_WH97X3YWSJQJP2E.png)

操作说明：

1. `start` 命令：通过映射所有段并设置 Unicorn 来初始化模拟器
2. `run` 命令：模拟指令，直到到达断点或发生错误
3. `Step`：模拟一条或 N 条指令（按住 `ALT`/`OPTION` 指定一个数字）
4. `stop`：中断仿真
5. `Reset`：重置仿真引擎并取消映射所有内存区域
6. `Jump to Pc`：只是跳转到当前 PC
7. `Change CPU context`：可以手动或通过 JSON 文件更新 CPU 上下文（见下文）
8. `Show Controls`：显示带有 `Start`/`Run`/`Step`/`Stop` 按钮的窗口
9. `Show CPU Context`：显示带有可用寄存器的窗口
10. `Show CPU Extended Context`：显示带有扩展寄存器（`FP`/`SIMD`）的窗口
11. `Show Stack`：显示带有当前 Stack 的窗口
12. `Show Memory Range`：允许显示特定的内存区域

我们点击`start`启动初始化模拟器，并不断确定：

![image-20220922175704258](https://bbs.kanxue.com/upload/attach/202210/905443_7WZMN3YS8N95C3X.png)

![image-20220922175725288](https://bbs.kanxue.com/upload/attach/202210/905443_8T2S579N6FG4T86.png)

![image-20220922175805710](https://bbs.kanxue.com/upload/attach/202210/905443_XVYSET5PCWEKHN4.png)

我们可以使用快捷键`ctrl+shfit+alt+s`进行快速单步步过：

![image-20220922180009607](https://bbs.kanxue.com/upload/attach/202210/905443_2DNQMEDZCA4TKQJ.png)

可以选择一次步过多少条指令：

![image-20220922184945143](https://bbs.kanxue.com/upload/attach/202210/905443_PBP7EJRB5F6C95T.png)

此时我们查看`0x4004`处的字符串，右键 `uEMu` --> `Show Memory Range`：

![image-20220922185053241](https://bbs.kanxue.com/upload/attach/202210/905443_D85Y6R3JAMCYXPB.png)

![image-20220922185122178](https://bbs.kanxue.com/upload/attach/202210/905443_YSHHDQWWJDW85ZX.png)

可以看见此时的还是加密的 QUR，还未解密，我们也可以在 so 中看到：

![image-20220922185226245](https://bbs.kanxue.com/upload/attach/202210/905443_HT534KSX9NFCFCK.png)

我们继续调试，按照我们上面的结果，这里应该为 `jni.go into _ini`，这里我们可以直接下断点，然后 `run` 过去：

![image-20220922191223988](https://bbs.kanxue.com/upload/attach/202210/905443_63B5R8X32J7AEV3.png)

![image-20220922190331570](https://bbs.kanxue.com/upload/attach/202210/905443_EY5AW3C3PTNR9Z8.png)

可以发现很好的进行了解密，其实原理和上面一样。

### 3.2 反虚假控制流

#### 3.2.1 符号执行

环境要求：

安装 `angr`：`pip install angr==8.19.4.5`，`angr` 版本为 `8.19.4.5`。

我们打开一个虚假控制流混淆后的样本：

![image-20220922200516763](https://bbs.kanxue.com/upload/attach/202210/905443_QBRYP3379SFAJBW.png)

很明显我们在这里看到了一些不透明谓词，这是一个虚假控制流混淆处理后的代码。

运行去混淆脚本：

`python debogus.py -f target_arm_bogus --addr 0x83B4`

![image-20220922200812611](https://bbs.kanxue.com/upload/attach/202210/905443_UEQ6MMU23ZG7XWX.png)

![image-20220922200840974](https://bbs.kanxue.com/upload/attach/202210/905443_6EZVMBMBFTTEDPK.png)

这里我们打开修复后的样本：

![image-20220922201047300](https://bbs.kanxue.com/upload/attach/202210/905443_FCZVFVXTAT2A5YE.png)

可以看到一些不可达块填充为 nop：

![image-20220922201128282](https://bbs.kanxue.com/upload/attach/202210/905443_YGKJAWTACZV4695.png)

去除虚假控制流的 IDApython 脚本：

```python
# 去除虚假控制流 idapython脚本
import ida_xref
import ida_idaapi
from ida_bytes import get_bytes, patch_bytes

# 将 mov 寄存器, 不透明谓词 修改为 mov 寄存器, 0
def do_patch(ea):
    if get_bytes(ea, 1) == b"\x8B": # mov eax-edi, dword
        reg = (ord(get_bytes(ea + 1, 1)) & 0b00111000) >> 3
        patch_bytes(ea, (0xB8 + reg).to_bytes(1, 'little') + b'\x00\x00\x00\x00\x90')
    else:
        print('error')

# 不透明谓词在.bss段的范围
start = 0x0011028
end = 0x0011030

for addr in range(start, end, 4):
    ref = ida_xref.get_first_dref_to(addr)
    print(hex(addr).center(20, '-'))
    # 获取所有交叉引用
    while ref != ida_idaapi.BADADDR:
        do_patch(ref)
        print('patch at ' + hex(ref))
        ref = ida_xref.get_next_dref_to(addr, ref)
    print('-' * 20)
```

#### 3.2.2 模拟执行

使用基于 Unicorn 的框架可以去虚假控制流，思路如下：

1. 通过 Unicorn 模拟执行目标函数，并记录可达块和不可达块的地址，将可达块的地址全部报错
2. 编写 IDApython 脚本，将不可达块地方全部填充为 `nop`，就可以除去虚假控制流

参考文章：`016K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3#2A6M7%4y4C8K9h3&6Y4i4K6u0W2j5$3y4Q4x3V1j5J5x3o6t1I4i4K6u0r3x3o6g2Q4x3V1j5I4y4q4)9J5c8X3!0D9L8s2k6E0x3#2)9J5c8R3.`

### 3.3 反控制流平坦化

#### 3.3.1 符号执行

环境：

安装 `angr`：`pip install angr==8.19.4.5`，`angr` 版本为 `8.19.4.5`。

我们先打开一个控制流平坦化的样本：

![image-20220922213932982](https://bbs.kanxue.com/upload/attach/202210/905443_R6UAAX4Z35246B6.png)

这是一个标准的控制流平坦化的样本，我们可以看见序言、主分发器、次分发器、结束块等。

我们通过符号执行来除去：

`python deflat.py -f samples/bin/check_passwd_arm_flat --addr 0x83B0`（目标函数地址）

![image-20220922214629123](https://bbs.kanxue.com/upload/attach/202210/905443_4J4NGTH8GG4VE6Q.png)

我们将修复后的样本打开：

原样本：

![image-20220922214902748](https://bbs.kanxue.com/upload/attach/202210/905443_Y4TQXRA7J3HDA9M.png)

去除混淆后的样本：

![image-20220922214827404](https://bbs.kanxue.com/upload/attach/202210/905443_NVJA4TY8C62TXXV.png)

#### 3.3.2 模拟执行

环境要求：

![image-20220922220425663](https://bbs.kanxue.com/upload/attach/202210/905443_56BQTCEKY43C4N2.png)

我们打开另外一个样本：

![image-20220922215820915](https://bbs.kanxue.com/upload/attach/202210/905443_296XD5GGTWB4BY2.png)

定位到函数的起始和终止：

![image-20220922215854085](https://bbs.kanxue.com/upload/attach/202210/905443_35GG5DRF6BC8D2N.png)

然后启动脚本：

![image-20220922215709565](https://bbs.kanxue.com/upload/attach/202210/905443_R8NP36NQ3VNHFW2.png)

![image-20220922215934225](https://bbs.kanxue.com/upload/attach/202210/905443_JPC4A86YZND66SQ.png)

打开修复后的样本：

![image-20220922220136390](https://bbs.kanxue.com/upload/attach/202210/905443_QGE9QEDJWC8Y5DN.png)

![image-20220922220154180](https://bbs.kanxue.com/upload/attach/202210/905443_6T2QC678963NB3T.png)

## 4. 总结

本文初步的简单介绍了当前 Ollvm 反混淆中的一些方法并进行了复现，样本的实例都来自于当前研究该领域的各个大佬开源脚本，这里就不一一提及了，感兴趣朋友可以查看参考文献。

本实验的实验样本全部上传至知识星球：安全后厨。

本实验的全部脚本全部上传至 GitHub：[WindXaa](<https://bbs.kanxue.com/elink@d1cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6i4K9h3&6V1h3r3q4S2i4K6u0r3b7h3&6V1M7X3!0A6k6q4)9J5k6s2u0W2N6X3g2J5M7$3f1>)` 或微信公众号：安全后厨。

## 5. 参考文章

- `https://bbs.pediy.com/thread-268108.htm#msg_header_h1_0`
- `http://missking.cc/2020/11/03/unicorn2/`
- `https://bbs.pediy.com/thread-266005.htm`

虚假控制流：

- `https://bbs.pediy.com/thread-257213.htm`
- `https://github.com/llxiaoyuan/llux`
- `https://bbs.pediy.com/thread-257213.htm`
- `https://github.com/cq674350529/deflat`

控制流平坦化：

- `https://bbs.pediy.com/thread-268108.htm`
- `http://missking.cc/2020/11/03/unicorn2/`
- `https://github.com/llxiaoyuan/llux`
