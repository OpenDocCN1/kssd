
# PE-Crypt V.1.02

> 原文：[`www.pediy.com/kssd/pediy03/forum367.htm`](https://www.pediy.com/kssd/pediy03/forum367.htm)

\[前 言\]: 寫這篇文章的目的是簡單的介紹 `PE-Crypt` 的保護機制,和 `Imp_list` 這個工具的應用,我想在大家讀過這篇文章後,大概也不會親自去實踐吧!理由是它蠻繁雜的,而且它已經有優秀的脫殼工具,這個工具是 `Bye PE-Crypt V1.02` by `Iczelion` & `Plushmm`, 這兩個軟體均可在 `Program tool` 中下載.

\[保 護 機 制\]:

(1) `Import Table` 重新編碼:此點與 `Asprotect` 相近,但 `Asprotect` 將其分割的更細,讓你更難重建,所以在這一部份,`PE-Crypt` 可算是比卡丘,而 `Asprotect` 則是進化了的雷丘.

(2) 防內部下 `BPX`:你在 `PE-Crypt` 下 `BPX` 是無法中斷的,因為它調用 `VXD call` 進入 `0` 級,將你下的斷點 `Recover`,使你的斷點失效,但它卻沒防 `BPM` (這也需要進入 `0` 級,`Win9X` 下可用 `Cdllgate` 或是 `hook IDT` 上的中斷去改 `drx` 的值)

(3) 可能因為調用 `VXDcall` 的關係,`BW2K` 無法找出它的 `OEP`.

\[使 用 工 具\]:

(1) `Soft-ICE V.4.01`
(2) `Procdump32`
(3) `Imp_list V.1.05`
(4) `Hex Workshop`

\[脫 殼\]:

(1) 先用 `Procdump32` 去分析它:

```
Entry point:29000
Size of image:32000
Image Base:400000
Section .idata
    VS:1000
    VO:15000
    RS:97C
    RO:13E00
```

(2) 用 `Symbole loader` 載入 `PECrypt.exe`,下 `d 415000`,此時`.idata` 還未解壓,下 `BPM 00415000`,按 `F5` 數次,直到出現 `Image_Import_Descriptor` 的結構,此時觀察一下`.idata section` 的結構,你會發現,在 `Image_Thunk_Data` 矩陣中,有些已經被編碼了,而 `Image_Import_By_Name` 的結構還在.

你可以在此時 `dump` 出`.idata section`,然後再去重建, 估計 `Bye PE-Crypt` 就是用這個方法.

我們打算用 `Imp_list` 來重建 `Import Table`,故我們在 `Image_Import_By_Name` 中的最後一個成員,下 `bpm` 斷點.按 `F5` 一直到這個函數也被編碼完成後,你再觀察一次`.idata section` 的結構變成:

```
Image_Import_Descriptor <--已被編碼
Image_Thunk_Data <--已被 API 位址所取代(這也是我們用 Imp_list 的原因)
Image_Import_By_Name <--已被編碼,所有的 API Name 都不見了.
```

(3) 觀察完`.idata section` 後,下 `BD*`,按 `F8 Trace`,你會經過繁複的解壓過程,一直到:

```
:0042E112  JMP EAX  § 此時 EAX=0040644D
```

聰明的你看到位址差那麼多,那你應該知道這就是 `OEP` 了.

此時下 `a eip / jmp eip`,按 `F5` 離開,用 `Procdump, dump(full)`,存檔,改 `EP` 為 `0000644D`,`Import Table RVA=00015000`

(4) 重建 `Import Table`: 按 `Ctrl-D` 叫出 `Soft-ICE`,此時還在 `PE Crypt` 的領空, 我們為了要用 `Imp_list` 來重建 `Import Table`, 故我們必須只保留 `Image_Thunk_Data` 這個矩陣結構,其餘將其清掉:

```
f 415000 l 415068-415000 00
f 415348 l 415cd0-415348 00
```

執行 `Imp_list`,它會產生三個文件,`Address.txt`,`Import1.bin`, `Imprt2.bin`,沒有 `Import0.bin`,那就有可能是因為 `Image_Thunk_Data` 的結構有錯, 以至於 `Imp_list` 無法工作,`OK` 我們打開 `Address.txt` 觀察一下, 你會看到:

```
BFF77796 ->BFF77796[1]=(00000068)KERNEL32.DLL!GetModuleHandleA
```

在下面你又會看到:

```
BFF77796 ->BFF77796[1]=(000001D8)KERNEL32.DLL!GetModuleHandleA
```

奇怪!怎麼會有兩組相同的 `API`,肯定有一組是錯的, 你仔細的觀察, 會發現在`(000001D8)`以前的 `Image_Thunk_Data` 的結構是錯的, 這也是作者設下的障眼法

按 `Ctrl-D` 再叫出 `Soft-ICE`, 下

```
f 415068 l 4151d4-415068 00
```

再執行一次 `Imp_list`, 這次會重建的很好, 有 `Import0.bin`

(5) 最後的工作: 用 `Hex workshop` 打開 `Import0.bin`, 選取適當的方塊黏貼至我們 `dump` 下來的檔案至偏移 `13E00` 處.

`JOHNSON` 2000/10/06 `4dd466d0e1e0cc123afbe504da2462ca`
