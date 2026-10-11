
# API Spy 32 v2.4

> 原文：[`www.pediy.com/kssd/pediy05/pediy50021.htm`](https://www.pediy.com/kssd/pediy05/pediy50021.htm)

目标：`APIS32` v2.4（`API Spy 32` v2.4）

最近一直心情不太好，也有将近 2 个月没有玩破解了，想找几个软件出出气，也当是复习一下。不料自己已经变懒了，不想分析什么算法之类的，一心只想爆破，不知道以后会不会变成爆破狂，呵呵！

不说废话了，今天的对象是一个协助 Crack 的辅助工具：`APIS32` v2.4（`API Spy 32` v2.4），用了监测程序调用了哪些 api 函数的工具，有了它，就不怕找不到函数下断点了，呵呵。那知道装上它一用，还要注册，那就不客气了，先拿它开刀了。

工具：`Trw`、`冲击波`、`pw32das`、`ultraedit`、`peid`

用 `peid` 检测得知，程序用 `petite` 1.2 加了壳，用`冲击波`＋`trw` 搞定它的壳；

用 `pw32das` 反编译脱壳后的程序，查找注册不成功的信息，来到这里：

* Reference To: `KERNEL32.lstrcatA`, Ord:0000h

|
:00401764 FF15F8024100 Call dword ptr [004102F8]
:0040176A E8312F0000 call 004046A0//关键 call，返回注册标志到 eax，跟进去！
:0040176F A374CE4000 mov dword ptr [0040CE74], eax//这里的[0040CE74]存放有注册成功与否的标志，其它地方还会进行检查，所以不能简单的改下面那个跳转！
:00401774 EB01 jmp 00401777
:00401776 B8 BYTE B8 －－－－－－－－－

|
* Referenced by a (U)nconditional or (C)onditional Jump at Address: 这里是花指令，在 trw 的动态调试时，

|:00401774(U) |＝＝看到的是 MOV [0040CE74],EAX 指令

|
:00401777 0AC0 or al, al －－－－－－－－－
:00401779 7402 je 0040177D//要走到下面的指令，这里就不能跳，但不要改这里，看上面！
:0040177B EB2C jmp 004017A9//这里就能跳过去，

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00401779(C)

|

* Possible StringData Ref from Data Obj ->"The registration information you "//出错信息！向上找跳，看哪里能跳过这里！
->"provided is incorrect. Please verify "
->"that you entered your name and code properly, and try again. If "
->"you encounter difficulties, please send mail to APIS32@Biosys.net or "
->"visit our web site http://madmat.hypermart.net"

|
:0040177D BFE8904000 mov edi, 004090E8
:00401782 BAE0D14000 mov edx, 0040D1E0
:00401787 83C9FF or ecx, FFFFFFFF
:0040178A 33C0 xor eax, eax
:0040178C F2 repnz
:0040178D AE scasb
:0040178E F7D1 not ecx
:00401790 2BF9 sub edi, ecx
:00401792 8BF7 mov esi, edi
:00401794 8BC1 mov eax, ecx
:00401796 8BFA mov edi, edx
:00401798 C1E902 shr ecx, 02
:0040179B F3 repz
:0040179C A5 movsd
:0040179D 8BC8 mov ecx, eax
:0040179F 83E103 and ecx, 00000003
:004017A2 F3 repz
:004017A3 A4 movsb
:004017A4 E9B1000000 jmp 0040185A

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0040177B(U)

|

* Possible StringData Ref from Data Obj ->"Registered to "

|
:004017A9 BF40904000 mov edi, 00409040
:004017AE BAE0D14000 mov edx, 0040D1E0
:004017B3 83C9FF or ecx, FFFFFFFF
:004017B6 33C0 xor eax, eax
:004017B8 F2 repnz
:004017B9 AE scasb
:004017BA F7D1 not ecx
:004017BC 2BF9 sub edi, ecx
:004017BE 8BF7 mov esi, edi
:004017C0 8BD9 mov ebx, ecx
:004017C2 8BFA mov edi, edx
:004017C4 83C9FF or ecx, FFFFFFFF
:004017C7 33C0 xor eax, eax
:004017C9 F2 repnz
:004017CA AE scasb
:004017CB 83C7FF add edi, FFFFFFFF
:004017CE 8BCB mov ecx, ebx
:004017D0 C1E902 shr ecx, 02
:004017D3 F3 repz
:004017D4 A5 movsd
:004017D5 8BCB mov ecx, ebx
:004017D7 83E103 and ecx, 00000003
:004017DA F3 repz
:004017DB A4 movsb
:004017DC BF00CF4000 mov edi, 0040CF00
:004017E1 BAE0D14000 mov edx, 0040D1E0
:004017E6 83C9FF or ecx, FFFFFFFF
:004017E9 33C0 xor eax, eax
:004017EB F2 repnz
:004017EC AE scasb
:004017ED F7D1 not ecx
:004017EF 2BF9 sub edi, ecx
:004017F1 8BF7 mov esi, edi
:004017F3 8BD9 mov ebx, ecx
:004017F5 8BFA mov edi, edx
:004017F7 83C9FF or ecx, FFFFFFFF
:004017FA 33C0 xor eax, eax
:004017FC F2 repnz
:004017FD AE scasb
:004017FE 83C7FF add edi, FFFFFFFF
:00401801 8BCB mov ecx, ebx
:00401803 C1E902 shr ecx, 02
:00401806 F3 repz
:00401807 A5 movsd
:00401808 8BCB mov ecx, ebx
:0040180A 83E103 and ecx, 00000003
:0040180D F3 repz
:0040180E A4 movsb
:0040180F 68E0D14000 push 0040D1E0
:00401814 8B4508 mov eax, dword ptr [ebp+08]
:00401817 50 push eax

* Reference To: `USER32.SetWindowTextA`, Ord:0000h

|
:00401818 FF15B4034100 Call dword ptr [004103B4]

* Possible StringData Ref from Data Obj ->"Thanks for Registering APIS32!"//注册成功的信息！

|
:0040181E BFA8904000 mov edi, 004090A8
:00401823 BAE0D14000 mov edx, 0040D1E0
:00401828 83C9FF or ecx, FFFFFFFF
:0040182B 33C0 xor eax, eax
:0040182D F2 repnz
。。。。。。

跟进上面那个 call，来到这里：

* Referenced by a CALL at Addresses:
|:004015CC , :0040176A , :00401D1A , :00401F39 , :004025E6//这里表明有 5 处调用这段子程序！所以改这里最方便！

|
:004046A0 51 push ecx
:004046A1 53 push ebx
:004046A2 55 push ebp
:004046A3 56 push esi
:004046A4 57 push edi
:004046A5 6A50 push 00000050
:004046A7 68A0C64000 push 0040C6A0

* Possible StringData Ref from Data Obj ->"UserKey"

|
:004046AC 6808964000 push 00409608
:004046B1 E81A030000 call 004049D0
:004046B6 83C40C add esp, 0000000C
:004046B9 83F810 cmp eax, 00000010//检查注册码是不是小于 16 位
:004046BC 7D08 jge 004046C6
:004046BE 33C0 xor eax, eax
:004046C0 5F pop edi
:004046C1 5E pop esi
:004046C2 5D pop ebp
:004046C3 5B pop ebx
:004046C4 59 pop ecx
:004046C5 C3 ret

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004046BC(C)

|
:004046C6 6A2F push 0000002F
:004046C8 6800CF4000 push 0040CF00

* Possible StringData Ref from Data Obj ->"UserName"

|



```
:004046CD 68F8954000                         push 004095F8
:004046D2 E8F9020000                         call 004049D0
:004046D7 83C40C                             add esp, 0000000C
:004046DA 83F805                             cmp eax, 00000005 // 检查用户名是不是小于 5 位
:004046DD 7D08                               jge 004046E7
:004046DF 33C0                               xor eax, eax
:004046E1 5F                                 pop edi
:004046E2 5E                                 pop esi
:004046E3 5D                                 pop ebp
:004046E4 5B                                 pop ebx
:004046E5 59                                 pop ecx
:004046E6 C3                                 ret
```

*   Referenced by a (U)nconditional or (C)onditional Jump at Address: `:004046DD(C)`

```
:004046E7 BFA0C64000                         mov edi, 0040C6A0 // 这里开始漫长的注册码计算和验证过程，我懒，不管它，直接找返回标志的地方
:004046EC 83C9FF                             or ecx, FFFFFFFF
:004046EF 33C0                               xor eax, eax
:004046F1 C605B1C6400000                     mov byte ptr [0040C6B1], 00
:004046F8 F2                                 repnz
:004046F9 AE                                 scasb
:004046FA F7D1                               not ecx
:004046FC 2BF9                               sub edi, ecx
:004046FE 8BC1                               mov eax, ecx
:00404700 8BF7                               mov esi, edi
:00404702 BFB4C64000                         mov edi, 0040C6B4
:00404707 C1E902                             shr ecx, 02
:0040470A F3                                 repz
:0040470B A5                                 movsd
:0040470C 8BC8                               mov ecx, eax
:0040470E 33C0                               xor eax, eax
:00404710 83E103                             and ecx, 00000003
:00404713 F3                                 repz
:00404714 A4                                 movsb
:00404715 BFA9C64000                         mov edi, 0040C6A9
:0040471A 83C9FF                             or ecx, FFFFFFFF
:0040471D F2                                 repnz
:0040471E AE                                 scasb
:0040471F F7D1                               not ecx
:00404721 2BF9                               sub edi, ecx
:00404723 8BD1                               mov edx, ecx
:00404725 8BF7                               mov esi, edi
:00404727 BFBCC64000                         mov edi, 0040C6BC
:0040472C C1E902                             shr ecx, 02
:0040472F F3                                 repz
:00404730 A5                                 movsd
:00404731 8BCA                               mov ecx, edx
:00404733 83E103                             and ecx, 00000003
:00404736 32DB                               xor bl, bl
:00404738 F3                                 repz
:00404739 A4                                 movsb
:0040473A BEA1C64000                         mov esi, 0040C6A1
:0040473F BFB4C64000                         mov edi, 0040C6B4
```

*   Referenced by a (U)nconditional or (C)onditional Jump at Address: `:00404763(C)`

```
:00404744 57                                 push edi
:00404745 E8E6010000                         call 00404930
:0040474A 8ACB                               mov cl, bl
:0040474C 83C404                             add esp, 00000004
:0040474F 80C150                             add cl, 50
:00404752 83C702                             add edi, 00000002
:00404755 32C1                               xor al, cl
:00404757 FEC3                               inc bl
:00404759 8846FF                             mov byte ptr [esi-01], al
:0040475C C60600                             mov byte ptr [esi], 00
:0040475F 46                                 inc esi
:00404760 80FB08                             cmp bl, 08
:00404763 72DF                               jb 00404744
:00404765 68B4C64000                         push 0040C6B4
:0040476A 68A0C64000                         push 0040C6A0
:0040476F E8EC010000                         call 00404960
:00404774 BF00CF4000                         mov edi, 0040CF00
:00404779 83C9FF                             or ecx, FFFFFFFF
:0040477C 33C0                               xor eax, eax
:0040477E 83C408                             add esp, 00000008
:00404781 F2                                 repnz
:00404782 AE                                 scasb
:00404783 F7D1                               not ecx
:00404785 2BF9                               sub edi, ecx
:00404787 33ED                               xor ebp, ebp
:00404789 8BD1                               mov edx, ecx
:0040478B 8BF7                               mov esi, edi
:0040478D BFBEC64000                         mov edi, 0040C6BE
:00404792 C1E902                             shr ecx, 02
:00404795 F3                                 repz
:00404796 A5                                 movsd
:00404797 8BCA                               mov ecx, edx
:00404799 83E103                             and ecx, 00000003
:0040479C F3                                 repz
:0040479D A4                                 movsb
:0040479E BF00CF4000                         mov edi, 0040CF00
:004047A3 83C9FF                             or ecx, FFFFFFFF
:004047A6 F2                                 repnz
:004047A7 AE                                 scasb
:004047A8 F7D1                               not ecx
:004047AA 49                                 dec ecx
:004047AB 80F908                             cmp cl, 08
:004047AE 884C2410                           mov byte ptr [esp+10], cl
:004047B2 7330                               jnb 004047E4
:004047B4 8B542410                           mov edx, dword ptr [esp+10]
:004047B8 BF00CF4000                         mov edi, 0040CF00
:004047BD 81E2FF000000                       and edx, 000000FF
:004047C3 83C9FF                             or ecx, FFFFFFFF
:004047C6 81C2BEC64000                       add edx, 0040C6BE
:004047CC F2                                 repnz
:004047CD AE                                 scasb
:004047CE F7D1                               not ecx
:004047D0 2BF9                               sub edi, ecx
:004047D2 8BC1                               mov eax, ecx
:004047D4 8BF7                               mov esi, edi
:004047D6 8BFA                               mov edi, edx
:004047D8 C1E902                             shr ecx, 02
:004047DB F3                                 repz
:004047DC A5                                 movsd
:004047DD 8BC8                               mov ecx, eax
:004047DF 83E103                             and ecx, 00000003
:004047E2 F3                                 repz
:004047E3 A4                                 movsb
```

*   Referenced by a (U)nconditional or (C)onditional Jump at Address: `:004047B2(C)`

```
:004047E4 C605C6C6400000                     mov byte ptr [0040C6C6], 00
:004047EB B9B4C64000                         mov ecx, 0040C6B4
:004047F0 BE08000000                         mov esi, 00000008
```

*   Referenced by a (U)nconditional or (C)onditional Jump at Address: `:00404819(C)`

```
:004047F5 8A01                               mov al, byte ptr [ecx]
:004047F7 3C20                               cmp al, 20
:004047F9 730E                               jnb 00404809
:004047FB 33D2                               xor edx, edx
:004047FD 25FF000000                         and eax, 000000FF
:00404802 8A510A                             mov dl, byte ptr [ecx+0A]
:00404805 0BD0                               or edx, eax
:00404807 EB0C                               jmp 00404815
```

*   Referenced by a (U)nconditional or (C)onditional Jump at Address: `:004047F9(C)`

```
:00404809 33D2                               xor edx, edx
:0040480B 25FF000000                         and eax, 000000FF
:00404810 8A510A                             mov dl, byte ptr [ecx+0A]
:00404813 33D0                               xor edx, eax
```

*   Referenced by a (U)nconditional or (C)onditional Jump at Address: `:00404807(U)`

```
:00404815 03EA                               add ebp, edx
:00404817 41                                 inc ecx
:00404818 4E                                 dec esi
:00404819 75DA                               jne 004047F5
:0040481B 33C0                               xor eax, eax // 这里向下就是返回的参数了，找于 al 相关的
:0040481D 5F                                 pop edi
:0040481E 85ED                               test ebp, ebp // 就是这里将 z 置 0 了，怎么才能让 ebp 为 0 呢，我不知道，不过上面有 xor eax, eax 这条指令，哈哈，把它变成 test eax,eax 吧，也就是 85ED=>85C0
:00404820 5E                                 pop esi
:00404821 5D                                 pop ebp
:00404822 0F94C0                             sete al // 在这里，程序让我们死翘翘了，向上看哪里把 z 这个 flag 置 0 了
:00404825 5B                                 pop ebx
:00404826 59                                 pop ecx
:00404827 C3                                 ret
```

拿出 `ultraedit`，将程序改了，运行测试，一切 ok，收工！

最后总结：



脱壳后查找：`33C0 5F 85ED 5E 5D`，改成：`33C0 5F 85C0 5E 5D`

`Cracker`: `Turkey/灭害灵`

`2002.3.25@18:00`

```c
#include <stdio.h>

#include <string.h>

#include <stdlib.h>

#include <time.h>

#include <conio.h>

void main(void)

{

    char * name="";

    char regname[8];

    char regcode[17];

    unsigned int table[16]={0x00,0x01,0x02,0x03,0x04,0x05,0x06,0x07,0x08,0x09,0x0A,0x0B,0x0C,0x0D,0x0E,0x0F};

    unsigned long eax;

    unsigned int bl,cl,dl=0xBB;

    int i,j,k,s,base;

    unsigned int inc1,inc2;

    int flag=0,success=1;

    clrscr();

    printf("\n********************************************************************************************\n");

    printf("APISPY32 V2.4 Registration Key Generator\n");

    printf("Author: ddcrack ( 2001/06/15 )\n");

    printf("********************************************************************************************\n\n");

    printf("Registration Name ( more than 5 letters ): ");

    scanf("%s",name);

    printf("\n");

    if(strlen(name)<5)

    do

    {

      printf("\nInput Registration Name again ( 5 letters at least ): ");

      scanf("%s",name);

      printf("\n");

    }

    while (strlen(name)<5);

    if(strlen(name)>=8)

      for(i=0;i<8;i++) regname[i]=*(name+i);

    else

      {

    for(i=0;i<strlen(name);i++) regname[i]=*(name+i);

    for(i=0;i<(8-strlen(name));i++) regname[strlen(name)+i]=*(name+i);

      }

    for(i=0;i<8;i++)

      { for(j=0;j<16;j++)

    { for(k=0;k<16;k++)

     { cl=table[j];

       cl=cl<<4;

       cl=cl|table[k];

       bl=0x50+(unsigned int)i;

       cl=cl^bl;

       eax=1;

       for(s=0;s<7;s++)

         { eax=eax*cl;

          if(eax>0x00008899) eax=eax%0x00008899;

         }

       eax=eax%dl;

       if(eax<0x00000020) continue;

       if(eax==(unsigned long)regname[i])

         {

          if(j<=9) inc1=0x30; else inc1=0x37;

          if(k<=9) inc2=0x30; else inc2=0x37;

          if(i<=3)

            {  regcode[2*i]=(char)(table[j]+inc1);

              regcode[2*i+1]=(char)(table[k]+inc2);

            }

          else

            {  regcode[2*i+1]=(char)(table[j]+inc1);

              regcode[2*i+2]=(char)(table[k]+inc2);

            }

          flag=1;

          break;

         }

       else flag=0;

     }

    if(flag==1) break;

      }

      if(flag==0)

    { printf("\nSorry! No such registration key which matchs your name.\n");

      success=0;

      break;

    }

    }

    if(success==1)

    { printf("Registration Key: ");

      randomize();

      base=random(16);

      if(base<=9) regcode[8]=(char)(base+0x30);

      else regcode[8]=(char)(base+0x37);

      for(i=0;i<17;i++) printf("%c",regcode[i]);

      printf("\n");

    }

    getch();

}
```

### 标题:看看我原来写的 `protectz` 的 keygen，就知道不用穷举了。(空)

*   作者：`[最弱智]`
*   时间：`2002-3-29 16:03:16`

详细信息:

> `RSA8` ;) `N=187`, `E=7`, `D=23`

>

> `4dd466d0e1e0cc123afbe504da2462ca`
