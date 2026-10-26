
# 【原创】一个会计软件的算法分析过程（详细）

标 题: 【原创】一个会计软件的算法分析过程（详细）

发帖人: liyangsj

时 间: 2005-01-10 12:16

详细信息:

一个会计软件的算法分析过程（详细）

【破解作者】 jsliyangsj

【作者邮箱】 sjcrack@yahoo.com.cn

【使用工具】 `peid` `OllyDbg1.10`

【破解平台】 Winxp

【软件名称】 总会计2000

【下载地址】 <http://www.sfzc.com.cn/download/kj/zkj2000.exe>

--------------------------------------------------------------------------------

## 分析

程序没有壳，是Microsoft Visual C++ 6.0编写

用`OD`载入，选择用户名，点击“确定” 让你输入序列号不正确则退出

输入码必须16位，否则报错，我用`OD`插件`ustrref.dll`查找字符串“解密口令长度不对”

```
0040F237    .  83F8 10     cmp eax,10        必须16位
0040F23A    .  74 21       je short ZKJ2000.0040F25D
0040F23C    .  6A 00       push 0
0040F23E    .  6A 00       push 0
0040F240    .  68 8C3E5100 push ZKJ2000.00513E8C
0040F245    .  8BCF        mov ecx,edi
0040F247    .  E8 F8320C00 call <jmp.&MFC42.#4224>   出错！！
```

我的输入码`1212121212121212`    我的机器码：`334Q0K1U4G3K3T52`

```
0040F25D    > \8D4D 94     lea ecx,dword ptr ss:[ebp-6C]
0040F260    .  51          push ecx
0040F261    .  E8 D42D0C00 call <jmp.&ZB\_DLL.Bm3GetYJInfo>
0040F266    .  83C4 04     add esp,4
0040F269    .  8D4D EC     lea ecx,dword ptr ss:[ebp-14]
0040F26C    .  E8 41310C00 call <jmp.&MFC42.#540>
0040F271    .  33F6        xor esi,esi
0040F273    .  68 A03D5100 push ZKJ2000.00513DA0
0040F278    .  8D4D EC     lea ecx,dword ptr ss:[ebp-14]
0040F27B    .  8975 FC     mov dword ptr ss:[ebp-4],esi
0040F27E    .  E8 95310C00 call <jmp.&MFC42.#860>
0040F283    .  8B45 EC     mov eax,dword ptr ss:[ebp-14]
0040F286    .  8B48 F8     mov ecx,dword ptr ds:[eax-8]
0040F289    .  51          push ecx
0040F28A    .  50          push eax
0040F28B    .  8D8D 4CEFFF>lea ecx,dword ptr ss:[ebp-10B4]
0040F291    .  FF15 B89D4E>call dword ptr ds:[<&ZB\_DLL.CBm3Blowfis>;  ZB\_DLL.CBm3Blowfish::CBm3Blowfish
0040F297    .  8B47 64     mov eax,dword ptr ds:[edi+64]
0040F29A    .  8D55 BC     lea edx,dword ptr ss:[ebp-44]
0040F29D    .  6A 0A       push 0A
0040F29F    .  52          push edx
0040F2A0    .  50          push eax
0040F2A1    .  C645 FC 01  mov byte ptr ss:[ebp-4],1
0040F2A5    .  E8 962D0C00 call <jmp.&ZB\_DLL.Bm3WordToByte>        ;  第一次处理输入码
0040F2AA    .  83C4 0C     add esp,0C
0040F2AD    .  8D45 BC     lea eax,dword ptr ss:[ebp-44]
0040F2B0    .  8D8D 4CEFFF>lea ecx,dword ptr ss:[ebp-10B4]
0040F2B6    .  6A 08       push 8
0040F2B8    .  50          push eax
0040F2B9    .  FF15 A09D4E>call dword ptr ds:[<&ZB\_DLL.CBm3Blowfis>;  第2次处理输入码
0040F2BF    .  33C0        xor eax,eax
0040F2C1    >  8A4C05 BC   mov cl,byte ptr ss:[ebp+eaw-44]
0040F2C5    .  8A5405 94   mov dl,byte ptr ss:[ebp+eaw-6C]            此处发现我最终的结果
```

