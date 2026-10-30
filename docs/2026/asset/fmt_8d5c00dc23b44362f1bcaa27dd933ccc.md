
```c
int __stdcall CVDecompress(ULONG data, int src, int length, int a4, int a5, int a6, int a7)
{
    // [COLLAPSED LOCAL DECLARATIONS. PRESS KEYPAD CTRL-"+" TO EXPAND]
    data_ = data;
    v8 = *(data + 36);
    if...
    result = 0;
    if ( length >= 0x20 )
    {
        src_ = src;
        BYTE1(result) = *(src + 1);
        LOBYTE(result) = *(src + 2);
        v11 = *(src + 3) | (result << 8);
        // 这里得到的还是length, 0x68
        if ( length < v11 || (HIBYTE(length) = *src, ULongSub(v11, 10u, &remain_len) < 0) )
        // 这里做了一个运算0x68-10，减去了头部的长度看是否还大于0
        {
            LABEL_33:
            result = 0;
        }
        else
        {
            HIBYTE(nStrip) = *(src_ + 8);
            strip = (src_ + 10);
            // 加上了头部的长度，现在指向strip 1的起始位置
            strip_idx = 0;
            strip_ = strip;
            strip__ = strip;
            LOBYTE(nStrip) = *(strip - 1);
            // 这里是strip的个数
            nStrip_ = nStrip;
            if ( nStrip )
            {
                v32 = 0;
                do
                {
                    if ( remain_len < 0x16 )
                        break;
                    HIBYTE(v14) = strip[1];
                    LOBYTE(v14) = strip[2];
                    strip_len = strip[3] | (v14 << 8);
                    if ( remain_len < strip_len )
                        break;
                    if ( *strip == 0x10 || *strip == 0x11 ) // 检查strip ID
                    {
                        if ( ULongSub(strip_len, 0xCu, &data) < 0 ) // 这里又减去了strip header的长度12个字节
                        // data的位置存储的是差值
                            goto LABEL_33;
                        HIBYTE(bottom_y) = strip[8];
                        HIBYTE(top_y) = strip[4];
                        LOBYTE(bottom_y) = strip[9];
                        LOBYTE(top_y) = strip[5];
                        v17 = bottom_y - top_y;
                        LOWORD(v17) = *(data_ + 46) * v17;
                        src = v17;
                        if ( v32 && !HIBYTE(length) && *strip == 0x11 ) // 当遍历到第二个strip的时候，条件都满足，进入if块
                        {
                            qmemcpy((*(data_ + 28) + v32), (*(data_ + 28) + v32 - 0x2000), 0x2000u); // 异常发生在这里
                            strip = strip_;
                        }
                        chunk = strip__ + 12;
                        // 这里又加上了strip header的长度，目前指向了chunk
                        chunk_ = strip + 12;
                        *(data_ + 56) = v32 + *(data_ + 32);
                        chunk__ = strip + 12;
                        *(data_ + 60) = a7;
                        while ( data >= 4 )
                        {
                            HIBYTE(v20) = chunk_[1];
                            LOBYTE(v20) = chunk_[2];
                            chunk_size = chunk_[3] | (v20 << 8);
                            chunk_size_ = chunk_size;
                            if ( data < chunk_size ) // 验证chunk_size的大小
                                break;
                            switch...
                            // 这里在检查chunk ID,根据chunk ID处理chunk数据
                            chunk_ = &chunk__[chunk_size_];
                            v22 = 1;
                            chunk += chunk_size_;
                            chunk__ += chunk_size_;
                            if ( chunk_size_ > 1 )
                                v22 = chunk_size_;
                            data -= v22;
                        }
                        a6 += a7 * src;
                        ++strip_idx;
                        v32 += 0x2000;
                    }
                    strip__ += strip_len;
                    // 接着处理下一个strip
                    remain_len -= strip_len;
                    strip += strip_len;
                    strip_ = strip;
                }
                while ( strip_idx < nStrip_ );
            }
            result = 1;
        }
    }
    return result;
}
```

上述代码理解可以看下面这张图：

