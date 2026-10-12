
# Macro MagicV4.1p

> 原文：[`www.pediy.com/kssd/pediy01/forum005.htm`](https://www.pediy.com/kssd/pediy01/forum005.htm)

用`W32DASM`破解`Macro MagicV4.1p`时间限制（转载希望保持完整）

作者：丁丁虾 又名：`DDXia`

文件大小：`866KB`

软件授权：共享软件

使用平台：`Win95/98/NT`

发布公司：`Home Page`

## 软件简介

用`Macro Magic`指定宏指令热键. 让你快速打开应用程序. 传送邮件. 输入密码等例行工作。

`http://www.newhua.com.cn/down/MMagic.exe`

## 破解过程

5 分钟搞定它的时间限制，在`W32DASM`中可以看到"Trial Expired"的字样，然后往上走

```
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004A0E6D(C)
|
:004A0E98 E82FF8FFFF                 call 004A06CC
:004A0E9D 85C0                       test eax, eax
:004A0E9F 7F75                       jg 004A0F16
```

修改`jg 004A0F16`为`jmp 004A0F16`

```
:004A0E9F EB75                       jmp 004A0F16
```

或者在原有指令后填充`nop`指令以保持长度一致（原跳转指令长度为2字节，跳转偏移量计算需调整，这里根据原文语境简化为修改跳转逻辑）：

```
\* Possible StringData Ref from Code Obj ->"It has been 30 days or more since "
                                       ->"you first installed Macro Magic. "
                                       ->" We sincerely hope that you have "
                                       ->"found it to be a worthwhile addition "
                                       ->"to your software library."
```

```
:004A0EA1 B8B4104A00                 mov eax, 004A10B4
:004A0EA6 E8D9DCF9FF                 call 0043EB84
:004A0EAB 8B0D38434C00               mov ecx, dword ptr [004C4338]
:004A0EB1 A178444C00                 mov eax, dword ptr [004C4478]
:004A0EB6 8B00                       mov eax, dword ptr [eax]
:004A0EB8 8B151C214A00               mov edx, dword ptr [004A211C]
:004A0EBE E84D11F9FF                 call 00432010
:004A0EC3 A138434C00                 mov eax, dword ptr [004C4338]
:004A0EC8 8B00                       mov eax, dword ptr [eax]
:004A0ECA 8B803C020000               mov eax, dword ptr [eax+0000023C]
:004A0ED0 33D2                       xor edx, edx
:004A0ED2 E8350FF8FF                 call 00421E0C
:004A0ED7 A138434C00                 mov eax, dword ptr [004C4338]
:004A0EDC 8B00                       mov eax, dword ptr [eax]
:004A0EDE 8B803C020000               mov eax, dword ptr [eax+0000023C]
```

```
\* Possible StringData Ref from Code Obj ->"Trial Expired"
```

```
:004A0EE4 BA5C124A00                 mov edx, 004A125C
:004A0EE9 E8B60FF8FF                 call 00421EA4
:004A0EEE A138434C00                 mov eax, dword ptr [004C4338]
:004A0EF3 8B00                       mov eax, dword ptr [eax]
:004A0EF5 8B8000020000               mov eax, dword ptr [eax+00000200]
:004A0EFB BA74124A00                 mov edx, 004A1274
:004A0F00 E89F0FF8FF                 call 00421EA4
:004A0F05 A138434C00                 mov eax, dword ptr [004C4338]
:004A0F0A 8B00                       mov eax, dword ptr [eax]
:004A0F0C E89BF2F8FF                 call 004301AC
:004A0F11 E95F010000                 jmp 004A1075
```

完成时间：`2000.2.18.8:10`

历时 `5` 分钟又 `14` 秒。哈哈
