
# 商品销售管理软件 v5.2.8

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-67.htm`](https://www.pediy.com/kssd/pediy08/pediy8-67.htm)

下载页面：`http://www.skycn.com/soft/24335.html`

【作者声明】：初学 Crack，偶只是一只小菜鸟，失误之处敬请诸位大侠赐教

【调试环境】：`WinXP`、`OllyDBG`、`PEiD`

【分析过程】：`PEID` 载入无壳。。`Delphi` 编译的，输入错误有提示，拦截下来，中断在 `005CBC12`

```
005CBC12  |.  55            push ebp
005CBC13  |.  68 64BE5C00   push projshop.005CBE64
005CBC18  |.  64:FF30       push dword ptr fs:[eax]
005CBC1B  |.  64:8920       mov dword ptr fs:[eax],esp
005CBC1E  |.  8D45 E8       lea eax,dword ptr ss:[ebp-18]
005CBC21  |.  50            push eax
005CBC22  |.  B9 02000000   mov ecx,2
005CBC27  |.  BA 19000000   mov edx,19
005CBC2C  |.  8B83 40030000 mov eax,dword ptr ds:[ebx+340]
005CBC32  |.  E8 8D8CE3FF   call projshop.004048C4 ; (2,19,32 位 MD5)
005CBC37  |.  8D83 40030000 lea eax,dword ptr ds:[ebx+340]
005CBC3D  |.  50            push eax
005CBC3E  |.  B9 18000000   mov ecx,18
005CBC43  |.  BA 09000000   mov edx,9
005CBC48  |.  8B83 40030000 mov eax,dword ptr ds:[ebx+340]
005CBC4E  |.  E8 718CE3FF   call projshop.004048C4 ; (18,9,32 位 MD5)
005CBC53  |.  8D45 FC       lea eax,dword ptr ss:[ebp-4]
005CBC56  |.  50            push eax
005CBC57  |.  B9 04000000   mov ecx,4
005CBC5C  |.  BA 01000000   mov edx,1
005CBC61  |.  8B83 40030000 mov eax,dword ptr ds:[ebx+340]
005CBC67  |.  E8 588CE3FF   call projshop.004048C4 ; (4,1,32 位 MD5 值)
005CBC6C  |.  8D45 F8       lea eax,dword ptr ss:[ebp-8]
005CBC6F  |.  50            push eax
005CBC70  |.  B9 04000000   mov ecx,4
005CBC75  |.  BA 05000000   mov edx,5
005CBC7A  |.  8B83 40030000 mov eax,dword ptr ds:[ebx+340]
005CBC80  |.  E8 3F8CE3FF   call projshop.004048C4 ; (4,5,32 位 MD5 值)
005CBC85  |.  8D45 F4       lea eax,dword ptr ss:[ebp-C]
005CBC88  |.  50            push eax
005CBC89  |.  B9 04000000   mov ecx,4
005CBC8E  |.  BA 09000000   mov edx,9
005CBC93  |.  8B83 40030000 mov eax,dword ptr ds:[ebx+340]
005CBC99  |.  E8 268CE3FF   call projshop.004048C4 ; (4,9,32 位 MD5 值)
005CBC9E  |.  8D45 F0       lea eax,dword ptr ss:[ebp-10]
005CBCA1  |.  50            push eax
005CBCA2  |.  B9 04000000   mov ecx,4
005CBCA7  |.  BA 0D000000   mov edx,0D
005CBCAC  |.  8B83 40030000 mov eax,dword ptr ds:[ebx+340]
005CBCB2  |.  E8 0D8CE3FF   call projshop.004048C4 ; (4,0D,32 位 MD5 值)
005CBCB7  |.  8D45 E4       lea eax,dword ptr ss:[ebp-1C]
005CBCBA  |.  50            push eax
005CBCBB  |.  B9 08000000   mov ecx,8
005CBCC0  |.  BA 11000000   mov edx,11
005CBCC5  |.  8B83 40030000 mov eax,dword ptr ds:[ebx+340]
005CBCCB  |.  E8 F48BE3FF   call projshop.004048C4 ; (8,11,32 位 MD5 值)
005CBCD0  |.  8B55 E4       mov edx,dword ptr ss:[ebp-1C] ; 32 位 MD5 值后 8 位
005CBCD3  |.  8D45 EC       lea eax,dword ptr ss:[ebp-14]
005CBCD6  |.  8B4D E8       mov ecx,dword ptr ss:[ebp-18] ; 上面截取的后 8 位的前 2 位
005CBCD9  |.  E8 D289E3FF   call projshop.004046B0 ; 合并
005CBCDE  |.  8D55 DC       lea edx,dword ptr ss:[ebp-24]
005CBCE1  |.  8B83 14030000 mov eax,dword ptr ds:[ebx+314]
005CBCE7  |.  E8 340AEFFF   call projshop.004BC720
005CBCEC  |.  8B45 DC       mov eax,dword ptr ss:[ebp-24]
005CBCEF  |.  8D55 E0       lea edx,dword ptr ss:[ebp-20]
005CBCF2  |.  E8 5DD3E3FF   call projshop.00409054
005CBCF7  |.  8B55 E0       mov edx,dword ptr ss:[ebp-20] ; 假注册码
005CBCFA  |.  8B45 EC       mov eax,dword ptr ss:[ebp-14] ; 合并后的真注册码
005CBCFD  |.  E8 AE8AE3FF   call projshop.004047B0 ; 真假注册码比较
005CBD02    0F85 15010000  jnz projshop.005CBE1D ; 不相等就跳
005CBD08  |.  8D55 D8       lea edx,dword ptr ss:[ebp-28]
005CBD0B  |.  8B83 1C030000 mov eax,dword ptr ds:[ebx+31C]
005CBD11  |.  E8 0A0AEFFF   call projshop.004BC720
005CBD16  |.  8B55 D8       mov edx,dword ptr ss:[ebp-28] ; 假 4 位序列号(1)
005CBD19  |.  8B45 FC       mov eax,dword ptr ss:[ebp-4] ; 真 4 位序列号(1)
005CBD1C  |.  E8 8F8AE3FF   call projshop.004047B0 ; 比较
005CBD21    0F85 F6000000  jnz projshop.005CBE1D ; 不等则跳
005CBD27  |.  8D55 D4       lea edx,dword ptr ss:[ebp-2C]
005CBD2A  |.  8B83 20030000 mov eax,dword ptr ds:[ebx+320]
005CBD30  |.  E8 EB09EFFF   call projshop.004BC720
005CBD35  |.  8B55 D4       mov edx,dword ptr ss:[ebp-2C] ; 假 4 位序列号(2)
005CBD38  |.  8B45 F8       mov eax,dword ptr ss:[ebp-8] ; 真 4 位序列号(2)
005CBD3B  |.  E8 708AE3FF   call projshop.004047B0 ; 比较
005CBD40    0F85 D7000000  jnz projshop.005CBE1D ; 不等跳
005CBD46  |.  8D55 D0       lea edx,dword ptr ss:[ebp-30]
005CBD49  |.  8B83 24030000 mov eax,dword ptr ds:[ebx+324]
005CBD4F  |.  E8 CC09EFFF   call projshop.004BC720
005CBD54  |.  8B55 D0       mov edx,dword ptr ss:[ebp-30] ; 假 4 位序列号(3)
005CBD57  |.  8B45 F4       mov eax,dword ptr ss:[ebp-C] ; 真 4 位序列号(3)
005CBD5A  |.  E8 518AE3FF   call projshop.004047B0 ; 比较
005CBD5F    0F85 B8000000  jnz projshop.005CBE1D ; 不等跳
005CBD65  |.  8D55 CC       lea edx,dword ptr ss:[ebp-34]
005CBD68  |.  8B83 28030000 mov eax,dword ptr ds:[ebx+328]
005CBD6E  |.  E8 AD09EFFF   call projshop.004BC720
005CBD73  |.  8B55 CC       mov edx,dword ptr ss:[ebp-34] ; 假 4 位序列号(4)
005CBD76  |.  8B45 F0       mov eax,dword ptr ss:[ebp-10] ; 真 4 位序列号(4)
005CBD79  |.  E8 328AE3FF   call projshop.004047B0 ; 比较
005CBD7E    0F85 99000000  jnz projshop.005CBE1D ; 不等跳
005CBD84  |.  B2 01         mov dl,1
005CBD86  |.  A1 8C414400   mov eax,dword ptr ds:[44418C]
005CBD8B  |.  E8 FC84E7FF   call projshop.0044428C
．．．．．．．．．．．．．．．．．．．．．．．．．．．．．．．．．．
```

