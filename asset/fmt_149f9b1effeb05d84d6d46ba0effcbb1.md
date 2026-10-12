
* `Possible StringData Ref from Code Obj ->"  days left."`                -->试用标志

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; `|`

`:00432C94` `68A0374100`           `push 004137A0`

* `Reference To: MSVBVM50.__vbaStrCat, Ord:0000h`

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; `|`

`:00432C99` `FF150C334600`         `Call dword ptr [0046330C]`

`:00432C9F` `8BD0`                 `mov edx, eax`

`:00432CA1` `8D4DBC`               `lea ecx, dword ptr [ebp-44]`

* `Reference To: MSVBVM50.__vbaStrMove, Ord:0000h`

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; `|`

`:00432CA4` `FF15D0344600`         `Call dword ptr [004634D0]`

`:00432CAA` `50`                   `push eax`

`:00432CAB` `8B4D80`               `mov ecx, dword ptr [ebp-80]`

`:00432CAE` `8B11`                 `mov edx, dword ptr [ecx]`

`:00432CB0` `8B4580`               `mov eax, dword ptr [ebp-80]`

`:00432CB3` `50`                   `push eax`

`:00432CB4` `FF5254`               `call [edx+54]`

`:00432CB7` `89857CFFFFFF`         `mov dword ptr [ebp+FFFFFF7C], eax`

`:00432CBD` `83BD7CFFFFFF00`       `cmp dword ptr [ebp+FFFFFF7C], 00000000`

`:00432CC4` `7D20`                 `jge 00432CE6`

`:00432CC6` `6A54`                 `push 00000054`

* `Possible StringData Ref from Code Obj ->"貼?檉??"`

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; `|`

`:00432CC8` `6814224100`           `push 00412214`

`:00432CCD` `8B4D80`               `mov ecx, dword ptr [ebp-80]`

`:00432CD0` `51`                   `push ecx`

`:00432CD1` `8B957CFFFFFF`         `mov edx, dword ptr [ebp+FFFFFF7C]`

`:00432CD7` `52`                   `push edx`

* `Reference To: MSVBVM50.__vbaHresultCheckObj, Ord:0000h`

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; `|`

`:00432CD8` `FF1524334600`         `Call dword ptr [00463324]`

`:00432CDE` `89855CFFFFFF`         `mov dword ptr [ebp+FFFFFF5C], eax`

`:00432CE4` `EB0A`                 `jmp 00432CF0`

* `Referenced by a (U)nconditional or (C)onditional Jump at Address:`

`|:00432CC4(C)`

`|`

`:00432CE6` `C7855CFFFFFF00000000` `mov dword ptr [ebp+FFFFFF5C], 00000000`

* `Referenced by a (U)nconditional or (C)onditional Jump at Address:`

`|:00432CE4(U)`

`|`

`:00432CF0` `8D45BC`               `lea eax, dword ptr [ebp-44]`

`:00432CF3` `50`                   `push eax`

`:00432CF4` `8D4DC0`               `lea ecx, dword ptr [ebp-40]`

`:00432CF7` `51`                   `push ecx`

`:00432CF8` `8D55C4`               `lea edx, dword ptr [ebp-3C]`

`:00432CFB` `52`                   `push edx`

`:00432CFC` `6A03`                 `push 00000003`

`conanxu[BCG]`

`conanxu@eastday.com` `4dd466d0e1e0cc123afbe504da2462ca`
