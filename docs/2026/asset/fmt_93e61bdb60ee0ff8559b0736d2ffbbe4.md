
# 序列号检查代码段

``` 
00485865       |.  51              push ecx
00485866       |.  E8 39F9FFFF     call oC4PE.004851A4                   ;  序列号检查函数4-1
0048586B       |.  83C4 08         add esp,8
0048586E       |.  84C0            test al,al
00485870       |.  74 6D           je short oC4PE.004858DF
00485872       |.  0FBE47 05       movsx eax,byte ptr ds:[edi+5]         ;  序列号第6位
00485876       |.  8D5424 10       lea edx,dword ptr ss:[esp+10]
0048587A       |.  52              push edx
0048587B       |.  50              push eax
0048587C       |.  E8 23F9FFFF     call oC4PE.004851A4                   ;  序列号检查函数4-1
00485881       |.  83C4 08         add esp,8
00485884       |.  84C0            test al,al
00485886       |.  74 57           je short oC4PE.004858DF
00485888       |.  8B7424 10       mov esi,dword ptr ss:[esp+10]         ;  序列号第6位前面运算结果
0048588C       |.  8B5424 08       mov edx,dword ptr ss:[esp+8]          ;  序列号中间4位的第2位前面运算结果
00485890       |.  C1E6 04         shl esi,4
00485893       |.  03F2            add esi,edx
00485895       |.  56              push esi
00485896       |.  57              push edi
00485897       |.  E8 77000000     call oC4PE.00485913                   ;  序列号检查函数4-2
0048589C       |.  0FBE4F 10       movsx ecx,byte ptr ds:[edi+10]        ;  序列号后8位的第6位
004858A0       |.  25 FF000000     and eax,0FF
004858A5       |.  83C4 08         add esp,8
004858A8       |.  3BC8            cmp ecx,eax                           ;  序列号后8位的第6位必须与序列号检查函数4-2结果相等
004858AA       |.  75 33           jnz short oC4PE.004858DF
004858AC       |.  81C6 38010000   add esi,138
004858B2       |.  56              push esi
004858B3       |.  E8 2D000000     call oC4PE.004858E5                   ;  序列号检查函数4-3
004858B8       |.  83C4 04         add esp,4
004858BB       |.  84C0            test al,al
004858BD       |.  74 20           je short oC4PE.004858DF
004858BF       |.  8B4424 18       mov eax,dword ptr ss:[esp+18]
004858C3       |.  85C0            test eax,eax
004858C5       |.  7E 0C           jle short oC4PE.004858D3
004858C7       |.  81FE 90010000   cmp esi,190
004858CD       |.  7C 10           jl short oC4PE.004858DF
004858CF       |.  3BF0            cmp esi,eax
004858D1       |.  7F 0C           jg short oC4PE.004858DF
004858D3       |>  8B5424 14       mov edx,dword ptr ss:[esp+14]
004858D7       |.  5F              pop edi
004858D8       |.  B0 01           mov al,1
004858DA       |.  8932            mov dword ptr ds:[edx],esi
004858DC       |.  5E              pop esi
004858DD       |.  59              pop ecx
004858DE       |.  C3              retn
004858DF       |>  5F              pop edi
004858E0       |.  32C0            xor al,al
004858E2       |.  5E              pop esi
004858E3       |.  59              pop ecx
004858E4       \.  C3              retn
```

其中“序列号检查函数4-1”分析如下：

## 序列号检查函数4-1

```
004851A4       /$  8B4424 04       mov eax,dword ptr ss:[esp+4]
004851A8       |.  83F8 32         cmp eax,32
004851AB       |.  7C 11           jl short oC4PE.004851BE
004851AD       |.  83F8 39         cmp eax,39
004851B0       |.  7F 0C           jg short oC4PE.004851BE
004851B2       |.  8B4C24 08       mov ecx,dword ptr ss:[esp+8]
004851B6       |.  83C0 CE         add eax,-32
004851B9       |.  8901            mov dword ptr ds:[ecx],eax
004851BB       |.  B0 01           mov al,1
004851BD       |.  C3              retn
004851BE       |>  83F8 41         cmp eax,41
004851C1       |.  7C 11           jl short oC4PE.004851D4
004851C3       |.  83F8 48         cmp eax,48
004851C6       |.  7F 0C           jg short oC4PE.004851D4
004851C8       |.  8B5424 08       mov edx,dword ptr ss:[esp+8]
004851CC       |.  83C0 C7         add eax,-39
004851CF       |.  8902            mov dword ptr ds:[edx],eax
004851D1       |.  B0 01           mov al,1
004851D3       |.  C3              retn
004851D4       |>  32C0            xor al,al
004851D6       \.  C3              retn
```

可以看出要求被检查的序列号必须满足在`0x32`到`0x39`之间或`0x41`到`0x48`之间，并将之间取`0x32`或`0x39`；

