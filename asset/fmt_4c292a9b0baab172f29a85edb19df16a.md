
```
004668B3 |. 85F6              test esi,esi
004668B5 |. 7C 22            jl short Five.004668D9
004668B7 |. 46               inc esi                     ;长度加 1，连长度一起做变换处理
004668B8 |. 8D4D 9B          lea ecx,dword ptr ss:[ebp-65] ;ECX 为注册名字符串长度及注册名字符串首地址
004668BB |. 8D9D D1FEFFFF    lea ebx,dword ptr ss:[ebp-12F] ;EBX 为存放解码后的数据存放首地址
004668C1 |> 33C0             xor eax,eax
004668C3 |. 8A01             |mov al,byte ptr ds:[ecx]      ;取一个字符，第一次为取长度
004668C5 |. BF 0A000000      |mov edi,0A                    ;EDI=10
004668CA |. 33D2             |xor edx,edx
004668CC |. F7F7             |div edi                       ;EAX/EDI（10），取余
004668CE |. 80C2 30          |add dl,30                     ;余数 DL=DL+30，即将余数转为 ASCII 码
004668D1 |. 42               |inc edx                       ;EDX+1
004668D2 |. 8813             |mov byte ptr ds:[ebx],dl      ;保存计算所得的数据
004668D4 |. 43               |inc ebx                       ;保存数据的地址加 1
004668D5 |. 41               |inc ecx                       ;注册名字符串地址加 1，取下一个字符
004668D6 |. 4E               |dec esi                       ;长度减 1
004668D7 |. 75 E8            \jnz short Five.004668C1
004668D9 |> 33C9             xor ecx,ecx
004668DB |. 8A4D 9B          mov cl,byte ptr ss:[ebp-65]    ;取注册名字符串长度
004668DE |. 83F9 01         cmp ecx,1                      ;做字符串长度检测
004668E1 |. 7C 1A            jl short Five.004668FD
004668E3 |. 8D840D D1FEFFFF lea eax,dword ptr ss:[ebp+ecx-12F] ;取刚处理好的数据，即对注册名字符进行二次变换处理，EAX 为数据的高地址
004668EA |> 8A55 9B         /mov dl,byte ptr ss:[ebp-65]   ;取长度
004668ED |. 2AD1            |sub dl,cl                     ;以固定长度减计数长度 DL-CL，CL=6，5，4，3，2，1
004668EF |. 03D2            |add edx,edx                   ;EDX+EDX 即（EDX-ECX）*2
004668F1 |. 8A18            |mov bl,byte ptr ds:[eax]      ;取一个字符到 BL
004668F3 |. 02D3            |add dl,bl                     ;DL+BL
004668F5 |. 8810            |mov byte ptr ds:[eax],dl      ;保存计算所得的数据
004668F7 |. 49              |dec ecx                       ;计数长度减 1
004668F8 |. 48              |dec eax                       ;数据地址减 1
004668F9 |. 85C9            |test ecx,ecx
004668FB |. 75 ED           \jnz short Five.004668EA
004668FD |> 8A45 9B         mov al,byte ptr ss:[ebp-65]    ;取注册名字符串长度
00466900 |. 3C 06           cmp al,6                       ;再做字符串长度检测
00466902 |. 73 30           jnb short Five.00466934        ;长度不小于 6，转
00466904 |. 33C9            xor ecx,ecx
00466906 |. 8AC8            mov cl,al
00466908 |. 41              inc ecx
00466909 |. 83F9 06        cmp ecx,6
0046690C |. 7F 26           jg short Five.00466934
0046690E |. 8D9C0D D0FEFFFF lea ebx,dword ptr ss:[ebp+ecx-130] ;这是字符串长度小于 6 的计算
00466915 |> 33C0           /xor eax,eax
00466917 |. 8A03           |mov al,byte ptr ds:[ebx]
00466919 |. 83C0 02        |add eax,2
0046691C |. 83E8 30        |sub eax,30
0046691F |. BE 0A000000    |mov esi,0A
00466924 |. 99             |cdq
00466925 |. F7FE           |idiv esi
00466927 |. 80C2 30        |add dl,30
0046692A |. 8853 01        |mov byte ptr ds:[ebx+1],dl
0046692D |. 41             |inc ecx
0046692E |. 43             |inc ebx
0046692F |. 83F9 07        |cmp ecx,7
00466932 |. 75 E1          \jnz short Five.00466915
00466934 |> B9 06000000    mov ecx,6                      ;置计数长度为 6
00466939 |. 8D85 D7FEFFFF   lea eax,dword ptr ss:[ebp-129]  ;取刚处理好的数据，即对注册名字符进行三次变换处理，EAX 为数据的高地址
0046693F |> B2 06          /mov dl,6                      ;固定长度
00466941 |. 2AD1            |sub dl,cl                     ;固定长度减计数长度 DL-CL，CL=6，5，4，3，2，1
00466943 |. 03D2            |add edx,edx                   ;EDX+EDX 即（EDX-ECX）*2
00466945 |. 8D1452          |lea edx,dword ptr ds:[edx+edx*2] ;EDX=EDX+EDX*2
00466948 |. 8A18            |mov bl,byte ptr ds:[eax]      ;取一个字符到 BL
0046694A |. 02D3            |add dl,bl                     ;DL+BL
0046694C |. 8810            |mov byte ptr ds:[eax],dl      ;存回原处
0046694E |. 8A10            |mov dl,byte ptr ds:[eax]
00466950 |. 80FA 7E         |cmp dl,7E                     ;将 DL+BL 和 7E（126）比较
00466953 |. 76 05           |jbe short Five.0046695A       ;小于等于则转
00466955 |. 80EA 78         |sub dl,78                     ;大于则减 78（120）
00466958 |. 8810            |mov byte ptr ds:[eax],dl      ;存回原处
0046695A |> 8A10            |mov dl,byte ptr ds:[eax]
0046695C |. 80FA 21         |cmp dl,21                     ;再和 21（33）比较
0046695F |. 73 05           |jnb short Five.00466966       ;不小于则转
00466961 |. 80C2 21         |add dl,21                     ;小于则加上 21（33）
00466964 |. 8810            |mov byte ptr ds:[eax],dl      ;存回原处
00466966 |> 49              |dec ecx                       ;长度减 1
00466967 |. 48              |dec eax                       ;存放地址减 1
00466968 |. 85C9            |test ecx,ecx
0046696A |. 75 D3           \jnz short Five.0046693F
0046696C |. C685 D8FEFFFF 00 mov byte ptr ss:[ebp-128],0
00466973 |. C605 7C9A4700 01 mov byte ptr ds:[479A7C],1    ;注册标志预先置 1，为 1 注册成功
0046697A |. B9 06000000     mov ecx,6                      ;置比较计数
0046697F |. 8D85 D2FEFFFF   lea eax,dword ptr ss:[ebp-12E]   ;取处理好的数据，从低地址开始取，EAX 为数据的低地址首址
00466985 |. 8D95 37FFFFFF   lea edx,dword ptr ss:[ebp-C9]    ;EDX 为输入的注册码字符串首址，它要将注册名处理后的数据和注册码进行比较！
0046698B |> 8A18            /mov bl,byte ptr ds:[eax]       ;取一个处理好的数据
0046698D |. 3A1A            |cmp bl,byte ptr ds:[edx]       ;和一个注册码相比较
0046698F |. 74 09           |je short Five.0046699A        ;相等则继续比较，看得出来，最后变换处理出来的数据就是注册码啦！
00466991 |. C605 7C9A4700 00 |mov byte ptr ds:[479A7C],0    ;不相等则清注册标志
00466998 |. EB 05           |jmp short Five.0046699F
0046699A |> 42              |inc edx
0046699B |. 40              |inc eax
0046699C |. 49              |dec ecx
0046699D |. 75 EC           \jnz short Five.0046698B
0046699F |> 33C0            xor eax,eax
004669A1 |. 5A              pop edx
004669A2 |. 59              pop ecx
004669A3 |. 59              pop ecx
004669A4 |. 64:8910         mov dword ptr fs:[eax],edx
004669A7 |. 68 C4694600     push Five.004669C4
004669AC |> 8D85 C8FDFFFF   lea eax,dword ptr ss:[ebp-238]
004669B2 |. BA 02000000     mov edx,2
004669B7 |. E8 A4D1F9FF     call Five.00403B60              ;这里应该是对输入的注册码进行校验，以防爆破破解
004669BC |. C3              retn
```

