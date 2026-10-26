
# 正宗 V2.31 版 简单算法分析

【破解日期】 2006年4月5日

【破解作者】 冷血书生[OCN][DFCG]

【作者邮箱】 meiyou

【作者主页】 meiyou

【使用工具】 OD

【破解平台】 Win9x/NT/2000/XP

【软件名称】 起名正宗 V2.31 用神版

【下载地址】 150K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3y4S2L8h3N6J5k6h3g2F1i4K6u0W2L8X3g2@1i4K6u0r3k6r3!0%4L8X3I4G2j5h3c8Q4x3V1k6k6f1#2)9#2k6W2q4y4h3W2A6Q4x3X3g2W2P5r3f1`.

【软件简介】 根据五格剖象法结合四柱，

a.提供个人名分析，个人辅助起名；
b.提供完备的公司、品牌、行业起名方案及行业名分析；
c.提供每个汉字的解释； d.提供判断号码吉凶的功能；
e.不须ODBC或BDE支持，速度快，功能全，代码小，概念准确；
f.可将分析的人名加注存档，是姓名爱好者分析研究的得力工具。
g.百家姓氏来源. h. 可结合卦象分析；
I.自动分析八字用神；

该版本程序在“四柱透视”中新增详细分析功能，该项功能综合业内认可的几种用神定位方法，从四柱表面组合、藏干五行、真假旺衰、与日元关联强度、寒热燥湿、得力与否、干支化合等诸多方面加以分析，迅速、准确判定五行喜忌，为您起名、析名时格局选定、字形、字五行取舍提供有力的参考依据；

八字用神在姓名学中占有举足轻重的地位，而多数用户不懂复杂的命理知识，Ver2.31用神版提供了准确的八字喜忌分析功能，五行喜忌自动推演，是姓名学爱好者、研究者的福音！

【软件大小】 4M

【加壳方式】 ASPack 1.00b -> Alexey Solodovnikov

【破解声明】 我是一只小菜鸟，偶得一点心得，愿与大家分享：）

--------------------------------------------------------------------------------

## 破解内容

```
004D8BB5   E8 B6F9FFFF   call Qmzz\_exe.004D8570                   ; 跟进,算法CALL
004D8BBA   8BF0         mov esi,eax                                     ; 真码转移
004D8BBC   3B75 FC       cmp esi,dword ptr ss:[ebp-4]                 ; 再比较
004D8BBF   0F85 15010000   jnz Qmzz\_exe.004D8CDA                   ; 不相等就OVER
```

////////////////////////////////////////////////////////////////////////////////////////////

///////////////////////////////// 跟进算法CALL ///////////////////////////////////////////

////////////////////////////////////////////////////////////////////////////////////////////

