
# 一种绕过全局钩子安装拦截的思路

> 原文：[`www.pediy.com/kssd/pediy10/92717.html`](https://www.pediy.com/kssd/pediy10/92717.html)

## 1. 核心思路

本文介绍了一种思路，利用安全软件驱动和 WINDOWS 本身处理全局钩子安装过程的不同，以绕过安全软件对全局钩子安装的拦截。这种方法并不保证能通用于任何安全软件。

## 2. 背景分析

### 2.1 安全软件的拦截机制

安全软件，例如 `HIPS`、`AV` 等，通常会安装 `NtUserSetWindowsHookEx` / `NtUserSetWinEventHook` 的钩子，拦截全局钩子注入，防止键盘、消息拦截或者 `DLL` 注入。

在这两个函数的参数中，都存在一个 `pstrlib` 参数，指向了要注册全局钩子的 `DLL` 路径名。这个路径是 `DOS` 路径名。通常安全软件会分配 `BUFFER`，将 `pstrlib` 这个 `UNICODE_STRING` 中的字符串读取出来，再加上 `NT` 路径头（`\\??\\`），然后进行相关处理后传递给 `RING3` 的处理程序。比较常见的例如使用 `RtlCopyUnicodeString`，或者直接 `RtlCopyMemory(LocalName, pstrlib->Buffer, pstrlib->Length)`。

### 2.2 Windows 内部处理逻辑

而 WINDOWS 函数自身，例如 `NtUserSetWindowsHookEx`、`NtUserSetWinEventHook`，他们是怎么处理这个参数的呢？

实际上，这些函数最终会调用内部函数 `GetHmodTableIndex` 将 `DLL` 名加入一个 `ATOM` `Table` 中，并返回一个 `INDEX`，记为 `ihmod`。等到 `CallNextHook` 或者分发 `winevent` `hook` 时，通过 `ihmod` 再取回这个 `index`，得到 `DLL` 名，并通过 `User` `mode` `callback` 调用 `LoadLibrary` 加载这个 `DLL`。

`GetHmodTableIndex` 则会调用一个 `UserAddAtom` 函数来将 `DLL` 名加入 `UserAtomTableHandle` 这个 `atom` 中。`UserAddAtom` 这个函数接收的 `DLL` 名参数，是一个 `LPCWSTR` 类型的字符串，来自 `pstrlib->Buffer`。`UserAddAtom` 并不关心 `pstrlib->Length` 的长度。等到向 `RING3` 程序注入这个 `DLL` 时，调用 `xxxLoadHmodIndex` 中才在 `UserGetAtomName` 获取这个字符串后，通过调用 `RtlInitUnicodeString` 重新计算这个字符串的长度。

## 3. 绕过方法

那么，很简单的方法就可以绕过安全软件的全局钩子拦截了：

1. `length` 传入 `0`，直接绕过检测。
2. `buffer` 为 `c:\\windows\\system32\\msctf.dll`。
   - `length` 传递为该字符串字节长度 - `sizeof(wchar)`。

也就是让安全软件得到错误的系统 `DLL` 名，实际注册的是其他的文件。
