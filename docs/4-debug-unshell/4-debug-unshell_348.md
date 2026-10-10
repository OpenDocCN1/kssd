

# Asprotect by C-pen

> 原文：[`www.pediy.com/kssd/pediy03/forum335.htm`](https://www.pediy.com/kssd/pediy03/forum335.htm)

 \[前    言\]:有許多朋友問我關於 Asprotect1.07 版的脫殼方法,我想我在這裡簡單寫一下,或許對各位有幫助,或許沒有,無論如何總是一個經驗分享,我使用 Imp\_list 來做為重建 Import Table 的工作,這並非正統,若你的實力夠的話,建議你還是寫個程式來重建.不過 Imp\_list 已經有針對 Asprotect1.07 升級了,只是沒有開放當漏罷了.    
\[目標軟件\]:Advanced Word 95 Password Recovery V1.2   
          URL: http://www.elcomsoft.com/aw95pr.html   
          為什麼選它,嘻!因為它只要修正 3 個 API(Thanks God!)   
\[使用工具\]:1.Soft-ICE 4.01   
          2.Icedump   
          3.Procdump32   
          4.BW2000    
          5.Imp\_list   
\[脫    殼\]:   
1.    先用 Procdump32 和 BW2000 去分析我們要的資料:   
    Size of image:33000   
    Image base:400000   
    原始的程式進入點(OEP):401000   
    .idata VO:8000(執行 Imp\_list 就會知道)   
2.    執行 Imp\_list,會產生 Address.txt,Import1.bin,Import2.bin 沒有最重要的 Import0.bin 沒關係,打開 Address.txt 你會發現 Shell32.dll 和 Comdlg32.dll 它們的函數成員是亂碼,這就是新版的 Asprotect 的保護機制,它先不還原這些 API 等到程式要用時它才會還原 APIname,Call GetProcAddress 去得到函數位址再返回這個位址去執行程式.   
    記下這些指針:Shell32.dllà 006C2024   
                Comdlg32.dllà 006C249C   
                            006C24B8   
3.    叫出 Soft-ICE 下 bpx createfilea do “d \*(esp+4)”,執行 AW95pr.exe 攔住後,按 F5 一次,按 F12 返回,按 F10Trace,此時是在第一段的 CRC  chek   
    按 F10 一直到一連串的 Jmp,Mov,此時就要按 F8 小心 Trace,它 ret 會到 Kernel 的領空,你一直按 F8 會返回 Aw95pr 的領空,若按 F10 程式就會執行,來到 Aw95pr 的領空,再按 F10Trace,如此反覆數次,一直到出現 Asprotect 的編碼程式,此時下 d 408000, 往下找,你會看到編碼後的指針,你會看到 3 類指針,   
    (1)API 位址  (2)Call XXXXXXXX  <--  這兩種是可用 Imp\_list 重建   
    (3)Push XXXXXXXX    
      Call XXXXXXXX      <--這裡就是那些 Shell32.dll,Comdlg32.dll 編碼的地方   
    我們的思路就是知道這些第三類指針代表那些 API 和求得它的位址,再將這些位址代入指針,再使用 Imp\_list 來順利重建.   
    那這些函數如何求?   
    我們以 006c2024 為例,下 d 006c2024 會看到 68 2e 20 6c 00,下 d 006c202e,會看到 65 17 6c 00,再下 d 006c1765,你會看到 Shell32.dll,在它的後面有一組數據 9F 0E C3 2B C1 AB 66 81 14 37 AE 0C 54 50 91 EC   
    DC 20 (9F –重此開始,  0E—表示 API 有 0Eh 個字元, DC—表中斷)   
    其實從 C3~EC 這是未還原的 API Name,它會跟一組變數取 XOR 後還原.   
    再繼續 Trace 下去,一直來到 OEP 即 00401000,下/dump 400000 33000C:\\Awdump.exe   
    ,下 Bpx 006C2024,006C249C,006C24B8   
    按 F5,程式出現,點 Open file,程式中斷在 006C249C,按 F8Trace,你會發現這個 Call 在還原 API Name=GetOpenFileNameA,Address=52086112   
    ,再按 Save 中斷在 006C24B8,API Name=GetSaveFileNameA,Address=52086163,再按 Save 一次,再中斷在 006C24B8,下 a eip/jmp 006C2024(偷吃步,因為只剩下一個 API)得到 API Name=ShellExecuteA,Address=7FD47579   
    再進去 AW95pr,在 00401000(即 OEP 處)改 006C2024-->7FD47579   
                    006C249C-->52086112   
                    006C24B8-->52086163   
    清掉 Hint,f 00408550 l 00408fc0-00408550 00   
    再用 Imp\_list 去重建 Import Table,這次會很成功,且有 Import0.bin   
    最後的工作大家應該都會做,不再贅述.   
很報歉!!寫的很亂,希望對大家有幫助.   
    JOHNSON      2000/09/25   4dd466d0e1e0cc123afbe504da2462ca