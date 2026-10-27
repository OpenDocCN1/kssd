
`upx_test_overlap`函数接受压缩数据和解压缩数据的内存区域，以及预期的解压缩数据的大小等参数。然后，它会调用相应的`*_test_overlap`函数（例如`upx_lzma_test_overlap`、`upx_ucl_test_overlap`等），来测试给定的解压缩方法是否会导致数据覆盖。如果测试发现会发生数据覆盖，那么`upx_test_overlap`函数会返回一个错误代码。否则，它会返回一个表示成功的代码。

### compress algorithm

关于`upx`压缩算法部分，在`src/compress`目录下，`upx`主要使用了下面几种压缩算法：

- `LZMA` 是 `Lempel–Ziv–Markov chain` 算法，它属于字典编码类压缩算法，压缩率较高。
- `UCL` 是 `Universal Codec Library`，也是一个字典编码类算法。
- `Zlib` 是 `DEFLATE`算法的实现，也属于字典编码类压缩算法。
- `Zstandard(Zstd)` 是 Facebook开发的压缩算法，它采用哈希函数的技术加快搜索过程。

这四种算法各有优劣：

- `LZMA` 和 `UCL` 压缩率较高，但速度相对慢一些。适合打包桌面软件使用。
- `Zlib` 和 `Zstd` 压缩率在中等水平，但速度快,适合网络传输。

在代码中，`upx` 对算法进行打包，根据 `method` 参数的不同，选择调用相应压缩算法的实现，例如在`src/compress/compress.cpp`：

```c
if (M_IS_LZMA(method)) {
r = upx_lzma_compress(...); // 使用LZMA算法
}
else if (M_IS_ZSTD(method)) {
r = upx_zstd_compress(...); // 使用Zstd算法
}
```

什么是字典编码类压缩算法？字典编码类压缩算法是一类基于重复字符串匹配的压缩算法，它的主要思想是：

1. 构建一个字符串到编码的映射（字典）
2. 在要压缩的数据流中查找重复的字符串
3. 将重复的字符串替换为字典中的编码

典型的字典编码类压缩算法有:`LZ77`、`LZ78`、`LZW`、`DEFLATE` 等。具体的做法是：

- 先将光标停在要压缩的数据流的起点
- 查找一个没有在字典中出现过的字符串
- 将这个字符串添加到字典中并分配一个编码
- 将这个编码写到压缩的数据流中
- 光标后移到该字符串结束点
- 重复上述过程直到压缩完所有数据

这样可以实现压缩的效果：

- 原始字符串 ==> 较短的编码字符串

在这篇文章算法部分不详细展开，后续可能会进行进一步分析。

## Loader

### p_w64pe_amd64.cpp: buildLoader

这里我们要回到`p_w64pe_amd64.cpp`的源码中进行分析，下面是`p_w64pe_amd64`中`PackW64PeAmd64`成员函数`buildLoader`：

