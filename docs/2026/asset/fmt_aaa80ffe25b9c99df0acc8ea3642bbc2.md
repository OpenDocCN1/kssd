
# [原创]为未公开 API 生成 Lib 并与 MSBuild 集成（一）

原文链接（GitHub 的 Markdown 渲染效果更佳）：<a href="elink@d7bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6w2e0W2y4G2k6Y4c8Q4x3V1k6n7L8r3!0Y4i4K6u0r3N6s2u0W2k6g2)9J5c8X3#2S2K9h3&6Q4x3V1k6i4K9h3&6V1L8%4N6K6i4@1f1%4i4@1u0o6i4K6V1$3i4@1f1%4i4@1p5^5i4K6S2n7i4K6u0r3i4@1f1@1i4@1t1^5i4@1u0m8i4@1f1$3i4K6W2o6i4@1q4m8i4@1f1#2i4K6R3#2i4@1q4o6i4@1f1#2i4@1u0o6i4K6R3H3b7g2m8u0i4@1f1%4i4K6V1@1i4K6W2r3i4@1f1$3i4K6R3^5i4K6V1H3e0r3W2T1i4@1f1#2i4@1t1&6i4@1t1$3i4@1f1@1i4@1t1^5i4K6S2q4e0g2y4n7N6h3W2D9k6q4!0q4z5g2)9&6b7W2)9^5y4W2!0q4y4W2)9^5z5q4)9&6x3q4!0q4c8W2!0n7b7#2)9^5z5q4!0q4y4q4!0n7z5q4)9^5x3q4!0q4c8W2!0n7b7#2)9^5z5q4!0q4y4q4!0n7z5q4)9^5x3q4!0q4c8W2!0n7b7#2)9^5z5b7`.`.">为未公开 API 生成 Lib 并与 MSBuild 集成（一）</a>

本文从 Lib 文件格式结构开始，描述 Lib 文件如何包含 Dll 的导出函数信息以及夹带 Obj 文件供链接，并从零构造 Lib，使得其包含任意未公开 API 导出函数信息。最后与 <a href="elink@bbcK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6D9k6h3q4J5L8W2)9J5k6h3#2A6j5%4u0G2M7$3!0X3N6q4)9J5k6h3y4G2L8g2)9J5c8X3g2F1i4K6u0V1N6i4y4Q4x3V1k6$3K9i4y4#2j5h3I4K6N6s2g2V1K9h3!0Q4x3V1k6E0M7$3u0#2K9h3I4V1i4K6u0r3L8i4y4T1N6h3W2D9k6l9`.`.">MSBuild</a> 集成，衔接到 Visual Studio C/C++ 项目中完成闭环。

注：本文“未公开 API”均特指 Windows SDK 中连 Lib 文件也没有其导入符号的未公开 API。

## 一、问题背景

以我们熟悉的 `CreateProcessInternalW` 为例，它由 `Kernel32.dll` 导出。Windows SDK 里没有它的声明，Kernel32.lib 里也没有它的符号。如果想调用，一般使用动态加载或生成 Stub Dll 借用其 Lib 的方式。

### 1. 动态加载

按照函数定义，定义其指针类型（参数略），然后通过 `LoadLibaray` + `GetProcAddress` 之类的方式动态加载 Dll 和寻址：

```
typedef BOOL(WINAPI* PFNCreateProcessInternalW)(...);
NTSTATUS Status;
HMODULE Kernel32Handle;
PFNCreateProcessInternalW pfnCreateProcessInternalW;
Status = LdrLoadDll(NULL, NULL, &(UNICODE_STRING)RTL_CONSTANT_STRING(L"Kernel32.dll"), &Kernel32Handle);
if (NT_SUCCESS(Status))
{
    Status = LdrGetProcedureAddress(
        Kernel32Handle,
        &(ANSI_STRING)RTL_CONSTANT_STRING("CreateProcessInternalW"),
        0,
        &pfnCreateProcessInternalW);
    if (NT_SUCCESS(Status))
    {
        // pfnCreateProcessInternalW(...)
    }
    // LdrUnloadDll(Kernel32Handle);
}
```

