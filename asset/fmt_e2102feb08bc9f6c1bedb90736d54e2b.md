
```
0054BB09   8B45F4                 mov     eax, [ebp-$0C]   //新 sn=12445-12345-12345-12345-12345  

0054BB0C   0FB64012               movzx   eax, byte ptr [eax+$12]  //sn[18]  

0054BB10   8B55F4                 mov     edx, [ebp-$0C]  

0054BB13   0FB65216               movzx   edx, byte ptr [edx+$16]  //sn[22]  

0054BB17   2BC2                   sub     eax, edx                   //sn[18]-sn[22]  

0054BB19   99                     cdq  

0054BB1A   33C2                   xor     eax, edx  

0054BB1C   2BC2                   sub     eax, edx  

0054BB1E   8BD8                   mov     ebx, eax                   //和上面的一样 绝对值  所以下面就不再多说了  

0054BB20   83C330                 add     ebx, +$30 

0054BB23   83FB39                 cmp     ebx, +$39  

0054BB26   7E08                   jle     0054BB30  

0054BB28   83FB41                 cmp     ebx, +$41  

0054BB2B   7D03                   jnl     0054BB30  

0054BB2D   83C307                 add     ebx, +$07  

0054BB30   8D45F4                 lea     eax, [ebp-$0C]  

|  

0054BB33   E8FC90EBFF             call    00404C34     

0054BB38   885808                 mov     [eax+$08], bl      //  bl 相减结果=>sn[8]  

0054BB3B   8B45F4                 mov     eax, [ebp-$0C]     //  新 sn=12445-12445-12345-12345-12345  

0054BB3E   0FB6400C               movzx   eax, byte ptr [eax+$0C]  //sn[12]  

0054BB42   8B55F4                 mov     edx, [ebp-$0C]  

0054BB45   0FB65210               movzx   edx, byte ptr [edx+$10]  //sn[16]  

0054BB49   2BC2                   sub     eax, edx                   //sn[12]-sn[16]  

0054BB4B   99                     cdq  

0054BB4C   33C2                   xor     eax, edx  

0054BB4E   2BC2                   sub     eax, edx  

........  

........  

|  

0054BB65   E8CA90EBFF             call    00404C34  

0054BB6A   88580E                 mov     [eax+$0E], bl              // bl 相减结果=>sn[14]  

0054BB6D   8B45F4                 mov     eax, [ebp-$0C]             //新 sn=12445-12445-12445-12345-12345  

0054BB70   0FB64006               movzx   eax, byte ptr [eax+$06]  //sn[6]  

0054BB74   8B55F4                 mov     edx, [ebp-$0C]  

0054BB77   0FB6520A               movzx   edx, byte ptr [edx+$0A]  //sn[10]  

0054BB7B   2BC2                   sub     eax, edx                   //sn[6]-sn[10]  

0054BB7D   99                     cdq  

.........  

...........  

0054BB97   E89890EBFF             call    00404C34  

0054BB9C   885814                 mov     [eax+$14], bl            // bl 相减结果=>sn[20]  

0054BB9F   8B45F4                 mov     eax, [ebp-$0C]            // 新 sn=12445-12445-12445-12445-12345  

0054BBA2   0FB600                 movzx   eax, byte ptr [eax]    //sn[0]  

0054BBA5   8B55F4                 mov     edx, [ebp-$0C]  

0054BBA8   0FB65204               movzx   edx, byte ptr [edx+$04] //sn[4]  

0054BBAC   2BC2                   sub     eax, edx                 //sn[0]-sn[4]  

0054BBAE   99                     cdq  

0054BBAF   33C2                   xor     eax, edx  

......  

......  

0054BBC8   E86790EBFF             call    00404C34  

0054BBCD   88581A                 mov     [eax+$1A], bl              // bl 相减结果=>sn[26]  

0054BBD0   8B45F4                 mov     eax, [ebp-$0C]              // 新 sn=12445-12445-12445-12445-12445  

0054BBD3   0FB64008               movzx   eax, byte ptr [eax+$08]   //sn[8]  

0054BBD7   8B55F4                 mov     edx, [ebp-$0C]  

0054BBDA   0FB6520A               movzx   edx, byte ptr [edx+$0A]   //sn[10]  

0054BBDE   2BC2                   sub     eax, edx                   //sn[8]-sn[10]  

0054BBE0   99                     cdq  

.......  

.......  

0054BBFA   E83590EBFF             call    00404C34  

0054BBFF   885801                 mov     [eax+$01], bl            // bl 相减结果=>sn[1]  

0054BC02   8B45F4                 mov     eax, [ebp-$0C]            // 新 sn=11445-12445-12445-12445-12445  

0054BC05   0FB64014               movzx   eax, byte ptr [eax+$14] //sn[20]  

0054BC09   8B55F4                 mov     edx, [ebp-$0C] 

0054BC0C   0FB65216               movzx   edx, byte ptr [edx+$16] //sn[22]  

0054BC10   2BC2                   sub     eax, edx                 //sn[20]-sn[22]  

0054BC12   99                     cdq  

.........  

........  

0054BC2C   E80390EBFF             call    00404C34  

0054BC31   885807                 mov     [eax+$07], bl           // bl 相减结果=>sn[7]  

0054BC34   8B45F4                 mov     eax, [ebp-$0C]          // 新 sn=11445-11445-12445-12445-12445  

0054BC37   0FB6400E               movzx   eax, byte ptr [eax+$0E] //sn[14]  

0054BC3B   8B55F4                 mov     edx, [ebp-$0C]  

0054BC3E   0FB65210               movzx   edx, byte ptr [edx+$10] //sn[16]  

0054BC42   2BC2                   sub     eax, edx                 //sn[14]-sn[16]  

0054BC44   99                     cdq  

.........  

.........  

0054BC5E   E8D18FEBFF             call    00404C34  

0054BC63   88580D                 mov     [eax+$0D], bl         //bl 相减结果=>sn[13]  

0054BC66   8B45F4                 mov     eax, [ebp-$0C]        // 新 sn=11445-11445-11445-12445-12445  

0054BC69   0FB64002               movzx   eax, byte ptr [eax+$02]  //sn[2]  

0054BC6D   8B55F4                 mov     edx, [ebp-$0C]  

0054BC70   0FB65204               movzx   edx, byte ptr [edx+$04]  //sn[4]  

0054BC74   2BC2                   sub     eax, edx                   //sn[2]-sn[4]  

0054BC76   99                     cdq  

00.........  

...........  

|  

0054BC90   E89F8FEBFF             call    00404C34  

0054BC95   885813                 mov     [eax+$13], bl           //bl 相减结果=>sn[19]  

0054BC98   8B45F4                 mov     eax, [ebp-$0C]          // 新 sn=11445-11445-11445-11445-12445  

0054BC9B   0FB6401A               movzx   eax, byte ptr [eax+$1A] //sn[26]  

0054BC9F   8B55F4                 mov     edx, [ebp-$0C]  

0054BCA2   0FB6521C               movzx   edx, byte ptr [edx+$1C]  //sn[28]  

0054BCA6   2BC2                   sub     eax, edx                   //sn[26]-sn[28]  

0054BCA8   99                     cdq  

..........  

.........  

0054BCC2   E86D8FEBFF             call    00404C34  

0054BCC7   885819                 mov     [eax+$19], bl              //bl 相减结果=>sn[25]  

0054BCCA   8B45F4                 mov     eax, [ebp-$0C]              // 新 sn=11445-11445-11445-11445-11445  

0054BCCD   0FB64013               movzx   eax, byte ptr [eax+$13]   //sn[19]  

0054BCD1   8B55F4                 mov     edx, [ebp-$0C] 

0054BCD4   0FB65212               movzx   edx, byte ptr [edx+$12]   //sn[18]  

0054BCD8   2BC2                   sub     eax, edx                   //sn[19]-sn[18]  

........  

........  

0054BCF4   E83B8FEBFF             call    00404C34  

0054BCF9   885803                 mov     [eax+$03], bl              //bl 相减结果=>sn[3]  

0054BCFC   8B45F4                 mov     eax, [ebp-$0C]             //新 sn=11405-11445-11445-11445-11445  

0054BCFF   0FB64018               movzx   eax, byte ptr [eax+$18]  //sn[24]  

0054BD03   8B55F4                 mov     edx, [ebp-$0C]  

0054BD06   0FB65219               movzx   edx, byte ptr [edx+$19]  //sn[25]  

0054BD0A   2BC2                   sub     eax, edx                   //sn[24]-sn[25]  

...........  

...........  

0054BD26   E8098FEBFF             call    00404C34  

0054BD2B   885809                 mov     [eax+$09], bl            //bl 相减结果=>sn[9]  

0054BD2E   8B45F4                 mov     eax, [ebp-$0C]           //新 sn=11405-11405-11445-11445-11445  

0054BD31   0FB64006               movzx   eax, byte ptr [eax+$06] //sn[6]  

0054BD35   8B55F4                 mov     edx, [ebp-$0C]  

0054BD38   0FB65207               movzx   edx, byte ptr [edx+$07] //sn[7]  
```
