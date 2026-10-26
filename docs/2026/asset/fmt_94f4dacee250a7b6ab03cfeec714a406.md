
```
00427F5F  |.  81E1 FF00>and ecx, 0xFF                  ; |
00427F65  |.  50        push eax                       ; |<%02X>
00427F66  |.  8B4424 23 mov eax, dword ptr ss:[esp+0x2>; |
00427F6A  |.  81E2 FF00>and edx, 0xFF                  ; |
00427F70  |.  51        push ecx                       ; |<%02X>
00427F71  |.  8B4C24 26 mov ecx, dword ptr ss:[esp+0x2>; |
00427F75  |.  52        push edx                       ; |<%02X>
00427F76  |.  8B5424 29 mov edx, dword ptr ss:[esp+0x2>; |
00427F7A  |.  25 FF0000>and eax, 0xFF                  ; |
00427F7F  |.  81E1 FF00>and ecx, 0xFF                  ; |
00427F85  |.  50        push eax                       ; |<%02X>
00427F86  |.  8B4424 2C mov eax, dword ptr ss:[esp+0x2>; |
00427F8A  |.  81E2 FF00>and edx, 0xFF                  ; |
00427F90  |.  51        push ecx                       ; |<%02X>
00427F91  |.  8B4C24 2F mov ecx, dword ptr ss:[esp+0x2>; |
00427F95  |.  52        push edx                       ; |<%02X>
00427F96  |.  8B5424 32 mov edx, dword ptr ss:[esp+0x3>; |
00427F9A  |.  25 FF0000>and eax, 0xFF                  ; |
00427F9F  |.  81E1 FF00>and ecx, 0xFF                  ; |
00427FA5  |.  50        push eax                       ; |<%02X>
00427FA6  |.  8B4424 35 mov eax, dword ptr ss:[esp+0x3>; |
00427FAA  |.  81E2 FF00>and edx, 0xFF                  ; |
00427FB0  |.  51        push ecx                       ; |<%02X>
00427FB1  |.  8B4C24 38 mov ecx, dword ptr ss:[esp+0x3>; |
00427FB5  |.  52        push edx                       ; |<%02X>
00427FB6  |.  8B5424 3B mov edx, dword ptr ss:[esp+0x3>; |
00427FBA  |.  25 FF0000>and eax, 0xFF                  ; |
00427FBF  |.  81E1 FF00>and ecx, 0xFF                  ; |
00427FC5  |.  50        push eax                       ; |<%02X>
00427FC6  |.  8B4424 3E mov eax, dword ptr ss:[esp+0x3>; |
00427FCA  |.  81E2 FF00>and edx, 0xFF                  ; |
00427FD0  |.  51        push ecx                       ; |<%02X>
00427FD1  |.  8B4C24 41 mov ecx, dword ptr ss:[esp+0x4>; |
00427FD5  |.  52        push edx                       ; |<%02X>
00427FD6  |.  8B5424 44 mov edx, dword ptr ss:[esp+0x4>; |
00427FDA  |.  25 FF0000>and eax, 0xFF                  ; |
00427FDF  |.  81E1 FF00>and ecx, 0xFF                  ; |
00427FE5  |.  50        push eax                       ; |<%02X>
00427FE6  |.  8B8424 04>mov eax, dword ptr ss:[esp+0x1>; |
00427FED  |.  81E2 FF00>and edx, 0xFF                  ; |
00427FF3  |.  51        push ecx                       ; |<%02X>
00427FF4  |.  52        push edx                       ; |<%02X>
00427FF5  |.  68 C06F54>push ScnRec\_.00546FC0          ; |format = "%02X%02X%02X%02X%02X%02X%02X%02X%02X%02X%02X%02X%02X%02X%02X%02X"
00427FFA  |.  50        push eax                       ; |s
00427FFB  |.  FF15 D436>call Xdword ptr ds:[<&MSVCRT.s>; \sprintf
00428001  |.  83C4 48   add esp, 0x48
00428004  |.  8D4C24 24 lea ecx, dword ptr ss:[esp+0x2>
00428008  |.  C78424 C4>mov dword ptr ss:[esp+0xC4], ->
00428013  |.  E8 48C806>call ScnRec\_.00494860
00428018  |.  56        push esi                       ; /block
00428019  |.  E8 CEC007>call <jmp.&MFC42u.#operator de>; \free
0042801E  |.  83C4 04   add esp, 0x4
00428021  |>  5F        pop edi
00428022  |.  5E        pop esi
00428023  |.  B8 010000>mov eax, 0x1
00428028  |.  5D        pop ebp
00428029  |.  EB 02     jmp XScnRec\_.0042802D
0042802B  |>  33C0      xor eax, eax
0042802D  |>  8B8C24 B0>mov ecx, dword ptr ss:[esp+0xB>
00428034  |.  5B        pop ebx
00428035  |.  64:890D 0>mov dword ptr fs:[0], ecx
0042803C  |.  81C4 B800>add esp, 0xB8
00428042  \.  C2 0800   retn 0x8
```

可以看到这个函数吧新的邮箱名字符串经过了`call1`，`call2`，`call33`次运算过后再用`sprintf`打印出来。

跟进`call1`处428710；

```
00428710  /$  56        push esi
00428711  |.  8BF1      mov esi, ecx
00428713  |.  E8 080000>call ScnRec\_.00428720；再跟进
00428718  |.  8BC6      mov eax, esi
0042871A  |.  5E        pop esi
0042871B  \.  C3        retn
```

`Call428720`处：

