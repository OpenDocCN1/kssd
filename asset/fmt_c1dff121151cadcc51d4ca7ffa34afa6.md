
**** 分隔符 ****

段首下断，输入试炼码。`F9` 运行，断下后调试。单步进入算法 `call`

**** 分隔符 ****

进入 算法 `call`

代码:

```
100014C1    55              push    ebp
 100014C2    8BEC            mov     ebp, esp
 100014C4    81EC 08040000   sub     esp, 408
 100014CA    53              push    ebx
 100014CB    56              push    esi
 100014CC    57              push    edi
 100014CD    8B7D 08         mov     edi, dword ptr [ebp+8]
 100014D0    33F6            xor     esi, esi
 100014D2    3977 08         cmp     dword ptr [edi+8], esi
 100014D5    0F84 F2000000   je      100015CD
 100014DB    3977 0C         cmp     dword ptr [edi+C], esi
 100014DE    0F84 E9000000   je      100015CD
 100014E4    8B5F 10         mov     ebx, dword ptr [edi+10]
 100014E7    3BDE            cmp     ebx, esi
 100014E9    74 30           je      short 1000151B
 100014EB    53              push    ebx
 100014EC    FF15 A4500010   call    dword ptr [<&KERNEL32.lstrlenW>] ; kernel32.lstrlenW
 100014F2    8D7C00 02       lea     edi, dword ptr [eax+eax+2]       ; 取用户名长度
 100014F6    8BC7            mov     eax, edi
 100014F8    83C0 03         add     eax, 3
 100014FB    24 FC           and     al, 0FC
 100014FD    E8 EE270000     call    10003CF0
 10001502    33C0            xor     eax, eax
 10001504    8BF4            mov     esi, esp
 10001506    50              push    eax
 10001507    50              push    eax
 10001508    8026 00         and     byte ptr [esi], 0
 1000150B    57              push    edi
 1000150C    56              push    esi
 1000150D    6A FF           push    -1
 1000150F    53              push    ebx
 10001510    50              push    eax
 10001511    50              push    eax
 10001512    FF15 A8500010   call    dword ptr [<&KERNEL32.WideCharTo>; kernel32.WideCharToMultiByte
 10001518    8B7D 08         mov     edi, dword ptr [ebp+8]           ; 宽字节转化为单字节
 1000151B    8D85 F8FBFFFF   lea     eax, dword ptr [ebp-408]
 10001521    56              push    esi
 10001522    50              push    eax
 10001523    E8 D8260000     call    10003C00
 10001528    59              pop     ecx
 10001529    33D2            xor     edx, edx                         ; edx 初值为 0
 1000152B    59              pop     ecx
 1000152C    33F6            xor     esi, esi                         ; esi 初值为 0
 1000152E    8955 FC         mov     dword ptr [ebp-4], edx           ; [ebp-4]初值为 0
 10001531    8D85 F8FBFFFF   lea     eax, dword ptr [ebp-408]         ; 用户名地址入 eax
 10001537    0FBE18          movsx   ebx, byte ptr [eax]             ; 循环取用户名 ASCII 值扩展送入 ebx
 1000153A    83FA 04         cmp     edx, 4                          ; 比较 edx 是否小于 4，进入不同算法
 1000153D    7D 14           jge     short 10001553                  ; 大于或等于 4 则跳向算法 2，即针对用户名 4 位后字符

```

……………………………………………………………………………………………………………………  

   当 `edx` 值小于 4 时即进入下面算法 1 部分，算法 1 也是计算注册码第一部分 16 进制值  

      `edx` 值是与取用户名 ASCII 所在位数值同步自增的，所以直接影响到用户名的字符部分  

      参与到注册码哪一部分的计算。  

……………………………………………………………………………………………………………………  

代码:

```
1000153F    6A 03           push    3                                 ; 常数 3 入栈
 10001541    81C3 E1100000   add     ebx, 10E1                         ; ebx=ebx+10E1h
 10001547    59              pop     ecx                                 ; 常数 3 出栈，ecx=3
 10001548    2BCA            sub     ecx, edx                            ; ecx=ecx-edx
 1000154A    C1E1 03         shl     ecx, 3                              ; ecx 逻辑左移 3 位
 1000154D    D3E3            shl     ebx, cl                             ; ebx 逻辑左移 cl 位
 1000154F    0BF3            or      esi, ebx                            ; esi=esi or ebx ，esi 最终值即为注册码第一部分 16 进制值
 10001551    EB 13           jmp     short 10001566

```

