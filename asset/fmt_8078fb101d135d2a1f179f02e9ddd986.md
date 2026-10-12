
# Whereisit3.0

> 原文：[`www.pediy.com/kssd/pediy03/forum351.htm`](https://www.pediy.com/kssd/pediy03/forum351.htm)

沒錯，這是 `Asprotect1.0` 加的殼，無法產生 `Import0.bin`，沒關係！我們打開 `Address.txt` 看看，你會發現如下的東東（注意！你的位址可能不一樣，以偏移為主）：

`0087B4A4->BFF76E28[0]=(00000224)KERNEL32.DLL!GetProcAddress`

`(00000224)`是偏移量，故 `ImageBase(400000)+.idata(VO)(2BF000)+偏移(00000224)=006BF224`，這個位址存放 `0087B4A4` 的值，你把它改成 `BFF76E28`，就是把它改成 `GetProcAddress` 這個 API 的位址（你的 API 位址也可能因 `Windows` 的版本不同而不同，以你的機器為主）

即將 `0087B4A4 -> BFF76E28`

如此 `Imp_list` 就會工作，替你 dump 出 `Import0.bin`，另在我這裡還遇到點小瑕疵，

在我的 `Shell.dll` 中它的最後 API 成員是 `SHGetDesktopFolder`，它的指針在 `002C242E`，在這個指針後面不乾淨，把它清為 `0` 後，程式就能工作，你不一定會遇到。

不過我還是說說比較好。

最後 `Asprotect` 從 `1.0` 版以後有多處的 `CheckSum`，你光跳過一個是沒用的，你必需找到它們一一將它們補上，否則它就會給你來個"程式執行無效，即將關閉."

不過這個軟件不需去改 `Section`，你怎麼會遇到，真奇怪!!!

JOHNSON 2000/09/30

什麼是 `Bounds check` 我就不說了，網上很多教程，你也可以讀 `frogsice` 的 `Readme`，那裡有很多 `Anti-Debugger` 的 Code，那是 Cracker 必備的基本知識，我發覺很多人都會用 `frogsice` 但卻沒有幾個人會去讀 `Readme`，真是奇怪！

你要在 `Soft-ICE` 攔它時，你需先裝載 `SoftICE Backdoor Keeper`，再重新啟動，令其生效，執行 `frogICE`，按 `Ctrl-D` 教出 `Soft-ICE`，下 `bpint 3`，再執行 `Whereisit`，你就會中斷在 `int 3`，`Bypass` 的方法很多，我習慣下 `e eip 0`，再下 `bpm (Import Table section)` 來到它處理 Import table 的地方；若沒有 `Soft-ICE` 只有 `Trw2k`，只要下 `faults off` 就可以 `Bypass`，你再試試！若有疑問再說吧！

最後你不介易我給你一個習題做做吧！`BLind Read&Write` 有用到 `Int 68` 去偵測 `Soft-ICE`，你仔細的讀 `frogsICE` 的 `Readme`，再去解決它！不難的，自己試試看，會有收穫的，祝你好運!!!

JOHNSON 2000/10/02 `4dd466d0e1e0cc123afbe504da2462ca`
