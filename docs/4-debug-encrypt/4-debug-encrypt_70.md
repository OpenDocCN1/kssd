
# 一个桌面工具的 MD5 算法分析(国庆献礼)

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-447.htm`](https://www.pediy.com/kssd/pediy07/pediy7-447.htm)

【破解日期】 2005 年 10 月 1 日  
【破解作者】 冷血书生\[[OCN]\]\[CZG]\[D.4s]\[PYG]\[PCG]  
【作者邮箱】 `colddoctor@126.com`  
【使用工具】 `OD`, `PEID`, MD5 计算工具  
【破解平台】 `Win9x/NT/2000/XP`  
【软件名称】 `Easy Sticky Note V2.00`  
【下载地址】 `http://www2.skycn.com/soft/14099.html`  
【软件简介】 `Easy Sticky Note` 是一个记录和提醒的电子便条工具软件，能像桌面便条那样“贴”在你的桌面上。软件可以设置便条平时处于隐藏状态，然后在指定时间跳出提醒。  
【软件大小】 865 KB  
【加壳方式】 没壳  
【破解声明】 我是一只小菜鸟，偶得一点心得，愿与大家分享：）

--------------------------------------------------------------------------------

【破解内容】

`PEID` 检测是 `Microsoft Visual C++ 6.0` 编写，没壳！！爽！！跟踪到是 MD5 算法！`OD` 载入，字符插件找到出错提示，在 `0040BD68` 下断：

试炼信息:

`NAME:lengxue`

`code:1111-2222-3333-4444`

