
# ASProtect 2.1x SKE + Delphi + Stolen Code

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-769.htm`](https://www.pediy.com/kssd/pediy07/pediy7-769.htm)

--------------------------------------------------------------------------------

【破解作者】 SYSCOM

【使用工具】 `Ollydbg1.10`, `LordPE`, `ImportREC 1.62`, `PEid 0.94`

【破解平台】 `Windows XP SP1`

【软件名称】 `MultiTranse 3.5.8`

【下载地址】 `Http://www.tialsoft.com/multitranse/`

【软件简介】 `Online Translation Software`

【加壳方式】 `ASProtect SKE 2.X`

【破解声明】 我是一只小菜鸟，偶得一点心得，愿与大家分享：）

--------------------------------------------------------------------------------

(0) 用 `PEid 0.94` 偵測為 `ASProtect 2.1x SKE -> Alexey Solodovnikov`

(1) 用 `ollydbg` 載入 `MultiTranse.exe`, 按 `ALT+O` 設置異常 `Memory Access Violation`。

按 `F9`, 會發生異常中斷在位址 `00BDB530`

```
00BDB530-- MOV DWORD PTR DS:[EAX],12C5EC7F
```

(2) 按 `CTRL+B` 搜尋 `83,C4,2C,5D,5F,5E,5B,C3`, 在 `00BDA8C5`, 設硬件斷點

```
00BDA8C5--E8 7ACEFFFF CALL 00BD7744 <= 設硬件斷點
00BDA8CA--83C4 2C ADD ESP,2C
00BDA8CD--5D POP EBP
00BDA8CE--5F POP EDI
00BDA8CF--5E POP ESI
00BDA8D0--5B POP EBX
00BDA8D1--C3 RETN
```

(3) 按 `CTRL+F2`, 重新載入, 按 `ALT+O` 忽略異常 `Memory Access Violation`。

按 `F9`, 會發生硬件中斷在位址 `00BDA8C5` 按 `F7 STEP` 走到 `00BD77BE`

```
00BD77BE--RETN <==== 返回到 00EA02CD => OEP 開始
00BD77BF--JMP SHORT 00BD77C2
00EA02CD--PUSH EBP <== OEP Start
00EA02CE--JMP 00EA0C22
00EA02D3--MOV DWORD PTR DS:[54B718],EAX
00EA02D8--ADD EAX,EBX
```

(4) 按 `CTRL+B` 搜尋 `89,45,F0,B8,00,07,00,00` 修改 `00BE7097` ==> `JMP 0054B060`

```
00BE7092--33C0 XOR EAX,EAX
00BE7094--8945 F0 MOV DWORD PTR SS:[EBP-10],EAX
00BE7097--B8 00070000 MOV EAX,700 修改 ==>JMP 0054B060(獲得 API 返回 Patch)
```

(5) 貼上 Patch Code, 在 `0054B000` 空白區, `CTRL+ Gray*` 將 `EIP=> 0054B000`

準備取出正確 `WIN32 API`

(6) 第一次, 設 `ESI=1`, `F2->` 中斷設在 `0054B05E`, 按 `F9`,

使 `5C0000` 區塊獲得 `IAT` (亂序)

(6.1) 在 `5C0000` 區塊中, 會發現一個, 指向 `00BEXXXX`, 殼的 API,

那個就是隱藏的 API => `GetProcAddress`, 修正它

(7) `CTRL+ Gray*` 將 `EIP=>0054B0D0`, `F2->` 中斷設在 `0054B0F7`,

按 `F9`, 在 `5C0000` 區塊獲得 `IAT(排序)`

(8) `COPY->IAT` `5C0000` 區塊, 到文字檔 `IAT.TXT`, 利用編輯軟體, 例如 `WinHex`, `UEdit32`

將 `IAT` 中, 各 `DLL` 中間, 插入 `4 BYTES 00 00 00 00` 區隔, 完成後, 再貼回 `005C0000`

(9) 第二次, 設 `ESI=2`, `F2->` 中斷設在 `0054B05E`, 按 `F9`, 使 `CODE` 重新定位 `IAT ADDRESS`

(10) Patch 完成後, `Full_Dump`, 然後執行 `ImportREC` 令 `RVA=1C0000`, `SIZE= 868`

-> `Fix Dump` 完成 `90%` 進度

修復 CODE AND 重建 IAT - Patch Code

\========================================================================

