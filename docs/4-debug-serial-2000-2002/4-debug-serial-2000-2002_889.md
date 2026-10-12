
# crackme

> 原文：[`www.pediy.com/kssd/pediy04/kanxue147.htm`](https://www.pediy.com/kssd/pediy04/kanxue147.htm)

一个 `crackme` 的破解

作者：未注册

下载：`http://person.longcity.net/home3/fpxfpx/crackme/u-x/vcrkme01.zip`

看了教程，找一个 `crackme` 来练手。

这个简单，适合我等菜鸟选手，用 `trw` 一路跟过来，就能注册。用 `unregistered` 来注册，注册码 `20012001`

\* Reference To: `USER32.GetDlgItemTextA`, Ord:`0104h`

```
 :0040121D 8B35C0504000              mov esi, dword ptr [004050C0]
 :00401223 68FF000000                push 000000FF
 :00401228 6830694000                push 00406930
 :0040122D 68E8030000                push 000003E8
 :00401232 50                        push eax
 :00401233 FFD6                      call esi                      ;读 unregistered
 :00401235 8B0D28694000              mov ecx, dword ptr [00406928]
 :0040123B 68FF000000                push 000000FF
 :00401240 68306A4000                push 00406A30
 :00401245 68EA030000                push 000003EA
 :0040124A 51                        push ecx
 :0040124B FFD6                      call esi                      ;读 20012001
 :0040124D 68306A4000                push 00406A30
 :00401252 6830694000                push 00406930
 :00401257 E8A4FDFFFF                call 00401000                 ;计算验证注册码
 :0040125C 83C408                    add esp, 00000008
 :0040125F 83F801                    cmp eax, 00000001             ;eax 应该是 1，不是 1 就失败，没有失败提示
 :00401262 A3646C4000                mov dword ptr [00406C64], eax
 :00401267 7565                      jne 004012CE
 :00401269 8B1528694000              mov edx, dword ptr [00406928]
 :0040126F 6A40                      push 00000040
```

\* Possible StringData Ref from Data Obj -> `"GOOD JOB! - CRACKED!"`

```
 :00401271 6880604000                push 00406080
```

\* Possible StringData Ref from Data Obj -> `"Send your solution to : v0id2k1@hotmail.com "`

```
 :00401276 6850604000                push 00406050
 :0040127B 52                        push edx
```

\* Reference To: `USER32.MessageBoxA`, Ord:`01BEh`

```
 :0040127C FF15C4504000              Call dword ptr [004050C4]
 :00401282 B801000000                mov eax, 00000001
 :00401287 5E                        pop esi
 :00401288 C21000                    ret 0010
```

注册码这样计算：

`+++++++++++++++++++ ASSEMBLY CODE LISTING ++++++++++++++++++`

```
//************ Start of Code in Object .text ************

Program Entry Point = 0040147E (vcrkme01.exe File Offset:0000647E)

:00401000 53                          push ebx
:00401001 8B5C240C                    mov ebx, dword ptr [esp+0C]
:00401005 55                          push ebp
:00401006 56                          push esi
:00401007 8B742410                    mov esi, dword ptr [esp+10]
:0040100B 8A0B                        mov cl, byte ptr [ebx]      ;取注册码第一位
:0040100D 33ED                        xor ebp, ebp                ;ebp=0
:0040100F 57                          push edi
:00401010 8A06                        mov al, byte ptr [esi]      ;取姓名第一位
:00401012 3AC1                        cmp al, cl                  ;比较，不一致失败
:00401014 0F8569010000                jne 00401183
:0040101A 8BFE                        mov edi, esi
:0040101C 83C9FF                      or ecx, FFFFFFFF
:0040101F 33C0                        xor eax, eax
:00401021 F2                          repnz
:00401022 AE                          scasb
:00401023 F7D1                        not ecx
:00401025 49                          dec ecx
:00401026 83F905                      cmp ecx, 00000005           ;姓名不能少于 5 个字母
:00401029 0F8254010000                jb 00401183
:0040102F 807B012D                    cmp byte ptr [ebx+01], 2D   ;注册码第二位要是‘-’
:00401033 0F854A010000                jne 00401183
:00401039 8BFE                        mov edi, esi
:0040103B 83C9FF                      or ecx, FFFFFFFF
:0040103E 33C0                        xor eax, eax
:00401040 33D2                        xor edx, edx
:00401042 F2                          repnz
:00401043 AE                          scasb
:00401044 F7D1                        not ecx
:00401046 49                          dec ecx
:00401047 7417                        je 00401060
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address: `:0040105E(C)`

```
:00401049 0FBE0C32                    movsx ecx, byte ptr [edx+esi]
:0040104D 03E9                        add ebp, ecx                ;累加 unregsitered 到 ebp
:0040104F 8BFE                        mov edi, esi
:00401051 83C9FF                      or ecx, FFFFFFFF
:00401054 33C0                        xor eax, eax
:00401056 42                          inc edx
:00401057 F2                          repnz
:00401058 AE                          scasb
:00401059 F7D1                        not ecx
:0040105B 49                          dec ecx
:0040105C 3BD1                        cmp edx, ecx
:0040105E 72E9                        jb 00401049
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address: `:00401047(C)`

```
:00401060 81C564600000                add ebp, 00006064           ;ebp=ebp+6064h=06575h
:00401066 55                          push ebp
```

\* Possible StringData Ref from Data Obj -> `"%lu"`

```
:00401067 6834604000                push 00406034
:0040106C 68306B4000                push 00406B30                 ;变成字符串 25973(06575h=25973)，到这里
:00401071 E8B6030000                call 0040142C
:00401076 8A16                        mov dl, byte ptr [esi]
:00401078 8BFE                        mov edi, esi
:0040107A 83C9FF                      or ecx, FFFFFFFF
:0040107D 33C0                        xor eax, eax
:0040107F 8815446B4000                mov byte ptr [00406B44], dl   ;[406b44]=u
:00401085 C605456B40002D              mov byte ptr [00406B45], 2D   ;[406b45]=-
:0040108C F2                          repnz
:0040108D AE                          scasb
:0040108E F7D1                        not ecx
:00401090 49                          dec ecx
:00401091 0FBE4431FF                  movsx eax, byte ptr [ecx+esi-01] ;取最后一个字母 d
:00401096 50                          push eax
:00401097 E8C4020000                  call 00401360                 ;变换为大写
:0040109C A2466B4000                  mov byte ptr [00406B46], al   ;[406b46]=D
:004010A1 BF306B4000                  mov edi, 00406B30
:004010A6 83C9FF                      or ecx, FFFFFFFF
:004010A9 33C0                        xor eax, eax
:004010AB F2                          repnz
:004010AC AE                          scasb
:004010AD F7D1                        not ecx
:004010AF 2BF9                        sub edi, ecx
:004010B1 81C564600000                add ebp, 00006064           ;ebp=6575h+6064h=50649
:004010B7 8BF7                        mov esi, edi
:004010B9 8BD1                        mov edx, ecx
:004010BB BF446B4000                  mov edi, 00406B44
:004010C0 83C9FF                      or ecx, FFFFFFFF
:004010C3 F2                          repnz
:004010C4 AE                          scasb
:004010C5 8BCA                        mov ecx, edx
:004010C7 4F                          dec edi
:004010C8 C1E902                      shr ecx, 02
:004010CB F3                          repz
:004010CC A5                          movsd
:004010CD 8BCA                        mov ecx, edx
:004010CF 55                          push ebp
:004010D0 83E103                      and ecx, 00000003
```

\* Possible StringData Ref from Data Obj -> `"%lu"`

```
:004010D3 6834604000                push 00406034
:004010D8 F3                          repz
:004010D9 A4                          movsb
```

\* Possible StringData Ref from Data Obj -> `"--"`

```
:004010DA BF30604000                  mov edi, 00406030
:004010DF 83C9FF                      or ecx, FFFFFFFF
```



# 汇编代码分析

## 004010E2
```
:004010E2 F2                          repnz   
:004010E3 AE                          scasb   
:004010E4 F7D1                        not ecx   
:004010E6 2BF9                        sub edi, ecx   
:004010E8 68306B4000                  push 00406B30   
:004010ED 8BF7                        mov esi, edi   
:004010EF 8BD1                        mov edx, ecx   
:004010F1 BF446B4000                  mov edi, 00406B44   
:004010F6 83C9FF                      or ecx, FFFFFFFF   
:004010F9 F2                          repnz   
:004010FA AE                          scasb   
:004010FB 8BCA                        mov ecx, edx   
:004010FD 4F                          dec edi   
:004010FE C1E902                      shr ecx, 02   
:00401101 F3                          repz   
:00401102 A5                          movsd   
:00401103 8BCA                        mov ecx, edx   
:00401105 83E103                      and ecx, 00000003   
:00401108 F3                          repz   
:00401109 A4                          movsb   
:0040110A E81D030000                  call 0040142C      ;50649 变成字符串到 406b30   
:0040110F BF306B4000                  mov edi, 00406B30   
:00401114 83C9FF                      or ecx, FFFFFFFF   
:00401117 33C0                        xor eax, eax   
:00401119 83C41C                      add esp, 0000001C   
:0040111C F2                          repnz   
:0040111D AE                          scasb   
:0040111E F7D1                        not ecx   
:00401120 2BF9                        sub edi, ecx   
:00401122 8BF7                        mov esi, edi   
:00401124 8BD1                        mov edx, ecx   
:00401126 BF446B4000                  mov edi, 00406B44   
:0040112B 83C9FF                      or ecx, FFFFFFFF   
:0040112E F2                          repnz   
:0040112F AE                          scasb   
:00401130 8BCA                        mov ecx, edx   
:00401132 4F                          dec edi   
:00401133 C1E902                      shr ecx, 02   
:00401136 F3                          repz   
:00401137 A5                          movsd   
:00401138 8BCA                        mov ecx, edx   
:0040113A 8BC3                        mov eax, ebx   
:0040113C 83E103                      and ecx, 00000003   
:0040113F F3                          repz   
:00401140 A4                          movsb   
:00401141 BE446B4000                  mov esi, 00406B44      ;现在 esi=u-D25973-5064
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address: `:00401168(C)`

## 00401146
```
:00401146 8A10                        mov dl, byte ptr [eax]      ;eax=20012001,比较   
:00401148 8A1E                        mov bl, byte ptr [esi]   
:0040114A 8ACA                        mov cl, dl   
:0040114C 3AD3                        cmp dl, bl   
:0040114E 7525                        jne 00401175   
:00401150 84C9                        test cl, cl   
:00401152 7416                        je 0040116A   
:00401154 8A5001                      mov dl, byte ptr [eax+01]   
:00401157 8A5E01                      mov bl, byte ptr [esi+01]   
:0040115A 8ACA                        mov cl, dl   
:0040115C 3AD3                        cmp dl, bl   
:0040115E 7515                        jne 00401175   
:00401160 83C002                      add eax, 00000002   
:00401163 83C602                      add esi, 00000002   
:00401166 84C9                        test cl, cl   
:00401168 75DC                        jne 00401146
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address: `:00401152(C)`

## 0040116A
```
:0040116A 33C0                        xor eax, eax   
:0040116C 33D2                        xor edx, edx   
:0040116E 85C0                        test eax, eax   
:00401170 0F94C2                      sete dl   
:00401173 EB12                        jmp 00401187
```

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses: `:0040114E(C)`, `:0040115E(C)`

## 00401175
```
:00401175 1BC0                        sbb eax, eax   
:00401177 83D8FF                      sbb eax, FFFFFFFF   
:0040117A 33D2                        xor edx, edx   
:0040117C 85C0                        test eax, eax   
:0040117E 0F94C2                      sete dl   
:00401181 EB04                        jmp 00401187
```

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses: `:00401014(C)`, `:00401029(C)`, `:00401033(C)`

## 00401183
```
:00401183 8B542414                    mov edx, dword ptr [esp+14]
```

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses: `:00401173(U)`, `:00401181(U)`

## 00401187
```
:00401187 B940000000                  mov ecx, 00000040   
:0040118C 33C0                        xor eax, eax   
:0040118E BF446B4000                  mov edi, 00406B44   
:00401193 F3                          repz   
:00401194 AB                          stosd   
:00401195 5F                          pop edi   
:00401196 5E                          pop esi   
:00401197 5D                          pop ebp   
:00401198 8BC2                        mov eax, edx   
:0040119A 5B                          pop ebx   
:0040119B C3                          ret
```

正确的注册码是：`u-D25973-5064` `4dd466d0e1e0cc123afbe504da2462ca`