```
0040BD68  53       push ebx               ; 下断
0040BD69  55       push ebp
0040BD6A  56       push esi
0040BD6B  8BF1     mov esi,ecx
0040BD6D  57       push edi
0040BD6E  8B86 70010000  mov eax,dword ptr ds:[esi+170]
0040BD74  83F8 02  cmp eax,2
0040BD77  0F8F E8010000  jg StickyNo.0040BF65
0040BD7D  40       inc eax
0040BD7E  6A 01    push 1
0040BD80  8986 70010000  mov dword ptr ds:[esi+170],eax
0040BD86  E8 38210200   call StickyNo.0042DEC3
0040BD8B  8D86 60010000  lea eax,dword ptr ds:[esi+160]
0040BD91  8DBE 5C010000  lea edi,dword ptr ds:[esi+15C]
0040BD97  50       push eax
0040BD98  8D4C24 1C    lea ecx,dword ptr ss:[esp+1C]
0040BD9C  57       push edi
0040BD9D  51       push ecx
0040BD9E  E8 C7390200   call StickyNo.0042F76A
0040BDA3  8D96 64010000  lea edx,dword ptr ds:[esi+164]
0040BDA9  33DB     xor ebx,ebx
0040BDAB  52       push edx
0040BDAC  50       push eax
0040BDAD  8D4424 1C    lea eax,dword ptr ss:[esp+1C]
0040BDB1  895C24 34    mov dword ptr ss:[esp+34],ebx
0040BDB5  50       push eax
0040BDB6  E8 AF390200   call StickyNo.0042F76A
0040BDBB  8D8E 68010000  lea ecx,dword ptr ds:[esi+168]
0040BDC1  8D5424 10    lea edx,dword ptr ss:[esp+10]
0040BDC5  51       push ecx
0040BDC6  50       push eax
0040BDC7  52       push edx
0040BDC8  C64424 38 01  mov byte ptr ss:[esp+38],1
0040BDCD  E8 98390200   call StickyNo.0042F76A
0040BDD2  50       push eax
0040BDD3  8BCF     mov ecx,edi
0040BDD5  C64424 30 02  mov byte ptr ss:[esp+30],2
0040BDDA  E8 95380200   call StickyNo.0042F674
0040BDDF  8D4C24 10    lea ecx,dword ptr ss:[esp+10]
0040BDE3  C64424 2C 01  mov byte ptr ss:[esp+2C],1
0040BDE8  E8 4E370200   call StickyNo.0042F53B
0040BDED  8D4C24 14    lea ecx,dword ptr ss:[esp+14]
0040BDF1  885C24 2C    mov byte ptr ss:[esp+2C],bl
0040BDF5  E8 41370200   call StickyNo.0042F53B
0040BDFA  8D4C24 18    lea ecx,dword ptr ss:[esp+18]
0040BDFE  C74424 2C FFFFF>mov dword ptr ss:[esp+2C],-1
0040BE06  E8 30370200   call StickyNo.0042F53B
0040BE0B  68 20694500   push StickyNo.00456920
0040BE10  8D4C24 14    lea ecx,dword ptr ss:[esp+14]
0040BE14  E8 90370200   call StickyNo.0042F5A9
0040BE19  8DAE 58010000  lea ebp,dword ptr ds:[esi+158]
0040BE1F  8D4424 18    lea eax,dword ptr ss:[esp+18]
0040BE23  BB 03000000   mov ebx,3
0040BE28  55       push ebp
0040BE29  50       push eax
0040BE2A  B9 486A4500   mov ecx,StickyNo.00456A48
0040BE2F  895C24 34    mov dword ptr ss:[esp+34],ebx
0040BE33  E8 28F4FFFF   call StickyNo.0040B260            ; 跟进 1
0040BE38  50       push eax
0040BE39  8D4C24 14    lea ecx,dword ptr ss:[esp+14]
0040BE3D  C64424 30 04  mov byte ptr ss:[esp+30],4
0040BE42  E8 2D380200   call StickyNo.0042F674
0040BE47  8D4C24 18    lea ecx,dword ptr ss:[esp+18]
0040BE4B  885C24 2C    mov byte ptr ss:[esp+2C],bl
0040BE4F  E8 E7360200   call StickyNo.0042F53B            ; LE 与 wefwfrw3rf32wasfaf 连接
0040BE54  8B4424 10    mov eax,dword ptr ss:[esp+10]      ; ASCII "LEwefwfrw3rf32wasfaf"
0040BE58  8B48 F8     mov ecx,dword ptr ds:[eax-8]
0040BE5B  51       push ecx
0040BE5C  8D4C24 18    lea ecx,dword ptr ss:[esp+18]
0040BE60  50       push eax
0040BE61  51       push ecx
0040BE62  E8 19A2FFFF   call StickyNo.00406080            ; 跟进 2
0040BE67  83C4 0C     add esp,0C
0040BE6A  50       push eax
0040BE6B  8D4C24 14    lea ecx,dword ptr ss:[esp+14]
0040BE6F  C64424 30 05  mov byte ptr ss:[esp+30],5
0040BE74  E8 FB370200   call StickyNo.0042F674
0040BE79  8D4C24 14    lea ecx,dword ptr ss:[esp+14]
0040BE7D  885C24 2C    mov byte ptr ss:[esp+2C],bl
0040BE81  E8 B5360200   call StickyNo.0042F53B
0040BE86  8D5424 1C    lea edx,dword ptr ss:[esp+1C]
0040BE8A  6A 10     push 10
0040BE8C  52       push edx
0040BE8D  8D4C24 18    lea ecx,dword ptr ss:[esp+18]
0040BE91  E8 65D20100   call StickyNo.004290FB
0040BE96  50       push eax
0040BE97  8D4C24 14    lea ecx,dword ptr ss:[esp+14]
0040BE9B  C64424 30 06  mov byte ptr ss:[esp+30],6
0040BEA0  E8 CF370200   call StickyNo.0042F674
0040BEA5  8D4C24 1C    lea ecx,dword ptr ss:[esp+1C]
0040BEA9  885C24 2C    mov byte ptr ss:[esp+2C],bl
0040BEAD  E8 89360200   call StickyNo.0042F53B
0040BEB2  8B07     mov eax,dword ptr ds:[edi]
0040BEB4  50       push eax                 ; 假码
0040BEB5  8B4424 14    mov eax,dword ptr ss:[esp+14]      
0040BEB9  50       push eax                 ; 真码
0040BEBA  E8 69F60000   call StickyNo.0041B528            ; 跟进 4
0040BEBF  83C4 08     add esp,8
0040BEC2  85C0     test eax,eax
0040BEC4  75 53     jnz short StickyNo.0040BF19         ; 不相等就注册失败

******************************************************************** 跟进 1 ********************************************************************

0040B260  6A FF     push -1                 ; 跟进来到这里
0040B262  68 4FEC4300   push StickyNo.0043EC4F
0040B267  64:A1 00000000 mov eax,dword ptr fs:[0]
0040B26D  50       push eax
0040B26E  64:8925 0000000>mov dword ptr fs:[0],esp
0040B275  83EC 0C     sub esp,0C
0040B278  8B4424 20    mov eax,dword ptr ss:[esp+20]
0040B27C  53       push ebx
0040B27D  56       push esi
0040B27E  50       push eax
0040B27F  8D4C24 2C    lea ecx,dword ptr ss:[esp+2C]
0040B283  C74424 14 00000>mov dword ptr ss:[esp+14],0
0040B28B  E8 20400200   call StickyNo.0042F2B0
0040B290  BB 01000000   mov ebx,1
0040B295  8D4C24 28    lea ecx,dword ptr ss:[esp+28]
0040B299  895C24 1C    mov dword ptr ss:[esp+1C],ebx
0040B29D  E8 D8E20100   call StickyNo.0042957A
0040B2A2  8D4C24 28    lea ecx,dword ptr ss:[esp+28]
0040B2A6  E8 83E20100   call StickyNo.0042952E
0040B2AB  8D4C24 28    lea ecx,dword ptr ss:[esp+28]
0040B2AF  E8 74470200   call StickyNo.0042FA28           ;用户名小写转大写
0040B2B4  6A 42     push 42
0040B2B6  6A 2E     push 2E
0040B2B8  8D4C24 30    lea ecx,dword ptr ss:[esp+30]
0040B2BC  E8 FDDD0100   call StickyNo.004290BE
0040B2C1  6A 42     push 42
0040B2C3  6A 20     push 20
0040B2C5  8D4C24 30    lea ecx,dword ptr ss:[esp+30]
0040B2C9  E8 F0DD0100   call StickyNo.004290BE
0040B2CE  8B4C24 28    mov ecx,dword ptr ss:[esp+28]
0040B2D2  8B41 F8     mov eax,dword ptr ds:[ecx-8]
0040B2D5  83F8 02     cmp eax,2                  ;用户名长度与 2 比较  
0040B2D8  7E 4C     jle short StickyNo.0040B326         ;小于或等于就跳
0040B2DA  8D5424 0C    lea edx,dword ptr ss:[esp+C]
0040B2DE  6A 02     push 2                   ; 2 入栈
0040B2E0  52       push edx
0040B2E1  8D4C24 30    lea ecx,dword ptr ss:[esp+30]
0040B2E5  E8 11DE0100   call StickyNo.004290FB
0040B2EA  68 381B4500   push StickyNo.00451B38            ; ASCII "wefwfrw3rf32wasfaf"
0040B2EF  50       push eax
0040B2F0  8D4424 10    lea eax,dword ptr ss:[esp+10]
0040B2F4  C64424 24 02  mov byte ptr ss:[esp+24],2
```