…………………………………………………………………………………………………………  

  当 `edx` 值大于或等于 4 时，进入算法 2.算法 2 是求注册码第二部分 16 进制值  

……………………………………………………………………………………………………………  

代码:

```
10001553    6A 07           push    7
 10001555    81C3 D2040000   add     ebx, 4D2                            ; ebx=ebx+4D2h
 1000155B    59              pop     ecx                                 ; ecx=7
 1000155C    2BCA            sub     ecx, edx                            ; ecx=ecx-edx
 1000155E    C1E1 03         shl     ecx, 3                               ; ecx 值逻辑左移 3 位
 10001561    D3E3            shl     ebx, cl                             ; ebx 逻辑左移 cl 位   
 10001563    095D FC         or      dword ptr [ebp-4], ebx              ; [ebp-4] 值与 ebx 相或， [ebp-4]最终值即为注册码第二部分 16 进制值     
 10001566    42              inc     edx                                 ; edx 自增 1
 10001567    83FA 08         cmp     edx, 8                               ; edx 值与 8 相比较
 1000156A    75 02           jnz     short 1000156E                       ; 不等于 8 则正常循环
 1000156C    33D2            xor     edx, edx                             ; 等于 8 则 edx 清 0，即用户名第 8 位起 4 位又进入算法 1
 1000156E    40              inc     eax                                 ; eax 自增 1
 1000156F    8038 00         cmp     byte ptr [eax], 0                   ; 判断用户名是否取完
 10001572  ^ 75 C3           jnz     short 10001537                      ; 未取完则继续循环取下一位
 10001574    3977 08         cmp     dword ptr [edi+8], esi              ; 比较 esi 值和试炼码第一部分 16 进制值
 10001577    75 4A           jnz     short 100015C3                      ; 不等则挂，爆破点
 10001579    8B45 FC         mov     eax, dword ptr [ebp-4]              ; ebp-4\]值传送到 eax
 1000157C    3947 0C         cmp     dword ptr [edi+C], eax              ; 比较\[ebp-4\]值和试炼码第二部分 16 进制值
 1000157F    75 42           jnz     short 100015C3                      ; 不等则挂，爆破点
 10001581    8B4D 0C         mov     ecx, dword ptr [ebp+C]              ; 上面两处相等则开始建立注册表项
 10001584    6A 01           push    1
 10001586    58              pop     eax
 10001587    8901            mov     dword ptr [ecx], eax
 10001589    8945 0C         mov     dword ptr [ebp+C], eax
 1000158C    8D45 08         lea     eax, dword ptr [ebp+8]
 1000158F    50              push    eax
 10001590    68 1C600010     push    1000601C                             ; ASCII "TypeInfo\\CLSID"
 10001595    68 00000080     push    80000000
 1000159A    FF15 1C500010   call    dword ptr [<&ADVAPI32.RegCreateK>; ADVAPI32.RegCreateKeyA
 100015A0    8D45 0C         lea     eax, dword ptr [ebp+C]
 100015A3    6A 04           push    4
 100015A5    50              push    eax
 100015A6    6A 03           push    3
 100015A8    6A 00           push    0
 100015AA    68 2C600010     push    1000602C                             ; ASCII "{39B6DCDB-01A1-49f0-8609-DB6C815D96BC}"
 100015AF    FF75 08         push    dword ptr [ebp+8]
 100015B2    FF15 18500010   call    dword ptr [<&ADVAPI32.RegSetValu>; ADVAPI32.RegSetValueExA
 100015B8    FF75 08         push    dword ptr [ebp+8]
 100015BB    FF15 24500010   call    dword ptr [<&ADVAPI32.RegCloseKe>; ADVAPI32.RegCloseKey
 100015C1    EB 06           jmp     short 100015C9
 100015C3    8B45 0C         mov     eax, dword ptr [ebp+C]
 100015C6    8320 00         and     dword ptr [eax], 0
 100015C9    33C0            xor     eax, eax
 100015CB    EB 05           jmp     short 100015D2
 100015CD    B8 05400080     mov     eax, 80004005
 100015D2    8DA5 ECFBFFFF   lea     esp, dword ptr [ebp-414]
 100015D8    5F              pop     edi
 100015D9    5E              pop     esi
 100015DA    5B              pop     ebx
 100015DB    C9              leave
 100015DC    C2 0800         retn    8
```
