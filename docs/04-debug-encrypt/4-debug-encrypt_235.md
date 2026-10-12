
# 流星网络电视 2.15

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-362.htm`](https://www.pediy.com/kssd/pediy08/pediy8-362.htm)

等我搞定了这个程序后，搜索了下论坛，看到已经有战友在去年我刚入道的时候已经发过，感觉非常郁闷！看了看文章，还是不一样，想了想还是发了吧。

这是一个失败的加密算法，虽然搞了非常强悍的算法，`n` 次 `DES`＋`n` 次 `MD5`，但是加密注册码与加密用户名＋机器码的形式一样，而且 `key` 也一样，根本就不用去算 `DES`＆`MD5`，虽然不是明码比较，但在某个地方直接下断就能得到注册码。

`ASPack` `2.12` -> `Alexey Solodovnikov`

`Borland Delphi` `6.0` - `7.0`

机器码：`9030D2D9`

用户名：`cyto`

注册码：`8765432112345678`

### 1. 对输入的注册码加密保存到注册表：

定位注册按钮：

```
005080E8   55                     push    ebp
```

对注册码长度的判断：

```
0050818B     8B45 FC            mov eax,dword ptr ss:[ebp-4]
0050818E     E8 11CAEFFF        call MeteorNe.00404BA4
00508193     83E8 10            sub eax,10
00508196     74 18              je short MeteorNe.005081B0
00508198     83E8 04            sub eax,4
0050819B     74 4B              je short MeteorNe.005081E8
0050819D     83E8 04            sub eax,4
005081A0     74 7B              je short MeteorNe.0050821D
005081A2     83E8 08            sub eax,8
005081A5     0F84 A7000000      je MeteorNe.00508252
005081AB     E9 88010000        jmp MeteorNe.00508338
```

看样子对输入的注册码有要求，可以为 16 位，20 位，24 位，32 位，得到不同的值，可能对应不同版，比如标准版，豪华版等等。

#### 输入注册码 16 位：

```
005081B9     8B45 FC       mov eax,dword ptr ss:[ebp-4]        ; 待加密值
005081BC     E8 1334F7FF   call MeteorNe.0047B5D4               ; MD5
005081C1     8B45 D4       mov eax,dword ptr ss:[ebp-2C]
005081C4     8D4D D8       lea ecx,dword ptr ss:[ebp-28]
005081C7     BA A4835000   mov edx,MeteorNe.005083A4            ; ASCII "Impressions"
005081CC     E8 1726F7FF   call MeteorNe.0047A7E8               ; DES
005081D1     8B45 D8       mov eax,dword ptr ss:[ebp-28]        ; 加密值
005081D4     B9 0A000000   mov ecx,0A
005081D9     BA 06000000   mov edx,6
005081DE     E8 21CCEFFF   call MeteorNe.00404E04               ; 取值
```

硬件访问断点可以跟踪到 `DES` 加解密的地方：

```
0047A2D0     8B45 0C              mov eax,dword ptr ss:[ebp+C]       ; 待加密的数据
0047A2D3     8B55 08              mov edx,dword ptr ss:[ebp+8]
0047A2D6     E8 81F9FFFF          call MeteorNe.00479C5C
0047A2DB     84DB                 test bl,bl
0047A2DD     0F85 B5000000        jnz MeteorNe.0047A398               ; 判断加密或解密
```

因为跟过好多 `DES`，看到数据窗口显示两排一样的值，感觉就是 `DES` 加密，然后代码窗口往下就看到了可爱的加解密判断。

比如输入练码：`8765432112345678`，

`MD5(注册码)`：堆栈 `ss:[0012FA0C]=00E13690`, `(ASCII "b367880002f343b544d1c40e2a5e4c55")`

`DES` 加密值：堆栈 `ss:[0012FA10]=00E1F9C4`, `(ASCII "367C0DBDA07EDD2B7FE9B592DD04FC4F52B4075FE9E30F203DF448112325EB0D")`

取值：`DBDA07EDD2`

#### 如果输入注册码为 20 位：

比如练码：`87654321123456788888`

`MD5`：堆栈 `ss:[0012FA04]=00E1E554`, `(ASCII "e4cff04950d1de18ec1a59fe8ec55b25")`

`DES`：堆栈 `ss:[0012FA08]=00E201BC`, `(ASCII "76F759CDA212914538DE07F2FC3A3825EA411498F0A27B9B00458A4EBE556AC2")`

取值：`F759CDA2129145`

#### 如果注册码为 24 位：

比如练码：`876543211234567888884444`

`MD5`：堆栈 `ss:[0012F9FC]=00E202EC`, `(ASCII "87249b9fe34297a230c0004a14062dc4")`

`DES`：堆栈 `ss:[0012FA00]=00E21454`, `(ASCII "956C42F9EF9A06A427F6D73543BBD4A632AF6ED7E98AFED82CC512DA66D3F99B")`

取值：`42F9EF9A06A427F6`

#### 如果注册码为 32 位：

比如练码为：`87654321123456788765432112345678`

`MD5`：堆栈 `ss:[0012F9F4]=00E21660`, `(ASCII "50ce9374eb73de35f8439b25a5bee3e2")`

`DES`：堆栈 `ss:[0012F9F8]=00E2177C`, `(ASCII "DDE34549170DF92BA0AA66896DEFA36D1A86CC0B7D2EAB9B2C1A9F1611537EBC")`

取值：`DF92BA0AA668`

算法很清楚，就是对输入的注册码 `MD5(注册码)`，然后 `DES()`，`key` 可能为 `Impressions`，然后根据 `xx` 版要去取值。

即经过运算为：`Char[DES(MD5(注册码),Impressions)]`

注册码长度与注册表里值的长度关系：16 对应 10，24 对应 14，28 对应 16，32 对应 12。

写入注册表，分别为：

`HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\olympic\pic`  `SUCCESS`  `"DBDA07EDD2"`

`HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\olympic\pic`  `SUCCESS`  `"F759CDA2129145"`

`HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\olympic\pic`  `SUCCESS`  `"42F9EF9A06A427F6"`

`HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\olympic\pic`  `SUCCESS`  `"DF92BA0AA668"`

### 2. 算法之旅：

搜索 `pic`：

```
0050409D     BA CC455000      mov edx,MeteorNe.005045CC            ; ASCII "pic"
```

#### 2.1 用户名＋机器码的运算：[用户名＋机器码运算中间值]

`MD5(cyto)`：

```
00504124     8B83 A0040000        mov eax,dword ptr ds:[ebx+4A0]
0050412A     E8 A574F7FF          call MeteorNe.0047B5D4
```

```
0047B38C     C700 01234567        mov dword ptr ds:[eax],67452301
0047B392     C740 04 89ABCDEF     mov dword ptr ds:[eax+4],EFCDAB89
0047B399     C740 08 FEDCBA98     mov dword ptr ds:[eax+8],98BADCFE
0047B3A0     C740 0C 76543210     mov dword ptr ds:[eax+C],10325476
```

堆栈 `ss:[0012FD50]=00E15EE0`, `(ASCII "369d1922e56a67d428fedd5c31ee6887")`

#### 2.2 XOR(MD5(cyto))：

```
00504132     BA 40465000      mov edx,MeteorNe.00504640             ; ASCII "killyou?yes"
00504137     8B45 FC          mov eax,dword ptr ss:[ebp-4]         ; MD5(cyto)
0050413A     E8 25B0FDFF      call MeteorNe.004DF164
0050413F     8B55 E8          mov edx,dword ptr ss:[ebp-18]        ; 计算值
```

进入 `call`：

```
004DF2EA     3245 E7          xor al,byte ptr ss:[ebp-19]
```

参数：

`00504640=MeteorNe.00504640` `(ASCII "killyou?yes")`

堆栈 `ss:[0012FD50]=00E15EE0`, `(ASCII "369d1922e56a67d428fedd5c31ee6887")`

运算：

顺取 `MD5(cyto)` 的字符，与内置字符串的第一个字符异或，循环次数为 32，得到结果

异或结果再与内置字符串第二个字符异或，一直循环到内置字符串结束。

总共循环：11×32。

最后值：

堆栈 `ss:[0012FD3C]=00E1767C`, `(ASCII "0207085500080303540407500706550503095754555504520200545407090906")`

#### 2.3 对机器码的 DES 加密：DES(机器码)

```
0050414D     BA 54465000      mov edx,MeteorNe.00504654            ; ASCII "http://www.jesen.cn"
00504152     8B83 A4040000    mov eax,dword ptr ds:[ebx+4A4]       ; 机器码"9030d2d9"
00504158     E8 8B66F7FF      call MeteorNe.0047A7E8               ; DES 加密
0050415D     8B55 E0          mov edx,dword ptr ss:[ebp-20]        ; 计算值
```

参数：机器码`"9030d2d9"`

结果：`ASCII "6DB43263A7568F1D"`

#### 2.4 对用户名计算值的 DES：DES(XOR(MD5(cyto)),DES(机器码))

```
0050415D     8B55 E0           mov edx,dword ptr ss:[ebp-20]         ; ASCII "6DB43263A7568F1D")
00504160     8D4D E4           lea ecx,dword ptr ss:[ebp-1C]
00504163     8B45 FC           mov eax,dword ptr ss:[ebp-4]          ; 用户名计算值
00504166     E8 7D66F7FF       call MeteorNe.0047A7E8
0050416B     8B55 E4           mov edx,dword ptr ss:[ebp-1C]         ; 加密值
```

这个是分步加密的，因为用户名计算值太大，8 个 8 个加密，比如：`02070855 00080303 ...`。

参数：

用户名计算值：`0207085500080303540407500706550503095754555504520200545407090906`

机器码加密值：`ASCII "6DB43263A7568F1D"`

加密值：堆栈 `ss:[0012FD38]=00E15F68`, `(ASCII "4C9BC7F1450EB1201E2D682D30498EC50B6DFB3F15DBC30E86FE6CD2DC0B310ED5D0A0D2112D64D64F667A2CAB3110E115C40D12F2C93C13573DA61D89B2D6CA")`

#### 2.5 然后对这个 DES 值 MD5 加密：MD5(DES(XOR(MD5(cyto)),DES(机器码)))

```
00504179     8B45 FC           mov eax,dword ptr ss:[ebp-4]          ; 用户名计算值的 DES 加密值
0050417C     E8 5374F7FF       call MeteorNe.0047B5D4                ; md5
00504181     8B55 DC           mov edx,dword ptr ss:[ebp-24]         ; 值
```

参数：

`4C9BC7F1450EB1201E2D682D30498EC50B6DFB3F15DBC30E86FE6CD2DC0B310ED5D0A0D2112D64D64F667A2CAB3110E115C40D12F2C93C13573DA61D89B2D6CA`

值：堆栈 `ss:[0012FD30]=00E15FF8`, `(ASCII "28528b2dd30e89192bb06c5e100741e2")`

这个暂时称为：用户名＋机器码运算中间值，

`[用户名＋机器码运算中间值]`＝`MD5(DES(XOR(MD5(cyto)),DES(机器码)))`

#### 2.6 运算分水岭：

判断注册表里注册码的长度，运算分水岭：

```
0050418C     8B83 98040000   mov eax,dword ptr ds:[ebx+498]
00504192     E8 0D0AF0FF     call MeteorNe.00404BA4
00504197     83E8 0A         sub eax,0A
0050419A     74 62           je short MeteorNe.005041FE       ; 标准版    
0050419C     83E8 02         sub eax,2
0050419F     74 17           je short MeteorNe.005041B8       ; 
005041A1     83E8 02         sub eax,2
005041A4     0F84 18010000   je MeteorNe.005042C2              ; VIP 版
005041AA     83E8 02         sub eax,2
005041AD     0F84 D3010000   je MeteorNe.00504386              ; 钻石版
005041B3     E9 A5030000     jmp MeteorNe.0050455D
```

根据注册表的加密值长度进行不同的运算。

因为我们是按照 16 位注册码来跟踪的，因此在 `0050419A` 处跳转。



## 2.7 根据要求取值：`Char[用户名＋机器码运算中间值]`

```
00504207      B9 05000000        mov ecx,5
0050420C      BA 10000000        mov edx,10
00504211      8B45 FC           mov eax,dword ptr ss:[ebp-4]
00504214      E8 EB0BF0FF        call MeteorNe.00404E04                ; 根据要求取值
00504219      FF75 C4            push dword ptr ss:[ebp-3C]           ; 92bb0
0050421C      8D45 C0            lea eax,dword ptr ss:[ebp-40]
0050421F      50                 push eax
00504220      B9 04000000        mov ecx,4
00504225      BA 04000000        mov edx,4
0050422A      8B45 FC            mov eax,dword ptr ss:[ebp-4]
0050422D      E8 D20BF0FF        call MeteorNe.00404E04
00504232      FF75 C0            push dword ptr ss:[ebp-40]           ; 28b2
00504235      8D45 BC            lea eax,dword ptr ss:[ebp-44]
00504238      50                 push eax
00504239      B9 04000000        mov ecx,4
0050423E      BA 12000000        mov edx,12
00504243      8B45 FC            mov eax,dword ptr ss:[ebp-4]
00504246      E8 B90BF0FF        call MeteorNe.00404E04
0050424B      FF75 BC            push dword ptr ss:[ebp-44]          ; bb06
0050424E      8D45 B8            lea eax,dword ptr ss:[ebp-48]
00504251      50                 push eax
00504252      B9 03000000        mov ecx,3
00504257      BA 1D000000        mov edx,1D
0050425C      8B45 FC            mov eax,dword ptr ss:[ebp-4]
0050425F      E8 A00BF0FF        call MeteorNe.00404E04
00504264      FF75 B8            push dword ptr ss:[ebp-48]          ; 41e
00504267      8D45 C8            lea eax,dword ptr ss:[ebp-38]
0050426A      BA 04000000        mov edx,4
0050426F      E8 F009F0FF        call MeteorNe.00404C64                ; 连接
00504274      8B45 C8            mov eax,dword ptr ss:[ebp-38]       ; (ASCII "92bb028b2bb0641e")
```

参数：`28528b2dd30e89192bb06c5e100741e2`

得到：`92bb028b2bb0641e`

## 2.8 再次 MD5：`MD5(Char[用户名＋机器码运算中间值])`

```
00504274      8B45 C8            mov eax,dword ptr ss:[ebp-38]       ; (ASCII "92bb028b2bb0641e")
00504277      8D55 CC            lea edx,dword ptr ss:[ebp-34]
0050427A      E8 5573F7FF        call MeteorNe.0047B5D4               ; MD5
0050427F      8B45 CC            mov eax,dword ptr ss:[ebp-34]       ; 值
```

参数：`92bb028b2bb0641e`

值：堆栈 `ss:[0012FD20]=00E16060`, `(ASCII "d8ffa201ad4be2508736a9d632ff8e05")`

## 2.9 DES 加密：`DES(MD5(Char[用户名＋机器码运算中间值]))`

```
0050427F      8B45 CC            mov eax,dword ptr ss:[ebp-34]       ; 待加密数据
00504282      8D4D D0            lea ecx,dword ptr ss:[ebp-30]
00504285      BA 70465000        mov edx,MeteorNe.00504670           ; ASCII "Impressions"
0050428A      E8 5965F7FF        call MeteorNe.0047A7E8               ; DES
0050428F      8B45 D0            mov eax,dword ptr ss:[ebp-30]       ; 值
```

参数：`d8ffa201ad4be2508736a9d632ff8e05`

结果：

堆栈 `ss:[0012FD24]=00E160C0`, `(ASCII "0B8412DE40B70E5B7B18714BB5984459CE91633C25CCBC63450818C6348DF172")`

## 2.10 取值：`Char[DES(MD5(Char[用户名＋机器码运算中间值]))]`

```
0050428F      8B45 D0            mov eax,dword ptr ss:[ebp-30]       ; 值
00504292      B9 0A000000        mov ecx,0A
00504297      BA 06000000        mov edx,6
0050429C      E8 630BF0FF        call MeteorNe.00404E04
```

参数：`0B8412DE40B70E5B7B18714BB5984459CE91633C25CCBC63450818C6348DF172`

结果：`2DE40B70E5`

## 2.11 取值：`Char[Char[DES(MD5(Char[用户名＋机器码运算中间值]))]]`

```
0050446B      B9 06000000        mov ecx,6
00504470      BA 04000000        mov edx,4
00504475      A1 AC115100        mov eax,dword ptr ds:[5111AC]
0050447A      E8 8509F0FF        call MeteorNe.00404E04
0050447F      8B85 70FFFFFF      mov eax,dword ptr ss:[ebp-90]
```

参数：`2DE40B70E5`

结果：堆栈 `ss:[0012FCC4]=00E16148`, `(ASCII "40B70E")`

## 2.12 转换：`Change[Char[Char[DES(MD5(Char[用户名＋机器码运算中间值]))]]]`

```
0050447F      8B85 70FFFFFF      mov eax,dword ptr ss:[ebp-90]       ; (ASCII "40B70E")
00504485      8D95 74FFFFFF      lea edx,dword ptr ss:[ebp-8C]
0050448B      E8 38ACFDFF        call MeteorNe.004DF0C8               ; 转换
00504490      8B85 74FFFFFF      mov eax,dword ptr ss:[ebp-8C]       ; (ASCII "343042373045")
```

## 2.13 DES 加密：`DES(Change[Char[Char[DES(MD5(Char[用户名＋机器码运算中间值]))]]])`

```
00504490      8B85 74FFFFFF      mov eax,dword ptr ss:[ebp-8C]       ; (ASCII "343042373045")
00504496      8D8D 78FFFFFF      lea ecx,dword ptr ss:[ebp-88]
0050449C      BA D0465000        mov edx,MeteorNe.005046D0           ; ASCII "sgpj-nmmd"
005044A1      E8 4263F7FF        call MeteorNe.0047A7E8               ; DES 加密
005044A6      8B85 78FFFFFF      mov eax,dword ptr ss:[ebp-88]       ; 值
```

参数：`343042373045`

结果：堆栈 `ss:[0012FCCC]=00E161B8`, `(ASCII "4647E7E41E3B465B2C30516C9360B162")`

这样就得到了最后比较的参数之一：`eax=4647E7E41E3B465B2C30516C9360B162`

得到这个值经过的运算为：

`DES(Change[Char[Char[DES(MD5(Char[用户名＋机器码运算中间值]))]]])`

其中 `[用户名＋机器码运算中间值]＝MD5(DES(XOR(MD5(cyto)),DES(机器码)))`

整合在一起就是：`DES(Change[Char[Char[DES(MD5(Char[MD5(DES(XOR(MD5(cyto)),DES(机器码)))]))]]])`

经过 4 次 `DES` ＋ 3 次 `MD5` ＋ 取值变换等。

## 2.14 取注册表，对注册码加密值的运算：

取值：`Char[注册码加密值]`

```
005044B4      B9 06000000        mov ecx,6
005044B9      BA 04000000        mov edx,4
005044BE      8B83 98040000      mov eax,dword ptr ds:[ebx+498]     ; 注册码(ASCII "DBDA07EDD2")
005044C4      E8 3B09F0FF        call MeteorNe.00404E04
005044C9      8B85 64FFFFFF      mov eax,dword ptr ss:[ebp-9C]      ; (ASCII "A07EDD")
```

参数：注册码在注册表里的值 `(ASCII "DBDA07EDD2")`

结果：`(ASCII "A07EDD")`

## 2.15 转换：`Change[Char[注册码加密值]]`

```
005044C9      8B85 64FFFFFF      mov eax,dword ptr ss:[ebp-9C]      ; (ASCII "A07EDD")
005044CF      8D95 68FFFFFF      lea edx,dword ptr ss:[ebp-98]
005044D5      E8 EEABFDFF        call MeteorNe.004DF0C8               ; 转换
005044DA      8B85 68FFFFFF      mov eax,dword ptr ss:[ebp-98]      ; (ASCII "413037454444")
```

## 2.16 DES 加密：`DES(Change[Char[注册码加密值]])`

```
005044DA      8B85 68FFFFFF      mov eax,dword ptr ss:[ebp-98]      ; (ASCII "413037454444")
005044E0      8D8D 6CFFFFFF      lea ecx,dword ptr ss:[ebp-94]
005044E6      BA D0465000        mov edx,MeteorNe.005046D0          ; ASCII "sgpj-nmmd"
005044EB      E8 F862F7FF        call MeteorNe.0047A7E8               ; DES 加密
005044F0      8B95 6CFFFFFF      mov edx,dword ptr ss:[ebp-94]      ; 值
```

参数：`413037454444`

结果：得到最后比较的参数之一

堆栈 `ss:[0012FCC0]=00E16258`, `(ASCII "D5EB524F0C87E52266E9EA42B23E3DB1")`

这个值经过的运算：`DES(Change[Char[注册码加密值]])`，

其中注册码加密值＝`Char[DES(MD5(注册码),Impressions)]`

## 2.17 这样得到了 2 个比较的参数，往下来到比较的地方：

```
005044F0      8B95 6CFFFFFF    mov edx,dword ptr ss:[ebp-94]
005044F6      58                pop eax
005044F7      E8 F407F0FF       call MeteorNe.00404CF0
005044FC      75 5F             jnz short MeteorNe.0050455D
```

如果 `eax=edx`，那么就通过注册。

## 3. 反推注册码：

开始输入的注册码经过 `md5` 加密＋`DES` 加密，好像不能反推，可是看看对用户名＋机器码的运算就可以发现直接可以得到注册码，连反推都省了。

在对用户名＋机器码的运算是这样的：

`DES(Change[Char[Char[DES(MD5(Char[MD5(DES(XOR(MD5(cyto)),DES(机器码)))])),Impressions)]],sgpj-nmmd)`

对注册码的运算是这样的：

`DES(Change[Char[Char[DES(MD5(注册码),Impressions)]],sgpj-nmmd)`

因为 `key` 相同，而且取值也是一样，只要注册码＝`Char[MD5(DES(XOR(MD5(cyto)),DES(机器码)))]`就可以，

也就是：`Char[用户名＋机器码运算中间值]＝92bb028b2bb0641e＝92BB028B2BB0641E`

机器码：`9030D2D9`

用户名：`cyto`

注册码：`92BB028B2BB0641E`

呵呵，提示注册给我了：标准版。

其他长度的注册码的运算大通小异，只是在运算分水岭上走向不同而已，也同样可以反推得到注册码。

`4dd466d0e1e0cc123afbe504da2462ca`