```c
void PackW64PeAmd64::buildLoader(const Filter *ft) {
// recompute tlsindex (see pack() below)
unsigned tmp_tlsindex = tlsindex;
const unsigned oam1 = ih.objectalign - 1;
const unsigned newvsize = (ph.u_len + rvamin + ph.overlap_overhead + oam1) & ~oam1;
if (tlsindex && ((newvsize - ph.c_len - 1024 + oam1) & ~oam1) > tlsindex + 4)
tmp_tlsindex = 0;
// prepare loader
initLoader(stub_amd64_win64_pe, sizeof(stub_amd64_win64_pe), 2);
addLoader("START");
if (ih.entry && isdll)
addLoader("PEISDLL0");
if (isefi)
addLoader("PEISEFI0");
addLoader(isdll ? "PEISDLL1" : "", "PEMAIN01",
icondir_count > 1 ? (icondir_count == 2 ? "PEICONS1" : "PEICONS2") : "",
tmp_tlsindex ? "PETLSHAK" : "", "PEMAIN02",
// ph.first_offset_found == 1 ? "PEMAIN03" : "",
M_IS_LZMA(ph.method) ? "LZMA_HEAD,LZMA_ELF00,LZMA_DEC20,LZMA_TAIL"
: M_IS_NRV2B(ph.method) ? "NRV_HEAD,NRV2B"
: M_IS_NRV2D(ph.method) ? "NRV_HEAD,NRV2D"
: M_IS_NRV2E(ph.method) ? "NRV_HEAD,NRV2E"
: "UNKNOWN_COMPRESSION_METHOD",
// getDecompressorSections(),
/*multipass ? "PEMULTIP" : */ "", "PEMAIN10");
addLoader(tmp_tlsindex ? "PETLSHAK2" : "");
if (ft->id) {
const unsigned texv = ih.codebase - rvamin;
assert(ft->calls > 0);
addLoader(texv ? "PECTTPOS" : "PECTTNUL");
addLoader("PEFILTER49");
}
if (soimport)
addLoader("PEIMPORT", importbyordinal ? "PEIBYORD" : "", kernel32ordinal ? "PEK32ORD" : "",
importbyordinal ? "PEIMORD1" : "", "PEIMPOR2", isdll ? "PEIERDLL" : "PEIEREXE",
"PEIMDONE");
if (sorelocs) {
addLoader(soimport == 0 || soimport + cimports != crelocs ? "PERELOC1" : "PERELOC2",
"PERELOC3", big_relocs ? "REL64BIG" : "", "RELOC64J");
if (0) {
addLoader(big_relocs & 6 ? "PERLOHI0" : "", big_relocs & 4 ? "PERELLO0" : "",
big_relocs & 2 ? "PERELHI0" : "");
}
}
if (use_dep_hack)
addLoader("PEDEPHAK");
// NEW: TLS callback support PART 1, the callback handler installation - Stefan Widmann
if (use_tls_callbacks)
addLoader("PETLSC");
addLoader("PEMAIN20");
if (use_clear_dirty_stack)
addLoader("CLEARSTACK");
addLoader("PEMAIN21");
if (ih.entry && isdll)
addLoader("PEISDLL9");
if (isefi)
addLoader("PEISEFI9");
addLoader(ih.entry || !ilinker ? "PEDOJUMP" : "PERETURN");
// NEW: TLS callback support PART 2, the callback handler - Stefan Widmann
if (use_tls_callbacks)
addLoader("PETLSC2");
addLoader("IDENTSTR,UPX1HEAD");
}
```

它的主要作用是创建和配置在解压缩UPX压缩文件时使用的加载器，`buildLoader`函数接受一个`Filter`类型的指针`ft`作为参数，这是表示在压缩和解压缩过程中使用的过滤器。

`buildLoader`函数内主要使用了两个函数，都在`packer.cpp`中：

`initLoader`：

```c
void Packer::initLoader(const void *pdata, int plen, int small, int pextra) {
owner_delete(linker);
linker = newLinker();
assert(bele == linker->bele);
linker->init(pdata, plen, pextra);
unsigned size;
char const *const ident = getIdentstr(&size, small);
linker->addSection("IDENTSTR", ident, size, 0);
}
```

`addLoader`：

```c
#define C const char *
#define N ACC_STATIC_CAST(void *, nullptr)
void Packer::addLoader(C a) { addLoaderVA(a, N); }
void Packer::addLoader(C a, C b) { addLoaderVA(a, b, N); }
void Packer::addLoader(C a, C b, C c) { addLoaderVA(a, b, c, N); }
void Packer::addLoader(C a, C b, C c, C d) { addLoaderVA(a, b, c, d, N); }
void Packer::addLoader(C a, C b, C c, C d, C e) { addLoaderVA(a, b, c, d, e, N); }
void Packer::addLoader(C a, C b, C c, C d, C e, C f) { addLoaderVA(a, b, c, d, e, f, N); }
void Packer::addLoader(C a, C b, C c, C d, C e, C f, C g) { addLoaderVA(a, b, c, d, e, f, g, N); }
void Packer::addLoader(C a, C b, C c, C d, C e, C f, C g, C h) {
addLoaderVA(a, b, c, d, e, f, g, h, N);
}
void Packer::addLoader(C a, C b, C c, C d, C e, C f, C g, C h, C i) {
addLoaderVA(a, b, c, d, e, f, g, h, i, N);
}
void Packer::addLoader(C a, C b, C c, C d, C e, C f, C g, C h, C i, C j) {
addLoaderVA(a, b, c, d, e, f, g, h, i, j, N);
}
#undef C
#undef N
```

这段代码是对`addLoader`进行重载，`addLoader`为重载的函数，参数个数不同，用于接收不同个数的`const char*`参数。

`#define C` 和 `#define N` 是用定义宏来分别表示 `const char*` 和 `nullptr`，每个`addLoader`内部都调用`addLoaderVA`，将可变参数打包传递给`addLoaderVA`，`addLoaderVA`才是实际的实现函数。

`addLoaderVA`：

```c
void Packer::addLoaderVA(const char *s, ...) {
va_list ap;
va_start(ap, s);
linker->addLoader(s, ap);
va_end(ap);
}
```

