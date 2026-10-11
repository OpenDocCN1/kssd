
# Ace FTP 1.30

> 原文：[`www.pediy.com/kssd/pediy03/forum747.htm`](https://www.pediy.com/kssd/pediy03/forum747.htm)

AceFTP 1.31 软件是一个巨好的 FTP 工具，很多地方都有下载这个软件的注册方式非常具有典型性，很容易就可 以跟出注册。对于我 machoman 这样的菜鸟水平的小混混也能用它来实践写写注册机的感觉。

结果:

注册名：CCG （China Crack Group 中国人自己的破解组 ^\_^)

注册码：6TYPY7-652A34-LMVJA7-YE4JYB

工具 ：

soft ice 4.05

dasm32

步骤:

我先在注册名处输入'CCG'

然后输入假注册码'31415926' (SunBird 老大说的好记^\_^,)

再用 ice 中的 bpx hmemcpy 下断点。再用 F12 进入程序空间。然后用

bd hmemcpy 屏蔽掉这个断点

国外的软件的注册方式,温柔的一般都是通过注册名算出注册码的先看看我的注册名在哪把

在 ice 下用 S 30:0 L ffffffff 'CCG' 命令查找我的注册码的位置.

然后就可以看见在内存单元的一处找到你下的名字,在我的机器上是这样的,不同机器上偏移位置可能不同

30:80607612 43 43 47 00 00 00 00 00-00 00 00 00 00 00 00 00 CCG.................

接下来要做的就是对这个内存位置下断点

bpm 30: 80607612