```
0040B2F9  50       push eax  

0040B2FA  E8 D1440200   call StickyNo.0042F7D0  

0040B2FF  50       push eax  

0040B300  8D4C24 2C    lea ecx,dword ptr ss:\[esp+2C\]  

0040B304  C64424 20 03  mov byte ptr ss:\[esp+20\],3  

0040B309  E8 66430200   call StickyNo.0042F674  

0040B30E  8D4C24 08    lea ecx,dword ptr ss:\[esp+8\]  

0040B312  C64424 1C 02  mov byte ptr ss:\[esp+1C\],2  

0040B317  E8 1F420200   call StickyNo.0042F53B  

0040B31C  885C24 1C    mov byte ptr ss:\[esp+1C\],bl  

0040B320  8D4C24 0C    lea ecx,dword ptr ss:\[esp+C\]  

0040B324  EB 58      jmp short StickyNo.0040B37E  

0040B326  68 341B4500   push StickyNo.00451B34             ; ASCII "AA"  

0040B32B  8D4C24 2C    lea ecx,dword ptr ss:\[esp+2C\]  

0040B32F  E8 E3450200   call StickyNo.0042F917  

0040B334  8D4C24 08    lea ecx,dword ptr ss:\[esp+8\]  

0040B338  6A 02      push 2  

0040B33A  51       push ecx  

0040B33B  8D4C24 30    lea ecx,dword ptr ss:\[esp+30\]  

0040B33F  E8 B7DD0100   call StickyNo.004290FB  

0040B344  68 381B4500   push StickyNo.00451B38             ; ASCII "wefwfrw3rf32wasfaf"  

0040B349  8D5424 10    lea edx,dword ptr ss:\[esp+10\]  

0040B34D  50       push eax  

0040B34E  52       push edx  

0040B34F  C64424 28 04  mov byte ptr ss:\[esp+28\],4  

0040B354  E8 77440200   call StickyNo.0042F7D0  

0040B359  50       push eax  

0040B35A  8D4C24 2C    lea ecx,dword ptr ss:\[esp+2C\]  

0040B35E  C64424 20 05  mov byte ptr ss:\[esp+20\],5  

0040B363  E8 0C430200   call StickyNo.0042F674  

0040B368  8D4C24 0C    lea ecx,dword ptr ss:\[esp+C\]  

0040B36C  C64424 1C 04  mov byte ptr ss:\[esp+1C\],4  

0040B371  E8 C5410200   call StickyNo.0042F53B  

0040B376  885C24 1C    mov byte ptr ss:\[esp+1C\],bl  

0040B37A  8D4C24 08    lea ecx,dword ptr ss:\[esp+8\]  

0040B37E  E8 B8410200   call StickyNo.0042F53B  

0040B383  8B7424 24    mov esi,dword ptr ss:\[esp+24\]  

0040B387  8D4424 28    lea eax,dword ptr ss:\[esp+28\]  

0040B38B  50       push eax  

0040B38C  8BCE      mov ecx,esi  

0040B38E  E8 1D3F0200   call StickyNo.0042F2B0  

0040B393  895C24 10    mov dword ptr ss:\[esp+10\],ebx  

0040B397  8D4C24 28    lea ecx,dword ptr ss:\[esp+28\]  

0040B39B  C64424 1C 00  mov byte ptr ss:\[esp+1C\],0  

0040B3A0  E8 96410200   call StickyNo.0042F53B  

0040B3A5  8B4C24 14    mov ecx,dword ptr ss:\[esp+14\]  

0040B3A9  8BC6      mov eax,esi  

0040B3AB  5E       pop esi  

0040B3AC  5B       pop ebx  

0040B3AD  64:890D 0000000>mov dword ptr fs:\[0\],ecx  

0040B3B4  83C4 18     add esp,18  

0040B3B7  C2 0800     retn 8  
```