`addLoaderVA`中使用了`linker->addLoader(s, ap)`将第一个固定参数 `s` 和可变参数 `ap` 传递给 `linker` 的 `addLoader` 方法，`linker` 是一个 `ElfLinker`类的对象，`addLoader`是`ElfLinker`类中定义的成员函数，`inker->addLoader()` 根据传入的第一个字符串参数和可变参数,将这些字符串添加到加载器中。

总结一下，`buildLoader`函数具体做了以下操作：

1. 初始化加载器：`initLoader(stub_amd64_win64_pe, sizeof(stub_amd64_win64_pe), 2);`用预定义的`stub_amd64_win64_pe`模板初始化加载器。
2. 添加各种代码片段到加载器：`addLoader()`函数用于向加载器添加不同的代码片段。这些代码片段由字符串参数标识，例如 `"START"`、 `"PEISDLL0"`、 `"PEMAIN01"`等，作为各个段的名称。
3. 根据不同的条件添加对应的代码片段：例如，如果文件是DLL或EFI，或者使用了特定的压缩方法（如`LZMA`或`NRV`），那么将添加相应的代码片段。
4. 支持TLS回调：如果启用了TLS回调（通过`use_tls_callbacks`变量控制），则会添加支持TLS回调的代码片段。
5. 最后，添加加载器的结束标记：`"IDENTSTR,UPX1HEAD"`。

标识符在`amd64-win64.pe.S`中：

![图片描述](upload/attach/202308/951654_DNNGJZNDXVEVHKW.png)

### linker.cpp*

根据源码我们可以发现 `linker` 是一个非常关键的类，主要用来构建和管理可执行文件的加载器。它的主要作用有：

1. 管理加载器的各个Section。可以添加、查找Section。
2. 提供加载器生成的框架。处理各个代码段的添加、重定位、链接,优化加载器的生成过程。
3. 实现不同的重定位(Relocation)类型，针对不同架构做重定位处理。
4. 维护符号(Symbol)信息。Symbol与Section相关联。
5. 获取最终连接好的加载器。
6. 封装可执行文件加载器的构建过程，如匹配算法、节对齐等。
7. 链接可执行文件中的数据,使加载器可以访问。
8. 提供便利的接口，如`addLoader`系列函数。
9. 链接外部符号，能够获取导入表等信息。

对应函数进行分类来分析代码可知：

![图片描述](upload/attach/202308/951654_J6GY44MUYKUZYFJ.png)

1. 管理加载器的各个Section：
   - `addSection` - 添加Section
   - `findSection` - 查找Section
   - `getSection/getSectionSize` - 获取Section信息
2. 提供加载器生成的框架：
   - `init` - 初始化加载器`amd64-win64.pe.S`
   - `addLoader` - 添加加载器代码
   - `getLoader` - 获取最终生成的加载器
3. 实现不同的重定位(Relocation)：
   - `relocate1` - 不同架构的重定位实现
   - `relocate` - 进行重定位
4. 维护符号信息：
   - `addSymbol` - 添加符号
   - `defineSymbol` - 定义符号
   - `getSymbolOffset` - 获取符号偏移
5. 获取最终连接好的加载器：
   - `getLoader` - 获取最终的加载器代码
6. 封装可执行文件加载器的构建过程：
   - `alignCode/alignData` - 代码/数据对齐
   - `preprocess*` - 预处理符号表和重定位信息
7. 链接可执行文件中的数据：
   - `defineSymbol` - 链接可执行文件符号
8. 提供便利的接口：
   - `addLoader` - 添加加载器代码段的接口
9. 链接外部符号：
   - `defineSymbol` - 可以链接外部库的符号

我们可以看到 `p_w64pe_amd64.cpp` 没有使用 `getLoader()` 函数，`getLoader()`是通用的加载器模板代码，而`p_w64pe_amd64.cpp` 针对具体的`amd64` PE平台重写了`buildLoader`方法直接生成了`amd64`平台的加载器代码，因此只使用`initLoader`、`addLoader`就可以了。

### amd64-win64.pe.S

在`src/stub/src`目录下可以看到这种名字为架构-系统.类型.S的汇编代码文件, 例如：`amd64-win64.pe.S`，这是针对windows 64位PE程序的汇编代码。

- `amd64`：指示该文件用于 AMD64（也称为 x86-64）架构
- `win64`：指示该文件用于 64 位 Windows 系统
- `pe`：指示该文件用于处理 PE（Portable Executable）
- `.S`：文件的扩展名，表明这是一个汇编语言源代码文件

这个汇编代码也整理的十分整齐，比如把入口点统一放在一起：
