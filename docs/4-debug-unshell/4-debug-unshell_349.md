
# C-pen 谈 Asprotect1.0

> 原文：[`www.pediy.com/kssd/pediy03/forum952.htm`](https://www.pediy.com/kssd/pediy03/forum952.htm)

4) 用 `Hex Workshop` 打开 `Import1.bin`，从偏移 `1000` 处开始选方块 `728`。

将其贴至 `iGdump.exe` 的 `.rdata section` 的 `Raw Offset` 处，即 `004B600`。

能告诉我这些数据的来历吗？谢谢！！

`igame.exe` 的 `.rdata section` 的 `Raw Offset` 值为：`26E00`，那么 `0004B600` 是怎么来的呢？

另外，为何从 `Import1.bin` 的偏移 `1000` 处剪 `728H` 个字节？`728H` 又是怎么来的？

请多指教，谢谢！

`Asprotect1.0` 针对只有 `.rdata section` 的编码方式，是它会放置 `Image_Thunk_Data` 这个矩阵结构在 `.rdata section` 的开头，即 `004C0000` 开始，`Size` 是 `728h`。

而 `Import1.bin` 则是 `Image_Thunk_Data` 所对应的 `API` 位址；我们为什么要将 `API` 的位址粘贴至 `Image_Thunk_Data` 中呢？因为 `Asprotect1.0`（1.07 以上编码方式又不同）会将 `Image_Thunk_Data` 分成两类：

1.  `01xxxxxx` <-- `Asprotect` 的转换 `Code`
2.  `API` 位址

举个例子来说明：例如 `iGame.exe` 它要调用 `RegCloseKey` 这个 `API` 时，它的代码是这样的：

```
Call [004C0000] <-- (A)
```

其中的 `[004C0000]` 在未脱壳时，其值是 `01xxxxxx`，上面的 `Code(A)` 可写成

```
Call 01xxxxxx
```

再经 `Asprotect` 的转换会指向 `RegCloseKey` 的位址即 `BFEA1644`，其时它最终可写成

```
Call BFEA1644
```

但是当我们脱壳后，`01xxxxxx` 的 `Code` 已不复存在，所以我们才用 `BFEA1644` 去取代 `01xxxxxx`，让程式能直接去 `Call` 这个 `API`，基本原理就是这么简单。

最后你问到 `iGame.exe` 的 `Raw Offset` 是 `26E00` 怎么会粘贴至 `4B600` 处，你仔细看文章，我写的是粘贴至 `iGdump.exe`（即我们 dump 下来的程式）它的 `Raw Offset` 是 `4B600`，这是 `Procdump` 分配的，若还有疑问建议你再多读点 `PE` 结构的数据，会比较容易吸收，祝好运！！！

`JOHNSON` 2000\10\09 `4dd466d0e1e0cc123afbe504da2462ca`
