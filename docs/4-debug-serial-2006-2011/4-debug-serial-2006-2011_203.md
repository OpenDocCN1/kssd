
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



```
005CBF76  |.  E8 75F5FFFF    call projshop.005CB4F0  
005CBF7B  |.  84C0           test al,al  
005CBF7D  |.  0F84 03010000 je projshop.005CC086  
005CBF83  |.  8D4D FC       lea ecx,dword ptr ss:[ebp-4]  
005CBF86  |.  8D85 FCFFFEFF lea eax,dword ptr ss:[ebp+FFFEFFFC]  
005CBF8C  |.  BA 71EC0F00   mov edx,0FEC71  
005CBF91  |.  E8 0AF6FFFF   call projshop.005CB5A0  
005CBF96  |.  8D85 F8FFFEFF lea eax,dword ptr ss:[ebp+FFFEFFF8]  
005CBF9C  |.  50            push eax  
005CBF9D  |.  8D95 E8FFFEFF lea edx,dword ptr ss:[ebp+FFFEFFE8]  
005CBFA3  |.  8B45 FC       mov eax,dword ptr ss:[ebp-4]  
005CBFA6  |.  E8 5DECFFFF   call projshop.005CAC08  
005CBFAB  |.  8D8D E8FFFEFF lea ecx,dword ptr ss:[ebp+FFFEFFE8]  
005CBFB1  |.  8B55 FC       mov edx,dword ptr ss:[ebp-4]  
005CBFB4  |.  B8 E4C05C00   mov eax,projshop.005CC0E4                  ;  ASCII "MD5String"  
005CBFB9  |.  E8 42010000   call projshop.005CC100  
005CBFBE  |.  8B95 F8FFFEFF mov edx,dword ptr ss:[ebp+FFFEFFF8]       ;  生成用户代码的 md5 值  
005CBFC4  |.  8D45 FC       lea eax,dword ptr ss:[ebp-4]  
005CBFC7  |.  E8 7084E3FF   call projshop.0040443C  
005CBFCC  |.  8D85 E0FFFEFF lea eax,dword ptr ss:[ebp+FFFEFFE0]  
005CBFD2  |.  50            push eax  
005CBFD3  |.  B9 07000000   mov ecx,7  
005CBFD8  |.  BA 01000000   mov edx,1  
005CBFDD  |.  8B45 FC       mov eax,dword ptr ss:[ebp-4]  
005CBFE0  |.  E8 DF88E3FF   call projshop.004048C4                    ;  取前 7 位  
005CBFE5  |.  8B85 E0FFFEFF mov eax,dword ptr ss:[ebp+FFFEFFE0]  
005CBFEB  |.  8D95 E4FFFEFF lea edx,dword ptr ss:[ebp+FFFEFFE4]  
005CBFF1  |.  E8 5ED0E3FF   call projshop.00409054  
005CBFF6  |.  8D85 E4FFFEFF lea eax,dword ptr ss:[ebp+FFFEFFE4]  
005CBFFC  |.  BA F8C05C00   mov edx,projshop.005CC0F8                 ;  ASCII "5281"  
005CC001  |.  E8 6686E3FF   call projshop.0040466C                    ;  合并为真正的用户代码,前 7 位+5281  
005CC006  |.  8B95 E4FFFEFF mov edx,dword ptr ss:[ebp+FFFEFFE4]  
005CC00C  |.  8B83 30030000 mov eax,dword ptr ds:[ebx+330]  
005CC012  |.  E8 D9B7E9FF   call projshop.004677F0  
005CC017  |.  8D95 D8FFFEFF lea edx,dword ptr ss:[ebp+FFFEFFD8]  
005CC01D  |.  8B83 30030000 mov eax,dword ptr ds:[ebx+330]  
005CC023  |.  E8 98B7E9FF   call projshop.004677C0  
005CC028  |.  8B85 D8FFFEFF mov eax,dword ptr ss:[ebp+FFFEFFD8]  
005CC02E  |.  8D95 DCFFFEFF lea edx,dword ptr ss:[ebp+FFFEFFDC]  
005CC034  |.  E8 1BD0E3FF   call projshop.00409054  
005CC039  |.  8B95 DCFFFEFF mov edx,dword ptr ss:[ebp+FFFEFFDC]  
005CC03F  |.  8D83 40030000 lea eax,dword ptr ds:[ebx+340]  
005CC045  |.  E8 AE83E3FF   call projshop.004043F8  
005CC04A  |.  8D85 D4FFFEFF lea eax,dword ptr ss:[ebp+FFFEFFD4]  
005CC050  |.  50            push eax  
005CC051  |.  8D95 E8FFFEFF lea edx,dword ptr ss:[ebp+FFFEFFE8]  
005CC057  |.  8B83 40030000 mov eax,dword ptr ds:[ebx+340]  
005CC05D  |.  E8 A6EBFFFF   call projshop.005CAC08  
005CC062  |.  8D8D E8FFFEFF lea ecx,dword ptr ss:[ebp+FFFEFFE8]  
005CC068  |.  8B55 FC       mov edx,dword ptr ss:[ebp-4]              ;  用户代码 md5 值  
005CC06B  |.  B8 E4C05C00   mov eax,projshop.005CC0E4                 ;  ASCII "MD5String"  
005CC070  |.  E8 8B000000   call projshop.005CC100                    ;  MD5  
005CC075  |.  8B95 D4FFFEFF mov edx,dword ptr ss:[ebp+FFFEFFD4]      ;  产生 32 位 MD5 值(序列号和注册码要用到)  
005CC07B  |.  8D83 40030000 lea eax,dword ptr ds:[ebx+340]  
005CC081  |.  E8 7283E3FF   call projshop.004043F8  
005CC086  |>  33C0          xor eax,eax  
005CC088  |.  5A            pop edx  
005CC089  |.  59            pop ecx  
005CC08A  |.  59            pop ecx  
005CC08B  |.  64:8910       mov dword ptr fs:[eax],edx  
005CC08E  |.  68 D4C05C00   push projshop.005CC0D4  
005CC093  |>  8D85 D4FFFEFF lea eax,dword ptr ss:[ebp+FFFEFFD4]  
005CC099  |.  E8 0683E3FF   call projshop.004043A4  
005CC09E  |.  8D85 D8FFFEFF lea eax,dword ptr ss:[ebp+FFFEFFD8]  
005CC0A4  |.  E8 FB82E3FF   call projshop.004043A4  
005CC0A9  |.  8D85 DCFFFEFF lea eax,dword ptr ss:[ebp+FFFEFFDC]
```

### 算法总结

`md5` 算法计算出 32 位用户代码 `md5` 值，取前 7 位，和字符串 `"5281"` 合并成最终的用户代码：前 7 位 + `5281`

利用用户代码，`md5` 算法计算出 32 位 `md5` 值，取此 `md5` 值的后 8 位，构成注册码：后 8 位 + 后 8 位的前 2 位

再取 9-24 位，每四位为一段序列号：`xxxx-xxxx-xxxx-xxxx`

除此之外，此软件的登陆密码也是明文显示，一样可轻松破解！

`4dd466d0e1e0cc123afbe504da2462ca`