### 跟进 2

```
00406080  6A FF      push -1                 ; 跟进来到这里  

00406082  68 98E54300   push StickyNo.0043E598  

00406087  64:A1 00000000 mov eax,dword ptr fs:\[0\]  

0040608D  50       push eax  

0040608E  64:8925 0000000>mov dword ptr fs:\[0\],esp  

00406095  83EC 60     sub esp,60  

00406098  56       push esi  

00406099  8B7424 7C    mov esi,dword ptr ss:\[esp+7C\]  

0040609D  57       push edi  

0040609E  8B7C24 7C    mov edi,dword ptr ss:\[esp+7C\]  

004060A2  6A 00      push 0  

004060A4  56       push esi  

004060A5  57       push edi  

004060A6  C74424 14 00000>mov dword ptr ss:\[esp+14\],0  

004060AE  E8 B6360200   call StickyNo.00429769  

004060B3  8D4C24 0C    lea ecx,dword ptr ss:\[esp+C\]  

004060B7  E8 A40A0000   call StickyNo.00406B60             ; 跟进 3  

004060BC  56       push esi  

004060BD  57       push edi  

004060BE  8D4C24 14    lea ecx,dword ptr ss:\[esp+14\]  

004060C2  C74424 78 00000>mov dword ptr ss:\[esp+78\],0  

004060CA  E8 810C0000   call StickyNo.00406D50  

004060CF  8B7424 78    mov esi,dword ptr ss:\[esp+78\]  

004060D3  8D4C24 0C    lea ecx,dword ptr ss:\[esp+C\]  

004060D7  56       push esi  

004060D8  E8 130B0000   call StickyNo.00406BF0             ; 跟进可以看到 MD5 计算,有兴趣的不妨进去看看!!  

004060DD  8B4C24 68    mov ecx,dword ptr ss:\[esp+68\]  

004060E1  8BC6      mov eax,esi  

004060E3  5F       pop edi  

004060E4  5E       pop esi  

004060E5  64:890D 0000000>mov dword ptr fs:\[0\],ecx  

004060EC  83C4 6C     add esp,6C  

004060EF  C3       retn  
```

