
# 系统机械师 SysMechanic 3.6d 的破解

> 原文：[`www.pediy.com/kssd/pediy03/forum1268.htm`](https://www.pediy.com/kssd/pediy03/forum1268.htm)

系统机械师 `SysMechanic` 3.6d 的破解。该软件是一款不错的系统工具，功能齐全，安装包只有 1M 多，可谓麻雀虽小却五脏具全。华军软件圆有下载。

开始动手：

1. 启动 `SICE`，运行软件，用户名：`wdfdiablo`，注册码：`11111-11111-1111111111`，`CTRL+D`。停电？看来 `SICE` 的名气太大了，不得不防它，换 `TRW2000` 上吧。
2. 下 `bpx hmemcpy`，点注册，拦断后 `bd *` 后按 `F12` 键 16 下，来到此处：

```
:0049730F FF75F4              push [ebp-0C]
:00497312 681C754900          push 0049751C
:00497317 8D55F0              lea edx, dword ptr [ebp-10]
:0049731A 8B8300020000        mov eax, dword ptr [ebx+00000200]
:00497320 E80348FCFF          call 0045BB28
:00497325 FF75F0              push [ebp-10]
:00497328 681C754900          push 0049751C
:0049732D 8D55EC              lea edx, dword ptr [ebp-14]
:00497330 8B8304020000        mov eax, dword ptr [ebx+00000204]
:00497336 E8ED47FCFF          call 0045BB28
:0049733B FF75EC              push [ebp-14]
:0049733E 8D45F8              lea eax, dword ptr [ebp-08]

* Possible Reference to String Resource ID=00005: "謭"

:00497341 BA05000000          mov edx, 00000005
:00497346 E809CCF6FF          call 00403F54
:0049734B 8B45F8              mov eax, dword ptr [ebp-08]
:0049734E 8D55FC              lea edx, dword ptr [ebp-04]
:00497351 E86E0BF7FF          call 00407EC4
:00497356 8D55E8              lea edx, dword ptr [ebp-18]
:00497359 8B83DC010000        mov eax, dword ptr [ebx+000001DC]
:0049735F E834C1F8FF          call 00423498
:00497364 8B45E8              mov eax, dword ptr [ebp-18]
:00497367 8B55FC              mov edx, dword ptr [ebp-04]
:0049736A E80971FFFF          call 0048E478              ----------------此为算注册码的 CALL，F8 进入
:0049736F 84C0                test al, al
:00497371 740F                je 00497382
:00497373 E8B06CFFFF          call 0048E028
:00497378 E85BA7FFFF          call 00491AD8
:0049737D E95F010000          jmp 004974E1
```

进入 `0049736A` 的 `CALL` 后：

```
* Referenced by a CALL at Addresses:
:0048F225  , :0049736A

:0048E478 55                  push ebp
:0048E479 8BEC                mov ebp, esp
:0048E47B 33C9                xor ecx, ecx
:0048E47D 51                  push ecx
:0048E47E 51                  push ecx
:0048E47F 51                  push ecx
:0048E480 51                  push ecx
:0048E481 51                  push ecx
:0048E482 53                  push ebx
:0048E483 8955F8              mov dword ptr [ebp-08], edx
:0048E486 8945FC              mov dword ptr [ebp-04], eax
:0048E489 8B45FC              mov eax, dword ptr [ebp-04]
:0048E48C E8B75BF7FF          call 00404048
:0048E491 8B45F8              mov eax, dword ptr [ebp-08]
:0048E494 E8AF5BF7FF          call 00404048
:0048E499 33C0                xor eax, eax
:0048E49B 55                  push ebp
:0048E49C 683DE64800          push 0048E63D
:0048E4A1 64FF30              push dword ptr fs:[eax]
:0048E4A4 648920              mov dword ptr fs:[eax], esp
:0048E4A7 8D55EC              lea edx, dword ptr [ebp-14]
:0048E4AA 8B45FC              mov eax, dword ptr [ebp-04]
:0048E4AD E8129AF7FF          call 00407EC4
:0048E4B2 8B55EC              mov edx, dword ptr [ebp-14]
:0048E4B5 8D45FC              lea eax, dword ptr [ebp-04]
:0048E4B8 E8F357F7FF          call 00403CB0
:0048E4BD 8D55EC              lea edx, dword ptr [ebp-14]
:0048E4C0 8B45F8              mov eax, dword ptr [ebp-08]
:0048E4C3 E8FC99F7FF          call 00407EC4
:0048E4C8 8B55EC              mov edx, dword ptr [ebp-14]
:0048E4CB 8D45F8              lea eax, dword ptr [ebp-08]
:0048E4CE E8DD57F7FF          call 00403CB0
:0048E4D3 B201                mov dl, 01

* Possible StringData Ref from Code Obj ->"?A"

:0048E4D5 A1B8EE4000          mov eax, dword ptr [0040EEB8]
:0048E4DA E8494BF7FF          call 00403028
:0048E4DF 8BD8                mov ebx, eax

* Possible StringData Ref from Code Obj ->"BXCELL@USA.NET"

:0048E4E1 BA54E64800          mov edx, 0048E654
:0048E4E6 8BC3                mov eax, ebx
:0048E4E8 8B08                mov ecx, dword ptr [eax]
:0048E4EA FF5134              call [ecx+34]

* Possible StringData Ref from Code Obj ->"RAYBIEZ"

:0048E4ED BA6CE64800          mov edx, 0048E66C
:0048E4F2 8BC3                mov eax, ebx
:0048E4F4 8B08                mov ecx, dword ptr [eax]
:0048E4F6 FF5134              call [ecx+34]

* Possible StringData Ref from Code Obj ->"REANIMATOR"

:0048E4F9 BA7CE64800          mov edx, 0048E67C
:0048E4FE 8BC3                mov eax, ebx
:0048E500 8B08                mov ecx, dword ptr [eax]
:0048E502 FF5134              call [ecx+34]

* Possible StringData Ref from Code Obj ->"CHIESARA2000"

:0048E505 BA90E64800          mov edx, 0048E690
:0048E50A 8BC3                mov eax, ebx
:0048E50C 8B08                mov ecx, dword ptr [eax]
:0048E50E FF5134              call [ecx+34]

* Possible StringData Ref from Code Obj ->"DITRICHNEUMAN@"

:0048E511 BAA8E64800          mov edx, 0048E6A8
:0048E516 8BC3                mov eax, ebx
:0048E518 8B08                mov ecx, dword ptr [eax]
:0048E51A FF5134              call [ecx+34]

* Possible StringData Ref from Code Obj ->"NAKLIMANUS"

:0048E51D BAC0E64800          mov edx, 0048E6C0
:0048E522 8BC3                mov eax, ebx
:0048E524 8B08                mov ecx, dword ptr [eax]
:0048E526 FF5134              call [ecx+34]

* Possible StringData Ref from Code Obj ->"ALHASAN1999"

:0048E529 BAD4E64800          mov edx, 0048E6D4
:0048E52E 8BC3                mov eax, ebx
:0048E530 8B08                mov ecx, dword ptr [eax]
:0048E532 FF5134              call [ecx+34]

* Possible StringData Ref from Code Obj ->"LILJOELUNA"

:0048E535 BAE8E64800          mov edx, 0048E6E8
:0048E53A 8BC3                mov eax, ebx
:0048E53C 8B08                mov ecx, dword ptr [eax]
:0048E53E FF5134              call [ecx+34]

* Possible StringData Ref from Code Obj ->"COKEBOTTLE"

:0048E541 BAFCE64800          mov edx, 0048E6FC
:0048E546 8BC3                mov eax, ebx
:0048E548 8B08                mov ecx, dword ptr [eax]
:0048E54A FF5134              call [ecx+34]

* Possible StringData Ref from Code Obj ->"[EVC]"

:0048E54D BA10E74800          mov edx, 0048E710
:0048E552 8BC3                mov eax, ebx
:0048E554 8B08                mov ecx, dword ptr [eax]
:0048E556 FF5134              call [ecx+34]

* Possible StringData Ref from Code Obj ->"[TMG]"

:0048E559 BA20E74800          mov edx, 0048E720
:0048E55E 8BC3                mov eax, ebx
```



:0048E560 `8B08`
`mov ecx, dword ptr [eax]`

:0048E562 `FF5134`
`call [ecx+34]`

* Possible StringData Ref from Code Obj -> `[DSI]`

:0048E565 `BA30E74800`
`mov edx, 0048E730`

:0048E56A `8BC3`
`mov eax, ebx`

:0048E56C `8B08`
`mov ecx, dword ptr [eax]`

:0048E56E `FF5134`
`call [ecx+34]`

* Possible StringData Ref from Code Obj -> `[TEX]`

:0048E571 `BA40E74800`
`mov edx, 0048E740`

:0048E576 `8BC3`
`mov eax, ebx`

:0048E578 `8B08`
`mov ecx, dword ptr [eax]`

:0048E57A `FF5134`
`call [ecx+34]`

* Possible StringData Ref from Code Obj -> `C909`

:0048E57D `BA50E74800`
`mov edx, 0048E750`

:0048E582 `8BC3`
`mov eax, ebx`

:0048E584 `8B08`
`mov ecx, dword ptr [eax]`

:0048E586 `FF5134`
`call [ecx+34]`

* Possible StringData Ref from Code Obj -> `ORBITALKID`

:0048E589 `BA60E74800`
`mov edx, 0048E760`

:0048E58E `8BC3`
`mov eax, ebx`

:0048E590 `8B08`
`mov ecx, dword ptr [eax]`

:0048E592 `FF5134`
`call [ecx+34]`

* Possible StringData Ref from Code Obj -> `MEBFATSTEMP`

:0048E595 `BA74E74800`
`mov edx, 0048E774`

:0048E59A `8BC3`
`mov eax, ebx`

:0048E59C `8B08`
`mov ecx, dword ptr [eax]`

:0048E59E `FF5134`
`call [ecx+34]`

:0048E5A1 `8D45F4`
`lea eax, dword ptr [ebp-0C]`

* Possible StringData Ref from Code Obj -> `;C0KE2000;LIDONG;ZOBEL/PC99!;MARGE@EUDORAMAIL.`
* `COM;ZHANGFAN;DAVY-BLIZZARD99;KING;(TEPEX);YAME`
* `EN@PIMAIL.COM.PK;SYS;PINKY AND `
* `THE BRAIN;POTSMOKE;BOBA FETT;WILLIAM;CZY;NAMEL`
* `ESS ONE;HOHO99_DZ@YAHOO.FR;ABRAHAM `
* `LINCOLN;LARRYGO98@YAHOO.COM;WINDOWSIGNS@NOCHAR`
* `GE.ZZN.COM;NELSON01@HOTMAIL.COM;NERDHACKER;NIT`
* `GRILL@YAHOO.COM;WQER@HOTMAIL.COM;ICARUS;SYSTEM`
* `;ANAHARAMY@AOL.COM-3;MP001-231411184785-1;BE_S`
* `A@U` --------------------此处估计是设计者的邮箱

:0048E5A4 `BA88E74800`
`mov edx, 0048E788`

:0048E5A9 `E80257F7FF`
`call 00403CB0`

:0048E5AE `8D45F0`
`lea eax, dword ptr [ebp-10]`

* Possible StringData Ref from Code Obj -> `;81708-ND059-4019484694;`

:0048E5B1 `BA50F04800`
`mov edx, 0048F050`

:0048E5B6 `E8F556F7FF`
`call 00403CB0`

:0048E5BB `8B55FC`
`mov edx, dword ptr [ebp-04]`

:0048E5BE `8BC3`
`mov eax, ebx`

:0048E5C0 `8B08`
`mov ecx, dword ptr [eax]`

:0048E5C2 `FF5150`
`call [ecx+50]`

:0048E5C5 `40`
`inc eax`

:0048E5C6 `7556`
`jne 0048E61E`

:0048E5C8 `6874F04800`
`push 0048F074`

:0048E5CD `FF75FC`
`push [ebp-04]`

:0048E5D0 `6874F04800`
`push 0048F074`

:0048E5D5 `8D45EC`
`lea eax, dword ptr [ebp-14]`

* Possible Reference to String Resource ID=00003: `M?>`

:0048E5D8 `BA03000000`
`mov edx, 00000003`

:0048E5DD `E87259F7FF`
`call 00403F54`

:0048E5E2 `8B45EC`
`mov eax, dword ptr [ebp-14]`

:0048E5E5 `8B55F4`
`mov edx, dword ptr [ebp-0C]`

:0048E5E8 `E88F5BF7FF`
`call 0040417C`

:0048E5ED `85C0`
`test eax, eax`

:0048E5EF `752D`
`jne 0048E61E`

:0048E5F1 `6874F04800`
`push 0048F074`

:0048E5F6 `FF75F8`
`push [ebp-08]`

:0048E5F9 `6874F04800`
`push 0048F074`

:0048E5FE `8D45EC`
`lea eax, dword ptr [ebp-14]`

* Possible Reference to String Resource ID=00003: `M?>`

:0048E601 `BA03000000`
`mov edx, 00000003`

:0048E606 `E84959F7FF`
`call 00403F54`

:0048E60B `8B45EC`
`mov eax, dword ptr [ebp-14]`
-------------- D `EAX`，可见假注册码

:0048E60E `8B55F0`
`mov edx, dword ptr [ebp-10]`
------------- D `EDX`，可见真注册码

:0048E611 `E8665BF7FF`
`call 0040417C`

:0048E616 `85C0`
`test eax, eax`

:0048E618 `7504`
`jne 0048E61E`

:0048E61A `33C0`
`xor eax, eax`
------------- 不等于就将 `EAX` 清 0

:0048E61C `EB02`
`jmp 0048E620`

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
* `:0048E5C6(C)`, `:0048E5EF(C)`, `:0048E618(C)`

:0048E61E `B001`
`mov al, 01`
------------- 已注册标志

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
* `:0048E61C(U)`

:0048E620 `8BD8`
`mov ebx, eax`

:0048E622 `33C0`
`xor eax, eax`

:0048E624 `5A`
`pop edx`

:0048E625 `59`
`pop ecx`

:0048E626 `59`
`pop ecx`

:0048E627 `648910`
`mov dword ptr fs:[eax], edx`

:0048E62A `6844E64800`
`push 0048E644`

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
* `:0048E642(U)`

:0048E62F `8D45EC`
`lea eax, dword ptr [ebp-14]`

* Possible Reference to String Resource ID=00005: `謭`

:0048E632 `BA05000000`
`mov edx, 00000005`

:0048E637 `E80056F7FF`
`call 00403C3C`

:0048E63C `C3`
`ret`

整理一下：用户名：`wdfdiablo`  注册码：`81708-ND059-4019484694`，（可以用 `crackcode2000` 来做注册机）因功力不够写不出注册机，那位大哥帮忙写一下吧？  `4dd466d0e1e0cc123afbe504da2462ca`
