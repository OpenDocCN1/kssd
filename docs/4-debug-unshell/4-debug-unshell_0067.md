
# ReloX 重定位表修復 (圖文)

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-654.htm`](https://www.pediy.com/kssd/pediy08/pediy8-654.htm)

需準備的工具：`Dll LoadEx`、`LordPE`、`ReloX`

如 `load.dll` 為加密文件，區段如下

![](img/46d8383c51fda39979212e64d43aa8be.jpg)

將 `load.dll` 脫殼 `Dump` 到另一處命名為 `unpack.dll` (可隨意命名)

然後將未脫殼的 `load.dll` 複製一份，如圖

![](img/1c9aa9848dfa17d1a0945f5abb9713df.jpg)

用 `Dll LoadEx` 載入未加殼的文件(這裡指 `load.dll` 與 `copy-load.dll`)

![](img/f6222d62b1091f0c929c7529649545b6.jpg)

不要關閉 `Dll LoadEx`

然後開啟 `LordPE`，在上面的窗口點選 `Dll LoadEx` 的圖示，在看到下面窗口看剛剛載入的兩個 `load.dll` 與 `copy-load.dll`

![](img/b8a20cf74ef77715703a04f89a28b4ca.jpg)

![](img/6de4134d173671aed8740e337298cd85.jpg)

這時候看到的 `load.dll` 的 `ImageBase` 為 `10000000`

`copy-load.dll` 的 `ImageBase` 為 `01C70000`

然後在 `LordPE` 上面將 `load.dll` 選擇 `dump full` 為 `01.dll`，將 `copy-load.dll` `dump` 為 `02.dll`

![](img/8dddbdd8a0b174235c381e90cbda2ace.jpg)

![](img/12e84e8d9a70dde79bbf9e98bdf6c148.jpg)

不要關閉 `LordPE`

接著開啟 `ReloX` 將剛剛 `Dump` 的 `01.dll` 與 `02.dll` 分別載入，再右邊的框框填入剛剛在 `LordPE` 上看到的 `ImageBase`

![](img/fc8be739031d9e59f0108c97aa378e04.jpg)

然後按下 `Selsct sections` 選擇要重定位的區段，這裡選第一個區段 (後有說明-->註)

![](img/ba316e495fbcac4f28fe9cc69179cef1.jpg)

接著按下 `Compare` 比較，窗口會彈出重定位表，最後按下 `Fix PE Module` 來修復已脫殼 `Dump` 後的文件 `unpack.dll`，會出現一個新的 `_unpack.dll`，這個就是已修復重定位的文件

![](img/3e84d61214eac2f59221d784dd2aa06e.jpg)

用 `LordPE` 打開來看已修復的文件，重定位表的區段為 `.reloc`

`ReloX` 修復完成後 `Dll LoadEx`、`LordPE` 才可關閉

註:

在脫殼的過程 `Dump` 下來已經知道，第一個區段裡面為 `.text`、`.rdata`、`.data` 的合併區段，修復重定位主要就選擇這 3 個區段。

許多加殼的文件會將這三個區段合併為一個區段。

所以在 `Dump` 後要完美修復必須找出這三個區段手工修復，其實沒修復為一個合併區段的状态也無妨。

`4dd466d0e1e0cc123afbe504da2462ca`
