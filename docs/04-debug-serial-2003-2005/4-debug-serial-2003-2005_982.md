
# 一个会计软件的算法分析过程（详细）

> 原文：`www.pediy.com/kssd/pediy07/pediy7-3.htm`

## 破解信息

【破解作者】 `jsliyangsj`  
【作者邮箱】 `sjcrack@yahoo.com.cn`  
【使用工具】 `peid` `OllyDbg1.10`  
【破解平台】 `Winxp`  
【软件名称】 `总会计 2000`  
【下载地址】 `www.sfzc.com.cn/download/kj/zkj2000.exe`

## 分析过程

程序没有壳，是 `Microsoft Visual C++ 6.0` 编写。

用 `OD` 载入，选择用户名，点击“确定”让你输入序列号，不正确则退出。

输入码必须 16 位，否则报错。我用 `OD` 插件 `ustrref.dll` 查找字符串“解密口令长度不对”。

### 关键指令：验证输入码长度

```assembly
0040F237    .   83F8 10     cmp eax,10        ; 必须 16 位
0040F23A    .   74 21       je short ZKJ2000.0040F25D
0040F23C    .   6A 00       push 0
0040F23E    .   6A 00       push 0
0040F240    .   68 8C3E5100 push ZKJ2000.00513E8C
0040F245    .   8BCF        mov ecx,edi
0040F247    .   E8 F8320C00 call <jmp.&MFC42.#4224>   ; 出错！！
```

我的输入码 `1212121212121212`，我的机器码：`334Q0K1U4G3K3T52`。

### 核心处理逻辑

```assembly
0040F25D    >   \8D4D 94    lea ecx,dword ptr ss:[ebp-6C]
0040F260    .   51          push ecx
0040F261    .   E8 D42D0C00 call <jmp.&ZB_DLL.Bm3GetYJInfo>
0040F266    .   83C4 04     add esp,4
0040F269    .   8D4D EC     lea ecx,dword ptr ss:[ebp-14]
0040F26C    .   E8 41310C00 call <jmp.&MFC42.#540>
0040F271    .   33F6        xor esi,esi
0040F273    .   68 A03D5100 push ZKJ2000.00513DA0
0040F278    .   8D4D EC     lea ecx,dword ptr ss:[ebp-14]
0040F27B    .   8975 FC     mov dword ptr ss:[ebp-4],esi
0040F27E    .   E8 95310C00 call <jmp.&MFC42.#860>
0040F283    .   8B45 EC     mov eax,dword ptr ss:[ebp-14]
0040F286    .   8B48 F8     mov ecx,dword ptr ds:[eax-8]
0040F289    .   51          push ecx
0040F28A    .   50          push eax
0040F28B    .   8D8D 4CEFFF>lea ecx,dword ptr ss:[ebp-10B4]
0040F291    .   FF15 B89D4E>call dword ptr ds:[<&ZB_DLL.CBm3Blowfis>;  ZB_DLL.CBm3Blowfish::CBm3Blowfish
0040F297    .   8B47 64     mov eax,dword ptr ds:[edi+64]
0040F29A    .   8D55 BC     lea edx,dword ptr ss:[ebp-44]
0040F29D    .   6A 0A       push 0A
0040F29F    .   52          push edx
0040F2A0    .   50          push eax
0040F2A1    .   C645 FC 01  mov byte ptr ss:[ebp-4],1
0040F2A5    .   E8 962D0C00 call <jmp.&ZB_DLL.Bm3WordToByte>        ;  第一次处理输入码
0040F2AA    .   83C4 0C     add esp,0C
0040F2AD    .   8D45 BC     lea eax,dword ptr ss:[ebp-44]
0040F2B0    .   8D8D 4CEFFF>lea ecx,dword ptr ss:[ebp-10B4]
0040F2B6    .   6A 08       push 8
0040F2B8    .   50          push eax
0040F2B9    .   FF15 A09D4E>call dword ptr ds:[<&ZB_DLL.CBm3Blowfis>;  第 2 次处理输入码
0040F2BF    .   33C0        xor eax,eax
0040F2C1    >   8A4C05 BC   mov cl,byte ptr ss:[ebp+eax-44]
0040F2C5    .   8A5405 94   mov dl,byte ptr ss:[ebp+eax-6C]            ; 此处发现我最终的结果
```

数据对比：

```text
0012D408  1CEB3030
0012D40C  00000F4A      ; 由于最后两位必须是 FF，修正 FFFF0F4A
```

继续执行比较逻辑：

```assembly
0040F2C9    .   3ACA        cmp cl,dl
0040F2CB    .   75 1E       jnz short ZKJ2000.0040F2EB
0040F2CD    .   40          inc eax
0040F2CE    .   83F8 06     cmp eax,6
0040F2D1    .^  7C EE       jl short ZKJ2000.0040F2C1
0040F2D3    .   8B45 C2     mov eax,dword ptr ss:[ebp-3E]
0040F2D6    .   3C FF       cmp al,0FF
0040F2D8    .   75 24       jnz short ZKJ2000.0040F2FE              ; 计算后的倒数 3，4 位必须是 FF
0040F2DA    .   891D B8FA51>mov dword ptr ds:[51FAB8],ebx
0040F2E0    .   8935 BCFA51>mov dword ptr ds:[51FABC],esi
0040F2E6    .   E9 BF000000 jmp ZKJ2000.0040F3AA
0040F2EB    >   56          push esi
0040F2EC    .   56          push esi
0040F2ED    .   68 643E5100 push ZKJ2000.00513E64
0040F2F2    .   8BCF        mov ecx,edi
0040F2F4    .   E8 4B320C00 call <jmp.&MFC42.#4224>
0040F2F9    .   E9 23030000 jmp ZKJ2000.0040F621
0040F2FE    >   25 FF000000 and eax,0FF
0040F303    .   891D BCFA51>mov dword ptr ds:[51FABC],ebx
0040F309    .   8BC8        mov ecx,eax
0040F30B    .   B8 ABAAAA AA mov eax,AAAAAAAB
0040F310    .   F7E1        mul ecx
0040F312    .   C1EA 03     shr edx,3
0040F315    .   81C2 D00700>add edx,7D0
0040F31B    .   8BC1        mov eax,ecx
0040F31D    .   8915 C0FA51>mov dword ptr ds:[51FAC0],edx
0040F323    .   33D2        xor edx,edx
0040F325    .   B9 0C000000 mov ecx,0C
0040F32A    .   F7F1        div ecx
0040F32C    .   42          inc edx
0040F32D    .   8915 C4FA51>mov dword ptr ds:[51FAC4],edx
0040F333    .   8D55 DC     lea edx,dword ptr ss:[ebp-24]
0040F336    .   52          push edx
0040F337    .   E8 14350C00 call <jmp.&MFC42.#3811>
0040F33C    .   8B00        mov eax,dword ptr ds:[eax]
0040F33E    .   56          push esi
0040F33F    .   8D4D E8     lea ecx,dword ptr ss:[ebp-18]
0040F342    .   8945 E8     mov dword ptr ss:[ebp-18],eax
0040F345    .   E8 00350C00 call <jmp.&MFC42.#3337>
0040F34A    .   8B40 14     mov eax,dword ptr ds:[eax+14]
0040F34D    .   8B0D C0FA51>mov ecx,dword ptr ds:[51FAC0]
0040F353    .   05 6C070000 add eax,76C
0040F358    .   3BC1        cmp eax,ecx
0040F35A    .   7C 45       jl short ZKJ2000.0040F3A1
0040F35C    .   56          push esi
0040F35D    .   8D4D E8     lea ecx,dword ptr ss:[ebp-18]
0040F360    .   E8 E5340C00 call <jmp.&MFC42.#3337>
0040F365    .   8B40 14     mov eax,dword ptr ds:[eax+14]
0040F368    .   8B0D C0FA51>mov ecx,dword ptr ds:[51FAC0]
0040F36E    .   05 6C070000 add eax,76C
0040F373    .   3BC1        cmp eax,ecx
0040F375    .   75 17       jnz short ZKJ2000.0040F38E
0040F377    .   56          push esi
0040F378    .   8D4D E8     lea ecx,dword ptr ss:[ebp-18]
0040F37B    .   E8 CA340C00 call <jmp.&MFC42.#3337>
0040F380    .   8B40 10     mov eax,dword ptr ds:[eax+10]
0040F383    .   8B0D C4FA51>mov ecx,dword ptr ds:[51FAC4]
0040F389    .   40          inc eax
0040F38A    .   3BC1        cmp eax,ecx
0040F38C    .   7E 13       jle short ZKJ2000.0040F3A1
0040F38E    >   56          push esi
0040F38F    .   56          push esi
0040F390    .   68 403E5100 push ZKJ2000.00513E40
0040F395    .   8BCF        mov ecx,edi
0040F397    .   E8 A8310C00 call <jmp.&MFC42.#4224>
0040F39C    .   E9 80020000 jmp ZKJ2000.0040F621
0040F3A1    >   8B45 C2     mov eax,dword ptr ss:[ebp-3E]
0040F3A4    .   891D B8FA51>mov dword ptr ds:[51FAB8],ebx
0040F3AA    >   80FC FF     cmp ah,0FF                          ; 计算后倒数 1，2 位必须是 FF
0040F3AD    .   0F84 4E0100>je ZKJ2000.0040F501
```

