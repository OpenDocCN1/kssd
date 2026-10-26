
`00402C6D`    `83E1` `03`         `and`     `ecx`, `3`     ；与运算`ecx`, `3`

`00402C70`    `83C6` `03`         `add`     `esi`, `3`     ；`esi+` `3`

`EAX` `00000001`

`ECX` `00000001`

`EDX` `00FC4FB8`

`EBX` `0012F794`

`ESP` `0012F758`

`EBP` `0012F798`

`ESI` `0012F791`

`EDI` `00FC4FB5`

`EIP` `00402C73` `AspAppEn.00402C73`

`00402C73`    `83C7` `03`         `add`     `edi`, `3`     ；`edi+` `3`

`EAX` `00000001`

`ECX` `00000001`

`EDX` `00FC4FB8`

`EBX` `0012F794`

`ESP` `0012F758`

`EBP` `0012F798`

`ESI` `0012F791`

`EDI` `00FC4FB8`

`EIP` `00402C76` `AspAppEn.00402C76`

`00402C76`    `F3:A>` `rep`        `movs` `byte ptr es:[edi]`, `byte ptr [esi]`       ；重复前缀，不知道是什么意思

`EAX` `00000001`

`ECX` `00000000`

`EDX` `00FC4FB8`

`EBX` `0012F794`

`ESP` `0012F758`

`EBP` `0012F798`

`ESI` `0012F790`

`EDI` `00FC4FB7`

`EIP` `00402C78` `AspAppEn.00402C78`

`00402C78`    `FC`              `cld`               ；处理器控制指令－标志位操作指令，不知道什么意思

`00402C79`    `5F`              `pop`     `edi`     ；`EDI`出栈  堆栈 [`0012F758`]=`00FC4FB8`

`EAX` `00000001`

`ECX` `00000000`

`EDX` `00FC4FB8`

`EBX` `0012F794`

`ESP` `0012F75C`

`EBP` `0012F798`

`ESI` `0012F790`

`EDI` `00FC4FB8`

`EIP` `00402C7A` `AspAppEn.00402C7A`

`00402C7A`    `5E`              `pop`     `esi`     ；`ESI`出栈  堆栈 [`0012F75C`]=`0012F791`

`EAX` `00000001`

`ECX` `00000000`

`EDX` `00FC4FB8`

`EBX` `0012F794`

`ESP` `0012F760`

`EBP` `0012F798`

`ESI` `0012F791`

`EDI` `00FC4FB8`

`EIP` `00402C7B` `AspAppEn.00402C7B`

`00402C7B`    `C3`              `retn`     ；返回到 `00404DA9`

`EAX` `00000001`

`ECX` `00000000`

`EDX` `00FC4FB8`

`EBX` `0012F794`

`ESP` `0012F764`

`EBP` `0012F798`

`ESI` `0012F791`

`EDI` `00FC4FB8`

`EIP` `00404DA9` `AspAppEn.00404DA9`

`00404DA9`    `89D8`  `mov`     `eax`, `ebx`      ；将`EBX`传送给`EAX`

`EAX` `0012F794`

`ECX` `00000000`

`EDX` `00FC4FB8`

`EBX` `0012F794`

`ESP` `0012F764`

`EBP` `0012F798`

`ESI` `0012F791`

`EDI` `00FC4FB8`

`EIP` `00404DAB` `AspAppEn.00404DAB`

`00404DAB`    `E8` `E8>` `call`    `00404C98`       ；转入`00404C98`处理

`EAX` `0012F794`

`ECX` `00000000`

`EDX` `00FC4FB8`

`EBX` `0012F794`

`ESP` `0012F760`

`EBP` `0012F798`

`ESI` `0012F791`

`EDI` `00FC4FB8`

`EIP` `00404C98` `AspAppEn.00404C98`

`00404C98`    `8B10`            `mov`     `edx`, `dword ptr [eax]`     ；堆栈`ds:[0012F794]`=`00000000`

`EAX` `0012F794`

`ECX` `00000000`

`EDX` `00000000`

`EBX` `0012F794`

`ESP` `0012F760`

