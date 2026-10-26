
```
005486E0 >   $  55              push ebp                                 ;  DriverUpdate.sub_005486E0

005486E1   .  8BEC            mov ebp,esp

005486E3   .  B9 38000000     mov ecx,0x38

005486E8   >  6A 00           push 0x0

005486EA   .  6A 00           push 0x0

005486EC   .  49              dec ecx

005486ED   .^ 75 F9           jnz short Computer.005486E8

005486EF   .  51              push ecx

005486F0   .  53              push ebx

005486F1   .  56              push esi

005486F2   .  57              push edi

005486F3   .  8955 F8         mov dword ptr ss:[ebp-0x8],edx

005486F6   .  8945 FC         mov dword ptr ss:[ebp-0x4],eax

005486F9   .  33C0            xor eax,eax

005486FB   .  55              push ebp

005486FC   .  68 7E945400     push Computer.0054947E

00548701   .  64:FF30         push dword ptr fs:[eax]

00548704   .  64:8920         mov dword ptr fs:[eax],esp

00548707   .  C645 F7 00      mov byte ptr ss:[ebp-0x9],0x0

0054870B   .  8B45 FC         mov eax,dword ptr ss:[ebp-0x4]

0054870E   .  66:8378 0A 00   cmp word ptr ds:[eax+0xA],0x0

00548713   .  74 18           je short Computer.0054872D

00548715   .  8B45 F8         mov eax,dword ptr ss:[ebp-0x8]

00548718   .  8378 10 00      cmp dword ptr ds:[eax+0x10],0x0

0054871C   .  74 0F           je short Computer.0054872D

0054871E   .  8B45 F8         mov eax,dword ptr ss:[ebp-0x8]

00548721   .  8B50 10         mov edx,dword ptr ds:[eax+0x10]

00548724   .  8B5D FC         mov ebx,dword ptr ss:[ebp-0x4]

00548727   .  8B43 0C         mov eax,dword ptr ds:[ebx+0xC]

0054872A   .  FF53 08         call dword ptr ds:[ebx+0x8]

0054872D   >  8B45 F8         mov eax,dword ptr ss:[ebp-0x8]

00548730   .  8B10            mov edx,dword ptr ds:[eax]

00548732   .  8B45 FC         mov eax,dword ptr ss:[ebp-0x4]

00548735   .  E8 A6140000     call <Computer.DriverUpdate.sub_00549BE0>

0054873A   .  84C0            test al,al

0054873C   .  0F84 360B0000   je Computer.00549278

00548742   .  8D45 E0         lea eax,dword ptr ss:[ebp-0x20]

00548745   .  50              push eax

00548746   .  A1 10A85B00     mov eax,dword ptr ds:[0x5BA810]

0054874B   .  8B00            mov eax,dword ptr ds:[eax]

0054874D   .  8B40 10         mov eax,dword ptr ds:[eax+0x10]

00548750   .  8945 B0         mov dword ptr ss:[ebp-0x50],eax

00548753   .  C645 B4 00      mov byte ptr ss:[ebp-0x4C],0x0

00548757   .  A1 10A85B00     mov eax,dword ptr ds:[0x5BA810]

0054875C   .  8B00            mov eax,dword ptr ds:[eax]

0054875E   .  8B40 14         mov eax,dword ptr ds:[eax+0x14]

00548761   .  8945 B8         mov dword ptr ss:[ebp-0x48],eax

00548764   .  C645 BC 00      mov byte ptr ss:[ebp-0x44],0x0

00548768   .  8D55 B0         lea edx,dword ptr ss:[ebp-0x50]

0054876B   .  B9 01000000     mov ecx,0x1

00548770   .  B8 9C945400     mov eax,Computer.0054949C                ;  %d.%d

00548775   .  E8 3E53ECFF     call <Computer.SysUtils.Format>

0054877A   .  A1 10A85B00     mov eax,dword ptr ds:[0x5BA810]

0054877F   .  8B00            mov eax,dword ptr ds:[eax]

00548781   .  8078 50 00      cmp byte ptr ds:[eax+0x50],0x0

00548785   .  74 0F           je short Computer.00548796

00548787   .  8D45 DC         lea eax,dword ptr ss:[ebp-0x24]

0054878A   .  BA AC945400     mov edx,Computer.005494AC                ;  [B]64[/B]

0054878F   .  E8 40C6EBFF     call <Computer.system.@LStrLAsg>

00548794   .  EB 0D           jmp short Computer.005487A3

00548796   >  8D45 DC         lea eax,dword ptr ss:[ebp-0x24]

00548799   .  BA B8945400     mov edx,Computer.005494B8                ;  [B]32[/B]

0054879E   .  E8 31C6EBFF     call <Computer.system.@LStrLAsg>

005487A3   >  8D45 D8         lea eax,dword ptr ss:[ebp-0x28]

005487A6   .  8B15 7CA05B00   mov edx,dword ptr ds:[0x5BA07C]          ;  Computer.005CA6A0

005487AC   .  83C2 0C         add edx,0xC

005487AF   .  B9 20000000     mov ecx,0x20

005487B4   .  E8 F3C7EBFF     call <Computer.system.@LStrFromArray>

005487B9   .  A0 BC945400     mov al,byte ptr ds:[0x5494BC]

005487BE   .  50              push eax

005487BF   .  8D45 D4         lea eax,dword ptr ss:[ebp-0x2C]

005487C2   .  50              push eax

005487C3   .  8D55 AC         lea edx,dword ptr ss:[ebp-0x54]

005487C6   .  8B45 F8         mov eax,dword ptr ss:[ebp-0x8]

005487C9   .  8B00            mov eax,dword ptr ds:[eax]

005487CB   .  E8 8036ECFF     call <Computer.SysUtils.UpperCase>

005487D0   .  8B45 AC         mov eax,dword ptr ss:[ebp-0x54]

005487D3   .  B9 C8945400     mov ecx,Computer.005494C8                ;  &

005487D8   .  BA D4945400     mov edx,Computer.005494D4                ;  @

005487DD   .  E8 4A97ECFF     call <Computer.SysUtils.StringReplace>

005487E2   .  8D45 D0         lea eax,dword ptr ss:[ebp-0x30]

005487E5   .  BA E0945400     mov edx,<Computer.aDefault_2>            ;  default

005487EA   .  E8 E5C5EBFF     call <Computer.system.@LStrLAsg>

005487EF   .  8D55 CC         lea edx,dword ptr ss:[ebp-0x34]

005487F2   .  8B45 F8         mov eax,dword ptr ss:[ebp-0x8]

005487F5   .  8B40 24         mov eax,dword ptr ds:[eax+0x24]

005487F8   .  E8 773FECFF     call <Computer.SysUtils.IntToStr>

005487FD   .  FF75 E0         push dword ptr ss:[ebp-0x20]

00548800   .  FF75 DC         push dword ptr ss:[ebp-0x24]

00548803   .  FF75 D8         push dword ptr ss:[ebp-0x28]

00548806   .  FF75 D4         push dword ptr ss:[ebp-0x2C]

00548809   .  FF75 D0         push dword ptr ss:[ebp-0x30]

0054880C   .  8D45 E4         lea eax,dword ptr ss:[ebp-0x1C]

0054880F   .  BA 05000000     mov edx,0x5

00548814   .  E8 A3C8EBFF     call <Computer.system.@LStrCatN>

00548819   .  8D4D C4         lea ecx,dword ptr ss:[ebp-0x3C]

0054881C   .  BA 64000000     mov edx,0x64

00548821   .  8B45 E4         mov eax,dword ptr ss:[ebp-0x1C]

00548824   .  E8 C333F7FF     call <Computer.EncryptUtil.[B]sub_004BBBEC>[/B]    

00548829   .  8D55 E4         lea edx,dword ptr ss:[ebp-0x1C]

0054882C   .  8B45 C4         mov eax,dword ptr ss:[ebp-0x3C]

0054882F   .  E8 1035F7FF     call <Computer.EncryptUtil.[B]sub_004BBD44>[/B]    ;这两个加密函数产生KEY

00548834   .  8B45 F8         mov eax,dword ptr ss:[ebp-0x8]

00548837   .  05 90000000     add eax,0x90

0054883C   .  8B55 E4         mov edx,dword ptr ss:[ebp-0x1C]

0054883F   .  E8 4CC5EBFF     call <Computer.system.@LStrAsg>

00548844   .  8D45 F0         lea eax,dword ptr ss:[ebp-0x10]

00548847   .  50              push eax

00548848   .  B8 F0945400     mov eax,<Computer.aHttpApi_driver>       ;  [B]986K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3q4H3K9g2)9J5k6h3c8J5K9i4k6W2M7W2)9J5k6e0x3$3x3s2y4S2k6X3g2Q4x3X3g2U0L8$3#2Q4x3V1k6S2M7s2m8K6k6i4u0$3i4K6u0r3M7h3c8Q4y4h3k6I4N6h3g2J5P5h3y4Q4x3X3f1J5i4K6u0W2x3q4)9J5k6i4m8Z5M7q4)9#2b7W2)9J5c8V1u0Q4y4f1b7`.

