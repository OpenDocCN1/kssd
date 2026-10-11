
# 公务员之路 5.1

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-392.htm`](https://www.pediy.com/kssd/pediy08/pediy8-392.htm)

秋季`gwy`招募时间到了，破文一篇以示纪念。

公务员之路 5.1

`ASPack 2.12` -> `Alexey Solodovnikov`

`Borland Delphi 4.0 - 5.0`

机器码：`GWY785775891301844`

## 1. 前期准备

### 注册按钮

```
004D5C9C     55                   push ebp
```

输入的注册码保存在所在文件夹下：

```
004D5394   mov edx,gwyzl.004D542C       ; ASCII "sysdata\\data0\\data.dat"
```

内容：

```
[Regsoft Info]
regno=87654321
```

算法查询到：`FGintRSA`。

搜索：`data.dat`，3 处，全部下断

运行程序断下：

```
00561BBC    mov edx,dumped.00561C5C      ; ASCII "sysdata\\data0\\data.dat"
```

返回后：

```
00560F2D     E8 560C0000        call dumped.00561B88            ; 取得注册码
00560F32     8D55 F4          lea edx,dword ptr ss:[ebp-C]
00560F35     B8 A0155600      mov eax,dumped.005615A0          ; ASCII "901158060733780652060333"
00560F3A     E8 B998EFFF      call dumped.0045A7F8
00560F3F     8D55 EC          lea edx,dword ptr ss:[ebp-14]
00560F42     B8 C4155600      mov eax,dumped.005615C4          ; ASCII "420278286234780046392679"
```

初步判断：

`N = 901158060733780652060333 (D) = BED3E05921571679EEAD (H)`

`E = 420278286234780046392679 (D) = 58FF54E068BD181EFD67 (H)`

## 2. 获得参数之一：注册码计算值

### 注册码的初步处理

```
00560F5E     E8 A194EFFF      call dumped.0045A404
```

将注册码查表转换：

```
0045A475     8D45 F8          lea eax,dword ptr ss:[ebp-8]
0045A478     8B55 FC          mov edx,dword ptr ss:[ebp-4]
0045A47B     0FB65432 FF      movzx edx,byte ptr ds:[edx+esi-1]
0045A480     8B9495 F4FBFFFF  mov edx,dword ptr ss:[ebp+edx*4-40C]   ; 这里取值
0045A487     E8 709CFAFF      call gwyzl.004040FC
0045A48C     46             inc esi
0045A48D     4B             dec ebx
0045A48E   ^ 75 E5          jnz short gwyzl.0045A475
```

对应关系：

`0=110100`；`1=110101`；`2=110110`；`3=110111`；`4=111000`；`5=111001`；
`6=111010`；`7=111011`；`8=111100`；`9=111101`。
`a=000000`；`b=000010`；`c=000100`；`d=000110`；`e=001000`；`f=001010`；
`g=001100`；`h=001110`；`i=010000`；`j=010010`；`k=010100`；`l=010110`；
`m=011000`；`n=011010`；`o=011100`；`p=011110`；`q=100000`；`r=100010`；
`s=100100`；`t=100110`；`u=101000`；`v=101010`；`w=101100`；`x=101110`；
`y=110000`；`z=110010`。
`A=000001`；`B=000011`；`C=000101`；`D=000111`；`E=001001`；`F=001011`；
`G=001101`；`H=001111`；`I=010001`；`J=010011`；`K=010101`；`L=010111`；
`M=011001`；`N=011011`；`O=011101`；`P=011111`；`Q=100001`；`R=100011`；
`S=100101`；`T=100111`；`U=101001`；`V=101011`；`W=101101`；`X=101111`；
`Y=110001`；`Z=110011`
`"="="111111"`；`"+"="111110"`

练码：`87654321`

转换值：`111100 111011 111010 111001 111000 110111 110110 110101`

然后取 `8bit` 转换成十六进制：

```
0045A4AF     8D85 F0FBFFFF    lea eax,dword ptr ss:[ebp-410]
0045A4B5     50             push eax
0045A4B6     B9 08000000     mov ecx,8
0045A4BB     BA 01000000     mov edx,1
0045A4C0     8B45 F8        mov eax,dword ptr ss:[ebp-8]
0045A4C3     E8 349EFAFF     call gwyzl.004042FC
0045A4C8     8B95 F0FBFFFF   mov edx,dword ptr ss:[ebp-410]
0045A4CE     8D45 F7        lea eax,dword ptr ss:[ebp-9]
0045A4D1     E8 2AFAFFFF     call gwyzl.00459F00
0045A4D6     8D85 ECFBFFFF   lea eax,dword ptr ss:[ebp-414]
0045A4DC     8A55 F7        mov dl,byte ptr ss:[ebp-9]
0045A4DF     E8 389BFAFF     call gwyzl.0040401C
0045A4E4     8B95 ECFBFFFF   mov edx,dword ptr ss:[ebp-414]
0045A4EA     8BC7           mov eax,edi
0045A4EC     E8 0B9CFAFF     call gwyzl.004040FC
0045A4F1     8D45 F8        lea eax,dword ptr ss:[ebp-8]
0045A4F4     B9 08000000     mov ecx,8
0045A4F9     BA 01000000     mov edx,1
0045A4FE     E8 399EFAFF     call gwyzl.0040433C
0045A503     4B             dec ebx
0045A504   ^ 75 A9          jnz short gwyzl.0045A4AF
```

原值：`111100 111011 111010 111001 111000 110111 110110 110101`

转换：`0100E120 F3 BE B9 E3 7D B5` 缶广}?.

### 加密：`RSA`

```
00560F80     8B80 1C030000   mov eax,dword ptr ds:[eax+31C]  ; 待加密值
00560F86     E8 C1BCEFFF     call dumped.0045CC4C
```

这里待加密值就是注册码的初步处理值。

#### 加密代码

```
0045CDE9     8D45 AC        lea eax,dword ptr ss:[ebp-54]
0045CDEC     50            push eax
0045CDED     8BCB          mov ecx,ebx
0045CDEF     BA 01000000   mov edx,1
0045CDF4     8B45 B4       mov eax,dword ptr ss:[ebp-4C]
0045CDF7     E8 0075FAFF   call gwyzl.004042FC
0045CDFC     EB 12        jmp short gwyzl.0045CE10

