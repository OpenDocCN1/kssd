
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22 | `ar` `-``x lib_ameba.a`  `ar` `-``x lib_codec.a`  `ar` `-``x lib_hs_uart_redirect.a`  `ar` `-``x lib_mdns.a`  `ar` `-``x lib_p2p.a`  `ar` `-``x lib_rtlstd.a`  `ar` `-``x lib_sdcard.a`  `ar` `-``x lib_usbh.a`  `ar` `-``x lib_wlan.a`  `ar` `-``x lib_xmodem.a`  `ar` `-``x lib_arduino_alexa.a`  `ar` `-``x lib_google_cloud_iot.a`  `ar` `-``x lib_i2c_redirect.a`  `ar` `-``x lib_mmf.a`  `ar` `-``x lib_platform.a`  `ar` `-``x lib_rtsp.a`  `ar` `-``x lib_usbd.a`  `ar` `-``x lib_websocket.a`  `ar` `-``x lib_wps.a`  `rm console_i2c.o`  `rm alexa_mem.o`  `arm``-``none``-``eabi``-``gcc` `-``w` `-``shared` `*``.o` `-``o  liball.so` |

然后就可以用IDA加载so进行分析了

# 漏洞分析

本节基于最新的SDK和[ameba-2.0.4-build20180817](https://bbs.kanxue.com/elink@f0eK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6S2L8h3u0A6L8%4c8Q4x3V1k6S2L8h3t1I4i4K6g2X3j5i4u0V1N6h3W2F1L8#2)9J5c8X3u0D9L8$3u0Q4x3V1k6E0j5i4y4@1k6i4u0Q4x3V1k6m8M7X3c8#2K9h3&6G2i4K6g2X3M7r3q4U0K9$3q4Y4k6g2)9J5c8Y4u0W2L8r3g2S2M7$3g2Q4x3V1k6S2L8h3g2T1j5g2)9J5k6o6u0Q4x3X3f1H3i4K6u0W2y4q4)9J5k6r3u0#2K9h3I4V1x3U0l9I4z5o6l9^5x3e0N6Q4x3X3g2@1j5i4u0Q4x3X3g2Y4P5R3`.`.)有漏洞版本SDK进行分析，同时对补丁和漏洞进行分析

### VD-1406 (CVE-2020-9395) – Stack-based buffer overflow vulnerability

`ClientEAPOLKeyRecvd`函数会调用`CheckMIC`来处理数据

|  |  |

| --- | --- |

| 1 | `CheckMIC(v5``-``>EAPOLMsgRecvd, v5``-``>PTK, &v5``-``>EAPOLMsgRecvd)` |

`EAPOLMsgRecvd`为外部数据

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7 | `bool` `__fastcall CheckMIC(OCTET_STRING EAPOLMsgRecvd, unsigned __int8` `*``key,` `int` `keylen)`  `{`    `v3` `=` `EAPOLMsgRecvd.Octet;`  `v4` `=` `EAPOLMsgRecvd.Octet[``20``];`  `v6` `=` `&tmpbuf[``95``];`  `rtl_memcpy_0(tmpbuf, EAPOLMsgRecvd.Octet, EAPOLMsgRecvd.Length);` |

`Octet`为数据地址，`Length`为数据长度，这里没有检查`Length`，从而导致栈溢出。

最新版本的`CheckMIC`函数如下

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10 | `bool` `__fastcall CheckMIC_constprop_14(``int` `data, unsigned` `int` `length,` `int` `a3)`  `{`      `v6` `=` `*``(data` `+` `20``);`  `if` `( length >` `0x200` `)` `/``/` `新增length的检查`  `return` `0``;`  `v8` `=` `&stack[``95``];`  `freertos_memcpy_0(stack, data, length);` `/``/`  `freertos_memset_0(&stack[``95``],` `0``,` `16``);` |

在最新版的SDK里面会检查`length`不能超过`0x200`

调用点

|  |  |

| --- | --- |

| 1 | `CheckMIC_constprop_14(sta``-``>EAPOLMsgRecvd.Octet, sta``-``>EAPOLMsgRecvd.Length, sta``-``>PTK) )` |

`EAPOLMsgRecvd`为外部数据，`Octet`为数据地址，`Length`为数据长度

### VD-1407 – Read out of bounds vulnerability

问题还是位于`CheckMIC`函数

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11  12  13 | `bool` `__fastcall CheckMIC(OCTET_STRING EAPOLMsgRecvd, unsigned __int8` `*``key,` `int` `keylen)`  `{`      `v3` `=` `EAPOLMsgRecvd.Octet;`  `v4` `=` `EAPOLMsgRecvd.Octet[``20``];`  `v6` `=` `&tmpbuf[``95``];`  `rtl_memcpy_0(tmpbuf, EAPOLMsgRecvd.Octet, EAPOLMsgRecvd.Length);`  `flag` `=` `v4 &` `7``;`  `v8` `=` `(ntohs_0(``*``&tmpbuf[``16``])` `+` `4``);` `/``/` `从外部数据取出长度 v8`  `if` `( flag` `=``=` `1` `)`  `{`  `rt_md5_hmac_0(&tmpbuf[``14``], v8, key,` `16``, &tmpbuf[``95``]);` `/``/` `作为 rt_md5_hmac参数` |

问题逻辑是，首先从数据包里面取出了2个字节作为长度存放到`v8`，然后没有检查`v8`是否会大于`Length`，就传给`rt_md5_hmac`去算一个hash，在`rt_md5_hmac`里面会越界读。

补丁版本

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11 | `v9` `=` `(ntohs_0(``*``&stack[``16``])` `+` `4``);`  `if` `( (length` `-` `13``) <``=` `v9 )` `/``/` `检查长度`  `return` `0``;`  `if` `( (v6 &` `7``) !``=` `1` `)`  `{`  `if` `( (v6 &` `7``)` `=``=` `2` `)`  `{`  `v8` `=` `digest;`  `rt_hmac_sha1_0(&stack[``14``], v9, a3,` `16``, digest);`  `return` `rtl_memcmp_0(v8, data` `+` `95``,` `16``)` `=``=` `0``;`  `}` |

增加了长度检查

### VD-1408 – Stack-based buffer overflow vulnerability

漏洞应该是位于`ClientEAPOLKeyRecvd`，函数会调用`DecWPA2KeyData`

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7 | `if` `( !DecWPA2KeyData_0(`  `v5,`  `EapolKeyMsg` `+` `95``,`  `(EapolKeyMsg[``94``]` `+` `(EapolKeyMsg[``93``] <<` `8``)),`  `&v5``-``>PTK[``16``],`  `16``,`  `decrypted_data) )` |

`EapolKeyMsg`为外部数据，这里3个参数表示长度，直接从外部数据中取出，然后在`DecWPA2KeyData`里面会解密数据

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11  12 | `int` `__fastcall DecWPA2KeyData(WPA_STA_INFO` `*``pStaInfo, unsigned __int8` `*``key,` `int` `keylen, unsigned __int8` `*``kek,` `int` `keklen, unsigned __int8` `*``kout)`  `{`    `*``default_key_iv` `=` `0xA6A6A6A6``;`  `*``&default_key_iv[``4``]` `=` `0xA6A6A6A6``;`  `if` `( (pStaInfo``-``>EapolKeyMsgRecvd.Octet[``2``] &` `7``)` `=``=` `1` `)`  `{`  `rtl_memcpy_0(tmp1, pStaInfo``-``>EAPOLMsgRecvd.Octet` `+` `63``,` `16``);`  `rtl_memcpy_0(&tmp1[``16``], kek, keklen);`  `sub_14D5C(&rc4_ctx, tmp1, keklen` `+` `16``);`  `rt_arc4_crypt_0(&rc4_ctx, tmp2, tmp1,` `256``);`  `rt_arc4_crypt_0(&rc4_ctx, tmp2, pStaInfo``-``>EapolKeyMsgRecvd.Octet` `+` `95``, keylen);` |

解密结果会存放在`tmp2`，`tmp2`是栈数组，大小为`257`，如果`keylen`比较大就会导致栈溢出。

补丁，在调用`ClientEAPOLKeyRecvd`时会检查取出的`keylen`.

|  |  |

| --- | --- |

| 1  2  3 | `v13` `=` `(key_msg[``94``]` `+` `(key_msg[``93``] <<` `8``));`  `if` `( sta``-``>EAPOLMsgRecvd.Length` `-` `0x70` `<``=` `v13`  `|| !DecWPA2KeyData_0(sta, key_msg` `+` `95``, v13, &sta``-``>PTK[``16``],`  `16``, &stack,` `255``) )` |

### VD-1409 – Stack-based buffer overflow vulnerability

这个漏洞位于 `AES_UnWRAP` 函数，由于函数的二进制代码位于rom，这里直接分析原文提供的代码

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24  25  26  27  28  29  30 | `void AES_UnWRAP(unsigned char` `*` `cipher,` `int` `cipher_len,`  `unsigned char` `*` `kek,` `int` `kek_len,`  `unsigned char` `*` `plain)`  `{`    `v5__cipher_len` `=` `(cipher_len` `+` `7``) & (cipher_len >>` `32``);`  `if` `( cipher_len >``=` `0` `)`  `v5__cipher_len` `=` `cipher_len;`  `v6__cipher` `=` `cipher;`  `v7__aligned_cipher_len` `=` `v5__cipher_len >>` `3``;`  `nblock` `=` `v7__aligned_cipher_len` `-` `1``;`  `aes_set_key(&ctx, kek,` `128``);`  `memcpy(A, v6__cipher,` `8u``);`  `if` `( v7__aligned_cipher_len` `-` `1` `>` `0` `)`  `{`  `v8__R` `=` `R;`  `v9__cipher_ptr` `=` `(``int``)(v6__cipher` `+` `8``);`  `v10__block_counter` `=` `0``;`  `do`  `{`  `v11__R` `=` `v8__R;`  `v12__cipher_ptr` `=` `(const void` `*``)v9__cipher_ptr;`  `+``+``v10__block_counter;`  `+``+``v8__R;`  `v9__cipher_ptr` `+``=` `8``;`  `/``/` `STACK OVERFLOW` `-``-``>`  `memcpy(v11__R, v12__cipher_ptr,` `8u``);`  `}`  `while` `( v10__block_counter !``=` `nblock );`  `}` |

主要问题是`AES_UnWRAP`循环解密时会不断把结果拷贝到栈数组`R`中，如果`cipher_len`过大，就会溢出数组`R`。

`R`的定义

|  |  |

| --- | --- |

| 1 | `unsigned __int8 R[``32``][``8``];` |

### VD-1410 – Stack-based buffer overflow vulnerability

漏洞位于`ClientEAPOLKeyRecvd`，函数会调用`DecWPA2KeyData`

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7 | `if` `( !DecWPA2KeyData_0(`  `v5,`  `EapolKeyMsg` `+` `95``,`  `(EapolKeyMsg[``94``]` `+` `(EapolKeyMsg[``93``] <<` `8``)),`  `&v5``-``>PTK[``16``],`  `16``,`  `decrypted_data) )` |

`EapolKeyMsg`为外部数据，这里3个参数表示长度，直接从外部数据中取出，然后在`DecWPA2KeyData`里面会解密数据

|  |  |

| --- | --- |

| 1  2  3  4  5 | `int` `__fastcall DecWPA2KeyData(WPA_STA_INFO` `*``pStaInfo, unsigned __int8` `*``key,` `int` `keylen, unsigned __int8` `*``kek,` `int` `keklen, unsigned __int8` `*``kout)`  `{`    `v9` `=` `kout;`  `rtl_memcpy_0(v9, v10, keylen);` |

`DecWPA2KeyData`解密完数据后会把`keylen`长度的数据拷贝到 `kout`，即`decrypted_data` 其大小为 `128`字节，所以可能导致栈溢出。

修复方案

|  |  |

| --- | --- |

| 1 | `!DecWPA2KeyData_0(sta, key_msg` `+` `95``, v13, &sta``-``>PTK[``16``],` `16``, &stack,` `0xFF``)` |

额外增加了一个参数，表示`Kout`的大小，然后在`DecWPA2KeyData`里面会去校验

|  |  |

| --- | --- |

| 1  2 | `if` `( key_len > kout_length || key_len >``=` `0x102` `)`  `return` `0``;` |

### VD-1411 – Stack-based buffer overflow vulnerability（未修复）

漏洞位于`ClientEAPOLKeyRecvd`

|  |  |

| --- | --- |

