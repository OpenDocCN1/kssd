
`push ebp`  
`:004043E1` `57`  
`push edi`  

*  
Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:00404428(C)  
|  
`:004043E2`  
`8BC3` `mov`  
`eax, ebx`  
`:004043E4` `B906000000` `mov`  
`ecx, 00000006`  
`:004043E9` `99`  
`cdq`  
`:004043EA` `F7F9`  
`idiv ecx`  
`:004043EC` `8B442410`  
`mov eax, dword ptr [esp+10]`  
`====>EAX=[esp+10]=231C281C`  
`:004043F0`  
`8A0C03` `mov cl, byte`  
`ptr [ebx+eax]`  
`====>依次取231C281C字符的HEX值`  
`:004043F3`  
`8AC3` `mov`  
`al, bl`  
`====>AL=BL`  
`:004043F5`  
`83C202` `add edx,`  
`00000002`  
`:004043F8` `FEC0`  
`inc al`  
`====>AL 加 1`  
`:004043FA`  
`F6E9` `imul`  
`cl`  
`1、 ====>AL=01 * 32=32`  
`2、`  
`====>AL=02 * 33=66`  
`3、 ====>AL=03 * 31=93`  
`4、 ====>AL=04 * 43=0C`  
`5、 ====>AL=05`  
`* 32=FA`  
`6、 ====>AL=06 * 38=50`  
`7、`  
`====>AL=07 * 31=57`  
`8、 ====>AL=08 * 43=18`  
`:004043FC`  
`84C0` `test`  
`al, al`  
`:004043FE` `7D02`  
`jge 00404402`  
`:00404400` `F6D8`  
`neg al`  
`3、 ====>AL=93`  
`NEG=6D`  
`5、 ====>AL=FA NEG=06`  
*  
Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:004043FE(C)  
|  
`:00404402`  
`0FBEC0` `movsx eax,`  
`al`  
`====>EAX=AL`  
`:00404405`  
`8ACA` `mov`  
`cl, dl`  
`====>CL=DL`  
`:00404407`  
`8BF8` `mov`  
`edi, eax`  
`====>EDI=EAX`  
`:00404409`  
`FEC9` `dec`  
`cl`  
`====>CL 减 1`  
`:0040440B`  
`8BE8` `mov`  
`ebp, eax`  
`====>EBP=EAX`  
`:0040440D`  
`D3FF` `sar`  
`edi, cl`  
`1、 ====>EDI=00000032 SAR 01=00000019`  
`2、 ====>EDI=00000066 SAR 02=00000019`  
`3、`  
`====>EDI=0000006D SAR 03=0000000D`  
`4、 ====>EDI=0000000C`  
`SAR 04=00000000`  
`5、 ====>EDI=00000006 SAR 05=00000000`  
`6、 ====>EDI=00000050 SAR 06=00000001`  
`7、`  
`====>EDI=00000057 SAR 01=0000002B`  
`8、 ====>EDI=00000018`  
`SAR 02=00000006`  
`:0040440F`  
`8ACA` `mov`  
`cl, dl`  
`====>CL=DL`  
`:00404411`  
`D3FD` `sar`  
`ebp, cl`  
`1、 ====>EBP=00000032 SAR 02=0000000C`  
`2、 ====>EBP=00000066 SAR 03=0000000C`  
`3、`  
`====>EBP=0000006D SAR 04=00000006`  
`4、 ====>EBP=0000000C`  
`SAR 05=00000000`  
`5、 ====>EBP=00000006 SAR 06=00000000`  
`6、 ====>EBP=00000050 SAR 07=00000000`  
`7、`  
`====>EBP=00000057 SAR 02=00000015`  
`8、 ====>EBP=00000018`  
`SAR 03=00000003`  
`:00404413`  
`8BCA` `mov`  
`ecx, edx`  
`:00404415` `8B542418`  
`mov edx, dword ptr [esp+18]`  
`:00404419` `D3E0`  
`shl eax, cl`  
`1、`  
`====>EAX=00000032 SHL 02=000000C8`  
`2、 ====>EAX=00000066`  
`SHL 03=00000330`  
`3、 ====>EAX=0000006D SHL 04=000006D0`  
`4、 ====>EAX=0000000C SHL 05=00000180`  
`5、`  
`====>EAX=00000006 SHL 06=00000180`  
`6、 ====>EAX=00000050`  
`SHL 07=00002800`  
`7、 ====>EAX=00000057 SHL 02=0000015C`  
`8、 ====>EAX=00000018 SHL 03=000000C0`  
`:0040441B`  
`33FD` `xor`  
`edi, ebp`  
`1、 ====>EDI=00000019 XOR 0000000C=00000015`  
`2、 ====>EDI=00000019 XOR 0000000C=00000015`  
`3、`  
`====>EDI=0000000D XOR 00000006=0000000B`  
`4、`  
`====>EDI=00000000 XOR 00000000=00000000`  
`5、 ====>EDI=00000000`  
`XOR 00000000=00000000`  
`6、 ====>EDI=00000001 XOR 00000000=00000001`  
`7、 ====>EDI=0000002B XOR 00000015=0000003E`  
`8、`  
`====>EDI=00000006 XOR 00000003=00000005`  
`:0040441D`  
`33F8` `xor`  
`edi, eax`  
`1、 ====>EDI=00000015 XOR 000000C8=000000DD`  
`2、 ====>EDI=00000015 XOR 00000330=00000325`  
`3、`  
`====>EDI=0000000B XOR 000006D0=000006DB`  
`4、`  
`====>EDI=00000000 XOR 00000180=00000180`  
`5、 ====>EDI=00000000`  
`XOR 00000180=00000180`  
`6、 ====>EDI=00000001 XOR 00002800=00002801`  
`7、 ====>EDI=0000003E XOR 0000015C=00000162`  
`8、`  
`====>EDI=00000005 XOR 000000C0=000000C5`  
`:0040441F`  
`03D7` `add`  
`edx, edi`  
`1、 ====>EDX=231C281C ADD 000000DD=231C28F9`  
`2、 ====>EDX=231C28F9 ADD 00000325=231C2C1E`  
`3、`  
`====>EDX=231C2C1E ADD 000006DB=231C32F9`  
`4、`  
`====>EDX=231C32F9 ADD 00000180=231C3479`  
`5、 ====>EDX=231C3479`  
`ADD 00000180=231C35F9`  
`6、 ====>EDX=231C35F9 ADD 00002801=231C5DFA`  
`7、 ====>EDX=231C5DFA ADD 00000162=231C5F5C`  
`8、`  
`====>EDX=231C5F5C ADD 000000C5=231C6021`  
呵呵，好了，循环的结果231C6021就是我的注册码了！  
`:00404421`  
`43` `inc`  
`ebx`  
`:00404422` `3BDE`  
`cmp ebx, esi`  
`:00404424` `89542418`  
`mov dword ptr [esp+18], edx`  
`====>[esp+18]=EDX`  
`:00404428`  
`7CB8` `jl 004043E2`  
`====>继续循环？！`  
`:0040442A`  
`5F` `pop`  
`edi`  
`:0040442B` `5D`  
`pop ebp`  
*  
Referenced by a (U)nconditional or (C)onditional Jump at Address:  
|:004043DE(C)  
|  
`:0040442C`  
`8B4C2410` `mov ecx, dword`  
`ptr [esp+10]`  
`====>ECX=231C6021`  
`:00404430`  
`8D542408` `lea edx, dword`  
`ptr [esp+08]`  
`:00404434` `51`  
`push ecx`  
*  
Possible StringData Ref from Data Obj ->"%X"  
|  
`:00404435` `6854144100`  
`push 00411454`  
`:0040443A` `52`  
`push edx`  
*  
Reference To: MFC42.Ordinal:0B02, Ord:0B02h  
|  
`:0040443B`  
`E8E0670000` `Call 0040AC20`  
`:00404440`  
`83C40C` `add esp,`  
`0000000C`  
`:00404443` `8D4C2408`  
`lea ecx, dword ptr [esp+08]`  
*  
Reference To: MFC42.Ordinal:106C, Ord:106Ch  
|  
`:00404447`  
`E8E6670000` `Call 0040AC32`  
`:0040444C`  
`8D4C2408` `lea ecx, dword`  
`ptr [esp+08]`  
* Reference  
To: MFC42.Ordinal:188A, Ord:188Ah  
|  
`:00404450`  
`E8CF660000` `Call 0040AB24`  
`:00404455`  
`8D4C2408` `lea ecx, dword ptr [esp+08]`  
*  
Reference To: MFC42.Ordinal:188B, Ord:188Bh  
|  
`:00404459`  
`E8C0660000` `Call 0040AB1E`  
`:0040445E`  
`8B742428` `mov esi, dword`  
`ptr [esp+28]`  
`:00404462` `8D442408`  
`lea eax, dword ptr [esp+08]`  
`:00404466` `50`  
`push eax`  
`:00404467` `8BCE`  
`mov ecx, esi`  
*  
Reference To: MFC42.Ordinal:0217, Ord:0217h  
|  
`:00404469`  
`E816670000` `Call 0040AB84`  
`:0040446E`  
`C744241401000000` `mov [esp+14], 00000001`  
`:00404476`  
`8D4C2408` `lea ecx, dword ptr [esp+08]`  
`:0040447A` `C644242000`  
`mov [esp+20], 00`  
*  
Reference To: MFC42.Ordinal:0320, Ord:0320h  
|  
`:0040447F`  
`E834660000` `Call 0040AAB8`  
`:00404484`  
`8B4C2418` `mov ecx, dword ptr [esp+18]`  
`:00404488` `8BC6`  
`mov eax, esi`  
`:0040448A` `5E`  
`pop esi`  
`:0040448B` `5B`  
`pop ebx`  
`:0040448C`  
`64890D00000000` `mov dword ptr fs:[00000000],`  
`ecx`  
`:00404493` `83C41C`  
`add esp, 0000001C`  
`:00404496` `C20400`  
`ret 0004`  
―――――――――――――――――――――――――――――――――  

