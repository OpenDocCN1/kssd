
# apLib 的使用

> 原文：[`www.pediy.com/kssd/pediy12/131361.html`](https://www.pediy.com/kssd/pediy12/131361.html)

看了这个使用 `apLib` 的帖子，很有感触[`bbs.pediy.com/showthread.php?t=94815`](http://bbs.pediy.com/showthread.php?t=94815)，把几个常用的 `apLib` 的函数整理了一下，高手飘过，基础的东西。

### 压缩过程

1. 调用 `aP_max_packed_size` 得到存放压缩文件的空间大小
2. 调用 `aP_workmem_size` 得到工作空间大小
3. 调用 `aPsafe_pack`
4. 调用 `WriteFile` 写入文件

### 解压过程

1. 调用 `aPsafe_get_orig_size` 得到解压后文件大小
2. 调用 `aPsafe_depack` 解压
3. 调用 `WriteFile` 写入解压后的文件

### 常用函数解析

`size_t aP_max_packed_size(size_t input_size)`

- **参数**：未压缩的数据大小
- **返回值**：压缩后的数据可能的所需要的最大值

Computes the maximum possible compressed size possible when compressing `input_size` bytes of incompressible data.

`size_t aP_workmem_size(size_t input_size)`

- **说明**：计算 `aP_pack` 函数所需要的 `workmem[]` buffer 的大小
- **参数**：未压缩的数据的字节数
- **返回值**：`work` buffer 所需要的空间长度，总是返回 `640k` (`640*1024`).

`size_t aPsafe_pack(const void *source, void *destination, size_t length, void *workmem, int (*callback)(size_t, size_t, size_t, void *), void *cbparam)`

`aP_pack` 函数的封装函数，添加了一个头部，头部包含了原始数据的大小和原始数据的 CRC 检验值。

- **参数**：
  - `source` - 指向被压缩数据的指针
  - `destination` - 存储压缩数据的指针
  - `length` - 原始数据（未压缩数据）的字节数
  - `workmem` - 压缩时指向工作空间的指针
  - `callback` - 回调函数的指针，也可以为 `NULL`
  - `cbparam` - 回调函数的参数
- **返回值**：成功返回压缩后数据的字节数，失败返回 `APLIB_ERROR`

`size_t aPsafe_get_orig_size(const void *source)`

返回解压后数据的长度。

- **参数**：
  - `source` - 被压缩的数据的指针
- **返回值**：成功解压后的数据长度，失败 `APLIB_ERROR`

`size_t aPsafe_depack(const void *source, size_t srclen, void *destination, size_t dstlen)`

`aP_depack_asm_safe` 函数的封装函数，函数检查压缩数据的 `CRC32` 校验值。

- **参数**：
  - `source` - 指向压缩后数据的指针
  - `srclen` - 压缩后的数据的字节数
  - `destination` - 解压后的数据存放空间的指针
  - `dstlen` - 解压后数据的字节数
- **返回值**：成功返回解压后的数据大小，错误返回 `APLIB_ERROR`

具体的实例还是看：[`bbs.pediy.com/showthread.php?t=94815`](http://bbs.pediy.com/showthread.php?t=94815)

附件为 `apLib` 的使用说明，头文件，库文件。

### 上传的附件

| ![](img/620251b33676a8e4b98ca0ece915ee8d.jpg) || aPLib-1.01.zip |

