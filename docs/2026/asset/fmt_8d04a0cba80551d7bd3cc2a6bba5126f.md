
```
  0040E33B         8D45 FC           lea eax,dword ptr ss:[ebp-4]
  0040E33E         BA 02000000       mov edx,2
  0040E343         E8 98E20F00       call **专家.0050C5E0
  0040E348         58                pop eax
  0040E349         66:C745 E0 2C00   mov word ptr ss:[ebp-20],2C
  0040E34F         FF45 EC           inc dword ptr ss:[ebp-14]
  0040E352         8B55 D0           mov edx,dword ptr ss:[ebp-30]
  0040E355         64:8915 00000000  mov dword ptr fs:[0],edx
  0040E35C         5E                pop esi
  0040E35D         5B                pop ebx
  0040E35E         8BE5              mov esp,ebp
  0040E360         5D                pop ebp
  0040E361         C3                retn
```

总结一下上面这个循环的作用是:对已经做过位交换处理的试验码，对其前20位重新计算覆盖，生成新的字符串，设他为`str`

具体算法看注册机。

我们继续分析下一段：

```
  004431C7         83C4 0C           add esp,0C
  004431CA         8D55 DC           lea edx,dword ptr ss:[ebp-24]
  004431CD         8D45 FC           lea eax,dword ptr ss:[ebp-4]
  004431D0         E8 3B940C00       call **专家.0050C610
  004431D5         FF8D 50FFFFFF     dec dword ptr ss:[ebp-B0]
  004431DB         8D45 DC           lea eax,dword ptr ss:[ebp-24]
  004431DE         BA 02000000       mov edx,2
  004431E3         E8 F8930C00       call **专家.0050C5E0
  004431E8         6A 14             push 14
  004431EA         6A 00             push 0
  004431EC         8D8D F0FEFFFF     lea ecx,dword ptr ss:[ebp-110]
  004431F2         51                push ecx
  004431F3         E8 A0A90B00       call **专家.004FDB98
  004431F8         83C4 0C           add esp,0C
  004431FB         33FF              xor edi,edi
  004431FD         6A 14             push 14
  004431FF         6A 00             push 0
  00443201         8D85 08FFFFFF     lea eax,dword ptr ss:[ebp-F8]
  00443207         50                push eax
  00443208         E8 8BA90B00       call **专家.004FDB98
  0044320D         83C4 0C           add esp,0C
  00443210         8D95 C4FEFFFF     lea edx,dword ptr ss:[ebp-13C]
  00443216         6A 14             push 14
  00443218         6A 00             push 0
  0044321A         52                push edx
  0044321B         E8 78A90B00       call **专家.004FDB98
  00443220         83C4 0C           add esp,0C
  00443223         66:C785 44FFFFFF >mov word ptr ss:[ebp-BC],80
  0044322C         8D45 D8           lea eax,dword ptr ss:[ebp-28]
  0044322F         E8 38E6FBFF       call **专家.0040186C
  00443234         8BD0              mov edx,eax
  00443236         FF85 50FFFFFF     inc dword ptr ss:[ebp-B0]
  0044323C         8B8D 30FFFFFF     mov ecx,dword ptr ss:[ebp-D0]
  00443242         8B81 DC020000     mov eax,dword ptr ds:[ecx+2DC]
  00443248         E8 FB7B0800       call **专家.004CAE48
  0044324D         8D45 D8           lea eax,dword ptr ss:[ebp-28]
  00443250         E8 EFE8FBFF       call **专家.00401B44             ; 取用户名
  00443255         57                push edi
  00443256         8BF8              mov edi,eax                    ; EDI=用户名
  00443258         33C0              xor eax,eax
  0044325A         83C9 FF           or ecx,FFFFFFFF
  0044325D         F2:AE             repne scas byte ptr es:[edi]
  0044325F         F7D1              not ecx                        ; ECX=注册名长度+1
  00443261         2BF9              sub edi,ecx
  00443263         8DB5 C4FEFFFF     lea esi,dword ptr ss:[ebp-13C]
  00443269         87F7              xchg edi,esi                   ; ESI=用户名
  0044326B         8BD1              mov edx,ecx                    ; EDX=ECX
  0044326D         8BC7              mov eax,edi
  0044326F         C1E9 02           shr ecx,2                      ; ECX=ECX DIV 4
  00443272         8D45 D8           lea eax,dword ptr ss:[ebp-28]
  00443275         F3:A5             rep movs dword ptr es:[edi],dw>
  00443277         8BCA              mov ecx,edx                    ; ECX=EDX
  00443279         BA 02000000       mov edx,2                      ; EDX=2
  0044327E         83E1 03           and ecx,3                      ; ECX=ECX AND 3
  00443281         F3:A4             rep movs byte ptr es:[edi],byt>; 用户名保存
  00443283         5F                pop edi
  00443284         FF8D 50FFFFFF     dec dword ptr ss:[ebp-B0]
  0044328A         E8 51930C00       call **专家.0050C5E0
  0044328F         C685 D7FEFFFF 00  mov byte ptr ss:[ebp-129],0
  00443296         8B85 30FFFFFF     mov eax,dword ptr ss:[ebp-D0]
  0044329C         05 00030000       add eax,300
```

