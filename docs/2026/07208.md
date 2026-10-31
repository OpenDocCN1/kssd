
# [原创]The XOR Secret in Our Computer System

*Rock*

`rock@phate.tw`

## 摘要

隨著資訊科技的進步，帶動各項產業資訊化、電腦化及數位化，雖然此等便利豐富人類生活，另一方面卻存在著不為人知的隱憂。本文即以一例`XOR` (Exclusive or)，並搭配二補數概念，分別就(1)技術面(2)教科書面(3)法律面及(4)密碼系統面等四個方向來探討。

## 一、前言

以前我們會形容不識字且沒接受教育的人為文盲，那現在我們對那些不會操作電腦也不知道如何開機的人又是怎麼稱呼？十幾年前，知名電影「`The Net`」(網路上身) 演活了駭客在網路普及的環境中所帶來的威脅，這也突顯兩個議題：資訊系統的安全性及人們的警覺性。基於電腦製造的技術突飛猛進，軟體及軔體升級便利也易於更新，且硬體多為積體電路及邏輯元件。因此，對多數人而言，軟體更容易比這些硬體存在漏洞或是缺陷。

本研究擬針對`XOR`(exclusive or, `XOR`)於`x86` 系列`CPU` (Central Processor Unit)組譯過程進行剖析，接著探討二補數數字系統(`Two’s complement number system`)機制隱含之盲點，進而將二者結合後，所產生更大的震撼。第二節為電腦系統現況，說明組合語轉碼的特例，第三節探討二補數系統的理論基礎缺陷，第四節則列舉四個構面所產生的難題，結論於最後著墨。

## 二、電腦系統現況

Intel為個人電腦`CPU`之龍頭，其市佔率在全球有舉足輕重之地位。因此，本文以`x86` 系列之`CPU` 為一開端，並說明自286時代即存在的問題。`CPU` 指令集(`CPU Instruction Set`)為電腦核心的指令集，不論高階語言或是低階語言，不論是編譯器或是組譯器，都要按照`CPU`的`Op Code format`將程式轉換成機器碼(`machine code`)。而 `CPU instruction Set` 本身就存在`CPU`內部，可看成是`CPU`內部的指令，`x86` 系列`CPU`提供簡要的組合語言指令集，使用這些組合語言指令，是不用透過組譯器再行組譯，即直接被`CPU` 所接受。根據現有學術文獻指出，早期`Pentium CPU` 存在除法缺陷[1]，爾後未曾出現學術性文章探討`CPU`缺陷問題。

舉一個`XOR` 指令來說明令人覺得疑惑之處。這個例子是以286的`assembly code` 為主，沒有使用假指令(`pseudo code`)，`CPU` 規格為 Intel Centrino Pentium Mobil processor 1100 MHz。所使用的軟體為 Windows XP 內所提供之 `debug.exe`，檔案大小為20634 bytes。讀者自行實證時，會發現指令碼前端新增一個`0x66`，此為32 bits mode，16 及 32 bit mode 差別只在於暫存器(`register`)的長度不同，並不影響指令基本轉碼的功能及目的。以下給出三個實例，分別說明暫存器之間進行`XOR` 運算所產生的變化，分別指出一個位元、三個位元及五個位元差異。如圖一所示。

.....完整文件置於附件中.....

.....敬請不吝指教.....謝謝.....

![](https://bbs.kanxue.com/upload/attach/200905/244702_vadlpdqyp1qzwaj.jpg)

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [The XOR secret in Our Computer System.rar](javascript:void(0);)

  （141.38kb，297次下载）

- [20090505055955203.jpg](javascript:void(0);)

  （189.09kb，716次下载）