### 跟进 3

```
00406B60  8BD1      mov edx,ecx              ; 跟进来到这里!看到下面的常数,我就不用多说了!MD5 算法  

00406B62  57       push edi  

00406B63  B9 10000000   mov ecx,10  

00406B68  33C0      xor eax,eax  

00406B6A  8D7A 04     lea edi,dword ptr ds:\[edx+4\]  

00406B6D  C702 FC1D4400  mov dword ptr ds:\[edx\],StickyNo.00441DFC  

00406B73  F3:AB     rep stos dword ptr es:\[edi\]  

00406B75  8942 48     mov dword ptr ds:\[edx+48\],eax  

00406B78  8942 44     mov dword ptr ds:\[edx+44\],eax  

00406B7B  C742 4C 0123456>mov dword ptr ds:\[edx+4C\],67452301  

00406B82  C742 50 89ABCDE>mov dword ptr ds:\[edx+50\],EFCDAB89  

00406B89  C742 54 FEDCBA9>mov dword ptr ds:\[edx+54\],98BADCFE  

00406B90  C742 58 7654321>mov dword ptr ds:\[edx+58\],10325476  

00406B97  8BC2      mov eax,edx  

00406B99  5F       pop edi  

00406B9A  C3       retn  
```

### 跟进 4

```
0041B528  55       push ebp                 ; 跟进来到这里  

0041B529  8BEC      mov ebp,esp  

0041B52B  833D 8CA04500 0>cmp dword ptr ds:\[45A08C\],0  

0041B532  53       push ebx  

0041B533  56       push esi  

0041B534  57       push edi  

0041B535  75 12     jnz short StickyNo.0041B549  

0041B537  FF75 0C     push dword ptr ss:\[ebp+C\]  

0041B53A  FF75 08     push dword ptr ss:\[ebp+8\]  

0041B53D  E8 CE350000   call StickyNo.0041EB10  

0041B542  59       pop ecx  

0041B543  59       pop ecx  

0041B544  E9 89000000   jmp StickyNo.0041B5D2  

0041B549  6A 19      push 19  

0041B54B  E8 AA320000   call StickyNo.0041E7FA  

0041B550  8B75 0C     mov esi,dword ptr ss:\[ebp+C\]         ; 假码送 ESI  

0041B553  8B7D 08     mov edi,dword ptr ss:\[ebp+8\]  

0041B556  59       pop ecx  

0041B557  66:0FB60F    movzx cx,byte ptr ds:\[edi\]          ; 依次取"4fee7c9b10388c33"ACII 值并送 CX  

0041B55B  0FB6C1     movzx eax,cl              ; CL 送 EAX,EAX 初始为 0  

0041B55E  47       inc edi               ; EDI+1  

0041B55F  894D 0C     mov dword ptr ss:\[ebp+C\],ecx  

0041B562  F680 A1A14500 0>test byte ptr ds:\[eax+45A1A1\],4  

0041B569  74 16     je short StickyNo.0041B581  

0041B56B  8A07      mov al,byte ptr ds:\[edi\]  

0041B56D  84C0      test al,al  

0041B56F  75 06     jnz short StickyNo.0041B577  

0041B571  8365 0C 00   and dword ptr ss:\[ebp+C\],0  

0041B575  EB 0A     jmp short StickyNo.0041B581  

0041B577  33D2      xor edx,edx  

0041B579  47       inc edi  

0041B57A  8AF1      mov dh,cl  

0041B57C  8AD0      mov dl,al  

0041B57E  8955 0C     mov dword ptr ss:\[ebp+C\],edx  

0041B581  66:0FB61E    movzx bx,byte ptr ds:\[esi\]          ; 取假码第一位 ASCII 值并送 BX  

0041B585  0FB6C3     movzx eax,bl              ; BL 送 EAX  

0041B588  46       inc esi               ; ESI+1  

0041B589  F680 A1A14500 0>test byte ptr ds:\[eax+45A1A1\],4  

0041B590  74 13     je short StickyNo.0041B5A5  

0041B592  8A06      mov al,byte ptr ds:\[esi\]  

0041B594  84C0      test al,al  

0041B596  75 04     jnz short StickyNo.0041B59C  

0041B598  33DB      xor ebx,ebx  

0041B59A  EB 09     jmp short StickyNo.0041B5A5  

0041B59C  33C9      xor ecx,ecx  

0041B59E  46       inc esi  

0041B59F  8AEB      mov ch,bl  

0041B5A1  8AC8      mov cl,al  

0041B5A3  8BD9      mov ebx,ecx  

0041B5A5  66:395D 0C   cmp word ptr ss:\[ebp+C\],bx          ; 假码第一位与 MD5 后结果第一位比较  

0041B5A9  75 09     jnz short StickyNo.0041B5B4         ; 不相等就 OVER  

0041B5AB  66:837D 0C 00  cmp word ptr ss:\[ebp+C\],0          ; 再与 0 比较  
```



