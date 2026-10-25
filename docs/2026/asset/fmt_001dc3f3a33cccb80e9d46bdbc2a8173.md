
| UTF-16BE | UTF-16 Big Endian (No BOM) | UTF-16 Big Endian (No BOM) | 文件不能带 BOM |
| UTF-16LE-BOM | UTF-16 Little Endian (With BOM) | UTF-16 Little Endian (With BOM) | 需要带 BOM |
| UTF-16BE-BOM | UTF-16 Big Endian (With BOM) | UTF-16 Big Endian (With BOM) | 需要带 BOM |
| UTF-16 | UTF-16 Big Endian (With BOM) | UTF-16 Big(Little) Endian (With BOM) | 需要带 BOM。虽然接受 UTF-16 Little Endian，但是签出时会按 UTF-16 Big Endian处理！会导致编码改变。 |

## 前言

在前面两篇文章 [记一次曲折的多资源文件拆分折腾过程（1）](https://bbs.kanxue.com/elink@8f0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1K9h3q4F1j5$3S2W2L8X3N6F1j5h3&6Q4x3X3g2Y4K9i4c8W2k6g2)9J5k6h3W2G2i4K6u0r3j5i4u0@1K9h3y4D9k6i4y4Q4x3V1k6@1M7X3!0#2j5X3I4W2M7$3S2G2L8%4c8Q4x3X3c8W2M7Y4u0G2M7W2)9J5k6q4u0o6x3e0l9J5x3q4)9J5k6s2g2F1k6i4S2H3k6h3y4@1k6h3c8Q4x3X3c8W2L8X3c8A6k6W2)9J5k6s2g2K6K9h3&6Y4i4K6u0V1L8i4g2D9N6r3W2H3L8r3g2Q4x3X3c8J5j5#2)9J5k6r3W2F1i4K6u0V1L8$3&6W2i4K6u0V1M7s2u0G2K9X3g2U0N6q4)9J5k6r3q4F1k6q4)9J5k6s2y4@1L8%4m8Q4x3X3c8Y4K9i4c8Q4x3X3c8@1M7X3g2S2N6r3W2F1k6#2)9J5k6s2u0U0i4K6u0V1j5i4y4Q4x3X3c8T1K9h3&6S2M7Y4W2Q4x3V1j5`.) 和 [记一次曲折的多资源文件拆分折腾过程（2）](https://bbs.kanxue.com/elink@e12K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1K9h3q4F1j5$3S2W2L8X3N6F1j5h3&6Q4x3X3g2Y4K9i4c8W2k6g2)9J5k6h3W2G2i4K6u0r3j5i4u0@1K9h3y4D9k6i4y4Q4x3V1k6@1M7X3!0#2j5X3I4W2M7$3S2G2L8%4c8Q4x3X3c8W2M7Y4u0G2M7W2)9J5k6q4u0o6x3e0l9J5x3q4)9J5k6s2g2F1k6i4S2H3k6h3y4@1k6h3c8Q4x3X3c8W2L8X3c8A6k6W2)9J5k6s2g2K6K9h3&6Y4i4K6u0V1L8i4g2D9N6r3W2H3L8r3g2Q4x3X3c8J5j5#2)9J5k6r3W2F1i4K6u0V1L8$3&6W2i4K6u0V1M7s2u0G2K9X3g2U0N6q4)9J5k6r3q4F1k6q4)9J5k6s2y4@1L8%4m8Q4x3X3c8Y4K9i4c8Q4x3X3c8@1M7X3g2S2N6r3W2F1k6#2)9J5k6s2u0U0i4K6u0V1j5i4y4Q4x3X3c8T1K9h3&6S2M7Y4W2Q4x3V1j5i4u0@1x3W2)9J5c8R3`.`.) 中，已经把折腾过程中遇到的问题都弄清楚了。因为对这个问题印象太深刻了，而且 `git` 又是开源的，于是特地翻看了 `git` 的源码，并且在 `windows` 上用 `gdb` 简单调试了一波。

> **说明：** 本篇文章适合 `geek` 阅读，全是一些源码查看及编译环境+调试的总结。

<!--more-->

## git 源码下载

在 `github` 上找到 `git for windows` 的仓库，adeK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6Y4K9i4c8Q4x3X3c8X3L8%4u0Q4x3X3c8%4K9h3&6V1L8%4N6K6i4K6u0r3k6$3W2@1i4@1g2r3i4@1u0o6i4K6S2o6i4@1f1@1i4@1u0p5i4@1u0r3i4@1f1%4i4K6V1@1i4@1p5^5i4@1f1#2i4@1p5$3i4K6R3J5i4@1f1@1i4@1t1^5i4K6S2n7i4@1f1#2i4K6V1I4i4@1u0p5i4@1f1@1i4@1u0n7i4@1p5@1i4@1f1#2i4K6R3#2i4K6S2n7i4@1f1&6i4K6W2m8i4K6R3$3i4@1f1@1i4@1t1^5i4K6S2n7i4@1f1$3i4K6W2p5i4@1p5#2i4@1f1#2i4K6S2p5i4@1t1K6i4@1f1#2i4K6S2r3i4@1q4r3i4@1f1K6i4K6R3H3i4K6R3J5

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7 | `# 说明：可能需要科学上网才能正常克隆`    `# 如果只想获得最新的提交可以使用下面的命令`  `git clone` `-``-``depth` `1` `https:``/``/``github.com``/``git``-``for``-``windows``/``git.git`    `# 如果想克隆所有提交集，执行下面的命令`  `git clone https:``/``/``github.com``/``git``-``for``-``windows``/``git.git` |

当然，也可以直接下载压缩包。在 `git` 仓库的主页的右上侧有一个名为 `Code` 的按钮，点击后，会弹出一个悬浮框，里面有几个选项，其中一个是 `Download ZIP` 按钮，点击此按钮即可下载压缩好的代码。

![download-git-zip-from-github](https://bbs.kanxue.com/upload/attach/202111/873494_HWZC43PDVYDJD5D.jpg)

> **说明：**
>
> `git-for-windows/git` `fork` 自 `git/git` ，也可以从 9deK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6Y4K9i4c8Q4x3V1k6Y4K9i4b7`. 下载。
>
> 如果仅仅想查看代码的话，可以通过此方式下载。如果想调试 `git` 的话，不必在这里手动下载，可以通过后面介绍的 `sdk init git` 进行下载。

## 明确目标

主要有两点疑惑需要解答。

1. 为什么在 `.gitattributes` 中把 `.rc` 的 `working-tree-mode` 设置成 `UTF-16LE` 后，在执行 `git add` 的时候会报错，提示设置成 `UTF-16`？

   ![git-add-warning-and-fail](https://bbs.kanxue.com/upload/attach/202111/873494_4H3K9NKWH8W52WW.jpg)

2. 执行 `git checkout .` 的时候，是否会把 `utf8` 编码的内容转换成 `working-tree-encoding` 中指定的编码格式吗？

## 查看代码

本以为很简单就能找到关键代码，但是代码复杂度远超过最初的预想。不过好在整个代码组织结构还是非常清楚规范的 —— 基本上每个子命令都有一个对应的 `.c` 文件，比如 `add` 对应的文件是 `builtin\add.c`。对应的功能入口函数名以 `cmd_` 开头，加上命令名，比如 `add` 命令对应的入口函数是 `cmd_add`。

### 查看 `git add` 对应的代码执行流程，确认在哪里做的检查，并报的警告。

`git add` 命令对应的入口函数是 `builtin/add.c` 中的 `cmd_add`。

刚开始查看的时候，是从 `cmd_add` 看起的，因为这个是 `git add` 对应的入口函数。由于对 `git` 源码不熟悉，花费了很长时间，于是转换思路，从报错信息往回找。搜索提示中的关键字 `contains a byte order mark (BOM).`，很快就搜到了关键函数——`convert.c` 中的 `static int validate_encoding(const char *path, const char *enc, const char *data, size_t len, int die_on_error)`

![validate_encoding](https://bbs.kanxue.com/upload/attach/202111/873494_M23Q23BVX73W9FX.jpg)

从代码上看，是进入到了 `has_prohibited_utf_bom(enc, data, len)` 对应的分支中。按住 `ctrl`，用鼠标点击对应的函数，就跳转到了对应函数的实现。对应的实现在 `utf8.c` 中。摘录如下：

![has_prohibited_utf_bom](https://bbs.kanxue.com/upload/attach/202111/873494_Y5DDGWDFFTZACUE.jpg)

大概意思是，如果指定了 `UTF-16BE` 或者 `UTF-16LE` （另外一个分支是 `UTF-32BE` `UTF-32LE`）并且在文件中检测到了对应的 `BOM` 头，就报错。对应的 `BOM` 头定义如下：

|  |  |

| --- | --- |

| 1  2  3  4 | `static const char utf16_be_bom[]` `=` `{``'\xFE'``,` `'\xFF'``};`  `static const char utf16_le_bom[]` `=` `{``'\xFF'``,` `'\xFE'``};`  `static const char utf32_be_bom[]` `=` `{``'\0'``,` `'\0'``,` `'\xFE'``,` `'\xFF'``};`  `static const char utf32_le_bom[]` `=` `{``'\xFF'``,` `'\xFE'``,` `'\0'``,` `'\0'``};` |

`.rc` 文件中确实有 `utf16_be_bom`。

![show-rc-bom-header](https://bbs.kanxue.com/upload/attach/202111/873494_WS8RUHPMZRE39Z6.jpg)

看来，如果 `working-tree-encoding` 指定为 `UTF-16LE` 或 `UTF-16BE`，那么文件中不能包含 `BOM` 头。也就是说 `UTF-16LE/BE` 对应的文件应该是不包含 `BOM` 头的。

值得注意的是，`validate_encoding()` 函数中还做了另外一个判断，也就是 `is_missing_required_utf_bom()`。如果 `working-tree-encoding` 指定为 `UTF-16`, 那么必须指定 `UTF-16LE` 或者 `UTF-16BE`。

找到了最关键的函数，向上找一找谁调用了这个函数，可以在 `validate_encoding()` 上，右键，选择 `Lookup References...` 就可以查看所有引用此函数的地方了，后面会多次用到这个方法，这里放一张动态图，后面就不赘述了。

![lookup-references](https://bbs.kanxue.com/upload/attach/202111/873494_STAFUAT9MR3FZTG.jpg)

> **说明：**如果已经有了搜索结果，`Source Insight` 会提示**追加**还是**覆盖**。我一般选的是追加。

因为调用链比较长，我挑选了一条可能的调用链，如下：

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10 | `cmd_add()` `/``/` `builtin``/``add.c`  `-``> add_files()` `/``/` `builtin``/``add.c`  `-``> add_file_to_index()` `/``/` `read``-``cache.c`  `-``> add_to_index()` `/``/` `read``-``cache.c`  `-``> index_path()` `/``/` `object``-``file``.c`  `-``> index_fd()` `/``/` `object``-``file``.c`  `-``> index_stream_convert_blob()` `/``/` `object``-``file``.c`  `-``> convert_to_git_filter_fd()` `/``/` `convert.c`  `-``> encode_to_git()` `/``/` `convert.c`  `-``> validate_encoding()` `/``/` `convert.c` |

整个过程是使用 `Lookup References...` 从下向上查找的。

至此，`git add .` 对应的调用栈就整理出来了。接下来，需要确定的是：在执行 `git checkout .` 时，是否会像文档中描述的那样执行编码转换工作？

### 查看 `git checkout .` 的执行流程，确认是否有编码转换相关调用。

与 `add` 一样，`checkout` 命令对应的入口函数也在 `builtin`目录下，对应的文件名是 `checkout.c`，对应的入口函数是 `cmd_checkout`。`git restore` 和 `git switch` 对应的入口函数的实现也在这个文件中，而且都调用了 `checkout_main`。

查找调用函数的过程中，发现在 `entry.c` 中的 `write_entry()` 函数中有如下代码片段

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11  12 | `/``*`  `*` `Convert` `from` `git internal` `format` `to working tree` `format`  `*``/`  `if` `(dco && dco``-``>state !``=` `CE_NO_DELAY) {`  `ret` `=` `async_convert_to_working_tree_ca(ca, ce``-``>name, new_blob, size, &buf, &meta, dco);`  `if` `(ret && string_list_has_string(&dco``-``>paths, ce``-``>name)) {`  `free(new_blob);`  `goto delayed;`  `}`  `}` `else` `{`  `ret` `=` `convert_to_working_tree_ca(ca, ce``-``>name, new_blob, size, &buf, &meta);`  `}` |

根据注释可以猜测，这个是把内部存储格式转换成工作目录所需的格式。

最后一个函数是 `reencode_string_len()`，对应的实现如下：

![reencode_string_len](https://bbs.kanxue.com/upload/attach/202111/873494_5767ZVUTUZNFFWU.jpg)

可以很明显的发现，`reencode_string_len()` 在处理 `out_encoding` 是 `UTF-16` 的时候，直接按大端进行转换。这就是为什么 `.rc` 文件的 `working_tree_encoding` 设置为 `UTF-16` 时，执行 `git checkout .` 后，文件编码变成 `UTF-16BE BOM` 的原因了。

手动整理的调用链如下：

|  |  |

| --- | --- |
