
`.text:00467E3B`   `add` `edx, 6` ; 余数再加 6

;++++++++++++++++++++++++++++++++++++++++++++++++++++++

`.text:00467E3E`

`.text:00467E3E` `loc_467E3E:`   `CODE XREF: sub_4679DB+455j`

`.text:00467E3E`   `push` `edx` ;上面结果就是注册码第 12 位

`.text:00467E3F`   `lea` `eax, [ebp+8]`

`.text:00467E42`   `push` `esi`

`.text:00467E43`   `push` `eax`

`.text:00467E44`   `call` `?Format@CString@@QAAXPBDZZ`  `CString::Format(char const *,...)`

`.text:00467E49`   `add` `esp, 0Ch`

`.text:00467E4C`   `lea` `eax, [ebp+8]`

`.text:00467E4F`   `lea` `ecx, [ebp-10h]`

`.text:00467E52`   `push` `eax`

`.text:00467E53`   `call` `??YCString@@QAEABV0@ABV0@@Z`  `CString::operator+=(CString const &)`

`.text:00467E58`   `mov` `eax, [ebp-14h]` ;用户名

`.text:00467E5B`   `cmp` `dword ptr [eax-8], 7` ;  长度大于 7？

`.text:00467E5F`   `jg`  `short` `loc_467E6D` ;  是，跳

`.text:00467E61`   `movsx`  `eax, byte ptr [eax+2]` ;  否，取用户名第 3 位

`.text:00467E65`   `push` `3`

`.text:00467E67`   `cdq`

`.text:00467E68`   `pop` `ecx`

`.text:00467E69`   `idiv` `ecx` ;  (否)除以 `ecx`(=3)

`.text:00467E6B`   `jmp` `short` `loc_467E79`

`.text:00467E6D`  +++++++++++++++++++++++++++++++++++++++++++++++++++用户名长度大于 7

`.text:00467E6D`

`.text:00467E6D` `loc_467E6D:`   `CODE XREF: sub_4679DB+484j`

`.text:00467E6D`   `movsx` `eax, byte ptr [eax+7]` ;  就取用户名第 8 位

`.text:00467E71`   `cdq`

`.text:00467E72`   `mov` `ecx, ebx`

`.text:00467E74`   `idiv` `ecx` ;     除以 `ecx`(=2)

`.text:00467E76`   `add` `edx, 6` ;       余数加 6

;++++++++++++++++++++++++++++++++++++++++++++++++++++++

`.text:00467E79`

`.text:00467E79` `loc_467E79:`   `CODE XREF: sub_4679DB+490j`

`.text:00467E79`   `push` `edx` ;上面的结果就是注册码第 13 位

`.text:00467E7A`   `lea` `eax, [ebp+8]`

`.text:00467E7D`   `push` `edi`

`.text:00467E7E`   `push` `eax`

`.text:00467E7F`   `call` `?Format@CString@@QAAXPBDZZ`  `CString::Format(char const *,...)`

`.text:00467E84`   `add` `esp, 0Ch`

`.text:00467E87`   `lea` `eax, [ebp+8]`

`.text:00467E8A`   `lea` `ecx, [ebp-10h]`

`.text:00467E8D`   `push` `eax`

`.text:00467E8E`   `call` `??YCString@@QAEABV0@ABV0@@Z`  `CString::operator+=(CString const &)` ;这里连接后注册码形如“xxx-yyyy-zzzz-”

`.text:00467E93`   `mov` `eax, [ebp-14h]` ;用户名

`.text:00467E96`   `cmp` `dword ptr [eax-8], 8` ;  长度大于 8？

`.text:00467E9A`   `jg`  `short` `loc_467EA8` ;  是，跳～

`.text:00467E9C`   `movsx` `eax, byte ptr [eax+4]` ;  否，取用户名第 5 位

`.text:00467EA0`   `push` `9`

`.text:00467EA2`   `cdq`

`.text:00467EA3`   `pop` `ecx`

`.text:00467EA4`   `idiv` `ecx` ;  (否)除以 `ecx`(=9)

`.text:00467EA6`   `jmp` `short` `loc_467EB4`

`.text:00467EA8`  +++++++++++++++++++++++++++++++++++++++++++++++++++用户名长度大于 8

`.text:00467EA8`

`.text:00467EA8` `loc_467EA8:`   `CODE XREF: sub_4679DB+4BFj`

`.text:00467EA8`   `movsx` `eax, byte ptr [eax+8]` ;  就取用户名第 9 位

`.text:00467EAC`   `push` `6`

`.text:00467EAE`   `cdq`

`.text:00467EAF`   `pop` `ecx`

`.text:00467EB0`   `idiv` `ecx` ;   除以 `ecx`(=6)

`.text:00467EB2`   `add` `edx, ebx` ;   余数加 `ebx`(=2)

;++++++++++++++++++++++++++++++++++++++++++++++++++++++

`.text:00467EB4`

`.text:00467EB4` `loc_467EB4:`   `CODE XREF: sub_4679DB+4CBj`

