
```
00402D69   .  83C4 08       add esp,8  
00402D6C   .  C2 1000       retn 10  
00402D6F   >  8B4C24 1C     mov ecx,dword ptr ss:[esp+1C]  
00402D73   .  6A 00         push 0                     ; /Style = MB_OK|MB_APPLMODAL  
00402D75   .  68 B0B74100   push openvcap.0041B7B0      ; |error  
00402D7A   .  68 0CB74100   push openvcap.0041B70C      ; |registration failed!  
00402D7F   .  51            push ecx                   ; |hOwner  
00402D80   .  FF15 68B24100 call dword ptr ds:<&USER32.Messa>; \\MessageBoxA  
00402D86   .  5F            pop edi  
00402D87   .  5E            pop esi  
00402D88   .  5D            pop ebp  
00402D89   .  B8 01000000   mov eax,1  
00402D8E   .  5B            pop ebx  
00402D8F   .  83C4 08       add esp,8  
00402D92   .  C2 1000       retn 10  
```

**算法总结**

软件采算法比较简单，注册名须不小于两位，注册码位数为 8 位以上，主要思路如下：

1. 注册名第一位的 `ascii` 码与 `57` `or` 运算，再与 `A` 进行 `idiv` 运算，余数 `9` 为注册码第一位；
2. 注册名第二位的 `ascii` 码与 `45` `or` 运算，再与 `A` 进行 `idiv` 运算，余数 `9` 为注册码第二位；
3. 注册名第一位的 `ascii` 码与 `42` `or` 运算，再与 `A` 进行 `idiv` 运算，余数 `8` 为注册码第三位；
4. 注册名第二位的 `ascii` 码与 `43` `or` 运算，再与 `A` 进行 `idiv` 运算，余数 `7` 为注册码第四位；

此时 `ebp=3C9`

5. 将 `3C9` 与 `43` `or` 运算，再与 `A` 进行 `idiv` 运算，余数 `9` 为注册码第五位；
6. 第六位以后任意

合起来，即注册名：`tigerisme` 注册码为：`99879*****`，这里还有一组通用注册码 `23913146`。

**特别说明**

本文仅是一些破解的心得和思路，完全是个人对程序的研究，无其他目的。

`4dd466d0e1e0cc123afbe504da2462ca`
