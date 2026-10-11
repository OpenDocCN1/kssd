
`SDPI:00481FA1`                   `call`    `StrCompare`                    ; 其中 `CurrentAPI` 指当然要操作的 `API`

`SDPI:00481FA1`                                                       ; `SpecialAPI` 指壳指的特殊 `API`

`SDPI:00481FA6`                   `test`    `eax, eax`

`SDPI:00481FA8`                   `jnz`     `short loc_481FB8`

`SDPI:00481FAA`                   `push`    `offset unk_407A3A`         ; `relativeAddress`

`SDPI:00481FAF`                   `call`    `GetAbsAddress`             ; 获取实际地址

`SDPI:00481FB4`                   `mov`     `[esi], eax`

`SDPI:00481FB6`                   `jmp`     `short loc_481FF6`

`SDPI:00481FB8` ; ----------------------------------------------------------------------------

`SDPI:00481FB8`

`SDPI:00481FB8` `loc_481FB8:`                               ; `CODE XREF: Disposal_IMP+587 j`

`SDPI:00481FB8`                   `mov`     `edx, [esp+444h+hdll]`

`SDPI:00481FBC`                   `push`    `edi`                   ; `APIName`

`SDPI:00481FBD`                   `push`    `edx`                   ; `hdll`

`SDPI:00481FBE`                   `call`    `API_GetProcAddr`

`SDPI:00481FC3`                   `push`    `eax`                   ; `APIAddress`

`SDPI:00481FC4`                   `call`    `HOOK_API_JMP`          ; 加密输入表,把 `call api` 搬到壳里去

`SDPI:00481FC9`                   `mov`     `[esi], eax`           ; 填充运算后的加密地址

`SDPI:00481FCB`                   `jmp`     `short loc_481FF6`

`SDPI:00481FCD` ; ----------------------------------------------------------------------------

`SDPI:00481FCD`

`SDPI:00481FCD` `loc_481FCD:`                               ; `CODE XREF: Disposal_IMP+3CF j`

`SDPI:00481FCD`                                                       ; `Disposal_IMP+3F5 j`

`SDPI:00481FCD`                   `mov`     `eax, [esp+444h+hdll]`

`SDPI:00481FD1`                   `push`    `edi`                   ; `APIName`

`SDPI:00481FD2`                   `push`    `eax`                   ; `hdll`

`SDPI:00481FD3`                   `call`    `API_GetProcAddr`       ; 获取 `API` 地址

`SDPI:00481FD8`                   `mov`     `[esi], eax`           ; 保存 `API`,把特殊 `DLL` 处理改成非特殊 `DLL`

`SDPI:00481FD8`                                                       ; 后直接到这里，然后 `patch` 下就可以获取

`SDPI:00481FD8`                                                       ; 到完整的输入表了

`SDPI:00481FDA`                   `mov`     `ecx, [esp+444h]`

`SDPI:00481FE1`                   `test`    `ecx, ecx`

`SDPI:00481FE3`                   `jnz`     `short loc_481FF6`

`SDPI:00481FE5`                   `mov`     `ecx, [esp+444h+var_428]`

`SDPI:00481FE9`                   `test`    `ecx, ecx`

`SDPI:00481FEB`                   `jz`      `short loc_481FF6`

`SDPI:00481FED`                   `mov`     `[ecx], eax`           ; 把 `api` 保存到申请的空间里

`SDPI:00481FEF`                   `add`     `ecx, 4`

`SDPI:00481FF2`                   `mov`     `[esp+444h+var_428], ecx`

`SDPI:00481FF6`

`SDPI:00481FF6` `loc_481FF6:`                               ; `CODE XREF: Disposal_IMP+3EA j`

`SDPI:00481FF6`                                                       ; `Disposal_IMP+41C j ...`

`SDPI:00481FF6`                   `xor`     `ecx, ecx`

`SDPI:00481FF8`                   `mov`     `cx, [ebx]`             ; 如果要获取完整的输入表，

`SDPI:00481FF8`                                                       ; 则这里不能让壳清除掉

`SDPI:00481FFB`                   `push`    `ecx`                   ; `int`

`SDPI:00481FFC`                   `push`    `0FFh`                  ; `char`

`SDPI:00482001`                   `push`    `edi`                   ; `LPCSTR`

`SDPI:00482002`                   `call`    `Clear_Data`            ; 清除数据

`SDPI:00482007`                   `mov`     `word ptr [ebx], 0`     ; 清除序号名

`SDPI:0048200C`

`SDPI:0048200C` `loc_48200C:`                               ; `CODE XREF: Disposal_IMP+346 j`

`SDPI:0048200C`                                                       ; `Disposal_IMP+36B j ...`

`SDPI:0048200C`                   `cmp`     `ebp, esi`              ; 如果不相等则清除 `ThunkValue`