0045CDFE     8D45 AC        lea eax,dword ptr ss:[ebp-54]
0045CE01     B9 01000000   mov ecx,1
0045CE06     BA 01000000   mov edx,1
0045CE0B     E8 2C75FAFF   call gwyzl.0040433C

0045CE10     8D45 A4        lea eax,dword ptr ss:[ebp-5C]
0045CE13     50            push eax
0045CE14     B9 01000000   mov ecx,1
0045CE19     BA 01000000   mov edx,1
0045CE1E     8B45 AC        mov eax,dword ptr ss:[ebp-54]
0045CE21     E8 D674FAFF   call gwyzl.004042FC
0045CE26     8B45 A4        mov eax,dword ptr ss:[ebp-5C]
0045CE29     BA 34D04500   mov edx,gwyzl.0045D034           ; 是否为 0？
0045CE2E     E8 D173FAFF   call gwyzl.00404204
0045CE33     75 0B        jnz short gwyzl.0045CE40        ; 非 0，跳转实现
0045CE35     8B45 AC        mov eax,dword ptr ss:[ebp-54]
0045CE38     E8 B772FAFF   call gwyzl.004040F4
0045CE3D     48            dec eax                         ; 50
0045CE3E   ^ 7F BE        jg short gwyzl.0045CDFE          ; 继续检查下一位是否为 1

0045CE40     8D55 F0        lea edx,dword ptr ss:[ebp-10]
0045CE43     8B45 AC        mov eax,dword ptr ss:[ebp-54]    ; 分组的待加密值
0045CE46     E8 41E9FFFF   call gwyzl.0045B78C
0045CE4B     8D45 B4        lea eax,dword ptr ss:[ebp-4C]
0045CE4E     8BCB          mov ecx,ebx
0045CE50     BA 01000000   mov edx,1
0045CE55     E8 E274FAFF   call gwyzl.0040433C
0045CE5A     8B45 AC        mov eax,dword ptr ss:[ebp-54]
0045CE5D     BA 34D04500   mov edx,gwyzl.0045D034
0045CE62     E8 9D73FAFF   call gwyzl.00404204
0045CE67     75 10        jnz short gwyzl.0045CE79
0045CE69     8D55 E8        lea edx,dword ptr ss:[ebp-18]
0045CE6C     8D45 B8        lea eax,dword ptr ss:[ebp-48]
0045CE6F     E8 4CE2FFFF   call gwyzl.0045B0C0
0045CE74     E9 91000000   jmp gwyzl.0045CF0A