0054884D   .  8985 6CFFFFFF   mov dword ptr ss:[ebp-0x94],eax

00548853   .  C685 70FFFFFF 0>mov byte ptr ss:[ebp-0x90],0xB

0054885A   .  8B45 E0         mov eax,dword ptr ss:[ebp-0x20]

0054885D   .  8985 74FFFFFF   mov dword ptr ss:[ebp-0x8C],eax

00548863   .  C685 78FFFFFF 0>mov byte ptr ss:[ebp-0x88],0xB

0054886A   .  8B45 DC         mov eax,dword ptr ss:[ebp-0x24]

0054886D   .  8985 7CFFFFFF   mov dword ptr ss:[ebp-0x84],eax

00548873   .  C645 80 0B      mov byte ptr ss:[ebp-0x80],0xB

00548877   .  8B45 D8         mov eax,dword ptr ss:[ebp-0x28]

0054887A   .  8945 84         mov dword ptr ss:[ebp-0x7C],eax

005487D   .  C645 88 0B      mov byte ptr ss:[ebp-0x78],0xB

00548881   .  8D95 68FFFFFF   lea edx,dword ptr ss:[ebp-0x98]

00548887   .  8B45 D4         mov eax,dword ptr ss:[ebp-0x2C]

0054888A   .  E8 BD9FFFFF     call <Computer.?Unit155.sub_0054284C>