![图片描述](https://bbs.kanxue.com/upload/attach/202107/600394_WNCUAGC2VAAZBJ3.png)

每次遇到 strip ID 是 `0x1100` 的时候，都会进行数据的复制操作，每次复制的字节数为 `0x2000`，这里并没有判断堆中是否能够容纳这么多的数据，查看一下堆块的大小：

```text
0:005> !heap -p -a edi
address 00173e78 found in _HEAP @ a0000
HEAP_ENTRY Size Prev Flags    UserPtr UserSize - state
00171e70 0c01 0000  [01] 00171e78    06000 - (busy)
? wmploc+93
```

可以看到这个堆的大小是 `0x6000`，也就是说复制三次数据就满了，第四次再复制的时候就会导致堆溢出。o(*￣▽￣*)ブ

## 4.4 小总结

### 4.4.1 关于 cvid 压缩格式

目前看来我对于 `CVID` 格式的理解是没什么问题的，`chunk` 是 `strip` 的一部分，`strip1` 后面跟着 `strip2`，以此类推

### 4.4.2 如何触发漏洞

1. `CVID` 数据长度就按照数据总长度设置，需要大于 `0x20`

2. 包含大于 3 个，且 ID 为 `0x1100` 的 `strip`

# 5. 本文 Windbg 学习总结

1. `!heap`：查看堆相关信息

    使用 `!heap -?` 查看帮助信息

    `!heap -p`：查看所有可能的 handle 值

    `!heap -p -h HANDLE`：查看对应 `HANDLE` 的详细分配信息

    `!heap -p -a ADDR`：显示具体某个堆块的详细信息

2. `dt STRUCTURE ADDR`：按照 `STRUCTURE` 结构显示 `ADDR` 处结构体各成员值

3. `!gflag [-? | flags]` 指令用于堆调试，重点标志：`htc`, `hfc`, `hpc`, `hpa`

4. `sx`：用于控制被调试程序发生某异常或特定事件时，调试器要采取的动作

    `sxe ld:ModuleName` 在首次加载 `ModuleName` 的时候中断。

# 6. 总结

这次漏洞分析花费的时间主要集中在前面对于堆溢出相关知识的复习，以及 `AVI` 格式的学习上。一旦对于格式有所了解，使用 `!gflag` 定位到漏洞函数后，代码分析就很简单了。

`!gflag` 和 `!heap` 命令在堆溢出的漏洞分析上帮助很大；如何使用 Windbg，在加载 DLL 之后设置断点也是新接触到的一个知识点。

对于该漏洞本身，在刚看到漏洞介绍时，我只以为是在单次 `strcpy()` 调用时复制的数据超过了目标空间大小，但实际调试时发现是多次同样大小的数据复制最终超过了目标空间大小，虽然原理相同，但这里需要判断的其实应该是复制的次数而不是大小了。

# 7. 参考资料

1. 《漏洞战争》

2. [Common WinDbg Commands (Thematically Grouped)](<https://bbs.kanxue.com/elink@492K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6A6L8X3c8T1k6#2)9J5k6h3W2F1k6X3!0Q4x3V1k6V1L8$3y4Q4x3V1j5I4i4K6u0V1j5$3!0E0L8h3!0F1i4K6u0V1j5$3#2V1M7#2)9J5k6h3S2@1L8h3H3`.)>)

3. [AVI文件格式详解](<https://bbs.kanxue.com/elink@388K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2j5%4y4V1L8W2)9J5k6h3&6W2N6q4)9J5c8X3y4Z5k6h3&6&6L8$3&6C8k6h3&6Q4x3V1k6S2M7Y4c8A6j5$3I4W2i4K6u0r3k6r3g2@1j5h3W2D9M7#2)9J5c8U0M7&6x3e0M7@1y4e0l9H3)>)

4. [cinepak文档](<https://bbs.kanxue.com/elink@2a9K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6E0N6h3I4@1K9h3#2W2k6r3W2S2i4K6u0W2j5%4S2Q4x3V1k6E0K9i4u0J5L8%4u0Q4x3V1k6U0K9h3&6W2M7r3q4C8i4K6u0W2N6s2S2@1)>)

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

[#漏洞分析](https://bbs.kanxue.com/forum-150-1-153.htm)

[#Windows](https://bbs.kanxue.com/forum-150-1-160.htm)