0041B5B0  74 16       je short StickyNo.0041B5C8          ; 相等就 OVER

0041B5B2 ^ EB A3      jmp short StickyNo.0041B557         ; 循环

--------------------------------------------------------------------------------

### 【算法总结】

(1) 注册码固定为 `16` 位

(2) 用户名若小写则转为大写，记为 `A`

(3) `"A 前两位"&"wefwfrw3rf32wasfaf"`，结果记为 `B`

(4) 将 `B` 进行 `MD5` 运算，结果记为 `C`

(5) 取 `C` 前十六位，则为注册码

一组可用的注册信息:

`NAME:lengxue`

`CODE:4fee-7c9b-1038-8c33`

--------------------------------------------------------------------------------

### 【内存注册机】

中断地址：`40BEB9`

中断次数：`1`

第一字节：`50`

指令长度：`1`

内存方式 `===》EAX===》指针 1 层`

--------------------------------------------------------------------------------

### 【爆破地址】

`0040BEC4  75 53      jnz short StickyNo.0040BF19  /// jnz ==>> je`

--------------------------------------------------------------------------------

### 【破解总结】

虽然是 `MD5` 算法，但还算是比较简单的，用工具就可以计算出注册码，不足之处，还请各位大侠指点！！

--------------------------------------------------------------------------------

### 【版权声明】

本文纯属技术交流，转载请注明作者并保持文章的完整，谢谢!

--------------------------------------------------------------------------------

2005.10.1 by 冷血书生

> 引用:
>
> ```
> 00406B7B  C742 4C 0123456>mov dword ptr ds:[edx+4C],67452301
> 00406B82  C742 50 89ABCDE>mov dword ptr ds:[edx+50],EFCDAB89
> 00406B89  C742 54 FEDCBA9>mov dword ptr ds:[edx+54],98BADCFE
> 00406B90  C742 58 7654321>mov dword ptr ds:[edx+58],10325476
> ```

支持，不过这个 `MD5` 没有变形，是个标准的 `MD5` 编码！

> 引用:
>
> ```
> 00406B7B  C742 4C 0123456>mov dword ptr ds:[edx+4C],67452301
> ```

执行过 `mov dword ptr ds:[edx+4C],67452301` 这句后，

内存中得到：`01 23 45 67`

`4dd466d0e1e0cc123afbe504da2462ca`
