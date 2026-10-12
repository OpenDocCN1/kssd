
```
:0042E285 51                                                                 push ecx
:0042E286 8D55D0                                                            lea edx, dword ptr [ebp-30]
:0042E289 6A00                                                              push 00000000
:0042E28B 52                                                                push edx

* Reference To: MSVBVM60.rtcMsgBox, Ord:0253h
|
:0042E28C FF1594104000                                                    Call dword ptr [00401094]
                                                                         ==== > 呵呵，胜利女神！

:0042E292 8D45A0                                                            lea eax, dword ptr [ebp-60]
:0042E295 8D4DB0                                                            lea ecx, dword ptr [ebp-50]
:0042E298 50                                                                push eax
:0042E299 8D55C0                                                            lea edx, dword ptr [ebp-40]
:0042E29C 51                                                                push ecx
:0042E29D 8D45D0                                                            lea eax, dword ptr [ebp-30]
:0042E2A0 52                                                                push edx
:0042E2A1 50                                                                push eax
:0042E2A2 6A04                                                              push 00000004

* Reference To: MSVBVM60.__vbaFreeVarList, Ord:0000h
|
:0042E2A4 FF1538104000                                                    Call dword ptr [00401038]

:0042E2AA 8B0E                                                              mov ecx, dword ptr [esi]
:0042E2AC 83C414                                                            add esp, 00000014
:0042E2AF 56                                                                push esi
:0042E2B0 FF911C030000                                                    call dword ptr [ecx+0000031C]
:0042E2B6 8D55E4                                                            lea edx, dword ptr [ebp-1C]
:0042E2B9 50                                                                push eax
:0042E2BA 52                                                                push edx
:0042E2BB FFD7                                                              call edi
:0042E2BD 8BF0                                                              mov esi, eax
:0042E2BF 6AFF                                                              push FFFFFFFF
:0042E2C1 56                                                                push esi
:0042E2C2 8B06                                                              mov eax, dword ptr [esi]
:0042E2C4 FF908C000000                                                    call dword ptr [eax+0000008C]
:0042E2CA 85C0                                                              test eax, eax
:0042E2CC DBE2                                                              fclex
:0042E2CE 7D12                                                              jge 0042E2E2
:0042E2D0 688C000000                                                        push 0000008C

* Possible StringData Ref from Code Obj ->"馧?檉??"
|
:0042E2D5 6884004100                                                        push 00410084
:0042E2DA 56                                                                push esi
:0042E2DB 50                                                                push eax

* Reference To: MSVBVM60.__vbaHresultCheckObj, Ord:0000h
|
:0042E2DC FF1568104000                                                    Call dword ptr [00401068]

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:0042E2CE(C)
|
:0042E2E2 8D4DE4                                                            lea ecx, dword ptr [ebp-1C]
:0042E2E5 FFD3                                                              call ebx
:0042E2E7 E986010000                                                        jmp 0042E472
```

…… ……省略…… ……

```
* Reference To: MSVBVM60.rtcMsgBox, Ord:0253h
|
:0042E334 FF1594104000                                                    Call dword ptr [00401094]
                                                                         ==== > BAD BOY！
```

—————————————————————————————————

用`S`大法加`BPM`会来到下面的地方。其实是同目录下的`nslock15vb5.ocx`领空。

…… ……省略…… ……

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:1647225B(C)
|
:1647226E 8D4588                                                            lea eax, dword ptr [ebp-78]
:16472271 50                                                                push eax

* Reference To: MSVBVM50.__vbaStrVarMove, Ord:0000h
|
:16472272 FF152C624716                                                    Call dword ptr [1647622C]
:16472278 8BD0                                                              mov edx, eax
```

☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆

下面的值是根据机器号生成的：（具体算法不清楚 `^O^` `^O^`）

```
00454268  31 00 45 00 36 00 34 00 31 00 38 00 35 00 35 00  1.E.6.4.1.8.5.5.
00454278  30 00 32 00 37 00 36 00 30 00 33 00 31 00 30 00  0.2.7.6.0.3.1.0.
00454288  30 00 37 00 35 00 37                                        0.7.5.7
```

☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆

```
:1647227A 8D4DB0                                                            lea ecx, dword ptr [ebp-50]

* Reference To: MSVBVM50.__vbaStrMove, Ord:0000h
|
:1647227D FF15AC634716                                                    Call dword ptr [164763AC]
:16472283 8D4D88                                                            lea ecx, dword ptr [ebp-78]

