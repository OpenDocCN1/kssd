

# 何判斷被 Asprotect 加殼

> 原文：[`www.pediy.com/kssd/pediy03/forum337.htm`](https://www.pediy.com/kssd/pediy03/forum337.htm)

 如何判斷被 Asprotect 加殼?我們簡單來談談:   
(1)先用 Procdump32 去分析,若 Section Name 除了.data 和.rsrc 外,沒有其他的 Section Name 且有多個.data 那它有八成是 Asprotect 加的殼.   
(2)執行 Imp\_list,看它的 Address.txt,RVA 是否跟軟件的某一 Section 的 VO 一樣,若一樣且 Address.txt 裡的函數還原的很好,那它幾乎可確定是 Asprotect1.0 以上加的殼,且有.idata 段.   
(3)若 RVA 不符合,且 Address.txt 一開始是 GetProcAddress 後面緊接著一大堆空白函數,到最後才出現一系列有系統的函數列,這就是沒有.idata 只有.radta 段.   
(4)若有.idata,你看它的 Address.txt 中的函數,若 Shell32.dll,Comdlg32.dll,Version.dll 它的成員都是亂碼,那它就是 Asprotect1.06 或 1.07   
(你可以參考 Azpr3.21,Aw95prV.1.2,WinNarigatorV.1.91 等)   
以上只是簡單的分類,不代表任何意義,還需要你多去 Trace,和明白 PE 結構,你會有更多的掌握.   4dd466d0e1e0cc123afbe504da2462ca