### 进入第一次处理输入码

调用地址：`0040F2A5`，指令：`. E8 962D0C00 call 第一次处理输入码 2 次 F7`

```assembly
10001B50 Z>/$  53          push ebx
10001B51  |.    56          push esi
10001B52  |.    8B7424 0C   mov esi,dword ptr ss:[esp+C]
10001B56  |.    57          push edi
10001B57  |.    56          push esi                              ; /String
10001B58  |.    FF15 2CA109>call dword ptr ds:[<&KERNEL32.lstrlenA>>; \lstrlenA
10001B5E  |.    A8 01       test al,1
10001B60  |.    0F85 980000>jnz ZB_DLL.10001BFE
10001B66  |.    8B4C24 18   mov ecx,dword ptr ss:[esp+18]
10001B6A  |.    D1E8        shr eax,1
10001B6C  |.    3BC1        cmp eax,ecx
10001B6E  |.    0F87 8A0000>ja ZB_DLL.10001BFE
10001B74  |.    33FF        xor edi,edi
10001B76  |.    85C0        test eax,eax
10001B78  |.    0F86 820000>jbe ZB_DLL.10001C00
10001B7E  |.    8B5C24 14   mov ebx,dword ptr ss:[esp+14]         ;  准备储存的地址
10001B82  |>    0FBE0E      /movsx ecx,byte ptr ds:[esi]
10001B85  |.    83F9 61     |cmp ecx,61                            ;  判断是否为小写
10001B88  |.    7C 0A       |jl short ZB_DLL.10001B94
10001B8A  |.    83F9 7A     |cmp ecx,7A
10001B8D  |.    7F 05       |jg short ZB_DLL.10001B94
10001B8F  |.    83E9 57     |sub ecx,57                            ;  小写就-57
```



10001B92   |.  EB 1C       |jmp short `ZB_DLL.10001BB0`

10001B94   |>  83F9 41     |cmp `ecx`,41

10001B97   |.  7C 0A       |jl short `ZB_DLL.10001BA3`                ; 判断是否为大写

10001B99   |.  83F9 5A     |cmp `ecx`,5A

10001B9C   |.  7F 05       |jg short `ZB_DLL.10001BA3`

10001B9E   |.  83E9 37     |sub `ecx`,37                              ; 大写就-37 得到原字母

10001BA1   |.  EB 0D       |jmp short `ZB_DLL.10001BB0`

10001BA3   |>  83F9 30     |cmp `ecx`,30

10001BA6   |.  7C 56       |jl short `ZB_DLL.10001BFE`                ; 判断是否为数字

10001BA8   |.  83F9 39     |cmp `ecx`,39

10001BAB   |.  7F 51       |jg short `ZB_DLL.10001BFE`

10001BAD   |.  83E9 30     |sub `ecx`,30                              ; 数字就-30

10001BB0   |>  8D14C9      |lea `edx`,`dword ptr ds:[ecx+ecx*8]`      ; 输入码第一位*8+原输入码临时保存

10001BB3   |.  0FBE4E 01   |movsx `ecx`,`byte ptr ds:[esi+1]`         ; 下一位注册码

10001BB7   |.  C1E2 02     |shl `edx`,2                               ; 第一次的结果左移 2 位

10001BBA   |.  83F9 61     |cmp `ecx`,61                              ; 检查是否为小写字母

10001BBD   |.  7C 0A       |jl short `ZB_DLL.10001BC9`

10001BBF   |.  83F9 7A     |cmp `ecx`,7A

10001BC2   |.  7F 05       |jg short `ZB_DLL.10001BC9`

10001BC4   |.  83E9 57     |sub `ecx`,57

10001BC7   |.  EB 1C       |jmp short `ZB_DLL.10001BE5`

10001BC9   |>  83F9 41     |cmp `ecx`,41                              ; 检查是否为大写

10001BCC   |.  7C 0A       |jl short `ZB_DLL.10001BD8`

10001BCE   |.  83F9 5A     |cmp `ecx`,5A

10001BD1   |.  7F 05       |jg short `ZB_DLL.10001BD8`

10001BD3   |.  83E9 37     |sub `ecx`,37                              ; 大写就-37 得到原字母

10001BD6   |.  EB 0D       |jmp short `ZB_DLL.10001BE5`

10001BD8   |>  83F9 30     |cmp `ecx`,30

10001BDB   |.  7C 21       |jl short `ZB_DLL.10001BFE`

10001BDD   |.  83F9 39     |cmp `ecx`,39

10001BE0   |.  7F 1C       |jg short `ZB_DLL.10001BFE`

10001BE2   |.  83E9 30     |sub `ecx`,30                              ; 数字-30