`004047B0` 这个 `call` 里的比较很简单，就不跟了

下面来看用户代码的生成和注册码及序列号要用到的 `md5` 值的生成，有兴趣的可以详细跟一下

```
005CBF18  /.  55            push ebp
005CBF19  |.  8BEC          mov ebp,esp
005CBF1B  |.  50            push eax
005CBF1C  |.  B8 10000000   mov eax,10
005CBF21  |>  81C4 04F0FFFF /add esp,-0FFC
005CBF27  |.  50            |push eax
005CBF28  |.  48            |dec eax
005CBF29  |.^ 75 F6         \jnz short projshop.005CBF21
005CBF2B  |.  8B45 FC       mov eax,dword ptr ss:[ebp-4]
005CBF2E  |.  83C4 D8       add esp,-28
005CBF31  |.  53            push ebx
005CBF32  |.  33C9          xor ecx,ecx
005CBF34  |.  898D D4FFFEFF mov dword ptr ss:[ebp+FFFEFFD4],ecx
005CBF3A  |.  898D DCFFFEFF mov dword ptr ss:[ebp+FFFEFFDC],ecx
005CBF40  |.  898D D8FFFEFF mov dword ptr ss:[ebp+FFFEFFD8],ecx
005CBF46  |.  898D E4FFFEFF mov dword ptr ss:[ebp+FFFEFFE4],ecx
005CBF4C  |.  898D E0FFFEFF mov dword ptr ss:[ebp+FFFEFFE0],ecx
005CBF52  |.  898D F8FFFEFF mov dword ptr ss:[ebp+FFFEFFF8],ecx
005CBF58  |.  894D FC       mov dword ptr ss:[ebp-4],ecx
005CBF5B  |.  8BD8          mov ebx,eax
005CBF5D  |.  33C0          xor eax,eax
005CBF5F  |.  55            push ebp
005CBF60  |.  68 CDC05C00   push projshop.005CC0CD
005CBF65  |.  64:FF30       push dword ptr fs:[eax]
005CBF68  |.  64:8920       mov dword ptr fs:[eax],esp
005CBF6B  |.  8D85 FCFFFEFF lea eax,dword ptr ss:[ebp+FFFEFFFC]
005CBF71  |.  83C9 FF       or ecx,FFFFFFFF
005CBF74  |.  33D2          xor edx,edx
```
