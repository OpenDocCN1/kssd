
`:00401954 E843310D00`
     `call 004D4A9C`
`:00401959 8D55F4`
         `lea edx, dword ptr [ebp-0C]`
`:0040195C 8D45F0`
                 `lea eax, dword ptr` `[ebp-10]`
`:0040195F E868310D00`              `call` `004D4ACC`
`:00401964 EB68`
   `jmp 004019CE`
\* 
Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004019D9(C)
|
`:00401966`
`66C78514FFFFFF7400`      `mov word ptr [ebp+FFFFFF14], 0074`
`:0040196F`
`8D45D4`                  `lea eax,` `dword ptr [ebp-2C]`
`:00401972 E8610E0000`
   `call 004027D8`
`:00401977 50`
           `push eax`
`:00401978 FF8520FFFFFF`
         `inc dword ptr [ebp+FFFFFF20]`
\* 
Possible StringData Ref from Data Obj ->"00燃撞"
     |
`:0040197E BAD3904E00`
   `mov edx, 004E90D3`
`:00401983 8D45D8`
         `lea eax, dword ptr [ebp-28]`
`:00401986 E86D2F0D00`
             `call 004D48F8`
`:0040198B FF8520FFFFFF`
           `inc dword ptr [ebp+FFFFFF20]`
`:00401991`
`8D55D8`                  `lea edx,` `dword ptr [ebp-28]`
`:00401994 8D45F4`
     `lea eax, dword ptr [ebp-0C]`
`:00401997 59`
                 `pop ecx`
`:00401998`
`E857310D00`              `call 004D4AF4`
`:0040199D`
`8D55D4`                  `lea edx,` `dword ptr [ebp-2C]`
`:004019A0 8D45F4`
     `lea eax, dword ptr [ebp-0C]`
`:004019A3 E824310D00`
           `call 004D4ACC`
`:004019A8 FF8D20FFFFFF`
           `dec dword ptr [ebp+FFFFFF20]`
`:004019AE`
`8D45D4`                  `lea eax,` `dword ptr [ebp-2C]`
`:004019B1 BA02000000`
   `mov edx, 00000002`
`:004019B6 E8E1300D00`
     `call 004D4A9C`
`:004019BB FF8D20FFFFFF`
     `dec dword ptr [ebp+FFFFFF20]`
`:004019C1 8D45D8`
             `lea eax, dword ptr [ebp-28]`
`:004019C4`
`BA02000000`              `mov edx, 00000002`
`:004019C9`
`E8CE300D00`              `call 004D4A9C`
\* 
Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00401964(U)
|
`:004019CE`
`8D45F4`                  `lea eax,` `dword ptr [ebp-0C]`
`:004019D1 E8520E0000`
   `call 00402828`
`:004019D6 83F809`
       `cmp eax, 00000009`
`:004019D9 7C8B`
             `jl 00401966`
     ====>比较用户号是否9位？不到9位则跳上去末尾添0
 ====>3297438--->329743800
`:004019DB`
`33D2`                    `xor` `edx, edx`
`:004019DD 8995FCFEFFFF`            `mov` `dword ptr [ebp+FFFFFEFC], edx`
`:004019E3 66C78514FFFFFF8000`      `mov` `word ptr [ebp+FFFFFF14], 0080`
`:004019EC 8D45D0`
         `lea eax, dword ptr [ebp-30]`
`:004019EF E8E40D0000`
             `call 004027D8`
`:004019F4 50`
                     `push` `eax`
`:004019F5 8D45F4`
 lea eax, dword ptr [ebp-0C]
`:004019F8 FF8520FFFFFF`
     `inc dword ptr [ebp+FFFFFF20]`
         ====>下面开始依次取序列号的值运算了！！
`:004019FE B901000000`
             `mov ecx, 00000001`
`:00401A03`
`BA01000000`              `mov edx, 00000001`
`:00401A08`
`E8C7320D00`              `call 004D4CD4`
`:00401A0D`
`8D45D0`                  `lea eax,` `dword ptr [ebp-30]`
`:00401A10 8B00`
       `mov eax, dword ptr [eax]`