```
0054B000--BE 01000000 MOV ESI,1 ;ESI=1 取出 API,ESI=2 重定位 API

0054B005--B8 00005C00 MOV EAX,MultiTra.005C0000 ;IAT 開始位址
0054B00A--A3 F4AF5400 MOV DWORD PTR DS:[54AFF4],EAX

0054B00F--BA 00104000 MOV EDX,MultiTra ;CODE 開始位址

0054B014--803A E8 CMP BYTE PTR DS:[EDX],0E8 ;是否 CALL ??
0054B017--75 19 JNZ SHORT MultiTra.0054B032
0054B019--8B5A 01 MOV EBX,DWORD PTR DS:[EDX+1]
0054B01C--03DA ADD EBX,EDX
0054B01E--83C3 05 ADD EBX,5 ;計算正確的 CALL ADDR
0054B021--81FB 0000EA00 CMP EBX,0EA0000 ;是否 ASPR 解碼位址??
0054B027--75 2C JNZ SHORT MultiTra.0054B055
0054B029--60 PUSHAD ;保存寄存器
0054B02A--8925 F0AF5400 MOV DWORD PTR DS:[54AFF0],ESP ;保存 ESP 值
0054B030--FFE2 JMP EDX ;進入 ASPR 解碼

0054B032--66:813A FF25 CMP WORD PTR DS:[EDX],25FF ;是否 JMP [XX]?
0054B037--75 1C JNZ SHORT MultiTra.0054B055
0054B039--8B5A 02 MOV EBX,DWORD PTR DS:[EDX+2]
0054B03C--81FB 08D25400 CMP EBX,MultiTra.0054D208 ;是否在原來 IAT 位址筥圍
0054B042--72 11 JB SHORT MultiTra.0054B055 ;小於 54D208?
0054B044--81FB 08E25400 CMP EBX,MultiTra.0054E208
0054B04A--77 09 JA SHORT MultiTra.0054B055 ;大於 54E208?
0054B04C--8B1B MOV EBX,DWORD PTR DS:[EBX]
0054B04E--83FE 01 CMP ESI,1 ;ESI=1??
0054B051--74 26 JE SHORT MultiTra.0054B079
0054B053--EB 4B JMP SHORT MultiTra.0054B0A0
0054B055--42 INC EDX ;計數器 +1
0054B056--81FA 00505400 CMP EDX,MultiTra.00545000
0054B05C--72 B6 JB SHORT MultiTra.0054B014 ;CODE 段結束?
0054B05E--EB FE JMP SHORT MultiTra.0054B05E ;Patch 結束 中斷點
0054B060--90 NOP ;<=取得 API 返回點
0054B061--8BDA MOV EBX,EDX
0054B063--891D F8AF5400 MOV DWORD PTR DS:[54AFF8],EBX ;保存 API
0054B069--8B25 F0AF5400 MOV ESP,DWORD PTR DS:[54AFF0] ;取回 ESP
0054B06F--61 POPAD ;還原寄存器
0054B070--8B1D F8AF5400 MOV EBX,DWORD PTR DS:[54AFF8] ;取出 API
0054B076--EB D6 JMP SHORT MultiTra.0054B04E
0054B078--90 NOP
0054B079--8B0D F4AF5400 MOV ECX,DWORD PTR DS:[54AFF4]
0054B07F--B8 00005C00 MOV EAX,MultiTra.005C0000 ;IAT 開始位址
0054B084--3918 CMP DWORD PTR DS:[EAX],EBX
0054B086--74 CD JE SHORT MultiTra.0054B055 ;是否重複??
0054B088--05 04000000 ADD EAX,4
0054B08D--3BC1 CMP EAX,ECX
0054B08F--72 F3 JB SHORT MultiTra.0054B084
0054B091--8919 MOV DWORD PTR DS:[ECX],EBX ;保存 API
0054B093--8305 F4AF5400 04ADD DWORD PTR DS:[54AFF4],4 ;指到下個 API ADDR
0054B09A--EB B9 JMP SHORT MultiTra.0054B055
0054B09C--90 NOP
0054B09D--90 NOP
0054B09E--90 NOP
0054B09F--90 NOP
0054B0A0--B8 00005C00 MOV EAX,MultiTra.005C0000 ;IAT 開始位址
0054B0A5--3918 CMP DWORD PTR DS:[EAX],EBX
0054B0A7--74 0E JE SHORT MultiTra.0054B0B7 ;是否找到??
0054B0A9--05 04000000 ADD EAX,4
0054B0AE--3D 6C085C00 CMP EAX,MultiTra.005C086C ;IAT 結束位址?
0054B0B3--72 F0 JB SHORT MultiTra.0054B0A5
0054B0B5--EB FE JMP SHORT MultiTra.0054B0B5
0054B0B7--66:C702 FF25 MOV WORD PTR DS:[EDX],25FF ;修正為 JMP
0054B0BC--2D 00005C00 SUB EAX,MultiTra.005C0000
0054B0C1--05 00105C00 ADD EAX,5C1000 ;修正 OFFSET
0054B0C6--8942 02 MOV DWORD PTR DS:[EDX+2],EAX ;重定位 ADDR
0054B0C9--EB 8A JMP SHORT MultiTra.0054B055
0054B0CB--90 NOP
0054B0CC--90 NOP
0054B0CD--90 NOP
0054B0CE--90 NOP
0054B0CF--90 NOP ;排序 API
0054B0D0--B9 2C085C00 MOV ECX,MultiTra.005C082C ;IAT 結束位址
0054B0D5--BE 00005C00 MOV ESI,MultiTra.005C0000 ;IAT 開始位址
0054B0DA--8BFE MOV EDI,ESI
0054B0DC--8B06 MOV EAX,DWORD PTR DS:[ESI]
0054B0DE--8B1F MOV EBX,DWORD PTR DS:[EDI]
0054B0E0--3BC3 CMP EAX,EBX
0054B0E2--76 05 JBE SHORT MultiTra.0054B0E9
0054B0E4--93 XCHG EAX,EBX ;如果大於則互換
0054B0E5--8906 MOV DWORD PTR DS:[ESI],EAX
0054B0E7--891F MOV DWORD PTR DS:[EDI],EBX
0054B0E9--83C7 04 ADD EDI,4
0054B0EC--3BF9 CMP EDI,ECX
0054B0EE--76 EC JBE SHORT MultiTra.0054B0DC ;LOOP X1
0054B0F0--83C6 04 ADD ESI,4
0054B0F3--3BF1 CMP ESI,ECX
0054B0F5--76 E3 JBE SHORT MultiTra.0054B0DA ;LOOP X2
0054B0F7--EB FE JMP SHORT MultiTra.0054B0F7 ;排序結束
```