【完 美 爆 破】：  
1、`0040464E`  
`85C0` `test`  
`eax, eax`  
改为： `33C0`  
`xor eax, eax`  
呵呵，使其“正确”设置404650处的BL值！  
2、`0040465D`  
`8B542414` `mov edx, dword`  
`ptr [esp+14]`  
改为： `8B142490`  
`mov edx, dword ptr [esp]` 补一个NOP  
呵呵，让程序把真的注册码保存起来，这样程序就自动显示真码了，相当于用程序本身做了个注册机！  
―――――――――――――――――――――――――――――――――  

【KeyMake之{63th}内存注册机】：  
中断地址：`0040463F`  
中断次数：1  
第一字节：`50`  
指令长度：1  
内存方式：`EAX`  
说明：把内存注册机放到`C:\WINDOWS\SYSTEM`目录下，然后用杀进程的工具把`C:\WINDOWS\SYSTEM\sgame.exe`的进程禁止掉！再运行内存注册机，按热键激活创想游戏控制软件，随意输入试炼码，确定后即可弹出正确注册码！  
―――――――――――――――――――――――――――――――――  

【注册信息保存】：  
`C:\WINDOWS`  
下的`Win.ini`中：  
[TrueThink]  
Password=C7  
RegisterCode=231C6021  