`:00401A12 E881280B00`
           `call 004B4298`
`:00401A17 8BD0`
                 `mov edx, eax`
                第一部分！  1、  ====>EDX=EAX=3
`:00401A19`
`C1E204`                  `shl edx,` `04`
           ====>EDX=3 SHL 4=30
`:00401A1C`
`03D0`                    `add` `edx, eax`
             ====>EDX=30 + 3=33
`:00401A1E`
`8D14D0`                  `lea edx,` `dword ptr [eax+8\*edx]`
                 ====>EDX=3 + 8\*33=19B
`:00401A21 81C2F6000000`
           `add edx, 000000F6`
   ====>EDX=19B + F6=291
`:00401A27`
`0195FCFEFFFF`            `add dword ptr [ebp+FFFFFEFC],` `edx`
             ====>[ebp+FFFFFEFC]=00 + 291=291
`:00401A2D`
`FF8D20FFFFFF`            `dec dword ptr [ebp+FFFFFF20]`
`:00401A33`
`8D45D0`                  `lea eax,` `dword ptr [ebp-30]`
`:00401A36 BA02000000`
   `mov edx, 00000002`
`:00401A3B E85C300D00`
     `call 004D4A9C`
`:00401A40 66C78514FFFFFF8C00`
 mov word ptr [ebp+FFFFFF14], 008C
`:00401A49 8D45CC`
           `lea eax, dword ptr [ebp-34]`
`:00401A4C`
`E8870D0000`              `call 004027D8`
`:00401A51`
`50`                      `push` `eax`
`:00401A52 8D45F4`
 lea eax, dword ptr [ebp-0C]
`:00401A55 FF8520FFFFFF`
     `inc dword ptr [ebp+FFFFFF20]`
`:00401A5B B901000000`
           `mov ecx, 00000001`
`:00401A60 BA02000000`
             `mov edx, 00000002`
`:00401A65`
`E86A320D00`              `call 004D4CD4`
`:00401A6A`
`8D45CC`                  `lea eax,` `dword ptr [ebp-34]`
`:00401A6D 8B00`
       `mov eax, dword ptr [eax]`
`:00401A6F E824280B00`
           `call 004B4298`
`:00401A74 8BD8`
                 `mov ebx, eax`
`:00401A76`
`8D45C8`                  `lea eax,` `dword ptr [ebp-38]`
                 ====>EAX=[ebp-38]=291
`:00401A79`
`E85A0D0000`              `call 004027D8`
`:00401A7E`
`50`                      `push` `eax`
`:00401A7F 8D45F4`
 lea eax, dword ptr [ebp-0C]
`:00401A82 FF8520FFFFFF`
     `inc dword ptr [ebp+FFFFFF20]`
`:00401A88 B901000000`
           `mov ecx, 00000001`
`:00401A8D BA02000000`
             `mov edx, 00000002`
`:00401A92`
`E83D320D00`              `call 004D4CD4`
`:00401A97`
`8D45C8`                  `lea eax,` `dword ptr [ebp-38]`
`:00401A9A 8B00`
       `mov eax, dword ptr [eax]`
`:00401A9C E8F7270B00`
           `call 004B4298`
`:00401AA1 0FAFD8`
               `imul ebx, eax`
  2、  ====>EBX=2 \* 2=4
`:00401AA4`
`81C371010000`            `add ebx, 00000171`
       ====>EBX=4 + 171=175
`:00401AAA`
`019DFCFEFFFF`            `add dword ptr [ebp+FFFFFEFC],` `ebx`
             ====>[ebp+FFFFFEFC]=291 + 175=406
`:00401AB0`
`FF8D20FFFFFF`            `dec dword ptr [ebp+FFFFFF20]`
`:00401AB6`
`8D45C8`                  `lea eax,` `dword ptr [ebp-38]`
`:00401AB9 BA02000000`
   `mov edx, 00000002`
