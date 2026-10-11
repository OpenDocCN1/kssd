
### 汇编代码分析（循环部分）

```
0040E2DD     8BD3          mov edx,ebx               ; EDX=EBX
0040E2DF     D1FA          sar edx,1                 ; EDX DIV 2
0040E2E1     79 03         jns short 专家.0040E2E6    ; EDX=0 就跳
0040E2E3     83D2 00       adc edx,0
0040E2E6     03C2          add eax,edx               ; EAX=EAX+EDX
0040E2E8     43            inc ebx                 ; EBX+1
0040E2E9     83C0 09       add eax,9                 ; EAX+9
0040E2EC     46            inc esi                 ; ESI+1
0040E2ED     83FB 28       cmp ebx,28                ; 是否遍历结束
0040E2F0     888415 6CFFFFFF  mov byte ptr ss:[ebp+edx-94],a; 交换后的字符串从第 1 位开始用 AL 覆盖
0040E2F7     7C 92         jl short 专家.0040E28B    ;
0040E2F9     C645 80 00    mov byte ptr ss:[ebp-80],0   ; 上面结果的下一位=0
0040E2FD     66:C745 E0 2C00 mov word ptr ss:[ebp-20],2C
0040E303     8D95 6CFFFFFF lea edx,dword ptr ss:[ebp-94]  ; EDX=上面的结果字符串的首指针
0040E309     8D45 F4       lea eax,dword ptr ss:[ebp-C]
0040E30C     E8 97E00F00   call 专家.0050C3A8
0040E311     8BD0          mov edx,eax
0040E313     FF45 EC       inc dword ptr ss:[ebp-14]
0040E316     8B45 08       mov eax,dword ptr ss:[ebp+8]
0040E319     E8 F2E20F00   call 专家.0050C610
0040E31E     8B45 08       mov eax,dword ptr ss:[ebp+8]
0040E321     BA 02000000   mov edx,2
0040E326     66:C745 E0 3800 mov word ptr ss:[ebp-20],38
0040E32C     50            push eax
0040E32D     8D45 F4       lea eax,dword ptr ss:[ebp-C]
0040E330     FF4D EC       dec dword ptr ss:[ebp-14]
0040E333     E8 A8E20F00   call 专家.0050C5E0
0040E338     FF4D EC       dec dword ptr ss:[ebp-14]
0040E33B     8D45 FC       lea eax,dword ptr ss:[ebp-4]
0040E33E     BA 02000000   mov edx,2
0040E343     E8 98E20F00   call 专家.0050C5E0
0040E348     58            pop eax
0040E349     66:C745 E0 2C00 mov word ptr ss:[ebp-20],2C
0040E34F     FF45 EC       inc dword ptr ss:[ebp-14]
0040E352     8B55 D0       mov edx,dword ptr ss:[ebp-30]
0040E355     64:8915 00000000 mov dword ptr fs:[0],edx
0040E35C     5E            pop esi
0040E35D     5B            pop ebx
0040E35E     8BE5          mov esp,ebp
0040E360     5D            pop ebp
0040E361     C3            retn
```

总结一下上面这个循环的作用是：对已经做过位交换处理的试验码，对其前 20 位重新计算覆盖，生成新的字符串，设他为 `str`。

具体算法看注册机。

### 汇编代码分析（后续段落）

