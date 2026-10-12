
### 第 9 关：ZLib 解压

用注册码 + 文件名 + 某字符串的 MD5 值做密钥，用 `AES-256` 算法将 `KeyFile` 解密出来以后，来到这里：

```
004D98B5    8D55 F4        lea edx, [ebp-C]  
004D98B8    8B45 F0        mov eax, [ebp-10]  ; AES-256 解密后字串  
004D98BB    E8 EC91FFFF    call fixed.004D2AAC  ; 用 ZLib 解压  
004D98C0    EB 06          jmp short fixed.004D98C8  
```

`ZLib` 是开源的，而且 `Delphi` 自带了 `ZLib` 库，有兴趣的可以 `F7` 跟进看看算法。

解压后来到这里：

```
004D98FB    8B45 F4        mov eax, [ebp-C]  ; ZLib 解压后字串  
004D98FE    E8 91B6F2FF    call fixed.00404F94  ; System.@LStrLen(String):Integer;  
004D9903    85C0           test eax, eax  
004D9905    74 1F         je short fixed.004D9926  
004D9907    8B45 F4        mov eax, [ebp-C]  ; ZLib 解压后字串  
004D990A    E8 85B6F2FF    call fixed.00404F94  ; System.@LStrLen(String):Integer;  
004D990F    83F8 64        cmp eax, 64  
004D9912    7C 12          jl short fixed.004D9926  ; 长度<100 就 Game Over 啦  
004D9914    EB 06          jmp short fixed.004D991C  
```

呵呵，`ZLib` 解压出来的东东还不能少于 100 字节呢。

过了这里，就过关啦！

可以 `Ctrl+F9` 返回啦！

让我们进入最后一关，挑战最终 Boss！

### 第 10 关：MD5 校检

从这里开始，就是第 10 关啦：

```
004D9954    55             push ebp  
004D9955    8BEC          mov ebp, esp  
004D9957    33C9          xor ecx, ecx  
004D9959    51            push ecx  
004D995A    51            push ecx  
004D995B    51            push ecx  
004D995C    51            push ecx  
004D995D    51            push ecx  
004D995E    53            push ebx  
004D995F    56            push esi  
004D9960    57            push edi  
004D9961    8945 FC       mov [ebp-4], eax  
004D9964    8B45 FC       mov eax, [ebp-4]  
004D9967    E8 18B8F2FF   call fixed.00405184  ; System.@LStrAddRef(void;void):Pointer;  
004D996C    33C0          xor eax, eax  
004D996E    55            push ebp  
004D996F    68 AC9A4D00   push fixed.004D9AAC  
004D9974    64:FF30       push dword ptr fs:[eax]  
004D9977    64:8920       mov fs:[eax], esp  
004D997A    EB 04         jmp short fixed.004D9980  
004D9980    C645 FB 00    mov byte ptr [ebp-5], 0  ; 注册标志位清 0  
004D9984    EB 04         jmp short fixed.004D998A  
004D998A    8B45 FC       mov eax, [ebp-4]  ; KeyFile 解密后的字串  
004D998D    E8 02B6F2FF   call fixed.00404F94  ; System.@LStrLen(String):Integer;  
004D9992    85C0          test eax, eax  
004D9994    75 0D         jnz short fixed.004D99A3  
004D99A3    33C0          xor eax, eax  
004D99A5    55            push ebp  
004D99A6    68 E0994D00   push fixed.004D99E0  
004D99AB    64:FF30       push dword ptr fs:[eax]  
004D99AE    64:8920       mov fs:[eax], esp  
004D99B1    EB 04         jmp short fixed.004D99B7  
```

取字符串的最后 32 字节：

```
004D99B7    8D45 F4       lea eax, [ebp-C]  
004D99BA    50            push eax  
004D99BB    8B45 FC       mov eax, [ebp-4]  ; KeyFile 解密后的字串  
004D99BE    E8 D1B5F2FF   call fixed.00404F94  ; System.@LStrLen(String):Integer;  
004D99C3    8BD0          mov edx, eax  
004D99C5    83EA 20       sub edx, 20  
004D99C8    42            inc edx  
004D99C9    B9 20000000   mov ecx, 20  
004D99CE    8B45 FC       mov eax, [ebp-4]  ; 字符串  
004D99D1    E8 1EB8F2FF   call fixed.004051F4  ; System.@LStrCopy  
004D99D6    33C0          xor eax, eax  
004D99D8    5A            pop edx  
004D99D9    59            pop ecx  
004D99DA    59            pop ecx  
004D99DB    64:8910       mov fs:[eax], edx  
004D99DE    EB 26         jmp short fixed.004D9A06  
004D9A06    33C0          xor eax, eax  
004D9A08    55            push ebp  
004D9A09    68 519A4D00   push fixed.004D9A51  
004D9A0E    64:FF30       push dword ptr fs:[eax]  
004D9A11    64:8920       mov fs:[eax], esp  
004D9A14    EB 06         jmp short fixed.004D9A1C  
```

取字符串的其余字节：