```
0004D857C   8955 FC       mov dword ptr ss:[ebp-4],edx               ; 机器码转移
004D857F   8B45 F8       mov eax,dword ptr ss:[ebp-8]
004D8582   E8 75C8F2FF   call Qmzz\_exe.00404DFC
004D8587   33C0         xor eax,eax
004D8589   55         push ebp
004D858A   68 31864D00   push Qmzz\_exe.004D8631
004D858F   64:FF30       push dword ptr fs:[eax]
004D8592   64:8920       mov dword ptr fs:[eax],esp
004D8595   8B45 F8       mov eax,dword ptr ss:[ebp-8]
004D8598   E8 77C6F2FF   call Qmzz\_exe.00404C14
004D859D   85C0         test eax,eax
004D859F   75 04       jnz short Qmzz\_exe.004D85A5
004D85A1   33DB         xor ebx,ebx
004D85A3   EB 76       jmp short Qmzz\_exe.004D861B
004D85A5   33F6         xor esi,esi
004D85A7   85C0         test eax,eax
004D85A9   7E 13       jle short Qmzz\_exe.004D85BE
004D85AB   BA 01000000   mov edx,1
004D85B0   8B4D F8       mov ecx,dword ptr ss:[ebp-8]
004D85B3   0FB64C11 FF   movzx ecx,byte ptr ds:[ecx+edx-1]           ; 依次取ASCII值
004D85B8   03F1         add esi,ecx
004D85BA   42         inc edx
004D85BB   48         dec eax
004D85BC ^ 75 F2       jnz short Qmzz\_exe.004D85B0         ; 循环计算
004D85BE   8BC6         mov eax,esi                             ; 上面结算结果
004D85C0   B9 DF000000   mov ecx,0DF                         ; DF
004D85C5   99         cdq
004D85C6   F7F9         idiv ecx                                 ; /DF
004D85C8   8BDA         mov ebx,edx                           ; 值为EDX
004D85CA   8BC6         mov eax,esi                           ; EAX转移
004D85CC   B9 85000000   mov ecx,85                         ; ECX=85
004D85D1   99         cdq
004D85D2   F7F9         idiv ecx                               ; /85
004D85D4   8855 F7       mov byte ptr ss:[ebp-9],dl           ; 值为DL
004D85D7   8BC6         mov eax,esi                           ; ESI转移
004D85D9   B9 AF000000   mov ecx,0AF                       ; ecx=AF
004D85DE   99         cdq
004D85DF   F7F9         idiv ecx                                 ; /AF
004D85E1   8BC2         mov eax,edx                           ; 值为EDX转移
004D85E3   33D2         xor edx,edx
004D85E5   8AD3         mov dl,bl
004D85E7   8BF2         mov esi,edx                           ; EDX=AB
004D85E9   C1E6 0C       shl esi,0C                           ; 左移C位
004D85EC   2BF2         sub esi,edx                           ; esi=esi-edx
004D85EE   33C9         xor ecx,ecx
004D85F0   8A4D F7       mov cl,byte ptr ss:[ebp-9]           ; 07
004D85F3   8BF9         mov edi,ecx                           ; edi=7
004D85F5   C1E7 08       shl edi,8                           ; 左移8位
004D85F8   2BF9         sub edi,ecx                           ; edi-ecx
004D85FA   03F7         add esi,edi                           ; esi+edi
004D85FC   8BF8         mov edi,eax                           ; eax转移
004D85FE   81E7 FF000000   and edi,0FF                       ; edi and 0FF
004D8604   6BFF 0F       imul edi,edi,0F                       ; edi=edi\*0F
004D8607   03F7         add esi,edi                           ; esi+edi
004D8609   03F2         add esi,edx                           ; esi+edx
004D860B   03F1         add esi,ecx                           ; esi+ecx
004D860D   25 FF000000   and eax,0FF                     ; eax and 0FF
004D8612   03F0         add esi,eax                           ; esi+eax
004D8614   8BC6         mov eax,esi                           ; ESI转移
004D8616   8B5D FC       mov ebx,dword ptr ss:[ebp-4]     ; 机器码转移
004D8619   33D8         xor ebx,eax                           ; eax xor 机器码=注册码
004D861B   33C0         xor eax,eax
004D861D   5A         pop edx
004D861E   59         pop ecx
004D861F   59         pop ecx
004D8620   64:8910       mov dword ptr fs:[eax],edx
004D8623   68 38864D00   push Qmzz\_exe.004D8638
004D8628   8D45 F8       lea eax,dword ptr ss:[ebp-8]
004D862B   E8 2CC3F2FF   call Qmzz\_exe.0040495C
004D8630   C3         retn
```

## 算法总结：

`7BB` + 用户名ASCII累加值= `sn1`

`sn1 mod DF` = `sn2`

`sn1 mod 85` = `sn3`

`sn1 mod AF` = `sn4`

`[(sn2 左移C位 - sn2) + (sn3 左移8位 - sn3) + (sn4 * F) + sn2 + sn3 + sn4] xor 机器码` = 注册码

--------------------------------------------------------------------------------

## 破解总结

没啥好总结的，纯属娱乐！

--------------------------------------------------------------------------------

【版权声明】 本文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
