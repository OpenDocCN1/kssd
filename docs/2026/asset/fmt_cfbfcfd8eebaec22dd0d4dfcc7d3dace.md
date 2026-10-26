
# PeX V0.99b脱壳――`PeX.exe`主程序

- 下载页面：`http://protools.anticrack.de/packers.htm` 以前DOWN的 ^O^
- 软件大小：45 KB

## 软件简介

code,data,import compression(based on `APLIB v0.26b` by Joergen Ibsen)&encryption；new technique was developed to increase compression ratio； protection against cracking&reverse engeenering； `bpx` protection； import table handling；advanced import table protection.

## 作者声明

初学Crack，只是感兴趣，没有其它目的。失误之处敬请诸位大侠赐教！

## 破解工具

`Ollydbg1.09`、`PEiD`、`LordPE`、`Import REConstructor V1.4.2+`

## 脱壳过程

PeX 壳应该是罕见的，至少我没见过用PeX保护的软件。既然loveboom兄弟提出来我就学习一下吧。呵呵，偶是脱壳白痴，碰不了猛壳，只好看看名不见经传的小壳，^O^ 但是这个壳却不算太弱呀，首次跟踪一不小心就OVER了。

下面的调试环境是：WinXP + `Ollydbg`

```
00408000     E9 F5000000          jmp pex.004080FA

                                  ====>进入OD后断在这！F7进入！

00408005     0D 0AC4C4C4          or eax,C4C4C40A
```

这种入口方式是固定的，很容易辨认  ^O^

这下面就要小心了，如果用F7走就很容易迷入陷阱！呵呵，所以这次采取另外的方法啦。

可以看见下面的`call 0040XXXX`，用F4走过去，但是这样还是要F4很多次，并且不小心就晕入循环！好了，往下看，使劲向下看：`popad`、`retn`在`004082FA`处，直接在`004082FA`下断，F9，砰，睁开眼睛，高兴的看见安全着陆啦！ :-) 这是对付PeX壳的有效方法，我已实验过N次啦。

