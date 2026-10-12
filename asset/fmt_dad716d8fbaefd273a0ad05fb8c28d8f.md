
====>ECX=0 OR D4310000=D4310000

:00405E15 03C1                add eax, ecx
①、  ====>EAX=000000BD + D4310000=D43100BD
……省 略……

:00405E17 3528181827        xor eax, 27181828
①、  ====>EAX=D43100BD XOR 27181828=F3291895
……省 略……

③、  ====>EAX=CCC9581B

:00405E1C 47                inc edi
:00405E1D 8BF0              mov esi, eax
:00405E1F 8B442410          mov eax, dword ptr [esp+10]
:00405E23 3B78F8            cmp edi, dword ptr [eax-08]
:00405E26 7CCC              jl 00405DF4
====>循环用户名位数次

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00405DF2(C)
|

:00405E28 6815CD5B07        push 075BCD15
:00405E2D E8B2AB0000        call 004109E4
:00405E32 83C404            add esp, 00000004

* Possible Reference to Dialog: DialogID_0064
|

* Possible Reference to Dialog: DialogID_7801, CONTROL_ID:0064, ""
|

:00405E35 BF64000000        mov edi, 00000064

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00405E53(C)
|

:00405E3A E8B2AB0000        call 004109F1
:00405E3F 8BD6              mov edx, esi
:00405E41 C1EA10            shr edx, 10
①、  ====>EDX=CCC9581B SHR 10=0000CCC9
……省 略……

:00405E44 C1E610            shl esi, 10
①、  ====>ESI=CCC9581B SHL 10=581B0000
……省 略……

:00405E47 0BD6              or edx, esi
①、  ====>EDX=0000CCC9 OR 581B0000=581BCCC9
……省 略……

:00405E49 03C2              add eax, edx
①、  ====>EAX=000033CB + 581BCCC9=581C0094
……省 略……

:00405E4B 3508053217        xor eax, 17320508
①、  ====>EAX=581C0094 XOR 17320508=4F2E059C
……省 略……

:00405E50 4F                dec edi
:00405E51 8BF0              mov esi, eax
:00405E53 75E5              jne 00405E3A
====>循环 100 次  EAX=F08417C0

:00405E55 8D4C2410          lea ecx, dword ptr [esp+10]
:00405E59 E83AFA0200        call 00435898
:00405E5E 8BC6              mov eax, esi
====>EAX=F08417C0

:00405E60 5F                pop edi
:00405E61 5E                pop esi
:00405E62 C3                ret

—————————————————————————————————

进入：`00405E06`  `call 004109F1`

* Referenced by a CALL at Addresses:
|:00404F45   , :00404F51   , :00405E06   , :00405E3A   , :00408899   
|:0040B6D5   
|

:004109F1 E8BD390000        call 004143B3
:004109F6 8B4814            mov ecx, dword ptr [eax+14]
:004109F9 69C9FD430300      imul ecx, 000343FD
①、  ====>ECX=14142153 * 000343FD=8096A807

:004109FF 81C1C39E2600      add ecx, 00269EC3
①、  ====>ECX=8096A807 + 00269EC3=80BD46CA

:00410A05 894814            mov dword ptr [eax+14], ecx
:00410A08 8BC1              mov eax, ecx
:00410A0A C1E810            shr eax, 10
①、  ====>EAX=80BD46CA SHR 10=000080BD

:00410A0D 25FF7F0000        and eax, 00007FFF
①、  ====>EAX=000080BD AND 00007FFF=000000BD

:00410A12 C3                ret

—————————————————————————————————

【算 法  总 结】：

对用户名简单运算，然后经过 100 次的循环得出注册码。

算法麻烦，写这点东西只是说说下面的如何让程序自动保存真的注册码。

—————————————————————————————————

【自动保存真码】：

:004064A7 3BC7              cmp eax, edi
:004064A9 7564              jne 0040650F

把上面的代码改为：

:004064A7 8BF8              mov edi,eax //把真注册码移入 EDI    
:004064A9 90                //把这个跳转 NOP 掉    
:004064AA 90 
:004064CF 57                push edi    //这里就自动保存了    

—————————————————————————————————

【KeyMake 之{109th}内存注册机】：

中断地址：`004064A7`

中断次数：`1`

第一字节：`3B`

指令长度：`2`

寄存器方式：`EDI`      


十进制值

—————————————————————————————————

【注册信息保存】：

```
REGEDIT4

[HKEY_CURRENT_USERSoftwareGeYong  Software 空当接龙工具 Options]

"UserName"="fly"

"RegisterCode"=dword:f08417c0
```

—————————————————————————————————

【整        理】：

信息码：`401012`

用户名：`fly`

注册码：`4035188672`

—————————————————————————————————

         ,     _/  
        /| _.-~/            _     ,        青春都一饷  
       ( /~   /             ~-. |  
       `\\  _/                ~ )          忍把浮名  
   _-~~~-.)  )_;;,.          _  //'    
  /'\_,   --~    ~~~-  ,;__,(  (.-~~~-.        换了破解轻狂  
 \`~ __( ,_..-- (     ,;'' /    ~--   /._`  
  /~~//'   /' `~         ) /--..\_, )_  `~    
  "  `~"  "      `"      /~'`    `\\~~   

            "     "   "~'  ""

            Cracked By 巢水工作坊——fly [OCN][FCG]

               2003-09-06  14:00

`4dd466d0e1e0cc123afbe504da2462ca`