`EBP` `0012F798`

`ESI` `0012F791`

`EDI` `00FC4FB8`

`EIP` `00404C9A` `AspAppEn.00404C9A`

`00404C9A`    `85D2`            `test`    `edx`, `edx`

`00404C9C`    `74` `1C`           `je`      `short` `00404CBA`     ；跳转已实现 到`00404CBA`

.

.

.

`00404CBA`    `C3`              `retn`     ；返回到 `00404DB0`

`EAX` `0012F794`

`ECX` `00000000`

`EDX` `00000000`

`EBX` `0012F794`

`ESP` `0012F764`

`EBP` `0012F798`

`ESI` `0012F791`

`EDI` `00FC5138`

`EIP` `00404DB0` `AspAppEn.00404DB0`

`00404DB0`    `893B`  `mov`     `dword ptr [ebx]`, `edi`     ；`edi`=`00FC4FB8` 堆栈 `ds:[0012F794]`=`00000000`

`00404DB2`    `5F`    `pop`     `edi`      ；`EDI`出栈

`EAX` `0012F794`

`ECX` `00000000`

`EDX` `00000000`

`EBX` `0012F794`

`ESP` `0012F768`

`EBP` `0012F798`

`ESI` `0012F791`

`EDI` `0012F84C`

`EIP` `00404DB3` `AspAppEn.00404DB3`

`00404DB3`    `5E`    `pop`     `esi`   ；`ESI`出栈   堆栈 [`0012F768`]=`00F66460`

`EAX` `0012F794`

`ECX` `00000000`

`EDX` `00000000`

`EBX` `0012F794`

`ESP` `0012F76C`

`EBP` `0012F798`

`ESI` `00F66460`

`EDI` `0012F84C`

`EIP` `00404DB4` `AspAppEn.00404DB4`

`00404DB4`    `5B`    `pop`     `ebx`     ；`EBX`出栈   堆栈 [`0012F76C`]=`00000014`

`EAX` `0012F794`

`ECX` `00000000`

`EDX` `00000000`

`EBX` `00000014`

`ESP` `0012F770`

`EBP` `0012F798`

`ESI` `00F66460`

`EDI` `0012F84C`

`EIP` `00404DB5` `AspAppEn.00404DB5`

`00404DB5`    `C3`    `retn`      ；返回到 `004FB47C`

`EAX` `0012F794`

`ECX` `00000000`

`EDX` `00000000`

`EBX` `00000014`

`ESP` `0012F774`

`EBP` `0012F798`

`ESI` `00F66460`

`EDI` `0012F84C`

`EIP` `004FB47C` `AspAppEn.004FB47C`

`004FB47C`    `8D45` >`lea`     `eax`, `dword ptr [ebp-4]`     ；堆栈地址=`EBP-4`=`0012F794`

`004FB47F`    `50`    `push`    `eax`

`EAX` `0012F794`

`ECX` `00000000`

`EDX` `00000000`

`EBX` `00000014`

`ESP` `0012F770`

`EBP` `0012F798`

`ESI` `00F66460`

`EDI` `0012F84C`

`EIP` `004FB480` `AspAppEn.004FB480`

`004FB480`    `8D45` >`lea`     `eax`, `dword ptr [ebp-C]`    ；`ebp-C` = `0012F78C`  有效地址传送

`EAX` `0012F78C`

`ECX` `00000000`

`EDX` `00000000`

`EBX` `00000014`

`ESP` `0012F770`

`EBP` `0012F798`

`ESI` `00F66460`

`EDI` `0012F84C`

`EIP` `004FB483` `AspAppEn.004FB483`

`004FB483`    `8BD7`  `mov`     `edx`, `edi`     ；将`EDI`传送给`EDX`

`EAX` `0012F78C`

`ECX` `00000000`

`EDX` `0012F84C`

`EBX` `00000014`

`ESP` `0012F770`

`EBP` `0012F798`

`ESI` `00F66460`

`EDI` `0012F84C`

`EIP` `004FB485` `AspAppEn.004FB485`

