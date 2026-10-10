

# Obsidium 脱壳手记\[申请邀请码\]

> 原文：[`www.pediy.com/kssd/pediy11/125848.html`](https://www.pediy.com/kssd/pediy11/125848.html)

【文章标题】: Obsidium 脱壳手记  
【文章作者】: imagic  
【作者邮箱】: afeiw@163.com  
【作者主页】: http://imagic.tk  
【作者 QQ 号】: 99970077  
【软件名称】: CrackMe  
【加壳方式】: Obsidium 1.3.4.2  
【保护方式】: 全保护  
【编写语言】: delphi  
【使用工具】: OD  
【操作平台】: WIN XP SP3  
  
【详细过程】  
Obsidium 的壳比较变态的地方在于模拟了一些系统 API 函数，还有一些 API 没有模拟但是已经事先被壳给出运行结果，例如：  
GetVersion,GetCommandLine 等函数，但是大不部分的 API 地址都能写脚本跟踪出来，这里，我是在到达 OEP 后来修复 IAT 的，  
大致步骤如下：  
1.到达 OEP，用内存断点可以直接到达伪 OEP，头部被偷了一段  
2.OEP 断下后，随便找个 API 进入 IAT 领空，记下 IAT 的段首和段尾，然后跟踪下 API，一路 F8，不出意外就会看到解密的 API。  
当然你有可能跟踪的正好是一个模拟的 API，那样就很难辨别了，这属于人品问题。  
3.基本上跟出第一个 API 后，写个脚本，将 API 还原，这样就可以搞定一大部分的 API。  
4.剩下几个模拟的 API，我跟的方法是这样：在 IAT 的地址下断，断下后返回到调用 CALL，然后在调用 CALL 后面下断，此时  
分析下压入的参数，然后 SHIFT+F9，到 CALL 后断下，分析返回值。十有八九就能知道这个 CALL 调用的 API。  
5.最后找个 DELPHI 的程序对照下头，修复被偷的几个字节。  
  
注意：dump 的时候记得先改掉内存的属性，不然不让 DUMP。  
  
详细过程如下：  
0046C000 > /EB 02           jmp short CrackMe\_.0046C004  
0046C002   |6A FD           push -3  
0046C004   \\E8 26000000     call CrackMe\_.0046C02F  
0046C009    EB 03           jmp short CrackMe\_.0046C00E  
  
载入后，ALT+M 到代码段下内存访问断点，SHIFT+F9  
  
0046E1E2    C702 33C0C300   mov dword ptr ds:\[edx\],0C3C033  
0046E1E8    EB 02           jmp short CrackMe\_.0046E1EC  
0046E1EA    0BDE            or ebx,esi  
  
断下后，ALT+M 到资源段下内存访问断点，SHIFT+F9  
  
0096AD64    8B02            mov eax,dword ptr ds:\[edx\]  
0096AD66    85C0            test eax,eax  
0096AD68    74 13           je short 0096AD7D  
0096AD6A    A9 00000080     test eax,80000000  
0096AD6F    75 04           jnz short 0096AD75  
  
断下后，继续到代码段下内存的断点,SHIFT+F9  
  
0044D32B    E8 348DFBFF     call CrackMe\_.00406064                        ; THIS IS OEP  
0044D330    A1 B8EF4400     mov eax,dword ptr ds:\[44EFB8\]  
0044D335    8B00            mov eax,dword ptr ds:\[eax\]  
0044D337    E8 C8E5FFFF     call CrackMe\_.0044B904  
  
刚好断在 DELPHI 程序的第一个 CALL，前面被偷了一些字节。一般 DELPHI 程序里第一个 CALL 里有一个 GetModuleHandle 的调用  
F7 跟进，跟进前找到 IAT 的段首和段尾 00451114-004516D4  
  
009B02F0    60              pushad  
009B02F1    9C              pushfd  
009B02F2    66:BD 73D5      mov bp,0D573  
009B02F6    B3 81           mov bl,81  
009B02F8   E9 1949FBFF     jmp 00964C16  
009B02FD    60              pushad  
  
一路 F8 下去  
  
00964D58    6A 01           push 1  
00964D5A    50              push eax  
00964D5B    6A 00           push 0  
00964D5D    FF76 04         push dword ptr ds:\[esi+4\]  
00964D60    FF37            push dword ptr ds:\[edi\]  
00964D62    FF53 54         call dword ptr ds:\[ebx+54\]  
  
跟到这里就到 API 了，这个 CALL 返回后 EAX 就是 API 地址了，需要注意的是像这样获取 API 的 CALL，类型有三个，不同 API，CALL  
有时候会不一样，这个多试着跟踪几个 API 就会发现。  
得到 API 地址后，将 API 地址写会 IAT。  
  
具体的 IAT 还原可以参照下脚本。  
  
这步 IAT 完成后，剩下的几个自己慢慢跟，基本上都能出来。  
  
最后修复被偷代码，找个 DELPHI 程序对比下，发现少了几行  
  
0044D320 > $  55            push ebp  
0044D321   .  8BEC          mov ebp,esp  
0044D323   .  83C4 F0       add esp,-10  
0044D326   .  B8 40D14400   mov eax,CrackMe.0044D140  
  
找个空位补上，然后改下内存属性，dump， 然后抓取 IAT，保存完事。  
  
  
【经验总结】  
跟踪壳的一部分代码后，发现可以用内存断点到达 OEP。  
  
壳开头的一段异常很有意思，记录下：  
  
0046C000 > /EB 02           jmp short CrackMe\_.0046C004  
0046C002   |6A FD           push -3  
0046C004   \\E8 26000000     call CrackMe\_.0046C02F  
0046C009    EB 03           jmp short CrackMe\_.0046C00E  
0046C00B    CE              into  
0046C00C    FE              ???                                           ; 未知命令  
0046C00D    B5 EB           mov ch,0EB  
0046C00F    0105 8B54240C   add dword ptr ds:\[C24548B\],eax  
0046C015    EB 02           jmp short CrackMe\_.0046C019  
0046C017    6923 8382B800   imul esp,dword ptr ds:\[ebx\],0B88283  
0046C01D    0000            add byte ptr ds:\[eax\],al  
0046C01F    24 EB           and al,0EB  
0046C021    0387 A68D33C0   add eax,dword ptr ds:\[edi+C0338DA6\]  
0046C027    EB 01           jmp short CrackMe\_.0046C02A  
0046C029    2D C3EB02B1     sub eax,B102EBC3  
0046C02E    88EB            mov bl,ch  
0046C030    02ED            add ch,ch  
0046C032    36:             prefix ss:  
0046C033    64:67:FF36 0000 push dword ptr fs:\[0\]  
0046C039    EB 03           jmp short CrackMe\_.0046C03E  
0046C03B    CA E661         retf 61E6  
0046C03E    64:67:8926 0000 mov dword ptr fs:\[0\],esp  
0046C044    EB 03           jmp short CrackMe\_.0046C049  
0046C046    98              cwde  
0046C047    61              popad  
0046C048  ^ E0 EB           loopdne short CrackMe\_.0046C035  
0046C04A    0373 FD         add esi,dword ptr ds:\[ebx-3\]  
0046C04D    2C 50           sub al,50  
0046C04F    EB 04           jmp short CrackMe\_.0046C055  
0046C051    2307            and eax,dword ptr ds:\[edi\]  
0046C053    E5 09           in eax,9  
0046C055    33C0            xor eax,eax  
0046C057    EB 03           jmp short CrackMe\_.0046C05C  
0046C059    90              nop  
0046C05A    B5 B2           mov ch,0B2  
0046C05C    8B00            mov eax,dword ptr ds:\[eax\]  
0046C05E    EB 03           jmp short CrackMe\_.0046C063  
0046C060    F4              hlt  
0046C061    68 C6C3EB03     push 3EBC3C6  
0046C066    A9 740CE9FA     test eax,FAE90C74  
0046C06B    0000            add byte ptr ds:\[eax\],al  
0046C06D    00EB            add bl,ch  
0046C06F    03BF ED2DE8D5   add edi,dword ptr ds:\[edi+D5E82DED\]  
0046C075    FFFF            ???                                           ; 未知命令  
0046C077    FFEB            jmp far ebx                                   ; 非法使用寄存器  
0046C079    0141 EB         add dword ptr ds:\[ecx-15\],eax  
0046C07C    036A EE         add ebp,dword ptr ds:\[edx-12\]  
0046C07F    3358 EB         xor ebx,dword ptr ds:\[eax-15\]  
0046C082    04 5D           add al,5D  
0046C084    34 91           xor al,91  
0046C086    A2 EB042A28     mov byte ptr ds:\[282A04EB\],al  
0046C08B    A7              cmps dword ptr ds:\[esi\],dword ptr es:\[edi\]  
0046C08C    5A              pop edx  
0046C08D    64:67:8F06 0000 pop dword ptr fs:\[0\]                          ; 异常出口  
0046C093    EB 04           jmp short CrackMe\_.0046C099  
0046C095    9D              popfd  
0046C096    44              inc esp  
0046C097    1D FA83C404     sbb eax,4C483FA  
0046C09C    EB 01           jmp short CrackMe\_.0046C09F  
0046C09E    3BE8            cmp ebp,eax  
0046C0A0    C3              retn  
  
\-  
/////////////////////////////////  
//Obsidium 1.3.4.2 参考脚本  
//可能有不完善之处，仅供参考  
////////////////////////////////  
  
VAR IAT\_Start  
var IAT\_End  
var IAT\_CurrentAddr  
var Des\_Addr  
var cmpData  
MOV IAT\_Start ,451114  //IAT 起始地址  
MOV  IAT\_CurrentAddr,451114  
MOV IAT\_End , 4516D4  //IAT 结束地址  
  
bprm 401000,FF  
ESTO  
bpmc 401000,FF  
bprm  464000,FFF  
ESTO  
bpmc 464000,FFF  
bprm 401000,62000  
ESTO  
bpmc 401000,62000  
cmt eip,”THIS IS OEP”  
  
ADD IAT\_CurrentAddr,4  
mov Des\_Addr,\[IAT\_CurrentAddr\]  
mov eip,Des\_Addr  
sto  
sto  
sto  
sto  
sto  
mov Des\_Addr,eip  
find eip,#506A00FF7604FF37FF5354#  
mov Des\_Addr,$RESULT  
add Des\_Addr,8  
bp Des\_Addr  
find eip,#52515068342CE0496A00FF5318#  
mov Des\_Addr,$RESULT  
add Des\_Addr,a  
bp Des\_Addr  
find eip,#6A006A456A0068CC971025FF37FF5354#  
mov Des\_Addr,$RESULT  
add Des\_Addr,d  
bp Des\_Addr                //上面是寻找解密 CALL 特征，然后下断  
  
jmp lable1  
loop1:  
ADD IAT\_CurrentAddr,4  
mov Des\_Addr,\[IAT\_CurrentAddr\]  
cmp IAT\_CurrentAddr,IAT\_End  
jae lable2  
mov eax,0  
mov ax,\[Des\_Addr\]  
cmp eax,9c60  
JNZ loop1  
cmp Des\_Addr,9b05bb  
JZ loop1  
mov eip,Des\_Addr  
lable1:  
ESTO  
sto  
mov \[IAT\_CurrentAddr\],eax  
Jmp loop1  
lable2:  
mov \[4516D4\],0  
MSG “修复 IAT 完成”  
RET