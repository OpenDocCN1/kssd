
# 导入表的获取与动态填充：初步学习笔记

> 原文：[www.pediy.com/kssd/pediy09/pediy09-466.htm](https://www.pediy.com/kssd/pediy09/pediy09-466.htm)

## 1. 导入表结构的获取

### 1. 获取 PE 基址
`IMAGE_DOS_HEADER`

### 2. 获取 PE 头部
`IMAGE_NT_HEADERS32`

### 3. 获取导入表的存储结构
`IMAGE_NT_HEADERS32.OptionalHeader.DataDirectory[1]`

- 导出表：`IMAGE_NT_HEADERS32.OptionalHeader.DataDirectory[0]`
- 导入表：`IMAGE_NT_HEADERS32.OptionalHeader.DataDirectory[1]`

结构其实就是一个 `IMAGE_DATA_DIRECTORY`，包含导入表的相对偏移和导入表的大小。

### 4. 导入表定位
`IMAGE_DATA_DIRECTORY.VirtualAddress + PE 基址`（第一步）

导入表第一个模块结构：

### 5. 获取第一个导入模块
- 结构：`IMAGE_IMPORT_DESCRIPTOR`
- 长度：20 字节
- 是一个结构表，最后以一个空结构结束
- 意思就是：
  - `IMAGE_IMPORT_DESCRIPTOR`: `kernel32.dll`
  - `IMAGE_IMPORT_DESCRIPTOR`: `user32.dll`
  - `IMAGE_IMPORT_DESCRIPTOR`: `NULL`

### 6. 获取导入模块下的函数结构
- 结构：`IMAGE_IMPORT_DESCRIPTOR`
- 长度：20 字节
- 结构获取：`IMAGE_IMPORT_DESCRIPTOR` 结构下有两个成员数指向函数结构体：
  - `OriginalFirstThunk`：序号获取 `IMAGE_THUNK_DATA` 函数结构
  - `FirstThunk`：直接获取函数序号
  - 两个都指向 `IMAGE_THUNK_DATA` 结构
  - 注意：如果两个都为空，表明这个模块已经被“榨干”了~_^
- `IMAGE_THUNK_DATA` 结构和 `IMAGE_IMPORT_DESCRIPTOR` 结构排序方式差不多
- 也是以...

### 7. 函数结构体
`IMAGE_THUNK_DATA`

结构体长度 20 字节 (`5 * sizeof(DWORD)`)。其中 `PIMAGE_IMPORT_BY_NAME` 指向函数名称结构 `IMAGE_IMPORT_BY_NAME`，该结构分为：
- `WORD Hint;`：函数序号，一般为空（除非有的时候一些函数是没有名称的，例如..._^）
- `BYTE Name[n];`：这是一个数组，不是字符指针了，就是函数名称

## 2. 导入表填充

### 8. 导入表填充
`IMAGE_THUNK_DATA`

主要结构体：`IMAGE_THUNK_DATA`。

`IMAGE_THUNK_DATA` 下只有一个指针指向 `IMAGE_IMPORT_BY_NAME`。`IMAGE_IMPORT_BY_NAME` 其实就是 `|函数导出序号|函数名|`。

`PE Loader` 在加载的时候会根据 `IMAGE_IMPORT_BY_NAME` 结构里面的序号或者函数名称所引函数，获取地址，然后填充 `IMAGE_THUNK_DATA`。没错，把 `IMAGE_THUNK_DATA` 给填充了。由于是把 `IMAGE_THUNK_DATA` 结构填充了，意思也就是说指向 `IMAGE_IMPORT_BY_NAME` 的指针被填充了。那也就是说我们只要填充完毕地址以后，`IMAGE_IMPORT_BY_NAME` 结构有没有都无所谓了。这也就解释了为什么加壳软件导入表动态填充完毕以后 `PE` 也不会出错的原因。

另外，为什么不需要修改原有导入表虚拟指向？一般函数调用都是直接 `call [offset MessageBoxA]`，类似这样的结构，翻译成显示地址就是 `call [0x40FFFF]` 这样的字样。他不是从导入表中定位的，所以你即使把整个 `PE Header` 破坏了也没什么事情。

Oh~~My Goddess~~~

### 9. 普通动态填充与高级动态填充

**普通保护：**
1. 解压释放所有资源和数据
2. 获取原导入表虚拟偏移
3. 获取导入表模块指针、地址
4. 进行填充

**特别保护：**
1. 解压释放数据资源
2. 导入表部分是空的（在加密的时候就被清 `NULL` 了）
3. 填充。至于他的导入表是存储在什么地方的那就不太相同了

不过您如果真是没事作的时候可以跟踪一下。至于 API 重定向之类的资料暂时没有找到比较全的，继续学习。

### 10. 其他
导入表部分解决了，下面开始看导出表部分...

`4dd466d0e1e0cc123afbe504da2462ca`
