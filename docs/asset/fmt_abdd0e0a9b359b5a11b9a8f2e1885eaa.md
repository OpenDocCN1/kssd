
# 汇编代码片段

```
mov bl,1

jmp label 3

xor bl,bl ; label 1

lea ecx,dword ptr ss:[esp+10] ; label 3
```

这样就补齐了第一个跳转跳过的内容。

## 搞定 `je label2` 跳转跳过的内容：

来到：

```
019201BE     84DB             test bl,bl             ; call 00437766 返回处
019201C0   ^ 0F84 4CFFFFFF    je 01920112            ; label2，修改，使不跳转
019201C6   ^ E9 D0FFFFFF      jmp 0192019B
0192019B     6A 00            push 0
0192019D     E9 15000000      jmp 019201B7
019201B7     6A 00            push 0
019201B9   ^ E9 77FEFFFF      jmp 01920035
01920035    /65:EB 01        jmp short 01920039
01920039     52              push edx
0192003A     66:9C           pushfw
0192003C     55              push ebp
0192003D     EB 01           jmp short 01920040
01920040     8D6C51 3B       lea ebp,dword ptr ds:[ecx+edx*2+3B]
01920044     8D6D C5         lea ebp,dword ptr ss:[ebp-3B]
01920047     1BE9            sbb ebp,ecx
01920049     036C24 38       add ebp,dword ptr ss:[esp+38]
0192004D     036C24 18       add ebp,dword ptr ss:[esp+18]
01920051     8D6C34 26       lea ebp,dword ptr ss:[esp+esi+26]
01920055     2BEE            sub ebp,esi
01920057     8D6C35 DA       lea ebp,dword ptr ss:[ebp+esi-26]
0192005B     F2:             prefix repne:
0192005F     2BEE            sub ebp,esi            ; BetterJP.0044EB80
01920061     64:EB 02        jmp short 01920066
01920066     8D6C1D 06       lea ebp,dword ptr ss:[ebp+ebx+6]
0192006A     2BEB            sub ebp,ebx
0192006C     51              push ecx
0192006D     034C24 18       add ecx,dword ptr ss:[esp+18]
01920071     36:EB 01        jmp short 01920075
01920075     C1F9 8E         sar ecx,8E
01920078     8D4C4B D2       lea ecx,dword ptr ds:[ebx+ecx*2-2E]
0192007C     8D8C3A 60DA4400 lea ecx,dword ptr ds:[edx+edi+44DA60]
01920083     2BCF            sub ecx,edi
01920085     2BCA            sub ecx,edx
01920087     51              push ecx
01920088     8F45 00         pop dword ptr ss:[ebp]
0192008B     59              pop ecx
0192008C     5D              pop ebp
0192008D     66:9D           popfw                  ; =push 44DA60
0192008F     E9 FD000000     jmp 01920191
01920191     68 79019201     push 1920179
01920196     E8 65Fe0100     call 01940000          ; 变形 call
019500ED    - FF6424 FC     jmp dword ptr ss:[esp-4] ; BetterJP.00437A00
```

相当于 `call 00437A00`

```
01920179     6A 01            push 1               ; 返回处
0192017B   ^ E9 AAFEFFFF     jmp 0192002A
0192002A     FF15 B0F84300   call dword ptr ds:[43F8B0] ; MSVCRT.exit
```

然后退出。

蓝绿色就是所需的代码，这样得到了第二个跳转跳过的内容：

```
call 00437766

test bl,bl 

je label2

push 0

push 0

push 44DA60

call 00437A00

push 1

call dword ptr ds:[43F8B0]      ; MSVCRT.exit

push 40efb4                   ; label2

retn
```

## 得到 `stolen code` 的整段汇编代码，如下：

```
call 00437C82

lea eax,dword ptr ss:[esp+10] 

lea ecx,dword ptr ds:[esi+0DC]

push eax

call 00434C60

mov eax,dword ptr ds:[eax]        ; mfc42.6BD156C8

push 0044dac8

push eax              ; mfc42.6BD156C8

mov dword ptr ss:[esp+138],0

call dword ptr ds:[43F874]        ; MSVCRT._mbscmp

add esp,8

test eax,eax

jnz label1

lea ecx,dword ptr ds:[esi+E0]

call 00423300

test eax,eax

sete al

je label1

mov bl,1

jmp label 3

xor bl,bl               ; label 1

lea ecx,dword ptr ss:[esp+10]   ; label 3

mov dword ptr ss:[esp+130],-1

call 00437766

test bl,bl 

je label2

push 0

push 0

push 44DA60

call 00437A00

push 1

call dword ptr ds:[43F8B0]        ; MSVCRT.exit

push 40efb4               ; label2

retn
```

因为长度关系，从 `0040EF31` 开始写代码：

```
0040EF31   /E9 01000000           jmp dumped_2.0040EF37       ; 这里开始写
0040EF37  - E9 C4105101           jmp 01920000               ; stolen code 开始处
...
0040EFB4     8B86 C8010000        mov eax,dword ptr ds:[esi+1C8] ; 返回处，到此为止
```

