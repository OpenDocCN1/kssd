
# 文件密使 2.0

> 原文：[`www.pediy.com/kssd/pediy03/forum1141.htm`](https://www.pediy.com/kssd/pediy03/forum1141.htm)

文件密使 2.0 暴力破解及注册机

说明：如何保护信息的安全使之不被窃取，而造成损失，密码是有效而且可行的办法。文件密使就是在这种理念下产生的一个对计算机信息进行加密的工具。

下载：`http://www.esoftware.com.cn/oload.php?url=http://ftp.eware.com.cn/pub/fmanager/encrypt/jiamiV2.0.exe`

## 一、暴力破解法

很简单，只用 `wdasm893` 即可，用那个可以反编译汉字的版本。

查找“文件密使 2.0 - ”，可找到 2 处，该字串是显示在窗体的标题栏的。其中一处是启动时判断有没有注册后显示的；一处是注册成功后显示的。

下面是启动时判断注册成功后显示处：

```
* Reference To: MFC42.Ordinal:0217, Ord:0217h
                        |
:004080DE E8D5290100       Call 0041AAB8
:004080E3 8BCD             mov ecx, ebp
:004080E5 E8069F0000       call 00411FF0   **进去便是判断注册的程序**
:004080EA 33C9            xor ecx, ecx
:004080EC 3BC1            cmp eax, ecx
:004080EE 7454            je 00408144   **该处便是跳到显示未注册处代码**
                          **只要讲该处的代码 NOP 掉就可以了**
:004080F0 B801000000      mov eax, 00000001
:004080F5 57             push edi
:004080F6 894668         mov dword ptr [esi+68], eax
:004080F9 898630130000   mov dword ptr [esi+00001330], eax
:004080FF 8986E4280000   mov dword ptr [esi+000028E4], eax
:00408105 8D442418       lea eax, dword ptr [esp+18]
* Possible StringData Ref from Data Obj ->"文件密使 2.0 - "
                        |
:00408109 681C854200      push 0042851C
:0040810E 50             push eax
* Reference To: MFC42.Ordinal:039E, Ord:039Eh
                        |
:0040810F E8722B0100      Call 0041AC86
:00408114 8B00            mov eax, dword ptr [eax]
:00408116 8BCE            mov ecx, esi
:00408118 50             push eax
:00408119 C78424C001000004000000 mov dword ptr [esp+000001C0], 00000004
```

若有人想来个完美的暴破，也可以到下处：

```
* Reference To: MFC42.Ordinal:0217, Ord:0217h
                        |
:00415CB5 E8FE4D0000      Call 0041AAB8
:00415CBA 8D8E50160000    lea ecx, dword ptr [esi+00001650]
:00415CC0 E82BC3FFFF      call 00411FF0
:00415CC5 85C0            test eax, eax
:00415CC7 752E            jne 00415CF7   **将此处跳过即可任意注册**
                      **其实只是在注册表内填了个注册名罢了 8-) **
:00415CC9 6A40            push 00000040
* Possible StringData Ref from Data Obj ->"warning"
                        |
:00415CCB 6890814200      push 00428190
* Possible StringData Ref from Data Obj ->"注册码错误，请重新输入"
                        |
:00415CD0 68008F4200      push 00428F00
:00415CD5 8BCB            mov ecx, ebx
```

总结一下：

1、`:004080EE 7454` ----> `9090`
2、` :00415CC7 752E` ----> `eb2e`

## 二、注册机的编写

此处便是 `:004080E5` 处 `call 00411FF0` 里面的内容，该处的内容在启动和注册时都要用到。

是判断的核心。

```
:00411FF0 6AFF             push FFFFFFFF
:00411FF2 6878E04100       push 0041E078
:00411FF7 64A100000000     mov eax, dword ptr fs:[00000000]
:00411FFD 50              push eax
:00411FFE 64892500000000  mov dword ptr fs:[00000000], esp
:00412005 83EC24          sub esp, 00000024
:00412008 55              push ebp
:00412009 56              push esi
:0041200A 57              push edi
:0041200B 8B7C2440        mov edi, dword ptr [esp+40]
:0041200F 33ED            xor ebp, ebp
:00412011 896C240C        mov dword ptr [esp+0C], ebp
:00412015 8B77F8          mov esi, dword ptr [edi-08] **注册名长度**
:00412018 85F6            test esi, esi
:0041201A 7527            jne 00412043
:0041201C 8D4C2440        lea ecx, dword ptr [esp+40]
:00412020 C7442438FFFFFF mov [esp+38], FFFFFFFF
* Reference To: MFC42.Ordinal:0320, Ord:0320h
                        |
:00412028 E895890000       Call 0041A9C2
:0041202D 5F              pop edi
:0041202E 5E              pop esi
:0041202F 33C0            xor eax, eax
:00412031 5D              pop ebp
:00412032 8B4C2424        mov ecx, dword ptr [esp+24]
:00412036 64890D00000000  mov dword ptr fs:[00000000], ecx
:0041203D 83C430          add esp, 00000030
:00412040 C20800          ret 0008
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0041201A(C)
|
:00412043 83FE20          cmp esi, 00000020  **长度与 32 比较**
:00412046 7E05           jle 0041204D
:00412048 BE20000000     mov esi, 00000020
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00412046(C)
|
:0041204D 33C0            xor eax, eax
:0041204F 85F6            test esi, esi
:00412051 7E14            jle 00412067
:00412053 8D4C2410        lea ecx, dword ptr [esp+10]
:00412057 2BF9            sub edi, ecx
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00412065(C)
|
:00412059 8D540410        lea edx, dword ptr [esp+eax+10]
:0041205D 40              inc eax
:0041205E 3BC6            cmp eax, esi
:00412060 8A0C17          mov cl, byte ptr [edi+edx]
:00412063 880A            mov byte ptr [edx], cl  **将名字放入[edx]处**
:00412065 7CF2            jl 00412059
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00412051(C)
|
:00412067 83FE20          cmp esi, 00000020
:0041206A 53              push ebx
:0041206B 8BCE            mov ecx, esi
:0041206D 7D30            jge 0041209F
:0041206F 8D7C2414        lea edi, dword ptr [esp+14]
:00412073 8D6C2414        lea ebp, dword ptr [esp+14]
:00412077 4F              dec edi
:00412078 2BEF            sub ebp, edi
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00412099(C)
|
:0041207A 8A1C0F          mov bl, byte ptr [edi+ecx] **取名字最后一位**
:0041207D 8D340F          lea esi, dword ptr [edi+ecx]
:00412080 8BC3            mov eax, ebx
:00412082 25FF000000      and eax, 000000FF
:00412087 99              cdq
:00412088 F7F9            idiv ecx
:0041208A 8BD0            mov edx, eax
:0041208C 8AC3            mov al, bl
:0041208E F6E9            imul cl
:00412090 02D0            add dl, al
:00412092 41              inc ecx
:00412093 83F920          cmp ecx, 00000020
:00412096 88142E          mov byte ptr [esi+ebp], dl
:00412099 7CDF            jl 0041207A
```

此处以上为取名字最后一位，做运算，再用运算结果重复运算，每运算一次生成一个数写入到名字后面的内存中，生成一张表，加上名字共 32 个字节。

```
:0041209B 8B6C2410        mov ebp, dword ptr [esp+10]
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0041206D(C)
|
:0041209F 33FF            xor edi, edi
:004120A1 33F6            xor esi, esi
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004120D6(C)
|
:004120A3 8BC6            mov eax, esi
:004120A5 33C9            xor ecx, ecx
:004120A7 99              cdq
:004120A8 83E203          and edx, 00000003
:004120AB C6443C1000      mov [esp+edi+10], 00
:004120B0 03C2            add eax, edx
:004120B2 C1F802          sar eax, 02
:004120B5 8D440414        lea eax, dword ptr [esp+eax+14]
```



* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`|:004120CA(C)`

```
:004120B9 8A1408    mov dl, byte ptr [eax+ecx]  **依次取刚才生成
:004120BC 8A5C3C10  mov bl, byte ptr [esp+edi+10] 的表的各字节**
:004120C0 32DA      xor bl, dl **俩俩异或**
:004120C2 41        inc ecx
:004120C3 83F908    cmp ecx, 00000008
:004120C6 885C3C10  mov byte ptr [esp+edi+10], bl
:004120CA 7CED      jl 004120B9
:004120CC 83C620    add esi, 00000020
:004120CF 47        inc edi
:004120D0 81FE80000000  cmp esi, 00000080
:004120D6 7CCB      jl 004120A3
```
此前一小段是将该内存表的各字节 8 个一组，依次异或，共生成 4 个数，依次填到名字前的四个字节中。将该四个数倒过来（即离名字最近的一位为高位），转为 10 进制即是注册码。

```
:004120D8 B803000000 mov eax, 00000003
:004120DD 5B         pop ebx
```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`|:004120EE(C)`

```
:004120DE 33C9      xor ecx, ecx
:004120E0 8A4C040C mov cl, byte ptr [esp+eax+0C]
:004120E4 03E9      add ebp, ecx
:004120E6 85C0      test eax, eax
:004120E8 7406      je 004120F0
:004120EA C1E508    shl ebp, 08
:004120ED 48        dec eax
:004120EE 79EE      jns 004120DE
```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`|:004120E8(C)`

```
:004120F0 8B542444 mov edx, dword ptr [esp+44]
:004120F4 8D4C2440 lea ecx, dword ptr [esp+40]
:004120F8 2BEA      sub ebp, edx
:004120FA C7442438FFFFFFFF mov [esp+38], FFFFFFFF
:00412102 F7DD      neg ebp
:00412104 1BED      sbb ebp, ebp
:00412106 45        inc ebp
:00412107 8BF5      mov esi, ebp
```
* Reference To: `MFC42.Ordinal:0320`, `Ord:0320h`

`|`

```
:00412109 E8B4880000 Call 0041A9C2
:0041210E 8B4C2430  mov ecx, dword ptr [esp+30]
:00412112 8BC6      mov eax, esi
:00412114 5F        pop edi
:00412115 5E        pop esi
:00412116 5D        pop ebp
:00412117 64890D00000000 mov dword ptr fs:[00000000], ecx
:0041211E 83C430    add esp, 00000030
:00412121 C20800    ret 0008
```
上面主要是讨论了名字长度不大于 32 位的时候，若大于则只取前 32 位，便用该 32 位做表，以后过程与小于 32 位的相同。

下面是注册机，用 `TC2` 写的，我的 `C` 很烂，所以估计很难看懂 8-）。

```
main()
{int i,k,num1,num2,num3,a[32];long num4,b[4];

char name[32];

printf("Jiami v2.0 Keymaker by KanKer.\n");

printf("\n******************************************\n");

printf("\nWelcome to visit my homepage:\nhttp://kanker.ccoo.com.");

printf("\n\n******************************************\n");

printf("\nPlease input your name(length<=32):");

gets(name);

i=strlen(name);

if (i<0x20)

{num4=name[i-1];

for(k=i;k<0x20;k++)

{num1=num4;

num1=num1&0xff;

num2=num1/k;

num3=(num1*k)%0x100;

num4=num2+num3;

a[k]=num4%0x100;

}

}

for(k=0;k<i;k++)

a[k]=name[k];

/***************

num4=1;

for(i=0;i<4;i++)

{

num3=0;

for(k=8*i;k<8*i+8;k++)

{num1=num3;

num2=a[k];

num3=num2^num1;

}

b[i]=num3*num4;

num4=num4*0x100;

}

num4=0;

for(i=3;i>=0;i--)

num4=num4+b[i];

printf("Your regcode is: %ld",num4);}
```
但用此注册机注出来有很多是负数，这在程序注册时是输不进去的，大概是原程序本身的问题吧？所以只能找个能注出正数的名字来注册了，呵呵。

Cracked by `KanKer`

`http://kanker.ccoo.com` `4dd466d0e1e0cc123afbe504da2462ca`
