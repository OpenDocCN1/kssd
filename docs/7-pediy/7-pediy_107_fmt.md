
# Awave Studio v7.0

> 原文：`www.pediy.com/kssd/pediy03/forum436.htm` ([链接](https://www.pediy.com/kssd/pediy03/forum436.htm))

## 一. 前言

用 SMC 去破解 ASProtect 的保護實在是一種痛苦的事，不過既然有人有興趣，那我們就來談談吧！在底下的文章，我將假設讀者已對 ASProtect 的保護有初步的了解，也跟蹤過它，對 SMC 的技巧有概念，最後先申明一點，因為 ASProtect 的多層次壓縮，所以你要用 SMC 的話，一定會很繁雜的，希望你有耐心看完並實作，也希望本文能對你有所幫助。

## 二. 使用工具

1. Soft-ICE V.4.01
2. FrogICE
3. 16 進位編輯器(建議使用 Hex workshop 和 Hiew(方便除錯))

## 三. 本文

### 1. ASProtect 的保護機制

#### (1) 動態位址

我們要用 SMC 首先要克服的就是 ASProtect 的 Relocation，它每次運行的位址都不一樣，而我們的 Code 是不會變動的，故你必須先了解 ASProtect 是如何 Relocation 的，答案是 ASProtect 調用 `GetTickCount` 函數，拿其值去當 `IMAGE_RELOCATION` 結構中的 `RelocCount` 成員，讀者有興趣可以去查閱 `IMAGE_RELOCATION` 結構的相關資訊及編程，在此不加詳述。其程式碼如下:

```
EAX=GetTickCount

25FFFF0100    AND EAX,0001FFFF
EB04          JMP   0051722F
```

我們改為:

```
B811110100    MOV EAX,00011111
EB04          JMP   0051722F
```

如此 `RelocCount` 就會是一個常數，用 Hex workshop 打開 `Awave.exe` 尋找

`25FFFF0100EB04` <-- 將是唯一

改 `B811110100EB04`

如此位址就會固定，不再是變動的了。

#### (2) 多層次壓縮

*   **A..** ASProtect 的程式碼 <-- 我們稱為第一層，這是可以用 16 進位編輯器直接修改的地方
*   **B.** 解壓部份代碼 <-- 第二層
*   **C.** 解壓部份代碼 <-- 第三層
*   **D.** 利用第三層代碼解壓第四層代碼 <-- 第四層亦就是 ASProtect，Anti-Debugger，CRC Check，Time Limit，NagScreen 等….的地方
*   **E.** 利用第四層代碼解壓主程式代碼，Import Table 加密等 <-- 第五層

如此層層相扣，不要說它在 Import Table 作手腳，如此多層次的壓縮，你要用內存補丁(指在有 Soft-ICE 的環境下) 幾乎很難工作。

#### (3) 首先我們要對付的是 ASProtect 的保護

這個軟體有 3 種，Anti-Debugger，BoundsCheck，CRC Check

**[Anti-Debugger]**

```
:005D6184      CALL  005D60D4  <-- \\.\SIWDEBUG
:005D6189      TEST  AL,AL
:005D618B(753A) JNZ   005D61C7  <-- NOP
:005D618D      CALL  KERNEL32!GetLastError
:005D6192      CMP   EAX,02
:005D6195(7530) JNZ   005D81C7  <-- NOP
:005D6197      MOV   EAX,[005DC878]
:005D619C      CALL  005D60D4  <-- \\.\SICE
:005D61A1      TEST  AL,AL
:005D61A3(7522) JNZ   005D61C7  <-- NOP
:005D61A5      CALL  KERNEL32!GetLastError
:005D61AA      CMP   EAX,02
:005D61AD(7518) JNZ   005D61C7  <-- NOP
:005D61AF      MOV   EAX,[005DC874]
:005D61B4      CALL  005D60D4  <-- \\.\NTICE
:005D61B9      TEST  AL,AL
:005D61BB(750A) JNZ   005D61C7  <-- NOP
:005D61BD      CALL  KERNEL32!GetLastError
:005D61C2      CMP   EAX,02
:005D61C5(7413) JZ    005D61DA  <-- JMP
```

**[BoundsCheck]**

```
:005D6234      MOV   EBP,4243484B
:005D6239      JMP   005D623D
:005D623D      MOV   AX,0004
:005D6241      JMP   005D6244
:005D6244      INT   3
```

我們只要改 `EBP` 的值，就能 ByPass BoundsCheck

改: `005D6235` `4B` --> `12`

**[CRC Check]**

```
:005D633E    MOV   EAX,[EBP-14]
:005D6341    CMP   EAX,[EBP-10]
:005D6344    JZ    005D6389
```

改: `005D6344` `7443` --> `EB43`

以上就是 ASProtect 保護的代碼，若你要一個一個的改，可能要改很多 Bytes，既然它最終是要跳至 `005D6389` 我們何不來個乾坤大挪移讓它一次跳的夠

改: `005D618B` `E9F9010000` JMP `005D6389`

**[改主程式的地方]**

找接近 OEP 的地方來 Modify 主程式(此時主程式的代碼已被解壓)我找到這個:

```
:005D607D 8B45F8 MOV EAX,[EBP-08]
:005D6080 EB02   JMP 005D6084
```

剛好 5 個 BYTES 正好符合我們做遠程跳躍

#### (4) SMC 開始

終於到了最重要亦是最痛苦的時刻了，希望大家能堅持下去，首先我們還須做個工作，就是找個地方(要夠大)來放我們的 Code,我們找 ASProtect 主程式的 Section

```
Section     VS      VA      RS      RO
----------------------------------------------------
Y38d5x    29000   157000   28800     5FE00
```

我找到 OffSet : `60B00`

載入時位址: `00557D00`

這地方有大量已初值化的 `00 00` 供我們使用，Thanks ASProtect!

**(A) 利用第四層 Modify 主程式**

主程式的破解改法，我們採用 Superboss 的改法，維持文章的一貫性

改: `004A486D` `40` --> `33`

還記得先前我們找到的修改主程式的地方吧!

```
:005D607D 8B45F8 MOV EAX,[EBP-08]
:005D6080 EB02   JMP 005D6084
```

改: `005D607D` `E97E1CF8FF` jmp `00557D00` <-- 跳至我們的家

按 F10 來到 `00557D00`，下 `a eip`

```
:00557D00 8B45F8   MOV EAX,[EBP-08]  <-- 還原代碼
:00557D03 C6056D484A0033 MOV Byte Ptr [004A486D],33
:00557D0A E975E30700 JMP 005D6084     <-- 跳至原代碼
```

要到這裏你須先 Bypass ASProtect 的保護才行，測試無誤後，記得抄下機器碼用 16 進位編輯器去改

**(B) 利用第三層 Modify 第四層**

我們用 Symbole Loader 來載入 `Awave.exe`，別忘了裝載 ICEdump 或改 Section 的 Charactics, 否則無法中斷的，還記得先前所說的 Anti-Debugger 嗎? 我們要 Modify 它，位址是 `005D618B` 下 `bpm 005D618B`，按 F5 一次，中斷在:

```
:005E555B F3A5   REPZ MOVSD
:005E555D 8BC8   MOV ECX,EAX
:005E555F 83E103 AND ECX,03
:005E5562 F3A4   REPZ MOVSB
:005E5564 5E     POP ESI
:005E5565 8B8539294400 MOV EAX,[EBP+00442939]
:005E556B 6800800000  PUSH 00008000
:005E5570 6A00         PUSH 00
:005E5572 50           PUSH EAX
```

走過 `005E5562`，你會發現第四層的代碼已被解壓

改: `005E556B` `6800800000` PUSH `00008000`

為: `005E556B` `E99F27F7FF` jmp `00557D0F`

```
:00557D0F 6800800000 push 00008000
:00557D14 C7058B615D00E9F90100 mov dword ptr [005D618B],0001F9E9
:00557D1E C6058F615D0000 mov byte ptr [005D618F],00
:00557D25 C7057D605D00E97E1CF8 mov dword ptr [005D607D],F81C7EE9
:00557D2F C60581605D00FF mov byte ptr [005D6081],FF
:00557D36 E935D80800 jmp 005E5570
```

恭喜! 又完成了一層，若你尚不了解，建議你畫個流程圖，仔細思考它們的邏輯關係那會幫你了解，OK 讓我們繼續吧

**(C) 利用第二層 Modify 第三層**

目標: 使 `005E556B` `6800800000` push `00008000`

變成: `005E556B` `E99F27F7FF` jmp `00557D0F`

`bpm 005E556B`，按 F5 二次，中斷在:

```
:005E527F F3A4   REPZ MOVSB
:005E5281 8B8539294400 MOV EAX,[EBP+00442939]
:005E5287 6800800000 PUSH 00008000   <-- 我們要 Modify 的地方
:005E528C 6A00    PUSH 00
```

原本應該是要 JMP 至 `00557D3B`，不過我想留些 Buffer 以防萬一，所以我們 JMP 至 `00557D6C`

改: `005E5287` `6800800000` PUSH `00008000`

為: `005E5287` `E9E02AF7FF` JMP `00557D6C`

```
:00557D6C 6800800000 push 00008000
:00557D71 C7056B555E00E99F27F7 mov dword ptr [005E556B],F7279FE9
:00557D7B C6056F555E00FF mov byte ptr [005E556F],FF
:00557D82 E905D50800 jmp 005E528C
```

**(D) 利用第一層 Modify 第二層**

終於到最後了，不知你有什麼感覺，是不是覺得 ASProtect 很機車呢?

目標: 使 `005E5287` `6800800000` push `00008000`

變成: `005E5287` `E9E02AF7FF` jmp `00557D6C`

`bpm 005E5287` 按 F5 一次，中斷在:

```
:00557698 F3A5  REPZ MOVSD
:0055769A 83C328 ADD EBX,28
:0055769D EBDA  JMP 00557679
```

走過 `00557698` 第二層的代碼就會被解壓，但是這一段代碼會走很多次，沒關係我們有 EDI 的值當我們的指標，此時 EDI=`005E5B94`

改: `0055769A` `83C328` ADD `EBX,28`
: `0055769D` `EBDA` JMP `00557679`

為: `0055769A` `E9E8060000` JMP `00557D87` 這可以直接用 Hex workshop 來改

```
:00557D87 83C328 add ebx,28
:00557D8A 81FF945B5E00 cmp edi,005E5B94
:00557D90 0F85E3F8FFFF jnz 00557679
:00557D96 C70587525E00E9E02AF7 mov dword ptr [005E5287],F72AE0E9
:00557DA0 C6058B525E00FF mov byte ptr [005E528B],FF
:00557DA7 E9CDF8FFFF jmp 00557679
```

## 四. 后記

很久沒有寫文章了,寫的很雜請原諒,明年可能會到上海發展不曉得你們歡迎否,噫!台灣真的越來越難混了.

慾海浮沈名利爭
電光石火步此生
紅塵情事揮不盡
觀世不笑是痴人

JOHNSON from Taiwan 2000\12\8 `4dd466d0e1e0cc123afbe504da2462ca`
