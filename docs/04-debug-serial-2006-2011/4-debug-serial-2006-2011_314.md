
# WinASO Registry Optimizer V3.0.9 算法分析

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-281.htm`](https://www.pediy.com/kssd/pediy09/pediy09-281.htm)

> 【破文标题】WinASO Registry Optimizer V3.0.9 算法分析

> 【破解日期】96.09.20

> 【破文作者】\[咖啡&chuan\]

> 【原版下载】`http://www.skycn.com/soft/27630.html`

> 【软件介绍】WinASO Registry Optimizer 是一个 Windows 优化工具和高级注册表清理工具，它允许您以简单的鼠标单击来安全的清理及修复注册表故障。通过修复陈旧信息及调整 Windows 注册表参数，它对系统速度的提升是值得令人注意的。WinASO Registry Optimizer 被很好的设计为修复普遍的问题，例如对 Internet Explorer 页面的非法修改。我们已对此工具进行了实际测试来确保您的系统安全。我们没有收到过任何对系统稳定性的抱怨。

-------------------------------------------------------------------------

查 `Borland Delphi 6.0` 以超字符串搜提示，祭出函 `BP bp MessageBoxA`，下後看堆窗看到 `0056A7C2` /`CALL`

------------------------------------------------------------------------------------------

```
0012F7BC   0056A7C2   /CALL 到 MessageBoxA 自 RegOpt.0056A7BD
0012F7C0   005F06FE   |hOwner = 005F06FE ('Register Information',class='TfrmRegister',parent=00200466)
0012F7C4   015ED010   |Text = "Sorry, that is an invalid license key. Please ensure you have entered the license key exactly as provided."
0012F7C8   0164EDB8   |Title = "Information"
0012F7CC   00000040   \Style = MB_OK|MB_ICONASTERISK|MB_APPLMODAL
```
----------------------------------------------------------------------------------------

破解分析如下：

