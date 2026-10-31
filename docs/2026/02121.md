
# anti-homeMade(1): 文件斧头 fileaxe v1.00 (1千字)

## 文件斧头 fileaxe.exe v1.00

`crack by 6767`

**工具：** 只用 `WDASM` 就能搞定

**说明：** 文件切割器，是否好用不详，刚下载的

**地址：** `ftp://211.100.8.153/download/99/10261_axefb.zip`

比较恶，未注册版本更改浏览器默认页。

先脱壳，反编译，找出错信息：

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

```
|:00466AD3(C), :00466AE4(C)
```

\* Possible StringData Ref from Code Obj ->`"错误的序列号！"`

```
:00466D48 B8E86D4600              mov eax, 00466DE8
:00466D4D E80255FEFF              call 0044C254
```

向上找出错的跳转：

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

```
|:00466A8F(C)
```

```
:00466ACF 837DF800                cmp dword ptr [ebp-08], 00000000    ＜－这里判断注册码是否为空
:00466AD3 0F846F020000            je 00466D48
:00466AD9 8B45F8                  mov eax, dword ptr [ebp-08]
:00466ADC 8B55F0                  mov edx, dword ptr [ebp-10]        ＜－正确注册码的位置
:00466ADF E8A0D2F9FF              call 00403D84
:00466AE4 0F855E020000            jne 00466D48
:00466AEA 8D9504FEFFFF            lea edx, dword ptr [ebp+FFFFFE04]
:00466AF0 33C0                    xor eax, eax
:00466AF2 E8CDBDF9FF              call 004028C4
:00466AF7 8B8504FEFFFF            mov eax, dword ptr [ebp+FFFFFE04]
:00466AFD 8D55EC                  lea edx, dword ptr [ebp-14]
:00466B00 E8AB16FAFF              call 004081B0
:00466B05 8D8500FEFFFF            lea eax, dword ptr [ebp+FFFFFE00]
```

\* Possible StringData Ref from Code Obj ->`"fileaxe.ini"`

```
:00466B0B B9C46D4600              mov ecx, 00466DC4
```

你的注册码可以用 `SI` 跟出来，下网后再跟吧。
