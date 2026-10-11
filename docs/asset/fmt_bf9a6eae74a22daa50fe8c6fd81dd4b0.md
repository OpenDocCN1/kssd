
`0041332E C0E302 shl bl, 02 -------------------------->bl 逻辑左移 2`

`00413331 02CB add cl, bl -------------------------->cl=cl+bl`

`00413333 8AD8 mov bl, al -------------------------->al 给 bl`

`00413335 880F mov byte ptr [edi], cl -------------->cl 给[edi]`

`00413337 8A06 mov al, byte ptr [esi] -------------->取下一个注册码给 al`

`00413339 47 inc edi ----------------------------->edi 加一`

`0041333A 50 push eax`

`0041333B 46 inc esi ----------------------------->esi 加一`

`0041333C E86FFFFFFF call 004132B0 ----------------------->换算`

`00413341 8AD0 mov dl, al -------------------------->al 给 dl`

`00413343 2403 and al, 03 -------------------------->al and 3`

`00413345 C0FA02 sar dl, 02 -------------------------->dl 算术右移 2`

`00413348 C0E304 shl bl, 04 -------------------------->bl 逻辑左移 4`

`0041334B 02D3 add dl, bl -------------------------->dl=dl+bl`

`0041334D 8AD8 mov bl, al -------------------------->bl 给 al`

`0041334F 8817 mov byte ptr [edi], dl -------------->dl 给[edi]`

`00413351 8A06 mov al, byte ptr [esi] -------------->取下一个注册码给 al`

`00413353 47 inc edi ----------------------------->edi 加一`

`00413354 50 push eax`

`00413355 46 inc esi ----------------------------->esi 加一`

`00413356 E855FFFFFF call 004132B0 ----------------------->换算`

`0041335B C0E306 shl bl, 06 -------------------------->bl 逻辑左移 6`

`0041335E 02C3 add al, bl -------------------------->al=al+bl`

`00413360 83C410 add esp, 00000010`

`00413363 8807 mov byte ptr [edi], al -------------->al 给 edi`

`00413365 8A06 mov al, byte ptr [esi] -------------->取下一个注册码给 al`

`00413367 47 inc edi ----------------------------->edi 加一`

`00413368 84C0 test al, al ------------------------->检查是否取完`

`0041336A 75A9 jne 00413315 ------------------------>没有取完，循环！`

`0041336C 5B pop ebx`

`0041336D 8807 mov byte ptr [edi], al -------------->将最后一个给[edi]`

`0041336F 5F pop edi`

`00413370 B801000000 mov eax, 00000001`

`00413375 5E pop esi`

`00413376 C3 ret`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

`:0041330E(C)`

`00413377 8B7C240C mov edi, dword ptr [esp+0C]`

`0041337B B801000000 mov eax, 00000001`

`00413380 C60700 mov byte ptr [edi], 00`

`00413383 5F pop edi`

`00413384 5E pop esi`

`00413385 C3 ret`

************************************************************

这个 CALL 作用是：如果注册码在 41 和 5A 之间，则注册码减 41；

如果注册码在 61 和 71 之间，则注册码减 47；

如果注册码在 30 和 39 之间，则注册码加 4。

* Referenced by a CALL at Addresses:

`:00413317 , :00413322 , :0041333C , :00413356`

`004132B0 8A442404 mov al, byte ptr [esp+04]`

`004132B4 3C41 cmp al, 41`

`004132B6 7C08 jl 004132C0`

`004132B8 3C5A cmp al, 5A`

`004132BA 7F04 jg 004132C0`

`004132BC 83E841 sub eax, 00000041`

`004132BF C3 ret`

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

`:004132B6(C), :004132BA(C)`

`004132C0 3C61 cmp al, 61`

`004132C2 7C08 jl 004132CC`

`004132C4 3C7A cmp al, 7A`

`004132C6 7F04 jg 004132CC`

`004132C8 83E847 sub eax, 00000047`

`004132CB C3 ret`

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

`:004132C2(C), :004132C6(C)`

`004132CC 3C30 cmp al, 30`

`004132CE 7C08 jl 004132D8`

`004132D0 3C39 cmp al, 39`

`004132D2 7F04 jg 004132D8`

`004132D4 83C004 add eax, 00000004`

`004132D7 C3 ret`

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:

`:004132CE(C), :004132D2(C)`

`004132D8 3C2B cmp al, 2B`

`004132DA 0F95C0 setne al`

`004132DD 83C03E add eax, 0000003E`

`004132E0 C3 ret`

由于本人的能力有限，做不出它的注册机，还请高人指点！！！！！

我算了一个

Name: `jackyspy [CCG]`

Serial: `lZll-V2nT-VGlW-ZZWV`

写了一个注册机, 算法有人贴了, 我就不再说了.

```c
/*
软件名称: 棋隐(Chinese Chess Learner)

在 VC 6.0 下通过编译

*/

#include <stdio.h>

#include <windows.h>

BYTE anti_convert(BYTE c)

{
    c-=0x30;
    if(c<=0xD)
        c=c*2+0x65;
    else if(c<=0x1E)
        c=0x5A-(BYTE)((c<<24)/0x555555+0.5);
    return c;
}

BYTE convert2char(BYTE c)

{
    if(c<26)
        c+='A';
    else if(c<52)
        c+='G';
    else if(c<62)
        c-=4;
    else if(c==62)
        c='+';
    else
        c='*';
    return c;
}

void main()

{
    BYTE result[]="@2NAME748748",serial[]="0000000000000000";
                // 其中 result 中的数值可以改变, 但是一定要遵循下面的规则:
                // 要求字符为'0'-'N'之间或者 0x8B-0x94 之间, 第二个数为 2,后六位必须是数字
                // 而且要大于 1000000(0x186A0)
    BYTE ResortRef[]={2,8,3,7,0x0A,9,0x0F,0,6,1,0x0B,0x0D,4,0x0C,5,0x0E,0};
    int i,temp;
    for(i=1,temp=0;i<12;i++)
        temp=temp*i+result[i];
    temp=temp>=0?temp:-temp;
    temp%=0x1A;temp+=0x41;
    result[0]=(BYTE)temp;
    for(i=1;i<12;i++)
        result[i]=anti_convert(result[i]);
    for(i=0;i<4;i++)
    {
        serial[i*4]=(result[i*3]&0xFC)>>2;
        serial[i*4+1]=((result[i*3+1]&0xF0)>>4)+((result[i*3]&3)<<4);
        serial[i*4+2]=((result[i*3+1]&0x0F)<<2)+((result[i*3+2]&0xC0)>>6);
        serial[i*4+3]=result[i*3+2]&0x3F;
    }
    for(i=0;i<16;i++)
        serial[i]=convert2char(serial[i]);
    for(i=0;i<16;i++)
    {
        putchar(serial[ResortRef[i]]);
        if(i%4==3)
            putchar('-');
    }
}
```

`4dd466d0e1e0cc123afbe504da2462ca`
