
## 汇编代码分析

```assembly
00403270   /$  55             push ebp
00403271   |.  8BEC           mov ebp,esp
00403273   |.  57             push edi
00403274   |.  56             push esi
00403275   |.  53             push ebx
00403276   |.  8B75 0C        mov esi,dword ptr ss:[ebp+C]      ; ESI 0012F6CC ASCII "RWMR1A-517"
00403279   |.  8B7D 08        mov edi,dword ptr ss:[ebp+8]      ; EDI 0012F74C ASCII "RWMR1A-631"
0040327C   |.  8D05 005A42>   lea eax,dword ptr ds:[425A00]     ; EAX 00425A00 Registra.00425A00
00403282   |.  8378 08 00     cmp dword ptr ds:[eax+8],0        ; [eax+8]=0。
00403286   |.  75 3B          jnz short Registra.004032C3
00403288   |.  B0 FF          mov al,0FF
0040328A   |.  8BC0           mov eax,eax
0040328C   |>  0AC0           /or al,al
0040328E   |.  74 2E          |je short Registra.004032BE        ; 比较完后就跳
00403290   |.  8A06           |mov al,byte ptr ds:[esi]          ; 取假码到 al
00403292   |.  46             |inc esi
00403293   |.  8A27           |mov ah,byte ptr ds:[edi]          ; 取真码到 ah
00403295   |.  47             |inc edi
00403296   |.  38C4           |cmp ah,al                         ; 比较假码和真码
00403298   |.^ 74 F2         |je short Registra.0040328C        ; 相等就继续比较，我们要修改内存，使之相等。
0040329A   |.  2C 41          |sub al,41                         ; 41=asc(A)
0040329C   |.  3C 1A          |cmp al,1A                         ; 1A=26，26 个字母
0040329E   |.  1AC9           |sbb cl,cl
004032A0   |.  80E1 20        |and cl,20
004032A3   |.  02C1           |add al,cl
004032A5   |.  04 41          |add al,41                         ; 41-->61
004032A7   |.  86E0           |xchg al,ah
004032A9   |.  2C 41          |sub al,41
004032AB   |.  3C 1A          |cmp al,1A
004032AD   |.  1AC9           |sbb cl,cl
004032AF   |.  80E1 20        |and cl,20
004032B2   |.  02C1           |add al,cl
004032B4   |.  04 41          |add al,41                         ; 35=asc(5)
004032B6   |.  38E0           |cmp al,ah
004032B8   |.^ 74 D2          \je short Registra.0040328C
004032BA   |.  1AC0           sbb al,al
004032BC   |.  1C FF          sbb al,0FF                        ; 不相等的话，我想应该是置 eax 为 FF
004032BE   |>  0FBEC0         movsx eax,al                      ; 跳到这里，eax 置 0
004032C1   |.  EB 78          jmp short Registra.0040333B       ; 收工咯！
004032C3   |>  F0:FF05 E86>   lock inc dword ptr ds:[426DE8]
004032CA   |.  833D E46D42>   cmp dword ptr ds:[426DE4],0
004032D1   |.  7F 04          jg short Registra.004032D7
004032D3   |.  6A 00          push 0
004032D5   |.  EB 15          jmp short Registra.004032EC
004032D7   |>  F0:FF0D E86>   lock dec dword ptr ds:[426DE8]
004032DE   |.  6A 13          push 13
004032E0   |.  E8 EB370000    call Registra.00406AD0
004032E5   |.  C70424 0100>   mov dword ptr ss:[esp],1
004032EC   |>  B8 FF000000    mov eax,0FF
004032F1   |.  33DB           xor ebx,ebx
004032F3   |.  90             nop
004032F4   |>  0AC0           /or al,al
004032F6   |.  74 27          |je short Registra.0040331F
004032F8   |.  8A06           |mov al,byte ptr ds:[esi]
004032FA   |.  46             |inc esi
004032FB   |.  8A1F           |mov bl,byte ptr ds:[edi]
004032FD   |.  47             |inc edi
004032FE   |.  38D8           |cmp al,bl
00403300   |.^ 74 F2          |je short Registra.004032F4
00403302   |.  50             |push eax
00403303   |.  53             |push ebx
00403304   |.  E8 57390000    |call Registra.00406C60
00403309   |.  8BD8           |mov ebx,eax
0040330B   |.  83C4 04        |add esp,4
0040330E   |.  E8 4D390000    |call Registra.00406C60
00403313   |.  83C4 04        |add esp,4
00403316   |.  38C3           |cmp bl,al
00403318   |.^ 74 DA          \je short Registra.004032F4
0040331A   |.  1BC0           sbb eax,eax
0040331C   |.  83D8 FF        sbb eax,-1
0040331F   |>  8BD8           mov ebx,eax
00403321   |.  58             pop eax
00403322   |.  0BC0           or eax,eax
00403324   |.  75 09          jnz short Registra.0040332F
00403326   |.  F0:FF0D E86>   lock dec dword ptr ds:[426DE8]
0040332D   |.  EB 0A          jmp short Registra.00403339
0040332F   |>  6A 13          push 13                           ; /Arg1 = 00000013
00403331   |.  E8 1A380000    call Registra.00406B50            ; \Registra.00406B50
00403336   |.  83C4 04        add esp,4
00403339   |>  8BC3           mov eax,ebx
0040333B   |>  5B             pop ebx
0040333C   |.  5E             pop esi
0040333D   |.  5F             pop edi
0040333E   |.  C9             leave
0040333F   \.  C3             retn
```

