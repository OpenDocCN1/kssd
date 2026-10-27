
```
004148DD  |.  8D4424 34   lea eax,dword ptr ss:[esp+34]
004148E1  |.  6A 04       push 4                              ;从第4位开始取
004148E3  |.  50          push eax                            ;返回地址
004148E4  |.  8D4C24 20   lea ecx,dword ptr ss:[esp+20]
004148E8  |.  C68424 B401>mov byte ptr ss:[esp+1B4],5
004148F0  |.  E8 030D0100 call <jmp.&MFC42.#4278>
004148F5  |.  8BE8        mov ebp,eax
004148F7  |.  6A 04       push 4                              ;取4位
004148F9  |.  8D4C24 14   lea ecx,dword ptr ss:[esp+14]
004148FD  |.  6A 00       push 0                              ;从第0位开始取
004148FF  |.  51          push ecx                            ;返回地址
00414900  |.  8D4C24 20   lea ecx,dword ptr ss:[esp+20]
00414904  |.  C68424 B401>mov byte ptr ss:[esp+1B4],6
0041490C  |.  E8 E70C0100 call <jmp.&MFC42.#4278>
00414911  |.  8B16        mov edx,dword ptr ds:[esi]                  ;  EDX="Y4E4"
00414913  |.  8B0F        mov ecx,dword ptr ds:[edi]                  ;  ECX="UPGU"
00414915  |.  8B00        mov eax,dword ptr ds:[eax]                  ;  EAX="0VY3"
00414917  |.  52          push edx                                    ;  edx入栈
00414918  |.  8B55 00     mov edx,dword ptr ss:[ebp]                  ;  edx="QM7E"
0041491B  |.  51          push ecx
0041491C  |.  52          push edx
0041491D  |.  50          push eax
0041491E  |.  8D8C24 C401>lea ecx,dword ptr ss:[esp+1C4]
00414925  |.  68 74844300 push Wordllk.00438474                       ;  ASCII "%s-%s-%s-%s"
0041492A  |.  51          push ecx                                    ;  返回值地址
0041492B  |.  C68424 C001>mov byte ptr ss:[esp+1C0],7
00414933  |.  E8 7E0C0100 call <jmp.&MFC42.#2818>                     ;  格式化字符串
00414938  |.  83C4 18     add esp,18                                  ;  ecx="0VY3-QM7E-UPGU-Y4E4"
0041493B  |.  8D4C24 10   lea ecx,dword ptr ss:[esp+10]
0041493F  |.  C68424 A801>mov byte ptr ss:[esp+1A8],6
```

下面是`0041464B  |.  E8 80E3FEFF call Wordllk.004029D0`

其实下面就是对DES密钥的处理，偶比较笨，一开始没看出来，所以就跟了一遍，看来得多学习才行啊，呵呵

