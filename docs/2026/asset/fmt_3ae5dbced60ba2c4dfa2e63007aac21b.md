
```text
  mov ebx, eax            <-

保存在ebx

:0045C9B5 8D45EC

lea eax, dword ptr [ebp-14]

:0045C9B8 BA08000000              mov edx,

00000008

:0045C9BD E86A79FAFF              call 0040432C

          <- 取前面的的8个字符

:0045C9C2 8B45EC

mov eax, dword ptr [ebp-14]        <- Irc的前8个字符

:0045C9C5 E8F6F4FFFF              call 0045BEC0

          <- 计算中间校验码，保存在al中

>>>>>>>>

    \* Referenced by a CALL at Addresses:

    |... ...

    |

    :0045BEC0 53

        push ebx

    :0045BEC1 56

        push esi

    :0045BEC2 57

        push edi

    :0045BEC3 8BF8

      mov edi, eax

    :0045BEC5 33DB

      xor ebx, ebx

    :0045BEC7 8BC7

      mov eax, edi

    :0045BEC9 E82A81FAFF

  call 00403FF8

    :0045BECE 85C0

      test eax, eax

    :0045BED0 7438

      je 0045BF0A

    :0045BED2 0FB637

      movzx esi, byte ptr [edi]        <-

取得第一个字符

    :0045BED5 8BC7

      mov eax, edi

    :0045BED7 E81C81FAFF

  call 00403FF8        <- 取长度，得到eax=8

    :0045BEDC 8BD0

      mov edx, eax

    :0045BEDE 83EA02

      sub edx, 00000002

    :0045BEE1 7C22

      jl 0045BF05

  <- 不会跳

    :0045BEE3 42

        inc edx

    :0045BEE4 B802000000

  mov eax, 00000002

    \* Referenced by a Jump at Address:

    |:0045BF03(C)

              <- 这个循环生成中间校验码到esi

    |

    :0045BEE9 A801

      test al, 01

  <- 对2取模

    :0045BEEB 740C

      je 0045BEF9

    :0045BEED 33C9

      xor ecx, ecx

    :0045BEEF 8A4C07FF

    mov cl, byte ptr [edi+eax-01]    <- 取下一个字符

    :0045BEF3 03C9

      add ecx, ecx

  <- cl\*=2

    :0045BEF5 33F1

      xor esi, ecx

  <- si^=cx

    :0045BEF7 EB08

      jmp 0045BF01

    \* Referenced by a Jump at Address:

    |:0045BEEB(C)

    |

    :0045BEF9 33C9

      xor ecx, ecx

    :0045BEFB 8A4C07FF

    mov cl, byte ptr [edi+eax-01]    <- 取下一个字符

    :0045BEFF 33F1

      xor esi, ecx        <-

si^cx

    \* Referenced by a Jump at Address:

    |:0045BEF7(U)

    |

    :0045BF01 40

        inc eax

    :0045BF02 4A

        dec edx

    :0045BF03 75E4

      jne 0045BEE9

  <- 未完继续

    |

    :0045BF05 8BDE

      mov ebx, esi

  <- bx=si

    :0045BF07 80E3FF

      and bl, FF

    \* Referenced by a (U)nconditional or (C)onditional Jump at

Address:

    |:0045BED0(C)

    |

    :0045BF0A 8BC3

      mov eax, ebx

  <- al=bl

    :0045BF0C 5F

        pop edi

    :0045BF0D 5E

        pop esi

    :0045BF0E 5B

        pop ebx

    :0045BF0F C3

        ret

  嘿嘿，这一段变成C就是：

    cCRCCheck='\0';

    for(i=0; i<8; i++)

        cCRCCheck^=(i>1 && i%2==0) ? cIrc[i]\*2 :

cIrc[i];

<<<<<<<<

:0045C9CA E8C5F2FFFF              call 0045BC94

          <- 对上面Call产生的中间校验码作镜像翻转，说不明白，编小段程序就知道啦

>>>>>>>>

\* Referenced by a CALL at Addresses:

|... ...

|

    :0045BC94 D0E8

      shr al, 1

    :0045BC96 D0D4

      rcl ah, 1

    :0045BC98 D0E8

      shr al, 1

    :0045BC9A D0D4

      rcl ah, 1

    :0045BC9C D0E8

      shr al, 1

    :0045BC9E D0D4

      rcl ah, 1

    :0045BCA0 D0E8

      shr al, 1

    :0045BCA2 D0D4

      rcl ah, 1

    :0045BCA4 D0E8

      shr al, 1

    :0045BCA6 D0D4

      rcl ah, 1

    :0045BCA8 D0E8

      shr al, 1

    :0045BCAA D0D4

      rcl ah, 1

    :0045BCAC D0E8

      shr al, 1

    :0045BCAE D0D4

      rcl ah, 1

    :0045BCB0 D0E8

      shr al, 1

    :0045BCB2 D0D4

      rcl ah, 1

    :0045BCB4 86E0

      xchg al, ah        <-

校验码保存到al

    :0045BCB6 C3

        ret

<<<<<<<<

:0045C9CF 25FF000000              and eax,

000000FF

:0045C9D4 3BD8

  cmp ebx, eax            <-

bl放的是Irc的最后两个，比较。因此最后两位是校验码

:0045C9D6 757E

  jne 0045CA56            <-

不是就去死

:0045C9D8 8B45EC

mov eax, dword ptr [ebp-14]

:0045C9DB E8FCF9FFFF              call 0045C3DC

          <- 将Irc变为立即数到eax

:0045C9E0 8945E8

mov dword ptr [ebp-18], eax    <- 保存

:0045C9E3 8D45D8

lea eax, dword ptr [ebp-28]        <- 缓冲区首址

:0045C9E6 50

  push eax

:0045C9E7 E890A8FAFF              call 0040727C

          <- GetSystemTime

:0045C9EC 8B35682C5700            mov esi, dword

ptr [00572C68]

:0045C9F2 83E60F

and esi, 0000000F

:0045C9F5 83FE00

cmp esi, 00000000            <-

esi==5

:0045C9F8 7C54

  jl 0045CA4E

  <- 不跳

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:0045CA4C(C)

|

:0045C9FA 0FB745D8                movzx

eax, word ptr [ebp-28]    <- 年

:0045C9FE C1E00A

shl eax, 0A

:0045CA01 0FB755DA                movzx

edx, word ptr [ebp-26]    <- 月

:0045CA05 0BC2

  or eax, edx

:0045CA07 C1E008

shl eax, 08

:0045CA0A 0FB755DE                movzx

edx, word ptr [ebp-22]    <- 日

:0045CA0E 0BC2

  or eax, edx

  比如 2001/08/27，则eax=0x1F44081B (0x1F44=2001\*4)。时间参与运算是为增加得法的难度和随机性，也说明正确的注册码有许多

:0045CA10 8B55E8

mov edx, dword ptr [ebp-18]    <- edx=Irc

:0045CA13 33D0

  xor edx, eax            <-

edx^=eax (Date)

:0045CA15 3355FC

xor edx, dword ptr [ebp-04]    <- edx^=Hwn

:0045CA18 8B5DF8

mov ebx, dword ptr [ebp-08]    <- ebx=0xDE7C

:0045CA1B F7D3

  not ebx            <-

ebx=~ebx=0xFFFF2183

:0045CA1D 33DA

  xor ebx, edx        <- ebx^=edx

:0045CA1F 8BD3

  mov edx, ebx        <- edx=ebx

:0045CA21 81E2FFFF0000            and edx, 0000FFFF

:0045CA27 C1EB12

shr ebx, 12

:0045CA2A B8FF1F0000              mov eax,

00001FFF        <-

:0045CA2F 3BDA

  cmp ebx, edx            <-

两个值不等则失败

:0045CA31 750D

  jne 0045CA40

:0045CA33 85D2

  test edx, edx            <-

这个值要大于0

:0045CA35 7E09

  jle 0045CA40

:0045CA37 3BC2

  cmp eax, edx            <-

要小于0x1FFF

:0045CA39 7C05

  jl 0045CA40

:0045CA3B 895DF0

mov dword ptr [ebp-10], ebx    <- 保存结果

:0045CA3E EB0E

  jmp 0045CA4E

... ...(不重要了)

:0045CA83 8B45F0

mov eax, dword ptr [ebp-10]    <-

:0045CA86 5E

  pop esi

:0045CA87 5B

  pop ebx

:0045CA88 8BE5

  mo
```