`004FB485`    `E8` `6A>` `call`    `00404EF4`     ；继续转入`00404EF4`处理子程序

和上面一样，（这里略...)运行完寄存器如下:

`EAX` `0012F78C`

`ECX` `00000000`

`EDX` `00000000`

`EBX` `00000014`

`ESP` `0012F770`

`EBP` `0012F798`

`ESI` `00F66460`

`EDI` `0012F84C`

`EIP` `004FB48A` `AspAppEn.004FB48A`

`004FB48A`    `8B55` >`mov`     `edx`, `dword ptr [ebp-C]`     ；`ebp-C`=`0012F78C` 堆栈 `ss:[0012F78C]`=`00000000`

`004FB48D`    `58`    `pop`     `eax`         ；堆栈 [`0012F770`]=`0012F794`

`EAX` `0012F794`

`ECX` `00000000`

`EDX` `00000000`

`EBX` `00000014`

`ESP` `0012F774`

`EBP` `0012F798`

`ESI` `00F66460`

`EDI` `0012F84C`

`EIP` `004FB48E` `AspAppEn.004FB48E`

`004FB48E`    `E8` `C5>` `call`    `00404F58`        ；转入`00404F58`处理子程序

`00404F58`    `85D2`            `test`    `edx`, `edx`

`00404F5A`    `74` `3F`           `je`      `short` `00404F9B`     跳转实现,转到`00404F8F`

.

.

.

`00404F8F`    `C3`              `retn`     ；返回到 `004FB493`

        `EAX` `0012F794`

`ECX` `00000000`

`EDX` `00000000`

`EBX` `00000014`

`ESP` `0012F774`

`EBP` `0012F798`

`ESI` `00F66460`

`EDI` `0012F84C`

`EIP` `004FB493` `AspAppEn.004FB493`

`004FB493`    `8B55` >`mov`     `edx`, `dword ptr [ebp-4]`     ；`ebp-4`=`0012F794` 堆栈 `ss:[0012F794]`=`00FC5138`

`EAX` `0012F794`

`ECX` `00000000`

`EDX` `00FC5138`

`EBX` `00000014`

`ESP` `0012F774`

`EBP` `0012F798`

`ESI` `00F66460`

`EDI` `0012F84C`

`EIP` `004FB496` `AspAppEn.004FB496`

`004FB496`    `8BC7`  `mov`     `eax`, `edi`     ；将`EDI`传送给`EAX`

`EAX` `0012F84C`

`ECX` `00000000`

`EDX` `00FC5138`

`EBX` `00000014`

`ESP` `0012F774`

`EBP` `0012F798`

`ESI` `00F66460`

`EDI` `0012F84C`

`EIP` `004FB498` `AspAppEn.004FB498`

`004FB498`    `B9` `FF>` `mov`     `ecx`, `0FF`     ；将`0FF`（十进制：255）传送给`ECX`

`EAX` `0012F84C`

`ECX` `000000FF`

`EDX` `00FC5138`

`EBX` `00000014`

`ESP` `0012F774`

`EBP` `0012F798`

`ESI` `00F66460`

`EDI` `0012F84C`

`EIP` `004FB49D` `AspAppEn.004FB49D`

`004FB49D`    `E8` `8A>` `call`    `00404F2C`     ；转入`00404F2C`处理子程序

`EAX` `0012F84C`

`ECX` `000000FF`

`EDX` `00FC5138`

`EBX` `00000014`

`ESP` `0012F770`

`EBP` `0012F798`

`ESI` `00F66460`

`EDI` `0012F84C`

`EIP` `00404F2C` `AspAppEn.00404F2C`

`00404F2C`    `53`              `push`    `ebx`     ；`EBX`入栈

`EAX` `0012F84C`

`ECX` `000000FF`

`EDX` `00FC5138`

`EBX` `00000014`

`ESP` `0012F76C`

`EBP` `0012F798`

`ESI` `00F66460`

`EDI` `0012F84C`

`EIP` `00404F2D` `AspAppEn.00404F2D`

`00404F2D`    `85D2`            `test`    `edx`, `edx`

