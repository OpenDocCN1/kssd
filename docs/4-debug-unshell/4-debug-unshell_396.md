

# ASProtect V1.2

> 原文：[`www.pediy.com/kssd/pediy03/forum625.htm`](https://www.pediy.com/kssd/pediy03/forum625.htm)

ASProtect V1.2 和 V1.1C 其實大同小異.我們來脫它.以下我只列重點,相信大家均能手動脫殼.   
(1)Import Table:使用最新版的 Imp\_list V.1.13(2001/01/06)來重建,   
                不過你如果選 Auto-detect Import 是無法重建的,估計   
                ASProtect 做了修正去妨它,沒關係它妨新版卻不妨舊版   
                用舊版去重建的目的,主要是去找.idata Section 它得   
                出的 Import Table 並不能用,那是因為 ASProtect V1.2   
                對 Import Table 又做了新的處理,只能用 Imp\_list   
                1.13 來重建,當然你也可以自己去程式中找.idata   
                Section,反正 ASProtect 的 Import Table 編碼處都長   
                的一樣.在 Imp\_list V1.13 中不選 Auto-detect   
                Import,RVA=66000,SIZE=2000,Level=1,如此即能重   
                建,再不然進入程式跟到編碼結束時下 a eip/jmp eip   
                再用 Imp\_list 來重建.   
(2)Hook:脫殼後的程式還是不能用因為 ASProtect 有留下 Hook,如何移除它   
        呢?那你得感謝 ASProtect,因為它完善的 SEH 系統會告訴你,重開   
        始的 SEH MessageBox 中得知它留下的 Hook 位址為 00C1C6D8,用   
        HexWorkshop 打開已脫殼的文件,查找 D8C6C100 你會發現在 RVA   
        64FAC,64FB0 中各有一個,用 W32Dasm 去反組譯,查找 0064FAC,你會   
        發現有如下的程式   
        CALL \[0064FAC\]   
        等價於   
        CALL 00C1C6D8  <-- 在我們已脫殼的文件中是不會有這個位址   
                          的程式碼   
        比較原程式發現原來 00C1C6D8 只是一個 RET,真是無聊!好了!那你   
        應該知道如何移除這個 Hook 吧!我的方法是找個位址放 RET,再將   
        RVA 64FAC,64FB0 指向這個位址,我選在隔壁即 RVA 64FB4 改:   
        RVA 64FAC 00C1C6D8 --> 00464FB4   
        RVA 64FB0 00C1C6D8 --> 00464FB4   
        RVA 64FB4 00 --> C3   
      (註:我標示的是用 RVA 而不是 OffSet,那是因為我是用 ICEdump 去脫   
          故我需令 OffSet=RVA,若你是用 Procdump 去脫.則 OffSet 並不   
          一定等於 RVA 端視你的操作選項而定)   
        
      JOHNSON 2001/02/13 

有關我說的重建 Import Table 的部份,證實第一種是無法重建的,因為 ASProtect 會干擾 Imp\_list 的運作,所以只能用第二種方法,本來想讓大家方便,沒想到變成隨便,Sorry!!   
  事實上我是用第二種方法去重建,第一種是推論,沒想到是錯誤!!故此方法必定可行,若你操作正確的話.   
  ASProtect 的對 Import table 編碼處在:   
:00C1F752 PUSHAD   
:00C1F753 CLD   
:00C1F754 MOV ESI,\[EBP-04\]   
:00C1F757 LODSD   
:00C1F758 OR EAX,EAX   
        .   
        .   
:00C1F79E CALL 00C1F454   
:00C1F7A3 POP EBX   
:00C1F7A4 JMP 00C1F774   
:00C1F7A6 POPAD        <--走到這編碼已完成   
        .   
        .   
:00C1F7C4 JMP 00C1F7E0   
在:00C1F7C4 下 a eip/jmp eip   
按 F5 離開 Soft-ICE 再使用 Imp\_list 即能重建 Import Table(別忘了 RVA=66000,SIZE=2000)   
  至於 hying 所說的還有其他的 Hook,能否具體說明,因我移除這兩個 Hook   
後程式運行的很正常.   
  最後更正上文的兩個錯誤:   
1.查找 0064FAC  -->  00464FAC   
2.CALL \[0064FAC\]  -->  CALL \[00464FAC\]   
真的老了,錯誤百出,還望海涵.   
  
    JOHNSON  2001/02/14  4dd466d0e1e0cc123afbe504da2462ca