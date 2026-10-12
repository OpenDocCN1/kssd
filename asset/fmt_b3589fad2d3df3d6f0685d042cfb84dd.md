
```
0040719F |. 8845 F5     mov   [ebp-B], al  
004071A2 |. 6A 04       push  4                   ; /Arg1 = 00000004  
004071A4 |. 8B4D E0     mov   ecx, [ebp-20]       ; |  
004071A7 |. 83C1 74     add   ecx, 74             ; |  
004071AA |. E8 81EAFFFF call  00405C30            ; \AVISplit.00405C30  
004071AF |. 8845 F9     mov   [ebp-7], al  
004071B2 |. 6A 05       push  5                   ; /Arg1 = 00000005  
004071B4 |. 8B4D E0     mov   ecx, [ebp-20]       ; |  
004071B7 |. 83C1 74     add   ecx, 74             ; |  
004071BA |. E8 71EAFFFF call  00405C30            ; \AVISplit.00405C30  
004071BF |. 8845 F7     mov   [ebp-9], al  
004071C2 |. 6A 06       push  6                   ; /Arg1 = 00000006  
004071C4 |. 8B4D E0     mov   ecx, [ebp-20]       ; |  
004071C7 |. 83C1 74     add   ecx, 74             ; |  
004071CA |. E8 61EAFFFF call  00405C30            ; \AVISplit.00405C30  
004071CF |. 8845 FE     mov   [ebp-2], al  
004071D2 |. 6A 07       push  7                   ; /Arg1 = 00000007  
004071D4 |. 8B4D E0     mov   ecx, [ebp-20]       ; |  
004071D7 |. 83C1 74     add   ecx, 74             ; |  
004071DA |. E8 51EAFFFF call  00405C30            ; \AVISplit.00405C30  
004071DF |. 8845 FB     mov   [ebp-5], al  
004071E2 |. 0FB655 EF   movzx edx, byte ptr [ebp-11]  
004071E6 |. 0FB645 FC   movzx eax, byte ptr [ebp-4]  
004071EA |. 83E8 30     sub   eax, 30             ; \\判断第一位注册码  
004071ED |. 3BD0        cmp   edx, eax  
004071EF |. 75 3C       jnz   short 0040722D      \\跳则死  
004071F1 |. 0FB64D F8   movzx ecx, byte ptr [ebp-8]     \\后面的判断是一样的，    
004071F5 |. 0FB655 FD   movzx edx, byte ptr [ebp-3]     \\他只检查前五位  
004071F9 |. 83EA 30     sub   edx, 30              
004071FC |. 3BCA        cmp   ecx, edx  
004071FE |. 75 2D       jnz   short 0040722D  
00407200 |. 0FB645 FF   movzx eax, byte ptr [ebp-1]  
00407204 |. 0FB64D F6   movzx ecx, byte ptr [ebp-A]  
00407208 |. 83E9 30     sub   ecx, 30  
0040720B |. 3BC1        cmp   eax, ecx  
0040720D |. 75 1E       jnz   short 0040722D  
0040720F |. 0FB655 FA   movzx edx, byte ptr [ebp-6]  
00407213 |. 0FB645 F5   movzx eax, byte ptr [ebp-B]  
00407217 |. 83E8 30     sub   eax, 30  
0040721A |. 3BD0        cmp   edx, eax  
0040721C |. 75 0F       jnz   short 0040722D      \\到这里前 5 位检查结束  
0040721E |. 0FB64D F4   movzx ecx, byte ptr [ebp-C]    
00407222 |. 0FB655 F9   movzx edx, byte ptr [ebp-7]  
00407226 |. 83EA 30     sub   edx, 30  
00407229 |. 3BCA        cmp   ecx, edx  
0040722B |. 74 58       je    short 00407285  
0040722D |> 0FB645 FC   movzx eax, byte ptr [ebp-4]  
00407231 |. 83F8 39     cmp   eax, 39  
00407234 |. 0F85 A7000000 jnz  004072E1  
0040723A |. 0FB64D FD   movzx ecx, byte ptr [ebp-3]  
0040723E |. 83F9 33     cmp   ecx, 33  
00407241 |. 0F85 9A000000 jnz  004072E1  
00407247 |. 0FB655 F6   movzx edx, byte ptr [ebp-A]  
0040724B |. 83FA 38     cmp   edx, 38  
0040724E |. 0F85 8D000000 jnz  004072E1  
00407254 |. 0FB645 F5   movzx eax, byte ptr [ebp-B]  
00407258 |. 83F8 38     cmp   eax, 38  
0040725B |. 0F85 80000000 jnz  004072E1  
00407261 |. 0FB64D F9   movzx ecx, byte ptr [ebp-7]  
00407265 |. 83F9 33     cmp   ecx, 33  
00407268 |. 75 77       jnz   short 004072E1  
0040726A |. 0FB655 F7   movzx edx, byte ptr [ebp-9]  
0040726E |. 83FA 31     cmp   edx, 31  
00407271 |. 75 6E       jnz   short 004072E1  
00407273 |. 0FB645 FE   movzx eax, byte ptr [ebp-2]  
00407277 |. 83F8 34     cmp   eax, 34  
0040727A |. 75 65       jnz   short 004072E1  
0040727C |. 0FB64D FB   movzx ecx, byte ptr [ebp-5]  
00407280 |. 83F9 36     cmp   ecx, 36  
00407283 |. 75 5C       jnz   short 004072E1  
00407285 |> 6A 00       push  0  
00407287 |. 6A 00       push  0  
00407289 |. 68 A4E24200 push  0042E2A4            ; registration has succeeded!  
0040728E |. E8 65F50100 call  004267F8  
00407293 |. 8B4D E0     mov   ecx, [ebp-20]  
00407296 |. 83C1 70     add   ecx, 70  
00407299 |. E8 62F2FFFF call  00406500  
0040729E |. 50          push  eax  
0040729F |. 68 C0E24200 push  0042E2C0            ; username  
004072A4 |. 68 CCE24200 push  0042E2CC            ; option  
004072A9 |. E8 A2ECFFFF call  00405F50  
004072AE |. 8BC8        mov   ecx, eax            ; |  
004072B0 |. E8 7FF60100 call  00426934            ; \AVISplit.00426934  
004072B5 |. 8B4D E0     mov   ecx, [ebp-20]  
004072B8 |. 83C1 74     add   ecx, 74  
004072BB |. E8 40F2FFFF call  00406500  
004072C0 |. 50          push  eax  
004072C1 |. 68 D4E24200 push  0042E2D4            ; registration_code  
004072C6 |. 68 E8E24200 push  0042E2E8            ; option  
004072CB |. E8 80ECFFFF call  00405F50  
004072D0 |. 8BC8        mov   ecx, eax            ; |  
004072D2 |. E8 5DF60100 call  00426934            ; \AVISplit.00426934  
004072D7 |. 8B4D E0     mov   ecx, [ebp-20]  
004072DA |. E8 7E830100 call  0041F65D  
004072DF |. EB 0E       jmp   short 004072EF  
004072E1 |> 6A 00       push  0  
004072E3 |. 6A 00       push  0  
004072E5 |. 68 F0E24200 push  0042E2F0            ; registration failed!
```

从上分析可以看出，这个算法很简单，基本上没有什么难度。

只是要求用户名 2 位以上，注册码 8 位以上，然后：

1. 取前两位注册码以 `abab` 形式排序，分别与 `53`、`41`、`56`、`49` 进行 `or` 运算
2. 将计算所得带符号数除法除 `oa`，得到余数序列，假设为 `a`
3. 用户名 `asc` 累加带符号数除法除 `oa`，得到余数序列，假设为 `b`
4. 注册码就是 `a + b +` 三位以上任意注册码

最后得到我的注册码是：

`gg1211`

`93913888`

`4dd466d0e1e0cc123afbe504da2462ca`