这一段没什么好说的，略过，

继续下一段：

```
  0044329C         05 00030000       add eax,300
  004432A1         E8 9EE8FBFF       call **专家.00401B44             ; 注意这里EAX里面出来一组数据(后来发现
                                                                       ;这组数据是很早以前就算好了的，
                                                                        ;怎么算出来的，马上讨论)
  004432A6         57                push edi                           ; 设这组数据为s1(字符串数据)
  004432A7         8BF8              mov edi,eax                        ; EDI=s1
  004432A9         33C0              xor eax,eax                        ; EAX=0
  004432AB         83C9 FF           or ecx,FFFFFFFF
  004432AE         F2:AE             repne scas byte ptr es:[edi]
  004432B0         F7D1              not ecx                            ; ECX=s1长度+1
  004432B2         2BF9              sub edi,ecx
  004432B4         8DB5 F0FEFFFF     lea esi,dword ptr ss:[ebp-110]
  004432BA         87F7              xchg edi,esi                       ; ESI=s1
  004432BC         8BD1              mov edx,ecx                        ; EDX=ECX
  004432BE         8BC7              mov eax,edi                        ; EAX=EDI
  004432C0         C1E9 02           shr ecx,2                          ; ECX=ECX DIV 4
  004432C3         8D85 F0FEFFFF     lea eax,dword ptr ss:[ebp-110]
  004432C9         F3:A5             rep movs dword ptr es:[edi],dword >; EAX=s1
  004432CB         8BCA              mov ecx,edx                        ; ECX=EDX
  004432CD         8D95 C4FEFFFF     lea edx,dword ptr ss:[ebp-13C]     ; EDX=用户名
  004432D3         83E1 03           and ecx,3                          ; ECX=ECX AND 3
  004432D6         F3:A4             rep movs byte ptr es:[edi],byte pt>
  004432D8         5F                pop edi
  004432D9         8985 20FFFFFF     mov dword ptr ss:[ebp-E0],eax      ; 保存s1到[EBP-E0]
  004432DF         8995 24FFFFFF     mov dword ptr ss:[ebp-DC],edx      ; 保存用户名到[EBP-DC]
  004432E5         8DB5 08FFFFFF     lea esi,dword ptr ss:[ebp-F8]
  004432EB         33DB              xor ebx,ebx                        ; EBX=0,下面循环的计数器
  004432ED         8B8D 24FFFFFF     mov ecx,dword ptr ss:[ebp-DC]      ; ECX=注册名
  004432F3         8B95 20FFFFFF     mov edx,dword ptr ss:[ebp-E0]      ; EDX=s1
  004432F9         8A01              mov al,byte ptr ds:[ecx]           ; AL=按位取注册名
  004432FB         3202              xor al,byte ptr ds:[edx]           ; AL=AL XOR s1的指定位
  004432FD         83C4 F8           add esp,-8
  00443300         8806              mov byte ptr ds:[esi],al           ; 把al写入以esi开始的内存地址
  00443302         0FBE0E            movsx ecx,byte ptr ds:[esi]        ; ECX=AL
  00443305         898D A8FAFFFF     mov dword ptr ss:[ebp-558],ecx     ; [EBP-558]=ECX
  0044330B         DB85 A8FAFFFF     fild dword ptr ss:[ebp-558]        ; 装载ECX到ST0
  00443311         DD1C24            fstp qword ptr ss:[esp]
  00443314         E8 97190C00       call **专家.00504CB0
  00443319         83C4 08           add esp,8
  0044331C         899D A4FAFFFF     mov dword ptr ss:[ebp-55C],ebx
  00443322         DB85 A4FAFFFF     fild dword ptr ss:[ebp-55C]        ; 压入计数器
  00443328         DEC9              fmulp st(1),st                     ; ST0=ST0*ST1=ECX*计数器
  0044332A         89BD A0FAFFFF     mov dword ptr ss:[ebp-560],edi     ; [EBP-560]=EDI
  00443330         DB85 A0FAFFFF     fild dword ptr ss:[ebp-560]        ; 压入[EBP-560]
```