```
0056A3AB  .  51           push ecx ---下
0056A3AC  .  53           push ebx
0056A3AD  .  56           push esi
0056A3AE  .  57           push edi
0056A3AF  .  8BD8         mov ebx,eax
0056A3B1  .  33C0         xor eax,eax
0056A3B3  .  55           push ebp
0056A3B4  .  68 F4A95600  push RegOpt.0056A9F4
0056A3B9  .  64:FF30      push dword ptr fs:[eax]
0056A3BC  .  64:8920      mov dword ptr fs:[eax],esp
0056A3BF  .  8D45 FC      lea eax,dword ptr ss:[ebp-4]
0056A3C2  .  8B15 84055A00 mov edx,dword ptr ds:[5A0584] ; RegOpt.005A55E0
0056A3C8  .  8B92 70080000 mov edx,dword ptr ds:[edx+870]
0056A3CE  .  E8 5DAEE9FF  call RegOpt.00405230
0056A3D3  .  8D55 F4      lea edx,dword ptr ss:[ebp-C]
0056A3D6  .  8B83 A4030000 mov eax,dword ptr ds:[ebx+3A4]
0056A3DC  .  E8 3BC9EEFF  call RegOpt.00456D1C
0056A3E1  .  8B45 F4      mov eax,dword ptr ss:[ebp-C]
0056A3E4  .  E8 9B45F6FF  call RegOpt.004CE984
0056A3E9  .  84C0         test al,al
0056A3EB  .  75 2E        jnz short RegOpt.0056A41B
0056A3ED  .  6A 40        push 40
0056A3EF  .  A1 84055A00  mov eax,dword ptr ds:[5A0584]
0056A3F4  .  8B80 6C080000 mov eax,dword ptr ds:[eax+86C]
0056A3FA  .  E8 5DB2E9FF  call RegOpt.0040565C
0056A3FF  .  50           push eax
0056A400  .  8B45 FC      mov eax,dword ptr ss:[ebp-4]
0056A403  .  E8 54B2E9FF  call RegOpt.0040565C
0056A408  .  50           push eax
0056A409  .  8BC3         mov eax,ebx
0056A40B  .  E8 0C44EFFF  call RegOpt.0045E81C
0056A410  .  50           push eax ; |hOwner
0056A411  .  E8 26DFE9FF  call <jmp.&user32.MessageBoxA> ; \MessageBoxA
0056A416  .  E9 6D050000  jmp RegOpt.0056A988
0056A41B  |> 8D55 F0      lea edx,dword ptr ss:[ebp-10]
0056A41E  .  8B83 A8030000 mov eax,dword ptr ds:[ebx+3A8]
0056A424  .  E8 F3C8EEFF  call RegOpt.00456D1C
0056A429  .  8B45 F0      mov eax,dword ptr ss:[ebp-10]
0056A42C  .  E8 5345F6FF  call RegOpt.004CE984
0056A431  .  84C0         test al,al
0056A433  .  75 2E        jnz short RegOpt.0056A463
0056A435  .  6A 40        push 40
0056A437  .  A1 84055A00  mov eax,dword ptr ds:[5A0584]
0056A43C  .  8B80 6C080000 mov eax,dword ptr ds:[eax+86C]
0056A442  .  E8 15B2E9FF  call RegOpt.0040565C
0056A447  .  50           push eax
0056A448  .  8B45 FC      mov eax,dword ptr ss:[ebp-4]
0056A44B  .  E8 0CB2E9FF  call RegOpt.0040565C
0056A450  .  50           push eax
0056A451  .  8BC3         mov eax,ebx
0056A453  .  E8 C443EFFF  call RegOpt.0045E81C
0056A458  .  50           push eax ; |hOwner
0056A459  .  E8 DEDEE9FF  call <jmp.&user32.MessageBoxA> ; \MessageBoxA
0056A45E  .  E9 25050000  jmp RegOpt.0056A988
0056A463  |> 8D55 EC      lea edx,dword ptr ss:[ebp-14]
0056A466  .  8B83 AC030000 mov eax,dword ptr ds:[ebx+3AC]
0056A46C  .  E8 ABC8EEFF  call RegOpt.00456D1C
0056A471  .  8B45 EC      mov eax,dword ptr ss:[ebp-14]
0056A474  .  E8 0B45F6FF  call RegOpt.004CE984
0056A479  .  84C0         test al,al
0056A47B  .  75 2E        jnz short RegOpt.0056A4AB
0056A47D  .  6A 40        push 40
0056A47F  .  A1 84055A00  mov eax,dword ptr ds:[5A0584]
0056A484  .  8B80 6C080000 mov eax,dword ptr ds:[eax+86C]
0056A48A  .  E8 CDB1E9FF  call RegOpt.0040565C
0056A48F  .  50           push eax
0056A490  .  8B45 FC      mov eax,dword ptr ss:[ebp-4]
0056A493  .  E8 C4B1E9FF  call RegOpt.0040565C
0056A498  .  50           push eax
0056A499  .  8BC3         mov eax,ebx
0056A49B  .  E8 7C43EFFF  call RegOpt.0045E81C
0056A4A0  .  50           push eax ; |hOwner
0056A4A1  .  E8 96DEE9FF  call <jmp.&user32.MessageBoxA> ; \MessageBoxA
0056A4A6  .  E9 DD040000  jmp RegOpt.0056A988
0056A4AB  |> 8D55 E8      lea edx,dword ptr ss:[ebp-18]
0056A4AE  .  8B83 B0030000 mov eax,dword ptr ds:[ebx+3B0]
0056A4B4  .  E8 63C8EEFF  call RegOpt.00456D1C
0056A4B9  .  8B45 E8      mov eax,dword ptr ss:[ebp-18]
0056A4BC  .  E8 C344F6FF  call RegOpt.004CE984
0056A4C1  .  84C0         test al,al
0056A4C3  .  75 2E        jnz short RegOpt.0056A4F3
0056A4C5  .  6A 40        push 40
0056A4C7  .  A1 84055A00  mov eax,dword ptr ds:[5A0584]
0056A4CC  .  8B80 6C080000 mov eax,dword ptr ds:[eax+86C]
0056A4D2  .  E8 85B1E9FF  call RegOpt.0040565C
0056A4D7  .  50           push eax
0056A4D8  .  8B45 FC      mov eax,dword ptr ss:[ebp-4]
0056A4DB  .  E8 7CB1E9FF  call RegOpt.0040565C
0056A4E0  .  50           push eax
0056A4E1  .  8BC3         mov eax,ebx
0056A4E3  .  E8 3443EFFF  call RegOpt.0045E81C
0056A4E8  .  50           push eax ; |hOwner
0056A4E9  .  E8 4EDEE9FF  call <jmp.&user32.MessageBoxA> ; \MessageBoxA
0056A4EE  .  E9 95040000  jmp RegOpt.0056A988
0056A4F3  |> 8D55 E4      lea edx,dword ptr ss:[ebp-1C]
0056A4F6  .  8B83 B4030000 mov eax,dword ptr ds:[ebx+3B4]
0056A4FC  .  E8 1BC8EEFF  call RegOpt.00456D1C
0056A501  .  8B45 E4      mov eax,dword ptr ss:[ebp-1C]
0056A504  .  E8 7B44F6FF  call RegOpt.004CE984
0056A509  .  84C0         test al,al
0056A50B  .  75 2E        jnz short RegOpt.0056A53B
0056A50D  .  6A 40        push 40
0056A50F  .  A1 84055A00  mov eax,dword ptr ds:[5A0584]
0056A514  .  8B80 6C080000 mov eax,dword ptr ds:[eax+86C]
0056A51A  .  E8 3DB1E9FF  call RegOpt.0040565C
0056A51F  .  50           push eax
0056A520  .  8B45 FC      mov eax,dword ptr ss:[ebp-4]
0056A523  .  E8 34B1E9FF  call RegOpt.0040565C
0056A528  .  50           push eax
0056A529  .  8BC3         mov eax,ebx
0056A52B  .  E8 EC42EFFF  call RegOpt.0045E81C
0056A530  .  50           push eax ; |hOwner
0056A531  .  E8 06DEE9FF  call <jmp.&user32.MessageBoxA> ; \MessageBoxA
0056A536  .  E9 4D040000  jmp RegOpt.0056A988
0056A53B  |> 8D55 E0      lea edx,dword ptr ss:[ebp-20]
0056A53E  .  8B83 A4030000 mov eax,dword ptr ds:[ebx+3A4]
```