得到代码：

```
0040EF31     E8 4C8D0200          call <jmp.&mfc42.#2621>
0040EF36     8D4424 10            lea eax,dword ptr ss:[esp+10]
0040EF3A     8D8E DC000000        lea ecx,dword ptr ds:[esi+DC]
0040EF40     50                  push eax
0040EF41     E8 1A5D0200          call dumped_4.00434C60
0040EF46     8B00                mov eax,dword ptr ds:[eax]
0040EF48     68 C8DA4400          push dumped_4.0044DAC8           ; ASCII "BetterJPEG Official Tester"
0040EF4D     50                  push eax
0040EF4E     C78424 38010000 00000000  mov dword ptr ss:[esp+138],0
0040EF59     FF15 74F84300        call dword ptr ds:[<&msvcrt._mbscmp>]    ; MSVCRT._mbscmp
0040EF5F     83C4 08              add esp,8
0040EF62     85C0                test eax,eax
0040EF64     75 16               jnz short dumped_4.0040EF7C
0040EF66     8D8E E0000000        lea ecx,dword ptr ds:[esi+E0]
0040EF6C     E8 8F430100          call dumped_4.00423300
0040EF71     85C0                test eax,eax
0040EF73     0F94C0              sete al
0040EF76     74 04               je short dumped_4.0040EF7C
0040EF78     B3 01               mov bl,1
0040EF7A     EB 02               jmp short dumped_4.0040EF7E
0040EF7C     32DB                xor bl,bl
0040EF7E     8D4C24 10            lea ecx,dword ptr ss:[esp+10]
0040EF82     C78424 30010000 FFFFFFFF  mov dword ptr ss:[esp+130],-1
0040EF8D     E8 D4870200          call <jmp.&mfc42.#800>
0040EF92     84DB                test bl,bl
0040EF94     74 16               je short dumped_4.0040EFAC
0040EF96     6A 00               push 0
0040EF98     6A 00               push 0
0040EF9A     68 60DA4400          push dumped_4.0044DA60           ; ASCII " The registration key you are using is not valid.

Please register to obtain a valid registration key."
0040EF9F     E8 5C8A0200          call <jmp.&mfc42.#1200>
0040EFA4     6A 01               push 1
0040EFA6     FF15 B0F84300        call dword ptr ds:[<&msvcrt.exit>]     ; MSVCRT.exit
0040EFAC     68 B4EF4000          push dumped_4.0040EFB4
0040EFB1     90                  nop
0040EFB2     90                  nop
0040EFB3     C3                  retn
```

### 二进制代码：

```
E8 4C 8D 02 00 8D 44 24 10 8D 8E DC 00 00 00 50 E8 1A 5D 02 00 8B 00 68 C8 DA 44 00 50 C7 84 24
38 01 00 00 00 00 00 00 FF 15 74 F8 43 00 83 C4 08 85 C0 75 16 8D 8E E0 00 00 00 E8 8F 43 01 00
85 C0 0F 94 C0 74 04 B3 01 EB 02 32 DB 8D 4C 24 10 C7 84 24 30 01 00 00 FF FF FF FF E8 D4 87 02
00 84 DB 74 16 6A 00 6A 00 68 60 DA 44 00 E8 5C 8A 02 00 6A 01 FF 15 B0 F8 43 00 68 B4 EF 40 00
90 90 C3
```

运行 ok，试了下，点击 `Help` 的 `About` 提示错误退出，点击帮助文件无反应，其他功能 ok，而且不再提示未注册，无试用时间限制。

### 其他观察

用 `ver` 插件得到 `ASProtect 1.35 build 01.26 Release`

从注册模式上来看确实是 `aspr 1.3X` 系列

`PEID` 把 `aspr 1.3X` 系列统统识别为

`ASProtect 2.x SKE -> Alexey Solodovnikov`

帮助文件没反应只要把帮助文件名改成和主文件一样的名字就可以了

比如脱壳后文件 `dumped_4.exe`

帮助名改成 `dumped_4.chm`

> 最初由 堀北真希 发布

帮助文件没反应只要把帮助文件名改成和主文件一样的名字就可以了

比如脱壳后文件 `dumped_4.exe`

帮助名改成 `dumped_4.chm`

谢谢！

还请指点下 `About` 是如何搞定的？

好多 Asprotect 加壳后的 `About` 一样的。

呵呵, 比较一下就知道了

也是 `SDK` 的一种

### 补充：解决 `About` 的问题

`OD` 载入脱壳修复后程序，搜索字符串，来到：

```
0040F68E     B9 86DA4400          mov ecx,gao_.0044DA86       ; ASCII " not valid.

Please register to obtain a valid registration key."
```

往下步进，修改这个 `je` 为 `jmp`：

```
0040F6B0  /0F84 23000000          je gao_.0040F6D9
```

因为相等的话就说明这个 key 无效，所以修改跳转。