> PS：如果想像以上 Demo 代码一样在 UM 使用 KM 及未公开定义，可以关注 <a href="elink@99dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2K6P5i4y4@1k6h3#2A6L8X3k6G2M7X3#2W2M7W2)9J5k6h3y4G2L8g2)9J5c8R3`.`.">SystemInformer (ProcessHacker)</a> 的 <a href="elink@cdcK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6%4K9h3&6K6K9h3c8W2M7Y4y4K6i4K6u0r3M7%4W2K6N6r3g2E0K9h3&6X3L8%4u0E0k6i4u0Q4x3V1k6@1M7X3g2W2i4K6u0r3L8h3q4K6N6r3g2J5i4K6u0r3M7r3S2F1N6l9`.`.">phnt</a>，我也积攒下了 <a href="elink@56dK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6w2e0W2y4G2k6Y4c8Q4x3V1k6i4K9h3&6@1k6i4S2H3L8%4u0@1M7H3`.`.">Wintexports</a> 自用。

显然，对于支持目标系统里必然存在的函数，远不及直接调用 `CreateProcessInternalW(...)` 方便，**调用未公开 API 的数量越多，动态加载的代码越冗杂繁复**，于是考虑下面的办法。

### 2. 生成 Stub Dll 借用其 Lib

编写 Stub Dll，其具有除实现为空外完全相同的函数定义（参数略）：

```
BOOL
WINAPI
CreateProcessInternalW(...)
{
    return FALSE;
}
```

注意，`CreateProcessInternalW` 和绝大多数 Windows API 一样使用 `stdcall` 调用约定，x86 下 Lib 的导入符号名称与 Dll 导出函数名称不一样，前者带有调用约定的修饰，而后者没有，要用<a href="elink@880K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6D9k6h3q4J5L8W2)9J5k6h3#2A6j5%4u0G2M7$3!0X3N6q4)9J5k6h3y4G2L8g2)9J5c8X3g2F1i4K6u0V1N6i4y4Q4x3V1k6U0M7s2m8Q4x3V1k6T1N6h3W2D9k6q4)9J5c8Y4u0W2k6X3g2J5k6h3&6U0k6g2)9J5c8X3#2G2k6s2g2D9k6g2)9J5k6r3c8W2k6X3W2F1K9i4c8A6L8$3&6Q4x3X3c8V1L8%4c8Q4x3X3c8V1k6h3k6Q4x3X3c8X3K9h3I4W2M7H3`.`.">模块定义文件（.def）</a>定义其导出：

```
LIBRARY KERNEL32
EXPORTS
CreateProcessInternalW
```

最后编译 Stub Dll，补充函数定义，取其一同生成的 Lib，便能直接调用 `CreateProcessInternalW(...)` 这般调用了。

一眼看去，似乎仍有改进的空间，重点在于 Lib 的生成：

1. 首先，理论上我们只需要 Dll 名称和 API 名称（包含它的修饰名，x86 的 stdcall 要用）、空函数体以及<a href="elink@6f8K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6D9k6h3q4J5L8W2)9J5k6h3#2A6j5%4u0G2M7$3!0X3N6q4)9J5k6h3y4G2L8g2)9J5c8X3g2F1i4K6u0V1N6i4y4Q4x3V1k6U0M7s2m8Q4x3V1k6T1N6h3W2D9k6q4)9J5c8Y4u0W2k6X3g2J5k6h3&6U0k6g2)9J5c8X3#2G2k6s2g2D9k6g2)9J5k6r3c8W2k6X3W2F1K9i4c8A6L8$3&6Q4x3X3c8V1L8%4c8Q4x3X3c8V1k6h3k6Q4x3X3c8X3K9h3I4W2M7H3`.`.">模块定义文件（.def）</a> 是信息冗余的；
2. 其次，理论上没有任何函数实现，完全不需要在生成过程中将编译器卷进来浪费时间；
3. 最次，Lib 格式的本质是档案，里面可以花式夹带 Dll 导出符号和目标文件。可以根据需要，同时包含多个 Dll 的导出符号，或者一对一地为每个未公开 API 所在 Dll 生成一个 Lib。