```
0056A544  |.  E8 D3C7EEFF   call RegOpt.00456D1C
0056A549  |.  8B45 E0       mov eax,dword ptr ss:[ebp-20]    ;  第 1 格
0056A54C  |.  8945 DC       mov dword ptr ss:[ebp-24],eax
0056A54F  |.  8B45 DC       mov eax,dword ptr ss:[ebp-24]
0056A552  |.  85C0          test eax,eax
0056A554  |.  74 05         je short RegOpt.0056A55B
0056A556  |.  83E8 04       sub eax,4
0056A559  |.  8B00          mov eax,dword ptr ds:[eax]
0056A55B  |>  83F8 04       cmp eax,4
0056A55E  |.  74 2E         je short RegOpt.0056A58E
0056A560  |.  6A 40         push 40
0056A562  |.  A1 84055A00   mov eax,dword ptr ds:[5A0584]
0056A567  |.  8B80 6C080000 mov eax,dword ptr ds:[eax+86C]
0056A56D  |.  E8 EAB0E9FF   call RegOpt.0040565C
0056A572  |.  50            push eax
0056A573  |.  8B45 FC       mov eax,dword ptr ss:[ebp-4]
0056A576  |.  E8 E1B0E9FF   call RegOpt.0040565C
0056A57B  |.  50            push eax
0056A57C  |.  8BC3          mov eax,ebx
0056A57E  |.  E8 9942EFFF   call RegOpt.0045E81C
0056A583  |.  50            push eax                         ; |hOwner
0056A584  |.  E8 B3DDE9FF   call <jmp.&user32.MessageBoxA>   ; \MessageBoxA
0056A589  |.  E9 FA030000   jmp RegOpt.0056A988
0056A58E  |>  8D55 D8       lea edx,dword ptr ss:[ebp-28]
0056A591  |.  8B83 A8030000 mov eax,dword ptr ds:[ebx+3A8]
0056A597  |.  E8 80C7EEFF   call RegOpt.00456D1C
0056A59C  |.  8B45 D8       mov eax,dword ptr ss:[ebp-28]
0056A59F  |.  8945 DC       mov dword ptr ss:[ebp-24],eax
0056A5A2  |.  8B45 DC       mov eax,dword ptr ss:[ebp-24]
0056A5A5  |.  85C0          test eax,eax
0056A5A7  |.  74 05         je short RegOpt.0056A5AE
0056A5A9  |.  83E8 04       sub eax,4
0056A5AC  |.  8B00          mov eax,dword ptr ds:[eax]
0056A5AE  |>  83F8 04       cmp eax,4
0056A5B1  |.  74 2E         je short RegOpt.0056A5E1
0056A5B3  |.  6A 40         push 40
0056A5B5  |.  A1 84055A00   mov eax,dword ptr ds:[5A0584]
0056A5BA  |.  8B80 6C080000 mov eax,dword ptr ds:[eax+86C]
0056A5C0  |.  E8 97B0E9FF   call RegOpt.0040565C
0056A5C5  |.  50            push eax
0056A5C6  |.  8B45 FC       mov eax,dword ptr ss:[ebp-4]
0056A5C9  |.  E8 8EB0E9FF   call RegOpt.0040565C
0056A5CE  |.  50            push eax
0056A5CF  |.  8BC3          mov eax,ebx
0056A5D1  |.  E8 4642EFFF   call RegOpt.0045E81C
0056A5D6  |.  50            push eax                         ; |hOwner
0056A5D7  |.  E8 60DDE9FF   call <jmp.&user32.MessageBoxA>   ; \MessageBoxA
0056A5DC  |.  E9 A7030000   jmp RegOpt.0056A988
0056A5E1  |>  8D55 D4       lea edx,dword ptr ss:[ebp-2C]
0056A5E4  |.  8B83 AC030000 mov eax,dword ptr ds:[ebx+3AC]
0056A5EA  |.  E8 2DC7EEFF   call RegOpt.00456D1C
0056A5EF  |.  8B45 D4       mov eax,dword ptr ss:[ebp-2C]
0056A5F2  |.  8945 DC       mov dword ptr ss:[ebp-24],eax
0056A5F5  |.  8B45 DC       mov eax,dword ptr ss:[ebp-24]
0056A5F8  |.  85C0          test eax,eax
0056A5FA  |.  74 05         je short RegOpt.0056A601
0056A5FC  |.  83E8 04       sub eax,4
0056A5FF  |.  8B00          mov eax,dword ptr ds:[eax]
0056A601  |>  83F8 04       cmp eax,4
0056A604  |.  74 2E         je short RegOpt.0056A634
0056A606  |.  6A 40         push 40
0056A608  |.  A1 84055A00   mov eax,dword ptr ds:[5A0584]
0056A60D  |.  8B80 6C080000 mov eax,dword ptr ds:[eax+86C]
0056A613  |.  E8 44B0E9FF   call RegOpt.0040565C
0056A618  |.  50            push eax
0056A619  |.  8B45 FC       mov eax,dword ptr ss:[ebp-4]
0056A61C  |.  E8 3BB0E9FF   call RegOpt.0040565C
0056A621  |.  50            push eax
0056A622  |.  8BC3          mov eax,ebx
0056A624  |.  E8 F341EFFF   call RegOpt.0045E81C
0056A629  |.  50            push eax                         ; |hOwner
0056A62A  |.  E8 0DDDE9FF   call <jmp.&user32.MessageBoxA>   ; \MessageBoxA
0056A62F  |.  E9 54030000   jmp RegOpt.0056A988
0056A634  |>  8D55 D0       lea edx,dword ptr ss:[ebp-30]
0056A637  |.  8B83 B0030000 mov eax,dword ptr ds:[ebx+3B0]
0056A63D  |.  E8 DAC6EEFF   call RegOpt.00456D1C
0056A642  |.  8B45 D0       mov eax,dword ptr ss:[ebp-30]
0056A645  |.  8945 DC       mov dword ptr ss:[ebp-24],eax
0056A648  |.  8B45 DC       mov eax,dword ptr ss:[ebp-24]
0056A64B  |.  85C0          test eax,eax
0056A64D  |.  74 05         je short RegOpt.0056A654
0056A64F  |.  83E8 04       sub eax,4
0056A652  |.  8B00          mov eax,dword ptr ds:[eax]
0056A654  |>  83F8 04       cmp eax,4
0056A657  |.  74 2E         je short RegOpt.0056A687
0056A659  |.  6A 40         push 40
0056A65B  |.  A1 84055A00   mov eax,dword ptr ds:[5A0584]
0056A660  |.  8B80 6C080000 mov eax,dword ptr ds:[eax+86C]
0056A666  |.  E8 F1AFE9FF   call RegOpt.0040565C
0056A66B  |.  50            push eax
0056A66C  |.  8B45 FC       mov eax,dword ptr ss:[ebp-4]
0056A66F  |.  E8 E8AFE9FF   call RegOpt.0040565C
0056A674  |.  50            push eax
0056A675  |.  8BC3          mov eax,ebx
0056A677  |.  E8 A041EFFF   call RegOpt.0045E81C
0056A67C  |.  50            push eax                         ; |hOwner
0056A67D  |.  E8 BADCE9FF   call <jmp.&user32.MessageBoxA>   ; \MessageBoxA
0056A682  |.  E9 01030000   jmp RegOpt.0056A988
0056A687  |>  8D55 CC       lea edx,dword ptr ss:[ebp-34]
0056A68A  |.  8B83 B4030000 mov eax,dword ptr ds:[ebx+3B4]
0056A690  |.  E8 87C6EEFF   call RegOpt.00456D1C
0056A695  |.  8B45 CC       mov eax,dword ptr ss:[ebp-34]
0056A698  |.  8945 DC       mov dword ptr ss:[ebp-24],eax
0056A69B  |.  8B45 DC       mov eax,dword ptr ss:[ebp-24]
0056A69E  |.  85C0          test eax,eax
0056A6A0  |.  74 05         je short RegOpt.0056A6A7
0056A6A2  |.  83E8 04       sub eax,4
0056A6A5  |.  8B00          mov eax,dword ptr ds:[eax]
0056A6A7  |>  83F8 04       cmp eax,4
0056A6AA  |.  74 2E         je short RegOpt.0056A6DA
0056A6AC  |.  6A 40         push 40
0056A6AE  |.  A1 84055A00   mov eax,dword ptr ds:[5A0584]
0056A6B3  |.  8B80 6C080000 mov eax,dword ptr ds:[eax+86C]
0056A6B9  |.  E8 9EAFE9FF   call RegOpt.0040565C
0056A6BE  |.  50            push eax
0056A6BF  |.  8B45 FC       mov eax,dword ptr ss:[ebp-4]
0056A6C2  |.  E8 95AFE9FF   call RegOpt.0040565C
0056A6C7  |.  50            push eax
0056A6C8  |.  8BC3          mov eax,ebx
0056A6CA  |.  E8 4D41EFFF   call RegOpt.0045E81C
0056A6CF  |.  50            push eax                         ; |hOwner
0056A6D0  |.  E8 67DCE9FF   call <jmp.&user32.MessageBoxA>   ; \MessageBoxA
0056A6D5  |.  E9 AE020000   jmp RegOpt.0056A988
0056A6DA  |>  8D55 C8       lea edx,dword ptr ss:[ebp-38]
0056A6DD  |.  8B83 A4030000 mov eax,dword ptr ds:[ebx+3A4]
0056A6E3  |.  E8 34C6EEFF   call RegOpt.00456D1C
0056A6E8  |.  8B45 C8       mov eax,dword ptr ss:[ebp-38]
0056A6EB  |.  E8 C4F7E9FF   call RegOpt.00409EB4
0056A6F0  |.  8BF0          mov esi,eax                      ;  457-s1
0056A6F2  |.  8D55 C4       lea edx,dword ptr ss:[ebp-3C]
0056A6F5  |.  8B83 A8030000 mov eax,dword ptr ds:[ebx+3A8]
0056A6FB  |.  E8 1CC6EEFF   call RegOpt.00456D1C
0056A700  |.  8B45 C4       mov eax,dword ptr ss:[ebp-3C]
0056A703  |.  E8 ACF7E9FF   call RegOpt.00409EB4
0056A708  |.  8BF8          mov edi,eax                      ;  8AE-s2
0056A70A  |.  8D55 C0       lea edx,dword ptr ss:[ebp-40]
0056A70D  |.  8B83 AC030000 mov eax,dword ptr ds:[ebx+3AC]
0056A713  |.  E8 04C6EEFF   call RegOpt.00456D1C
```



