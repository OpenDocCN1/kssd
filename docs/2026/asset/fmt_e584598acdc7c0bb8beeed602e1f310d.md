
![verify-exception-record](https://bbs.kanxue.com/upload/attach/202310/873494_84SJJQUWHVNF3RJ.webp)

> **注意：** `0000004e 78dfa980` 是调用栈帧 `0e` 的 `Child-SP` 的值。加上偏移 `0x4f0` 就得到了 `_EXCEPTION_RECORD` 对象的首地址。

从上图可知，`ExceptionCode` 的值是 `0xc0000005` （常见的访问违例），导致异常的指令地址是 `0x00007ffd bbde9863`。从 `ExceptionInformation[0]` 可知导致异常的访问类型是**读取**（`0` 表示读取，`1` 表示写入），从 `ExceptionInformation[1]` 可知，导致异常的访问地址是 `0x000001bc 12e12052` 。

> **说明：** 对以上字段的解读可以参考《软件调试》第一版 第 `12` 章 `320` 页。

执行 `.cxr 0000004e 78dfa980` 切换线程上下文，然后执行 `k` 命令查看调用栈。

![change-context-and-view-callstack](https://bbs.kanxue.com/upload/attach/202310/873494_XX34B6S4XWYGWNQ.webp)

从上图红框高亮部分可以看出，在读取地址 `0x000001bc 12e12052` 时出错了，导致异常的指令地址是 `0x00007ffd bbde9863`。与 `_EXCEPTION_RECORD` 中的信息完全吻合。

上面的猜测应该是正确的。此时，突然想起早些时候也查过一个类似的问题。当时在看雪群里问了一下，有一位大佬说 `esp` 指向了异常上下文，当时没往心里去。

我想我肯定忘不了这个至关重要的规律了！

## 尝试破案

执行 `!address 0x1bc12e12052` 查看地址属性，可以发现，该地址对应的页面确实不可访问。

![view-address-page-information](https://bbs.kanxue.com/upload/attach/202310/873494_AJYCZDGGP5RD8PM.webp)

本想继续调查一下具体原因，奈何没有对应的调试符号，只简单查看了一下反汇编代码，没有特别的发现。由于手头 `bug` 比较多，而且没有调试符号，查起来会比较耗时间，剩下的工作就留给平台同事继续调查吧。

## KiUserExceptionDispatcher 伪代码

通过前面的反汇编代码，结合在 `IDA` 中对 `KiUserExceptionDispatcher()` 进行 `F5` 结果，伪代码整理如下：

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19 | `void` `KiUserExceptionDispatcher(EXCEPTION_RECORD* exceptionRecord, CONTEXT* contextRecord)`  `{`  `if` `(Wow64PrepareForException)`  `{`  `Wow64PrepareForException(exceptionRecord, contextRecord);`  `}`    `DWORD` `result;`  `if` `(RtlDispatchException(exceptionRecord, contextRecord) )`  `{`  `result = RtlGuardRestoreContext(contextRecord, 0);`  `}`  `else`  `{`  `result = ZwRaiseException(exceptionRecord, contextRecord, FALSE);`  `}`    `RtlRaiseStatus(result);`  `}` |

> **疑问：** `windbg` 调用栈中给出的名字是 `ntdll!KiUserExceptionDispatch`，少了 `er`（好像也可以用 `ntdll!KiUserExceptionDispatcher`），在 `IDA` 中对应的函数名字是 `ntdll!KiUserExceptionDispatcher`。

## 总结

- `64` 位程序中 `KiUserExceptionDispatch()` 对应栈帧的 `Child-SP` 保存了 `_CONTEXT` 参数，`rsp+4f0` 处保存了 `_EXCEPTION_RECORD` 参数，可以根据这个规律直接找到异常发生时的线程上下文信息及异常信息。

- 当直接通过 `k` 命令得到的调用栈不符合预期时，可以通过 `.cxr context` 切换到 `context` 指定的线程上下文后再次尝试。

- 可以通过 `?? sizeof(struct_name)` 查看某个结构体的大小。

- 可以通过 `!address addr` 查看某个地址对应的页面属性。

## 参考资料

- 《软件调试》第一版 第 `12` 章 未处理异常和 `JIT` 调试

- `RtlDispatchException()` [314K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3y4G2k6r3g2%4j5i4u0J5K9h3!0J5i4K6u0W2j5$3&6Q4x3V1k6F1N6r3c8G2j5#2)9J5c8Y4N6A6L8X3&6@1i4K6u0r3M7Y4c8D9i4K6u0r3L8h3W2H3M7#2)9J5c8W2u0@1L8p5c8A6M7%4m8S2N6r3y4Z5c8i4S2U0k6i4m8@1K9h3!0F1i4K6u0W2K9s2c8E0](https://bbs.kanxue.com/elink@b99K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3y4G2k6r3g2%4j5i4u0J5K9h3!0J5i4K6u0W2j5$3&6Q4x3V1k6F1N6r3c8G2j5#2)9J5c8Y4N6A6L8X3&6@1i4K6u0r3M7Y4c8D9i4K6u0r3L8h3W2H3M7#2)9J5c8W2u0@1L8p5c8A6M7%4m8S2N6r3y4Z5c8i4S2U0k6i4m8@1K9h3!0F1i4K6u0W2K9s2c8E0)

- 调试笔记之VTUNE崩溃 [df2K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3q4V1N6X3c8T1k6#2)9J5k6h3!0J5k6#2)9J5c8X3u0D9L8$3N6K6i4K6u0r3j5h3c8$3k6r3u0Y4i4K6g2X3M7%4W2K6N6r3g2E0i4K6u0r3j5i4u0@1K9h3y4D9k6i4y4Q4x3V1j5%4x3o6j5K6i4K6u0W2j5i4y4H3P5l9`.`.](https://bbs.kanxue.com/elink@aefK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3q4V1N6X3c8T1k6#2)9J5k6h3!0J5k6#2)9J5c8X3u0D9L8$3N6K6i4K6u0r3j5h3c8$3k6r3u0Y4i4K6g2X3M7%4W2K6N6r3g2E0i4K6u0r3j5i4u0@1K9h3y4D9k6i4y4Q4x3V1j5%4x3o6j5K6i4K6u0W2j5i4y4H3P5l9`.`.)

## More

`64` 位程序发生异常时，可以使用这个规律查找到异常发生时的线程上下文信息，运行在 `32` 位系统下的程序是否也满足这个规律？运行在 `64` 位系统下的 `32` 位程序呢？敬请期待~

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

[#基础知识](https://bbs.kanxue.com/forum-41-1-130.htm)

[#其他](https://bbs.kanxue.com/forum-41-1-138.htm)