\========================================================================

(11) 執行 `LordPE`, 選定 `MultiTranse.exe` 按右 key Mouse,

選 `dump region` 取出 `Stolen Code` 區段, 忽略無用模擬 `API VM`,

取出必須的 `M1-M3 Stolen Code VM` 區段, 及 `OEP` 區段

```
VM Address          Size
===========================
00BC0000            33000    <-ASProtect 解密 CODE 區段
00C40000             4000    <-ASProtect 資料 DATA 區段
```



```
00EA0000            3000     <-OEP  Stolen Code  

00F00000            1000     <-M1   Stolen Code 

00F10000            1000     <-M1   Stolen Code 

00F20000            1000     <-M2   Stolen Code 

00F30000            1000     <-M2   Stolen Code 

00F40000            1000     <-M2   Stolen Code   

00F50000            1000     <-M3   Stolen Code 

00F60000            1000     <-M3   Stolen Code    
```

===========================

全部抓出來後,再用`LordPE`,附加在`DUMP`檔尾,並修正`PE Header`

(12) 修正 4 個,預先解碼的 `API` (注意:不是每個 `ASProtect`,都會預先解碼)

如果沒遇到,可以跳過這修正..

```
00401384--JMP  DWORD PTR DS:[kernel32.GetModuleFileNameA]  

00406F50--JMP  DWORD PTR DS:[ADVAPI32.RegCloseKey]  

00406F58--JMP  DWORD PTR DS:[ADVAPI32.RegCreateKeyExA]  

00406F98--JMP  DWORD PTR DS:[ADVAPI32.RegQueryValueExA]  
```

(13) 修正 `Route Check`

修正前....

```
00BE8542--MOV  EAX,DWORD PTR DS:[EAX+34]    

00BE8545--CALL EAX                                          

00BE8547--SUB  DWORD PTR SS:[EBP+C],EAX
```

修正後

```
00BE8542--NOP  

00BE8543--NOP  

00BE8544--MOV EAX,DWORD PTR SS:[ESP+58]  

00BE8548--SUB EAX,5  
```

(14) 開始 RUN ,跨平台 TEST ~OK~!!!!!

PS.(1)只要是 `ASProtect 2.1x SKE` + `Delphi` (有/無 `Stolen Code` 都可)

都可適用,此 `PATCH` ....(改一下位址,即可套用)![](img/a7e67d4f8f4afe4c3cd0617906cb3d9d.jpg)

如果是 `VC ++` 程序,請找我以前發的一篇文章,套用 `PATCH`,即可

(2)那些分離的 `Stolen Code`,基本上你時間多,都可以,還原回去的

相關的文章,可以參考 `loveboom` 兄,所寫的   `ASProtect` 系列   

新手脫殼,如有錯誤,還望高手不吝指正....![](img/d8960f6c7afa20ba0ca27920887f4540.jpg) 

`4dd466d0e1e0cc123afbe504da2462ca`