```
004D9A1C    8D45 EC       lea eax, [ebp-14]  
004D9A1F    50            push eax  
004D9A20    8B45 FC       mov eax, [ebp-4]  ; 字符串  
004D9A23    E8 6CB5F2FF   call fixed.00404F94  ; System.@LStrLen(String):Integer;  
004D9A28    8BC8          mov ecx, eax  
004D9A2A    83E9 20       sub ecx, 20  
004D9A2D    BA 01000000   mov edx, 1  
004D9A32    8B45 FC       mov eax, [ebp-4]  ; 字符串  
004D9A35    E8 BAB7F2FF   call fixed.004051F4  ; System.@LStrCopy;  
004D9A3A    8B45 EC       mov eax, [ebp-14]  
004D9A3D    8D4D F0       lea ecx, [ebp-10]  
004D9A40    33D2          xor edx, edx  
004D9A42    E8 A966FFFF   call fixed.004D00F0  ; 加上某字符串，取 MD5，这个 Call 在第 7 关计算 AES-256 密钥时见过  
004D9A47    33C0          xor eax, eax  
004D9A49    5A            pop edx  
004D9A4A    59            pop ecx  
004D9A4B    59            pop ecx  
004D9A4C    64:8910       mov fs:[eax], edx  
004D9A4F    EB 23         jmp short fixed.004D9A74  
004D9A47    33C0          xor eax, eax  
004D9A49    5A            pop edx  
004D9A4A    59            pop ecx  
004D9A4B    59            pop ecx  
004D9A4C    64:8910       mov fs:[eax], edx  
004D9A4F    EB 23         jmp short fixed.004D9A74  
004D9A74    EB 04         jmp short fixed.004D9A7A  
004D9A7A    8B45 F0       mov eax, [ebp-10]  ; 计算出的 MD5 值  
004D9A7D    8B55 F4       mov edx, [ebp-C]  ; 注册码后 32 位  
004D9A80    E8 5BB6F2FF   call fixed.004050E0  ; System.@LStrCmp;  
004D9A85    0F9445 FB     sete [ebp-5]  ;设置注册标志位  
```

原来是这样：`ZLib` 解压出来的不小于 100 字节的东东，后 32 字节作为校检码，如果和前面字节的 `MD5` 相等，就注册成功啦！

到这里，历尽艰难险阻，终于通关啦！！！

### 总结

`KeyFile` 的文件名用了自定义的算法，其余都用了标准算法，尤其是用 `MD5` 值做密钥，用了高强度的 `AES-256` 算法。

写到这里已经很累了，不想做注册机了。

下面举一个例子，作为注册流程和算法总结：

- 用户名：`Ptero`
- 注册码：`COFFB00000-00000000`
- 程序名：`EasyCHM.EXE`
- `KeyFile` 文件名：`chm2259abbf22f.dll`

`KeyFile` 内容：

```
00000000h: 2C 00 00 00 08 CD EA 55 F3 C0 8E 1B 4F AA 26 04  ; ,....完 U 罄?O?.  
00000010h: 72 FC 23 E7 68 6F DD 7D DD DC EF 48 86 4E 70 1E  ; r?鏷 o 輢蒈颒哊 p.  
00000020h: 62 BC 9E 40 C0 1A 4C 5A EE 80 DB A8 F4 61 98 C8  ; b 紴@?LZ 顎郇鬭樔  
00000030h: 65 68 53 B6                                           ; ehS  
```

算法：

1. 用 `MD5` 的方法验证注册码前 2 位：`"QE"`、`"EC"`、`"ET"` 为个人用户，`"CO"`、`"LT"`、`"LD"` 为企业用户
2. 如果是企业用户，则取注册码前 4 位 `"FFB0"` 异或 `4Fh=FFFFh`（`65535`）为许可证数量
3. 用第 6 关的算法算出 `KeyFile` 文件名 `"chm2259abbf22f.dll"`
4. 注册码 + 程序名（大写） + 特定字符串：`"COFFB00000-00000000EASYCHM.EXEbZxvRs2puRGpimSWzVBwwLTExBEHS1CJDJFawAZRwKFs4NCZRoxQX5N1W+RvRs2p5oimqGRL3ApwwA0bg0OZRsgor8nq"`
5. 取 `MD5` 值（小写）的 `Ascii` 码：`"dd069b9bb7ce9f3570ae1a2d40ca38b2"`
6. 用此 `MD5` 的 `Ascii` 码作为 256 位初始密钥，解密 `KeyFile` 第 4 字节以后的内容
7. `KeyFile` 前 4 字节为明文长度（`2Ch`）

解密为：

```
00000000h: 78 9C 33 30 A0 1C 98 A7 A5 18 98 5B 58 9A A4 99  ; x?0?槯?榌 X 殼?  
00000010h: 98 A5 98 9A 98 25 25 1A 24 1B 26 25 9A 58 18 24  ; 槬槡?%.$.&%歑.$  
00000020h: 5B 98 5A 58 58 18 02 00 DF 40 15 30               ; \榋 XX...這.0  
```

8. 再用 `ZLib` 解压，得到：`"000000000000000000000000000000000000000000000000000000000000000000007fd07894f46d546ba0c1ba480c858881"`
9. 此字符串的前 68 字节 + 特定字符串：`"00000000000000000000000000000000000000000000000000000000000000000000bZxvRs2puRGpimSWzVBwwLTExBEHS1CJDJFawAZRwKFs4NCZRoxQX5N1W+RvRs2p5oimqGRL3ApwwA0bg0OZRsgor8nq"`
10. 取 `MD5`：`7fd07894f46d546ba0c1ba480c858881`，再转换成 `Ascii` 码

与字符串的后 32 字节比较，如果相等即注册成功！

### 后记

其实这篇文章还有很多不完善之处，比如 19 楼提到的，只是表面上注册成功了，实际上还是有功能限制的。相关算法我现在正在研究之中，过段时间会再发一篇的。

附：[`aes 库(delphi).rar`]

### 版权声明

本文纯属技术交流，转载请注明作者并保持文章的完整，谢谢

### 破文信息

- **标题**：`EasyCHM 3.50` 注册流程及算法分析(续)
- **作者**：`Ptero`
- **工具**：`FI`, `OllyDbg`, `Dede`, `IDA`, `MD5` 工具
