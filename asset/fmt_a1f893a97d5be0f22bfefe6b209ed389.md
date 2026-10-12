
```
004C1BED    8D45 DC         lea eax,dword ptr ss:[ebp-24]
004C1BF0    8B55 F8         mov edx,dword ptr ss:[ebp-8]
004C1BF3    8A541A FF       mov dl,byte ptr ds:[edx+ebx-1]      //ds:[00DDB14C]=30 ('0')，dl=4C ('L')
004C1BF7    E8 E828F4FF     call dumped_.004044E4
004C1BFC    8B45 DC         mov eax,dword ptr ss:[ebp-24]
004C1BFF    8D55 E0         lea edx,dword ptr ss:[ebp-20]
004C1C02    E8 1969F4FF     call dumped_.00408520
004C1C07    8B45 E0         mov eax,dword ptr ss:[ebp-20]
004C1C0A    BA 2C1D4C00     mov edx,dumped_.004C1D2C
004C1C0F    E8 EC2AF4FF     call dumped_.00404700
004C1C14    75 12           jnz short dumped_.004C1C28            //向上循环运算取字符所对应的 ASCII 码值
004C1C16    8D45 F4         lea eax,dword ptr ss:[ebp-C]
004C1C19    BA 381D4C00     mov edx,dumped_.004C1D38
004C1C1E    E8 A129F4FF     call dumped_.004045C4
004C1C23    E9 91000000     jmp dumped_.004C1CB9
004C1C28    8D45 D4         lea eax,dword ptr ss:[ebp-2C]
004C1C2B    8B55 F8         mov edx,dword ptr ss:[ebp-8]
004C1C2E    8A541A FF       mov dl,byte ptr ds:[edx+ebx-1]      //ds:[00DDB14C]=30 ('0')，dl=4C ('L')
004C1C32    E8 AD28F4FF     call dumped_.004044E4
004C1C37    8B45 D4         mov eax,dword ptr ss:[ebp-2C]
004C1C3A    8D55 D8         lea edx,dword ptr ss:[ebp-28]
004C1C3D    E8 DE68F4FF     call dumped_.00408520
004C1C42    8B45 D8         mov eax,dword ptr ss:[ebp-28]
004C1C45    BA 441D4C00     mov edx,dumped_.004C1D44
004C1C4A    E8 B12AF4FF     call dumped_.00404700
004C1C4F    75 0F           jnz short dumped_.004C1C60            //向上循环运算取字符所对应的 ASCII 码值
004C1C51    8D45 F4         lea eax,dword ptr ss:[ebp-C]
004C1C54    BA 501D4C00     mov edx,dumped_.004C1D50
004C1C59    E8 6629F4FF     call dumped_.004045C4
004C1C5E    EB 59           jmp short dumped_.004C1CB9
004C1C60    8D45 CC         lea eax,dword ptr ss:[ebp-34]
004C1C63    8B55 F8         mov edx,dword ptr ss:[ebp-8]
004C1C66    8A541A FF       mov dl,byte ptr ds:[edx+ebx-1]      //ds:[00DDB14C]=30 ('0')，dl=4C ('L')
004C1C6A    E8 7528F4FF     call dumped_.004044E4
004C1C6F    8B45 CC         mov eax,dword ptr ss:[ebp-34]
004C1C72    8D55 D0         lea edx,dword ptr ss:[ebp-30]
004C1C75    E8 A668F4FF     call dumped_.00408520
004C1C7A    8B45 D0         mov eax,dword ptr ss:[ebp-30]
004C1C7D    BA 5C1D4C00     mov edx,dumped_.004C1D5C
004C1C82    E8 792AF4FF     call dumped_.00404700
004C1C87    75 0F           jnz short dumped_.004C1C98            //向上循环运算取字符所对应的 ASCII 码值
004C1C89    8D45 F4         lea eax,dword ptr ss:[ebp-C]
004C1C8C    BA 681D4C00     mov edx,dumped_.004C1D68
004C1C91    E8 2E29F4FF     call dumped_.004045C4
004C1C96    EB 21           jmp short dumped_.004C1CB9
004C1C98    8D45 C8         lea eax,dword ptr ss:[ebp-38]
004C1C9B    8B55 F8         mov edx,dword ptr ss:[ebp-8]
004C1C9E    0FB6541A FF     movzx edx,byte ptr ds:[edx+ebx-1]   //机器码逐位 HEX 值转换
004C1CA3    83C2 31         add edx,31                           //edx=edx+31=30     关键计算值①
004C1CA6    83E2 7F         and edx,7F                           //edx=edx+7F=61     关键计算值②
004C1CA9    E8 3628F4FF     call dumped_.004044E4
004C1CAE    8B55 C8         mov edx,dword ptr ss:[ebp-38]        //edx=61
004C1CB1    8D45 F4         lea eax,dword ptr ss:[ebp-C]
004C1CB4    E8 0B29F4FF     call dumped_.004045C4
004C1CB9    43              inc ebx                              //ebx 自加 1，指向下一位
004C1CBA    4E              dec esi
004C1CBB  ^ 0F85 B6FEFFFF  jnz dumped_.004C1B77                   //向上作循环运算 12 次
004C1CC1    8BC7            mov eax,edi
004C1CC3    8B55 F4         mov edx,dword ptr ss:[ebp-C]         //真码出现，ASCII "aaarvsjvsccx"
004C1CC6    E8 8D26F4FF     call dumped_.00404358
004C1CCB    33C0            xor eax,eax                          //运算完毕，异或清零
004C1CCD    5A              pop edx                              //edx 清零
004C1CCE    59              pop ecx                              //ecx=2
004C1CCF    59              pop ecx
004C1CD0    64:8910         mov dword ptr fs:[eax],edx
004C1CD3    68 ED1C4C00     push dumped_.004C1CED
004C1CD8    8D45 C8         lea eax,dword ptr ss:[ebp-38]        //eax 清零
004C1CDB    BA 0E000000     mov edx,0E
004C1CE0    E8 4326F4FF     call dumped_.00404328
004C1CE5    C3              retn
004C1CE6  ^ E9 4120F4FF     jmp dumped_.00403D2C
004C1CEB  ^ EB EB          jmp short dumped_.004C1CD8
004C1CED    5F              pop edi
004C1CEE    5E              pop esi                              //esi 清零
004C1CEF    5B              pop ebx
004C1CF0    8BE5            mov esp,ebp
004C1CF2    5D              pop ebp
004C1CF3    C3              retn                                 //计算完毕返回
```

#### 算法总结

注册验证非常简单：

1. 把机器码（A）中的字符逐个转换成 ASCII 码，另存为（B）
2. 再逐个把机器码所对应的（B）字符的值加上 31，计算完后另存为（C）
3. 再把转换完后的（C），再转换为新的字符，作为序列号（SN）

#### 注册爆破点

```
004C1EB5    0F85 DA000000   jnz dumped_.004C1F95                 jnz 改 nop
```

#### 以本机为例

机器码（A）：`000AEB97B625`

---

ASCII 码（B）：`303030414542393742363235`
（B）+31 ：`313131313131313131313131`

---

转换码（C）：`6161617276736A6873676366`

---

注册码（SN）：`aaarvsjvsccx`

===

注册信息:

机器码：`000AEB97B625`

序列号：`aaarvsjvsccx`

===

#### 注册信息保存位置

`HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\hzred5`

〓本文完〓

---

版权所有（C）2005 KuNgBiM[DFCG] &nbsp;&nbsp;&nbsp;&nbsp; Copyright (C) 2005 KuNgBiM[DFCG]

---

Cracked BY KuNgBiM[DFCG]
2005-06-02
6:26:26 AM

`4dd466d0e1e0cc123afbe504da2462ca`
