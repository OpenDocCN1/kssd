
# ReGet pro V.1.7

> 原文：[`www.pediy.com/kssd/pediy03/forum415.htm`](https://www.pediy.com/kssd/pediy03/forum415.htm)

`ReGet pro V.1.7` 我以前脫過，不過沒留筆記，基本上和我以前寫的脫 `iGame` 的方法一樣，不同的是 `iGame` 是 `ASProtect V.1.0` 而 `ReGet` 是 `ASProtect V1.07`，它們都是只有 `.rdata` Section 而沒 `.idata` Section，所以基本上我們只要 Rebuild IAT，程式就能運行，IAT 的 RVA 就是 `.rdata` Section 的 VA。

重建 IAT 的工具當然是用 `Imp_list` 來重建，它產生的 `Import1.bin` 文件就是 IAT，具體可參考我脫 `iGame` 的文章。

如果你成功的 Dump 和 Rebuild IAT 後程式已能運行，若你想要進一步的 Rebuild Import Table，你可以在脫殼後的文件的 `.rdata` Section 後半部（事實上這也是原始的 Import Table 所在）找尋你要的 Import Table 的 RVA，再用 `Soft-ICE` 的 `M addr1 L addr2` 的指令將 IAT Copy 至 `Image_Thunk_Data` 的區域，用 `Imp_list` 來重建 Import Table，將 `Image_Import_Descriptor` 的 `OriginalFirstThunk` 指向 `Image_Thunk_Data`，而 `FirstThunk` 指向 IAT，再將 `Image_Thunk_Data` 覆蓋 IAT，將目錄表的 Import Table 的 RVA 和 SIZE 修正即可重建 Import Table.

`JOHNSON` 2000\11\24   `4dd466d0e1e0cc123afbe504da2462ca`