```
004029D0  /$  81EC F40000>sub esp,0F4
004029D6  |.  33C9        xor ecx,ecx
004029D8  |.  53          push ebx
004029D9  |.  55          push ebp
004029DA  |.  56          push esi
004029DB  |.  8BB424 0401>mov esi,dword ptr ss:[esp+104]
004029E2  |.  57          push edi
004029E3  |>  33C0        /xor eax,eax
004029E5  |.  8A81 107443>|mov al,byte ptr ds:[ecx+437410]            ;  取表1
004029EB  |.  8BD0        |mov edx,eax
004029ED  |.  83E0 07     |and eax,7                                  ;  eax界于0到7之间
004029F0  |.  C1FA 03     |sar edx,3                                  ;  edx界于0到7之间
004029F3  |.  8A1C45 A073>|mov bl,byte ptr ds:[eax\*2+4373A0]          ;  取表2
004029FA  |.  8A1432      |mov dl,byte ptr ds:[edx+esi]               ;  取表3
004029FD  |.  84D3        |test bl,dl
004029FF  |.  0F95C0      |setne al
00402A02  |.  88440C 4C   |mov byte ptr ss:[esp+ecx+4C],al
00402A06  |.  41          |inc ecx
00402A07  |.  83F9 38     |cmp ecx,38
00402A0A  |.^ 7C D7       \jl short Wordllk.004029E3
00402A0C  |.  B8 1E000000 mov eax,1E
00402A11  |.  33D2        xor edx,edx
00402A13  |.  894424 10   mov dword ptr ss:[esp+10],eax
00402A17  |>  66:83BC24 0>/cmp word ptr ss:[esp+10C],1
00402A20  |.  74 03       |je short Wordllk.00402A25
00402A22  |.  8D0412      |lea eax,dword ptr ds:[edx+edx]
00402A25  |>  8DBC84 8800>|lea edi,dword ptr ss:[esp+eax\*4+88]
00402A2C  |.  8DAC84 8400>|lea ebp,dword ptr ss:[esp+eax\*4+84]
00402A33  |.  33C0        |xor eax,eax
00402A35  |.  33C9        |xor ecx,ecx
00402A37  |.  8A8A 487443>|mov cl,byte ptr ds:[edx+437448]
00402A3D  |.  C707 000000>|mov dword ptr ds:[edi],0
00402A43  |.  C745 00 000>|mov dword ptr ss:[ebp],0
00402A4A  |.  8BF1        |mov esi,ecx
00402A4C  |>  83F9 1C     |/cmp ecx,1C
00402A4F  |.  7D 06       ||jge short Wordllk.00402A57
00402A51  |.  8A5C0C 4C   ||mov bl,byte ptr ss:[esp+ecx+4C]           ;  取0,1表
00402A55  |.  EB 04       ||jmp short Wordllk.00402A5B
00402A57  |>  8A5C0C 30   ||mov bl,byte ptr ss:[esp+ecx+30]           ;  移动位置
00402A5B  |>  885C04 14   ||mov byte ptr ss:[esp+eax+14],bl
00402A5F  |.  40          ||inc eax
00402A60  |.  41          ||inc ecx
00402A61  |.  83F8 1C     ||cmp eax,1C
00402A64  |.^ 7C E6       |\jl short Wordllk.00402A4C
00402A66  |.  B8 1C000000 |mov eax,1C
00402A6B  |.  8D4E 1C     |lea ecx,dword ptr ds:[esi+1C]
00402A6E  |>  83F9 38     |/cmp ecx,38
00402A71  |.  7D 06       ||jge short Wordllk.00402A79
00402A73  |.  8A5C0C 4C   ||mov bl,byte ptr ss:[esp+ecx+4C]           ;  取0,1表
00402A77  |.  EB 04       ||jmp short Wordllk.00402A7D
00402A79  |>  8A5C0C 30   ||mov bl,byte ptr ss:[esp+ecx+30]
00402A7D  |>  885C04 14   ||mov byte ptr ss:[esp+eax+14],bl
00402A81  |.  40          ||inc eax
00402A82  |.  41          ||inc ecx
00402A83  |.  83F8 38     ||cmp eax,38
00402A86  |.^ 7C E6       |\jl short Wordllk.00402A6E
00402A88  |.  33C0        |xor eax,eax
00402A8A  |>  33C9        |/xor ecx,ecx
00402A8C  |.  8A88 587443>||mov cl,byte ptr ds:[eax+437458]
00402A92  |.  8A5C0C 14   ||mov bl,byte ptr ss:[esp+ecx+14]
00402A96  |.  84DB        ||test bl,bl
00402A98  |.  74 0F       ||je short Wordllk.00402AA9
00402A9A  |.  8B0C85 B073>||mov ecx,dword ptr ds:[eax\*4+4373B0]
00402AA1  |.  8B75 00     ||mov esi,dword ptr ss:[ebp]
00402AA4  |.  0BF1        ||or esi,ecx
00402AA6  |.  8975 00     ||mov dword ptr ss:[ebp],esi
00402AA9  |>  33C9        ||xor ecx,ecx
00402AAB  |.  8A88 707443>||mov cl,byte ptr ds:[eax+437470]
00402AB1  |.  8A5C0C 14   ||mov bl,byte ptr ss:[esp+ecx+14]
00402AB5  |.  84DB        ||test bl,bl
00402AB7  |.  74 0D       ||je short Wordllk.00402AC6
00402AB9  |.  8B0C85 B073>||mov ecx,dword ptr ds:[eax\*4+4373B0]
00402AC0  |.  8B37        ||mov esi,dword ptr ds:[edi]
00402AC2  |.  0BF1        ||or esi,ecx
00402AC4  |.  8937        ||mov dword ptr ds:[edi],esi
00402AC6  |>  40          ||inc eax
00402AC7  |.  83F8 18     ||cmp eax,18
00402ACA  |.^ 7C BE       |\jl short Wordllk.00402A8A
00402ACC  |.  8B4424 10   |mov eax,dword ptr ss:[esp+10]
00402AD0  |.  42          |inc edx
00402AD1  |.  83E8 02     |sub eax,2
00402AD4  |.  83F8 FE     |cmp eax,-2
00402AD7  |.  894424 10   |mov dword ptr ss:[esp+10],eax
00402ADB  |.^ 0F8F 36FFFF>\jg Wordllk.00402A17
00402AE1  |.  8D9424 8400>lea edx,dword ptr ss:[esp+84]
00402AE8  |.  52          push edx
00402AE9  |.  E8 12000000 call Wordllk.00402B00
00402AEE  |.  83C4 04     add esp,4
00402AF1  |.  5F          pop edi
00402AF2  |.  5E          pop esi
00402AF3  |.  5D          pop ebp
00402AF4  |.  5B          pop ebx
00402AF5  |.  81C4 F40000>add esp,0F4
00402AFB  \.  C3          retn
```

------------------------------------------------------------------------

## 破解总结

算法不难，偶比较笨，就用C来解释吧

1.

```
unsigned char name[9]="fenjianr",temp[8],str[9]="12345678";
int i;
for (i=0;i<8;i++) {
  temp[i]=name[8-i];
  temp[i]+=str[i];
}
```

2. 得出来的8字节`temp`的Hex形式为：`31A4A1959EA0A59D`，做为DES密钥，DES加密“abcdefgh12345678”，得到`串1`＝`"裓法文簶.c嚫焄."`，其Hex形式为`D15CB7A8CEC4BA982E6387B89F5B2EC5`

3. `Base64Encode(串1)＝0Vy3qM7EupguY4e4n1suxQ==`，转换为大写并取前16位用`-`连接即可

------------------------------------------------------------------------

## 版权声明

本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢! 2007年4月6日

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)
