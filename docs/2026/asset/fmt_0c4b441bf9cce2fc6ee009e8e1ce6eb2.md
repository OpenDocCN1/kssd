
```
004F3263    .  E>call <jmp.&MSVBVM60.\_\_vbaStrMove>

004F3268    .  5>push eax

004F3269    .  E>call <jmp.&MSVBVM60.#616>                 ;  计算出来的字符串的前20位

004F326E    .  8>mov edx,eax

004F3270    .  8>lea ecx,dword ptr ss:[ebp-2C]

004F3273    .  E>call <jmp.&MSVBVM60.\_\_vbaStrMove>

004F3278    .  8>lea eax,dword ptr ss:[ebp-40]

004F327B    .  5>push eax

004F327C    .  8>lea eax,dword ptr ss:[ebp-3C]

004F327F    .  5>push eax

004F3280    .  6>push 2

004F3282    .  E>call <jmp.&MSVBVM60.\_\_vbaFreeStrList>

004F3287    .  8>add esp,0C

004F328A    .  F>push dword ptr ss:[ebp-38]

004F328D    .  F>push dword ptr ss:[ebp-2C]

004F3290    .  6>push 1

004F3292    .  E>call <jmp.&MSVBVM60.\_\_vbaStrComp>         ;  用去掉组合好的与“-”的输入码与计算出来的比较了！

…………………………………………………………………………………………………………………………

进入call stmagic.004F2F98

…………………………………………………………………………………………………………………………

006CD2BC    .  8>mov eax,dword ptr ss:[ebp-70]

006CD2BF    .  C>mov dword ptr ds:[eax],6A09E667      ;  固定值   用于起始就算和最后计算注册码的

006CD2C5    .  8>mov eax,dword ptr ss:[ebp-70]

006CD2C8    .  8>lea ecx,dword ptr ss:[ebp-3D0]

006CD2CE    .  C>mov dword ptr ds:[eax+4],BB67AE85    ;  固定值   用于起始就算和最后计算注册码的

006CD2D5    .  8>mov eax,dword ptr ss:[ebp-70]

006CD2D8    .  5>push ecx

006CD2D9    .  C>mov dword ptr ds:[eax+8],3C6EF372    ;  固定值   用于起始就算和最后计算注册码的

006CD2E0    .  8>mov eax,dword ptr ss:[ebp-70]

006CD2E3    .  F>push dword ptr ss:[ebp+C]

006CD2E6    .  C>mov dword ptr ds:[eax+C],A54FF53A    ;  固定值   用于起始就算和最后计算注册码的

006CD2ED    .  8>mov eax,dword ptr ss:[ebp-70]

006CD2F0    .  5>push esi

006CD2F1    .  C>mov dword ptr ds:[eax+10],510E527F   ;  固定值   用于起始就算和最后计算注册码的

006CD2F8    .  8>mov eax,dword ptr ss:[ebp-70]

006CD2FB    .  C>mov dword ptr ds:[eax+14],9B05688C   ;  固定值   用于起始就算和最后计算注册码的

006CD302    .  8>mov eax,dword ptr ss:[ebp-70]

006CD305    .  C>mov dword ptr ds:[eax+18],1F83D9AB   ;  固定值   用于起始就算和最后计算注册码的

006CD30C    .  8>mov eax,dword ptr ss:[ebp-70]

006CD30F    .  C>mov dword ptr ds:[eax+1C],5BE0CD19   ;  固定值   用于起始就算和最后计算注册码的

006CD316    .  8>mov eax,dword ptr ds:[esi]

006CD318    .  F>call dword ptr ds:[eax+50]           ;  计算与输入码一起的固定值

006CD31B    .  8>lea eax,dword ptr ss:[ebp-3D0]       ;  出来后只是把每段开头组合

……………………………………………………………………………………………………………………

call dword ptr ds:[eax+50]           ;  计算与输入码一起的固定值

……………………………………………………………………………………………………………………

006CCF19    > /3>cmp ecx,dword ptr ss:[ebp-20]

006CCF1C    . |8>mov eax,ecx

006CCF1E    . |6>push 4                               ;  固定值4

006CCF20    . |9>cdq

006CCF21    . |5>pop edi

006CCF22    . |0>jge stmagic.006CD001

006CCF28    . |F>idiv edi

006CCF2A    . |6>push 3

006CCF2C    . |C>mov dword ptr ss:[ebp-48],2

006CCF33    . |C>mov dword ptr ss:[ebp-68],4008

006CCF3A    . |8>mov dword ptr ss:[ebp-28],eax

006CCF3D    . |8>mov eax,ecx

006CCF3F    . |9>cdq

006CCF40    . |F>idiv edi

006CCF42    . |5>pop edi

006CCF43    . |6>push 1

006CCF45    . |5>pop eax

006CCF46    . |8>mov dword ptr ss:[ebp-40],eax

006CCF49    . |2>sub edi,edx

006CCF4B    . |8>mov edx,dword ptr ss:[ebp+C]

006CCF4E    . |0>jo stmagic.006CD105

006CCF54    . |6>imul edi,edi,8

006CCF57    . |0>jo stmagic.006CD105

006CCF5D    . |8>mov dword ptr ss:[ebp-60],edx

006CCF60    . |8>lea edx,dword ptr ss:[ebp-48]

006CCF63    . |0>add ecx,eax

006CCF65    . |5>push edx

006CCF66    . |0>jo stmagic.006CD105

006CCF6C    . |8>lea eax,dword ptr ss:[ebp-68]

006CCF6F    . |5>push ecx

006CCF70    . |5>push eax

006CCF71    . |8>lea eax,dword ptr ss:[ebp-58]

006CCF74    . |5>push eax

006CCF75    . |E>call <jmp.&MSVBVM60.#632>

006CCF7A    . |8>lea eax,dword ptr ss:[ebp-58]

006CCF7D    . |5>push eax

006CCF7E    . |8>lea eax,dword ptr ss:[ebp-38]

006CCF81    . |5>push eax

006CCF82    . |E>call <jmp.&MSVBVM60.\_\_vbaStrVarVal>  ;  得到各个组合的头输入码

006CCF87    . |5>push eax

006CCF88    . |E>call <jmp.&MSVBVM60.#693>            ;  得到各个头输入码的ASCII码

006CCF8D    . |0>movzx eax,al

006CCF90    . |8>lea ecx,dword ptr ss:[ebp-38]

006CCF93    . |8>mov dword ptr ss:[ebp-24],eax

006CCF96    . |E>call <jmp.&MSVBVM60.\_\_vbaFreeStr>

006CCF9B    . |8>lea eax,dword ptr ss:[ebp-58]

006CCF9E    . |5>push eax

006CCF9F    . |8>lea eax,dword ptr ss:[ebp-48]

006CCFA2    . |5>push eax

006CCFA3    . |6>push 2

006CCFA5    . |E>call <jmp.&MSVBVM60.\_\_vbaFreeVarList>

006CCFAA    . |8>mov edx,dword ptr ds:[ebx]

006CCFAC    . |8>add esp,0C

006CCFAF    . |8>lea eax,dword ptr ss:[ebp-7C]

006CCFB2    . |8>mov ecx,edi

006CCFB4    . |5>push eax

006CCFB5    . |8>mov dword ptr ss:[ebp-88],edx

006CCFBB    . |E>call <jmp.&MSVBVM60.\_\_vbaI2I4>

006CCFC0    . |5>push eax

006CCFC1    . |8>mov eax,dword ptr ss:[ebp-88]

006CCFC7    . |F>push dword ptr ss:[ebp-24]           ;  压入各个头输入吗的ASCII

006CCFCA    . |5>push ebx

006CCFCB    . |F>call dword ptr ds:[eax+24]

006CCFCE    . |F>push dword ptr ss:[ebp-28]

006CCFD1    . |F>push dword ptr ss:[ebp-30]

006CCFD4    . |E>call <jmp.&MSVBVM60.\_\_vbaDerefAry1>

006CCFD9    . |F>push dword ptr ss:[ebp-28]

006CCFDC    . |8>mov edi,dword ptr ds:[eax]

006CCFDE    . |0>or edi,dword ptr ss:[ebp-7C]

006CCFE1    . |F>push dword ptr ss:[ebp-30]

006CCFE4    . |E>call <jmp.&MSVBVM60.\_\_vbaDerefAry1>

006CCFE9    . |8>mov dword ptr ds:[eax],edi

006CCFEB    . |8>mov eax,dword ptr ss:[ebp-34]

006CCFEE    . |8>add eax,1

006CCFF1    . |0>jo stmagic.006CD105

006CCFF7    . |8>mov dword ptr ss:[ebp-34],eax

006CCFFA    . |8>mov ecx,eax

006CCFFC    .^\E>jmp stmagic.006CCF19                 ;  上面有输入码取一部分的个数次循环为的是把取出的注册码组合起来

006CD001    >  F>idiv edi                             ;  用取出的输入码个数除以4

006CD003    .  6>push 4

006CD005    .  8>mov edi,dword ptr ds:[ebx]

006CD007    .  8>mov dword ptr ss:[ebp-28],eax        ;  得到的商储存在EBP-28

006CD00A    .  8>mov eax,ecx                          ;  得到次数

006CD00C    .  9>cdq

006CD00D    .  5>pop ecx

006CD00E    .  F>idiv ecx                             ;  用取出的输入码个数除以4 为了得到余数

006CD010    .  6>push 3                               ;  固定值3

006CD012    .  8>lea eax,dword ptr ss:[ebp-7C]        ;  存储地址

006CD015    .  5>pop ecx                              ;  得到固定值3

006CD016    .  5>push eax

006CD017    .  2>sub ecx,edx                          ;  固定值3减去   次数除以的4的余数

006CD019    .  0>jo stmagic.006CD105

006CD01F    .  6>imul ecx,ecx,8                       ;  被减数乘以8

006CD022    .  0>jo stmagic.006CD105

006CD028    .  E>call <jmp.&MSVBVM60.\_\_vbaI2I4>

006CD02D    .  5>push eax                             ;  得到刚计算的乘积 作为固定值取值！！！

006CD02E    .  6>push 80                              ;  函数中被处理的数！

006CD033    .  5>push ebx

006CD034    .  F>call dword ptr ds:[edi+24]           ;  计算函数

006CD037    .  F>push dword ptr ss:[ebp-28]

006CD03A    .  F>push dword ptr ss:[ebp-30]

006CD03D    .  E>call <jmp.&MSVBVM60.\_\_vbaDerefAry1>

006CD042    .  F>push dword ptr ss:[ebp-28]           ;  个数除以4的商

006CD045    .  8>mov edi,dword ptr ds:[eax]           ;  得到输入码段数据，没有满4个的ASCII码

006CD047    .  0>or edi,dword ptr ss:[ebp-7C]         ;  与刚才得到的数据进行OR运算

006CD04A    .  F>push dword ptr ss:[ebp-30]

006CD04D    .  E>call <jmp.&MSVBVM60.\_\_vbaDerefAry1>

006CD052    .  8>lea ecx,dword ptr ss:[ebp-7C]        ;  下面函数存储地址

006CD055    .  8>mov dword ptr ds:[eax],edi           ;  储存在与输入码一起的地方

006CD057    .  8>mov eax,dword ptr ds:[ebx]

006CD059    .  5>push ecx                             ;  得到存放地址

006CD05A    .  6>push 3                               ;  固定值产生了与输入码一起的固定值

006CD05C    .  F>push dword ptr ss:[ebp-20]           ;  得到参与运算输入码的个数

006CD05F    .  5>push ebx

006CD060    .  F>call dword ptr ds:[eax+24]           ;  参与运算输入吗的个数与1FFFFFFF  AND运算，并乘以00000008 作为输入吗的第15个（从0开始）倒数第1个

006CD063    .  8>mov eax,esi

006CD065    .  8>sub eax,1

006CD068    .  0>jo stmagic.006CD105

006CD06E    .  5>push eax

006CD06F    .  F>push dword ptr ss:[ebp-30]

006CD072    .  E>call <jmp.&MSVBVM60.\_\_vbaDerefAry1>

006CD077    .  8>mov ecx,dword ptr ss:[ebp-7C]        ;  得到计算出来的值
```
