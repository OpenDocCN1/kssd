
# PE EXPLORER 1.3

> 原文：[`www.pediy.com/kssd/pediy03/forum1098.htm`](https://www.pediy.com/kssd/pediy03/forum1098.htm)

如何完美破解 PE EXPLORER 1.3（加入 BCG 的第二篇）

PE EXPLORER 1.3 是个非常不错的 CRACK 辅助软件，可以对 PE 格式的文件进行反编译、属性编辑等！

- **破解人**：大老
- **注册方式**：注册码
- **使用限制**：30 天试用
- **工具**：`TRW2000`、`W32DASM`、`大老专版`
- **大老的主页**：`http://dalao2001.yeah.net`
- **软件下载**：`www.exetools`

这是我第二次在破解这个程序，它的加密方法和上一版没有变。好，咱们来破掉它。

## 第一步：算出它的注册码！

```
=====================================================================

:004D29F1 8D4000                 lea eax, dword ptr [eax+00]
:004D29F4 55                     push ebp
:004D29F5 8BEC                   mov ebp, esp
:004D29F7 83C4BC                 add esp, FFFFFFBC
:004D29FA 53                     push ebx
:004D29FB 56                     push esi
:004D29FC 33D2                   xor edx, edx
:004D29FE 8955BC                 mov dword ptr [ebp-44], edx
:004D2A01 8955E8                 mov dword ptr [ebp-18], edx
:004D2A04 8955E4                 mov dword ptr [ebp-1C], edx
:004D2A07 8945EC                 mov dword ptr [ebp-14], eax
:004D2A0A 33C0                   xor eax, eax
:004D2A0C 55                     push ebp
:004D2A0D 68812C4D00             push 004D2C81
:004D2A12 64FF30                 push dword ptr fs:[eax]
:004D2A15 648920                 mov dword ptr fs:[eax], esp
:004D2A18 8D45C3                 lea eax, dword ptr [ebp-3D]
:004D2A1B B165                   mov cl, 65
:004D2A1D BA21000000             mov edx, 00000021
:004D2A22 E86500F3FF             call 00402A8C
:004D2A27 33C0                   xor eax, eax
:004D2A29 8945F8                 mov dword ptr [ebp-08], eax
:004D2A2C 33C0                   xor eax, eax
:004D2A2E 8945F4                 mov dword ptr [ebp-0C], eax
:004D2A31 8D45C3                 lea eax, dword ptr [ebp-3D]
:004D2A34 8B159CF94E00           mov edx, dword ptr [004EF99C]
:004D2A3A E8A14CF3FF             call 004076E0
:004D2A3F 8D45C3                 lea eax, dword ptr [ebp-3D]
:004D2A42 8945FC                 mov dword ptr [ebp-04], eax
:004D2A45 60                     pushad
:004D2A46 8B7DFC                 mov edi, dword ptr [ebp-04]
:004D2A49 B818E41736             mov eax, 3617E418
:004D2A4E 3107                   xor dword ptr [edi], eax
:004D2A50 B82EFC35A9             mov eax, A935FC2E
:004D2A55 314704                 xor dword ptr [edi+04], eax
:004D2A58 B8B972D857             mov eax, 57D872B9
:004D2A5D 314708                 xor dword ptr [edi+08], eax
:004D2A60 B837B43D49             mov eax, 493DB437
:004D2A65 31470C                 xor dword ptr [edi+0C], eax
:004D2A68 8B07                   mov eax, dword ptr [edi]
:004D2A6A 334704                 xor eax, dword ptr [edi+04]
:004D2A6D 8B5F08                 mov ebx, dword ptr [edi+08]
:004D2A70 335F0C                 xor ebx, dword ptr [edi+0C]
:004D2A73 8945F8                 mov dword ptr [ebp-08], eax
:004D2A76 895DF4                 mov dword ptr [ebp-0C], ebx
:004D2A79 61                     popad
:004D2A7A A1A0F94E00             mov eax, dword ptr [004EF9A0]
:004D2A7F E85C11F3FF             call 00403BE0
:004D2A84 83F810                 cmp eax, 00000010  ; 比较注册码是否是 16 位
:004D2A87 0F8CD1010000           jl 004D2C5E
:004D2A8D 8D45E8                 lea eax, dword ptr [ebp-18]
:004D2A90 50                     push eax
:004D2A91 B908000000             mov ecx, 00000008
:004D2A96 BA01000000             mov edx, 00000001
:004D2A9B A1A0F94E00             mov eax, dword ptr [004EF9A0]
:004D2AA0 E83F13F3FF             call 00403DE4
:004D2AA5 8D45E4                 lea eax, dword ptr [ebp-1C]
:004D2AA8 50                     push eax
:004D2AA9 B908000000             mov ecx, 00000008
:004D2AAE BA09000000             mov edx, 00000009
:004D2AB3 A1A0F94E00             mov eax, dword ptr [004EF9A0]
:004D2AB8 E82713F3FF             call 00403DE4
:004D2ABD 8D4DBC                 lea ecx, dword ptr [ebp-44]
:004D2AC0 BA08000000             mov edx, 00000008
:004D2AC5 8B45F8                 mov eax, dword ptr [ebp-08]
:004D2AC8 E8DB45F3FF             call 004070A8
:004D2ACD 8B55BC                 mov edx, dword ptr [ebp-44]
:004D2AD0 8B45E8                 mov eax, dword ptr [ebp-18] ; 前 8 位的真假注册码
:004D2AD3 E81812F3FF             call 00403CF0  ; 比较前八位注册码
:004D2AD8 0F8560010000           jne 004D2C3E  ; 跳就 OVER
:004D2ADE 8D4DBC                 lea ecx, dword ptr [ebp-44]
:004D2AE1 BA08000000             mov edx, 00000008
:004D2AE6 8B45F4                 mov eax, dword ptr [ebp-0C]
:004D2AE9 E8BA45F3FF             call 004070A8
:004D2AEE 8B55BC                 mov edx, dword ptr [ebp-44]
:004D2AF1 8B45E4                 mov eax, dword ptr [ebp-1C] ; 后 8 位的真假注册码
:004D2AF4 E8F711F3FF             call 00403CF0  ; 比较后八位注册码
:004D2AF9 0F853F010000           jne 004D2C3E  ; 跳就 OVER
:004D2AFF B88CF94E00             mov eax, 004EF98C
:004D2B04 8B159CF94E00           mov edx, dword ptr [004EF99C]
:004D2B0A E8A90EF3FF             call 004039B8
:004D2B0F B890F94E00             mov eax, 004EF990
:004D2B14 8B4DE4                 mov ecx, dword ptr [ebp-1C]
:004D2B17 8B55E8                 mov edx, dword ptr [ebp-18]
:004D2B1A E80D11F3FF             call 00403C2C
:004D2B1F B201                   mov dl, 01

* Possible StringData Ref from Code Obj ->"`A 淍 LA 訞孉 MmrSra 媡@

                                                                
:004D2B21 A1D0C94000             mov eax, dword ptr [0040C9D0]
:004D2B26 E86502F3FF             call 00402D90
:004D2B2B 8BD8                   mov ebx, eax
:004D2B2D 8BC3                   mov eax, ebx
:004D2B2F E860D4F3FF             call 0040FF94
:004D2B34 BA00020000             mov edx, 00000200
:004D2B39 8BC3                   mov eax, ebx
:004D2B3B 8B08                   mov ecx, dword ptr [eax]
:004D2B3D FF11                   call dword ptr [ecx]
:004D2B3F 33C9                   xor ecx, ecx
:004D2B41 33D2                   xor edx, edx
:004D2B43 8BC3                   mov eax, ebx
:004D2B45 8B30                   mov esi, dword ptr [eax]
:004D2B47 FF560C                 call [esi+0C]
:004D2B4A BE08000000             mov esi, 00000008
:004D2B4F 8D55C3                 lea edx, dword ptr [ebp-3D]
:004D2B52 B920000000             mov ecx, 00000020
:004D2B57 8BC3                   mov eax, ebx
:004D2B59 E88ED0F3FF             call 0040FBEC
:004D2B5E 8D55F8                 lea edx, dword ptr [ebp-08]
:004D2B61 B904000000             mov ecx, 00000004
:004D2B66 8BC3                   mov eax, ebx
:004D2B68 E87FD0F3FF             call 0040FBEC
:004D2B6D 8D55F4                 lea edx, dword ptr [ebp-0C]
:004D2B70 B904000000             mov ecx, 00000004
====================================================================
```

我的注册码是 `dalao/9B2B793D1EE5C68E`

## 第二步：去掉时间限制！

这个软件很奇怪！当你输入正确的注册码后将在 PE EXPLORER 的目录中生成 `pexdata.rdat` 文件，但是还是有时间限制！我猜这个软件的注册有两方面：一个是注册码，一个是网上注册，然后把注册标志写到某个地方！才能完全注册去掉时间限制！不过不要紧，下面中只要改半个字节即可去掉时间限制！嘿嘿!

```
===============================================================================

* Possible StringData Ref from Code Obj ->"12345678FEDCBA98"

                                                                
:004D33A0 8B15A8F94E00           mov edx, dword ptr [004EF9A8]
:004D33A6 E80D06F3FF             call 004039B8
:004D33AB 8B45F0                 mov eax, dword ptr [ebp-10]
:004D33AE 80782401               cmp byte ptr [eax+24], 01
```



:004D33B2 `7508` jne `004D33BC`

:004D33B4 `8B45F0` mov eax, dword ptr `[ebp-10]`

:004D33B7 `E838F6FFFF` call `004D29F4`

:004D33BC `E8AFDAFFFF` call `004D0E70`

:004D33C1 `8B45F0` mov eax, dword ptr `[ebp-10]` ====》注意这个！

:004D33CB `0F85DA070000` jne `004D3BAB` =======〉不调就是试用版

:004D33D1 `8B45F0` mov eax, dword ptr `[ebp-10]`

:004D33D4 `8B9834020000` mov ebx, dword ptr `[eax+00000234]`

:004D33DA `83C305` add ebx, `00000005`

`===============================================================================`

一般改法只要改掉这句 `004D33CB 0F85DA070000 jne 004D3BAB`

把（`0F85DA070000` 改成 `0F84DA070000`）即可！

现在咱们要提升一个档次！我讲讲高级的改法！嘿嘿！

大家注意`[eax+00000254]`这个内存地址！找出和这个注册标志相关的程序！

`:004D33C4 80B85402000000 cmp byte ptr [eax+00000254], 00`

用 `BPM` 下断点！以下就是核心程序

:004D3169 `89836C020000` mov dword ptr `[ebx+0000026C]`, eax

:004D316F `8B8364020000` mov eax, dword ptr `[ebx+00000264]`

:004D3175 `3B45F0` cmp eax, dword ptr `[ebp-10]`

:004D3178 `751A` jne `004D3194` ===》跳就是注册版

:004D317A `8B8368020000` mov eax, dword ptr `[ebx+00000268]`

:004D3180 `3B45EC` cmp eax, dword ptr `[ebp-14]`

:004D3183 `750F` jne `004D3194` ===》跳就是注册版

:004D3185 `8B836C020000` mov eax, dword ptr `[ebx+0000026C]`

:004D318B `3B45E8` cmp eax, dword ptr `[ebp-18]`

:004D318E `7504` jne `004D3194` ===》跳就是注册版

:004D3190 `33C0` xor eax, eax

:004D3192 `EB02` jmp `004D3196` ===〉到这你就 OVER

:004D3194 `B001` mov al, `01`

:004D3196 `8883540000` mov byte ptr `[ebx+00000254]`, al

:004D319C `8B45F4` mov eax, dword ptr `[ebp-0C]`

:004D319F `50` push eax

:004D31A0 `B85C0D4D00` mov eax, `004D0D5C`

:004D31A5 `668B0D9C624F00` mov cx, word ptr `[004F629C]`

:004D31AC `8B150CFA4E00` mov edx, dword ptr `[004EFA0C]`

:004D31B2 `E8C1DCFFFF` call `004D0E78`

:004D31B7 `33C0` xor eax, eax

:004D31B9 `5A` pop edx

:004D31BA `59` pop ecx

:004D31BB `59` pop ecx

大家现在明白了吧嘿嘿！高级改法吧 `004D3178` 地址的（`750F` 改成 `EB0F`）

收工有啥错误的地方请大家多提批评！

大老

`http://dalao2001.yeah.net`

2001.6.29 `4dd466d0e1e0cc123afbe504da2462ca`
