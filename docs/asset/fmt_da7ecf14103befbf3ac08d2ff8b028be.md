
```
004092EF   push        offset string "mode != NULL" (004281d8)  

004092F4   push        0  

004092F6   push        48h  

004092F8   push        offset string "\_open.c" (00428d04)  

004092FD   push        2  

004092FF   call        \_CrtDbgReport (00408800)  

00409304   add         esp,14h  

00409307   cmp         eax,1  

0040930A   jne         \_openfile+6Dh (0040930d)  

0040930C   int         3  

0040930D   xor         edx,edx  

0040930F   test        edx,edx  

00409311   jne         \_openfile+49h (004092e9)  

00409313   cmp         dword ptr [str],0  

00409317   jne         \_openfile+97h (00409337)  

00409319   push        offset string "str != NULL" (00428154)  

0040931E   push        0  

00409320   push        49h  

00409322   push        offset string "\_open.c" (00428d04)  

00409327   push        2  

00409329   call        \_CrtDbgReport (00408800)  

0040932E   add         esp,14h  

00409331   cmp         eax,1  

00409334   jne         \_openfile+97h (00409337)  

00409336   int         3  

00409337   xor         eax,eax  

00409339   test        eax,eax  

0040933B   jne         \_openfile+73h (00409313)  

0040933D   mov         ecx,dword ptr [mode]  

00409340   mov         dl,byte ptr [ecx]  

00409342   mov         byte ptr [ebp-20h],dl  

00409345   cmp         byte ptr [ebp-20h],61h  

00409349   je          \_openfile+0DCh (0040937c)  

0040934B   cmp         byte ptr [ebp-20h],72h  

0040934F   je          \_openfile+0B9h (00409359)  

00409351   cmp         byte ptr [ebp-20h],77h  

00409355   je          \_openfile+0CAh (0040936a)  

00409357   jmp         \_openfile+0EEh (0040938e)  

00409359   mov         dword ptr [modeflag],0  

00409360   mov         eax,dword ptr [streamflag]  

00409363   or          al,1  

00409365   mov         dword ptr [streamflag],eax  

00409368   jmp         \_openfile+0F5h (00409395)  

0040936A   mov         dword ptr [modeflag],301h  

00409371   mov         ecx,dword ptr [streamflag]  

00409374   or          ecx,2  

00409377   mov         dword ptr [streamflag],ecx  

0040937A   jmp         \_openfile+0F5h (00409395)  

0040937C   mov         dword ptr [modeflag],109h  

00409383   mov         edx,dword ptr [streamflag]  

00409386   or          edx,2  

00409389   mov         dword ptr [streamflag],edx  

0040938C   jmp         \_openfile+0F5h (00409395)  

0040938E   xor         eax,eax  

00409390   jmp         $L1486+0A0h (004095b6)  

00409395   mov         dword ptr [whileflag],1  
//这里开始进入迷茫区,是一个大的循环.(不要放弃!!核心部分即将出现!! )  

0040939C   mov         eax,dword ptr [mode]  

0040939F   add         eax,1  

004093A2   mov         dword ptr [mode],eax  

004093A5   mov         ecx,dword ptr [mode]  

004093A8   movsx       edx,byte ptr [ecx]  

004093AB   test        edx,edx  

004093AD   je          $L1486+29h (0040953f)  

004093B3   cmp         dword ptr [whileflag],0  

004093B7   je          $L1486+29h (0040953f)  

004093BD   mov         eax,dword ptr [mode]  

004093C0   movsx       ecx,byte ptr [eax]  

004093C3   mov         dword ptr [ebp-24h],ecx  

004093C6   mov         edx,dword ptr [ebp-24h]  

004093C9   sub         edx,2Bh  

004093CC   mov         dword ptr [ebp-24h],edx  

004093CF   cmp         dword ptr [ebp-24h],49h  

004093D3   ja          $L1486+1Dh (00409533)  

004093D9   mov         ecx,dword ptr [ebp-24h]  

004093DC   xor         eax,eax  

004093DE   mov         al,byte ptr  (004095e5)[ecx]  

004093E4   jmp         dword ptr [eax*4+4095BDh]  

$L1462:  

004093EB   mov         edx,dword ptr [modeflag]  

004093EE   and         edx,2  

004093F1   test        edx,edx  

004093F3   je          $L1462+13h (004093fe)  

004093F5   mov         dword ptr [whileflag],0  

004093FC   jmp         $L1462+35h (00409420)  

004093FE   mov         eax,dword ptr [modeflag]  

00409401   or          al,2  

00409403   mov         dword ptr [modeflag],eax  

00409406   mov         ecx,dword ptr [modeflag]  

00409409   and         ecx,0FEh  

0040940C   mov         dword ptr [modeflag],ecx  

0040940F   mov         edx,dword ptr [streamflag]  

00409412   or          dl,80h  

00409415   mov         dword ptr [streamflag],edx  

00409418   mov         eax,dword ptr [streamflag]  

0040941B   and         al,0FCh  

0040941D   mov         dword ptr [streamflag],eax  

00409420   jmp         $L1486+24h (0040953a)  

$L1465:  

00409425   mov         ecx,dword ptr [modeflag]  

00409428   and         ecx,0C000h  

0040942E   test        ecx,ecx  

00409430   je          $L1465+16h (0040943b)  

00409432   mov         dword ptr [whileflag],0  

00409439   jmp         $L1465+1Fh (00409444)  

0040943B   mov         edx,dword ptr [modeflag]  

0040943E   or          dh,80h  

00409441   mov         dword ptr [modeflag],edx  

00409444   jmp         $L1486+24h (0040953a)  

$L1468:  

00409449   mov         eax,dword ptr [modeflag]  

0040944C   and         eax,0C000h  

00409451   test        eax,eax  

00409453   je          $L1468+15h (0040945e)  

00409455   mov         dword ptr [whileflag],0  

0040945C   jmp         $L1468+1Eh (00409467)  

0040945E   mov         ecx,dword ptr [modeflag]  

00409461   or          ch,40h  

00409464   mov         dword ptr [modeflag],ecx  

00409467   jmp         $L1486+24h (0040953a)  

$L1471:  

0040946C   cmp         dword ptr [commodeset],0  

00409470   je          $L1471+0Fh (0040947b)  

00409472   mov         dword ptr [whileflag],0  

00409479   jmp         $L1471+1Fh (0040948b)  

0040947B   mov         dword ptr [commodeset],1  

00409482   mov         edx,dword ptr [streamflag]  

00409485   or          dh,40h  

00409488   mov         dword ptr [streamflag],edx  

0040948B   jmp         $L1486+24h (0040953a)  

$L1474:  

00409490   cmp         dword ptr [commodeset],0  

00409494   je          $L1474+0Fh (0040949f)  

00409496   mov         dword ptr [whileflag],0  

0040949D   jmp         $L1474+1Fh (004094af)  

0040949F   mov         dword ptr [commodeset],1  

004094A6   mov         eax,dword ptr [streamflag]  

004094A9   and         ah,0BFh  

004094AC   mov         dword ptr [streamflag],eax  

004094AF   jmp         $L1486+24h (0040953a)  

$L1477:  

004094B4   cmp         dword ptr [scanset],0  

004094B8   je          $L1477+0Fh (004094c3)  

004094BA   mov         dword ptr [whileflag],0  

004094C1   jmp         $L1477+1Fh (004094d3)  

004094C3   mov         dword ptr [scanset],1  

004094CA   mov         ecx,dword ptr [modeflag]  

004094CD   or          ecx,20h  

004094D0   mov         dword ptr [modeflag],ecx  

004094D3   jmp         $L1486+24h (0040953a)  

$L1480:  

004094D5   cmp         dword ptr [scanset],0  

004094D9   je          $L1480+0Fh (004094e4)  

004094DB   mov         dword ptr [whileflag],0  

004094E2   jmp         $L1480+1Fh (004094f4)  

004094E4   mov         dword ptr [scanset],1  

004094EB   mov         edx,dword ptr [modeflag]  

004094EE   or          edx,10h  

004094F1   mov         dword ptr [modeflag],edx  

004094F4   jmp         $L1486+24h (0040953a)  

$L1483:  

004094F6   mov         eax,dword ptr [modeflag]  

004094F9   and         eax,1000h  

004094FE   test        eax,eax  

00409500   je          $L1483+15h (0040950b)  

00409502   mov         dword ptr [whileflag],0  

00409509   jmp         $L1483+1Eh (00409514)  

0040950B   mov         ecx,dword ptr [modeflag]  

0040950E   or          ch,10h  

00409511   mov         dword ptr [modeflag],ecx  

00409514   jmp         $L1486+24h (0040953a)  

$L1486:  

00409516   mov         edx,dword ptr [modeflag]  

00409519   and         edx,40h  

0040951C   test        edx,edx  

0040951E   je          $L1486+13h (00409529)  

00409520   mov         dword ptr [whileflag],0  

00409527   jmp         $L1486+1Bh (00409531)  
```
