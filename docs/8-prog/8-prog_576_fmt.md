
# ATL 基础：`BSTR`、`CComBSTR` 与 `SysAllocString`

> 原文：[`www.pediy.com/kssd/pediy11/125705.html`](https://www.pediy.com/kssd/pediy11/125705.html)

ATL 提供了 `BSTR` 和 `CComBSTR`，还有 `OLEAUTO32.DLL` 导出一个 API 叫 `SysAllocString`。这些东西有什么意思？有什么用呢？

`BSTR` 是一个 typedef，你可以理解为 `typedef WCHAR* BSTR`。

它就是一个指针。所以：

```
BSTR p = L"hello";
```

编译是没问题的。不过不推荐这么做。因为按 MSDN，`BSTR` 应该只接收 `SysAllocString` 的返回值。

就是说，第一，`SysAllocString` 的返回值是一个 `BSTR`；第二，如果我们看到一个 `BSTR`，那它应该总从由一个 `SysAllocString` 得到的，它总是应该用 `SysFreeString` 释放。

我们可以想像，`SysAllocString` 实际上做了一个 `malloc` 分配了一块内存，然后把内存长度放第一个 `DWORD`，把这个 `DWORD` 后面位置作为 `BSTR` 返回。因为在 `BSTR` 减 4 的位置已经保存了块长度，所以对 `BSTR` 不应该用 `wcslen` 之类的方法来得到它的长度，要用 `SysStringLen`。这就是为什么不推荐用 `BSTR p = L"hello"` 这种方法直接给 `BSTR` 赋值，因为这样得到的 `BSTR` 不符合减 4 位置有块长度的规范，也不能用 `SysStringLen` 得到长度，也不能用 `SysFreeString` 来释放。乱了规矩。

又一个问题，为什么需要 `SysAllocString`？我用 `new` 或 `malloc` 不行吗？因为用 ATL 常常是为了写 COM。而 COM 控件是跨语言的。你用 C++ 写一个 COM 控件，可能被 HTML 中的一个 `VBSCRIPT` 或 `PHP` 调用。你返回的那个 `BSTR` 字串可能需要在 `VBSCRIPT` 中释放。而 `new` 或 `malloc` 等常规的 C++ 分配内存函数，都要求内存在本模块内释放。所以，`SysAllocString` 分配的内存，是为了支持跨模块使用，并在外地 `free` 的。

那么 `CComBSTR` 又是什么呢？上面讲了，`BSTR` 只是一个指针，它太简单了。如果我要实现字串查找，字串合并等复杂的操作就要自己写代码了。于是就有了 `CComBSTR` 这个类。它只有一个成员 `BSTR m_str` 但它有大量的方法函数，足够你所可能需要的所有字串操作。

最后结论：

* `BSTR` 只是一个指针，要小心使用，除了 `SysAllocString` 不要对它赋值。
* 如果发现一个 `BSTR`，那它一定是用 `SysAllocString` 得到的。一定要用 `SysStringLen` 来得到长度。一定要用 `SysFreeString` 来释放它。
* 如果需要对 `BSTR` 做字串操作，把它变成 `CComBSTR`，总有一个现成的函数直接使用。
* 这一系列东西，都是为了 COM 跨模块把字串当作返回值使用的。如果一个字串只在本模块内有效，还是用 `new` 或 `malloc` 或 `CString` 来得简单。