```
004431C7     83C4 0C       add esp,0C
004431CA     8D55 DC       lea edx,dword ptr ss:[ebp-24]
004431CD     8D45 FC       lea eax,dword ptr ss:[ebp-4]
004431D0     E8 3B940C00   call 专家.0050C610
004431D5     FF8D 50FFFFFF dec dword ptr ss:[ebp-B0]
004431DB     8D45 DC       lea eax,dword ptr ss:[ebp-24]
004431DE     BA 02000000   mov edx,2
004431E3     E8 F8930C00   call 专家.0050C5E0
004431E8     6A 14         push 14
004431EA     6A 00         push 0
004431EC     8D8D F0FEFFFF lea ecx,dword ptr ss:[ebp-110]
004431F2     51            push ecx
004431F3     E8 A0A90B00   call 专家.004FDB98
004431F8     83C4 0C       add esp,0C
004431FB     33FF          xor edi,edi
004431FD     6A 14         push 14
004431FF     6A 00         push 0
00443201     8D85 08FFFFFF lea eax,dword ptr ss:[ebp-F8]
00443207     50            push eax
00443208     E8 8BA90B00   call 专家.004FDB98
0044320D     83C4 0C       add esp,0C
00443210     8D95 C4FEFFFF lea edx,dword ptr ss:[ebp-13C]
00443216     6A 14         push 14
00443218     6A 00         push 0
0044321A     52            push edx
0044321B     E8 78A90B00   call 专家.004FDB98
00443220     83C4 0C       add esp,0C
00443223     66:C785 44FFFFFF mov word ptr ss:[ebp-BC],80
0044322C     8D45 D8       lea eax,dword ptr ss:[ebp-28]
0044322F     E8 38E6FBFF   call 专家.0040186C
00443234     8BD0          mov edx,eax
00443236     FF85 50FFFFFF inc dword ptr ss:[ebp-B0]
0044323C     8B8D 30FFFFFF mov ecx,dword ptr ss:[ebp-D0]
00443242     8B81 DC020000 mov eax,dword ptr ds:[ecx+2DC]
00443248     E8 FB7B0800   call 专家.004CAE48
0044324D     8D45 D8       lea eax,dword ptr ss:[ebp-28]
00443250     E8 EFE8FBFF   call 专家.00401B44           ; 取用户名
00443255     57            push edi
00443256     8BF8          mov edi,eax               ; EDI=用户名
00443258     33C0          xor eax,eax
0044325A     83C9 FF       or ecx,FFFFFFFF
0044325D     F2:AE         repne scas byte ptr es:[edi]
0044325F     F7D1          not ecx                   ; ECX=注册名长度+1
00443261     2BF9          sub edi,ecx
00443263     8DB5 C4FEFFFF lea esi,dword ptr ss:[ebp-13C]
00443269     87F7          xchg edi,esi              ; ESI=用户名
0044326B     8BD1          mov edx,ecx               ; EDX=ECX
0044326D     8BC7          mov eax,edi
0044326F     C1E9 02       shr ecx,2                 ; ECX=ECX DIV 4
00443272     8D45 D8       lea eax,dword ptr ss:[ebp-28]
00443275     F3:A5         rep movs dword ptr es:[edi],dw
00443277     8BCA          mov ecx,edx               ; ECX=EDX
00443279     BA 02000000   mov edx,2                 ; EDX=2
0044327E     83E1 03       and ecx,3                 ; ECX=ECX AND 3
00443281     F3:A4         rep movs byte ptr es:[edi],byt; 用户名保存
00443283     5F            pop edi
00443284     FF8D 50FFFFFF dec dword ptr ss:[ebp-B0]
0044328A     E8 51930C00   call 专家.0050C5E0
0044328F     C685 D7FEFFFF 00 mov byte ptr ss:[ebp-129],0
00443296     8B85 30FFFFFF mov eax,dword ptr ss:[ebp-D0]
0044329C     05 00030000   add eax,300
```

这一段没什么好说的，略过。

### 汇编代码分析（数据获取部分）

```
0044329C     05 00030000   add eax,300
004432A1     E8 9EE8FBFF   call 专家.00401B44           ; 注意这里 EAX 里面出来一组数据(后来发现
                                                        ; 这组数据是很早以前就算好了的，
                                                        ; 怎么算出来的，马上讨论)
004432A6     57            push edi                    ; 设这组数据为 s1(字符串数据)
004432A7     8BF8          mov edi,eax                 ; EDI=s1
004432A9     33C0          xor eax,eax                 ; EAX=0
004432AB     83C9 FF       or ecx,FFFFFFFF
004432AE     F2:AE         repne scas byte ptr es:[edi]
004432B0     F7D1          not ecx                     ; ECX=s1 长度+1
004432B2     2BF9          sub edi,ecx
004432B4     8DB5 F0FEFFFF lea esi,dword ptr ss:[ebp-110]
```
