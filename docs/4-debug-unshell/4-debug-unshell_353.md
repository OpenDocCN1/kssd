
# iGdump

> 原文：[`www.pediy.com/kssd/pediy03/forum362.htm`](https://www.pediy.com/kssd/pediy03/forum362.htm)

4) 用 `Hex Workshop` 打開 `Import1.bin`，從偏移 `1000` 處開始選方塊 `728`。

將其貼至 `iGdump.exe` 的 `.rdata` section 的 `Raw Offset` 處，即 `004B600`。

能告訴我這些數據的來歷嗎？謝謝！！

`igame.exe` 的 `.rdata` section 的 `Raw Offset` 值為：`26E00`，那麼 `0004B600` 是怎麼來的呢？

另外，為何從 `Import1.bin` 的偏移 `1000` 處剪 `728H` 個字节？`728H` 又是怎麼來的？

請多指教，謝謝！

`Asprotect1.0` 針對只有 `.rdata` section 的編碼方式，是它會將 `Image_Thunk_Data` 這個矩陣結構放在 `.rdata` section 的開頭，即 `004C0000` 開始，`Size` 是 `728h`，而 `Import1.bin` 則是 `Image_Thunk_Data` 所對應的 API 位址；我們為什麼要將 API 的位址粘貼至 `Image_Thunk_Data` 中呢？因為 `Asprotect1.0`（1.07 以上編碼方式又不同）會將 `Image_Thunk_Data` 分成兩類：

(1) `01xxxxxx` <-- `Asprotect` 的轉換 `Code`

(2) `API` 位址

舉個例子來說明：例如 `iGame.exe` 它要調用 `RegCloseKey` 這個 API 時，它的代碼是這樣的：

```
Call [004C0000] <-- (A)
```

其中的 `[004C0000]` 在未脫殼時，其值是 `01xxxxxx`，上面的 `Code(A)` 可寫成：

```
Call 01xxxxxx
```

再經 `Asprotect` 的轉換會指向 `RegCloseKey` 的位址即 `BFEA1644`，其時它最終可寫成：

```
Call BFEA1644
```

但是當我們脫殼後，`01xxxxxx` 的 `Code` 已不復存在，所以我們才用 `BFEA1644` 去取代 `01xxxxxx`，讓程式能直接去 `Call` 這個 API，基本原理就是那麼簡單。

最後你問到 `iGame.exe` 的 `Raw Offset` 是 `26E00` 怎麼會粘貼至 `4B600` 處，你仔細看文章，我寫的是粘貼至 `iGdump.exe`（即我們 dump 下來的程式）它的 `Raw Offset` 是 `4B600`，這是 `Procdump` 分配的，若還有疑問我建議你再多讀點 `PE` 結構的資料，會比較容易吸收，祝好運！！！

`JOHNSON` 2000\10\09   `4dd466d0e1e0cc123afbe504da2462ca`