在下面的位置可以看见以下代码。关于是如何 找到这个位置的,就需要有一点耐性,呵呵,我就是没有,所以好多软件都搞不定:\_(,经常听初学者说如何找 到关键的注册码比较的位置,我要说的就是要多在内存中下断点,看看你的信息在哪里被读取处理,这样你就 会顺藤摸瓜找对地方.注册信息一般都会在内存中搬动几次的,我通过几次的查询在如下位置发现了关键的 地方,这里就可以具体分析注册码了,看见 4bd321 处的 CCG 了吗?我就是根据它来找到注册部分的^\_^

第一段程序的功能是把用户名形成密码的一个输入。

```asm
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:004BD355(C)

:004BD31B 8D45E4                lea eax, dword ptr [ebp-1C]
:004BD31E 8B55F4                mov edx, dword ptr [ebp-0C]//大写的 'CCG'
:004BD321 8A541AFF              mov dl, byte ptr [edx+ebx-01]//顺序取其中的字符
:004BD325 E81E6CF4FF            call 00403F48
:004BD32A 8B45E4                mov eax, dword ptr [ebp-1C]//分离的字符
:004BD32D 8B55EC                mov edx, dword ptr [ebp-14]//串"FJ7ESTYULPVBNMCRY567AA8234452734",这就是注册码的样本字符
:004BD330 E8D76FF4FF            call 0040430C//这个调用,求出根据用户名选取的字符，这里要跟进去看看分析
:004BD335 85C0                  test eax, eax//这里该不为 0 的字符选取.
:004BD337 7E1A                  jle 004BD353//跳就是不需要的字符
:004BD339 8D45E0                lea eax, dword ptr [ebp-20]
:004BD33C 8B55F4                mov edx, dword ptr [ebp-0C]
:004BD33F 8A541AFF              mov dl, byte ptr [edx+ebx-01]
:004BD343 E8006CF4FF            call 00403F48
:004BD348 8B55E0                mov edx, dword ptr [ebp-20]
:004BD34B 8D45F0                lea eax, dword ptr [ebp-10]
:004BD34E E8D56CF4FF            call 00404028

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:004BD337(C)

:004BD353 43                    inc ebx
:004BD354 4E                    dec esi//名字字符的长度减一
:004BD355 75C4                  jne 004BD31B//循环取数
```

注册码算法是把根据用户名字符形成的字符串的顺序每个字符在样本字符集合中查找偏移 位置，再把偏移位置求平方加上上次的结果，再减去本身，把结果除以 0x20 取余，把余 数加 1，的结果在样本字符的结果偏移地址的字符作为真正注册码的一个字符。

用数学公式表示为：

```c
offset_character[i]=mahine[(distance*distance-distance-last_distance)%0x20+1]
```

这个公式不包刮每 6 个字符间隔的一个'-'字符。

```asm
:004BD314(C)

:004BD357 8D45F4                lea eax, dword ptr [ebp-0C]
:004BD35A 8B55F0                mov edx, dword ptr [ebp-10]
:004BD35D E8D66AF4FF            call 00403E38
:004BD362 8B45F4                mov eax, dword ptr [ebp-0C]；字符'CC'
:004BD365 E8B66CF4FF            call 00404020 ；这个调用只是判断字符串的长度
:004BD36A 83F818                cmp eax, 00000018；根据注册名选取的小于 24 个字符吗？
:004BD36D 7D2E                  jge 004BD39D
:004BD36F BE18000000            mov esi, 00000018
:004BD374 2BF0                  sub esi, eax//在 ESI 寄存器中是要添加的字符个数,凑足 24 个字符
:004BD376 85F6                  test esi, esi
:004BD378 7E23                  jle 004BD39D
:004BD37A BB01000000            mov ebx, 00000001

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:004BD39B(C)

:004BD37F 8D45DC                lea eax, dword ptr [ebp-24]
:004BD382 8B55EC                mov edx, dword ptr [ebp-14]
:004BD385 8A541AFF              mov dl, byte ptr [edx+ebx-01]
:004BD389 E8BA6BF4FF            call 00403F48//串行添加字符的调用程序,把样本字符中符合条件的添加到 CC 后面凑足 24 个字符
:004BD38E 8B55DC                mov edx, dword ptr [ebp-24]
:004BD391 8D45F4                lea eax, dword ptr [ebp-0C]
:004BD394 E88F6CF4FF            call 00404028
:004BD399 43                    inc ebx
:004BD39A 4E                    dec esi  //循环记数,直到凑足 24 个字符为止,这个新的字符串就是用来生成注册码的输入量
:004BD39B 75E2                  jne 004BD37F

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
:004BD36D(C), :004BD378(C)

:004BD39D 8B45F4                mov eax, dword ptr [ebp-0C]
:004BD3A0 E87B6CF4FF            call 00404020
:004BD3A5 83F818                cmp eax, 00000018
:004BD3A8 7E16                  jle 004BD3C0；把字符集的数据添加到用户名选
                                ；取字符后，加 24 个字符。
:004BD3AA 8D45F4                lea eax, dword ptr [ebp-0C]
:004BD3AD 50                    push eax
:004BD3AE B918000000            mov ecx, 00000018
:004BD3B3 BA01000000            mov edx, 00000001
:004BD3B8 8B45F4                mov eax, dword ptr [ebp-0C]
:004BD3BB E8686EF4FF            call 00404228

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:004BD3A8(C)

:004BD3C0 8D45F0                lea eax, dword ptr [ebp-10]
:004BD3C3 E8D869F4FF            call 00403DA0
:004BD3C8 BB01000000            mov ebx, 00000001

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
:004BD44F(C)

:004BD3CD 8B45F4                mov eax, dword ptr [ebp-0C]；生成的字符串(用户名+样本字符选取)
:004BD3D0 8A4418FF              mov al, byte ptr [eax+ebx-01]
:004BD3D4 8845EB                mov byte ptr [ebp-15], al
:004BD3D7 8D45D8                lea eax, dword ptr [ebp-28]
:004BD3DA 8A55EB                mov dl, byte ptr [ebp-15]
:004BD3DD E8666BF4FF            call 00403F48
:004BD3E2 8B45D8                mov eax, dword ptr [ebp-28]；顺序取出其中的每个字符。
:004BD3E5 8B55EC                mov edx, dword ptr [ebp-14]；样本字符集合地址
:004BD3E8 E81F6FF4FF            call 0040430C；在样本字符集合中查找，该字符的位置
:004BD3ED 8BF0                  mov esi, eax；ESI 中为该字符在样本字符集合中的偏移地址
:004BD3EF 8B45EC                mov eax, dword ptr [ebp-14]；样本集合地址
:004BD3F2 E8296CF4FF            call 00404020；求样本字符集合的长度
:004BD3F7 50                    push eax；长度为 0x20,32 个，它将作为取模的模数
:004BD3F8 8BC6                  mov eax, esi
:004BD3FA F7EE                  imul esi；偏移位置求平方
:004BD3FC 2BC6                  sub eax, esi；再把结果减去一个偏移地址
:004BD3FE 03C7                  add eax, edi；edi 中为上次的偏移位置，跟 EAX 求和。
:004BD400 5A                    pop edx；样本字符集合的长度，32 个
:004BD401 8BCA                  mov ecx, edx
:004BD403 99                    cdq
:004BD404 F7F9                  idiv ecx；把上一步的结果除以 32，0X20
:004BD406 8BF2                  mov esi, edx；把余数放在 esi 中
:004BD408 46                    inc esi；余数再加 1
:004BD409 8BFE                  mov edi, esi
:004BD40B 8B45EC                mov eax, dword ptr [ebp-14]；样本字符集合地址
:004BD40E 8A4430FF              mov al, byte ptr [eax+esi-01]；把集合中的该位置的字
                                ；符取出，为注册码的一个字符。
:004BD412 8845EA                mov byte ptr [ebp-16], al
:004BD415 8D45D4                lea eax, dword ptr [ebp-2C]
:004BD418 8A55EA                mov dl, byte ptr [ebp-16]
:004BD41B E8286BF4FF            call 00403F48
```



### 汇编代码分析

:004BD420 `8B55D4` `mov edx, dword ptr [ebp-2C]`
:004BD423 `8D45F0` `lea eax, dword ptr [ebp-10]`
:004BD426 `E8FD6BF4FF` `call 00404028`
:004BD42B `8BC3` `mov eax, ebx`
:004BD42D `B906000000` `mov ecx, 00000006`；结果位置为 6 的倍数加'-'字符
:004BD432 `99` `cdq`
:004BD433 `F7F9` `idiv ecx`
:004BD435 `85D2` `test edx, edx`
:004BD437 `7512` `jne 004BD44B`
:004BD439 `83FB18` `cmp ebx, 00000018`；
:004BD43C `7D0D` `jge 004BD44B`；
:004BD43E `8D45F0` `lea eax, dword ptr [ebp-10]`
:004BD441 `BAD4D44B00` `mov edx, 004BD4D4`
:004BD446 `E8DD6BF4FF` `call 00404028`

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
`:004BD437(C), :004BD43C(C)`

:004BD44B `43` `inc ebx`
:004BD44C `83FB19` `cmp ebx, 00000019`；比较注册码转换完了吗？
:004BD44F `0F8578FFFFFF` `jne 004BD3CD`；没完，跳到 `4BD3CD` 循环生成下一个注册码字符。
:004BD455 `8B45F8` `mov eax, dword ptr [ebp-08]`
:004BD458 `8B55F0` `mov edx, dword ptr [ebp-10]`
:004BD45B `E89469F4FF` `call 00403DF4`
:004BD460 `33C0` `xor eax, eax`
:004BD462 `5A` `pop edx`
:004BD463 `59` `pop ecx`
:004BD464 `59` `pop ecx`
:004BD465 `648910` `mov dword ptr fs:[eax], edx`

\* Possible StringData Ref from Code Obj -> `_^\[`

:004BD468 `6897D44B00` `push 004BD497`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
`:004BD495(U)`

:004BD46D `8D45D4` `lea eax, dword ptr [ebp-2C]`
:004BD470 `BA05000000` `mov edx, 00000005`
:004BD475 `E84A69F4FF` `call 00403DC4`
:004BD47A `8D45EC` `lea eax, dword ptr [ebp-14]`
:004BD47D `BA03000000` `mov edx, 00000003`
:004BD482 `E83D69F4FF` `call 00403DC4`
:004BD487 `8D45FC` `lea eax, dword ptr [ebp-04]`
:004BD48A `E81169F4FF` `call 00403DA0`
:004BD48F `C3` `ret`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
`:004BD39B(C)`

--------------------------------------------------------------------------------

//以下的汇编程序段的作用是：把用户输入的用户名串行化在样本字符集合中进行查表，如
//用户名字符在样本字符集合中存在就把该字符记录下来返回后保存在数组，若不在就返回
//`EAX` 为零，不保存。

:0040430C `85C0` `test eax, eax`//字符为 0 吗？
:0040430E `7440` `je 00404350`
:00404310 `85D2` `test edx, edx`//串存在吗？
:00404312 `7431` `je 00404345`
:00404314 `53` `push ebx`
:00404315 `56` `push esi`
:00404316 `57` `push edi`
:00404317 `89C6` `mov esi, eax`
:00404319 `89D7` `mov edi, edx`
:0040431B `8B4FFC` `mov ecx, dword ptr [edi-04]`
:0040431E `57` `push edi`
:0040431F `8B56FC` `mov edx, dword ptr [esi-04]`
:00404322 `4A` `dec edx`
:00404323 `781B` `js 00404340`
:00404325 `8A06` `mov al, byte ptr [esi]`
:00404327 `46` `inc esi`
:00404328 `29D1` `sub ecx, edx`
:0040432A `7E14` `jle 00404340`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
`:0040433E(U)`

:0040432C `F2` `repnz`
:0040432D `AE` `scasb`
:0040432E `7510` `jne 00404340`
:00404330 `89CB` `mov ebx, ecx`
:00404332 `56` `push esi`
:00404333 `57` `push edi`
:00404334 `89D1` `mov ecx, edx`
:00404336 `F3` `repz`
:00404337 `A6` `cmpsb`
:00404338 `5F` `pop edi`
:00404339 `5E` `pop esi`
:0040433A `740C` `je 00404348`//要记录的字符。
:0040433C `89D9` `mov ecx, ebx`
:0040433E `EBEC` `jmp 0040432C`

\* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
`:00404323(C), :0040432A(C), :0040432E(C)`

:00404340 `5A` `pop edx`
:00404341 `31C0` `xor eax, eax`
:00404343 `EB08` `jmp 0040434D`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
`:00404312(C)`

:00404345 `31C0` `xor eax, eax`
:00404347 `C3` `ret`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
`:0040433A(C)`

:00404348 `5A` `pop edx`
:00404349 `89F8` `mov eax, edi`
:0040434B `29D0` `sub eax, edx`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
`:00404343(U)`

:0040434D `5F` `pop edi`
:0040434E `5E` `pop esi`
:0040434F `5B` `pop ebx`

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:
`:0040430E(C)`

:00404350 `C3` `ret`

### 后记

程序把用户名和注册号放在注册表中的 `HKEY_CURRENT_USER\Software\Visicom Media\AceExpertFTP\Registration`。

### 注册机及原代码

//编译方法 `cl ace_jm.c user32.lib`

```c
#include<windows.h>
#include<stdio.h>
#include<math.h>
#include<string.h>

char *dpoint,*dpoint1,*point,*point1,charstr[100],covert[100],username[100],char1,convertname[100];
DWORD k,wwf,wwf2,rtn2,len,number,i,j,rtn,lena,charmm;
char *rtww;

//机器注册码字符集合
LPSTR machine="FJ7ESTYULPVBNMCRY567AA8234452734";

void main(void)
{
    //第一部分程序是在 ace1.30 中用户名的变换算法，在程序中把用户名的变换字符跟注册码的数据交换
    //算法进行比较，如果正确则注册成功。它只是把用户名字符串的长度进行截取。根据长度算出的间隔长
    //度，取出比较字符。
    ///////////////////////////////////////////////////////////////////////////////////////////
    printf("please input big characters:\n");
    scanf("%s",charstr);
    getchar();
    strcpy(username,charstr);

    //把输入的小写变成大写字符，这个程序第一次出
    number=lstrlen(username);
    if(number<5)
    {
        printf("please input user name characters can 't lower 5 NULL\n");
        return;
    }
    point=&char1;
    point1=convertname;
    for(i=0;i<number;i++)
    {
        point=&username[i];
        rtn=IsCharAlpha(*point);
        if(rtn==0)
        {
            printf("IsCharAlpha error");
            return;
        }
        CharUpper(point);
        *point1=*point;
        point1=point1+1;
    }

    //下面的这段程序是把用户输入的用户名在密码字符集合中进行查表，如用户名字符在密码字符集合中存
    //在就把该字符保存在数组 convert 中，并把其后的空间顺序用密码字符集合添加满 24 个字符。
    strcpy(charstr,convertname);
    printf("%s",convertname);
    len=strlen(charstr);
    point=charstr;
    k=0;
    for (i=0;i<=len;i++,point++)
    {
        rtww=strchr(machine,*point);
        if(rtww!=0)
        {
            covert[k++]=*point;}
    }
    lena=lstrlen(covert);
    strncpy(&covert[lena],machine,24-lena);
    //printf("\nthe string is:%s\n",covert);
}
```



//下面的程序段是在字符集中定位字符，并按 6 个一组进行分组排列成注册码。

```
point=covert;
point1=machine;
dpoint1=username;
wwf2=0;
for (i=0;i<24;i++,point++,dpoint1++)
{
  rtww=strchr(machine,*point);
  if(rtww!=0)
  { wwf=rtww-point1+1;
    number=(wwf*wwf-wwf+wwf2)%0x20+1;
     wwf2=number;  
    dpoint=point1+wwf2-1;
    *dpoint1=*dpoint;
   if((i+1)%6==0&&i<23)
  *(++dpoint1)=0x2d;
    }
}
printf("\nresult string:%s\n",username);
}  4dd466d0e1e0cc123afbe504da2462ca
```
