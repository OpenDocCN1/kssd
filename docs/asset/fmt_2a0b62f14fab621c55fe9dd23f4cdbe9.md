
# 写了一个为软件添加 Section 段的工具（附源码）

> 原文：[`www.pediy.com/kssd/pediy11/104430.html`](https://www.pediy.com/kssd/pediy11/104430.html)

最近学习做 CM 和 PE 文件格式，有些时候总是需要添加一个 Section 段来写代码，就写了这么个小工具，没啥技术含量，仅供对 PE 格式一点不懂的菜鸟参考！

用了这段时间发现有 bug，有些 exe 程序加上新的 Section 段后不能运行

修订了一下

### ① 在 `SetPeInfo` 函数中添加代码

在 `SetPeInfo(void *ImageOfPe, DWORD dwFileSize, DWORD dwOffset)` 函数里面添加这样 2 句话：

```
//****************for a bug******************/

pNtHead->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_BOUND_IMPORT].Size = 0;

pNtHead->OptionalHeader.DataDirectory[IMAGE_DIRECTORY_ENTRY_BOUND_IMPORT].VirtualAddress = 0;

//****************for a bug******************/
```

### ② 修改 `SetPeInfo` 函数中的代码

将原来的 `SetPeInfo(void *ImageOfPe, DWORD dwFileSize, DWORD dwOffset)` 函数中的代码：

```
//3.Add 0x200 to SizeofImage

pNtHead->OptionalHeader.SizeOfImage += 0x200;
```

修改为：

```
//3.Add 0x1000 to SizeofImage

pNtHead->OptionalHeader.SizeOfImage += 0x1000;
```

由于已经告知修订的源代码了，所以这里就只上传修订后的可执行程序。

#### 上传的附件

| | |
|---|---|
| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) | PE Section Appended 源代码.rar |
| | |
| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) | Tool 程序.rar |
| | |
| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) | Tool-1.1.rar |
