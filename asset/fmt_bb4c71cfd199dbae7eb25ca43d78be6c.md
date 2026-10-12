
# Word Password Recovery Master 2.0.0.4 程序分析

> 原文：[`www.pediy.com/kssd/pediy10/50730.html`](https://www.pediy.com/kssd/pediy10/50730.html)

主题：Word Password Recovery Master 2.0.0.4 (在线恢复 word 加密文档) 程序分析
作者：lzype
时间：2007-8-30
下载地址：[http://www.rixler.com/download/wpasrec.zip](http://www.rixler.com/download/wpasrec.zip)

本程序可用于解密 word 97/2000/xp/2003 采用默认加密方式加密的文档(40bit 有效密钥)。

### 脱壳

ASProtect 2.1x SKE -> Alexey Solodovnikov
Version: ASProtect 1.35 build 01.26 Release \[Extract\]

使用 `Aspr2.XX_IATfixer_v2.2s.osc` 脚本脱壳，用 `ImportREC` 进行修复。详细脱壳方法请搜索本论坛或网页。冰系列脱壳参考：[http://www.hack58.com/Soft/html/12/94/2007/200702079696.htm](http://www.hack58.com/Soft/html/12/94/2007/200702079696.htm)

### 本主题相关内容

- [http://bbs1.pediy.com/showthread.php?t=26635&highlight=Word+Password+Recovery+Master](http://bbs1.pediy.com/showthread.php?t=26635&highlight=Word+Password+Recovery+Master)
- [http://bbs1.pediy.com/showthread.php?t=26893&highlight=Word+Password+Recovery+Master](http://bbs1.pediy.com/showthread.php?t=26893&highlight=Word+Password+Recovery+Master)

未注册版本只能恢复前 200 个字符，并在后面添加 “To view full document, please register”。

用抓包工具 `IRIS 4.07.1` 发现数据传输中与文档相关的内容为：
- 发送 `engine_data` 242 个字符 应是 121 字节 16 进制数
- 接收 `key` 20 个字符 应是 10 字节 16 进制数，用于解密

初步观察解密过程有以下 4 步：
1. 从服务器搜索解密密钥
2. 直接复制文档为 `"1 (Demo).doc"` (设原文档为 `1.doc`)
3. 解密 `"1 (Demo).doc"`
4. 若未注册用 word 打开 `"1 (Demo).doc"` 删除位置 200 处以后的内容，然后保存为明文档

### 调试分析

用 `OllyICE` 载入 `WPasRec.exe`，`f9` 运行，选择要解密的文档 `1.doc`。用断点设置工具下断点：`CopyFileA`，然后在 `WPasRec` 中点 `"Remove"`。

程序停在：
```
7C830053    8BFF            mov     edi, edi
```

按 `ctrl+f9` 返回到：
```
0041B03B  |.  6A 00         |push    0                        ; /FailIfExists = FALSE
0041B03D  |.  50            |push    eax                      ; |NewFileName
0041B03E  |.  51            |push    ecx                      ; |ExistingFileName
0041B03F  |.  FF15 70B24500 |call    dword ptr [<&kernel32.CopyFileA>; \CopyFileA
0041B045  |.  85C0          |test    eax, eax                 ;  返回到这里
```

此时文件被复制为 `"1 (Demo).doc"`，与原文档相同。一直按 `f8` 单步运行，并在每次 `call` 后观察 `"1 (Demo).doc"` 的变化。

直到 `0041B143` 处：
```
0041B143  |.  FF10          call    dword ptr [eax]           ;  Word_Pas.0040C4B0
```

文档被修改，可用 word 打开，但密码变了，说明文档被程序用一个已知的密码重新加密。再按 `f8`，直到 `0041B143` 处：
```
0041B26F  |.  68 B8F54600   push    0046F5B8                  ;  ASCII "Z!gjVy>#XelYjT"
```

发现可疑字符，用这 14 个字符居然可以打开 `"1 (Demo).doc"`，内容与原文档一样，惊喜！！！[](img/f43c1e514327696dccbb0da997496b2a.jpg "扮鬼脸")

暂时分析到此。。。

### 小结

1. 在 `0041B143` 处的 `call` 后暂停，可用 `0041B26F` 处的字符 `"Z!gjVy>#XelYjT"` 打开 `"1 (Demo).doc"`，另存为明文档即可，突破 200 字符限制。
2. `0041B143` 处的 `call` 重新加密文档，此过程需要进一步分析。
3. 与服务器传输的数据 `engine_data` 和 `key` 与文档和原密钥之间的关系需要进一步分析。

### 附：Word 加密过程

适用于 97, 2000, XP, 2003

(1) 将最多 15 字节的口令->Unicode->`md5`
(2) 取前 5 字节+16 字节随机数 `a1`=`21` 字节,反复 16 次得 336 字节->`md5`
(3) 取前 5 字节(`s[5]`)+4 字节计数器(初始 0)共 9 字节->`md5` 后作为 `RC4` 的密钥对文档进行加密.每加密 512 字节计数器+1 后重复(3),即第 6 字节顺序变化
(4) 另取随机数 `a2`->`md5` 得 `b`,`a2` 和 `b` 用 `RC4` 的第一次密钥加密后的 32 字节 `AB` 存放文档特定位置:`i=(*((int *)(data+0x240))+0x200);i-=i%512`

从 `i` 处开始每 512 字节找 `01 00 01 00`,其后的 48 字节就是 `a1` 和 `AB`。

说明：
(1) `0x200` 处标志码 `\xEC\xA5\xC1\x00`
(2) `0x20B` 字节低位为 1 表示加密

由此提出破解方法 1：穷尽 5 字节 `s`,(4)中的 `AB` 可用于验证，只要知道这 5 字节即可解密文档。(直接穷尽约需 10 天！[](img/9d43c9aee695f95b024aafd1b69ffa5a.jpg "伤心") )

改进方法 2：建立数据库直接查表。由方法 2 提出 2 个问题：
1. 必须找到一段固定的 5 字节或全 0 处，由这 5 字节得到 `RC4` 的乱数才能查表。
2. 数据库如何建立。

希望有兴趣的共同研究！(QQ:64646157 Email:64646157@qq.com)![](img/c72cb8eba397ac67dda6e1ac58a9b627.jpg "微笑")