为了方便取注册码中间4位的第2位以及注册码第6位都为`0x41`（A），在`004858A8`处可以确定序列号后8位的第6位为`0x4E`（N），

这样注册码修改为`GP3EHA78-1A34-12B45N7A`；由于注册码中间4位的第2位以及注册码第6位取得比较合理，序列号检查函数4-3能够满足条件，

（说明：其实这里我费了很多功夫的，但要说明很麻烦，我就省略了，有兴趣的可以试一试了）

下面接着分析：

## 后续检查代码

```
0051411F       |.  83C4 0C         add esp,0C
00514122       |.  84C0            test al,al
00514124       |.  0F84 BF000000   je oC4PE.005141E9
0051412A       |.  56              push esi
0051412B       |.  E8 331AF7FF     call oC4PE.00485B63                   ;   序列号检查函数五
```

其中“序列号检查函数五”分析如下：

## 序列号检查函数五

```
00485B63       /$  51              push ecx
00485B64       |.  56              push esi
00485B65       |.  8B7424 0C       mov esi,dword ptr ss:[esp+C]
00485B69       |.  8D4424 04       lea eax,dword ptr ss:[esp+4]
00485B6D       |.  0FBE4E 0B       movsx ecx,byte ptr ds:[esi+B]         ;  序列号中间4位的第3位
00485B71       |.  50              push eax
00485B72       |.  51              push ecx
00485B73       |.  E8 2CF6FFFF     call oC4PE.004851A4                   ;  序列号检查函数4-1
00485B78       |.  83C4 08         add esp,8
00485B7B       |.  84C0            test al,al
00485B7D       |.  0F84 82000000   je oC4PE.00485C05
00485B83       |.  0FBE46 12       movsx eax,byte ptr ds:[esi+12]        ;  序列号后8位的第4位
00485B87       |.  8D5424 0C       lea edx,dword ptr ss:[esp+C]
00485B8B       |.  52              push edx
00485B8C       |.  50              push eax
00485B8D       |.  E8 12F6FFFF     call oC4PE.004851A4                   ;  序列号检查函数4-1
00485B92       |.  83C4 08         add esp,8
00485B95       |.  84C0            test al,al
00485B97       |.  74 6C           je short oC4PE.00485C05
00485B99       |.  56              push esi
00485B9A       |.  E8 6B000000     call oC4PE.00485C0A
00485B9F       |.  0FBE4E 07       movsx ecx,byte ptr ds:[esi+7]         ;  序列号的第8位
00485BA3       |.  25 FF000000     and eax,0FF
00485BA8       |.  83C4 04         add esp,4
00485BAB       |.  3BC8            cmp ecx,eax                           ;  序列号第8位必须与前面运算结果相等
00485BAD       |.  75 56           jnz short oC4PE.00485C05
00485BAF       |.  8B4C24 0C       mov ecx,dword ptr ss:[esp+C]          ;  序列号后8位的第4位运算结果
00485BB3       |.  8B5424 04       mov edx,dword ptr ss:[esp+4]          ;  序列号中间4位的第3位运算结果
00485BB7       |.  C1E1 04         shl ecx,4
00485BBA       |.  03CA            add ecx,edx
00485BBC       |.  BE 13000000     mov esi,13
00485BC1       |.  8BC1            mov eax,ecx
00485BC3       |.  99              cdq
00485BC4       |.  F7FE            idiv esi
00485BC6       |.  85D2            test edx,edx
00485BC8       |.  75 3B           jnz short oC4PE.00485C05
00485BCA       |.  B8 F31ACA6B     mov eax,6BCA1AF3
00485BCF       |.  F7E9            imul ecx
00485BD1       |.  C1FA 03         sar edx,3
00485BD4       |.  8BC2            mov eax,edx
00485BD6       |.  C1E8 1F         shr eax,1F
00485BD9       |.  03D0            add edx,eax
00485BDB       |.  83FA 03         cmp edx,3
00485BDE       |.  7C 16           jl short oC4PE.00485BF6
00485BE0       |.  83FA 08         cmp edx,8
00485BE3       |.  7F 11           jg short oC4PE.00485BF6
00485BE5       |.  B8 01000000     mov eax,1
00485BEA       |.  33C9            xor ecx,ecx
00485BEC       |.  3BC0            cmp eax,eax
00485BEE       |.  5E              pop esi
00485BEF       |.  0F94C1          sete cl
00485BF2       |.  8AC1            mov al,cl
00485BF4       |.  59              pop ecx
00485BF5       |.  C3              retn
00485BF6       |>  33C0            xor eax,eax
00485BF8       |.  33C9            xor ecx,ecx
00485BFA       |.  83F8 01         cmp eax,1
00485BFD       |.  5E              pop esi
00485BFE       |.  0F94C1          sete cl
00485C01       |.  8AC1            mov al,cl
00485C03       |.  59              pop ecx
00485C04       |.  C3              retn
00485C05       |>  32C0            xor al,al
00485C07       |.  5E              pop esi
```