```
0012D408  1CEB3030
0012D40C  00000F4A      由于最后两位必须是FF，修正FFFF0F4A
```

```
0040F2C9    .  3ACA        cmp cl,dl
0040F2CB    .  75 1E       jnz short ZKJ2000.0040F2EB
0040F2CD    .  40          inc eax
0040F2CE    .  83F8 06     cmp eax,6
0040F2D1    .^ 7C EE       jl short ZKJ2000.0040F2C1
0040F2D3    .  8B45 C2     mov eax,dword ptr ss:[ebp-3E]
0040F2D6    .  3C FF       cmp al,0FF
0040F2D8    .  75 24       jnz short ZKJ2000.0040F2FE              ;  计算后的倒数3，4位必须是FF
0040F2DA    .  891D B8FA51>mov dword ptr ds:[51FAB8],ebx
0040F2E0    .  8935 BCFA51>mov dword ptr ds:[51FABC],esi
0040F2E6    .  E9 BF000000 jmp ZKJ2000.0040F3AA
0040F2EB    >  56          push esi
0040F2EC    .  56          push esi
0040F2ED    .  68 643E5100 push ZKJ2000.00513E64
0040F2F2    .  8BCF        mov ecx,edi
0040F2F4    .  E8 4B320C00 call <jmp.&MFC42.#4224>
0040F2F9    .  E9 23030000 jmp ZKJ2000.0040F621
0040F2FE    >  25 FF000000 and eax,0FF
0040F303    .  891D BCFA51>mov dword ptr ds:[51FABC],ebx
0040F309    .  8BC8        mov ecx,eax
0040F30B    .  B8 ABAAAAAA mov eax,AAAAAAAB
0040F310    .  F7E1        mul ecx
0040F312    .  C1EA 03     shr edx,3
0040F315    .  81C2 D00700>add edx,7D0
0040F31B    .  8BC1        mov eax,ecx
0040F31D    .  8915 C0FA51>mov dword ptr ds:[51FAC0],edx
0040F323    .  33D2        xor edx,edx
0040F325    .  B9 0C000000 mov ecx,0C
0040F32A    .  F7F1        div ecx
0040F32C    .  42          inc edx
0040F32D    .  8915 C4FA51>mov dword ptr ds:[51FAC4],edx
0040F333    .  8D55 DC     lea edx,dword ptr ss:[ebp-24]
0040F336    .  52          push edx
0040F337    .  E8 14350C00 call <jmp.&MFC42.#3811>
0040F33C    .  8B00        mov eax,dword ptr ds:[eax]
0040F33E    .  56          push esi
0040F33F    .  8D4D E8     lea ecx,dword ptr ss:[ebp-18]
0040F342    .  8945 E8     mov dword ptr ss:[ebp-18],eax
0040F345    .  E8 00350C00 call <jmp.&MFC42.#3337>
0040F34A    .  8B40 14     mov eax,dword ptr ds:[eax+14]
0040F34D    .  8B0D C0FA51>mov ecx,dword ptr ds:[51FAC0]
0040F353    .  05 6C070000 add eax,76C
0040F358    .  3BC1        cmp eax,ecx
0040F35A    .  7C 45       jl short ZKJ2000.0040F3A1
0040F35C    .  56          push esi
0040F35D    .  8D4D E8     lea ecx,dword ptr ss:[ebp-18]
0040F360    .  E8 E5340C00 call <jmp.&MFC42.#3337>
0040F365    .  8B40 14     mov eax,dword ptr ds:[eax+14]
0040F368    .  8B0D C0FA51>mov ecx,dword ptr ds:[51FAC0]
0040F36E    .  05 6C070000 add eax,76C
0040F373    .  3BC1        cmp eax,ecx
0040F375    .  75 17       jnz short ZKJ2000.0040F38E
0040F377    .  56          push esi
0040F378    .  8D4D E8     lea ecx,dword ptr ss:[ebp-18]
0040F37B    .  E8 CA340C00 call <jmp.&MFC42.#3337>
0040F380    .  8B40 10     mov eax,dword ptr ds:[eax+10]
0040F383    .  8B0D C4FA51>mov ecx,dword ptr ds:[51FAC4]
0040F389    .  40          inc eax
0040F38A    .  3BC1        cmp eax,ecx
0040F38C    .  7E 13       jle short ZKJ2000.0040F3A1
0040F38E    >  56          push esi
0040F38F    .  56          push esi
0040F390    .  68 403E5100 push ZKJ2000.00513E40
0040F395    .  8BCF        mov ecx,edi
0040F397    .  E8 A8310C00 call <jmp.&MFC42.#4224>
0040F39C    .  E9 80020000 jmp ZKJ2000.0040F621
0040F3A1    >  8B45 C2     mov eax,dword ptr ss:[ebp-3E]
0040F3A4    .  891D B8FA51>mov dword ptr ds:[51FAB8],ebx
0040F3AA    >  80FC FF     cmp ah,0FF                              ;  计算后倒数1，2位必须是FF
0040F3AD    .  0F84 4E0100>je ZKJ2000.0040F501
```

