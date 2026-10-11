
```
010031D2   |.  56                    push    esi                              ; 列  
010031D3   |.  E8 D7FEFFFF          call    winmine.010030AF                 ; 显示该位置的数字  
010031D8   |.  A1 F4560001          mov     eax, dword ptr ds:[10056F4]      ;   
010031DD   |.  3B05 FC560001        cmp     eax, dword ptr ds:[10056FC]  
010031E3   |.  75 07                jnz     short winmine.010031EC           ; 雷扫完了么  
010031E5   |.  6A 01                push    1  
010031E7   |>  E8 86FDFFFF          call    winmine.01002F72                 ; 游戏成功结束  
010031EC   |>  5F                   pop     edi                              ; 没扫完，继续  
010031ED   |.  5E                   pop     esi  
010031EE   |.  5D                   pop     ebp  
010031EF   |.  5B                   pop     ebx  
010031F0   \.  C2 0800              retn    8  
```

根据行、列，调用下面过程计算，内存中的值如果高位是 `1`（`0x80`）就是雷

```
010033E9   /$  8B4424 08           mov     eax, dword ptr ss:[esp+8]   //行
010033ED   |.  8B4C24 04           mov     ecx, dword ptr ss:[esp+4]   //列
010033F1   |.  C1E0 05             shl     eax, 5
010033F4   |.  8D9408 00570001     lea     edx, dword ptr ds:[eax+ecx+1005700]  //位置
010033FB   |.  8A8408 00570001     mov     al, byte ptr ds:[eax+ecx+1005700]    
01003402   |.  8AC8                mov     cl, al
01003404   |.  83E1 1F             and     ecx, 1F
01003407   |.  83F9 0D             cmp     ecx, 0D
0100340A   |.  75 05               jnz     short winmine.01003411
0100340C   |.  6A 09               push    9
0100340E   |.  59                  pop     ecx
0100340F   |.  EB 07               jmp     short winmine.01003418
01003411   |>  83F9 0F             cmp     ecx, 0F
01003414   |.  75 02               jnz     short winmine.01003418
01003416   |.  33C9                xor     ecx, ecx
01003418   |>  24 E0               and     al, 0E0
0100341A   |.  0AC1                or      al, cl
0100341C   |.  8802                mov     byte ptr ds:[edx], al
0100341E   \.  C2 0800             retn    8
```

根据行、列，将上面过程变换后内存中的值恢复

```
01003421   /$  8B4424 08           mov     eax, dword ptr ss:[esp+8]
01003425   |.  8B4C24 04           mov     ecx, dword ptr ss:[esp+4]
01003429   |.  C1E0 05             shl     eax, 5
0100342C   |.  8D9408 00570001     lea     edx, dword ptr ds:[eax+ecx+1005700]
01003433   |.  8A8408 00570001     mov     al, byte ptr ds:[eax+ecx+1005700]
0100343A   |.  8AC8                mov     cl, al
0100343C   |.  83E1 1F             and     ecx, 1F
0100343F   |.  83F9 09             cmp     ecx, 9
01003442   |.  75 04               jnz     short winmine.01003448
01003444   |.  6A 0D               push    0D
01003446   |.  EB 06               jmp     short winmine.0100344E
01003448   |>  85C9                test    ecx, ecx
0100344A   |.  75 03               jnz     short winmine.0100344F
0100344C   |.  6A 0F               push    0F
0100344E   |>  59                  pop     ecx
0100344F   |>  24 E0               and     al, 0E0
01003451   |.  0AC1                or      al, cl
01003453   |.  8802                mov     byte ptr ds:[edx], al
01003455   \.  C2 0800             retn    8
```

## 二、逆向

1、用 `ExeScope` 为 `Winmine.exe` 添加一个菜单项：`ID=0x259=601`（大于 `0x20B`）,`Name="过关"`。

2、在 `Winmine.exe` 的消息循环中作如下修改，使自己的添加的菜单选项能够被响应。

```
01001A30    > \B9 4E020000        mov     ecx, 24E
01001A35    .  3BC1              cmp     eax, ecx
```

===〉 修改成

```
01001A30      /E9 4B2F0000        jmp     winmine.01004980  //跳转到自己的处理代码处
01001A35    . |3BC1              cmp     eax, ecx
```

3、在 `01004980` 处写入如下代码：

下面的代码是在 `Winmine.exe` 中找了一段全为零的地方写入的，也可以添加一个节后写入节中。

```
01004980       3D 58020000       cmp     eax, 258                     ;  
01004985       7F 0A             jg      short winmine.01004991       ;  > 0x258  
01004987       B9 4E020000       mov     ecx, 24E                     ;  恢复原来的代码
0100498C     ^ E9 A4D0FFFF       jmp     winmine.01001A35             ;  跳回
01004991       3D 59020000       cmp     eax, 259                     ;  是否选择了“过关”
01004996       7F 40             jg      short winmine.010049D8       ;  > 0x259 ？跳 继续处理其它
```

下面是过关代码

```
01004998       B8 01000000       mov     eax, 1                       ;  双重循环遍历;
0100499D       B9 01000000       mov     ecx, 1
010049A2       50                push    eax
010049A3       51                push    ecx
010049A4       50                push    eax
010049A5       51                push    ecx
010049A6       50                push    eax
010049A7       51                push    ecx
010049A8       E8 3CEAFFFF       call    winmine.010033E9             ;  变换
010049AD       A8 80             test    al, 80                       ; 是否为雷
010049AF       75 07             jnz     short winmine.010049B8       ;  
010049B1       E8 F9E6FFFF       call    winmine.010030AF             ; 不是雷，点开
010049B6       EB 05             jmp     short winmine.010049BD
010049B8       E8 64EAFFFF       call    winmine.01003421             ; 是雷，恢复
010049BD       59                pop     ecx
010049BE       58                pop     eax
010049BF       41                inc     ecx                            ;  列数增 1
010049C0       3B0D CC520001     cmp     ecx, dword ptr ds:[10052CC]    ;  遍历完该行的所有列？
010049C6     ^ 7E DA             jle     short winmine.010049A2
010049C8       40                inc     eax                            ;  行数增 1
010049C9       3B05 C8520001     cmp     eax, dword ptr ds:[10052C8]    ;  遍历完所有行？
010049CF     ^ 7E CC             jle     short winmine.0100499D
010049D1       6A 01             push    1
010049D3       E8 9AE5FFFF       call    winmine.01002F72              ;  游戏过关
010049D8     ^ E9 58D3FFFF       jmp     winmine.01001D35             ;  返回到原来的消息循环结束处
```

在 `OD` 中写入代码后，复制到可执行文件，保存即可！

4、运行 `Winmine.exe`,选择菜单中的“过关”即可！

【完】

`qINGfENG`  
`2005-01-14`
`4dd466d0e1e0cc123afbe504da2462ca`