0054888F   .  8B85 68FFFFFF   mov eax,dword ptr ss:[ebp-0x98]

00548895   .  8945 8C         mov dword ptr ss:[ebp-0x74],eax

00548898   .  C645 90 0B      mov byte ptr ss:[ebp-0x70],0xB

0054889C   .  8B45 D0         mov eax,dword ptr ss:[ebp-0x30]

0054889F   .  8945 94         mov dword ptr ss:[ebp-0x6C],eax

005488A2   .  C645 98 0B      mov byte ptr ss:[ebp-0x68],0xB

005488A6   .  8B45 CC         mov eax,dword ptr ss:[ebp-0x34]

005488A9   .  8945 9C         mov dword ptr ss:[ebp-0x64],eax

005488AC   .  C645 A0 0B      mov byte ptr ss:[ebp-0x60],0xB

005488B0   .  8B45 E4         mov eax,dword ptr ss:[ebp-0x1C]

005488B3   .  8945 A4         mov dword ptr ss:[ebp-0x5C],eax

005488B6   .  C645 A8 0B      mov byte ptr ss:[ebp-0x58],0xB

005488BA   .  8D95 6CFFFFFF   lea edx,dword ptr ss:[ebp-0x94]

005488C0   .  B9 07000000     mov ecx,0x7

005488C5   .  B8 30955400     mov eax,<Computer.aS?osSPfSCnSHid>       ; [B] %s?os=%s&pf=%s&cn=%s&hid=%s&nw=%s&err=%s&c=%s[/B]

005488CA   .  E8 E951ECFF     call <Computer.SysUtils.Format>

005488CF   .  6A 01           push 0x1

005488D1   .  8B45 F8         mov eax,dword ptr ss:[ebp-0x8]

005488D4   .  8B00            mov eax,dword ptr ds:[eax]

005488D6   .  8945 B0         mov dword ptr ss:[ebp-0x50],eax

005488D9   .  C645 B4 0B      mov byte ptr ss:[ebp-0x4C],0xB

005488DD   .  8B45 F0         mov eax,dword ptr ss:[ebp-0x10]

005488E0   .  8945 B8         mov dword ptr ss:[ebp-0x48],eax

005488E3   .  C645 BC 0B      mov byte ptr ss:[ebp-0x44],0xB

005488E7   .  8D4D B0         lea ecx,dword ptr ss:[ebp-0x50]

005488EA   .  A1 ECA95B00     mov eax,dword ptr ds:[0x5BA9EC]

005488EF   .  8B00            mov eax,dword ptr ds:[eax]

005488F1   .  BA 68955400     mov edx,Computer.00549568                ;  [B]设备%s, URL:%s[/B]

005488F6   .  E8 D1E6FFFF     call <Computer.?Unit160.sub_00546FCC>

005488FB   .  B2 01           mov dl,0x1

005488FD   .  A1 64255400     mov eax,dword ptr ds:[0x542564]

00548902   .  E8 919DFFFF     call <Computer.?Unit155.TWinHTTPLib.Crea>