```
0056A718  |.  8B45 C0        mov eax,dword ptr ss:[ebp-40]
0056A71B  |.  E8 94F7E9FF   call RegOpt.00409EB4
0056A720  |.  8945 F8       mov dword ptr ss:[ebp-8],eax     ;  D05-s3
0056A723  |.  0FAFF7        imul esi,edi                      ;  s2*s1
0056A726  |.  81EE 2B060000 sub esi,62B
0056A72C  |.  81FE 10270000 cmp esi,2710                      ;  S2*S1-62B>10000---A1
0056A732  |.  7D 2E         jge short RegOpt.0056A762
0056A734  |.  6A 40         push 40
0056A736  |.  A1 84055A00   mov eax,dword ptr ds:[5A0584]
0056A73B  |.  8B80 6C080000 mov eax,dword ptr ds:[eax+86C]
0056A741  |.  E8 16AFE9FF   call RegOpt.0040565C
0056A746  |.  50            push eax
0056A747  |.  8B45 FC       mov eax,dword ptr ss:[ebp-4]
0056A74A  |.  E8 0DAFE9FF   call RegOpt.0040565C
0056A74F  |.  50            push eax
0056A750  |.  8BC3          mov eax,ebx
0056A752  |.  E8 C540EFFF   call RegOpt.0045E81C
0056A757  |.  50            push eax                          ; |hOwner
0056A758  |.  E8 DFDBE9FF   call <jmp.&user32.MessageBoxA>   ; \MessageBoxA
0056A75D  |.  E9 26020000   jmp RegOpt.0056A988

0056A762  |>  8D55 B8       lea edx,dword ptr ss:[ebp-48]
0056A765  |.  8BC6          mov eax,esi
0056A767  |.  E8 0CF6E9FF   call RegOpt.00409D78
0056A76C  |.  8B45 B8       mov eax,dword ptr ss:[ebp-48]
0056A76F  |.  8D4D BC       lea ecx,dword ptr ss:[ebp-44]     ;  取 A1 之前 4 位
0056A772  |.  BA 04000000   mov edx,4
0056A777  |.  E8 14DDEDFF   call RegOpt.00448490
0056A77C  |.  8B45 BC       mov eax,dword ptr ss:[ebp-44]     ;  取 A1 第 4.5.3.6 位=T1
0056A77F  |.  50            push eax
0056A780  |.  8D55 B4       lea edx,dword ptr ss:[ebp-4C]
0056A783  |.  8B83 B0030000 mov eax,dword ptr ds:[ebx+3B0]
0056A789  |.  E8 8EC5EEFF   call RegOpt.00456D1C
0056A78E  |.  8B55 B4       mov edx,dword ptr ss:[ebp-4C]     ;  比第 4 格是否 T1 的值
0056A791  |.  58            pop eax
0056A792  |.  E8 11AEE9FF   call RegOpt.004055A8
0056A797  |.  74 2E         je short RegOpt.0056A7C7
0056A799  |.  6A 40         push 40
0056A79B  |.  A1 84055A00   mov eax,dword ptr ds:[5A0584]
0056A7A0  |.  8B80 6C080000 mov eax,dword ptr ds:[eax+86C]
0056A7A6  |.  E8 B1AEE9FF   call RegOpt.0040565C
0056A7AB  |.  50            push eax
0056A7AC  |.  8B45 FC       mov eax,dword ptr ss:[ebp-4]
0056A7AF  |.  E8 A8AEE9FF   call RegOpt.0040565C
0056A7B4  |.  50            push eax
0056A7B5  |.  8BC3          mov eax,ebx
0056A7B7  |.  E8 6040EFFF   call RegOpt.0045E81C
0056A7BC  |.  50            push eax                          ; |hOwner
0056A7BD  |.  E8 7ADBE9FF   call <jmp.&user32.MessageBoxA>   ; \MessageBoxA
0056A7C2  |.  E9 C1010000   jmp RegOpt.0056A988

0056A7C7  |>  8B75 F8       mov esi,dword ptr ss:[ebp-8]
0056A7CA  |.  81C6 01020000 add esi,201                       ;  s3+201..s4
0056A7D0  |.  0FAFF7        imul esi,edi                      ;  s4*s2
0056A7D3  |.  81EE F50D0000 sub esi,0DF5                      ;  (s3+201)+(s4*s2)-DF5..A2
0056A7D9  |.  8D55 AC       lea edx,dword ptr ss:[ebp-54]
0056A7DC  |.  8BC6          mov eax,esi
0056A7DE  |.  E8 95F5E9FF   call RegOpt.00409D78              ;  要入才能得到 T2 ***要注意
0056A7E3  |.  8B45 AC       mov eax,dword ptr ss:[ebp-54]
0056A7E6  |.  8D4D B0       lea ecx,dword ptr ss:[ebp-50]
0056A7E9  |.  BA 04000000   mov edx,4
0056A7EE  |.  E8 9DDCEDFF   call RegOpt.00448490
0056A7F3  |.  8B45 B0       mov eax,dword ptr ss:[ebp-50]     ;  取 T2 第後 4 位
0056A7F6  |.  50            push eax
0056A7F7  |.  8D55 A8       lea edx,dword ptr ss:[ebp-58]
0056A7FA  |.  8B83 B4030000 mov eax,dword ptr ds:[ebx+3B4]
0056A800  |.  E8 17C5EEFF   call RegOpt.00456D1C
0056A805  |.  8B55 A8       mov edx,dword ptr ss:[ebp-58]     ;  比第 5 格是否 T2
0056A808  |.  58            pop eax
0056A809  |.  E8 9AADE9FF   call RegOpt.004055A8
0056A80E  |.  74 2E         je short RegOpt.0056A83E
0056A810  |.  6A 40         push 40
0056A812  |.  A1 84055A00   mov eax,dword ptr ds:[5A0584]
0056A817  |.  8B80 6C080000 mov eax,dword ptr ds:[eax+86C]
0056A81D  |.  E8 3AAEE9FF   call RegOpt.0040565C
0056A822  |.  50            push eax
0056A823  |.  8B45 FC       mov eax,dword ptr ss:[ebp-4]
0056A826  |.  E8 31AEE9FF   call RegOpt.0040565C
0056A82B  |.  50            push eax
0056A82C  |.  8BC3          mov eax,ebx
0056A82E  |.  E8 E93FEFFF   call RegOpt.0045E81C
0056A833  |.  50            push eax                          ; |hOwner
0056A834  |.  E8 03DBE9FF   call <jmp.&user32.MessageBoxA>   ; \MessageBoxA

--------------------------------------------------------------------------------
```

破解解：

一、件有五格入框，只算第 4 格第 5 格。前 3 格任意 `s1` `s2` `s3`

二、分入的值 16 制，如入 111116 制=457

三、第 4 格方式 `S2*S1-62B>10000--A1`;必大於 10000

再取 `A1` 的 4.5.6.3 位，`T1` 得到第 4 格的

四、第 5 格方式 `(s3+201)+(s4*s2)-DF5=0822531F..A2`

再取 `A2` 的後 4 位，`T2` 得到第 5 格的

PS. 小弟是菜在破解程中遇到一疑，就是 `T2` 的部份(`S3` 入的是 333316 制 `D05`)

是 `0822531F` 可是怎是 `8542239` 不知道 `8542239` 是如何而；後我有追 `0056A7DE` `call` 就是不明白。

此小子贼野已表破解文章，小弟

希望先予以指

`4dd466d0e1e0cc123afbe504da2462ca`
