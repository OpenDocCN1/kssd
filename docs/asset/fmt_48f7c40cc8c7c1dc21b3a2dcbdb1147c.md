
* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

`|:004BAF53(C), :004BAF5C(C)`

```
:004BAFBA 895DEC             mov dword ptr [ebp-14], ebx
:004BAFBD 895DF4             mov dword ptr [ebp-0C], ebx
:004BAFC0 894DF8             mov dword ptr [ebp-08], ecx
:004BAFC3 8955FC             mov dword ptr [ebp-04], edx
:004BAFC6 8B45FC             mov eax, dword ptr [ebp-04]
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`|:004BAF50(C)`

```
:004BAFC9 E81E90F4FF         call 00403FEC
:004BAFCE 33C0               xor eax, eax
:004BAFD0 55                 push ebp
:004BAFD1 686EB04B00         push 004BB06E
:004BAFD6 64FF30             push dword ptr fs:[eax]
:004BAFD9 648920             mov dword ptr fs:[eax], esp
:004BAFDC 33F6               xor esi, esi
:004BAFDE 8D45F4             lea eax, dword ptr [ebp-0C]
:004BAFE1 8B55FC             mov edx, dword ptr [ebp-04]
:004BAFE4 E8678CF4FF         call 00403C50
:004BAFE9 8B45F4             mov eax, dword ptr [ebp-0C]
:004BAFEC E8478EF4FF         call 00403E38
:004BAFF1 85C0               test eax, eax
:004BAFF3 7E3D               jle 004BB032
:004BAFF5 8945F0             mov dword ptr [ebp-10], eax
:004BAFF8 BB01000000         mov ebx, 00000001
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`|:004BB030(C)`

```
:004BAFFD 8D45EC             lea eax, dword ptr [ebp-14]
:004BB000 50                 push eax
:004BB001 B901000000         mov ecx, 00000001
:004BB006 8BD3               mov edx, ebx
:004BB008 8B45F4             mov eax, dword ptr [ebp-0C]
:004BB00B E83090F4FF         call 00404040
:004BB010 8B45EC             mov eax, dword ptr [ebp-14]
:004BB013 E8E48FF4FF         call 00403FFC
:004BB018 8A00               mov al, byte ptr [eax]    //依次取名字的字符
:004BB01A 33D2               xor edx, edx              //清空寄存器
:004BB01C 8AD0               mov dl, al                //将取出的字符 ascii 给 dl
:004BB01E 8BCA               mov ecx, edx              //再给 ecx
:004BB020 0FAFCA             imul ecx, edx             //ecx*edx，也就是将取出字符的 ascii 码开方！
:004BB023 8D4317             lea eax, dword ptr [ebx+17] //ebx+17 中依次是 18、19。。。传给 eax
:004BB026 F7EE               imul esi                  //乘 esi，这里第一次 esi 是 0,
:004BB028 03C8               add ecx, eax              //ecx+eax 相加
:004BB02A 8BF1               mov esi, ecx              //保存运算结果到 esi 中
:004BB02C 43                 inc ebx                   //计数，循环多少次？名字的字符串个数！
:004BB02D FF4DF0             dec [ebp-10]
:004BB030 75CB               jne 004BAFFD
```

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`|:004BAFF3(C)`

```
:004BB032 8B45F8             mov eax, dword ptr [ebp-08] //这个是将输入的注册码 Hex 值给 eax
:004BB035 2D4D050000         sub eax, 0000054D           //eax-0x54D 传给 eax
:004BB03A 3BF0               cmp esi, eax                //看结果是不是等于 esi（上面运算出的值）
:004BB03C 7508               jne 004BB046                //不能跳哟！
:004BB03E 85F6               test esi, esi
:004BB040 7404               je 004BB046
:004BB042 B301               mov bl, 01                  //看到这个了吧！哈哈哈哈！舒服呀！
:004BB044 EB02               jmp 004BB048
```

### 算法

以姓名 `TAE!` 为例：

* `A = 0x54² + 0x18 * 0`
* `B = 0x41² + 0x19 * A`
* `C = 0x45² + 0x20 * B`
* `D = 0x21² + 0x21 * C`
* `SN = D + 0x54D`

#### 得到注册码的简单方法

到了 `004BB032` 那一行，可以用下命令 `? esi+54D`，即可得到正确注册码！！

#### 一个可用的注册码

* **Name**: `TAE!`
* **Sn**: `126929743`

下面这个 ascii 涂鸦就是用这个软件做的，呵呵！

```text
                    .----.
                _.'__   `.
            .--(#)(##)---/#\
            .' @            /###\
            :        ,    #####
            -..___.-' _. \###/
                    ;\_:    `"'
                .'"""""\`.
               /,  JOE  ,\
               //  COOL!  \\
             `.-__________.-'
            _______. | .'____
          (________|________)
```

```c
#include "stdio.h"

main()
{
char string[81];
unsigned long i,a=0,b=0;
unsigned c;

printf("******OICQ PIC 3.20******\nThis keygen is maked by Pretender\nPlease input your name\n");
gets(string);
for(i=0;(c=string[i])!='\0';i++)
{a=(24+i)*a;
if(c>127) c=256+c;
b=c*c;
a=a+b;
}
printf("Your name is ");
puts(string);
printf("Your Register code is %ld\n",a+0x54d);
printf("Welcome to WWW.CRACKNOW.COM\n");}
```
`4dd466d0e1e0cc123afbe504da2462ca`