`.text:00467EB4`   `push` `edx` ;注册码第 15 位

`.text:00467EB5`   `lea` `eax, [ebp+8]`

`.text:00467EB8`   `push` `esi`

`.text:00467EB9`   `push` `eax`

`.text:00467EBA`   `call` `?Format@CString@@QAAXPBDZZ`  `CString::Format(char const *,...)`

`.text:00467EBF`   `add` `esp, 0Ch`

`.text:00467EC2`   `lea` `eax, [ebp+8]`

`.text:00467EC5`   `lea` `ecx, [ebp-10h]`

`.text:00467EC8`   `push` `eax`

`.text:00467EC9`   `call` `??YCString@@QAEABV0@ABV0@@Z`  `CString::operator+=(CString const &)`

`.text:00467ECE`   `mov` `eax, [ebp-14h]` ;用户名

`.text:00467ED1`   `cmp` `dword ptr [eax-8], 9` ;  长度大于 9？

`.text:00467ED5`   `jg`  `short` `loc_467EE1` ;  是，跳～

`.text:00467ED7`   `movsx` `eax, byte ptr [eax]` ;  否，取用户名第 1 位

`.text:00467EDA`   `cdq`

`.text:00467EDB`   `mov` `ecx, ebx`

`.text:00467EDD`   `idiv` `ecx` ;   除以 `ecx`(=2)

`.text:00467EDF`   `jmp` `short` `loc_467EEC`

`.text:00467EE1` +++++++++++++++++++++++++++++++++++++++++++++++++++用户名长度大于 9

`.text:00467EE1`

`.text:00467EE1` `loc_467EE1:`   `CODE XREF: sub_4679DB+4FAj`

`.text:00467EE1`   `movsx` `eax, byte ptr [eax+9]` ;   取用户名第 10 位

`.text:00467EE5`   `cdq`

`.text:00467EE6`   `mov` `ecx, ebx`

`.text:00467EE8`   `idiv` `ecx` ;   除以 `ecx`(=2)

`.text:00467EEA`   `add` `edx, ebx` ;   余数加上 `ebx`(=2)

;++++++++++++++++++++++++++++++++++++++++++++++++++++++

`.text:00467EEC`

`.text:00467EEC` `loc_467EEC:`   `CODE XREF: sub_4679DB+504j`

`.text:00467EEC`   `push` `edx` ;注册码第 16 位

`.text:00467EED`   `lea` `eax, [ebp+8]`

`.text:00467EF0`   `push` `esi`

`.text:00467EF1`   `push` `eax`

`.text:00467EF2`   `call` `?Format@CString@@QAAXPBDZZ`  `CString::Format(char const *,...)`

`.text:00467EF7`   `add` `esp, 0Ch`

`.text:00467EFA`   `lea` `eax, [ebp+8]`

`.text:00467EFD`   `lea` `ecx, [ebp-10h]`

`.text:00467F00`   `push` `eax`

`.text:00467F01`   `call` `??YCString@@QAEABV0@ABV0@@Z`  `CString::operator+=(CString const &)`

`.text:00467F06`   `mov` `eax, [ebp-14h]` ;用户名

`.text:00467F09`   `cmp` `dword ptr [eax-8], 0Ah` ;  长度大于 10D？

`.text:00467F0D`   `jg`  `short` `loc_467F1B` ;  是，跳～

`.text:00467F0F`   `movsx` `eax, byte ptr [eax+1]` ;  否，取用户名第 2 位

`.text:00467F13`   `push` `7`

`.text:00467F15`   `cdq`

`.text:00467F16`   `pop` `ecx`

`.text:00467F17`   `idiv` `ecx` ;   除以 `ecx`(=7)

`.text:00467F19`   `jmp` `short` `loc_467F27`

`.text:00467F1B`  +++++++++++++++++++++++++++++++++++++++++++++++++++用户名长度大于 10

`.text:00467F1B`

`.text:00467F1B` `loc_467F1B:`   `CODE XREF: sub_4679DB+532j`

`.text:00467F1B`   `movsx` `eax, byte ptr [eax+0Ah]` ;   取用户名第 11 位

`.text:00467F1F`   `push` `7`

`.text:00467F21`   `cdq`

`.text:00467F22`   `pop` `ecx`

`.text:00467F23`   `idiv` `ecx` ;   除以 `ecx`(=7)

`.text:00467F25`   `add` `edx, ebx` ;   余数加上 `ebx`(=2)

;++++++++++++++++++++++++++++++++++++++++++++++++++++++

`.text:00467F27`

`.text:00467F27` `loc_467F27:`   `CODE XREF: sub_4679DB+53Ej`

`.text:00467F27`   `push` `edx` ;注册码第 17 位

`.text:00467F28`   `lea` `eax, [ebp+8]`

`.text:00467F2B`   `push` `esi`

`.text:00467F2C`   `push` `eax`

`.text:00467F2D`   `call` `?Format@CString@@QAAXPBDZZ`  `CString::Format(char const *,...)`

`.text:00467F32`   `add` `esp, 0Ch`