* Reference To: MSVBVM50.__vbaFreeVar, Ord:0000h
|
:16472286 FF1524624716                                                    Call dword ptr [16476224]

* Reference To: MSVBVM50.__vbaStrCopy, Ord:0000h
|
:1647228C 8B3D44634716                                                    mov edi, dword ptr [16476344]

* Possible StringData Ref from Code Obj ->"/\\/e7C0$0fT\\/\\/ArE"
|
:16472292 BA1C3B4616                                                        mov edx, 16463B1C
```

☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆

这个值是固定的：

```
004544EC  2F 00 5C 00 2F 00 65 00 37 00 43 00 30 00 24 00  /.\\./.e.7.C.0.$.
004544FC  30 00 66 00 54 00 5C 00 2F 00 5C 00 2F 00 41 00  0.f.T.\\./.\\./.A.
0045450C  72 00 45                                                    r.E
```

☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆☆

```
:16472297 8D4D9C                                                            lea ecx, dword ptr [ebp-64]
:1647229A FFD7                                                              call edi
:1647229C 8D4D9C                                                            lea ecx, dword ptr [ebp-64]
:1647229F 8D55B0                                                            lea edx, dword ptr [ebp-50]
:164722A2 51                                                                push ecx
:164722A3 52                                                                push edx
:164722A4 E857060000                                                        call 16472900
                                                                         ==== > 上面 2 组值异或运算  得到下面的结果

:164722A9 8BD0                                                              mov edx, eax
                                                                         ==== > EDX=1870553471
```

—————————————————————————————————

继续跟踪：`GO GO GO` `^O^` `^O^`

```
:164689C8 FF92F4070000                                                    call dword ptr [edx+000007F4]
:164689CE 8B0E                                                              mov ecx, dword ptr [esi]
:164689D0 8D55DC                                                            lea edx, dword ptr [ebp-24]
:164689D3 52                                                                push edx
:164689D4 56                                                                push esi
:164689D5 FF91F8070000                                                    call dword ptr [ecx+000007F8]
:164689DB 8B45E0                                                            mov eax, dword ptr [ebp-20]
                                                                         ==== > EDX=1870553471

:164689DE 8B4DDC                                                            mov ecx, dword ptr [ebp-24]
                                                                         ==== > ECX=R64DG-RXGMD-G9BM6-V3H9G-2R348
                                                                         ==== > 这个是你的 Windows 的 ProductKey

:164689E1 50                                                                push eax
:164689E2 51                                                                push ecx

* Reference To: MSVBVM50.__vbaStrCat, Ord:0000h
|
:164689E3 FF1550624716                                                    Call dword ptr [16476250]
                                                                         ==== > 把上面 2 组字符连接起来！

* Reference To: MSVBVM50.__vbaStrMove, Ord:0000h
|
:164689E9 8B1DAC634716                                                    mov ebx, dword ptr [164763AC]
:164689EF 8BD0                                                              mov edx, eax
                                                                         ==== > EDX=1870553471R64DG-RXGMD-G9BM6-V3H9G-2R348

:164689F1 8D4DD8                                                            lea ecx, dword ptr [ebp-28]
:164689F4 FFD3                                                              call ebx
:164689F6 50                                                                push eax
:164689F7 E874710000                                                        call 1646FB70
                                                                         ==== > 加密算法 CALL ①！     得出下面的结果！

:164689FC 8BD0                                                              mov edx, eax
                                                                         ==== > EDX=566C71F6575BD4B395FE34B037F4E04B4DD6D78B

:164689FE 8D4DE8                                                            lea ecx, dword ptr [ebp-18]
:16468A01 FFD3                                                              call ebx
:16468A03 8D55D8                                                            lea edx, dword ptr [ebp-28]
:16468A06 8D45DC                                                            lea eax, dword ptr [ebp-24]
:16468A09 52                                                                push edx
:16468A0A 8D4DE0                                                            lea ecx, dword ptr [ebp-20]
:16468A0D 50                                                                push eax
:16468A0E 51                                                                push ecx
:16468A0F 6A03                                                              push 00000003

* Reference To: MSVBVM50.__vbaFreeStrList, Ord:0000h
|
:16468A11 FF154C634716                                                    Call dword ptr [1647634C]
:16468A17 0FBF4646                                                        movsx eax, word ptr [esi+46]
:16468A1B 83C410                                                            add esp, 00000010
```
