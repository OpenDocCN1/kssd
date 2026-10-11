
# 金山毒霸

> 原文：[`www.pediy.com/kssd/pediy03/forum1144.htm`](https://www.pediy.com/kssd/pediy03/forum1144.htm)

## 金山毒霸试用版完全破解

### 基本信息

*   Henan China Peiyou
*   破解对象: `金山毒霸 2001-06-16 试用版`
*   下载地址: `ftp://202.102.249.209/files/ftproot/3/iDuba_trial616.EXE`

### 所用工具

1.  `w32dasm 8.93` 超级增强版 (感谢中国的 `cracker` 们的修改)
2.  `fi 2.40`
3.  `unaspack211` (其它版本好象不行,这个软件比较牛，`win9x` 界面绝酷解压软件竟然只有 8K)
    *   链接: `http://mud.sz.jsinfo.net/per/aaron/files/unpackers/win/unaspack211c.zip`
4.  `trw2000 1.22`

### 分析过程

用 `exescope` 查看，提示已压缩，用 `Fi` 查看不能识别所加壳的类型。

只好用 `trw2000` 跟踪，加载 `F10` 执行，看到什么了-----`Kav9x!aspack`

不会吧!!!这真是歪打正着，随决定向解压缩，再跟踪。

用 `aspack1.09` 解压，提示未压缩，我 Kao,明明压缩了，装什么蒜!

到网上找了个 `unaspack211` 解压缩成功.

用 `w32dasm` 超级增强版，串式参考（右二)按钮，找到出错信息，按如下修改，成功破解:

*   Referenced by a (U)nconditional or (C)onditional Jump at Address:
    *   `:004C7704(C)`
    *   `|`

    ```text
    :004C7711 E80AF6FFFF              call 004C6D20
    :004C7716 84C0                    test al, al
    :004C7718 0F85B5000000           jne 004C77D3==============================>跳转 I
    ```

    应改 `0F 85 B5 00 00 00` 改为 `E9 B6 00 00 00 90`

    ```text
    :004C771E A134D34C00              mov eax, dword ptr [004CD334]
    :004C7723 8B00                    mov eax, dword ptr [eax]
    :004C7725 8B5024                  mov edx, dword ptr [eax+24]
    ```

*   Possible StringData Ref from Code Obj ->`"序列号不正确，请重新安装金山毒霸！"`
    *   `|`

    ```text
    :004C7728 B8C07E4C00              mov eax, 004C7EC0
    :004C772D E87AA8FBFF              call 00481FAC
    :004C7732 E8DDBFF3FF              call 00403714
    :004C7737 E916060000              jmp 004C7D52
    ```

*   Referenced by a (U)nconditional or (C)onditional Jump at Address:
    *   `:004C76EF(C)`
    *   `|`

    ```text
    :004C773C A184F94C00              mov eax, dword ptr [004CF984]
    ```

*   Possible StringData Ref from Code Obj ->`"SHAREVERSION"`
    *   `|`

    ```text
    :004C7741 BAEC7E4C00              mov edx, 004C7EEC
    :004C7746 E889C8F3FF              call 00403FD4
    :004C774B 7409                    je 004C7756
    :004C774D 833D84F94C0000          cmp dword ptr [004CF984], 00000000
    :004C7754 7527                    jne 004C777D
    ```

*   Referenced by a (U)nconditional or (C)onditional Jump at Address:
    *   `:004C774B(C)`
    *   `|`

    ```text
    :004C7756 E8319DFBFF              call 0048148C
    :004C775B 84C0                    test al, al
    :004C775D 7574                    jne 004C77D3==============================>跳转 II
    :004C775F A134D34C00              mov eax, dword ptr [004CD334]
    :004C7764 8B00                    mov eax, dword ptr [eax]
    :004C7766 8B5024                  mov edx, dword ptr [eax+24]
    ```

*   Possible StringData Ref from Code Obj ->`"试用版已过期！"`
    *   `|`

    ```text
    :004C7769 B8FC7E4C00              mov eax, 004C7EFC
    :004C776E E839A8FBFF              call 00481FAC
    :004C7773 E89CBFF3FF              call 00403714
    :004C7778 E9D5050000              jmp 004C7D52
    ```

*   Referenced by a (U)nconditional or (C)onditional Jump at Address:
    *   `:004C7754(C)`===================================>如果没有这个调用 `4c7795` 处的跳转就不用改了。
    *   `|`

    ```text
    :004C777D A184F94C00              mov eax, dword ptr [004CF984]
    ```

*   Possible StringData Ref from Code Obj ->`"NETEASE"`
    *   `|`

    ```text
    :004C7782 BA147F4C00              mov edx, 004C7F14
    :004C7787 E848C8F3FF              call 00403FD4
    :004C778C 7527                    jne 004C77B5===============>改为 jmp 004C77D3
    :004C778E E8F99CFBFF              call 0048148C
    ```

    应这样改:`75 27 E8 F9 9C FB FF` 改为 `E9 42 00 00 00 90 90`

    为了学会这种改法，我可费了好大劲。

    ```text
    :004C7793 84C0                    test al, al
    :004C7795 753C                    jne 004C77D3==============================>跳转 III
    :004C7797 A134D34C00              mov eax, dword ptr [004CD334]
    :004C779C 8B00                    mov eax, dword ptr [eax]
    :004C779E 8B5024                  mov edx, dword ptr [eax+24]
    ```

*   Possible StringData Ref from Code Obj ->`"网易专用版已过期！"`
    *   `|`

    ```text
    :004C77A1 B81C7F4C00              mov eax, 004C7F1C
    :004C77A6 E801A8FBFF              call 00481FAC
    :004C77AB E864BFF3FF              call 00403714
    :004C77B0 E99D050000              jmp 004C7D52
    ```

*   Referenced by a (U)nconditional or (C)onditional Jump at Address:
    *   `:004C778C(C)`
    *   `|`

    ```text
    :004C77B5 A134D34C00              mov eax, dword ptr [004CD334]
    :004C77BA 8B00                    mov eax, dword ptr [eax]
    :004C77BC 8B5024                  mov edx, dword ptr [eax+24]
    ```

*   Possible StringData Ref from Code Obj ->`"不可识别的版本！"`
    *   `|`

    ```text
    :004C77BF B8307F4C00              mov eax, 004C7F30
    :004C77C4 E8E3A7FBFF              call 00481FAC
    :004C77C9 E846BFF3FF              call 00403714
    :004C77CE E97F050000              jmp 004C7D52
    ```

*   Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
    *   `:004C770B(C), :004C7718(C), :004C775D(C), :004C7795(C)`
    *   `|` 正常执行程序:

    ```text
    :004C77D3 B201                    mov dl, 01
    ```

### 总结

以上三处跳转的 `75` 改为 `EB` 即可，其实只改跳转 2 即可去除过期提示，但我怕在什么条件下又来个 `NAG`，所以一不做二不休，干脆改完全算了，省得日后麻烦。

如果想压缩，可以再用 `aspack` 或 `upx` 等一些优秀压缩软件再压一下。
`4dd466d0e1e0cc123afbe504da2462ca`
