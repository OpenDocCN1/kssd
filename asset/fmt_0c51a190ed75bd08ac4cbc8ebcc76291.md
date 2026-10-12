
`Base64Decode`、`Base64Encode`、`BlowFishDecrypt`、`BlowFishEncrypt`、`CRC32`、`CRCFileCheck`、`DecryptStringFun1`、`DecryptStringFun2`、`DesDecrypt`、`DesEncrypt`、`EncryptStringFun1`、`EncryptStringFun2`、`FileDecrypt`、`FileEncrypt`、`GetDllVersion`、`GetHardDiskId`、`GetMainBoardId`、`MD5Encrypt`、`RSADecrypt`、`RSAEncrypt`、`SHAEncrypt`、`Secret16Encrypt`。

调用完`reg.dll`还不忘释放：

```
004CF2D3   |> \56                 push esi                      ; /hLibModule
004CF2D4   |.  E8 BB7AF3FF        call <jmp.&KERNEL32.FreeLibrary>         ; \FreeLibrary
```

程序还带了个`dll`：`D:\WINNT\system32\smartread.dll`，可能用来取得机器码。

### 5. 加密解析

2 次加密，第一次对输入的注册码，第二次对机器码，跟踪过第二次加密的值与输入的注册码无关，是定值，说明第二次加密的跟输入的注册码无关。因此从第二次加密入手。

### 5.1 第二次加密：对机器码的加密

```
004CF2A5   |.  50                 push eax
004CF2A6   |.  FF55 F8            call dword ptr ss:[ebp-8]        ;  堆栈 ss:[0012FCB0]=10010880
```

入栈参数：

```
0012FB90    00F55E64    ASCII "0604C-5FBB0"
0012FB94    004CF520    ASCII "sbipxa"

100106C3     E8 78CAFFFF   call 1000D140
```

这是对`sbipxa`下硬件访问断点找到的，跨过该`call`后得到 32 个字符，猜测`MD5`散列，试了下果然是。

`MD5("sbipxa") = "61146E463A46267B3489D53D8E96BB96"`

取前 8 位＝`61146E46`，如果有`F`就替换为`8`。

```
100109C6     83F8 08          cmp eax,8
100109C9     7D 0F            jge short 100109DA
100109CB     807C05 E4 46    cmp byte ptr ss:[ebp+eax-1C],46
100109D0     75 05           jnz short 100109D7
100109D2     C64405 E4 38    mov byte ptr ss:[ebp+eax-1C],38
100109D7     40             inc eax
100109D8   ^ EB EC          jmp short 100109C6
```

对`61146E46`的填充到 12 位：

由上面返回来到的，填充结果：`0012ACA0 01BE0D6C "61146E466114"`

```
10001303     8A0C0E          mov cl,byte ptr ds:[esi+ecx]
10001306     8D4424 24       lea eax,dword ptr ss:[esp+24]
1000130A     884C24 24       mov byte ptr ss:[esp+24],cl
1000130E     50             push eax
1000130F     6A 01          push 1
10001311     8D4C24 18       lea ecx,dword ptr ss:[esp+18]
10001315     E8 76080000     call 10001B90
1000131A     8B4C24 10       mov ecx,dword ptr ss:[esp+10]
1000131E     46             inc esi
1000131F     8379 F8 0C      cmp dword ptr ds:[ecx-8],0C
10001323   ^ 7C DE          jl short 10001303
```

逆序：`0012FB0C 34 31 31 36 36 34 45 36 34 31 31 36 411664E64116`

```
1000132F     8B7B 08        mov edi,dword ptr ds:[ebx+8]
10001332     C1E7 08        shl edi,8
10001335     897B 08        mov dword ptr ds:[ebx+8],edi
10001338     8BD7          mov edx,edi
1000133A     0FBE48 FC      movsx ecx,byte ptr ds:[eax-4]
1000133E     0BCA          or ecx,edx
10001340     8B53 0C        mov edx,dword ptr ds:[ebx+C]
10001343     C1E2 08        shl edx,8
10001346     894B 08        mov dword ptr ds:[ebx+8],ecx
10001349     8953 0C        mov dword ptr ds:[ebx+C],edx
1000134C     8BF2          mov esi,edx
1000134E     0FBE10        movsx edx,byte ptr ds:[eax]
10001351     0BD6          or edx,esi
10001353     8B73 10        mov esi,dword ptr ds:[ebx+10]
10001356     C1E6 08        shl esi,8
10001359     8953 0C        mov dword ptr ds:[ebx+C],edx
1000135C     8973 10        mov dword ptr ds:[ebx+10],esi
1000135F     8BFE          mov edi,esi
10001361     0FBE70 04      movsx esi,byte ptr ds:[eax+4]
10001365     0BF7          or esi,edi
10001367     40            inc eax
10001368     8973 10        mov dword ptr ds:[ebx+10],esi
1000136B     8D3C28         lea edi,dword ptr ds:[eax+ebp]
1000136E     83FF 04        cmp edi,4
10001371   ^ 7C BC          jl short 1000132F
```

