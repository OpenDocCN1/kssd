
```
004CC6C3  ^ E9 C447FCFF        jmp IEAccele.00490E8C                  ; 直接找到这里，F2下断，F9运行中断后，F8返回程序OEP
004CC6C8  ^ E0 C6              loopdne short IEAccele.004CC690
004CC6CA    4C                 dec esp
004CC6CB    00F0               add al,dh
004CC6CD    C6                 ???                                    ; 未知命令
004CC6CE    4C                 dec esp
004CC6CF    0010               add byte ptr ds:[eax],dl
004CC6D1    37                 aaa
004CC6D2    49                 dec ecx
```

### 飞向光明之巅：

```
00490E8C    55                 push ebp                               ; OEP
00490E8D    8BEC               mov ebp,esp
00490E8F    83C4 F0            add esp,-10
00490E92    B8 D40B4900        mov eax,IEAccele.00490BD4
00490E97    E8 CC5AF7FF        call IEAccele.00406968
00490E9C    A1 D8234900        mov eax,dword ptr ds:[4923D8]
00490EA1    8B00               mov eax,dword ptr ds:[eax]
00490EA3    E8 BC04FDFF        call IEAccele.00461364
00490EA8    A1 D8234900        mov eax,dword ptr ds:[4923D8]
00490EAD    8B00               mov eax,dword ptr ds:[eax]
00490EAF    BA EC0E4900        mov edx,IEAccele.00490EEC              ; ASCII "IE Accelerator 2.22"
00490EB4    E8 B700FDFF        call IEAccele.00460F70
00490EB9    8B0D E4244900      mov ecx,dword ptr ds:[4924E4]          ; IEAccele.00493FC8
00490EBF    A1 D8234900        mov eax,dword ptr ds:[4923D8]
00490EC4    8B00               mov eax,dword ptr ds:[eax]
00490EC6    8B15 40B74800      mov edx,dword ptr ds:[48B740]          ; IEAccele.0048B78C
00490ECC    E8 AB04FDFF        call IEAccele.0046137C
00490ED1    A1 D8234900        mov eax,dword ptr ds:[4923D8]
00490ED6    8B00               mov eax,dword ptr ds:[eax]
00490ED8    E8 1F05FDFF        call IEAccele.004613FC
00490EDD    E8 D634F7FF        call IEAccele.004043B8
```

`LordPE`抓取内存镜像，`ImportREC`修复：`OEP=00490E8C-00400000=00090E8C`，自动获取IAT，`RVA=0009413C`，`SIZE=00000710`，指针全部有效，修复后程序运行正常！目标为：Borland Delphi 6.0 - 7.0编译程序。

附件：`IAT.txt`为`ImportREC`获取的完全输入表

---

## 算法分析部分

再次打开`OllyICE`载入脱壳后的文件，利用字符串插件直捣“黄龙”（过程略....）