`SDPI:0048200E`                   `jz`      `short loc_48201A`

`SDPI:00482010`                   `push`    `4`                     ; `int`

`SDPI:00482012`                   `push`    `0`                     ; `char`

`SDPI:00482014`                   `push`    `ebp`                   ; `LPCSTR`

`SDPI:00482015`                   `call`    `Clear_Data`            ; 擦除输入表信息

`SDPI:0048201A`

`SDPI:0048201A` `loc_48201A:`                               ; `CODE XREF: Disposal_IMP+5ED j`

`SDPI:0048201A`                   `mov`     `ebx, [esp+34h]`

`SDPI:0048201E`                   `mov`     `edi, [esp+444h+hMem_IMPVA]`

`SDPI:00482022`                   `add`     `ebp, 4`

`SDPI:00482025`                   `add`     `esi, 4`

`SDPI:00482028`                   `jmp`     `loc_481CE7`            ; 上面载入 `DLL` 后，这里填充当前 `DLL` 的加载函数

`SDPI:00482028`                                                       ; 填充完跳去处理下一个 `DLL`

`SDPI:0048202D` ; ----------------------------------------------------------------------------

`SDPI:0048202D`

`SDPI:0048202D` `DoneCurrDll_48202D:`                         ; `CODE XREF: Disposal_IMP+2CA j`

`SDPI:0048202D`                   `call`    `CRC_480467`

`SDPI:00482032`                   `mov`     `edx, [edi+4]`

`SDPI:00482035`                   `mov`     `eax, [esp+444h+var_420]`   ; 当前 `DLL` 处理完毕则，清除 `DLL` 名

`SDPI:00482039`                   `push`    `edx`                   ; `int`

`SDPI:0048203A`                   `push`    `0FFh`                  ; `char`

`SDPI:0048203F`                   `push`    `eax`                   ; `LPCSTR`

`SDPI:00482040`                   `call`    `Clear_Data`            ; 这里也是擦除输入表信息，

`SDPI:00482040`                                                       ; 不能让壳清除掉输入表信息

`SDPI:00482045`                   `push`    `14h`                   ; `int`

`SDPI:00482047`                   `push`    `0FFh`                  ; `char`

`SDPI:0048204C`                   `push`    `edi`                   ; `LPCSTR`

`SDPI:0048204D`                   `call`    `Clear_Data`            ; 清除数据

`SDPI:00482052`                   `add`     `edi, 14h`

`SDPI:00482055`                   `mov`     `ebp, [esp+2Ch]`

`SDPI:00482059`                   `mov`     `[esp+444h+hMem_IMPVA], edi`

`SDPI:0048205D`                   `jmp`     `loc_481B4A`            ; 获取输入表信息

`SDPI:00482062` ; ----------------------------------------------------------------------------

`SDPI:00482062`

`SDPI:00482062` `loc_482062:`                               ; `CODE XREF: Disposal_IMP+130 j`

`SDPI:00482062`                   `pop`     `edi`                   ; 输入表处理完毕跳来这里

`SDPI:00482063`                   `pop`     `esi`

`SDPI:00482064`                   `pop`     `ebp`

`SDPI:00482065`                   `mov`     `eax, 1`

`SDPI:0048206A`                   `pop`     `ebx`

`SDPI:0048206B`                   `add`     `esp, 430h`

`SDPI:00482071`                   `retn`    `4`

`SDPI:00482071` `Disposal_IMP`    `endp` ; `sp = -4`

`SDPI:00482071`

`SDPI:00482071` ; -----------------------------------------------------------------------

`SDPI:0048233E` ; ----------------------------------------------------------------------------

`SDPI:00482343` `aGetprocesstimes` `db` `'GetProcessTimes',0`

`SDPI:00482353` `aKernel32_dll`   `db` `'Kernel32.dll',0`

`SDPI:00482360`

`SDPI:00482360` ; ************************************************ SUBROUTINE ************************************************

`SDPI:00482360`

`SDPI:00482360`

`SDPI:00482360` `Anti_DBG_482360` `proc near`                       ; `CODE XREF: SDPI:0047AB09 j`

`SDPI:00482360`

`SDPI:00482360` `var_2C`          = `dword ptr -2Ch`

`SDPI:00482360` `var_28`          = `dword ptr -28h`

`SDPI:00482360` `var_24`          = `dword ptr -24h`

`SDPI:00482360` `var_20`          = `dword ptr -20h`

`SDPI:00482360` `var_14`          = `dword ptr -14h`

`SDPI:00482360` `var_C`           = `dword ptr -0Ch`

`SDPI:00482360`

`SDPI:00482360`                   `sub`     `esp, 10h`

`SDPI:00482363`                   `push`    `esi`

`SDPI:00482364`                   `call`    `Get_Version`           ; 获取操作系统

`SDPI:00482369`                   `cmp`     `eax, 80000000h`
```

```