```
004080FA     60                   pushad

004080FB     E8 01000000          call pex.00408101

                                  ====>F7进去就OVER啦  :-(

00408100     E8 83C404E8          call E8454588

00408105     0100                 add dword ptr ds:[eax],eax

00408107     0000                 add byte ptr ds:[eax],al

00408109   - E9 5D81EDD5          jmp D62E026B

0040810E     2240 00              and al,byte ptr ds:[eax]

00408111     E8 06020000          call pex.0040831C

00408116     E8 EB08EB02          call 032B8A06

0040811B     CD20 FF24249A        vxdcall 9A2424FF

00408121     66:BE 4746           mov si,4647

00408125     E8 01000000          call pex.0040812B

0040812A     9A 598D9527 2340     call far 4023:27958D59

00408131     00E8                 add al,ch

00408133     0100                 add dword ptr ds:[eax],eax

00408135     0000                 add byte ptr ds:[eax],al

00408137     6958 66 BF4D4AE8     imul ebx,dword ptr ds:[eax+66],E84A4DBF

0040813E     C101 00              rol dword ptr ds:[ecx],0

00408141     008D 52F9E801        add byte ptr ss:[ebp+1E8F952],cl

00408147     0000                 add byte ptr ds:[eax],al

00408149     00E8                 add al,ch

0040814B     5B                   pop ebx

0040814C     68 CCFFE29A          push 9AE2FFCC

00408151     FFE4                 jmp esp

00408153     69FF A5452540        imul edi,edi,402545A5

00408159     00E9                 add cl,ch

0040815B     E8 B9FFFFFF          call pex.00408119

00408160     EB 02                jmp short pex.00408164

00408162     CD20 8BC4EB02        vxdcall 2EBC48B

00408168     CD20 81001600        vxdcall 160081

0040816E     0000                 add byte ptr ds:[eax],al

00408170     0F85 A6010000        jnz pex.0040831C

00408176     69E8 00000000        imul ebp,eax,0

0040817C     58                   pop eax

0040817D     99                   cdq

0040817E     80CA 15              or dl,15

00408181     8D0402               lea eax,dword ptr ds:[edx+eax]

00408184     50                   push eax

00408185     E8 72010000          call pex.004082FC

0040818A     66:3D 86F3           cmp ax,0F386

0040818E     74 03                je short pex.00408193

00408190   - E9 8D95CB23          jmp 240C1722

00408195     40                   inc eax

00408196     00E8                 add al,ch

00408198     67:0100              add dword ptr ds:[bx+si],eax

0040819B     00E8                 add al,ch

0040819D     0100                 add dword ptr ds:[eax],eax

0040819F     0000                 add byte ptr ds:[eax],al

004081A1     6983 C4048DBD CA2540>imul eax,dword ptr ds:[ebx+BD8D04C4],pex.004025CA

004081AB     B9 89210000          mov ecx,2189

004081B0     BA 0CC8B7E1          mov edx,E1B7C80C

004081B5     8A07                 mov al,byte ptr ds:[edi]

004081B7     D2C0                 rol al,cl

004081B9     D2C8                 ror al,cl

004081BB     32C1                 xor al,cl

004081BD     F6D0                 not al

004081BF     32C5                 xor al,ch

004081C1     32C2                 xor al,dl

004081C3     32C6                 xor al,dh

004081C5     D2C0                 rol al,cl

004081C7     02C1                 add al,cl

004081C9     02C5                 add al,ch

004081CB     F6D8                 neg al

004081CD     02C2                 add al,dl

004081CF     02C6                 add al,dh

004081D1     D2C8                 ror al,cl

004081D3     2AC1                 sub al,cl

004081D5     2AC5                 sub al,ch

004081D7     F6D0                 not al

004081D9     2AC2                 sub al,dl

004081DB     2AC6                 sub al,dh

004081DD     D3C2                 rol edx,cl

004081DF     8807                 mov byte ptr ds:[edi],al

004081E1     47                   inc edi

004081E2     49                   dec ecx

004081E3   ^ 75 D0                jnz short pex.004081B5

004081E5     E8 01000000          call pex.004081EB

004081EA     E8 83C4040F          call 0F454672

004081EF     0BE8                 or ebp,eax

004081F1     2BD2                 sub edx,edx

004081F3     64:8B02              mov eax,dword ptr fs:[edx]

004081F6     8B20                 mov esp,dword ptr ds:[eax]

004081F8     64:8F02              pop dword ptr fs:[edx]

004081FB     58                   pop eax

004081FC     5D                   pop ebp

004081FD     C3                   retn

004081FE     9A 8B954525 4000     call far 0040:2545958B

00408205     E8 F9000000          call pex.00408303

0040820A     E8 01000000          call pex.00408210

0040820F     C783 C404BB73 4E0000>mov dword ptr ds:[ebx+73BB04C4],6A00004E

00408219     04 68                add al,68

0040821B     0030                 add byte ptr ds:[eax],dh

0040821D     0000                 add byte ptr ds:[eax],al

0040821F     53                   push ebx

00408220     6A 00                push 0

00408222     FF95 49254000        call dword ptr ss:[ebp+402549]

00408228     E8 01000000          call pex.0040822E

0040822D     E8 83C40468          call 684546B5

00408232     0040 00              add byte ptr ds:[eax],al

00408235     0053 50              add byte ptr ds:[ebx+50],dl

00408238     E8 01000000          call pex.0040823E

0040823D   - E9 83C40450          jmp 504546C5

00408242     8D95 CA254000        lea edx,dword ptr ss:[ebp+4025CA]

00408248     52                   push edx

00408249     E8 0E000000          call pex.0040825C

0040824E     E8 01000000          call pex.00408254

00408253     6983 C4045A5E 0E56CB>imul eax,dword ptr ds:[ebx+5E5A04C4],60CB560E

0040825D     8B7424 24            mov esi,dword ptr ss:[esp+24]

00408261     8B7C24 28            mov edi,dword ptr ss:[esp+28]

00408265     FC                   cld

00408266     B2 80                mov dl,80

00408268     A4                   movs byte ptr es:[edi],byte ptr ds:[esi]

00408269     E8 68000000          call pex.004082D6

0040826E   ^ 73 F8                jnb short pex.00408268

00408270     2BC9                 sub ecx,ecx

00408272     E8 5F000000          call pex.004082D6

00408277     73 1A                jnb short pex.00408293

00408279     2BC0                 sub eax,eax

0040827B     E8 56000000          call pex.004082D6

00408280     73 20                jnb short pex.004082A2

00408282     41                   inc ecx

00408283     B0 10                mov al,10

00408285     E8 4C000000          call pex.004082D6

0040828A     12C0                 adc al,al

0040828C   ^ 73 F7                jnb short pex.00408285

0040828E     75 3C                jnz short pex.004082CC

00408290     AA                   stos byte ptr es:[edi]

00408291   ^ EB D6                jmp short pex.00408269

00408293     E8 4A000000          call pex.004082E2

00408298     49                   dec ecx
```