`00404F2F`    `74` `18`           `je`      `short` `00404F49`     ；跳转未实现

`00404F31`    `8B5A` `FC`         `mov`     `ebx`, `dword ptr [edx-4]`     ；`edx-4` = `00FC5134` `ds:[00FC5134]`=`00000001`

`EAX` `0012F84C`

`ECX` `000000FF`

`EDX` `00FC5138`

`EBX` `00000001`

`ESP` `0012F76C`

`EBP` `0012F798`

`ESI` `00F66460`

`EDI` `0012F84C`

`EIP` `00404F34` `AspAppEn.00404F34`

`00404F34`    `85DB`            `test`    `ebx`, `ebx`

`00404F36`    `74` `11`           `je`      `short` `00404F49`     ；跳转未实现

`00404F38`    `39D9`            `cmp`     `ecx`, `ebx`     ；比较`EBX`和`ECX`

`EAX` `0012F84C`

`ECX` `000000FF`

`EDX` `00FC5138`

`EBX` `00000001`

`ESP` `0012F76C`

`EBP` `0012F798`

`ESI` `00F66460`

`EDI` `0012F84C`

`00404F3A`    `7C` `02`           `jl`      `short` `00404F3E`     ；跳转未实现

`00404F3C`    `89D9`            `mov`     `ecx`, `ebx`     ；将`EBX`传送给`ECX`

`EAX` `0012F84C`

`ECX` `00000001`

`EDX` `00FC5138`

`EBX` `00000001`

`ESP` `0012F76C`

`EBP` `0012F798`

`ESI` `00F66460`

`EDI` `0012F84C`

`EIP` `00404F3E` `AspAppEn.00404F3E`

`00404F3E`    `8808`            `mov`     `byte ptr [eax]`, `cl`     ；将`CL`传送到`byte ptr [eax]` `cl`=`01` 堆栈 `ds:[0012F84C]`=`00`运行后`0012F84C`=`01`

`00404F40`    `40`              `inc`     `eax`         ；`EAX+1`

`EAX` `0012F84D`

`ECX` `00000001`

`EDX` `00FC5138`

`EBX` `00000001`

`ESP` `0012F76C`

`EBP` `0012F798`

`ESI` `00F66460`

`EDI` `0012F84C`

`EIP` `00404F41` `AspAppEn.00404F41`

`00404F41`    `92`              `xchg`    `eax`, `edx`     数据交换，`EDX`和`EAX`数据交换

`EAX` `00FC5138`

`ECX` `00000001`

`EDX` `0012F84D`

`EBX` `00000001`

`ESP` `0012F76C`

`EBP` `0012F798`

`ESI` `00F66460`

`EDI` `0012F84C`

`EIP` `00404F42` `AspAppEn.00404F42`

`00404F42`    `E8` `F5DCFFFF`     `call`    `00402C3C`     ；继续转入`00402C3C`处理子程序

`00402C3C`    `56`              `push`    `esi`

`EAX` `00FC5138`

`ECX` `00000001`

`EDX` `0012F84D`

`EBX` `00000001`

`ESP` `0012F764`

`EBP` `0012F798`

`ESI` `00F66460`

`EDI` `0012F84C`

`EIP` `00402C3D` `AspAppEn.00402C3D`

`00402C3D`    `57`              `push`    `edi`

`EAX` `00FC5138`

`ECX` `00000001`

`EDX` `0012F84D`

`EBX` `00000001`

`ESP` `0012F760`

`EBP` `0012F798`

`ESI` `00F66460`

`EDI` `0012F84C`

`EIP` `00402C3E` `AspAppEn.00402C3E`

`00402C3E`    `89C6`            `mov`     `esi`, `eax`

`EAX` `00FC5138`

`ECX` `00000001`

`EDX` `0012F84D`

`EBX` `00000001`

`ESP` `0012F760`

`EBP` `0012F798`

`ESI` `00FC5138`

`EDI` `0012F84C`

`EIP` `00402C40` `AspAppEn.00402C40`

`00402C40`    `89D7`            `mov`     `edi`, `edx`

`EAX` `00FC5138`