0045CE79     837F 04 00     cmp dword ptr ds:[edi+4],0
0045CE7D     74 13        je short gwyzl.0045CE92
0045CE7F     8D45 E8        lea eax,dword ptr ss:[ebp-18]
0045CE82     50            push eax
0045CE83     8B4D F8        mov ecx,dword ptr ss:[ebp-8]
0045CE86     8BD7          mov edx,edi
0045CE88     8D45 F0        lea eax,dword ptr ss:[ebp-10]
0045CE8B     E8 44F7FFFF   call gwyzl.0045C5D4
0045CE90     EB 78        jmp short gwyzl.0045CF0A

...

0045CF0A     8D45 F0        lea eax,dword ptr ss:[ebp-10]
0045CF0D     E8 5EDBFFFF   call gwyzl.0045AA70
0045CF12     8D45 AC        lea eax,dword ptr ss:[ebp-54]
0045CF15     E8 5A6FFAFF   call gwyzl.00403E74
0045CF1A     8D55 AC        lea edx,dword ptr ss:[ebp-54]
0045CF1D     8D45 E8        lea eax,dword ptr ss:[ebp-18]
0045CF20     E8 83E7FFFF   call gwyzl.0045B6A8
0045CF25     EB 10        jmp short gwyzl.0045CF37

0045CF27     8D45 AC        lea eax,dword ptr ss:[ebp-54]
0045CF2A     8B4D AC        mov ecx,dword ptr ss:[ebp-54]
0045CF2D     BA 34D04500   mov edx,gwyzl.0045D034
0045CF32     E8 0972FAFF   call gwyzl.00404140

0045CF37     8B45 AC        mov eax,dword ptr ss:[ebp-54]    ; 得到的加密值
0045CF3A     E8 B571FAFF   call gwyzl.004040F4
0045CF3F     8BD3          mov edx,ebx
0045CF41     4A            dec edx
0045CF42     8BCA          mov ecx,edx
0045CF44     99            cdq
0045CF45     F7F9          idiv ecx                            ; 4F
0045CF47     85D2          test edx,edx
```



# 反汇编分析与加密逻辑说明

## 核心加密循环片段

```asm
0045CF49   ^ 75 DC          jnz short gwyzl.0045CF27           ; 补足加密值为 4F 位，十进制为 79 位  

0045CF4B     8D45 B0         lea eax,dword ptr ss:[ebp-50]  

0045CF4E     8B55 AC         mov edx,dword ptr ss:[ebp-54]  

0045CF51     E8 A671FAFF     call gwyzl.004040FC  

0045CF56     8D45 E8         lea eax,dword ptr ss:[ebp-18]  

0045CF59     E8 12DBFFFF     call gwyzl.0045AA70  

0045CF5E     4E              dec esi                            ; esi 为加密次数，即 m 的分组数  

0045CF5F   ^ 0F85 84FEFFFF  jnz gwyzl.0045CDE9                 ; 不为零继续下一组加密  
```

## 数据验证与计算

**待加密值**：`111100111011111010111001111000110111110110110101`

**加密值**：`1000111001010010010100000100110100011101111000001111101011001101100101101000111` ＝ `472928268EF07D66CB47`

**验证**：

- `N` = `BED3E05921571679EEAD` (H)
- `E` = `58FF54E068BD181EFD67` (H)
- `M` = `F3BEB9E37DB5`
- `C` = `M^E MOD(N)` = `472928268EF07D66CB47`

## 查找首个 `111` 并处理

**操作**：找第一个 `111` 并去除前面的字符。

```asm
0045CF7D     8D45 B0          lea eax,dword ptr ss:[ebp-50]  

0045CF80     B9 01000000      mov ecx,1  

0045CF85     BA 01000000      mov edx,1  

0045CF8A     E8 AD73FAFF      call dumped.0040433C  

0045CF8F     8D45 A0          lea eax,dword ptr ss:[ebp-60]  

0045CF92     50              push eax  

0045CF93     B9 03000000      mov ecx,3  

0045CF98     BA 01000000      mov edx,1  

0045CF9D     8B45 B0          mov eax,dword ptr ss:[ebp-50]  

0045CFA0     E8 5773FAFF      call dumped.004042FC  

