

# 压缩壳之 dePACK 壳分析

> 原文：[`www.pediy.com/kssd/pediy10/83585.html`](https://www.pediy.com/kssd/pediy10/83585.html)

前面已经有过几篇压缩壳的文章，接下来还是对于压缩壳的一点个人分析,![](img/f43c1e514327696dccbb0da997496b2a.jpg "扮鬼脸")   
壳开始代码的其中一部分  

代码:

```
loc\_46B003:                             ; CODE XREF: startj
.depack:0046B003                 pusha
.depack:0046B004                 push    offset dword\_401000 ; .text
.depack:0046B009                 push    0A181h
.depack:0046B00E                 call    DePack
.depack:0046B013                 push    offset unk\_44B000 ; .rdata
.depack:0046B018                 push    0C81h
.depack:0046B01D                 call    DePack
.depack:0046B022                 push    offset unk\_457000 ; .data
.depack:0046B027                 push    581h
.depack:0046B02C                 call    DePack
.depack:0046B031                 nop
.depack:0046B032                 jmp     short loc\_46B035

```

从这段代码中我们可以看到，这个壳是分区段分别压缩和解压的（注释）,进入 depack 的解压函数  
代码:

```
push    ebp
mov     ebp, esp
pusha
push    ebp
mov     esi, \[ebp+arg\_0\]
mov     edi, \[ebp+arg\_4\] ; 401000
call    sub\_46B04F
jmp     short loc\_46B053
DePack endp

```

loc\_46B053:  
addebx, 200h  
pushebx; 46b24d  
pushedi; 401000  
发现其实壳是将压缩后的区块放到 46b24d 中进行处理的  
  
DataCopy:               ; 将数据段中数据转移出来,到：46b24d  
moveax, \[edi\]  
mov     \[ebx\], eax  
addedi, 4          ; 下一 DWORD  
addebx, 4  
decesi; a181  
jnzshort DataCopy  ; 将数据段中数据转移出来,到：46b24d  
; ---------------------------------------------------------------------------  
.depack:0046B11B  
.depack:0046B11B loc\_46B11B:                             ; CODE XREF: DePack+B4j  
.depack:0046B11B                 deceax  
.depack:0046B11C                 shleax, 8          ; offs <<= 8;  
.depack:0046B11F                 moval, \[esi\]       ; offs += \*ud.source;  
.depack:0046B121                 incesi; \*ud.source++  
.depack:0046B122                 movebp, eax  
.depack:0046B124                 movecx, 1  
.depack:0046B129  
.depack:0046B129 loc\_46B129:                             ; CODE XREF: DePack:loc\_46B13Dj  
.depack:0046B129                 adddl, dl  
.depack:0046B12B                 jnzshort loc\_46B132  
.depack:0046B12D                 movdl, \[esi\]  
.depack:0046B12F                 incesi  
.depack:0046B130                 adcdl, dl  
.depack:0046B132  
.depack:0046B132 loc\_46B132:                             ; CODE XREF: DePack+EEj  
.depack:0046B132                 adcecx, ecx  
.depack:0046B134                 adddl, dl  
.depack:0046B136                 jnzshort loc\_46B13D  
.depack:0046B138                 movdl, \[esi\]  
.depack:0046B13A                 incesi  
.depack:0046B13B                 adcdl, dl  
.depack:0046B13D  
.depack:0046B13D loc\_46B13D:                             ; CODE XREF: DePack+F9j  
.depack:0046B13D                 jbshort loc\_46B129  
.depack:0046B13F                 cmpeax, 7D00h      ; 32000  
.depack:0046B144                 jnbshort loc\_46B160 ;  if (offs >= 32000) len++;  
.depack:0046B146                 cmpeax, 500h       ; if (offs >= 1280) len++;  
.depack:0046B14B                 jbshort loc\_46B15B ; if (offs < 128) len += 2;  
.depack:0046B14D                 incecx  
.depack:0046B14E                 pushesi  
.depack:0046B14F                 movesi, edi  
.depack:0046B151                 subesi, eax  
.depack:0046B153                 repmovsb  
.depack:0046B155                 popesi  
.depack:0046B156                 jmp     nexttag         ; call getbit  
.depack:0046B15B ; ---------------------------------------------------------------------------  
.depack:0046B15B  
.depack:0046B15B loc\_46B15B:                             ; CODE XREF: DePack+10Ej  
.depack:0046B15B                 cmpeax, 7Fh        ; if (offs < 128) len += 2;  
.depack:0046B15E                 jashort loc\_46B163  
.depack:0046B160  
.depack:0046B160 loc\_46B160:                             ; CODE XREF: DePack+107j  
.depack:0046B160                 addecx, 2          ; len += 2;  
.depack:0046B163  
.depack:0046B163 loc\_46B163:                             ; CODE XREF: DePack+121j  
.depack:0046B163                 pushesi  
.depack:0046B164                 movesi, edi  
.depack:0046B166                 subesi, eax  
.depack:0046B168                 repmovsb  
.depack:0046B16A                 popesi  
.depack:0046B16B                 jmp     nexttag         ; call getbit  
.depack:0046B170 ; ---------------------------------------------------------------------------  
.depack:0046B170  
.depack:0046B170 loc\_46B170:                             ; CODE XREF: DePack:loc\_46B094j  
.depack:0046B170                 moval, \[esi\]  
.depack:0046B172                 incesi  
.depack:0046B173                 xorecx, ecx  
.depack:0046B175                 shral, 1  
.depack:0046B178                 jzshort loc\_46B18C  
.depack:0046B17A                 adcecx, 2  
.depack:0046B17D                 movebp, eax  
.depack:0046B17F                 pushesi; domatch  
.depack:0046B180                 movesi, edi  
.depack:0046B182                 subesi, eax  
.depack:0046B184                 repmovsb  
.depack:0046B186                 popesi  
.depack:0046B187                 jmp     nexttag         ; call getbit  
.depack:0046B18C ; ---------------------------------------------------------------------------  
.depack:0046B18C  
.depack:0046B18C loc\_46B18C:                             ; CODE XREF: DePack+13Bj  
.depack:0046B18C                 popebp  
.depack:0046B18D                 subedi, \[ebp+arg\_4\]  
.depack:0046B190                 mov     \[ebp+var\_4\], edi  
.depack:0046B193                 popa  
.depack:0046B194                 popebp  
.depack:0046B195                 retn  
  
一个和经典压缩算法了  
getbit：  
  
代码:

```
   /\* check if tag is empty \*/
    if (!ud->bitcount--)
    {
        /\* load next tag \*/
        ud->tag = \*ud->source++;
        ud->bitcount = 7;
    }

    /\* shift bit out of tag \*/
    bit = (ud->tag >> 7) & 0x01;
    ud->tag <<= 1;

    return bit\[color=#008000\];

```

\---------------------------------------------------------------------------  
代码:

```
     if (aP\_getbit(&ud))
        {
            if (aP\_getbit(&ud))
            {
                if (aP\_getbit(&ud))
                {
                    offs = 0;

                    for (i = 4; i; i--) offs = (offs << 1) + aP\_getbit(&ud);

                    if (offs)
                    {
                        \*ud.destination = \*(ud.destination - offs);
                        ud.destination++;
                    } else {
                        \*ud.destination++ = 0x00;
                    }

                    LWM = 0;

                } else {

                    offs = \*ud.source++;

                    len = 2 + (offs & 0x0001);

                    offs >>= 1;

                    if (offs)
                    {
                        for (; len; len--)
                        {
                            \*ud.destination = \*(ud.destination - offs);
                            ud.destination++;
                        }
                    } else done = 1;

                    R0 = offs;
                    LWM = 1;
                }

            } else {

                offs = aP\_getgamma(&ud);

                if ((LWM == 0) && (offs == 2))
                {
                    offs = R0;

                    len = aP\_getgamma(&ud);

                    for (; len; len--)
                    {
                        \*ud.destination = \*(ud.destination - offs);
                        ud.destination++;
                    }

                } else {

                    if (LWM == 0) 
            offs -= 3; 
            else 
            offs -= 2;

                    offs <<= 8;
                    offs += \*ud.source++;

                    len = aP\_getgamma(&ud);

                    if (offs >= 32000) len++;
                    if (offs >= 1280) len++;
                    if (offs < 128) len += 2;

                    for (; len; len--)
                    {
                        \*ud.destination = \*(ud.destination - offs);
                        ud.destination++;
                    }

                    R0 = offs;
                }

                LWM = 1;
            }

        } else {

            \*ud.destination++ = \*ud.source++;
            LWM = 0;
        }

```

以上的几个壳都是很“规矩”的壳，以后将对反调试 复杂变换等的壳进行学习 ![](img/022c916b9660e467a8663bd57a4ae6d1.jpg "狂笑")  
好好学习，天天进步！