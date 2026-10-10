

# VB power wrap V1.7

> 原文：[`www.pediy.com/kssd/pediy03/forum322.htm`](https://www.pediy.com/kssd/pediy03/forum322.htm)

 噫!我以前貼過 Armadillo 的脫殼方法,怎麼不能舉一反三.   
\[目標軟體\]:VB power wrap V1.7   
\[保護機制\]:Armadillo 保護,防 Soft-ICE,時間限制.   
\[使用工具\]:1.Trw2000   
          2.Bw2000   
          3.Procdump32   
          4.Regmon   
\[手動脫殼\]:1.第一次執行被 Armadillo 保護的軟體,Armadillo 會在 C:\\WINDOWS\\TEMP\\的目錄下解壓一個 Armxxxx.tmp 的文件,這是保護系統的 dll,程式用它 Anti-Debugger,設置時間標誌等等,並將所取得的環境資訊寫入註冊表中,以供程式比對運用,這個 Key 的路徑在 HKLM\\Software\\The Sillicon Realms Toolworks\\Armadillo,故若你的軟件試用期已過,你便需要把這個 Key 刪除,重新獲得試用。   
2.若你有安裝 Soft-ICE,須先把它移除,以利作業,首先執行 BW2000,按 track,再執行 vbrap.exe,首先在 BW2000 中顯示 004085BB,等到 NagBox 出現,按 OK 後會再出現一個位址 0048535 C 這是 VBRAP.tmp0 的 OEP 這才是我們要的。   
3.用 Trw2000 載入 vbrap.exe,下 faults off,再下 G 0048535C 等到 NagBox 出現按 OK 後程式會中斷在 0048535C,再下 makepe(不是很穩定)或下 suspend 用 Procdump32,選 VBRAP.tmp0,dump(full),改 OEP 等於 0008535C 即可脫殼.   
          C-pen  2000\\9\\17   4dd466d0e1e0cc123afbe504da2462ca