0045CFA5     8B45 A0          mov eax,dword ptr ss:[ebp-60]  

0045CFA8     BA 40D04500      mov edx,dumped.0045D040         ; ASCII "111"  

0045CFAD     E8 5272FAFF      call dumped.00404204  

0045CFB2     74 0D            je short dumped.0045CFC1  

0045CFB4     8B45 B0          mov eax,dword ptr ss:[ebp-50]  

0045CFB7     E8 3871FAFF      call dumped.004040F4  

0045CFBC     83F8 03          cmp eax,3  

0045CFBF   ^ 7F BC           jg short dumped.0045CF7D  
```

**待处理值**：`1000111001010010010100000100110100011101111000001111101011001101100101101000111`

**处理后值**：`001010010010100000100110100011101111000001111101011001101100101101000111`

**十六进制**：`2928268EF07D66CB47`

## 加密值转换为字符 (Base64)

**操作**：加密值转换成字符，用到 `Base64`。

```asm
00560F97     E8 E892EFFF      call dumped.0045A284  
```

## 调整加密值长度为 6 的倍数

**操作**：加密值的调整，使为 6 的倍数。

```asm
0045A2F5     8D45 F8         lea eax,dword ptr ss:[ebp-8]  

0045A2F8     8B55 FC         mov edx,dword ptr ss:[ebp-4]  

0045A2FB     0FB65432 FF    movzx edx,byte ptr ds:[edx+esi-1]  

0045A300     8B9495 F4FBFFFF mov edx,dword ptr ss:[ebp+edx*4-40C] ; 转换成 2 进制  

0045A307     E8 F09DFAFF    call dumped.004040FC  

0045A30C     46            inc esi  

0045A30D     4B            dec ebx  

0045A30E   ^ 75 E5        jnz short dumped.0045A2F5  
...
0045A31F     8B45 F8         mov eax,dword ptr ss:[ebp-8]      ; JIAMIZHI  

0045A322     E8 CD9DFAFF    call dumped.004040F4  

0045A327     B9 06000000    mov ecx,6  

0045A32C     99            cdq  

0045A32D     F7F9          idiv ecx  

0045A32F     85D2          test edx,edx  

0045A331   ^ 75 DF        jnz short dumped.0045A312          ; 调整到 6 的倍数  
```

**说明**：如果加密值的 2 进制长度不够 6 整除，在后面添 `0`。

## 6-bit 分组与查表转换

**操作**：取 6bit 转换，再查表。

```asm
0045A350     8D85 F0FBFFFF   lea eax,dword ptr ss:[ebp-410]  

0045A356     50            push eax  

0045A357     B9 06000000    mov ecx,6  

0045A35C     BA 01000000    mov edx,1  

0045A361     8B45 F8        mov eax,dword ptr ss:[ebp-8]  

0045A364     E8 939FFAFF    call dumped.004042FC  

0045A369     8B95 F0FBFFFF  mov edx,dword ptr ss:[ebp-410]  

0045A36F     8D45 F4        lea eax,dword ptr ss:[ebp-C]  

0045A372     E8 1DFCFFFF    call dumped.00459F94  

0045A377     8D85 ECFBFFFF  lea eax,dword ptr ss:[ebp-414]  

0045A37D     8B55 F4        mov edx,dword ptr ss:[ebp-C]  

0045A380     8A92 6F3B5600  mov dl,byte ptr ds:[edx+563B6F] ; 查表  

0045A386     E8 919CFAFF    call dumped.0040401C  

0045A38B     8B95 ECFBFFFF  mov edx,dword ptr ss:[ebp-414]  

0045A391     8BC7           mov eax,edi  

0045A393     E8 649DFAFF    call dumped.004040FC  

0045A398     8D45 F8        lea eax,dword ptr ss:[ebp-8]  

0045A39B     B9 06000000    mov ecx,6  

0045A3A0     BA 01000000    mov edx,1  

0045A3A5     E8 929FFAFF    call dumped.0040433C  

0045A3AA     4B            dec ebx  