```
0048A8D4    55                 push ebp                            ; KuNgBiM算法分析开始
0048A8D5    8BEC               mov ebp,esp
0048A8D7    B9 0A000000        mov ecx,0A
0048A8DC    6A 00              push 0
0048A8DE    6A 00              push 0
0048A8E0    49                 dec ecx
0048A8E1  ^ 75 F9              jnz short Dumped\_.0048A8DC
0048A8E3    51                 push ecx
0048A8E4    53                 push ebx
0048A8E5    56                 push esi
0048A8E6    57                 push edi
0048A8E7    8945 FC            mov dword ptr ss:[ebp-4],eax
0048A8EA    33C0               xor eax,eax
0048A8EC    55                 push ebp
0048A8ED    68 DDAC4800        push Dumped\_.0048ACDD
0048A8F2    64:FF30            push dword ptr fs:[eax]
0048A8F5    64:8920            mov dword ptr fs:[eax],esp
0048A8F8    8D55 E4            lea edx,dword ptr ss:[ebp-1C]
0048A8FB    8B45 FC            mov eax,dword ptr ss:[ebp-4]
0048A8FE    8B80 00030000      mov eax,dword ptr ds:[eax+300]
0048A904    E8 4B6AFBFF        call Dumped\_.00441354
0048A909    8B45 E4            mov eax,dword ptr ss:[ebp-1C]
0048A90C    E8 4F9EF7FF        call Dumped\_.00404760
0048A911    05 57040000        add eax,457
0048A916    8D55 E8            lea edx,dword ptr ss:[ebp-18]
0048A919    E8 26E2F7FF        call Dumped\_.00408B44
0048A91E    8D55 E0            lea edx,dword ptr ss:[ebp-20]
0048A921    8B45 FC            mov eax,dword ptr ss:[ebp-4]
0048A924    8B80 00030000      mov eax,dword ptr ds:[eax+300]
0048A92A    E8 256AFBFF        call Dumped\_.00441354               ; 比较是否为黑名单用户
0048A92F    8B45 E0            mov eax,dword ptr ss:[ebp-20]
0048A932    BA F4AC4800        mov edx,Dumped\_.0048ACF4            ; distinct
0048A937    E8 689FF7FF        call Dumped\_.004048A4
0048A93C    0F84 EA020000      je Dumped\_.0048AC2C
0048A942    8D55 DC            lea edx,dword ptr ss:[ebp-24]
0048A945    8B45 FC            mov eax,dword ptr ss:[ebp-4]
0048A948    8B80 00030000      mov eax,dword ptr ds:[eax+300]
0048A94E    E8 016AFBFF        call Dumped\_.00441354               ; 比较是否为黑名单用户
0048A953    8B45 DC            mov eax,dword ptr ss:[ebp-24]
0048A956    BA 08AD4800        mov edx,Dumped\_.0048AD08            ; team insane
0048A95B    E8 449FF7FF        call Dumped\_.004048A4
0048A960    0F84 C6020000      je Dumped\_.0048AC2C
0048A966    8D55 D8            lea edx,dword ptr ss:[ebp-28]
0048A969    8B45 FC            mov eax,dword ptr ss:[ebp-4]
0048A96C    8B80 00030000      mov eax,dword ptr ds:[eax+300]
0048A972    E8 DD69FBFF        call Dumped\_.00441354               ; 比较是否为黑名单用户
0048A977    8B45 D8            mov eax,dword ptr ss:[ebp-28]
0048A97A    BA 1CAD4800        mov edx,Dumped\_.0048AD1C            ; tnt!2000
0048A97F    E8 209FF7FF        call Dumped\_.004048A4
0048A984    0F84 A2020000      je Dumped\_.0048AC2C
0048A98A    8D55 D4            lea edx,dword ptr ss:[ebp-2C]
0048A98D    8B45 FC            mov eax,dword ptr ss:[ebp-4]
0048A990    8B80 00030000      mov eax,dword ptr ds:[eax+300]
0048A996    E8 B969FBFF        call Dumped\_.00441354               ; 比较是否为黑名单用户
0048A99B    8B45 D4            mov eax,dword ptr ss:[ebp-2C]
0048A99E    BA 30AD4800        mov edx,Dumped\_.0048AD30            ; -=demian/tnt!=-
0048A9A3    E8 FC9EF7FF        call Dumped\_.004048A4
0048A9A8    0F84 7E020000      je Dumped\_.0048AC2C
0048A9AE    8D55 D0            lea edx,dword ptr ss:[ebp-30]
0048A9B1    8B45 FC            mov eax,dword ptr ss:[ebp-4]
0048A9B4    8B80 00030000      mov eax,dword ptr ds:[eax+300]
0048A9BA    E8 9569FBFF        call Dumped\_.00441354               ; 比较是否为黑名单用户
0048A9BF    8B45 D0            mov eax,dword ptr ss:[ebp-30]
0048A9C2    BA 48AD4800        mov edx,Dumped\_.0048AD48            ; -=demian/tnt!=-
0048A9C7    E8 D89EF7FF        call Dumped\_.004048A4
0048A9CC    0F84 5A020000      je Dumped\_.0048AC2C
0048A9D2    8D55 CC            lea edx,dword ptr ss:[ebp-34]
0048A9D5    8B45 FC            mov eax,dword ptr ss:[ebp-4]
0048A9D8    8B80 00030000      mov eax,dword ptr ds:[eax+300]
0048A9DE    E8 7169FBFF        call Dumped\_.00441354               ; 比较是否为黑名单用户
0048A9E3    8B45 CC            mov eax,dword ptr ss:[ebp-34]
0048A9E6    BA 64AD4800        mov edx,Dumped\_.0048AD64            ; distinct
0048A9EB    E8 B49EF7FF        call Dumped\_.004048A4
0048A9F0    0F84 36020000      je Dumped\_.0048AC2C
0048A9F6    8D55 C8            lea edx,dword ptr ss:[ebp-38]
0048A9F9    8B45 FC            mov eax,dword ptr ss:[ebp-4]
0048A9FC    8B80 00030000      mov eax,dword ptr ds:[eax+300]
0048AA02    E8 4D69FBFF        call Dumped\_.00441354               ; 比较是否为黑名单用户
0048AA07    8B45 C8            mov eax,dword ptr ss:[ebp-38]
0048AA0A    BA 78AD4800        mov edx,Dumped\_.0048AD78            ; tmg
0048AA0F    E8 909EF7FF        call Dumped\_.004048A4
0048AA14    0F84 12020000      je Dumped\_.0048AC2C
0048AA1A    68 84AD4800        push Dumped\_.0048AD84               ; ASCII "C"    // 取固定字符串
0048AA1F    8B45 FC            mov eax,dword ptr ss:[ebp-4]
0048AA22    FFB0 2C030000      push dword ptr ds:[eax+32C]         ; ASCII "MW"   // 取固定字符串
0048AA28    68 90AD4800        push Dumped\_.0048AD90               ; ASCII "20"   // 取固定字符串
0048AA2D    FF75 E8            push dword ptr ss:[ebp-18]          ; ASCII "1124" // 取用户名长度+“1111”
0048AA30    68 9CAD4800        push Dumped\_.0048AD9C               ; ASCII "-"    // 用连接符号连接
0048AA35    8D55 C0            lea edx,dword ptr ss:[ebp-40]
0048AA38    8B45 FC            mov eax,dword ptr ss:[ebp-4]
0048AA3B    8B80 00030000      mov eax,dword ptr ds:[eax+300]
0048AA41    E8 0E69FBFF        call Dumped\_.00441354               ; 取用户名
```