```
00428720  /$  8BD1      mov edx, ecx
00428722  |.  57        push edi
00428723  |.  33C0      xor eax, eax
00428725  |.  B9 100000>mov ecx, 0x10
0042872A  |.  8D7A 18   lea edi, dword ptr ds:[edx+0x18]
0042872D  |.  8942 14   mov dword ptr ds:[edx+0x14], eax
00428730  |.  8942 10   mov dword ptr ds:[edx+0x10], eax
00428733  |.  C702 0123>mov dword ptr ds:[edx], 0x67452301
00428739  |.  C742 04 8>mov dword ptr ds:[edx+0x4], 0xEFCDAB89
00428740  |.  C742 08 F>mov dword ptr ds:[edx+0x8], 0x98BADCFE
00428747  |.  C742 0C 7>mov dword ptr ds:[edx+0xC], 0x10325476
0042874E  |.  83C2 58   add edx, 0x58
00428751  |.  F3:AB     rep stos dword ptr es:[edi]
00428753  |.  B9 100000>mov ecx, 0x10
00428758  |.  8BFA      mov edi, edx
0042875A  |.  F3:AB     rep stos dword ptr es:[edi]
0042875C  |.  C602 80   mov byte ptr ds:[edx], 0x80
0042875F  |.  5F        pop edi
00428760  \.  C3        retn
```

可以看到很奇怪的一些数据这些数据是`md5`的初始化数据也就`01234567890abcdeffedcba09876543210`不过光看到这些还不能确定是`md5`还要往后面看。返回后继续跟到`call2`处：

```
00428904  |.  23CB      and ecx, ebx
00428906  |.  8BFB      mov edi, ebx
00428908  |.  0BC1      or eax, ecx
0042890A  |.  8B4C24 14 mov ecx, dword ptr ss:[esp+0x14]
0042890E  |.  03C1      add eax, ecx
00428910  |.  8B4C24 10 mov ecx, dword ptr ss:[esp+0x10]
00428914  |.  8D8C01 78>lea ecx, dword ptr ds:[ecx+eax+0xD76AA478]；看到这些数据的时候基本上能确定是md5加密了。
0042891B  |.  8BC1      mov eax, ecx
0042891D  |.  C1E8 19   shr eax, 0x19
00428920  |.  C1E1 07   shl ecx, 0x7
00428923  |.  0BC1      or eax, ecx
00428925  |.  03C3      add eax, ebx
00428927  |.  8BC8      mov ecx, eax
00428929  |.  23F8      and edi, eax
0042892B  |.  F7D1      not ecx
0042892D  |.  23CD      and ecx, ebp
0042892F  |.  0BCF      or ecx, edi
00428931  |.  8B7C24 18 mov edi, dword ptr ss:[esp+0x18]
00428935  |.  03CF      add ecx, edi
00428937  |.  8D940A 56>lea edx, dword ptr ds:[edx+ecx+0xE8C7B756]；特殊常数
0042893E  |.  8BCA      mov ecx, edx
00428940  |.  C1E9 14   shr ecx, 0x14
00428943  |.  C1E2 0C   shl edx, 0xC
00428946  |.  0BCA      or ecx, edx
00428948  |.  03C8      add ecx, eax
0042894A  |.  8BD1      mov edx, ecx
```

后面的算法还有很多就不贴了，，，

跟出来后直接f8过`call3`了，跟到`sprintf`处可以看到加密后的字符串了：`C3F4499B98E4FAA793BBB76F3F6D66DA`

返回后会来到下面

```
004280F0  |> /8D5424 34 /lea edx, dword ptr ss:[esp+0x34]
004280F4  |. |6A 21     |push 0x21
004280F6  |. |52        |push edx
004280F7  |. |8D4424 18 |lea eax, dword ptr ss:[esp+0x18]
004280FB  |. |6A FF     |push -0x1
004280FD  |. |50        |push eax
004280FE  |. |6A 00     |push 0x0
00428100  |. |6A 00     |push 0x0
00428102  |. |FFD3      |call Xebx
00428104  |. |8D4C24 34 |lea ecx, dword ptr ss:[esp+0x34]
00428108  |. |8D5424 10 |lea edx, dword ptr ss:[esp+0x10]
0042810C  |. |51        |push ecx
0042810D  |. |52        |push edx
0042810E  |. |8BCD      |mov ecx, ebp
00428110  |. |E8 7BFDFF>|call ScnRec\_.00427E90                          ；md5加密
00428115  |. |33F6      |xor esi, esi
00428117  |> |66:8B4474>|/mov ax, word ptr ss:[esp+esi\*2+0x34]
0042811C  |. |50        ||push eax                                      ; /w
0042811D  |. |FF15 D036>||call Xdword ptr ds:[<&MSVCRT.iswalpha>]       ; \iswalpha
00428123  |. |83C4 04   ||add esp, 0x4
00428126  |. |85C0      ||test eax, eax
00428128  |. |0F85 8600>||jnz ScnRec\_.004281B4
0042812E  |. |8BC6      ||mov eax, esi
00428130  |. |25 010000>||and eax, 0x80000001
00428135  |. |79 05     ||jns XScnRec\_.0042813C
00428137  |. |48        ||dec eax
00428138  |. |83C8 FE   ||or eax, 0xFFFFFFFE
0042813B  |. |40        ||inc eax
0042813C  |> |74 16     ||je XScnRec\_.00428154
0042813E  |. |33C9      ||xor ecx, ecx
00428140  |. |66:8B4C74>||mov cx, word ptr ss:[esp+esi\*2+0x34]
00428145  |. |81E1 0100>||and ecx, 0x80000001
0042814B  |. |79 05     ||jns XScnRec\_.00428152
0042814D  |. |49        ||dec ecx
0042814E  |. |83C9 FE   ||or ecx, 0xFFFFFFFE
00428151  |. |41        ||inc ecx
00428152  |>  75 60     ||jnz ScnRec\_.004281B4
```