0045A3AB   ^ 75 A3        jnz short dumped.0045A350  
```

**核心逻辑**：集中在 `ds:[edx+563B6F]`

**`edx` 的来源**：

**加密值** = `2928268EF07D66CB47` = `001010010010100000100110100011101111000001111101011001101100101101000111`

**拆分为 6bit**：`001010 010010 100000 100110 100011 101111 000001 111101 011001 101100 101101 000111`

**找对应关系得到 `edx`**：

```
0012F6BC   00FFBA6C    ASCII "00000000"     // 1  
0012F6C0   00F724D0    ASCII "00000001"     // 2  
0012F6C4   00FFE2E0    ASCII "00000010"     // 3  
0012F6C8   00FFE2F8    ASCII "00000011"     // 4  
0012F6CC   00FFE310    ASCII "00000100"     // 5  
0012F6D0   00FFE328    ASCII "00000101"     // 6  
0012F6D4   00FFE340    ASCII "00000110"     // 7  
0012F6D8   00FFE358    ASCII "00000111"     // 8  
0012F6DC   00FFE370    ASCII "00001000"     // 9  
0012F6E0   00FFE388    ASCII "00001001"     // A  
0012F6E4   00FFE3A0    ASCII "00001010"     // B  
0012F6E8   00FFE3B8    ASCII "00001011"     // C  
0012F6EC   00FFE3D0    ASCII "00001100"     // D  
0012F6F0   00FFE3E8    ASCII "00001101"     // E  
0012F6F4   00FFE400    ASCII "00001110"     // F  
0012F6F8   00FFE418    ASCII "00001111"     // 10  
0012F6FC   00FFE430    ASCII "00010000"     //  
0012F700   00FFE448    ASCII "00010001"     //  
0012F704   00FFE460    ASCII "00010010"     //  
0012F708   00FFE478    ASCII "00010011"     //  
0012F70C   00FFE490    ASCII "00010100"     //  
0012F710   00FFE4A8    ASCII "00010101"     //  
0012F714   00FFE4C0    ASCII "00010110"     //  
0012F718   00FFE4D8    ASCII "00010111"     //  
0012F71C   00FFE4F0    ASCII "00011000"     //  
0012F720   00FFE508    ASCII "00011001"     //  
0012F724   00FFE520    ASCII "00011010"     //  
0012F728   00FFE538    ASCII "00011011"     //  
0012F72C   00FFE550    ASCII "00011100"     //  
0012F730   00FFE568    ASCII "00011101"     //  
0012F734   00FFE580    ASCII "00011110"     //  
0012F738   00FFE598    ASCII "00011111"     // 20  
0012F73C   00FFE5B0    ASCII "00100000"     //  
0012F740   00FFE5C8    ASCII "00100001"     //  
0012F744   00FFE5E0    ASCII "00100010"     //  
0012F748   00FFE5F8    ASCII "00100011"     //  
0012F74C   00FFE610    ASCII "00100100"     //  
0012F750   00FFE628    ASCII "00100101"     //  
0012F754   00FFE640    ASCII "00100110"     //  
0012F758   00FFE658    ASCII "00100111"     //  
0012F75C   00FFE670    ASCII "00101000"     //  
0012F760   00FFE688    ASCII "00101001"     //  
0012F764   00FFE6A0    ASCII "00101010"     //  
0012F768   00FFE6B8    ASCII "00101011"     //  
0012F76C   00FFE6D0    ASCII "00101100"     //  
0012F770   00FFE6E8    ASCII "00101101"     //  
0012F774   00FFE700    ASCII "00101110"     //  
0012F778   00FFE718    ASCII "00101111"     // 30  
0012F77C   00FFE730    ASCII "00110000"     //  
0012F780   00FFE748    ASCII "00110001"     //  
0012F784   00FFE760    ASCII "00110010"     //  
0012F788   00FFE778    ASCII "00110011"     //  
0012F78C   00FFE790    ASCII "00110100"     //  
0012F790   00FFE7A8    ASCII "00110101"     //  
0012F794   00FFE7C0    ASCII "00110110"     //  
0012F798   00FFE7D8    ASCII "00110111"     //  
0012F79C   00FFE7F0    ASCII "00111000"     //  
0012F7A0   00FFE808    ASCII "00111001"     //  
0012F7A4   00FFE820    ASCII "00111010"     //  
0012F7A8   00FFE838    ASCII "00111011"     //  
0012F7AC   00FFE850    ASCII "00111100"     //  
0012F7B0   00FFE868    ASCII "00111101"     //  
0012F7B4   00FFE880    ASCII "00111110"     //  
0012F7B8   00FFE898    ASCII "00111111"     // 40  
```

后面还有 `n` 多，不过用不上了。

**查表地址**：`[563B6F]`



## 1. Base64 查表与加密值转换

```
00563B60                                                               C0  
00563B70  61 41 62 42 63 43 64 44 65 45 66 46 67 47 68 48  aAbBcCdDeEfFgGhH  
00563B80  69 49 6A 4A 6B 4B 6C 4C 6D 4D 6E 4E 6F 4F 70 50  iIjJkKlLmMnNoOpP  
00563B90  71 51 72 52 73 53 74 54 75 55 76 56 77 57 78 58  qQrRsStTuUvVwWxX  
00563BA0  79 59 7A 5A 30 31 32 33 34 35 36 37 38 39 2B 3D  yYzZ0123456789+=  
```

这个就是 Base64 的表。

加密值=`2928268EF07D66CB47`=`001010010010100000100110100011101111000001111101011001101100101101000111`

查表值：`00F72A6C  66 6A 71 74 52 58 41 39 4D 77 57 44                      fjqtRXA9MwWD`

### 3. 获得参数之二：硬盘号计算值

**取硬盘号：**

```
00560FAB     call gwyzl.0045DD24                    ; jmp to offset diskseri.GetDiskSerial
```

`eax=1000CED8 (diskseri.1000CED8), ASCII "WD-WCAD13523066"`

**计算硬盘号：**

```
00560FEF     8B45 DC            mov eax,dword ptr ss:[ebp-24]  
00560FF2     0FB64418 FF        movzx eax,byte ptr ds:[eax+ebx-1]  
00560FF7     B9 0A000000        mov ecx,0A  
00560FFC     33D2               xor edx,edx  
00560FFE     F7F1               div ecx  
00561000     8BC2               mov eax,edx  
00561002     8D95 68FFFFFF      lea edx,dword ptr ss:[ebp-98]  
00561008     E8 A79CEAFF        call gwyzl.0040ACB4  
0056100D     8B95 68FFFFFF      mov edx,dword ptr ss:[ebp-98]  
00561013     8B45 FC            mov eax,dword ptr ss:[ebp-4]  
00561016     05 18030000        add eax,318  
0056101B     E8 DC30EAFF        call gwyzl.004040FC  
00561020     43                 inc ebx  
00561021     4E                 dec esi  
00561022     75 CB              jnz short gwyzl.00560FEF  
```

硬盘号：`01005844  57 44 2D 57 43 41 44 31 33 35 32 33 30 36 36 00  WD-WCAD13523066.`

值：`010026FC  37 38 35 37 37 35 38 39 31 33 30 31 38 34 34 00  785775891301844.`

顺取硬盘号，除以 `A` 得到的余数即是。

**然后连接 `GWY`：**

```
00561053     BA 10165600   mov edx,gwyzl.00561610                   ; ASCII "GWY"  
```

`ds:[00F849F0]=0100DC34, (ASCII "GWY785775891301844")`

得到的这个就是注册时显示的机器码。

### 4. 比较

**对注册码加密值的取值：**

```
00561487     8B80 18030000   mov eax,dword ptr ds:[eax+318]  
0056148D     E8 622CEAFF     call gwyzl.004040F4                    ; 长度  
00561492     8BC8            mov ecx,eax  
00561494     49              dec ecx                                ; 11 位（H）  
00561495     8B45 FC         mov eax,dword ptr ss:[ebp-4]  
00561498     8B80 1C030000   mov eax,dword ptr ds:[eax+31C]         ; 注册码加密值  
0056149E     BA 0A000000     mov edx,0A                             ; 从第 A＋1 位开始取,取 11 位  
005614A3     E8 542EEAFF     call gwyzl.004042FC  
```

注册码加密值：`ds:[00F849F4]=01005828, (ASCII "fjqtRXA9MwWD")`

取值：堆栈 `ss:[0012FAEC]=00FEBBD0, (ASCII "wWD")`

**硬盘号计算值的取值：**

```
005614B9     8B80 18030000   mov eax,dword ptr ds:[eax+318]         ; 硬盘号计算值  
005614BF     E8 302CEAFF     call gwyzl.004040F4                     ; 取长度  
005614C4     8BC8            mov ecx,eax                             ; 长度  
005614C6     49              dec ecx                                 ; -1，11 位（H）  
005614C7     8B45 FC         mov eax,dword ptr ss:[ebp-4]  
005614CA     8B80 18030000   mov eax,dword ptr ds:[eax+318]  
005614D0     BA 01000000     mov edx,1  
005614D5     E8 222EEAFF     call gwyzl.004042FC  
```

去除最后 1 位：堆栈 `ss:[0012FAE8]=0100EF24, (ASCII "GWY78577589130184")`

**然后比较：**

```
005614DA     8B95 14FFFFFF   mov edx,dword ptr ss:[ebp-EC]          ; 参数之二：硬盘号计算值  
005614E0     58              pop eax                                 ; 参数之一：注册码计算值  
005614E1     E8 1E2DEAFF     call gwyzl.00404204  
005614E6     75 15           jnz short gwyzl.005614FD  
```

如果两者相等就是注册成功。

### 5. 算法流程

**注册码计算：**

取得注册码，查表转换（查表值为 `6bit×`），然后 `RSA` 加密，加密值为二进制形式，如果 `m` 太大就分组加密，每组加密值如果不能为 `4F` 整除在前面添 `0`，最后整合起来，去除第一个 `111` 的前面的字符（包括 `111`），剩下的长度要被 `6` 整除，否则在后面添 `0`，然后取 `6bit` 查表并 `Base64` 得到字符串，从第 `10` 位开始取 `17` 位的字符，得到参数之一。

**硬盘号计算：**

顺取硬盘号，除以 `A` 的余数（即十进制的个位数），连接 `GWY` 得到机器码，取前 `17` 位，得到参数之二。

**如果 `M>N`，`RSA` 分批加密：**

比如练码：`87654321123456788765432112345678`

转换后：
```
0012FA68   00FFE384     ASCII "111100111011111010111001111000110111110110110101110101110110110111111000111001111010111011111100111100111011111010111001111000110111110110110101110101110110110111111000111001111010111011111100"  
```

即：`F3BEB9E3 7DB5D76DF8E7AEFCF3BE B9E37DB5D76DF8E7AEFC`

**第一次加密：`F3BEB9E3`**

```
0012FA60   00FFE6E4     ASCII "10000110011000101100101011110000111101001000000010101111111110110000001110011000"  
```

`N=BED3E05921571679EEAD（H）`

`E=58FF54E068BD181EFD67（H）`

`M1=F3BEB9E3`

`C1=M~E MOD(N)=8662CAF0F480AFFB0398`

**第二次加密：`7DB5D76DF8E7AEFCF3BE`**

```
0012FA60   00FFB554     ASCII "11100101010011110000011101101100111110000001110010111101011010100111011010100"  
```

`N=BED3E05921571679EEAD（H）`

`E=58FF54E068BD181EFD67（H）`

`C2=M~E MOD(N)=1CA9E0ED9F0397AD4ED4`

`D=17443D32FC96AE3`

`M2=7DB5D76DF8E7AEFCF3BE`

**第 3 次加密：`B9E37DB5D76DF8E7AEFC`**

```
0012FA60   00FFEE64     ASCII "10111101111111001010110111101100011001100101111100101000101001110111111101011110"  
```

`N=BED3E05921571679EEAD（H）`

`E=58FF54E068BD181EFD67（H）`

`C3=M~E MOD(N)=BDFCADEC665F28A77F5E`

`D=17443D32FC96AE3`

`M3=B9E37DB5D76DF8E7AEFC`

每次加密值的长度补 `0` 到 `4F` 的整数倍，然后连接。

### 6. 反推注册码

硬盘号计算值：`GWY78577589130184`

补上前 9 位：假设为 `a87654321GWY78577589130184`

**逆查 Base64 表：**

```
00563B60                                                               C0  
00563B70  61 41 62 42 63 43 64 44 65 45 66 46 67 47 68 48  aAbBcCdDeEfFgGhH  
00563B80  69 49 6A 4A 6B 4B 6C 4C 6D 4D 6E 4E 6F 4F 70 50  iIjJkKlLmMnNoOpP  
00563B90  71 51 72 52 73 53 74 54 75 55 76 56 77 57 78 58  qQrRsStTuUvVwWxX  
00563BA0  79 59 7A 5A 30 31 32 33 34 35 36 37 38 39 2B 3D  yYzZ0123456789+=  
```

`1  2  3  4  5  6  7  8  9  A  B  C  D  E  F  10`

逆查表值：`01 3D 3C 3B 3A 39 38 37 36 0E 2E 32 3C 3D 3A 3C 3C 3A 3D 3E 36 38 35 36 3D 39`

还原为注册码加密值：`C`

**各减 1 后转换为 2 进制：**

```
00 3C 3B 3A 39 38 37 36 35 0D 2D 31 3B 3C 39 3B 3B 39 3C 3D 35 37 34 35 3C 38  