10001BE5   |>  03D1        |add `edx`,`ecx`                           ; 把第一个输入码计算再左移的数值加上第二位

10001BE7   |.  81FA FF0000>|cmp `edx`,0FF                             ; 看看有没有超过 0FF

10001BED   |.  7F 0F       |jg short `ZB_DLL.10001BFE`

10001BEF   |.  88141F      |mov `byte ptr ds:[edi+ebx]`,`dl`         ; 储存到 0012D430 开始的地址

10001BF2   |.  47          |inc `edi`                                ; 准备储存到下一个地址

10001BF3   |.  83C6 02     |add `esi`,2                               ; 两位两位计算储存，

10001BF6   |.  3BF8        |cmp `edi`,`eax`                           ; 比较次数 8 次 16 的一半

10001BF8   |.^ 72 88       \\jb short `ZB_DLL.10001B82`

10001BFA   |.  5F          |pop `edi`

10001BFB   |.  5E          |pop `esi`

10001BFC   |.  5B          |pop `ebx`

10001BFD   |.  C3         |retn

10001BFE   |>  33C0        |xor `eax`,`eax`

10001C00   |>  5F          |pop `edi`

10001C01   |.  5E          |pop `esi`

10001C02   |.  5B          |pop `ebx`

10001C03   \\.  C3         |retn

分析：上面是两个输入码一次处理，第一个输入码（当成十六进制只能 0～7）*9 的值左移 2 位＋第 2 个输入码的值

第 2 个输入码如果是数字，直接相加，，若是大写字母 ACSII-37 再相加即可，（输入字符不可以小写）

………………………………………………………………………………………………………………………………

进入第 2 次处理输入码

0040F2B9   .  FF15 A09D4E>call `dword ptr ds:[<&ZB_DLL.CBm3Blowfis>`>;  第 2 次处理输入码

………………………………………………………………………………………………………………………………

10001260 Z>/$  8B4424 08  mov `eax`,`dword ptr ss:[esp+8]`

10001264   |.  53         |push `ebx`

10001265   |.  8BD9       |mov `ebx`,`ecx`

10001267   |.  8BC8       |mov `ecx`,`eax`

10001269   |.  C1E9 03    |shr `ecx`,3

1000126C   |.  C1E1 03    |shl `ecx`,3

1000126F   |.  3BC8       |cmp `ecx`,`eax`

10001271   |.  74 06      |je short `ZB_DLL.10001279`

10001273   |.  33C0       |xor `eax`,`eax`

10001275   |.  5B         |pop `ebx`

10001276   |.  C2 0800    |retn 8

10001279   |>  85C0       |test `eax`,`eax`

1000127B   |.  76 20      |jbe short `ZB_DLL.1000129D`

1000127D   |.  56         |push `esi`

1000127E   |.  8B7424 0C  |mov `esi`,`dword ptr ss:[esp+C]`          ; 第一次计算码地址 0012D430

10001282   |.  57         |push `edi`

10001283   |.  8D78 07    |lea `edi`,`dword ptr ds:[eax+7]`

10001286   |.  C1EF 03    |shr `edi`,3

10001289   |>  8D56 04    /lea `edx`,`dword ptr ds:[esi+4]`          ; 计算后的第 5 位注册码的地址

1000128C   |.  8BCB       |mov `ecx`,`ebx`                           ; 密码表地址

1000128E   |.  52         |push `edx`                                ; 第 5 位地址

1000128F   |.  56         |push `esi`                                ; 第 1 位地址

10001290   |.  E8 2B040000 |call `ZB_DLL.CBm3Blowfish::Blowfish_dec`;  进入

10001295   |.  83C6 08    |add `esi`,8

10001298   |.  4F         |dec `edi`

10001299   |.^ 75 EE      \\jnz short `ZB_DLL.10001289`

1000129B   |.  5F         |pop `edi`

1000129C   |.  5E         |pop `esi`

1000129D   |>  B8 01000000 mov `eax`,1

100012A2   |.  5B         |pop `ebx`

100012A3   \\.  C2 0800    |retn 8

………………………………………………………………………………………………………………………………

