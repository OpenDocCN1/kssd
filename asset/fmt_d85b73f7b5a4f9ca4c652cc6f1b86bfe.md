
```
:00405107 80FB5A              cmp bl, 5A

:0040510A 0F8FF4000000        jg 00405204

:00405110 8A5F01              mov bl, byte ptr [edi+01]    //注册码第 2 位->BL 且在 A-Z 之间?

:00405113 80FB41              cmp bl, 41

:00405116 0F8CE8000000        jl 00405204

:0040511C 80FB5A              cmp bl, 5A

:0040511F 0F8FDF000000        jg 00405204

:00405125 8A4702              mov al, byte ptr [edi+02]    //注册码第 3 位在 A-Z 之间?

:00405128 3C41                cmp al, 41

:0040512A 88442413            mov byte ptr [esp+13], al

:0040512E 0F8CD0000000        jl 00405204

:00405134 3C5A                cmp al, 5A

:00405136 0F8FC8000000        jg 00405204

:0040513C 8A4703              mov al, byte ptr [edi+03]    //注册码第 4 位在 A-Z 之间?

:0040513F 3C41                cmp al, 41

:00405141 88442417            mov byte ptr [esp+17], al

:00405145 0F8CB9000000        jl 00405204

:0040514B 3C5A                cmp al, 5A

:0040514D 0F8FB1000000        jg 00405204

:00405153 8A4704              mov al, byte ptr [edi+04]    //注册码第 5 位在 A-Z 之间?

:00405156 3C41                cmp al, 41

:00405158 0F8CA6000000        jl 00405204

:0040515E 3C5A                cmp al, 5A

:00405160 0F8F9E000000        jg 00405204

:00405166 8A4705              mov al, byte ptr [edi+05]    //注册码第 6 位在 A-Z 之间?

:00405169 3C41                cmp al, 41

:0040516B 88442415            mov byte ptr [esp+15], al

:0040516F 0F8C8F000000        jl 00405204

:00405175 3C5A                cmp al, 5A

:00405177 0F8F87000000        jg 00405204

:0040517D 8A4706              mov al, byte ptr [edi+06]    //注册码第 7 位在 A-Z 之间?

:00405180 3C41                cmp al, 41

:00405182 88442412            mov byte ptr [esp+12], al

:00405186 7C7C                jl 00405204

:00405188 3C5A                cmp al, 5A

:0040518A 7F78                jg 00405204

:0040518C 8A4707              mov al, byte ptr [edi+07]    //注册码第 8 位在 A-Z 之间?

:0040518F 3C41                cmp al, 41

:00405191 88442414            mov byte ptr [esp+14], al

:00405195 7C6D                jl 00405204

:00405197 3C5A                cmp al, 5A

:00405199 7F69                jg 00405204

:0040519B 8A4708              mov al, byte ptr [edi+08]    //注册码第 9 位在 A-Z 之间?

:0040519E 3C41                cmp al, 41

:004051A0 88442416            mov byte ptr [esp+16], al

:004051A4 7C5E                jl 00405204

:004051A6 3C5A                cmp al, 5A

:004051A8 7F5A                jg 00405204

:004051AA 8A4709              mov al, byte ptr [edi+09]    //注册码第 10 位->AL 且在 A-Z 之间?

:004051AD 3C41                cmp al, 41

:004051AF 7C53                jl 00405204

:004051B1 3C5A                cmp al, 5A

:004051B3 7F4F                jg 00405204

:004051B5 3A5C2419            cmp bl, byte ptr [esp+19]    //真假注册码第 2 位比较

:004051B9 7549                jne 00405204

:004051BB 8A5C2413            mov bl, byte ptr [esp+13]    //假注册码第 3 位->[ESP+13]

:004051BF 3A5C241A            cmp bl, byte ptr [esp+1A]    //真假注册码第 3 位比较

:004051C3 753F                jne 00405204

:004051C5 8A5C2417            mov bl, byte ptr [esp+17]    //假注册码第 4 位->[ESP+17]

:004051C9 3A5C241B            cmp bl, byte ptr [esp+1B]    //真假注册码第 3 位比较

:004051CD 7535                jne 00405204

:004051CF 8A5C2415            mov bl, byte ptr [esp+15]    //假注册码第 6 位->[ESP+15]

:004051D3 3A5C241D            cmp bl, byte ptr [esp+1D]    //真假注册码第 6 位比较

:004051D7 752B                jne 00405204

:004051D9 8A5C2412            mov bl, byte ptr [esp+12]    //假注册码第 7 位->[ESP+12]

:004051DD 3A5C241E            cmp bl, byte ptr [esp+1E]    //真假注册码第 7 位比较

:004051E1 7521                jne 00405204

:004051E3 8A5C2414            mov bl, byte ptr [esp+14]    //假注册码第 8 位->[ESP+12]

:004051E7 3A5C241F            cmp bl, byte ptr [esp+1F]    //真假注册码第 8 位比较

:004051EB 7517                jne 00405204

:004051ED 384C2416            cmp byte ptr [esp+16], cl    //真假注册码第 9 位比较

:004051F1 7511                jne 00405204

:004051F3 3AC2                cmp al, dl                   //真假注册码第 10 位比较

:004051F5 750D                jne 00405204                 //想混过去没门(TANY 语)

:004051F7 33C0                xor eax, eax                 //oo 混过来了:)

:004051F9 5F                  pop edi

:004051FA 5E                  pop esi

:004051FB 5D                  pop ebp

:004051FC 5B                  pop ebx

:004051FD 81C408010000        add esp, 00000108

:00405203 C3                  ret
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

```
:00405101(C), :0040510A(C), :00405116(C), :0040511F(C), :0040512E(C)
:00405136(C), :00405145(C), :0040514D(C), :00405158(C), :00405160(C)
:0040516F(C), :00405177(C), :00405186(C), :0040518A(C), :00405195(C)
:00405199(C), :004051A4(C), :004051A8(C), :004051AF(C), :004051B3(C)
:004051B9(C), :004051C3(C), :004051CD(C), :004051D7(C), :004051E1(C)
:004051EB(C), :004051F1(C), :004051F5(C)
```

```
:00405204 5F                  pop edi                        //晕~~~没混过来:(

:00405205 5E                  pop esi

:00405206 5D                  pop ebp

:00405207 83C8FF              or eax, FFFFFFFF

:0040520A 5B                  pop ebx

:0040520B 81C408010000        add esp, 00000108

:00405211 C3                  ret
```

完活-\_-,这是我第一次也是最后一次写详细的了，完全是为了初学者，自己努力吧:)

这个玩艺破解用 10 分钟，写出来花几个小时，我晕~~~~~~

部分注册码：(用户名任意)

`Name:ZOMBIE`

`Code:RNFKCCNNWW`

`Code:ALCKENLEUN`

———————————————————————————————>

```
        .-"     "-.     Cracked by zombieys     >
        /          \     My oicq is 1789655      >
       |            |   My hp is zombieys.yeah.net>
       |, .-. .-. ,|   My hp is zombieys.126.com >
         |)(\_\_/  \\\_\_)(|                        >
         |/   /\\   \\|     Thanks for your supports >
     (@\_)@)   (\_   ^^   \_)                        >
   \_ )\\\_\_\_\_\_\_\_\\\_\_|IIIIII|\_\_/\_\_\_\_\_        >
\_)@8@8{}<\_\_\_\_\_\_\_\_|-\\IIIIII/-|\_\_\_\_zombieys\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_>
```

4dd466d0e1e0cc123afbe504da2462ca
