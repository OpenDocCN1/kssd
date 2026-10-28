
```cpp
while (StackWalk64(IMAGE_FILE_MACHINE_AMD64, hProcess, hThread, &frame, &context, NULL, SymFunctionTableAccess64, SymGetModuleBase64, NULL))
{
    DWORD64 moduleBase = SymGetModuleBase64(hProcess, frame.AddrPC.Offset);
    DWORD64 offset = 0;
    char symbolBuff[sizeof(SYMBOL_INFO) + MAX_SYM_NAME * sizeof(TCHAR)] = { 0 };
    PSYMBOL_INFO symbol = (PSYMBOL_INFO)symbolBuff;
    symbol->SizeOfStruct = sizeof(SYMBOL_INFO);
    symbol->MaxNameLen = MAX_SYM_NAME;

    if (SymFromAddr(hProcess, frame.AddrPC.Offset, &offset, symbol))
    {
        found.push_back(symbol->Name);
        printf(
            "\\_ Frame %d\n"
            "  |_ Name: %s\n"
            "  |_ Address: 0x%p\n\n",
            idx,
            symbol->Name,
            symbol->Address
        );

        idx++;
    }
}
```

运行代码并找到所有符号后，我们来看看这两个向量是否匹配：

```cpp
if (std::equal(expected.begin(), expected.end(), found.begin())) {
    printf("[ CLEAN ]\n");
}
else {
    printf("[ DIRTY ]\n");
}
```

将其指向正常的线程：