`:00401ABE E8D92F0D00`
     `call 004D4A9C`
`:00401AC3 FF8D20FFFFFF`
     `dec dword ptr [ebp+FFFFFF20]`
`:00401AC9 8D45CC`
             `lea eax, dword ptr [ebp-34]`
`:00401ACC`
`BA02000000`              `mov edx, 00000002`
`:00401AD1`
`E8C62F0D00`              `call 004D4A9C`
`:00401AD6`
`66C78514FFFFFF9800`      `mov word ptr [ebp+FFFFFF14], 0098`
`:00401ADF`
`8D45C4`                  `lea eax,` `dword ptr [ebp-3C]`
`:00401AE2 E8F10C0000`
   `call 004027D8`
`:00401AE7 50`
           `push eax`
`:00401AE8 8D45F4`
             `lea eax, dword ptr [ebp-0C]`
`:00401AEB`
`FF8520FFFFFF`            `inc dword ptr [ebp+FFFFFF20]`
`:00401AF1`
`B901000000`              `mov ecx, 00000001`
`:00401AF6`
`BA03000000`              `mov edx, 00000003`
`:00401AFB`
`E8D4310D00`              `call 004D4CD4`
`:00401B00`
`8D45C4`                  `lea eax,` `dword ptr [ebp-3C]`
`:00401B03 8B00`
       `mov eax, dword ptr [eax]`
`:00401B05 E88E270B00`
           `call 004B4298`
`:00401B0A 8BD0`
                 `mov edx, eax`
    3、  ====>EDX=EAX=9
`:00401B0C`
`C1E209`                  `shl edx,` `09`
           ====>EDX=9 SHL 9=1200
`:00401B0F`
`2BD0`                    `sub` `edx, eax`
             ====>EDX=1200 - 9=11F7
`:00401B11`
`81C207F2FFFF`            `add edx, FFFFF207`
       ====>EDX=11F7 + -0DF9=3FE
`:00401B17`
`0195FCFEFFFF`            `add dword ptr [ebp+FFFFFEFC],` `edx`
             ====>[ebp+FFFFFEFC]=406 + 3FE=804
`:00401B1D`
`FF8D20FFFFFF`            `dec dword ptr [ebp+FFFFFF20]`
`:00401B23`
`8D45C4`                  `lea eax,` `dword ptr [ebp-3C]`
`:00401B26 BA02000000`
   `mov edx, 00000002`
`:00401B2B E86C2F0D00`
     `call 004D4A9C`
`:00401B30 66C78514FFFFFFA400`
 mov word ptr [ebp+FFFFFF14], 00A4
`:00401B39 8D45C0`
           `lea eax, dword ptr [ebp-40]`
`:00401B3C`
`E8970C0000`              `call 004027D8`
`:00401B41`
`50`                      `push` `eax`
`:00401B42 8D45F4`
 lea eax, dword ptr [ebp-0C]
`:00401B45 FF8520FFFFFF`
     `inc dword ptr [ebp+FFFFFF20]`
`:00401B4B B901000000`
           `mov ecx, 00000001`
`:00401B50 BA04000000`
             `mov edx, 00000004`
`:00401B55`
`E87A310D00`              `call 004D4CD4`
`:00401B5A`
`8D45C0`                  `lea eax,` `dword ptr [ebp-40]`
`:00401B5D 8B00`
       `mov eax, dword ptr [eax]`
                    4、  ====>EAX=7
`:00401B5F`
`E834270B00`              `call 004B4298`
`:00401B64`
`8D1440`                  `lea edx,` `dword ptr [eax+2\*eax]`
                 ====>EDX=7 + 2\*7=15
`:00401B67 C1E203`
                 `shl edx, 03`
         ====>EDX=15 SHL 3=A8
`:00401B6A`
`2BD0`                    `sub` `edx, eax`
             ====>EDX=A8 - 7=A1
`:00401B6C`
`C1E202`                  `shl edx,` `02`
