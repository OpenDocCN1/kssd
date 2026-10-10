

# 中国游戏中心大厅程序

> 原文：[`www.pediy.com/kssd/pediy03/forum363.htm`](https://www.pediy.com/kssd/pediy03/forum363.htm)

  
下载：（http://202.96.162.68/MyGames/DownLoad/Setup-Plaza.exe）   

>  Asprtect 已經出到 1.1 了,怎麼還在 1.0 打轉,我只講重建 ImportTable 的部份,跟蹤技巧省略,自己看教程慢慢摸索,才能進步。這個軟件有兩個程式被 Asprotect 加殼,一個是 iGame.exe, 一個是 GameDev.dll,我們以 iGame.exe 為例,dll 的脫殼方法基本一樣,只是要點小技巧, 用 Procdump32 去分析 iGame.exe   
> 你會發現沒有.idata section 只有.rdata section,Asprotect 針對只有.rdata section 會有另外一種編碼方式,我們不需重建 Import Table,只要   
> Modify Image\_Thunk\_Data 的矩陣結構即可。   
>   那要如何 Modify 它呢？當然還是有力的工具 Imp\_list,這次我們用 Imp\_list   
> V1.09,可在老俄那邊下載, 記下.rdata section 的資訊 VO=4C000,Vs=11000   
> \[脫  殼\]:   
>     (1)用 BW2K,得知 OEP=004224D6   
>     (2)執行 Imp\_list,會分別得出,Address.txt,Import0.bin, Import1.   
>         bin, Import2.bin   
>     (3)在無 Soft-ICE 下,按 Ctrl-N 叫出 Trw2K, 下 faults off,再下 CreatefileA do "d \*(esp+4)",執行 iGame.exe,攔住後,按 F12/F10,直到返   
> 回主程式,下 BD \*,G 004224D6,下 Suspend 用 Procdump32(dump(full)),存成   
> iGdump.exe,改 EP=000224D6   
>       (4)用 Hex Workshop 打開 Import1.bin,從偏移 1000 處開始選方塊 728   
>         將其貼至 iGdump.exe 的,rdata section 的 Raw Offset 處, 即   
>         004B600.   
> 這樣即可脫穎成功.   
>       C-Pen  2000\\10\\08   4dd466d0e1e0cc123afbe504da2462ca