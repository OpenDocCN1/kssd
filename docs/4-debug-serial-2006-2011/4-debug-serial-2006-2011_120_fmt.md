
# Quickfill v2.4

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-374.htm`](https://www.pediy.com/kssd/pediy08/pediy8-374.htm)

## 文章信息
【文章标题】: `Quickfill` `v2.4` 注册算法分析  
【文章作者】: `elance`  
【作者邮箱】: no  
【作者主页】: no  
【作者 QQ 号】: `275199621`  
【软件名称】: `Quickfill` `v2.4`  
【软件大小】: no  
【下载地址】: 自己搜索下载  
【加壳方式】: no  
【保护方式】: 注册码  
【使用工具】: `od`  
【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

## 详细过程

软件已更新到 `2.41` 了，注册算法好像也变了（我没试过 `2.41` 的，听说是变了）。好，其他废话不说了，进入正题：

点注册按钮后来到：

```
004EC1FE  |.  E8 F58AF1FF   call    00404CF8                ；求字符长度
004EC203  |.  0FAFF0        imul    esi, eax                ；用户名和注册码有一项为空
004EC206  |.  85F6          test    esi, esi                ；就出错
004EC208  |.  0F84 CF010000 je      004EC3DD                ；出错
004EC20E  |.  8D55 E8       lea     edx, [ebp-18]
004EC211  |.  8B83 0C030000 mov     eax, [ebx+30C]
004EC217  |.  E8 0048F7FF   call    00460A1C
004EC21C  |.  8B45 E8       mov     eax, [ebp-18]
004EC21F  |.  8D55 EC       lea     edx, [ebp-14]
004EC222  |.  E8 A9CCF1FF   call    00408ED0                ;  若用户名为小写则转为大写
004EC227  |.  8B45 EC       mov     eax, [ebp-14]
004EC22A  |.  8D4D F0       lea     ecx, [ebp-10]
004EC22D  |.  66:BA 3E03    mov     dx, 33E                 ；下边要用到的参数，设为 A
004EC231  |.  E8 CAD0FDFF   call    004C9300                ;  此处为关键，对用户名作变换
F7 进入 004C9300;
004C9343  |.  8A543A FF     |mov     dl, [edx+edi-1]         ；逐为取用户名（大写）
004C9347  |.  0FB7CE        |movzx   ecx, si                 ；参数 A
004C934A  |.  C1E9 08       |shr     ecx, 8                  ；A 右移 8 位
004C934D  |.  32D1          |xor     dl, cl                  ；用户名和 A 作 OR
004C934F  |.  885438 FF     |mov     [eax+edi-1], dl
004C9353  |.  8B45 00       |mov     eax, [ebp]
004C9356  |.  0FB64438 FF   |movzx   eax, byte ptr [eax+edi-1]
004C935B  |.  66:03F0       |add     si, ax                  ；A=A+用户名
004C935E  |.  66:69C6 D500  |imul    ax, si, 0D5             ；运算
004C9363  |.  66:05 9805    |add     ax, 598                 ；运算
004C9367  |.  8BF0          |mov     esi, eax                ；A=运算结果
004C9369  |.  43            |inc     ebx
004C936A  |.  FE4C24 04     |dec     byte ptr [esp+4]
004C936E  |.^ 75 C1         \jnz    short 004C9331
```

做完上边的，用户名的变换也完成了；

然后就到了计算注册码的过程了：

```
00409459  |> /31D2          /xor    edx, edx                ；此时 EAX 里存的是用户名的变换，ECX 为定值 A
0040945B  |. |F7F1          |div    ecx                     ；EAX/ECX
0040945D  |. |4E            |dec    esi
0040945E  |. |80C2 30       |add    dl, 30
00409461  |. |80FA 3A       |cmp    dl, 3A
00409464  |. |72 03         |jb     short 00409469
00409466  |. |80C2 07       |add    dl, 7                   ；将余数加 7 得到注册码
00409469  |> |8816          |mov    [esi], dl
0040946B  |. |09C0          |or     eax, eax
0040946D  |.^ \75 EA        \jnz   short 00409459
```

算法总结：

将用户名钻为大写，然后逐位变换：`A0=0X033E`，`A[i]=(n[i]+A[i-1])*0x0d5+0x598`，`n[i]=(A>>8) or n[i]`；得到变换后的 `n[]`后，`n[i]/(0x0a)`，取其余数 `d[i]`，`d[i]=d[i]+7`，对 `d[i]` 只娶一个字。（如，如算出来的是 163，则只取 16）

提供一组可用码：

用户名：`elance`
注册码：`F1922162025`

## 经验总结

没啥好总结的，呵呵，娱乐而已！！！

## 版权声明
本文原创于看雪技术论坛和霏凡论坛, 转载请注明作者并保持文章的完整, 谢谢!

2006 年 10 月 25 日 16:45:31
`4dd466d0e1e0cc123afbe504da2462ca`