这些“改进”太微不足道，但有时候我就这么点追求。先参考一些已有的工具：

- <a href="elink@f52K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6J5k6h3q4U0N6r3!0K6i4K6u0W2L8%4u0Y4i4K6u0r3">ReactOS</a> 的 <a href="elink@fcdK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6J5k6h3q4U0N6r3!0K6i4K6u0r3M7X3g2S2j5%4c8G2M7#2)9J5c8X3u0D9L8$3u0Q4x3V1k6E0j5i4y4@1k6i4u0Q4x3V1k6K6k6r3E0Q4x3V1k6@1L8$3!0D9M7#2)9J5c8Y4y4H3k6h3x3J5k6r3g2X3i4K6u0r3M7%4m8W2j5K6u0V1k6h3k6Q4x3X3g2U0">spec2def</a> 生成工具。

  它根据自定的 spec 文件描述函数原型生成 Lib。spec 比起手写空函数体加配套 <a href="elink@8e2K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6D9k6h3q4J5L8W2)9J5k6h3#2A6j5%4u0G2M7$3!0X3N6q4)9J5k6h3y4G2L8g2)9J5c8X3g2F1i4K6u0V1N6i4y4Q4x3V1k6U0M7s2m8Q4x3V1k6T1N6h3W2D9k6q4)9J5c8Y4u0W2k6X3g2J5k6h3&6U0k6g2)9J5c8X3#2G2k6s2g2D9k6g2)9J5k6r3c8W2k6X3W2F1K9i4c8A6L8$3&6Q4x3X3c8V1L8%4c8Q4x3X3c8V1k6h3k6Q4x3X3c8X3K9h3I4W2M7H3`.`.">模块定义文件（.def）</a> 要简单得多：

```
@ stdcall CreateProcessInternalW(ptr wstr wstr ptr ptr long long ptr wstr ptr ptr long)
```

据此生成汇编源文件，再生成 Stub Dll + Lib。卷入汇编器总比卷入 C 编译器生成得快。

- <a href="elink@bf5K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2E0j5i4y4E0x3K6u0Q4x3X3g2U0L8$3#2Q4x3V1j5`.">MASM32</a> 的 `inc2l` 工具。

  它根据汇编 inc 文件里的过程声明生成汇编源文件，然后生成 Stub Dll + Lib。

- <a href="elink@f31K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6D9k6h3q4J5L8W2)9J5k6h3#2A6j5%4u0G2M7$3!0X3N6q4)9J5k6h3y4G2L8g2)9J5c8X3g2F1i4K6u0V1N6i4y4Q4x3V1k6U0M7s2m8Q4x3V1k6T1N6h3W2D9k6q4)9J5c8Y4u0W2k6X3g2J5k6h3&6U0k6b7`.`.">MSVC 库管理器（Lib.exe）</a>。

  它虽然可以根据<a href="elink@1a1K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6D9k6h3q4J5L8W2)9J5k6h3#2A6j5%4u0G2M7$3!0X3N6q4)9J5k6h3y4G2L8g2)9J5c8X3g2F1i4K6u0V1N6i4y4Q4x3V1k6U0M7s2m8Q4x3V1k6T1N6h3W2D9k6q4)9J5c8Y4u0W2k6X3g2J5k6h3&6U0k6g2)9J5c8X3#2G2k6s2g2D9k6g2)9J5k6r3c8W2k6X3W2F1K9i4c8A6L8$3&6Q4x3X3c8V1L8%4c8Q4x3X3c8V1k6h3k6Q4x3X3c8X3K9h3I4W2M7H3`.`.">模块定义文件（.def）</a> 直接生成 Lib，但其生成导入符号的名称类型总指定为 `IMPORT_OBJECT_NAME`，即表示导入名称与导入符号名称完全相同，不适用于 x86 下 stdcall 调用约定的 API（此场景下应使用 `IMPORT_OBJECT_NAME_UNDECORATE`，表示导入名称为导入符号名称去除调用约定的修饰后的结果）。