7. 容易看到这一个系列的都是这样注册。

但是当你注册 `WMVCR.exe` 时，就不是那么一回事了。

分析一下 `WMVCR.exe`，发现这个程序自己有注册码验证的过程，自己也加了一些小花样，但不难分析。

过程基本相同，在找到 `"-"` 后的一段比较，稍有改变，如下：

```assembly
00403B47   |.  4E           dec esi                          ; esi 指向"-"的位置，但已被置为 00
                                                   ; esi 减 1 后指向"-"前面的字符
```

```assembly
00403B48   |.  8806         mov byte ptr ds:[esi],al          ; 再置 0
00403B4A   |.  8A4E FB      mov cl,byte ptr ds:[esi-5]        ; "-"前面的第 6 个字符
00403B4D   |.  83EE 05      sub esi,5
00403B50   |.  B0 47        mov al,47                         ; asc(47)="G"
00403B52   |.  3AC8         cmp cl,al                         ; 是"G"吗？
00403B54   |.  74 32        je short WMVCR.00403B88           ; Must jump
00403B56   |.  3846 01      cmp byte ptr ds:[esi+1],al        ; "-"前面的第 5 个字符
00403B59   |.  74 2D        je short WMVCR.00403B88           ; Must jump
00403B5B   |.  3846 03      cmp byte ptr ds:[esi+3],al        ; "-"前面的第 3 个字符
00403B5E   |.  74 28        je short WMVCR.00403B88           ; Must jump
```

如果注册码的 `"-"` 前面的第 6、5 或 3 个字符是 `"G"`，则跳到正确的地方置 `eax=2`。

但前面 4 个字符串又必须是 `"RWMV"`，这两个条件同时满足，`"-"` 前的长度最小为 7。

如果 `eax` 不为 2，分析下面的流程，发现有一个办法可以跳到 `00403B88`：

要满足：`edx>2` 或 `eax<=2`，但 `eax>=6F20503B`。

通过跟踪 `WMVCR.0040FA30`，发现这个 `call` 是计算 `"-"` 前的字母的和，和保存在：`edx:eax`，这是多大的数啊！死路！

```assembly
00403B60   |.  8D4C24 10    lea ecx,dword ptr ss:[esp+10]
00403B64   |>  8A01          /mov al,byte ptr ds:[ecx]
00403B66   |.  3C 30         |cmp al,30
00403B68   |.  7C 04         |jl short WMVCR.00403B6E
00403B6A   |.  3C 39         |cmp al,39
00403B6C   |.  7E 03         |jle short WMVCR.00403B71
00403B6E   |>  41            |inc ecx
00403B6F   |.^ EB F3         \jmp short WMVCR.00403B64
00403B71   |>  51            push ecx
00403B72   |.  E8 B9BE0000  call WMVCR.0040FA30               ; ****,edx>2
00403B77   |.  83C4 04       add esp,4
00403B7A   |.  83FA 02       cmp edx,2
00403B7D   |.  7C 19         jl short WMVCR.00403B98           ; No Jump
00403B7F   |.  7F 07         jg short WMVCR.00403B88
00403B81   |.  3D 3B50206F  cmp eax,6F20503B
00403B86   |.  76 10         jbe short WMVCR.00403B98          ; No JUMP
00403B88   |>  5F            pop edi
00403B89   |.  5E            pop esi
00403B8A   |.  5D            pop ebp
00403B8B   |.  B8 02000000   mov eax,2
00403B90   |.  5B            pop ebx
00403B91   |.  81C4 C00000> add esp,0C0
00403B97   |.  C3            retn
```

### 破解总结

Too easy.

注册机用 `win32 c` 编写。国庆节期破解完，这两天学习很忙，挤时间写的。另外两篇也会陆续写出来。

### Greetings

看雪论坛, FCG 论坛, DFCG 论坛, PYG 论坛等

### 完稿时间等

2005.10.8，下午 16:45，天气: 晴

武汉

### 注册机源码

附件: `source.rar`

`4dd466d0e1e0cc123afbe504da2462ca`
