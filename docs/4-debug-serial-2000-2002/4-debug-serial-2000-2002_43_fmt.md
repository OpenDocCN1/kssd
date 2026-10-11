
# `dll show4.4`

> 原文：[`www.pediy.com/kssd/pediy01/forum034.htm`](https://www.pediy.com/kssd/pediy01/forum034.htm)

注册 `dll show4.4`

近日装了一个 `dll show4.4`，它是有 30 天使用限制。

运行后选 “Register”，填入：

- `User　Name`：你的姓名
- `Orginazation`：随便
- `Registration`：乱填

运行 `TRW2000 1.06`

`BPX HMEMCPY`

`G`

回到 `dll show`，按下 OK 按扭，`TRW` 跳出。

下指令：

`BD`

按 `F12` 11 下，改按 `F10`，走到：

`CS:40D5EB  cmp eax, 0D5FCE3C`

`CS:40D5F0  je  40D5FE` <-- **

此处改 `je` 为 `jne`

清除断点 `bc`

`g`

注册完毕。

注：从新运行 `dll show`，选 “Register”，你会看到真正的注册码。
