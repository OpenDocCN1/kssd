
# PC Guard /Armadillo

> 原文：[`www.pediy.com/kssd/pediy02/forum287.htm`](https://www.pediy.com/kssd/pediy02/forum287.htm)

**\[前言\]**: `PC Guard for Win32` 和 `Armadillo` 都是 Shareware 的保護軟體，它們都防 Soft-ICE，在沒有 `Bww` 之前你可能要花費蠻久的時間才能找到 `OEP`(Thanks D.BOY)。

## (一)PC Guard for Win32 V.3.03D

1. 執行 `frogice`
2. 執行 `bww`,選 `Tarck`
3. 執行 `Pcgwin32.exe`,此時 `bww` 的窗口出現 `00410FF0`(Bingo!!)
4. 用 `Trw2000` 載入 `Pcgwin32.exe` 下 `G 00410ff0` 中斷後直接 `Makepe` 或下 `Suspend` 用 `Procdump`,`dump`(full)均可,記得用 `Procdump` 要修改 `OEP`.

## (二)Armadillo V.1.83Beta-2

1. 執行 `frogice`
2. 按 `Ctrl-D` 叫出 `Soft-ICE`,下 `bpint 3`
3. 執行 `Armadillo.exe`,中斷後下 `e eip 0` 騙過程式,此後再執行 `Armadillo.exe`,它就不會再去攔 `INT 3` 了.會直接執行程式.
4. 執行 `bww`,選 `Tarck`
5. 執行 `Armadillo.exe`,此時 `bww` 的窗口出現 `0040856b` 這是 `Armadillo.exe` 的 `OEP` 這不是我們要的,等到主程式出來後,在 `bww` 的窗口再顯示一個位址 `00413550` 這才是我們要的,這是 `Armadillo.Tmp0` 的 `OEP`,這才是真正的主程式,我們要把它 `Dump` 出來.
6. 用 `Trw2000` 載入 `Armadillo.exe`,下 `G 00413550`,再下 `Makepe` 或用 `Procdump`,`Dump`(full),不過你要選 `Armadillo.tmp0` 而不是 `Armadillo.exe`

再次 Thanks D.BOY `JOHNSON From Taiwan 2000\08\31 4dd466d0e1e0cc123afbe504da2462ca`