到此，记下“`abcdef`”的处理数据“`YRA:3`”，退出调试程序。

打开`Five.exe`，在“注册名”框内输入`abcdef`，在“注册码”框内输入`YRA:3`。

点“注册”，“恭喜你已经成功注册！”。

破解成功，心里甚喜，虽然算法很简单，但毕竟这是我的第一个实例破解啊！

我要从这里开始飞向光明之巅！

### 破解总结

这个软件算法很简单，就是将输入的注册名经过三次变换，而变换的结果就是注册码。

第一步变换，是将输入的注册名的 ASCII 码除以 10 取余，再将余数转换成 ASCII 码，再加 1：

`d=a%10+30+1`

第二步变换，是以注册名字符串的长度为一固定值减去以注册名字符串的长度作计数后的值，

所得值再乘以 2，然后加到第一步变换得的数值上，得到第二次值：

`d=d+(c-(c-n))*2` 即 `d=d+(c-n)*2` (`n=6,5,4,3,2,1`)

第三步变换，是以注册名字符串的长度为一固定值减去以注册名字符串的长度作计数后的值，

所得值乘以 2，再将所得值乘以 2 加上其本身，然后再加上第二步变换得的数值，和第二步差不多：

`d=d+(c-n)*2+((c-n)*2)*2` (`n=6,5,4,3,2,1`)

第三步变换所得的结果就是程序的注册码了。

`4dd466d0e1e0cc123afbe504da2462ca`
