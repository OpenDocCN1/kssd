
![图片描述](https://bbs.kanxue.com/upload/attach/202604/1075564_GU9HBA9KNQ3PP98.webp)

有点奇怪的题。

## ezheap

![图片描述](https://bbs.kanxue.com/upload/attach/202604/1075564_UPZSV5D2XBEYMB5.webp)

![图片描述](https://bbs.kanxue.com/upload/attach/202604/1075564_QUCKUXFB2GEWVM4.webp)

有 `seccomp`。

![图片描述](https://bbs.kanxue.com/upload/attach/202604/1075564_3PV7PXWJEPDTKHU.webp)

`ban` 了 `execve` 和 `execveat`。

用 `orw`。

![图片描述](https://bbs.kanxue.com/upload/attach/202604/1075564_JNBYT7H38F8X2DW.webp)

`c++` 编译。

![图片描述](https://bbs.kanxue.com/upload/attach/202604/1075564_FVMM634EN2KDCRV.webp)

全开。

![图片描述](https://bbs.kanxue.com/upload/attach/202604/1075564_VC9SFBH26CPZ5BT.webp)

我去，`2.39`。

![图片描述](https://bbs.kanxue.com/upload/attach/202604/1075564_V4UPBGK6QTJRNSV.webp)

有 `backdoor function`，那还用个蛋的 `orw`。

![图片描述](https://bbs.kanxue.com/upload/attach/202604/1075564_2ZKCX3TVVCQ34PP.webp)

![图片描述](https://bbs.kanxue.com/upload/attach/202604/1075564_JRQMQ4UHA48RP9B.webp)

这笔记看着就高级。

翻译一下：

| 编号 | 选项名称 | 翻译 |
| --- | --- | --- |
| `[1]` | `Register model artifact` | 注册模型产物 |
| `[2]` | `Stream artifact chunk` | 流式传输产物分片 |
| `[3]` | `Bootstrap async scheduler` | 引导异步调度器 |
| `[4]` | `Inspect scheduler queue` | 检查调度队列 |
| `[5]` | `Allocate session tensor` | 分配会话张量 |
| `[6]` | `Complete batch inference` | 完成批处理推理 |
| `[7]` | `Patch session metadata` | 修复/更新会话元数据 |
| `[8]` | `Provision worker profile` | 配置工作节点配置文件 |
| `[9]` | `Dispatch async task` | 派遣异步任务。 |
| `[10]` | `Runtime telemetry` | 运行时遥测 |
| `[11]` | `Operator handbook` | 操作员手册 |
| `[0]` | `Shutdown gateway` | 关闭网关 |

根本看不懂。

这出题人怎么这么坏啊。

•᷄ࡇ•᷅

![图片描述](https://bbs.kanxue.com/upload/attach/202604/1075564_C7X3TCTTYDENGGP.webp)

什么叫 `main` 函数里还有加密？？？

不对，不是加密，这是自校验。

![图片描述](https://bbs.kanxue.com/upload/attach/202604/1075564_T3KSVAXNUKB5PBR.webp)

底下这一块才是要看的。

兮，这 `c++` 反汇编我还不太看的懂。

![图片描述](https://bbs.kanxue.com/upload/attach/202604/1075564_W6T54A8YNKUUPFZ.webp)

慢慢看了。

![图片描述](https://bbs.kanxue.com/upload/attach/202604/1075564_RTZ4NE6CVJMBYSU.webp)

![图片描述](https://bbs.kanxue.com/upload/attach/202604/1075564_6GBJ4ZJAW73EN6Z.webp)

先清零 `buf` 的 376+16 字节。

`malloc` 了 `0x200`，512 bytes。

`v23 = begin`，`v29 = the chunk's end`。

```cpp
*(__m128i *)&buf[376] = _mm_unpacklo_epi64((__m128i)v23, (__m128i)v23);
```

将两个 64 位整数交叉组合成一个 128 位的向量。

两个参数都是 `v23`，故高 64 位、低 64 位都是 `v23`。

`sub_8A60` 和 `sub_3570`：传入了 `buf` 的起始地址。

先看看 `8a60`。

![图片描述](https://bbs.kanxue.com/upload/attach/202604/1075564_N9ZZX3AHPVREZSM.webp)

控制台这一块。

![图片描述](https://bbs.kanxue.com/upload/attach/202604/1075564_U93ZP6AUKC2NQR4.webp)

`ban execve`。

![图片描述](https://bbs.kanxue.com/upload/attach/202604/1075564_ZDXUWMXVS2M3BKU.webp)

`buf` 共用。

![图片描述](https://bbs.kanxue.com/upload/attach/202604/1075564_DTYNTMR45TMMP8Y.webp)

用来 `int(str)` 的。

![图片描述](https://bbs.kanxue.com/upload/attach/202604/1075564_PFVY3MPH8YJ3MSR.webp)

`8a60` 结束后 `free buf` 的，`6720` 也是。

### 看看 `case1`

![图片描述](https://bbs.kanxue.com/upload/attach/202604/1075564_VK8UYP8JYTVHFJT.webp)

`calloc`。

![图片描述](https://bbs.kanxue.com/upload/attach/202604/1075564_HTRET4UDMAM2YEF.webp)

打印 `calloc` 的返回地址。

![图片描述](https://bbs.kanxue.com/upload/attach/202604/1075564_MVRQ3JM2WR5DN6D.webp)

单个输入 `v9`（`items`）和 `v42`（`stride`）被限制在 32 位以内。

`n32_1 = (std::ios_base *)(v42 * v9);`

乘积被存入了 64 位的 `n32_1` 中。

`n32 = (unsigned int)n32_1;`

强制将 64 位的 `n32_1` 转换为 32 位的 `unsigned int`。

`v13 = calloc(n32, 1uLL);`

使用截断后的 32 位小数值去分配堆内存。

存在 Integer Truncation（高位砍掉）。

这样就可以伪造所记录的 `calloc` 的 chunk 大小。

### 看看 `case2`

![图片描述](https://bbs.kanxue.com/upload/attach/202604/1075564_3ACJY22D6PSKAZS.webp)

```cpp
// 1. 获取 Case 1 中记录的 64 位 chunk size
n0x5F_1 = *(_QWORD *)(buf + 16);

// 2. 获取输入的 offset
n0x5F_2 = n0x5F;

// 3. 尺寸检查
if (n0x5F >= n0x5F_1) {
    // 如果 offset 大于等于记录的大小，则报错
    std::__ostream_insert(..., "stream exceeds declared artifact size", ...);
} else {
    // 4. 窗口策略检查
    if (n0x5F <= 0x5F) { // 0x5F = 95
        // 将 payload 拷贝到 arena_ptr + offset 的位置
        // 限制条件：offset <= 95
    }
}
```

### 看看 `case3`

什么叫引导异步调度器？

![图片描述](https://bbs.kanxue.com/upload/attach/202604/1075564_WPCBCWFEQ7XUBFT.webp)

启动后台的任务分发系统。

创建总控台（Scheduler Control Plane）：分配一个全局的调度器控制块（`queue_ctrl`），里面记录了调度器的状态，并且初始化了安全策略（`strict_policy = 1`）。

创建任务槽（Task Descriptors）：循环 8 次，创建 8 个任务结构体（Task 0 到 Task 7）。每个结构体里预先填好了默认的回调函数指针（`handler`）和参数（`context`）。

```cpp
// 1. 分配控制块（防火墙开关）
v10 = malloc(0x38);                    // 实际会分配出 0x40 大小的物理堆块

*((_WORD *)v10 + 4) = 257;             // 在偏移 +0x08 的位置，写入了 strict_policy = 1

*(_QWORD *)(a1 + 32) = v10;            // 把指针挂载到全局上下文 a1(buf) 上

// 2. 循环分配 8 个任务块（执行流入口）
while (n7 != 8) {
    v15 = malloc(0x50);                // 实际会分配出 0x60 大小的物理堆块

    *((_QWORD *)v15 + 3) = sub_30A0;   // 在偏移 +0x18 的位置，写入了默认的函数指针
    *((_QWORD *)v15 + 4) = v15;        // 在偏移 +0x20 的位置，写入了参数指针

    // 把 Task 指针挂载到全局上下文 a1(buf) 偏移 +40 开始的数组中
    *(_QWORD *)(a1 + 8 * n7 + 40) = v15;

    ++n7;
}
```

![图片描述](https://bbs.kanxue.com/upload/attach/202604/1075564_YRTYF9KGHBGGU2R.webp)

可以通过 `case2` 的溢出覆盖 `v10`。

### 看看 `case4`

![图片描述](https://bbs.kanxue.com/upload/attach/202604/1075564_JXUBJSQAJEUUDHC.webp)

`4` 可以泄露 `addr`。

![图片描述](https://bbs.kanxue.com/upload/attach/202604/1075564_FCXY8HERYCPZXQF.webp)

### 看看 `case5`

```cpp
// 1. 校验 Slot 是否已被占用（n0xF 为 0-15）
// a1 + 104 (0x68) 处开始存指针数组
if (*(_QWORD *)(a1 + 8 * n0xF + 104)) {
    std::__ostream_insert(..., "session slot already allocated", ...);
    return;
}

// 2. 分配 Session 核心对象（申请 0x50，实际分配 0x60 的堆块）
v11 = operator new(0x50uLL);

*(_QWORD *)v11 = n0xF;                // +0x00: 存入 slot 号
*(_DWORD *)(v11 + 4) = size;          // +0x04: 存入 tensor_bytes 大小

// 3. 分配 Payload 缓冲区
if (size) {
    v14 = malloc(size);               // 根据输入的大小分配数据区
    *((_QWORD *)v11 + 1) = v14;       // +0x08: payload_ptr（数据指针）
}

// 4. 初始化别名（alias）与回调函数
__snprintf_chk(v11 + 16, 32LL, ... , "%s", alias); // +0x10: 32 字节的 alias 字符串
*((_QWORD *)v11 + 7) = sub_2C50;                    // +0x38: postproc_func（默认函数指针）

// 5. 登记到全局上下文（a1 为 392 字节的 buf）
*(_QWORD *)(a1 + 8 * n0xF + 104) = v11; // 登记对象指针到 buf[0x68 + slot*8]
*(_BYTE *)(a1 + n0xF + 360) = 1;        // 将 active 标志位置 1（buf[0x168 + slot]）
```

### `case6`

![图片描述](https://bbs.kanxue.com/upload/attach/202604/1075564_3FNXBSEFP9MYXRE.webp)

UAF。

### 看看 `case7`

![图片描述](https://bbs.kanxue.com/upload/attach/202604/1075564_F3VBQSCKQRJADJE.webp)

可以用它改 `fd`。

这就是 `edit`。

### 看看 `case8`

`Case 8` 表面上是一个配置节点的业务功能，但在 Pwn 攻击链中，它是一个无损的 64 位任意内存内容填充器。

| 物理内存偏移 | `Case 8` 以为它在写什么？（Worker Profile） | 实际传入的 Payload | `Case 9` 以为它在读什么？（Task Descriptor） |
| --- | --- | --- | --- |
| `+0x00` | `cpu_quota` | `0` | `task_id` |
| `+0x08` | `mem_quota` | `0` | 状态字段 / 占位 |
| `+0x10` | `io_weight` | `0` | 状态字段 / 占位 |
| `+0x18` | `latency_slo` | `backdoor_addr` | `handler`<br>（后门函数指针） |
| `+0x20` | `replicas` | `task_0_addr + 40` | `context`<br>（传给后门函数的参数指针！） |
| `+0x28` | `memo` | `b"\x00"*8` | `tag`<br>字符串（随便） |

![图片描述](https://bbs.kanxue.com/upload/attach/202604/1075564_AEURBQFDKPRS9C8.webp)

### 看看 `case9`

```cpp
// 1. 提取结构体与函数指针

// a1：392 字节的全局上下文（buf）
// n7：输入的 task_id（例如 0）
// 从 a1 + 40 (0x28) 处取出 Task 0 的绝对堆地址
v10 = *(_QWORD *)(a1 + 8 * n7 + 40);

if (v10 && (sub_2FF0_1 = *(void (__fastcall **)(_QWORD))(v10 + 24)) != 0LL) {
    // v10 + 24 (+0x18)：handler 函数指针
    // v10 + 32 (+0x20)：context 参数指针

    // 2. 白名单与策略校验
    if (sub_2FF0_1 == sub_2FF0                    // 检查是否为默认函数 A
      || (char *)sub_2FF0_1 == (char *)sub_30A0    // 检查是否为默认函数 B
      || *(_BYTE *)(*(_QWORD *)(a1 + 32) + 8LL) == 0 // 核心：检查 strict_policy 是否为 0
      || sub_2FF0_1 == sub_2F40)                  // 检查是否为默认函数 C
    {
        // 校验通过，执行函数调用
        // 如果 strict_policy 被篡改为 0，则可以执行被替换后的 handler
        sub_2FF0_1(*(_QWORD *)(v10 + 32));
    } else {
        // 策略拦截
        std::__ostream_insert(..., "policy engine blocked non-whitelisted handler", ...);
    }
}
```

可以通过 `case2` 把 `strict_policy` 改成 `0`。
