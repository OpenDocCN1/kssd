
```
00405B7F  |.  7F 15            |jg      short 00405B96

00405B81  |.  0FBE45 F8        |movsx   eax, byte ptr [ebp-8]

00405B85  |.  B9 DB000000      |mov     ecx, 0DB /////DB 入 ecx

00405B8A  |.  2BC8             |sub     ecx, eax/////

00405B8C  |.  8B15 1CE54500    |mov     edx, [45E51C]

00405B92  |.  884C15 E4        |mov     [ebp+edx-1C], cl

00405B96  |>^ E9 60FFFFFF      \jmp     00405AFB///////////////////////循环

计算完成后的数组我们设为 X，X 共 21 位

////////////////////////////////计算完成后来到这里

00405B9B  |> \8A45 E5          mov     al, [ebp-1B]

00405B9E  |.  8845 F8          mov     [ebp-8], al///////////X 的第二位入 X 最后一位

00405BA1  |.  8A4D F6          mov     cl, [ebp-A]

00405BA4  |.  884D E5          mov     [ebp-1B], cl//////////X 的倒数第三位入第二位

00405BA7  |.  8A55 F8          mov     dl, [ebp-8]

00405BAA  |.  8855 F6          mov     [ebp-A], dl////////最后一位入 X 的倒数第三位

00405BAD  |.  8A45 E7          mov     al, [ebp-19]

00405BB0  |.  8845 F8          mov     [ebp-8], al//////////X 的第四位入最后一位

00405BB3  |.  8A4D EE          mov     cl, [ebp-12]

00405BB6  |.  884D E7          mov     [ebp-19], cl/////////X 的第十一位入第四位

00405BB9  |.  8A55 F8          mov     dl, [ebp-8]

00405BBC  |.  8855 EE          mov     [ebp-12], dl///////X 的最后一位入第十一位

00405BBF  |.  8A45 E9          mov     al, [ebp-17]

00405BC2  |.  8845 F8          mov     [ebp-8], al///////////X 第六位入最后一位

00405BC5  |.  8A4D F0          mov     cl, [ebp-10]

00405BC8  |.  884D E9          mov     [ebp-17], cl///////////X 的第十三位入第六位

00405BCB  |.  8A55 F8          mov     dl, [ebp-8]

00405BCE  |.  8855 F0          mov     [ebp-10], dl//////////X 的最后一位入第十三位

00405BD1  |.  8A45 EB          mov     al, [ebp-15]

00405BD4  |.  8845 F8          mov     [ebp-8], al//////////X 第八位入最后一位

00405BD7  |.  8A4D F2          mov     cl, [ebp-E]

00405BDA  |.  884D EB          mov     [ebp-15], cl/////////X 的第十五位入第八位

00405BDD  |.  8A55 F8          mov     dl, [ebp-8]

00405BE0  |.  8855 F2          mov     [ebp-E], dl//////////X 的最后一位入第十五位

00405BE3  |.  8A45 ED          mov     al, [ebp-13]

00405BE6  |.  8845 F8          mov     [ebp-8], al///////////X 第十位入最后一位

00405BE9  |.  8A4D F4          mov     cl, [ebp-C]

00405BEC  |.  884D ED          mov     [ebp-13], cl/////////X 的第十七位入第十位

00405BEF  |.  8A55 F8          mov     dl, [ebp-8]

00405BF2  |.  8855 F4          mov     [ebp-C], dl//////////X 的最后一位入第十七位

00405BF5  |.  C745 E0 00000>   mov     dword ptr [ebp-20], 0

00405BFC  |. /EB 09            jmp     short 00405C07

00405BFE  |> /8B45 E0          /mov    eax, [ebp-20]

00405C01  |. |83C0 01         |add     eax, 1

00405C04  |. |8945 E0         |mov     [ebp-20], eax

00405C07  |> |837D E0 14      cmp     dword ptr [ebp-20], 14

00405C0B  |. |7D 20           |jge     short 00405C2D

00405C0D  |. |8B4D 08        |mov     ecx, [ebp+8]

00405C10  |. |034D E0        |add     ecx, [ebp-20]

00405C13  |. |0FBE11         |movsx   edx, byte ptr [ecx]

00405C16  |. |8B45 E0        |mov     eax, [ebp-20]

00405C19  |. |0FBE4C05 E4    |movsx   ecx, byte ptr [ebp+eax-1C]

00405C1E  |. |3BD1           |cmp     edx, ecx

00405C20      |74 09         je      short 00405C2B

00405C22  |. |C745 FC 00000> |mov     dword ptr [ebp-4], 0

00405C29  |. |EB 02          |jmp     short 00405C2D

00405C2B  |>^\EB D1          \jmp     short 00405BFE

00405C2D  |>  8B45 FC        mov     eax, [ebp-4]

00405C30  |.  8BE5           mov     esp, ebp

00405C32  |.  5D            pop     ebp

00405C33  \.  C2 0400        retn    4/////////////////完成后返回

/////////////////////////////////////////////////

程序算法到此完成

第一次写破文，不当之处请指正

--------------------------------------------------------------------------------

【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

2006 年 11 月 14 日 12:10:05

`4dd466d0e1e0cc123afbe504da2462ca`
```