00000000 00111100 00111011 00111010 00111001 00111000 00110111 00110110 00110101 00001101 00101101 00110001 00111011 00111100 00111001 00111011 00111011 00111001 00111100 00111101 00110101 00110111 00110100 00110101 00111100 00111000  
```

剔除各个的前 2 位 `0`，然后整合：

```
000000 111100 111011 111010 111001 111000 110111 110110 110101 001101 101101 110001 111011 111100 111001 111011 111011 111001 111100 111101 110101 110111 110100 110101 111100 111000  
```

即：`156bit`

```
000000111100111011111010111001111000110111110110110101001101101101110001111011111100111001111011111011111001111100111101110101110111110100110101111100111000  
```

因为 `156bit` 的后面 `0` 不够多，再补 `6` 个 `0`：`162bit`

```
000000111100111011111010111001111000110111110110110101001101101101110001111011111100111001111011111011111001111100111101110101110111110100110101111100111000000000  
```

取前面 `160bit`（被 `8` 整除）：

```
0000001111001110111110101110011110001101111101101101010011011011011100011110111111001110011110111110111110011111001111011101011101111101001101011111001110000000  
```

补上前 `3bit`：`163bit`

```
1110000001111001110111110101110011110001101111101101101010011011011011100011110111111001110011110111110111110011111001111011101011101111101001101011111001110000000  
```

**`RSA` 解密：`M` 的获取**

需要解密的值：`163bit`：

```
1110000001111001110111110101110011110001101111101101101010011011011011100011110111111001110011110111110111110011111001111011101011101111101001101011111001110000000  
```

太长了，分组（从后面开始，`79bit` 一组）：

`11100`

`0000111100111011111010111001111000110111110110110101001101101101110001111011111`

`1001110011110111110111110011111001111011101011101111101001101011111001110000000`

即：

`1C`

`79DF5CF1BEDA9B6E3DF`

`4E7BEF9F3DD77D35F380`

**分别解密：**

`N=BED3E05921571679EEAD（H）`

`E=58FF54E068BD181EFD67（H）`

`D=17443D32FC96AE3`

`C1=1C`

`M1=C~D MOD(N)=A1746E7221ACE7603DFE`

`C2=79DF5CF1BEDA9B6E3DF`

`M2=C~D MOD(N)=3DF275EA7952434E00C0`

`C3=4E7BEF9F3DD77D35F380`

`M3=C~D MOD(N)=6C8233DA6EC579DB56F3`

即 `M` 为：

`A1746E7221ACE7603DFE3DF275EA7952434E00C06C8233DA6EC579DB56F3`

二进制为：`240bit`

```
101000010111010001101110011100100010000110101100111001110110000000111101111111100011110111110010011101011110101001111001010100100100001101001110000000001100000001101100100000100011001111011010011011101100010101111001110110110101011011110011  
```

**获得注册码：**

分割成 `6bit`：



`101000 010111 010001 101110 011100 100010 000110 101100 111001 110110 000000 111101 111111 100011 110111 110010 011101 011110 101001 111001 010100 100100 001101 001110 000000 001100 000001 101100 100000 100011 001111 011010 011011 101100 010101 111001 110110 110101 011011 110011`

逆查表：

`101000 010111 010001 101110 011100 100010 000110 101100`

`u L I x o r d w`

`111001 110110 000000 111101 111111 100011 110111 110010`

`5 2 a 9 = R 3 z`

`011101 011110 101001 111001 010100 100100 001101 001110`

`O p U 5 k s G h`

`000000 001100 000001 101100 100000 100011 001111 011010`

`a g A w q R H n`

`011011 101100 010101 111001 110110 110101 011011 110011`

`N w K 5 2 1 N Z`

机器码：`GWY785775891301844`

注册码：`uLIxordw52a9=R3zOpU5ksGhagAwqRHnNwK521NZ`

追踪注册码与脱壳就是有不同的感受！

`4dd466d0e1e0cc123afbe504da2462ca`