进入第一次处理输入码：`0040F2A5`    .  `E8 962D0C00 call` 第一次处理输入码 2次`F7`

```
10001B50 Z>/$  53          push ebx
10001B51   |.  56          push esi
10001B52   |.  8B7424 0C   mov esi,dword ptr ss:[esp+C]
10001B56   |.  57          push edi
10001B57   |.  56          push esi                                ; /String
10001B58   |.  FF15 2CA109>call dword ptr ds:[<&KERNEL32.lstrlenA>>; \lstrlenA
10001B5E   |.  A8 01       test al,1
10001B60   |.  0F85 980000>jnz ZB\_DLL.10001BFE
10001B66   |.  8B4C24 18   mov ecx,dword ptr ss:[esp+18]
10001B6A   |.  D1E8        shr eax,1
10001B6C   |.  3BC1        cmp eax,ecx
10001B6E   |.  0F87 8A0000>ja ZB\_DLL.10001BFE
10001B74   |.  33FF        xor edi,edi
10001B76   |.  85C0        test eax,eax
10001B78   |.  0F86 820000>jbe ZB\_DLL.10001C00
10001B7E   |.  8B5C24 14   mov ebx,dword ptr ss:[esp+14]           ;  准备储存的地址
10001B82   |>  0FBE0E      /movsx ecx,byte ptr ds:[esi]
10001B85   |.  83F9 61     |cmp ecx,61                             ;  判断是否为小写
10001B88   |.  7C 0A       |jl short ZB\_DLL.10001B94
10001B8A   |.  83F9 7A     |cmp ecx,7A
10001B8D   |.  7F 05       |jg short ZB\_DLL.10001B94
10001B8F   |.  83E9 57     |sub ecx,57                             ;  小写就-57
10001B92   |.  EB 1C       |jmp short ZB\_DLL.10001BB0
10001B94   |>  83F9 41     |cmp ecx,41
10001B97   |.  7C 0A       |jl short ZB\_DLL.10001BA3               ;  判断是否为大写
10001B99   |.  83F9 5A     |cmp ecx,5A
10001B9C   |.  7F 05       |jg short ZB\_DLL.10001BA3
10001B9E   |.  83E9 37     |sub ecx,37                             ;  大写就-37得到原字母
10001BA1   |.  EB 0D       |jmp short ZB\_DLL.10001BB0
10001BA3   |>  83F9 30     |cmp ecx,30
```