00548907   .  8945 EC         mov dword ptr ss:[ebp-0x14],eax

0054890A   .  8B45 EC         mov eax,dword ptr ss:[ebp-0x14]

0054890D   .  C740 4C 409C000>mov dword ptr ds:[eax+0x4C],0x9C40

00548914   .  8B45 EC         mov eax,dword ptr ss:[ebp-0x14]

00548917   .  C740 48 3075000>mov dword ptr ds:[eax+0x48],0x7530

0054891E   .  8B45 EC         mov eax,dword ptr ss:[ebp-0x14]

00548921   .  C740 50 3075000>mov dword ptr ds:[eax+0x50],0x7530

00548928   .  33C9            xor ecx,ecx

0054892A   .  8B55 F0         mov edx,dword ptr ss:[ebp-0x10]

0054892D   .  8B45 EC         mov eax,dword ptr ss:[ebp-0x14]

00548930   .  E8 5FA1FFFF     call <Computer.[B]QueryDownLoad[/B]>

00548935   .  84C0            test al,al

00548937   .  0F84 2C090000   je Computer.00549269

0054893D   .  8D45 E8         lea eax,dword ptr ss:[ebp-0x18]

00548940   .  E8 1BE3EBFF     call <Computer.system.@IntfClear>

00548945   .  33C0            xor eax,eax

00548947   .  55              push ebp

00548948   .  68 E0895400     push Computer.005489E0

0054894D   .  64:FF30         push dword ptr fs:[eax]

00548950   .  64:8920         mov dword ptr fs:[eax],esp

00548953   .  8D45 C8         lea eax,dword ptr ss:[ebp-0x38]

00548956   .  8B55 EC         mov edx,dword ptr ss:[ebp-0x14]

00548959   .  8B52 14         mov edx,dword ptr ds:[edx+0x14]

0054895C   .  8B52 04         mov edx,dword ptr ds:[edx+0x4]

0054895F   .  E8 D0C5EBFF     call <Computer.system.@LStrFromPChar>

00548964   .  8D95 64FFFFFF   lea edx,dword ptr ss:[ebp-0x9C]

0054896A   .  8B45 C8         mov eax,dword ptr ss:[ebp-0x38]

0054896D   .  E8 429AFFFF     call <Computer.?Unit153.sub_005423B4>

00548972   .  8B95 64FFFFFF   mov edx,dword ptr ss:[ebp-0x9C]

00548978   .  8D45 C8         lea eax,dword ptr ss:[ebp-0x38]

0054897B   .  E8 54C4EBFF     call <Computer.system.@LStrLAsg>

00548980   .  8D8D 5CFFFFFF   lea ecx,dword ptr ss:[ebp-0xA4]

00548986   .  BA 64000000     mov edx,0x64

0054898B   .  8B45 C8         mov eax,dword ptr ss:[ebp-0x38]

0054898E   .  E8 5932F7FF     call <Computer.EncryptUtil.sub_004BBBEC>

00548993   .  8B85 5CFFFFFF   mov eax,dword ptr ss:[ebp-0xA4]

00548999   .  8D95 60FFFFFF   lea edx,dword ptr ss:[ebp-0xA0]

0054899F   .  E8 FC4BECFF     call <Computer.SysUtils.StrPas>

005489A4   .  8B95 60FFFFFF   mov edx,dword ptr ss:[ebp-0xA0]

005489AA   .  8D45 C8         lea eax,dword ptr ss:[ebp-0x38]

005489AD   .  E8 22C4EBFF     call <Computer.system.@LStrLAsg>

005489B2   .  8B45 C8         mov eax,dword ptr ss:[ebp-0x38]

005489B5   .  E8 52960500     call <Computer.?Unit233.sub_005A200C>

005489BA   .  8D85 58FFFFFF   lea eax,dword ptr ss:[ebp-0xA8]

005489C0   .  8B55 C8         mov edx,dword ptr ss:[ebp-0x38]

005489C3   .  E8 14CCEBFF     call <Computer.system.@WStrFromLStr>

005489C8   .  8B85 58FFFFFF   mov eax,dword ptr ss:[ebp-0xA8]

005489CE   .  8D55 E8         lea edx,dword ptr ss:[ebp-0x18]

005489D1   .  E8 3216FFFF     call <Computer.superobject.sub_0053A008>

005489D6   .  33C0            xor eax,eax

005489D8   .  5A              pop edx

005489D9   .  59              pop ecx

005489DA   .  59              pop ecx

005489DB   .  64:8910         mov dword ptr fs:[eax],edx

```