![图片描述](https://bbs.kanxue.com/upload/attach/202308/876202_639EUVSNMTPGFHX.png)

我们得到了`CLEAN`消息。然后，我们来看看有问题的线程：

![图片描述](https://bbs.kanxue.com/upload/attach/202308/876202_TA86USCHV4582Q5.png)

显然，这段代码并不适用于实际生产环境，编写这种逻辑的细节非常具有挑战性。然而，一些EDR供应商已经开始采用这种技术。随着对混淆和蒙蔽终端保护的研究不断增加，这对于蓝队和红队来说都是一个有用的技术。

说到红队，关于修复这个线程问题的研究已经在进行中。

这种技术最初由[Peter Winter-Smith](https://bbs.kanxue.com/elink@3b3K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4k6h3u0Q4x3X3g2S2M7X3y4Z5K9i4k6W2i4K6u0W2L8%4u0Y4i4K6u0r3N6$3g2T1i4K6u0r3x3U0l9J5x3U0p5I4x3U0j5I4z5o6t1#2x3K6y4E0M7q4)9#2k6W2)9J5c8X3S2@1N6s2m8K6i4K6y4m8i4K6u0r3i4K6u0r3N6s2N6A6N6s2c8W2M7W2)9J5k6h3y4G2L8g2)9J5c8Y4m8W2N6r3g2J5N6$3W2F1N6s2u0K6L8h3W2@1K9l9`.`.)首次推广，他在这个领域经常出现，然后由[mgeeky](https://bbs.kanxue.com/elink@d40K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4k6h3u0Q4x3X3g2S2M7X3y4Z5K9i4k6W2i4K6u0W2L8%4u0Y4i4K6u0r3N6$3g2T1i4K6u0r3x3U0l9J5x3U0p5I4x3U0j5I4z5o6t1#2x3K6y4E0M7q4)9#2k6W2)9J5c8X3S2@1N6s2m8K6i4K6y4m8i4K6u0r3i4K6u0r3N6s2N6A6N6s2c8W2M7W2)9J5k6h3y4G2L8g2)9J5c8X3#2S2M7X3W2#2M7%4A6T1K9i4b7`.)在[ThreadStackSpoofer](https://bbs.kanxue.com/elink@b35K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4k6h3u0Q4x3X3g2S2M7X3y4Z5K9i4k6W2i4K6u0W2L8%4u0Y4i4K6u0r3N6$3g2T1i4K6u0r3x3U0l9J5x3U0p5I4x3U0j5I4z5o6t1#2x3K6y4E0M7q4)9#2k6W2)9J5c8X3S2@1N6s2m8K6i4K6y4m8i4K6u0r3i4K6u0r3k6$3W2@1K9s2g2T1i4K6u0W2j5$3!0E0i4K6u0r3L8h3N6W2k6h3E0&6i4K6u0r3g2r3S2J5k6h3q4V1f1%4c8S2j5$3E0e0M7r3!0G2k6X3g2J5)中重新解释。然而，这个概念验证将返回地址设置为0，从而消除了对用于shellcode注入的内存地址的引用。

> 这是一个Thread Stack Spoofing技术的示例实现，旨在逃避对检查的线程调用堆栈中shellcode帧的引用的恶意代码分析员、AV和EDR的检测。其思想是隐藏线程调用堆栈中对shellcode的引用，从而伪装包含恶意代码的分配。

如果我们移除Vulpes中的休眠遮蔽，调用堆栈将如下所示：

![图片描述](https://bbs.kanxue.com/upload/attach/202308/876202_RJ8TGQ77UAEAH9C.png)

这种技术旨在通过将返回地址存储到一个变量中，将返回地址设置为0，然后恢复返回地址来掩盖这些地址。

以下是来自上述代码库的一个快速示例代码：

```cpp
void WINAPI MySleep(DWORD _dwMilliseconds)
{
    [...]
    auto overwrite = (PULONG_PTR)_AddressOfReturnAddress();
    const auto origReturnAddress = *overwrite;
    *overwrite = 0;

    [...]
    *overwrite = origReturnAddress;
}
```

在最近的一个项目中，[William Burgess](https://bbs.kanxue.com/elink@634K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4k6h3u0Q4x3X3g2S2M7X3y4Z5K9i4k6W2i4K6u0W2L8%4u0Y4i4K6u0r3N6$3g2T1i4K6u0r3x3U0l9J5x3U0p5I4x3U0j5I4z5o6t1#2x3K6y4E0M7q4)9#2k6W2)9J5c8X3S2@1N6s2m8K6i4K6y4m8i4K6u0r3i4K6u0r3N6s2N6A6N6s2c8W2M7W2)9J5k6h3y4G2L8g2)9J5c8X3A6G2k6h3S2G2N6%4N6G2L8r3j5`.)创建了[CallStackSpoofer](https://bbs.kanxue.com/elink@9c0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4k6h3u0Q4x3X3g2S2M7X3y4Z5K9i4k6W2i4K6u0W2L8%4u0Y4i4K6u0r3N6$3g2T1i4K6u0r3x3U0l9J5x3U0p5I4x3U0j5I4z5o6t1#2x3K6y4E0M7q4)9#2k6W2)9J5c8X3S2@1N6s2m8K6i4K6y4m8i4K6u0r3i4K6u0r3k6$3W2@1K9s2g2T1i4K6u0W2j5$3!0E0i4K6u0r3j5$3!0#2L8Y4c8W2M7X3y4W2M7s2c8Q4x3V1k6o6j5h3I4D9f1%4c8S2j5$3E0e0M7r3!0G2k6X3g2J5)，将这一技术推向更高的水平，完全伪造堆栈。

参见：[Spoofing Call Stacks to confused EDRs](https://bbs.kanxue.com/elink@cd2K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4k6h3u0Q4x3X3g2S2M7X3y4Z5K9i4k6W2i4K6u0W2L8%4u0Y4i4K6u0r3N6$3g2T1i4K6u0r3x3U0l9J5x3U0p5I4x3U0j5I4z5o6t1#2x3K6y4E0M7q4)9#2k6W2)9J5c8X3S2@1N6s2m8K6i4K6y4m8i4K6u0r3i4K6u0r3L8r3q4T1M7#2)9J5k6i4N6A6N6r3S2K6k6h3y4#2M7X3g2Q4x3X3g2U0L8$3#2Q4x3V1k6T1L8r3!0Y4i4K6u0r3M7%4m8G2L8$3k6A6L8X3N6Q4x3X3c8U0j5h3I4D9i4K6u0V1M7%4c8S2j5$3E0K6i4K6u0V1N6r3!0Q4x3X3c8U0L8$3&6X3N6i4y4W2i4K6u0V1k6h3c8J5M7H3`.`.)

通过使用预定义的堆栈向量，该项目能够模拟：

- WMI
- RPC
- SVCHost

以下是[WMI 的预定义容器](https://bbs.kanxue.com/elink@7c3K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4k6h3u0Q4x3X3g2S2M7X3y4Z5K9i4k6W2i4K6u0W2L8%4u0Y4i4K6u0r3N6$3g2T1i4K6u0r3x3U0l9J5x3U0p5I4x3U0j5I4z5o6t1#2x3K6y4E0M7q4)9#2k6W2)9J5c8X3S2@1N6s2m8K6i4K6y4m8i4K6u0r3i4K6u0r3k6$3W2@1K9s2g2T1i4K6u0W2j5$3!0E0i4K6u0r3j5$3!0#2L8Y4c8W2M7X3y4W2M7s2c8Q4x3V1k6o6j5h3I4D9f1%4c8S2j5$3E0e0M7r3!0G2k6X3g2J5i4K6u0r3j5X3I4G2j5W2)9J5c8U0l9%4y4U0j5%4x3X3f1J5z5h3g2W2y4U0R3H3z5o6k6S2y4U0l9%4k6o6W2X3x3r3u0V1k6e0u0U0k6o6t1J5y4K6f1@1j5K6y4S2j5K6q4Q4x3V1k6h3N6h3I4U0j5h3&6d9j5i4k6W2L8W2)9J5c8W2k6#2L8r3y4S2L8W2u0S2N6X3g2F1i4K6u0W2j5%4m8H3i4K6t1K6e0o6p5I4z5b7`.`.)示例：

```cpp
std::vector<StackFrame> wmiCallStack =
{
    StackFrame(L"C:\\Windows\\SYSTEM32\\kernelbase.dll", 0x2c13e, 0, FALSE),
    StackFrame(L"C:\\Windows\\Microsoft.NET\\Framework64\\v4.0.30319\\CorperfmonExt.dll", 0xc669, 0, TRUE),
    StackFrame(L"C:\\Windows\\Microsoft.NET\\Framework64\\v4.0.30319\\CorperfmonExt.dll", 0xc71b, 0, FALSE),
    StackFrame(L"C:\\Windows\\Microsoft.NET\\Framework64\\v4.0.30319\\CorperfmonExt.dll", 0x2fde, 0, FALSE),
    StackFrame(L"C:\\Windows\\Microsoft.NET\\Framework64\\v4.0.30319\\CorperfmonExt.dll", 0x2b9e, 0, FALSE),
    StackFrame(L"C:\\Windows\\Microsoft.NET\\Framework64\\v4.0.30319\\CorperfmonExt.dll", 0x2659, 0, FALSE),
    StackFrame(L"C:\\Windows\\Microsoft.NET\\Framework64\\v4.0.30319\\CorperfmonExt.dll", 0x11b6, 0, FALSE),
    StackFrame(L"C:\\Windows\\Microsoft.NET\\Framework64\\v4.0.30319\\CorperfmonExt.dll", 0xc144, 0, FALSE),
    StackFrame(L"C:\\Windows\\SYSTEM32\\kernel32.dll", 0x17034, 0, FALSE),
    StackFrame(L"C:\\Windows\\SYSTEM32\\ntdll.dll", 0x52651, 0, FALSE),
};
```

通过实施这种类型的技术，将极大地增加了实施我们之前展示的调用堆栈完整性检查的难度（尽管我们的演示中使用的是硬编码的值，但观点仍然成立）。

## 结论

这是一篇相当长的帖子，我们试图为EDR（终端检测与响应）机制提供一些清晰的解释，以便不仅能够识别恶意活动，而且能够预防它。在此过程中，我们讨论了常见的问题和一些增强措施，以保护系统免受绕过攻击的影响。

在这个过程中，我们试图更多地揭示“X绕过EDR”的故事情节，虽然攻击可能暂时逃避检测，但通常还是会留下活动的日志记录。

下一集将讨论ETW（事件跟踪）和AMSI（反恶意软件扫描接口）！

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

最后于  2023-8-28 14:47

被Max\_hhg编辑

，原因： 格式优化

[#基础知识](https://bbs.kanxue.com/forum-41-1-130.htm)
