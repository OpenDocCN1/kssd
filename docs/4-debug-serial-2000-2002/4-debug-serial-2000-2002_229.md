
# Banner Maker Professional 3.0.1.0

> 原文：[`www.pediy.com/kssd/pediy03/forum504.htm`](https://www.pediy.com/kssd/pediy03/forum504.htm)

小弟是个初学者,以后还望各位大虾多多指点;)

`Banner Maker Professional 3.0.1.0` 的破解

介绍:一个简单易用的制作广告条的软件, 也可以看做一个简单的制图软件.

难度:非常简单,建议高手略过.

到程序的注册窗口,输入注册名,注册码 按 `OK`,什么!?注册码没反应,退出,再运行,还是未注册,&#$^@!#$&~ 看来程序似乎有 BUG 哦!呵呵 :)

让我来修理修理它;-)

切到 `TRW` 下,下断点 `Bpx hmemcpy`,回到注册窗口,按 `OK` 我们又回到了 `TRW` 下

按 `N` 多次 `F12` 后,

```
:00501588 E89F23F3FF    call 0043392C          
:0050158D 8B45F8        mov eax, dword ptr [ebp-08]  <------我们来到了这里  EAX 是你输入的注册码(当然是假的咯 ;-)
:00501590 BAD8165000    mov edx, 005016D8      * 下 D 005016d8 后显示一串数据 3122545210621
:00501595 E8FE2AF0FF    call 00404098          * 比较注册码的 CALL
:0050159A 0F85AB000000  jne 0050164B
:005015A0 A1145D5100    mov eax, dword ptr [00515D14]
:005015A5 C6800808000000 mov byte ptr [eax+00000808], 00
:005015AC B201          mov dl, 01
:005015AE A168B44500    mov eax, dword ptr [0045B468]
:005015B3 E8B09FF5FF    call 0045B568
```

进入哪个 `CALL`,来到了这儿:

```
:00404098 53            push ebx
:00404099 56            push esi
:0040409A 57            push edi
:0040409B 89C6          mov esi, eax
:0040409D 89D7          mov edi, edx
:0040409F 39D0          cmp eax, edx           * 非常有趣哦,将 3122545210621 和你的注册名比较?
:004040A1 0F848F000000  je 00404136
:004040A7 85F6          test esi, esi
:004040A9 7468          je 00404113
:004040AB 85FF          test edi, edi
:004040AD 746B          je 0040411A
:004040AF 8B46FC        mov eax, dword ptr [esi-04]
:004040B2 8B57FC        mov edx, dword ptr [edi-04]
:004040B5 29D0          sub eax, edx
:004040B7 7702          ja 004040BB
:004040B9 01C2          add edx, eax
```

用 `3122545210621` 作注册码,成功!

后来发现,在注册名栏中输入任意字符,用 `3122545210621` 作注册码,全部成功!

`4dd466d0e1e0cc123afbe504da2462ca`