对`411664E64116`循环计算，结果与机器码`0604C-5FBB0`异或：

`0012FB0C 34 31 31 36 36 34 45 36 34 31 31 36 411664E64116`

```
A           B           C

100014D2     8B5C24 10      mov ebx,dword ptr ss:[esp+10]
100014D6     8B41 08         mov eax,dword ptr ds:[ecx+8]     ; 取出 A
100014D9     A8 01          test al,1
100014DB     74 42          je short 1000151F
100014DD     8B79 14         mov edi,dword ptr ds:[ecx+14]    ; 80000062
100014E0     8B59 2C         mov ebx,dword ptr ds:[ecx+2C]    ; 80000000
100014E3     33F8           xor edi,eax                      ; A=80000062 XOR A
100014E5     8B41 0C         mov eax,dword ptr ds:[ecx+C]     ; 取出 B
100014E8     D1EF          shr edi,1                        ; A/2
100014EA     0BFB          or edi,ebx                       ; OR 80000000
100014EC     A8 01          test al,1
100014EE     8979 08         mov dword ptr ds:[ecx+8],edi     ; A 保存回原地址 12FB0C
100014F1     74 1A          je short 1000150D
100014F3     8B79 18         mov edi,dword ptr ds:[ecx+18]    ; 40000020
100014F6     BB 01000000     mov ebx,1
100014FB     33F8          xor edi,eax                      ; B XOR 40000020
100014FD     8B41 30         mov eax,dword ptr ds:[ecx+30]    ; C0000000
10001500     D1EF          shr edi,1                        ; B/2
10001502     0BF8          or edi,eax                       ; B OR C0000000
10001504     895C24 10      mov dword ptr ss:[esp+10],ebx    ; 赋值 ebx＝1
10001508     8979 0C         mov dword ptr ds:[ecx+C],edi     ; B 保存回原地址 12FB10
1000150B     EB 45          jmp short 10001552
1000150D     8B79 24         mov edi,dword ptr ds:[ecx+24]    ; 3FFFFFFF
10001510     D1E8          shr eax,1                        ; B/2
10001512     23C7          and eax,edi                      ; AND 3FFFFFFF
10001514     33DB          xor ebx,ebx                      ; 赋值 ebx＝0
10001516     8941 0C         mov dword ptr ds:[ecx+C],eax     ; 保存回原地址 12FB10
10001519     895C24 10      mov dword ptr ss:[esp+10],ebx
1000151D     EB 33          jmp short 10001552
1000151F     8B79 20         mov edi,dword ptr ds:[ecx+20]    ; 7FFFFFFF
10001522     D1E8          shr eax,1                        ; A/2
10001524     23C7          and eax,edi                      ; AND 7FFFFFFF
10001526     8941 08         mov dword ptr ds:[ecx+8],eax     ; 保存回原地址 12FB0C
10001529     8B41 10         mov eax,dword ptr ds:[ecx+10]    ; 取出 C
1000152C     A8 01          test al,1
1000152E     74 16          je short 10001546
10001530     8B51 1C         mov edx,dword ptr ds:[ecx+1C]    ; 10000002
10001533     33D0          xor edx,eax                      ; C XOR 10000002
10001535     8B41 34         mov eax,dword ptr ds:[ecx+34]    ; F0000000
10001538     D1EA          shr edx,1                        ; C/2
1000153A     0BD0          or edx,eax                       ; OR F0000000
1000153C     8951 10         mov dword ptr ds:[ecx+10],edx    ; C 保存回原地址 12FB14
1000153F     BA 01000000     mov edx,1                      ; 赋值 edx＝1
10001544     EB 0C          jmp short 10001552
10001546     8B51 28         mov edx,dword ptr ds:[ecx+28]    ; 0FFFFFFF
10001549     D1E8          shr eax,1                        ; C/2
1000154B     23C2          and eax,edx                      ; AND 0FFFFFFF
1000154D     33D2          xor edx,edx                      ; 赋值 edx＝0
1000154F     8941 10         mov dword ptr ds:[ecx+10],eax    ; 保存回原地址 12FB14
10001552     8AC2           mov al,dl
10001554     32C3           xor al,bl
10001556     8A5C24 0F      mov bl,byte ptr ss:[esp+F]
1000155A     D0E3           shl bl,1
1000155C     0AC3           or al,bl
1000155E     4E             dec esi                          ; 循环指针
1000155F     884424 0F      mov byte ptr ss:[esp+F],al       ; 最后需要的值
10001563   ^ 0F85 69FFFFFF   jnz 100014D2
10001569     8B4C24 18      mov ecx,dword ptr ss:[esp+18]
1000156D     5F             pop edi
1000156E     5E             pop esi
1000156F     5B             pop ebx
10001570     8A11           mov dl,byte ptr ds:[ecx]         ; 12ACD0 保存了顺取机器码的一个字符
```