10001290   |.  E8 2B040000 |call `ZB_DLL.CBm3Blowfish::Blowfish_dec>;  进入

………………………………………………………………………………………………………………………………

100016C0 Z>/$  8B51 44    mov `edx`,`dword ptr ds:[ecx+44]`   ; 不变值   D3B2368E

100016C3   |.  53         |push `ebx`                         ; 0012C3C0

100016C4   |.  55         |push `ebp`

100016C5   |.  56         |push `esi`                         ; 0012D430

100016C6   |.  8B7424 10  |mov `esi`,`dword ptr ss:[esp+10]`  ; 0012D430

100016CA   |.  57         |push `edi`                         ; 1，下面没有 PUSH 了就是说堆栈的值一个都不变

100016CB   |.  8B06       |mov `eax`,`dword ptr ds:[esi]`     ; 取计算后的前六位

100016CD   |.  33D0       |xor `edx`,`eax`                    ; 前 6 位与 D3B2368E XOR

100016CF   |.  33C0       |xor `eax`,`eax`                    ; 清零

100016D1   |.  895424 14  |mov `dword ptr ss:[esp+14]`,`edx`  ; XOR 后保存到 0012C394（临时储存后面好多都储存这里）

100016D5   |.  8BFA       |mov `edi`,`edx`                    ; 值又给了 EDI

100016D7   |.  8A4424 16  |mov `al`,`byte ptr ss:[esp+16]`    ; 指针   刚才算出来的第 3，4 位

100016DB   |.  C1EF 18    |shr `edi`,18                       ; 指针  把刚才数值 SHR 18 位就是取第 1，2 两位

100016DE   |.  8B8481 4804>mov `eax`,`dword ptr ds:[ecx+eax*4>`; 固定值可以变化

100016E5   |.  8B5CB9 48  |mov `ebx`,`dword ptr ds:[ecx+edi*4>`; 固定值可以变化

100016E9   |.  03C3        |add `eax`,`ebx`                    ; 两位固定值相加

100016EB   |.  33DB       |xor `ebx`,`ebx`                    ; 其中一个清零

100016ED   |.  8ADE        |mov `bl`,`dh`                      ; 第 6，7 位给了 BH

100016EF   |.  8BBC99 4808>mov `edi`,`dword ptr ds:[ecx+ebx*4>`; 固定值可以变化

100016F6   |.  33C7        |xor `eax`,`edi`

100016F8   |.  8BFA        |mov `edi`,`edx`                    ; 取代 edi

100016FA   |.  81E7 FF0000>and `edi`,0FF                       ; 指针  取最后 2 位 A8

10001700   |.  8B9CB9 480C>mov `ebx`,`dword ptr ds:[ecx+edi*4>`; 固定值可以变化

10001707   |.  8B79 40    |mov `edi`,`dword ptr ds:[ecx+40]`  ; 不变值  F01AE404

1000170A   |.  03C3        |add `eax`,`ebx`                    ; EAX 与固定值相加

1000170C   |.  33DB        |xor `ebx`,`ebx`                    ; 清零固定值

1000170E   |.  33C7        |xor `eax`,`edi`                    ; EAX 值再与另一固定值

10001710   |.  8B7C24 18  |mov `edi`,`dword ptr ss:[esp+18]`  ; 输入码的计算后的后 8 位地址

10001714   |.  3307        |xor `eax`,`dword ptr ds:[edi]`     ; 刚才上面算出来的 EAX，再与后四位 XOR

10001716   |.  894424 14  |mov `dword ptr ss:[esp+14]`,`eax`  ; 把结果保存在 0012C394

1000171A   |.  8BE8        |mov `ebp`,`eax`

1000171C   |.  8A5C24 16  |mov `bl`,`byte ptr ss:[esp+16]`     ; 指针  取刚才保存的 3，4 位

10001720   |.  C1ED 18    |shr `ebp`,18                        ; 指针  把刚才保存的值 SHR18 就是前第 1，2 为

10001723   |.  8B9C99 4804>mov `ebx`,`dword ptr ds:[ecx+ebx*4>`; 固定值可以变化

1000172A   |.  035CA9 48  |add `ebx`,`dword ptr ds:[ecx+ebp*4>`; 固定值可以变化

1000172E   |.  8BEB        |mov `ebp`,`ebx`

10001730   |.  33DB        |xor `ebx`,`ebx`

10001732   |.  8ADC        |mov `bl`,`ah`                       ; 指针 取刚保存的第 5，6 位

10001734   |.  33AC99 4808>xor `ebp`,`dword ptr ds:[ecx+ebx*4>`; 固定值可以变化



```
1000173B   |.  8BD8        mov ebx,eax                     ;  把算好的给了 EBX  
1000173D   |.  81E3 FF000>and ebx,0FF                     ;  指针 取最后两位  

10001743   |.  03AC99 480C>add ebp,dword ptr ds:\[ecx+ebx\*4>;  固定值可以变化  

1000174A   |.  8B59 3C     mov ebx,dword ptr ds:\[ecx+3C\]   ;  不变值  

1000174D   |.  33EB        xor ebp,ebx                     ;  刚加好的数与刚才的固定值 XOR  

1000174F   |.  33DB        xor ebx,ebx  

10001751   |.  33D5        xor edx,ebp                     ;  EDX 与刚才的值 XOR  

10001753   |.  895424 14   mov dword ptr ss:\[esp+14\],edx   ;  临时储存 0012C394  

10001757   |.  8A5C24 16   mov bl,byte ptr ss:\[esp+16\]     ;  取它的 3，4 位作指针  

1000175B   |.  C1EA 18     shr edx,18                    ;  指针  

1000175E   |.  8BAC99 4804>mov ebp,dword ptr ds:\[ecx+ebx\*4>;  固定值可以变化  

10001765   |.  8B5C91 48   mov ebx,dword ptr ds:\[ecx+edx\*4>;  固定值可以变化  

10001769   |.  8B5424 14   mov edx,dword ptr ss:\[esp+14\]   ;  从 0012C943 取刚才的值  

1000176D   |.  03EB        add ebp,ebx                     ;  相加  

1000176F   |.  33DB        xor ebx,ebx                     ;  清零  

10001771   |.  8ADE        mov bl,dh                       ;  第 6，7 位当指针  

10001773   |.  33AC99 4808>xor ebp,dword ptr ds:\[ecx+ebx\*4>;  取固定值与 EBP XOR  

1000177A   |.  8BDA        mov ebx,edx                     ;  从 0012C094 中取的值给了 EBX  

1000177C   |.  81E3 FF000>and ebx,0FF                     ;  取后两位作指针  

10001782   |.  03AC99 480C>add ebp,dword ptr ds:\[ecx+ebx\*4>;  固定值可以变化  

10001789   |.  8B59 38     mov ebx,dword ptr ds:\[ecx+38\]   ;  不变的值  

1000178C   |.  33EB        xor ebp,ebx  

1000178E   |.  33DB        xor ebx,ebx  

10001790   |.  33C5        xor eax,ebp  

10001792   |.  894424 14   mov dword ptr ss:\[esp+14\],eax   ;  储存并准备分成 4 组，分别作指针取值计算改变 EDX  

10001796   |.  8BE8        mov ebp,eax  

10001798   |.  8A5C24 16   mov bl,byte ptr ss:\[esp+16\]  

1000179C   |.  C1ED 18     shr ebp,18  

1000179F   |.  8B9C99 4804>mov ebx,dword ptr ds:\[ecx+ebx\*4>  

100017A6   |.  035CA9 48   add ebx,dword ptr ds:\[ecx+ebp\*4>  

100017AA   |.  8BEB        mov ebp,ebx  

100017AC   |.  33DB        xor ebx,ebx  

100017AE   |.  8ADC        mov bl,ah  

100017B0   |.  33AC99 4808>xor ebp,dword ptr ds:\[ecx+ebx\*4>  

100017B7   |.  8BD8        mov ebx,eax  

100017B9   |.  81E3 FF000>and ebx,0FF  

100017BF   |.  03AC99 480C>add ebp,dword ptr ds:\[ecx+ebx\*4>  

100017C6   |.  8B59 34     mov ebx,dword ptr ds:\[ecx+34\]  

100017C9   |.  33EB        xor ebp,ebx  

100017CB   |.  33D5        xor edx,ebp  

100017CD   |.  33DB        xor ebx,ebx  

100017CF   |.  895424 14   mov dword ptr ss:\[esp+14\],edx   ;  储存并准备分成 4 组，分别作指针取值计算改变 EAX  

100017D3   |.  8A5C24 16   mov bl,byte ptr ss:\[esp+16\]  

100017D7   |.  C1EA 18     shr edx,18  

100017DA   |.  8BAC99 4804>mov ebp,dword ptr ds:\[ecx+ebx\*4>  

100017E1   |.  8B5C91 48   mov ebx,dword ptr ds:\[ecx+edx\*4>  

100017E5   |.  8B5424 14   mov edx,dword ptr ss:\[esp+14\]  

100017E9   |.  03EB        add ebp,ebx  

100017EB   |.  33DB        xor ebx,ebx  

100017ED   |.  8ADE        mov bl,dh  

100017EF   |.  33AC99 4808>xor ebp,dword ptr ds:\[ecx+ebx\*4>  

100017F6   |.  8BDA        mov ebx,edx  

100017F8   |.  81E3 FF000>and ebx,0FF  

100017FE   |.  03AC99 480C>add ebp,dword ptr ds:\[ecx+ebx\*4>  

10001805   |.  8B59 30     mov ebx,dword ptr ds:\[ecx+30\]  

10001808   |.  33EB        xor ebp,ebx  

1000180A   |.  33DB        xor ebx,ebx  

1000180C   |.  33C5        xor eax,ebp  

1000180E   |.  894424 14   mov dword ptr ss:\[esp+14\],eax   ;  储存并准备分成 4 组，分别作指针取值计算改变  

10001812   |.  8BE8        mov ebp,eax  

10001814   |.  8A5C24 16   mov bl,byte ptr ss:\[esp+16\]  

10001818   |.  C1ED 18     shr ebp,18  

1000181B   |.  8B9C99 4804>mov ebx,dword ptr ds:\[ecx+ebx\*4>  

10001822   |.  035CA9 48   add ebx,dword ptr ds:\[ecx+ebp\*4>  

10001826   |.  8BEB        mov ebp,ebx  

10001828   |.  33DB        xor ebx,ebx  

1000182A   |.  8ADC        mov bl,ah  

1000182C   |.  33AC99 4808>xor ebp,dword ptr ds:\[ecx+ebx\*4>  

10001833   |.  8BD8        mov ebx,eax  

10001835   |.  81E3 FF000>and ebx,0FF  

1000183B   |.  03AC99 480C>add ebp,dword ptr ds:\[ecx+ebx\*4>  

10001842   |.  8B59 2C     mov ebx,dword ptr ds:\[ecx+2C\]  

10001845   |.  33EB        xor ebp,ebx  

10001847   |.  33DB        xor ebx,ebx  

10001849   |.  33D5        xor edx,ebp  

1000184B   |.  895424 14   mov dword ptr ss:\[esp+14\],edx   ;  储存并准备分成 4 组，分别作指针取值计算改变  

1000184F   |.  8A5C24 16   mov bl,byte ptr ss:\[esp+16\]  

10001853   |.  C1EA 18     shr edx,18  

10001856   |.  8BAC99 4804>mov ebp,dword ptr ds:\[ecx+ebx\*4>  

1000185D   |.  8B5C91 48   mov ebx,dword ptr ds:\[ecx+edx\*4>  

10001861   |.  8B5424 14   mov edx,dword ptr ss:\[esp+14\]  

10001865   |.  03EB        add ebp,ebx  

10001867   |.  33DB        xor ebx,ebx  

10001869   |.  8ADE        mov bl,dh  

1000186B   |.  33AC99 4808>xor ebp,dword ptr ds:\[ecx+ebx\*4>  

10001872   |.  8BDA        mov ebx,edx  

10001874   |.  81E3 FF000>and ebx,0FF  

1000187A   |.  03AC99 480C>add ebp,dword ptr ds:\[ecx+ebx\*4>  

10001881   |.  8B59 28     mov ebx,dword ptr ds:\[ecx+28\]  

10001884   |.  33EB        xor ebp,ebx  

10001886   |.  33DB        xor ebx,ebx  

10001888   |.  33C5        xor eax,ebp  

1000188A   |.  894424 14   mov dword ptr ss:\[esp+14\],eax   ;  储存并准备分成 4 组，分别作指针取值计算改变  

1000188E   |.  8BE8        mov ebp,eax  

10001890   |.  8A5C24 16   mov bl,byte ptr ss:\[esp+16\]  

10001894   |.  C1ED 18     shr ebp,18  

10001897   |.  8B9C99 4804>mov ebx,dword ptr ds:\[ecx+ebx\*4>  

1000189E   |.  035CA9 48   add ebx,dword ptr ds:\[ecx+ebp\*4>  

100018A2   |.  8BEB        mov ebp,ebx  

100018A4   |.  33DB        xor ebx,ebx  

100018A6   |.  8ADC        mov bl,ah  

100018A8   |.  33AC99 4808>xor ebp,dword ptr ds:\[ecx+ebx\*4>  

100018AF   |.  8BD8        mov ebx,eax  

100018B1   |.  81E3 FF000>and ebx,0FF  

100018B7   |.  03AC99 480C>add ebp,dword ptr ds:\[ecx+ebx\*4>  

100018BE   |.  8B59 24     mov ebx,dword ptr ds:\[ecx+24\]  

100018C1   |.  33EB        xor ebp,ebx  

100018C3   |.  33DB        xor ebx,ebx  

100018C5   |.  33D5        xor edx,ebp  

100018C7   |.  895424 14   mov dword ptr ss:\[esp+14\],edx   ;  储存并准备分成 4 组，分别作指针取值计算改变  

100018CB   |.  8A5C24 16   mov bl,byte ptr ss:\[esp+16\]  

100018CF   |.  C1EA 18     shr edx,18  

100018D2   |.  8BAC99 4804>mov ebp,dword ptr ds:\[ecx+ebx\*4>  

100018D9   |.  8B5C91 48   mov ebx,dword ptr ds:\[ecx+edx\*4>  

100018DD   |.  8B5424 14   mov edx,dword ptr ss:\[esp+14\]  

100018E1   |.  03EB        add ebp,ebx  

100018E3   |.  33DB        xor ebx,ebx  

100018E5   |.  8ADE        mov bl,dh  

100018E7   |.  33AC99 4808>xor ebp,dword ptr ds:\[ecx+ebx\*4>  

100018EE   |.  8BDA        mov ebx,edx  

100018F0   |.  81E3 FF000>and ebx,0FF  

100018F6   |.  03AC99 480C>add ebp,dword ptr ds:\[ecx+ebx\*4>  

100018FD   |.  8B59 20     mov ebx,dword ptr ds:\[ecx+20\]  

10001900   |.  33EB        xor ebp,ebx  

10001902   |.  33DB        xor ebx,ebx  

10001904   |.  33C5        xor eax,ebp  

10001906   |.  894424 14   mov dword ptr ss:\[esp+14\],eax   ;  储存并准备分成 4 组，分别作指针取值计算改变  

1000190A   |.  8BE8        mov ebp,eax  

1000190C   |.  8A5C24 16   mov bl,byte ptr ss:\[esp+16\]  

10001910   |.  C1ED 18     shr ebp,18  

10001913   |.  8B9C99 4804>mov ebx,dword ptr ds:\[ecx+ebx\*4>  

1000191A   |.  035CA9 48   add ebx,dword ptr ds:\[ecx+ebp\*4>  

1000191E   |.  8BEB        mov ebp,ebx  

10001920   |.  33DB        xor ebx,ebx  

10001922   |.  8ADC        mov bl,ah  

10001924   |.  33AC99 4808>xor ebp,dword ptr ds:\[ecx+ebx\*4>  

1000192B   |.  8BD8        mov ebx,eax  

1000192D   |.  81E3 FF000>and ebx,0FF  

10001933   |.  03AC99 480C>add ebp,dword ptr ds:\[ecx+ebx\*4>  

1000193A   |.  8B59 1C     mov ebx,dword ptr ds:\[ecx+1C\]  

1000193D   |.  33EB        xor ebp,ebx  

1000193F   |.  33DB        xor ebx,ebx  

10001941   |.  33D5        xor edx,ebp
```



## 汇编代码段：指针取值与计算

```asm
10001943   |.  895424 14    mov dword ptr ss:[esp+14],edx   ;  储存并准备分成 4 组，分别作指针取值计算改变  
10001947   |.  8A5C24 16    mov bl,byte ptr ss:[esp+16]  
1000194B   |.  C1EA 18      shr edx,18  
1000194E   |.  8BAC99 4804>mov ebp,dword ptr ds:[ecx+ebx*4>  
10001955   |.  8B5C91 48    mov ebx,dword ptr ds:[ecx+edx*4>  
10001959   |.  8B5424 14    mov edx,dword ptr ss:[esp+14]  
1000195D   |.  03EB         add ebp,ebx  
1000195F   |.  33DB         xor ebx,ebx  
10001961   |.  8ADE         mov bl,dh  
10001963   |.  33AC99 4808>xor ebp,dword ptr ds:[ecx+ebx*4>  
1000196A   |.  8BDA         mov ebx,edx  
1000196C   |.  81E3 FF0000>and ebx,0FF  
10001972   |.  03AC99 480C>add ebp,dword ptr ds:[ecx+ebx*4>  
10001979   |.  8B59 18      mov ebx,dword ptr ds:[ecx+18]  
1000197C   |.  33EB         xor ebp,ebx  
1000197E   |.  33DB         xor ebx,ebx  
10001980   |.  33C5         xor eax,ebp  
10001982   |.  894424 14    mov dword ptr ss:[esp+14],eax   ;  储存并准备分成 4 组，分别作指针取值计算改变  
10001986   |.  8BE8         mov ebp,eax  
10001988   |.  8A5C24 16    mov bl,byte ptr ss:[esp+16]  
1000198C   |.  C1ED 18      shr ebp,18  
1000198F   |.  8B9C99 4804>mov ebx,dword ptr ds:[ecx+ebx*4>  
10001996   |.  035CA9 48    add ebx,dword ptr ds:[ecx+ebp*4>  
1000199A   |.  8BEB         mov ebp,ebx  
1000199C   |.  33DB         xor ebx,ebx  
1000199E   |.  8ADC         mov bl,ah  
100019A0   |.  33AC99 4808>xor ebp,dword ptr ds:[ecx+ebx*4>  
100019A7   |.  8BD8         mov ebx,eax  
100019A9   |.  81E3 FF0000>and ebx,0FF  
100019AF   |.  03AC99 480C>add ebp,dword ptr ds:[ecx+ebx*4>  
100019B6   |.  8B59 14      mov ebx,dword ptr ds:[ecx+14]  
100019B9   |.  33EB         xor ebp,ebx  
100019BB   |.  33DB         xor ebx,ebx  
100019BD   |.  33D5         xor edx,ebp  
100019BF   |.  895424 14    mov dword ptr ss:[esp+14],edx   ;  储存并准备分成 4 组，分别作指针取值计算改变  
100019C3   |.  8A5C24 16    mov bl,byte ptr ss:[esp+16]  
100019C7   |.  C1EA 18      shr edx,18  
100019CA   |.  8BAC99 4804>mov ebp,dword ptr ds:[ecx+ebx*4>  
100019D1   |.  8B5C91 48    mov ebx,dword ptr ds:[ecx+edx*4>  
100019D5   |.  8B5424 14    mov edx,dword ptr ss:[esp+14]  
100019D9   |.  03EB         add ebp,ebx  
100019DB   |.  33DB         xor ebx,ebx  
100019DD   |.  8ADE         mov bl,dh  
100019DF   |.  33AC99 4808>xor ebp,dword ptr ds:[ecx+ebx*4>  
100019E6   |.  8BDA         mov ebx,edx  
100019E8   |.  81E3 FF0000>and ebx,0FF  
100019EE   |.  03AC99 480C>add ebp,dword ptr ds:[ecx+ebx*4>  
100019F5   |.  8B59 10      mov ebx,dword ptr ds:[ecx+10]  
100019F8   |.  33EB         xor ebp,ebx  
100019FA   |.  33C5         xor eax,ebp  
100019FC   |.  33DB         xor ebx,ebx  
100019FE   |.  894424 14    mov dword ptr ss:[esp+14],eax   ;  储存并准备分成 4 组，分别作指针取值计算改变  
10001A02   |.  8BE8         mov ebp,eax  
10001A04   |.  8A5C24 16    mov bl,byte ptr ss:[esp+16]  
10001A08   |.  8B9C99 4804>mov ebx,dword ptr ds:[ecx+ebx*4>  
10001A0F   |.  C1ED 18      shr ebp,18  
10001A12   |.  035CA9 48    add ebx,dword ptr ds:[ecx+ebp*4>  
10001A16   |.  8BEB         mov ebp,ebx  
10001A18   |.  33DB         xor ebx,ebx  
10001A1A   |.  8ADC         mov bl,ah  
10001A1C   |.  33AC99 4808>xor ebp,dword ptr ds:[ecx+ebx*4>  
10001A23   |.  8BD8         mov ebx,eax  
10001A25   |.  81E3 FF0000>and ebx,0FF  
10001A2B   |.  03AC99 480C>add ebp,dword ptr ds:[ecx+ebx*4>  
10001A32   |.  8B59 0C      mov ebx,dword ptr ds:[ecx+C]  
10001A35   |.  33EB         xor ebp,ebx  
10001A37   |.  33DB         xor ebx,ebx  
10001A39   |.  33D5         xor edx,ebp  
10001A3B   |.  895424 14    mov dword ptr ss:[esp+14],edx   ;  储存并准备分成 4 组，分别作指针取值计算改变  
10001A3F   |.  8A5C24 16    mov bl,byte ptr ss:[esp+16]  
10001A43   |.  C1EA 18      shr edx,18  
10001A46   |.  8BAC99 4804>mov ebp,dword ptr ds:[ecx+ebx*4>  
10001A4D   |.  8B5C91 48    mov ebx,dword ptr ds:[ecx+edx*4>  
10001A51   |.  8B5424 14    mov edx,dword ptr ss:[esp+14]  
10001A55   |.  03EB         add ebp,ebx  
10001A57   |.  33DB         xor ebx,ebx  
10001A59   |.  8ADE         mov bl,dh  
10001A5B   |.  33AC99 4808>xor ebp,dword ptr ds:[ecx+ebx*4>  
10001A62   |.  8BDA         mov ebx,edx  
10001A64   |.  81E3 FF0000>and ebx,0FF  
10001A6A   |.  03AC99 480C>add ebp,dword ptr ds:[ecx+ebx*4>  
10001A71   |.  8B59 08      mov ebx,dword ptr ds:[ecx+8]  
10001A74   |.  33EB         xor ebp,ebx  
10001A76   |.  33DB         xor ebx,ebx  
10001A78   |.  33C5         xor eax,ebp  
10001A7A   |.  894424 14    mov dword ptr ss:[esp+14],eax   ;  断点 1 储存并准备分成 4 组，分别作指针取值计算改变此处的 EAX 值与 10001AB5 同  
10001A7E   |.  8BE8         mov ebp,eax  
10001A80   |.  8A5C24 16    mov bl,byte ptr ss:[esp+16]  
10001A84   |.  C1ED 18      shr ebp,18  
10001A87   |.  8B9C99 4804>mov ebx,dword ptr ds:[ecx+ebx*4>  
10001A8E   |.  035CA9 48    add ebx,dword ptr ds:[ecx+ebp*4>  
10001A92   |.  8BEB         mov ebp,ebx  
10001A94   |.  33DB         xor ebx,ebx  
10001A96   |.  8ADC         mov bl,ah  
10001A98   |.  33AC99 4808>xor ebp,dword ptr ds:[ecx+ebx*4>  
10001A9F   |.  8BD8         mov ebx,eax  
10001AA1   |.  81E3 FF0000>and ebx,0FF  
10001AA7   |.  03AC99 480C>add ebp,dword ptr ds:[ecx+ebx*4>  
10001AAE   |.  8B59 04      mov ebx,dword ptr ds:[ecx+4]  
10001AB1   |.  8B09         mov ecx,dword ptr ds:[ecx]  
10001AB3   |.  33EB         xor ebp,ebx  
10001AB5   |.  33C8         xor ecx,eax                 ;  现在开始倒推：ECX 是固定 EB4A4B27 我的结果码 1CEB3030 所以我的 EAX 必须是 F7A17B17  
10001AB7   |.  33D5         xor edx,ebp                 ;  断点 2  推出 EDX 的值等同与上面 10001A3B  EDX 的值  
10001AB9   |.  890E         mov dword ptr ds:[esi],ecx      ;  终于写入了 0012D430  
10001ABB   |.  8917         mov dword ptr ds:[edi],edx  
10001ABD   |.  5F           pop edi  
10001ABE   |.  5E           pop esi  
10001ABF   |.  5D           pop ebp  
10001AC0   |.  5B           pop ebx  
10001AC1   \.  C2 0800      retn 8  
10001AC4       90          nop  
```

## 固定指针取值表

这里发现一张表是固定的，用于指针取之，计算：

```
0012C3C0  EB4A4B27
0012C3C4  80A9ECF0
0012C3C8  5336A06B
0012C3CC  E6332B22
0012C3D0  CB6960CA
0012C3D4  9ACE42FF
0012C3D8  B9088D52
0012C3DC  9B060A56
0012C3E0  1C38C9E5
0012C3E4  9C698BD0
0012C3E8  46184F38
0012C3EC  AA4B0D2E
0012C3F0  2170D567
0012C3F4  9B2CE490
0012C3F8  F86572A2
0012C3FC  71F00093
0012C400  F01AE404
0012C404  D3B2368E
0012C408  A34609DE
0012C40C  35B8F70C
0012C410  90B734C1
0012C414  BFA4758B
0012C418  DBAAB349
0012C41C  FA12B5D1
0012C420  1FD69CA1
0012C424  233CAC84
0012C428  6F3B7B61
0012C42C  133A7550
0012C430  0203C026
0012C434  E8CA811F
0012C438  D1314F7B
0012C43C  FDEF2BA1
0012C440  37B7E9F5
0012C444  20558ACB
0012C448  1B41EA90
0012C44C  F6148C49
0012C450  C809C03C
0012C454  B46B3421
0012C458  DE0FF17D
0012C45C  81ED4BC3
0012C460  CE28F22E
0012C464  B65890BF
0012C468  96C39A3C
0012C46C  E125F8C9
0012C470  8C34C5D9
0012C474  8AB9A855
0012C478  89ECFD8A
0012C47C  51913F5F
0012C480  4BF8C08A
0012C484  A29B8CFB
0012C488  CF1390D9
0012C48C  16B3B339
0012C490  834FAFEA
0012C494  0F92840A
0012C498  1F1E653D
0012C49C  1EC2826E
0012C4A0  0256D647
0012C4A4  D7B81E1F
0012C4A8  A91B0C57
```

下面还有还多的！！

## 逆向推导过程

我的计算结果最后和 `1CEB3030` `FFFF0F4A` 比较先推出 `EAX`，再根据 `EAX` 推出 `EDX`，然后把 `EDX` 带入上面 `EDX`，

这样一点一点向上推，得到：我的经过第一次计算的前 8 位应该是 `346C23AD`，后 8 位应该是 `AE6576A4`

然后再把他们用第一次计算过程反计算！

最后我的输入码是 `1G300Z4T4U2T3A4K`

## 比较码生成位置

产生比较码：`1CEB3030` `FFFF0F4A` 的地方：

```
0040F261    .  E8 D42D0C00 call <jmp.&ZB_DLL.Bm3GetYJInfo>
```

## 进入代码段

```
10001C10 Z>/$  81EC A40500>sub esp,5A4
10001C16   |.  57          push edi
10001C17   |.  B9 10000000 mov ecx,10
10001C1C   |.  33C0        xor eax,eax
```



```
10001C1E   |.  8D7C24 0C   lea edi,dword ptr ss:[esp+C]  

10001C22   |.  F3:AB       rep stos dword ptr es:[edi]  

10001C24   |.  8D4C24 0C   lea ecx,dword ptr ss:[esp+C]  

10001C28   |.  8D4424 50   lea eax,dword ptr ss:[esp+50]  

10001C2C   |.  51          push ecx  

10001C2D   |.  C64424 54 0>mov byte ptr ss:[esp+54],0  

10001C32   |.  C64424 10 3>mov byte ptr ss:[esp+10],37  

10001C37   |.  894424 14   mov dword ptr ss:[esp+14],eax  

10001C3B   |.  66:C74424 1>mov word ptr ss:[esp+18],100  

10001C42   |.  E8 719B0800 call <jmp.&NETAPI32.Netbios>  

10001C47   |.  8A4424 50   mov al,byte ptr ss:[esp+50]  

10001C4B   |.  84C0        test al,al  

10001C4D   |.  0F86 CA0000>jbe ZB_DLL.10001D1D  

10001C53   |.  8A5424 51   mov dl,byte ptr ss:[esp+51]  

10001C57   |.  B9 10000000 mov ecx,10  

10001C5C   |.  33C0        xor eax,eax  

10001C5E   |.  8D7C24 0C   lea edi,dword ptr ss:[esp+C]  

10001C62   |.  F3:AB       rep stos dword ptr es:[edi]  

10001C64   |.  8D4424 0C   lea eax,dword ptr ss:[esp+C]  

10001C68   |.  56          push esi  

10001C69   |.  50          push eax  

10001C6A   |.  C64424 14 3>mov byte ptr ss:[esp+14],32  

10001C6F   |.  885424 44   mov byte ptr ss:[esp+44],dl  

10001C73   |.  E8 409B0800 call <jmp.&NETAPI32.Netbios>  

10001C78   |.  B9 10000000 mov ecx,10  

10001C7D   |.  33C0        xor eax,eax  

10001C7F   |.  8D7C24 10   lea edi,dword ptr ss:[esp+10]  

10001C83   |.  8D5424 1A   lea edx,dword ptr ss:[esp+1A]  

10001C87   |.  F3:AB       rep stos dword ptr es:[edi]  

10001C89   |.  BF 28700C10 mov edi,ZB_DLL.100C7028              ;  ASCII "\*              "  

10001C8E   |.  83C9 FF     or ecx,FFFFFFFF  

10001C91   |.  F2:AE       repne scas byte ptr es:[edi]  

10001C93   |.  F7D1        not ecx  

10001C95   |.  2BF9        sub edi,ecx  

10001C97   |.  884424 40   mov byte ptr ss:[esp+40],al  

10001C9B   |.  8BC1        mov eax,ecx  

10001C9D   |.  8BF7        mov esi,edi  

10001C9F   |.  8BFA        mov edi,edx  

10001CA1   |.  C64424 10 3>mov byte ptr ss:[esp+10],33  

10001CA6   |.  C1E9 02     shr ecx,2  

10001CA9   |.  F3:A5       rep movs dword ptr es:[edi],dword ptr ds>  

10001CAB   |.  8BC8        mov ecx,eax  

10001CAD   |.  8D5424 10   lea edx,dword ptr ss:[esp+10]  

10001CB1   |.  83E1 03     and ecx,3  

10001CB4   |.  52          push edx  

10001CB5   |.  F3:A4       rep movs byte ptr es:[edi],byte ptr ds:[>  

10001CB7   |.  8D8C24 5801>lea ecx,dword ptr ss:[esp+158]  

10001CBE   |.  66:C74424 1>mov word ptr ss:[esp+1C],258  

10001CC5   |.  894C24 18   mov dword ptr ss:[esp+18],ecx  

10001CC9   |.  E8 EA9A0800 call <jmp.&NETAPI32.Netbios>  

10001CCE   |.  84C0        test al,al  

10001CD0   |.  5E          pop esi  

10001CD1   |.  75 4A       jnz short ZB_DLL.10001D1D  

10001CD3   |.  8B8424 AC05>mov eax,dword ptr ss:[esp+5AC]  

10001CDA   |.  8A8C24 5001>mov cl,byte ptr ss:[esp+150]  

10001CE1   |.  8A9424 5101>mov dl,byte ptr ss:[esp+151]  

10001CE8   |.  5F          pop edi  

10001CE9   |.  8808        mov byte ptr ds:[eax],cl  

10001CEB   |.  8A8C24 4E01>mov cl,byte ptr ss:[esp+14E]  

10001CF2   |.  8850 01     mov byte ptr ds:[eax+1],dl  

10001CF5   |.  8A9424 4F01>mov dl,byte ptr ss:[esp+14F]  

10001CFC   |.  8848 02     mov byte ptr ds:[eax+2],cl  

10001CFF   |.  8A8C24 5001>mov cl,byte ptr ss:[esp+150]  

10001D06   |.  8850 03     mov byte ptr ds:[eax+3],dl  

10001D09   |.  8A9424 5101>mov dl,byte ptr ss:[esp+151]  

10001D10   |.  8848 04     mov byte ptr ds:[eax+4],cl  

10001D13   |.  8850 05     mov byte ptr ds:[eax+5],dl  

10001D16   |.  81C4 A40500>add esp,5A4  

10001D1C   |.  C3          retn                         

10001D1D   |>  8D8424 A803>lea eax,dword ptr ss:[esp+3A8]          没上网到这里  

10001D24   |.  68 00010000 push 100                        ; /pFileSystemNameSize = 00000100  

10001D29   |.  8D4C24 0C   lea ecx,dword ptr ss:[esp+C]          ; |  

10001D2D   |.  50          push eax                          ; |pFileSystemNameBuffer  

10001D2E   |.  8D5424 54   lea edx,dword ptr ss:[esp+54]        ; |  

10001D32   |.  51          push ecx                          ; |pFileSystemFlags  

10001D33   |.  8D4424 10   lea eax,dword ptr ss:[esp+10]        ; |  

10001D37   |.  52          push edx                          ; |pMaxFilenameLength  

10001D38   |.  50          push eax                          ; |pVolumeSerialNumber  

10001D39   |.  8D8C24 BC04>lea ecx,dword ptr ss:[esp+4BC]      ; |  

10001D40   |.  68 00010000 push 100                        ; |MaxVolumeNameSize = 100 (256.)  

10001D45   |.  51          push ecx                          ; |VolumeNameBuffer  

10001D46   |.  68 24700C10 push ZB_DLL.100C7024             ; |RootPathName = "C:\\"  

10001D4B   |.  C74424 28 0>mov dword ptr ss:[esp+28],0           ; |  

10001D53   |.  FF15 24A109>call dword ptr ds:[<&KERNEL32.GetVolumeI>; \GetVolumeInformationA  

10001D59   |.  8B8424 AC05>mov eax,dword ptr ss:[esp+5AC]  

10001D60   |.  8B4C24 04   mov ecx,dword ptr ss:[esp+4]  

10001D64   |.  8A5424 06   mov dl,byte ptr ss:[esp+6]  

10001D68   |.  5F          pop edi  

10001D69   |.  8848 02     mov byte ptr ds:[eax+2],cl  

10001D6C   |.  8A4C24 03   mov cl,byte ptr ss:[esp+3]  

10001D70   |.  C600 30     mov byte ptr ds:[eax],30  

10001D73   |.  C640 01 30  mov byte ptr ds:[eax+1],30  

10001D77   |.  8868 03     mov byte ptr ds:[eax+3],ch  

10001D7A   |.  8850 04     mov byte ptr ds:[eax+4],dl  

10001D7D   |.  8848 05     mov byte ptr ds:[eax+5],cl  

10001D80   |.  81C4 A40500>add esp,5A4  

10001D86   \.  C3          retn  
```

分上网和没上网之分！！！！(估计它分为网络版本和,单机版本)

整理：
- 机器码：`334Q0K1U4G3K3T52`
- 序列号：`1G300Z4T4U2T3A4K`

还有我发现，此软件，重新验证机制不太行，它写一文件在 `C:\WINDOWS\SYSTEM32\regyskjinfo16.sys`。

所以你也可以这样破解:你可以自己创建 `C:\WINDOWS\SYSTEM32\regyskjinfo16.sys`，并写上 32（是 32 位）的数字,如 `12121212121212121212121212121212` 即可通过！！

不行你换个数字好了！！

爆破点：

```
0040F2CB    . /75 1E       jnz short ZKJ2000.0040F2EB             ;  爆破点  
0040F2CD    . |40         inc eax  
0040F2CE    . |83F8 06    cmp eax,6  
0040F2D1    . ^|7C EE     jl short ZKJ2000.0040F2C1  
0040F2D3    . |8B45 C2    mov eax,dword ptr ss:[ebp-3E]  
0040F2D6    . |3C FF      cmp al,0FF  
0040F2D8    . |75 24      jnz short ZKJ2000.0040F2FE             ;  爆破点  
0040F3AD    . /0F84 4E0100>je ZKJ2000.0040F501                   ;  爆破点  
```

注意：爆破后要输入两次序列号，就是在 `regyskjinfo16.sys` 写两次，就是 32 位的，（不知道有没有限制）

`4dd466d0e1e0cc123afbe504da2462ca`