比起以上已有的轮子还能进一步卷的，是做到能用<a href="elink@72cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6J5k6h3q4U0N6r3!0K6i4K6u0r3M7X3g2S2j5%4c8G2M7#2)9J5c8X3u0D9L8$3u0Q4x3V1k6E0j5i4y4@1k6i4u0Q4x3V1k6K6k6r3E0Q4x3V1k6@1L8$3!0D9M7#2)9J5c8Y4y4H3k6h3x3J5k6r3g2X3i4K6u0r3M7%4m8W2j5K6u0V1k6h3k6Q4x3X3g2U0">spec2def</a> 这样简洁的函数定义，还能不卷入汇编器/编译器这类代码生成工具，甚至还能实现一个 Lib 包含多个 Dll 的导出符号。不妨从文件结构开始，从零构建 Lib。

## 二、生成包含 Dll 导出符号的 Lib

### 1. Lib 文件结构

先祭出 Microsoft Learn 的祖传文档——<a href="elink@3bbK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6D9k6h3q4J5L8W2)9J5k6h3#2A6j5%4u0G2M7$3!0X3N6q4)9J5k6h3y4G2L8g2)9J5c8X3g2F1i4K6u0V1N6i4y4Q4x3V1k6%4K9h3&6V1L8%4N6K6i4K6u0r3N6$3W2F1x3K6u0Q4x3V1k6V1k6h3u0#2k6#2)9J5c8Y4m8W2i4K6u0V1k6X3!0J5L8h3q4@1i4K6t1K6j5i4u0U0K9r3W2$3k6g2)9J5k6r3I4A6j5Y4u0S2M7Y4W2Q4x3X3c8X3K9h3I4W2i4K6u0V1k6X3!0J5L8h3q4@1">PE Format - Archive (Library) File Format - Win32 apps | Microsoft Learn</a>。

我已将踩过的坑对应的文档错漏提交给 <a href="elink@454K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6y4K9h3y4J5L8%4y4G2k6Y4c8p5L8$3y4K6">Microsoft Docs</a> 并得到合入（<a href="elink@1b7K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6y4K9h3y4J5L8%4y4G2k6Y4c8p5L8$3y4K6i4K6u0r3N6$3W2F1x3K6u0Q4x3V1k6H3N6h3I4D9i4K6u0r3x3e0f1#2y4b7`.`.">PR #1555</a>、<a href="elink@a17K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6y4K9h3y4J5L8%4y4G2k6Y4c8p5L8$3y4K6i4K6u0r3N6$3W2F1x3K6u0Q4x3V1k6H3N6h3I4D9i4K6u0r3x3e0f1^5y4R3`.`.">PR #1586</a>、<a href="elink@c2bK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6y4K9h3y4J5L8%4y4G2k6Y4c8p5L8$3y4K6i4K6u0r3N6$3W2F1x3K6u0Q4x3V1k6H3N6h3I4D9i4K6u0r3x3e0j5J5x3R3`.`.">PR #1622</a>），此外需要注意的本文会提及。与某些单纯复制或者翻译微软文档就拿来发的博客相比，算是实践出真知了。

**Lib（Archive）文件主要包含导入符号（Import Symbol），供链接器链接。这些符号可以指向外部 Dll 的导出，也可以存在于 Lib 自身夹带的 Obj 文件里。**

整体布局比较简单。开头固定的文件签名后，一系列成员（Archive Member）依次排列，每个成员都以成员头（`IMAGE_ARCHIVE_MEMBER_HEADER`）开始，后跟成员数据。

![](upload/attach/202307/853701_WVNE465XHGGBXJM.png)

- Archive Member Header（成员头）
