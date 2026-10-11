
# ToggleMOUSE 4.4.6

> 原文：[`www.pediy.com/kssd/pediy01/forum109.htm`](https://www.pediy.com/kssd/pediy01/forum109.htm)

`http://www.toggle.com`

这是个很好的鼠标增强工具。它可以输入注册码，不过这里我们不是从输入注册码时开始跟踪，而是跟踪它启动时的判断过程，从而找到注册码。

用 `RegMon` 看一下，发现它启动的时候要读如下几个键：

`HKEY_CURRENT_USER\Software\ToggleSoftware\ToggleMOUSE\Registration\Name`
`HKEY_CURRENT_USER\Software\ToggleSoftware\ToggleMOUSE\Registration\Company`
`HKEY_CURRENT_USER\Software\ToggleSoftware\ToggleMOUSE\Registration\RegNumber`

反汇编其主程序，找到如下的代码片段：

```
* Possible StringData Ref from Data Obj ->"Registration"
                              |
:004108AB BEB8F64300             mov esi, 0043F6B8
* Possible StringData Ref from Data Obj ->"RegNumber"
                              |
:004108B0 68ACF64300             push 0043F6AC
:004108B5 56                     push esi
:004108B6 8BC8                   mov ecx, eax
:004108B8 E83AE30100             call 0042EBF7    //读取注册码
:004108BD 894508                 mov dword ptr [ebp+08], eax  //保存注册码
:004108C0 E82DDF0100             call 0042E7F2
:004108C5 8B4004                 mov eax, dword ptr [eax+04]
:004108C8 53                     push ebx
* Possible StringData Ref from Data Obj ->"Name"
                              |
:004108C9 68A4F64300             push 0043F6A4
:004108CE 8D4DE8                 lea ecx, dword ptr [ebp-18]
:004108D1 56                     push esi
:004108D2 51                     push ecx
:004108D3 8BC8                   mov ecx, eax
:004108D5 E889E30100             call 0042EC63    //读取 Name
:004108DA 50                     push eax
:004108DB 8D4DF0                 lea ecx, dword ptr [ebp-10]
:004108DE E880480100             call 00425163
:004108E3 8D4DE8                 lea ecx, dword ptr [ebp-18]
:004108E6 E88B470100             call 00425076
:004108EB E802DF0100             call 0042E7F2
:004108F0 8B4004                 mov eax, dword ptr [eax+04]
:004108F3 53                     push ebx
* Possible StringData Ref from Data Obj ->"Company"
                              |
:004108F4 689CF64300             push 0043F69C
:004108F9 8D4DE8                 lea ecx, dword ptr [ebp-18]
:004108FC 56                     push esi
:004108FD 51                     push ecx
:004108FE 8BC8                   mov ecx, eax
:00410900 E85EE30100             call 0042EC63   //读取 Company
:00410905 50                     push eax
:00410906 8D4DEC                 lea ecx, dword ptr [ebp-14]
:00410909 E855480100             call 00425163
:0041090E 8D4DE8                 lea ecx, dword ptr [ebp-18]
:00410911 E860470100             call 00425076
:00410916 E8D7DE0100             call 0042E7F2
:0041091B 8B4004                 mov eax, dword ptr [eax+04]
:0041091E 33DB                   xor ebx, ebx
:00410920 53                     push ebx
* Possible StringData Ref from Data Obj ->"LastPageIndex"
                              |
:00410921 688CF64300             push 0043F68C
* Possible StringData Ref from Data Obj ->"Settings"
                              |
:00410926 685CE34300             push 0043E35C
:0041092B 8BC8                   mov ecx, eax
:0041092D E8C5E20100             call 0042EBF7 //读取注册码的校验和
:00410932 8945E8                 mov dword ptr [ebp-18], eax //保存校验和
:00410935 8D45F0                 lea eax, dword ptr [ebp-10]
:00410938 50                     push eax
:00410939 8D4DF8                 lea ecx, dword ptr [ebp-08]
:0041093C E822480100             call 00425163
:00410941 8D4DF8                 lea ecx, dword ptr [ebp-08]
:00410944 E8E34B0100             call 0042552C
:00410949 8B45F8                 mov eax, dword ptr [ebp-08]
:0041094C 8B37                   mov esi, dword ptr [edi]
:0041094E 3958F8                 cmp dword ptr [eax-08], ebx
:00410951 7E21                   jle 00410974
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00410972(C)
|
:00410953 0FBE0403               movsx eax, byte ptr [ebx+eax] //以下计算注册码
:00410957 50                     push eax
:00410958 E8B27A0000             call 0041840F
:0041095D 85C0                   test eax, eax
:0041095F 8B45F8                 mov eax, dword ptr [ebp-08]
:00410962 59                     pop ecx
:00410963 7409                   je 0041096E
:00410965 0FBE0C03               movsx ecx, byte ptr [ebx+eax]
:00410969 03CB                   add ecx, ebx
:0041096B 0FAFF1                 imul esi, ecx
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00410963(C)
|
:0041096E 43                     inc ebx
:0041096F 3B58F8                 cmp ebx, dword ptr [eax-08]
:00410972 7CDF                   jl 00410953     //循环
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00410951(C)
:00410974 6A64                   push 00000064
:00410976 8BC6                   mov eax, esi
:00410978 33D2                   xor edx, edx
:0041097A 59                     pop ecx
:0041097B F7F1                   div ecx
:0041097D 8B4D08                 mov ecx, dword ptr [ebp+08] //你输入的假注册码
:00410980 81F121332153           xor ecx, 53213321
:00410986 3B4DE8                 cmp ecx, dword ptr [ebp-18] //检查校验和
:00410989 7512                   jne 0041099D
:0041098B 394508                 cmp dword ptr [ebp+08], eax //比较真假注册码
:0041098E 750D                   jne 0041099D
:00410990 85C0                   test eax, eax
:00410992 7409                   je 0041099D
:00410994 C7470801000000         mov [edi+08], 00000001 //good guy
:0041099B EB04                   jmp 004109A1
* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:
|:00410989(C), :0041098E(C), :00410992(C)
|
:0041099D 83670800               and dword ptr [edi+08],00000000  //bag guy
```

根据以上代码可知校验和等于注册码和常数 `0x53213321` 异或，校验和放在这里：

`HKEY_CURRENT_USER\Software\ToggleSoftware\ToggleMOUSE\Settings\LastPageIndex`

注册机也很好写，因其计算过程很简单。

自 `4.5` 版本开始，其 `Trial version` 和 `Licensed version` 是分开的，即 `trial` 版无法成为注册版了，而 `Licensed version` 是可以输入注册码的，但是需要注册码才能下载得到。`4.5.3` 版本用如下两个键来判断是否过期：

`HKEY_CURRENT_USER\Software\ToggleSoftware\ToggleMOUSE\Settings\ScrollSensitivity`
`HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Explorer\s363`

其中 `s363` 这个键最初没有。当剩下的天数小于 `0x1C` 时它会生成这个键。

相关的代码如下：

```
* Referenced by a CALL at Addresses:
|:00406978  , :0040E480  , :00413DBC  , :00414548  , :00415A4D  
|
:004132BC 55                     push ebp
:004132BD 8BEC                   mov ebp, esp
:004132BF 51                     push ecx
:004132C0 51                     push ecx
:004132C1 A1F84C4400             mov eax, dword ptr [00444CF8]
:004132C6 53                     push ebx
:004132C7 56                     push esi
:004132C8 8BF1                   mov esi, ecx
:004132CA 8945FC                 mov dword ptr [ebp-04], eax
:004132CD 8945F8                 mov dword ptr [ebp-08], eax
:004132D0 8B4508                 mov eax, dword ptr [ebp+08]
:004132D3 57                     push edi
:004132D4 8906                   mov dword ptr [esi], eax
:004132D6 E8E3DC0100             call 00430FBE
:004132DB 8B4004                 mov eax, dword ptr [eax+04]
:004132DE BF50160000             mov edi, 00001650
:004132E3 57                     push edi
* Possible StringData Ref from Data Obj ->"Settings"
                              |
```



:004132E4 `BB8C244400`  mov `ebx`, `0044248C`   

* `Possible StringData Ref from Data Obj -> "ScrollSensitivity"`   

   |   

:004132E9 `68843E4400`  push `00443E84`   

:004132EE `53`  push `ebx`   

:004132EF `8BC8`  mov `ecx`, `eax`   

:004132F1 `E86ED80100`  call `00430B64`  //读取安装日期   

:004132F6 `3BC7`  cmp `eax`, `edi`   

:004132F8 `894604`  mov dword ptr `[esi+04]`, `eax`   

:004132FB `7523`  jne `00413320`   

:004132FD `6A00`  push `00000000`   

:004132FF `E814740000`  call `0041A718`   

:00413304 `59`  pop `ecx`   

:00413305 `894604`  mov dword ptr `[esi+04]`, `eax`   

:00413308 `E8B1DC0100`  call `00430FBE`   

:0041330D `FF7604`  push `[esi+04]`   

:00413310 `8B4004`  mov `eax`, dword ptr `[eax+04]`   

:00413313 `8BC8`  mov `ecx`, `eax`   

* `Possible StringData Ref from Data Obj -> "ScrollSensitivity"`   

   |   

:00413315 `68843E4400`  push `00443E84`   

:0041331A `53`  push `ebx`   

:0041331B `E830B60100`  call `0042E950` //读取安装日期   

* `Referenced by a (U)nconditional or (C)onditional Jump at Address:`   

`|:004132FB(C)`   

   |   

:00413320 `8B06`  mov `eax`, dword ptr `[esi]`   

:00413322 `69C09F410000`  imul `eax`, `0000419F`   

:00413328 `50`  push `eax`   

:00413329 `E8B6620000`  call `004195E4`   

:0041332E `99`  cdq   

:0041332F `B910270000`  mov `ecx`, `00002710`   

:00413334 `F7F9`  idiv `ecx`   

:00413336 `8D45F8`  lea `eax`, dword ptr `[ebp-08]`   

:00413339 `52`  push `edx`   

* `Possible StringData Ref from Data Obj -> "s%u"`   

   |   

:0041333A `68803E4400`  push `00443E80`   

:0041333F `50`  push `eax`   

:00413340 `E8FF260100`  call `00425A44` //生成键名"s363"   

:00413345 `83C40C`  add `esp`, `0000000C`   

:00413348 `8D45F8`  lea `eax`, dword ptr `[ebp-08]`   

:0041334B `8BCC`  mov `ecx`, `esp`   

:0041334D `50`  push `eax`   

:0041334E `E8B03F0100`  call `00427303`   

:00413353 `51`  push `ecx`   

* `Possible StringData Ref from Data Obj  -> "Software\\Microsoft\\Windows\\CurrentVersion\\Explorer"`   

   |   

:00413354 `BB4C3E4400`  mov `ebx`, `00443E4C`   

:00413359 `8BCC`  mov `ecx`, `esp`   

:0041335B `53`  push `ebx`   

:0041335C `E89B420100`  call `004275FC` //读取 s363 的值   

:00413361 `BF01000080`  mov `edi`, `80000001`   

:00413366 `8D4508`  lea `eax`, dword ptr `[ebp+08]`   

:00413369 `57`  push `edi`   

:0041336A `50`  push `eax`   

:0041336B `E893310000`  call `00416503`   

:00413370 `83C410`  add `esp`, `00000010`   

:00413373 `8D4DFC`  lea `ecx`, dword ptr `[ebp-04]`   

:00413376 `50`  push `eax`   

:00413377 `E8FF420100`  call `0042767B`   

:0041337C `8D4D08`  lea `ecx`, dword ptr `[ebp+08]`   

:0041337F `E80A420100`  call `0042758E`   

:00413384 `FF75FC`  push `[ebp-04]`   

:00413387 `E853670000`  call `00419ADF`   

:0041338C `85C0`  test `eax`, `eax`   

:0041338E `59`  pop `ecx`   

:0041338F `7425`  je `004133B6`   

:00413391 `8B4604`  mov `eax`, dword ptr `[esi+04]`   

:00413394 `FF75FC`  push `[ebp-04]`   

:00413397 `894508`  mov dword ptr `[ebp+08]`, `eax`   

:0041339A `E840670000`  call `00419ADF`   

:0041339F `3B4508`  cmp `eax`, dword ptr `[ebp+08]`   

:004133A2 `59`  pop `ecx`   

:004133A3 `7D0B`  jge `004133B0` //这里一定要跳   

* `Referenced by a (U)nconditional or (C)onditional Jump at Address:`   

`|:004133A3(C)`   

   |   

:004133B0 `8B4508`  mov `eax`, dword ptr `[ebp+08]`   

* `Referenced by a (U)nconditional or (C)onditional Jump at Address:`   

`|:004133AE(U)`   

   |   

:004133B3 `894604`  mov dword ptr `[esi+04]`, `eax`   

* `Referenced by a (U)nconditional or (C)onditional Jump at Address:`   

`|:0041338F(C)`   

   |   

:004133B6 `8BCE`  mov `ecx`, `esi`   

:004133B8 `E86D000000`  call `0041342A` //计算剩下的天数   

:004133BD `83F81C`  cmp `eax`, `0000001C`   

:004133C0 `7D4F`  jge `00413411`  //让它跳走   

:004133C2 `68A0664400`  push `004466A0`   

:004133C7 `FF75FC`  push `[ebp-04]`   

:004133CA `E820620000`  call `004195EF`   

:004133CF `59`  pop `ecx`   

:004133D0 `85C0`  test `eax`, `eax`   

:004133D2 `59`  pop `ecx`   

:004133D3 `753C`  jne `00413411`   

:004133D5 `FF7604`  push `[esi+04]`   

:004133D8 `8D45FC`  lea `eax`, dword ptr `[ebp-04]`   

* `Possible StringData Ref from Data Obj -> "%d"`   

   |   

:004133DB `6818254400`  push `00442518`   

:004133E0 `50`  push `eax`   

:004133E1 `E85E260100`  call `00425A44` //写入 s363   

:004133E6 `59`  pop `ecx`   

:004133E7 `8D45FC`  lea `eax`, dword ptr `[ebp-04]`   

:004133EA `59`  pop `ecx`   

:004133EB `8BCC`  mov `ecx`, `esp`   

:004133ED `50`  push `eax`   

:004133EE `E8103F0100`  call `00427303`   

:004133F3 `51`  push `ecx`   

:004133F4 `8D45F8`  lea `eax`, dword ptr `[ebp-08]`   

:004133F7 `8BCC`  mov `ecx`, `esp`   

:004133F9 `50`  push `eax`   

:004133FA `E8043F0100`  call `00427303`   

:004133FF `51`  push `ecx`   

:00413400 `8BCC`  mov `ecx`, `esp`   

:00413402 `53`  push `ebx`   

:00413403 `E8F4410100`  call `004275FC`   

:00413408 `57`  push `edi`   

:00413409 `E897310000`  call `004165A5`   

:0041340E `83C410`  add `esp`, `00000010`   

* `Referenced by a (U)nconditional or (C)onditional Jump at Addresses:`   

`|:004133C0(C), :004133D3(C)`   

   |   

:00413411 `8D4DF8`  lea `ecx`, dword ptr `[ebp-08]`   

:00413414 `E875410100`  call `0042758E`   

:00413419 `8D4DFC`  lea `ecx`, dword ptr `[ebp-04]`   

:0041341C `E86D410100`  call `0042758E`   

:00413421 `8BC6`  mov `eax`, `esi`   

:00413423 `5F`  pop `edi`   

:00413424 `5E`  pop `esi`   

:00413425 `5B`  pop `ebx`   

:00413426 `C9`  leave   

:00413427 `C20400`  ret `0004`
