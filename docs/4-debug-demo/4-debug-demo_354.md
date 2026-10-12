
# Windows Commander 4.5

> 原文：[`www.pediy.com/kssd/pediy02/forum191.htm`](https://www.pediy.com/kssd/pediy02/forum191.htm)

由於 `JOHNSON` 已被註冊，故以 `C-pen` 代替，感謝看雪、`dr0`、`lijj` 的關心。我們以 `Windows Commander 4.5` 為例，`4.51` 應該差不多吧：

### (1) 脫殼

`Aspack` 壓的，用 `Trw2000` 下 `PNEWSEC` 再下 `MakePE` 即可。

### (2) CRC Check

程式一旦被 `Loader` 或中斷後，即會認為被病毒感染而結束程式，這是防跟蹤的手段。我們可用反向 `Trace` 法即下 `Bpx MessageBoxA` 去攔那個警告訊息，返回後找可跳過此訊息的 `JMP`；我找到：

```
:004DF1D7 SUB EAX,00000536 <--CRC 出錯時 EAX=536
:004DF1DC JZ 004DF4F5 <--跳至 MessgeBoxA
:004DF1E2 JMP 004DF937 &szlig;跳至正常程式
```

如此只要將 `004DF1DC JZ 004DF4F5` 改為 `NOP` 即可，但是如此程式還是會結束，因為還有一處檢測：

```
:004E8171 MOV EAX,[00534048] <--取 CRC flag
:004E8176 SUB EAX,000023B9   <--正確值為 23B9
:004E817B JZ 004E8171        <--若正確即跳躍
:004E817D MOV EAX,00000001   <--否則 EAX 賦值 1
:004E8182 CALL 004044E4      <--CALL 離開程式
:004E8187 CALL 004498E0      <--正常執行程式
```

因此只要再將 `004E817B JZ 004E8187` 改 `jmp` 即可瓦解程式的 `CRC Check`。

### (3) NagBox

未註冊版有令人討厭的 `NagBox`，我們把它拿掉，用 `fimon` 分析，發現它試圖讀 `Wincmd.key` 這個 `Keyfile`（其實有經驗的應該知道這個軟體從舊版開始就是用 `Keyfile` 保護）。故我們假照 `Wincmd.key` 這個檔，這個版的正確字元須為 `1024Byts`。下 `bpx createfilea do "d *(esp+4)"` 和 `bpx Readfile`，程式首先會去讀你的 `Keyfile` 是否為 `80h`。若是它會出現你的 `Keyfile` 是舊版的訊息，其實只要不是 `80h` 你會來到：

```
:004CDF0A LEA EDX,[EBP+FFFFF6D4]
:004CDF10 MOV ECX,00000068
:004CDF15 MOV EAX,[EBP-0118]
:004CDF1B CALL 004C9C0C
```
…………….略…………………….

以上一大串的程式是做 `Keyfile` 取樣和轉換的工作，有恆心和毅力的人可以試看看，我可沒有。按 `F10 Trace` 來到：

```
:004CDFBF CALL 00402A0C  <--Keyfile 比對
:004CDFC4 JZ 004CE078     <--正確時會 JMP,先下 R fl Z 跳過
:004CDFCA XOR EBX,EBX    <--EBX 在此之前為 1,一旦被清掉就是未註冊版
```

繼續 `Trace` 直到：

```
:004CE3DA CALL 00402A0C <--再比對一次
:004CE3DF JZ 004CE3E3   <--等於 0 跳至 Good-Guy
:004CE3E1 XOR EBX,EBX   <--清掉 EBX,Bad-Guy

.
.
:004CE404 MOV [0053E6B8],BL <--[0053E6B8]賦值 1 為已註冊
```

**[後記]**

改法應該很清楚了，只是這樣破了之後在註冊訊息會有一大堆亂碼。那是因為我們沒去分析 `Keyfile`。若你看著礙眼可去分析 `Keyfile` 去顯示你要的註冊訊息，或者去找印出這些字樣的地方，它應該是 `push xxxxxxxx Call xxxxxxxx`。只要讓它傳遞值為 `0` 就不會印出字樣，或是用 `SMC` 讓它顯示你要的字樣。由於我不用這個軟體，所以後面的工作我沒有做，故無法跟你經驗分享，實在抱歉。

希望你會喜歡！若有問題可和我聯絡：My E-mail: `f41400@ms39.hinet.net`

`JOHNSON` from Taiwan `4dd466d0e1e0cc123afbe504da2462ca`
