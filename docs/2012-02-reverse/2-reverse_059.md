
# 出现错误对话框的杀手锏－－暂停--初学者适用

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-350.htm`](https://www.pediy.com/kssd/pediy07/pediy7-350.htm)

## 理论

出现一个对话框必然是程序调用一个子程序来完成，在汇编状态下也就是必须要调用一个 `CALL` 来完成程序的调用。当程序出现错误对话框时，此时用 `OD` 的暂停键，会暂停在调用这个子程序代码里面。但在汇编代码里面，这个完成绘制对话框的 `CALL` 里面也会调用很多的 `CALL`，而且会一层一层地调用，最终还会调用系统。这一点可以暂停后我们会在堆栈中发现。

那怎么才能一步步从最后面的一层一层的 `CALL` 慢慢返回到我们要找的最原始的调用子程序的 `CALL` 呢？这个是关键。如果找到了，这个 `CALL` 所在的段也就是最关键的了，能跳过它的跳转也是最关键的跳转了。这一点并不难。

我们再来说一下 `CALL` 的调用。调用一个子程序的时候：
1. 向堆栈中压入下一行程序的地址；
2. `JMP` 到 `call` 的子程序地址处。

关键是第 1 点：在堆栈中压入了下一行程序的地址，其目的是为了调用子程序结束后返回用的。如果走到一个 `CALL` 的时候我们 `F7` 一次跟进后，会停在所调用的子程序的开头第一个代码处，这时看堆栈的第一行压入了调用时的下一行地址。这个也是我们要利用的：

我们只要在程序某段的开头第一行代码下断（下面也行，这样可以避免不必要的干扰），中断后在堆栈中的第一行“选反汇编中跟踪”，就可以返回到上一层的调用这里的地方了。回到上一层后，如果发现关键段不是这段，再在此段的开头第一行下断，向上找。这样可以一层一层向上找，直到找到最初的调用的 `CALL`，这里就是关键段了！

当然，暂停后用 `Alt+K` 也可以看的到（但觉得有时不能准确定位）。

## 实践

我的用的是 `photoshop7.0`。`8.0` 不上手，但发现市面所能找到的都有时间限制（我这里），只能在 `2003` 之前用，所以我的机器一段时间还处在 `2` 年前火了，改之。

将时间改在 `2004` 运行程序，当程序“初始化……”时，出现错误对话框“不能完成请求，因为该序列号已过时”；此时暂停。

暂停后在堆栈中找能返回主程序：

`0012E7C8 | 00BC0067 返回到 Photosho.00BC0067 来自 USER32.DialogBoxParamW`

返回到这里：

```
00BC0061  |.  FF>call dword ptr ds:[<&USER32.Dialog>; \DialogBoxParamW
00BC0067  |.  89>mov dword ptr ss:[ebp-24],eax
```

在这一段的开头部分下断：(并先改时间到 `2003` 年看看 `2003` 年能不能断下，如果能断下，证明时间限制的分叉点还在上面还要继续向上一层找关键哦！）

```
00BBFF20  /$  55 push ebp
00BBFF21  |.  8B>mov ebp,esp
00BBFF23  |.  6A>push -1
00BBFF25  |.  68>push Photosho.00FC6F38
00BBFF2A  |.  68>push Photosho.00C3C678                 ;  SE 句柄安装
00BBFF2F  |.  64>mov eax,dword ptr fs:[0]
00BBFF35  |.  50 push eax
00BBFF36  |.  64>mov dword ptr fs:[0],esp
00BBFF3D  |.  83>sub esp,54
00BBFF40  |.  53 push ebx
00BBFF41  |.  56 push esi
00BBFF42  |.  57 push edi
00BBFF43  |.  C7>mov dword ptr ss:[ebp-24],-1
00BBFF4A  |.  C7>mov dword ptr ss:[ebp-20],0
00BBFF51  |.  A1>mov eax,dword ptr ds:[1203FA0]
00BBFF56  |.  85>test eax,eax
00BBFF58  |.  0F>jnz Photosho.00BC0027
00BBFF5E  |.  33>xor edi,edi
00BBFF60  |.  A1>mov eax,dword ptr ds:[1203F98]
00BBFF65  |.  85>test eax,eax
00BBFF67  |.  74>je short Photosho.00BBFF74
00BBFF69  |.  68>push Photosho.00F72A18                 ; /String = "IsolationAware function called after IsolationAwareCleanup "
00BBFF6E  |.  FF>call dword ptr ds:[<&KERNEL32.Outp>; \OutputDebugStringA
00BBFF74  |>  A1>mov eax,dword ptr ds:[1203FA0]
00BBFF79  |.  85>test eax,eax
00BBFF7B  |.  75>jnz short Photosho.00BBFFFB
00BBFF7D  |.  A1>mov eax,dword ptr ds:[1203F98]
00BBFF82  |.  85>test eax,eax
00BBFF84  |.  75>jnz short Photosho.00BBFF8F
00BBFF86  |.  E8>call Photosho.004BEEB0
00BBFF8B  |.  85>test eax,eax
00BBFF8D  |.  74>je short Photosho.00BC0000
00BBFF8F  |>  8B>mov ebx,dword ptr ds:[11648A4]
00BBFF95  |.  A1>mov eax,dword ptr ds:[128EE48]
00BBFF9A  |.  85>test eax,eax
00BBFF9C  |.  75>jnz short Photosho.00BBFFF0
00BBFF9E  |.  A1>mov eax,dword ptr ds:[128EE4C]
00BBFFA3  |.  85>test eax,eax
00BBFFA5  |.  75>jnz short Photosho.00BBFFDB
00BBFFA7  |.  68>push Photosho.01176B58                 ; /pModule = "Kernel32.dll"
00BBFFAC  |.  FF>call dword ptr ds:[<&KERNEL32.GetM>; \GetModuleHandleW
00BBFFB2  |.  8B>mov esi,eax
00BBFFB4  |.  85>test esi,esi
00BBFFB6  |.  75>jnz short Photosho.00BBFFD0
00BBFFB8  |.  FF>call dword ptr ds:[<&KERNEL32.GetL>; [GetLastError
00BBFFBE  |.  83>cmp eax,78
00BBFFC1  |.  75>jnz short Photosho.00BBFFD0
00BBFFC3  |.  68>push Photosho.01176B48                 ; /pModule = "Kernel32.dll"
00BBFFC8  |.  FF>call dword ptr ds:[<&KERNEL32.GetM>; \GetModuleHandleA
00BBFFCE  |.  8B>mov esi,eax
00BBFFD0  |>  8B>mov eax,esi
00BBFFD2  |.  A3>mov dword ptr ds:[128EE4C],eax
00BBFFD7  |.  85>test esi,esi
00BBFFD9  |.  74>je short Photosho.00BBFFE7
00BBFFDB  |>  68>push Photosho.01176B38                 ; /ProcNameOrOrdinal = "ActivateActCtx"
00BBFFE0  |.  50 push eax                                ; |hModule
00BBFFE1  |.  FF>call dword ptr ds:[<&KERNEL32.GetP>; \GetProcAddress
00BBFFE7  |>  A3>mov dword ptr ds:[128EE48],eax
00BBFFEC  |.  85>test eax,eax
00BBFFEE  |.  74>je short Photosho.00BBFFF7
00BBFFF0  |>  8D>lea ecx,dword ptr ss:[ebp-20]
00BBFFF3  |.  51 push ecx
00BBFFF4  |.  53 push ebx
00BBFFF5  |.  FF>call eax
00BBFFF7  |>  85>test eax,eax
00BBFFF9  |.  74>je short Photosho.00BC0000
00BBFFFB  |>  BF>mov edi,1
00BC0000  |>  85>test edi,edi
00BC0002  |.  75>jnz short Photosho.00BC0027
00BC0004  |.  FF>call dword ptr ds:[<&KERNEL32.GetL>; [GetLastError
00BC000A  |.  83>cmp eax,7F
00BC000D  |.  74>je short Photosho.00BC0014
00BC000F  |.  83>cmp eax,78
00BC0012  |.  75>jnz short Photosho.00BC001F
00BC0014  |>  BF>mov edi,1
00BC0019  |.  89>mov dword ptr ds:[1203FA0],edi
00BC001F  |>  85>test edi,edi
00BC0021  |.  75>jnz short Photosho.00BC0027
00BC0023  |.  33>xor eax,eax
00BC0025  |.  EB>jmp short Photosho.00BC002C
00BC0027  |>  B8>mov eax,1
00BC002C  |>  85>test eax,eax
00BC002E  |.  75>jnz short Photosho.00BC0046
00BC0030  |.  83>or eax,FFFFFFFF
00BC0033  |.  8B>mov ecx,dword ptr ss:[ebp-10]
00BC0036  |.  64>mov dword ptr fs:[0],ecx
00BC003D  |.  5F pop edi
00BC003E  |.  5E pop esi
00BC003F  |.  5B pop ebx
00BC0040  |.  8B>mov esp,ebp
00BC0042  |.  5D pop ebp
00BC0043  |.  C2>retn 14
00BC0046  |>  C7>mov dword ptr ss:[ebp-4],0
00BC004D  |.  8B>mov edx,dword ptr ss:[ebp+18]
00BC0050  |.  52 push edx                                 ; /lParam
00BC0051  |.  8B>mov eax,dword ptr ss:[ebp+14]             ; |
00BC0054  |.  50 push eax                                  ; |DlgProc
00BC0055  |.  8B>mov ecx,dword ptr ss:[ebp+10]             ; |
00BC0058  |.  51 push ecx                                  ; |hOwner
00BC0059  |.  8B>mov edx,dword ptr ss:[ebp+C]              ; |
00BC005C  |.  52 push edx                                   ; |pTemplate
00BC005D  |.  8B>mov eax,dword ptr ss:[ebp+8]              ; |
00BC0060  |.  50 push eax                                   ; |hInst
00BC0061  |.  FF>call dword ptr ds:[<&USER32.Dialog>; \DialogBoxParamW
00BC0067  |.  89>mov dword ptr ss:[ebp-24],eax
```

下断：`00BBFF20 /$ 55 push ebp` `ctrl+f2` 重新运行 `f9` 断在这里

在堆栈中：`0012E854 00BBDDA6 返回到 Photosho.00BBDDA6 来自 Photosho.00BBFF20`

```
00BBDDA1  |.  E8>call Photosho.00BBFF20                     ; \第 2 次
00BBDDA6  |.  EB>jmp short Photosho.00BBDDC2
```

这段：

```
00BBDC30  /$  55 push ebp                                    ;  第 2 次
00BBDC31  |.  8B>mov ebp,esp
00BBDC33  |.  6A>push -1
00BBDC35  |.  68>push Photosho.00F49346                     ;  SE 句柄安装
00BBDC3A  |.  64>mov eax,dword ptr fs:[0]
00BBDC40  |.  50 push eax
00BBDC41  |.  64>mov dword ptr fs:[0],esp
00BBDC48  |.  81>sub esp,288
00BBDC4E  |.  53 push ebx
00BBDC4F  |.  56 push esi
00BBDC50  |.  57 push edi
00BBDC51  |.  89>mov dword ptr ss:[ebp-10],esp
00BBDC54  |.  8D>lea ecx,dword ptr ss:[ebp-18]
00BBDC57  |.  E8>call Photosho.00919EF0
00BBDC5C  |.  33>xor ebx,ebx
00BBDC5E  |.  89>mov dword ptr ss:[ebp-4],ebx
00BBDC61  |.  C6>mov byte ptr ss:[ebp-28],1
00BBDC65  |.  FF>call dword ptr ds:[<&USER32.GetFoc>; [GetFocus
00BBDC6B  |.  89>mov dword ptr ss:[ebp-1C],eax
00BBDC6E  |.  8D>lea ecx,dword ptr ss:[ebp-30]
00BBDC71  |.  E8>call Photosho.00BCE790
00BBDC76  |.  C6>mov byte ptr ss:[ebp-4],1
00BBDC7A  |.  88>mov byte ptr ss:[ebp-24],bl
00BBDC7D  |.  E8>call Photosho.009DD610
00BBDC82  |.  84>test al,al
00BBDC84  |.  74>je short Photosho.00BBDCBD
00BBDC86  |.  8B>mov eax,dword ptr ss:[ebp+C]
```



```
00BBDC89  |.  66>mov si,word ptr ds:\[eax+1E\]  

00BBDC8D  |.  66>inc si  

00BBDC8F  |.  88>mov byte ptr ss:\[ebp-4\],bl  

00BBDC92  |.  8D>lea ecx,dword ptr ss:\[ebp-30\]  

00BBDC95  |.  E8>call Photosho.00BCE7D0  

00BBDC9A  |.  C7>mov dword ptr ss:\[ebp-4\],-1  

00BBDCA1  |.  8D>lea ecx,dword ptr ss:\[ebp-18\]  

00BBDCA4  |.  E8>call Photosho.00919F70  

00BBDCA9  |.  66>mov ax,si  

00BBDCAC  |.  8B>mov ecx,dword ptr ss:\[ebp-C\]  

00BBDCAF  |.  64>mov dword ptr fs:\[0\],ecx  

00BBDCB6  |.  5F pop edi  

00BBDCB7  |.  5E pop esi  

00BBDCB8  |.  5B pop ebx  

00BBDCB9  |.  8B>mov esp,ebp  

00BBDCBB  |.  5D pop ebp  

00BBDCBC  |.  C3 retn  

00BBDCBD  |>  C6>mov byte ptr ds:\[1254582\],1  

00BBDCC4  |.  C6>mov byte ptr ds:\[1254583\],1  

00BBDCCB  |.  68>push Photosho.01255E9E  

00BBDCD0  |.  E8>call Photosho.00B46720  

00BBDCD5  |.  83>add esp,4  

00BBDCD8  |.  E8>call Photosho.00BBAF70  

00BBDCDD  |.  33>xor esi,esi  

00BBDCDF  |.  89>mov dword ptr ss:\[ebp-34\],esi  

00BBDCE2  |.  8B>mov ecx,dword ptr ds:\[125457C\]  

00BBDCE8  |.  51 push ecx                   ; /hWnd => 002101D6 ('Adobe Photoshop',class='Photoshop')  

00BBDCE9  |.  8B>mov edi,dword ptr ds:\[<&USER32.IsW>; |USER32.IsWindow  

00BBDCEF  |.  FF>call edi                   ; \\IsWindow  

00BBDCF1  |.  85>test eax,eax  

00BBDCF3  |.  74>je short Photosho.00BBDD00  

00BBDCF5  |.  8B>mov edx,dword ptr ds:\[125457C\]  

00BBDCFB  |.  89>mov dword ptr ss:\[ebp-34\],edx  

00BBDCFE  |.  EB>jmp short Photosho.00BBDD15  

00BBDD00  |>  A1>mov eax,dword ptr ds:\[125555C\]  

00BBDD05  |.  50 push eax  

00BBDD06  |.  FF>call edi  

00BBDD08  |.  85>test eax,eax  

00BBDD0A  |.  74>je short Photosho.00BBDD18  

00BBDD0C  |.  8B>mov ecx,dword ptr ds:\[125555C\]  

00BBDD12  |.  89>mov dword ptr ss:\[ebp-34\],ecx  

00BBDD15  |>  8B>mov esi,dword ptr ss:\[ebp-34\]  

00BBDD18  |>  53 push ebx  

00BBDD19  |.  56 push esi  

00BBDD1A  |.  E8>call Photosho.00BBE1A0  

00BBDD1F  |.  83>add esp,8  

00BBDD22  |.  88>mov byte ptr ss:\[ebp-3C\],al  

00BBDD25  |.  E8>call Photosho.00C22960  

00BBDD2A  |.  88>mov byte ptr ss:\[ebp-38\],al  

00BBDD2D  |.  A1>mov eax,dword ptr ds:\[125555C\]  

00BBDD32  |.  3B>cmp esi,eax  

00BBDD34  |.  74>je short Photosho.00BBDD5F  

00BBDD36  |.  50 push eax  

00BBDD37  |.  FF>call edi  

00BBDD39  |.  85>test eax,eax  

00BBDD3B  |.  74>je short Photosho.00BBDD5F  

00BBDD3D  |.  8B>mov edx,dword ptr ds:\[125555C\]  

00BBDD43  |.  52 push edx                   ; /hWnd => 002101D6 ('Adobe Photoshop',class='Photoshop')  

00BBDD44  |.  FF>call dword ptr ds:\[<&USER32.IsWind>; \\IsWindowEnabled  

00BBDD4A  |.  85>test eax,eax  

00BBDD4C  |.  74>je short Photosho.00BBDD5F  

00BBDD4E  |.  53 push ebx                   ; /Enable  

00BBDD4F  |.  A1>mov eax,dword ptr ds:\[125555C\]     ; |  

00BBDD54  |.  50 push eax                   ; |hWnd => 002101D6 ('Adobe Photoshop',class='Photoshop')  

00BBDD55  |.  FF>call dword ptr ds:\[<&USER32.Enable>; \\EnableWindow  

00BBDD5B  |.  C6>mov byte ptr ss:\[ebp-24\],1  

00BBDD5F  |>  8B>mov eax,dword ptr ss:\[ebp+C\]  

00BBDD62  |.  3B>cmp eax,ebx  

00BBDD64  |.  74>je short Photosho.00BBDDD7  

00BBDD66  |.  66>mov cx,word ptr ss:\[ebp+14\]  

00BBDD6A  |.  66>mov word ptr ss:\[ebp-50\],cx  

00BBDD6E  |.  C7>mov dword ptr ss:\[ebp-4C\],1  

00BBDD75  |.  89>mov dword ptr ss:\[ebp-48\],ebx  

00BBDD78  |.  89>mov dword ptr ss:\[ebp-44\],eax  

00BBDD7B  |.  8B>mov edi,dword ptr ss:\[ebp+8\]  

00BBDD7E  |.  8A>mov dl,byte ptr ds:\[edi\]  

00BBDD80  |.  88>mov byte ptr ss:\[ebp-40\],dl  

00BBDD83  |.  A0>mov al,byte ptr ds:\[12554EF\]  

00BBDD88  |.  84>test al,al  

00BBDD8A  |.  74>je short Photosho.00BBDDA8  

00BBDD8C  |.  8D>lea eax,dword ptr ss:\[ebp-50\]  

00BBDD8F  |.  50 push eax  

00BBDD90  |.  68>push Photosho.00BBAFC0  

00BBDD95  |.  56 push esi  

00BBDD96  |.  68>push 7E5  

00BBDD9B  |.  E8>call Photosho.00BE7140  

00BBDDA0  |.  50 push eax                   ; |Arg1  

00BBDDA1  |.  E8>call Photosho.00BBFF20     ; \\第 2 次  

00BBDDA6  |.  EB>jmp short Photosho.00BBDDC2  

……………………………………………………………………………………………………………………

下断：`00BBDC30` `/$` `55` `push` `ebp` `ctrl+f2` `重新运行` `f9` `断在这里`

……………………………………………………………………………………………………………………

堆栈中：

`0012EB10` `00BBDC23` `返回到` `Photosho.00BBDC23` `来自` `Photosho.00BBDC30`

……………………………………………………………………………………………………………………

`00BBDC10` `/$` `8B>mov` `eax,dword` `ptr` `ss:\[esp+8\]` `;` `第` `3` `次`

`00BBDC14` `|.` `8B>mov` `ecx,dword` `ptr` `ss:\[esp+4\]`

`00BBDC18` `|.` `6A>push` `0` `;` `/Arg4` `=` `00000000`

`00BBDC1A` `|.` `6A>push` `0` `;` `|Arg3` `=` `00000000`

`00BBDC1C` `|.` `50` `push` `eax` `;` `|Arg2`

`00BBDC1D` `|.` `51` `push` `ecx` `;` `|Arg1`

`00BBDC1E` `|.` `E8>call` `Photosho.00BBDC30` `;` `\\第` `3` `次`

`00BBDC23` `|.` `83>add` `esp,10`

`00BBDC26` `|.` `48` `dec` `eax`

`00BBDC27` `\\.` `C3` `retn`

……………………………………………………………………………………………………………………

下断：`00BBDC10` `/$` `8B>mov` `eax,dword` `ptr` `ss:\[esp+8\]` `;` `第` `3` `次` `ctrl+f2` `重新运行` `f9` `断在这里`

……………………………………………………………………………………………………………………

堆栈中：

`0012EB24` `00B6CCB6` `返回到` `Photosho.00B6CCB6` `来自` `Photosho.00BBDC10`

……………………………………………………………………………………………………………………

`00B6CC20` `/$` `6A>push` `-1` `;` `第` `4` `次`

`00B6CC22` `|.` `68>push` `Photosho.00F42BDE` `;` `SE` `句柄安装`

`00B6CC27` `|.` `64>mov` `eax,dword` `ptr` `fs:\[0\]`

`00B6CC2D` `|.` `50` `push` `eax`

`00B6CC2E` `|.` `64>mov` `dword` `ptr` `fs:\[0\],esp`

`00B6CC35` `|.` `83>sub` `esp,24`

`00B6CC38` `|.` `8D>lea` `ecx,dword` `ptr` `ss:\[esp+4\]`

`00B6CC3C` `|.` `E8>call` `Photosho.00B724D0`

`00B6CC41` `|.` `68>push` `Photosho.00B72800` `;` `入口地址`

`00B6CC46` `|.` `68>push` `Photosho.00B724D0` `;` `入口地址`

`00B6CC4B` `|.` `6A>push` `5`

`00B6CC4D` `|.` `8D>lea` `eax,dword` `ptr` `ss:\[esp+14\]`

`00B6CC51` `|.` `6A>push` `4`

`00B6CC53` `|.` `50` `push` `eax`

`00B6CC54` `|.` `C7>mov` `dword` `ptr` `ss:\[esp+40\],0`

`00B6CC5C` `|.` `E8>call` `Photosho.00C3BB6A`

`00B6CC61` `|.` `8B>mov` `ecx,dword` `ptr` `ss:\[esp+38\]`

`00B6CC65` `|.` `B8>mov` `eax,2`

`00B6CC6A` `|.` `51` `push` `ecx`

`00B6CC6B` `|.` `89>mov` `dword` `ptr` `ss:\[esp+30\],eax`

`00B6CC6F` `|.` `8D>lea` `ecx,dword` `ptr` `ss:\[esp+8\]`

`00B6CC73` `|.` `89>mov` `dword` `ptr` `ss:\[esp+4\],eax`

`00B6CC77` `|.` `E8>call` `Photosho.00B72820`

`00B6CC7C` `|.` `68>push` `Photosho.01253320`

`00B6CC81` `|.` `8D>lea` `ecx,dword` `ptr` `ss:\[esp+C\]`

`00B6CC85` `|.` `E8>call` `Photosho.00B72820`

`00B6CC8A` `|.` `8B>mov` `ecx,dword` `ptr` `ss:\[esp+34\]`

`00B6CC8E` `|.` `66>mov` `dx,word` `ptr` `ss:\[esp+3C\]`

`00B6CC93` `|.` `8D>lea` `eax,dword` `ptr` `ss:\[esp\]`

`00B6CC97` `|.` `C6>mov` `byte` `ptr` `ss:\[esp+1C\],1`

`00B6CC9C` `|.` `50` `push` `eax`

`00B6CC9D` `|.` `51` `push` `ecx`

`00B6CC9E` `|.` `66>mov` `word` `ptr` `ss:\[esp+26\],0`

`00B6CCA5` `|.` `66>mov` `word` `ptr` `ss:\[esp+28\],5`

`00B6CCAC` `|.` `66>mov` `word` `ptr` `ss:\[esp+2A\],dx`

`00B6CCB1` `|.` `E8>call` `Photosho.00BBDC10` `;` `第` `4` `次`

`00B6CCB6` `|.` `83>add` `esp,8`

`00B6CCB9` `|.` `68>push` `Photosho.00B72800` `;` `入口地址`

……………………………………………………………………………………………………………………

下断：`00B6CC20` `/$` `6A>push` `-1` `;` `第` `4` `次` `ctrl+f2` `重新运行` `f9` `断在这里`

……………………………………………………………………………………………………………………

堆栈

`0012EB60` `00B6D23A` `返回到` `Photosho.00B6D23A` `来自` `Photosho.00B6CC20`

……………………………………………………………………………………………………………………

`00B6D220` `/$` `51` `push` `ecx` `;` `第` `5` `次`

`00B6D221` `|.` `8B>mov` `eax,dword` `ptr` `ss:\[esp+C\]`

`00B6D225` `|.` `8B>mov` `ecx,dword` `ptr` `ss:\[esp+8\]`

`00B6D229` `|.` `50` `push` `eax`

`00B6D22A` `|.` `8D>lea` `edx,dword` `ptr` `ss:\[esp+7\]`

`00B6D22E` `|.` `51` `push` `ecx`

`00B6D22F` `|.` `52` `push` `edx`

`00B6D230` `|.` `C6>mov` `byte` `ptr` `ss:\[esp+F\],0`

`00B6D235` `|.` `E8>call` `Photosho.00B6CC20` `;` `第` `5` `次`

`00B6D23A` `|.` `83>add` `esp,10`

`00B6D23D` `\\.` `C3` `retn`

……………………………………………………………………………………………………………………

下断 `00B6D220` `/$` `51` `push` `ecx` `;` `第` `5` `次` `ctrl+f2` `重新运行` `f9` `断在这里`
```



# 堆栈分析

### 返回堆栈

```
0012EB74    00B7F78F    返回到 Photosho.00B7F78F 来自 Photosho.00B6D220
```

### 代码段 `00B7F760`

```
00B7F760    $   53  push ebx                     ;  第 6 次
00B7F761    .   8B> mov ebx,dword ptr ss:[esp+C]
00B7F765    .   56  push esi
00B7F766    .   8B> mov esi,dword ptr ss:[esp+14]
00B7F76A    .   57  push edi
00B7F76B    .   8B> mov edi,dword ptr ss:[esp+10]
00B7F76F    .   56  push esi
00B7F770    .   53  push ebx
00B7F771    .   57  push edi
00B7F772    .   E8> call Photosho.00B7F540
00B7F777    .   83> add esp,0C
00B7F77A    .   8B> mov ecx,esi
00B7F77C    .   E8> call Photosho.00B6FF70
00B7F781    .   84> test al,al
00B7F783    .   75> jnz short Photosho.00B7F792
00B7F785    .   66> mov ax,word ptr ds:[edi]
00B7F788    .   50  push eax
00B7F789    .   56  push esi
00B7F78A    .   E8> call Photosho.00B6D220       ;  第 6 次
00B7F78F    .   83> add esp,8
00B7F792    |>  C6> mov byte ptr ds:[1256262],0
```

### 堆栈状态

```
0012EB8C    0043339D    返回到 Photosho.0043339D 来自 Photosho.00B7F760  返回后到了最后点
```

### 代码段 `00433160`

```
00433160     $  55  push ebp                     ;  第 7 次到了
00433161     .  8B> mov ebp,esp
00433163     .  6A> push -1
00433165     .  68> push Photosho.00E5396A        ;  SE 句柄安装
0043316A     .  64> mov eax,dword ptr fs:[0]
00433170     .  50  push eax
00433171     .  64> mov dword ptr fs:[0],esp
00433178     .  81> sub esp,54C
0043317E     .  53  push ebx
0043317F     .  56  push esi
00433180     .  57  push edi
00433181     .  89> mov dword ptr ss:[ebp-10],esp
00433184     .  33> xor ebx,ebx
00433186     .  88> mov byte ptr ss:[ebp-114],bl
0043318C     .  88> mov byte ptr ss:[ebp-113],bl
00433192     .  8D> lea eax,dword ptr ss:[ebp-114]
00433198     .  50  push eax
00433199     .  68> push Photosho.012561E8
0043319E     .  E8> call Photosho.00417E80
004331A3     .  83> add esp,8
004331A6     .  84> test al,al
004331A8     .  0F> jnz Photosho.004332DC
004331AE     .  E8> call Photosho.00B74B90
004331B3     .  8D> lea ecx,dword ptr ss:[ebp-22C]
004331B9     .  E8> call Photosho.00B724D0
004331BE     .  89> mov dword ptr ss:[ebp-4],ebx
004331C1     .  E8> call Photosho.00B6E090
004331C6     .  50  push eax
004331C7     .  6A> push -1
004331C9     .  68> push Photosho.011648EC        ;  ASCII "$$$/MacApp/MissingComponents=Adobe Photoshop can not run on this computer because ⁰"
004331CE     .  8D> lea ecx,dword ptr ss:[ebp-234]
004331D4     .  E8> call Photosho.00B72530
004331D9     .  C6> mov byte ptr ss:[ebp-4],1
004331DD     .  8D> lea ecx,dword ptr ss:[ebp-234]
004331E3     .  51  push ecx
004331E4     .  8D> lea ecx,dword ptr ss:[ebp-22C]
004331EA     .  E8> call Photosho.00B72820
004331EF     .  88> mov byte ptr ss:[ebp-4],bl
004331F2     .  8D> lea ecx,dword ptr ss:[ebp-234]
004331F8     .  E8> call Photosho.00B72800
004331FD     .  8D> lea edx,dword ptr ss:[ebp-114]
00433203     .  52  push edx
00433204     .  8D> lea eax,dword ptr ss:[ebp-238]
0043320A     .  50  push eax
0043320B     .  E8> call Photosho.004335D0
00433210     .  83> add esp,8
00433213     .  C6> mov byte ptr ss:[ebp-4],2
00433217     .  6A> push 1
00433219     .  50  push eax
0043321A     .  53  push ebx
0043321B     .  8D> lea ecx,dword ptr ss:[ebp-22C]
00433221     .  E8> call Photosho.00B737B0
00433226     .  88> mov byte ptr ss:[ebp-4],bl
00433229     .  8D> lea ecx,dword ptr ss:[ebp-238]
0043322F     .  E8> call Photosho.00B72800
00433234     .  88> mov byte ptr ss:[ebp-228],bl
0043323A     .  88> mov byte ptr ss:[ebp-227],bl
00433240     .  6A> push 1                       ;  /Arg3 = 00000001
00433242     .  68> push 0FF                     ;  |Arg2 = 000000FF
00433247     .  8D> lea ecx,dword ptr ss:[ebp-228]     ;  |
0043324D     .  51  push ecx                    ;  |Arg1
0043324E     .  8D> lea ecx,dword ptr ss:[ebp-22C]     ;  |
00433254     .  E8> call Photosho.00B72F30        ;  \Photosho.00B72F30
00433259     .  68> push Photosho.011648E8
0043325E     .  8D> lea ecx,dword ptr ss:[ebp-228]
00433264     .  E8> call Photosho.00B784E0
00433269     .  68> push Photosho.01203F7C
0043326E     .  8D> lea ecx,dword ptr ss:[ebp-440]
00433274     .  E8> call Photosho.00B781C0
00433279     .  8B> mov esi,eax
0043327B     .  68> push Photosho.01203F7C
00433280     .  8D> lea ecx,dword ptr ss:[ebp-544]
00433286     .  E8> call Photosho.00B781C0
0043328B     .  8B> mov ebx,eax
0043328D     .  68> push Photosho.01203F7C
00433292     .  8D> lea ecx,dword ptr ss:[ebp-33C]
00433298     .  E8> call Photosho.00B781C0
0043329D     .  50  push eax
0043329E     .  56  push esi
0043329F     .  53  push ebx
004332A0     .  8D> lea edx,dword ptr ss:[ebp-228]
004332A6     .  52  push edx
004332A7     .  E8> call Photosho.00BBAD70
004332AC     .  68> push 89
004332B1     .  E8> call Photosho.00BBE060
004332B6     .  83> add esp,14
004332B9     .  C7> mov dword ptr ss:[ebp-4],-1
004332C0     .  8D> lea ecx,dword ptr ss:[ebp-22C]
004332C6     .  E8> call Photosho.00B72800
004332CB     .  8B> mov ecx,dword ptr ss:[ebp-C]
004332CE     .  64> mov dword ptr fs:[0],ecx
004332D5     .  5F  pop edi
004332D6     .  5E  pop esi
004332D7     .  5B  pop ebx
004332D8     .  8B> mov esp,ebp
004332DA     .  5D  pop ebp
004332DB     .  C3  retn
004332DC     >  6A> push 0A
004332DE     .  E8> call Photosho.00C161A0
004332E3     .  E8> call Photosho.0040FE90
004332E8     .  E8> call Photosho.00C20380
004332ED     .  E8> call Photosho.0040FE90
004332F2     .  E8> call Photosho.0040FE90
004332F7     .  E8> call Photosho.00BA0340
004332FC     .  E8> call Photosho.0040FE90
00433301     .  68> push 0A4
00433306     .  E8> call Photosho.00B97E30
0043330B     .  83> add esp,8
0043330E     .  89> mov dword ptr ss:[ebp-54C],eax
00433314     .  C7> mov dword ptr ss:[ebp-4],3
0043331B     .  3B> cmp eax,ebx
0043331D     .  74> je short Photosho.0043332A
0043331F     .  8B> mov ecx,eax
00433321     .  E8> call Photosho.008449B0
00433326     .  8B> mov esi,eax
00433328     .  EB> jmp short Photosho.0043332C
0043332A     >  33> xor esi,esi
0043332C     >  89> mov dword ptr ss:[ebp-548],esi
00433332     .  C7> mov dword ptr ss:[ebp-4],-1
00433339     .  88> mov byte ptr ss:[ebp-124],bl
0043333F     .  8B> mov ecx,esi
00433341     .  E8> call Photosho.00844A90
00433346     .  68> push Photosho.00433430
0043334B     .  E8> call Photosho.00C3BFE8
00433350     .  68> push Photosho.00433440
00433355     .  E8> call Photosho.00C3BFD8
0043335A     .  83> add esp,8
0043335D     .  8B> mov edi,eax
0043335F     .  89> mov dword ptr ss:[ebp-120],edi
00433365     .  C7> mov dword ptr ss:[ebp-4],4
0043336C     .  C6> mov byte ptr ss:[ebp-4],5
00433370     .  8B> mov ecx,esi
00433372     .  E8> call Photosho.00846920
00433377     .  EB> jmp short Photosho.004333E1
00433379     .  C6> mov byte ptr ss:[ebp-124],1
00433380     .  8B> mov ecx,dword ptr ss:[ebp-230]
00433386     .  8D> lea eax,dword ptr ds:[ecx+10]
00433389     .  66> cmp word ptr ds:[eax],0
0043338D     .  74> je short Photosho.004333A6
0043338F     .  8D> lea edx,dword ptr ds:[ecx+18]
00433392     .  52  push edx
00433393     .  83> add ecx,14
00433396     .  51  push ecx
00433397     .  50  push eax
00433398     .  E8> call Photosho.00B7F760        ;  第 7 次
0043339D     .  83> add esp,0C
```

# 断点设置

下断位置：

```
00433160     $  55  push ebp               ;  第 7 次到了
```

# 调试策略

## 定位转折点

如何确定转折点就在这段代码中？

将时间调整到 2003 年时发现也断下来了！

## 观察方法

慢慢跟读，并且把时间一次改成 2004 年，一次改成 2003 年，观察到底哪里不同！

```
00433160     $  55  push ebp                     ;  第 7 次到了
00433161     .  8B> mov ebp,esp
00433163     .  6A> push -1
00433165     .  68> push Photosho.00E5396A        ;  SE 句柄安装
0043316A     .  64> mov eax,dword ptr fs:[0]
00433170     .  50  push eax
```



```
00433171   .  64>mov dword ptr fs:[0],esp

00433178   .  81>sub esp,54C

0043317E   .  53 push ebx

0043317F   .  56 push esi

00433180   .  57 push edi

00433181   .  89>mov dword ptr ss:[ebp-10],esp

00433184   .  33>xor ebx,ebx

00433186   .  88>mov byte ptr ss:[ebp-114],bl

0043318C   .  88>mov byte ptr ss:[ebp-113],bl

00433192   .  8D>lea eax,dword ptr ss:[ebp-114]

00433198   .  50 push eax

00433199   .  68>push Photosho.012561E8

0043319E   .  E8>call Photosho.00417E80

004331A3   .  83>add esp,8

004331A6   .  84>test al,al

004331A8   .  0F>jnz Photosho.004332DC

004331AE   .  E8>call Photosho.00B74B90

004331B3   .  8D>lea ecx,dword ptr ss:[ebp-22C]

004331B9   .  E8>call Photosho.00B724D0

004331BE   .  89>mov dword ptr ss:[ebp-4],ebx

004331C1   .  E8>call Photosho.00B6E090

004331C6   .  50 push eax

004331C7   .  6A>push -1

004331C9   .  68>push Photosho.011648EC                 ;  ASCII "$$$/MacApp/MissingComponents=Adobe Photoshop can not run on this computer because ⁰"

004331CE   .  8D>lea ecx,dword ptr ss:[ebp-234]

004331D4   .  E8>call Photosho.00B72530

004331D9   .  C6>mov byte ptr ss:[ebp-4],1

004331DD   .  8D>lea ecx,dword ptr ss:[ebp-234]

004331E3   .  51 push ecx

004331E4   .  8D>lea ecx,dword ptr ss:[ebp-22C]

004331EA   .  E8>call Photosho.00B72820

004331EF   .  88>mov byte ptr ss:[ebp-4],bl

004331F2   .  8D>lea ecx,dword ptr ss:[ebp-234]

004331F8   .  E8>call Photosho.00B72800

004331FD   .  8D>lea edx,dword ptr ss:[ebp-114]

00433203   .  52 push edx

00433204   .  8D>lea eax,dword ptr ss:[ebp-238]

0043320A   .  50 push eax

0043320B   .  E8>call Photosho.004335D0

00433210   .  83>add esp,8

00433213   .  C6>mov byte ptr ss:[ebp-4],2

00433217   .  6A>push 1

00433219   .  50 push eax

0043321A   .  53 push ebx

0043321B   .  8D>lea ecx,dword ptr ss:[ebp-22C]

00433221   .  E8>call Photosho.00B737B0

00433226   .  88>mov byte ptr ss:[ebp-4],bl

00433229   .  8D>lea ecx,dword ptr ss:[ebp-238]

0043322F   .  E8>call Photosho.00B72800

00433234   .  88>mov byte ptr ss:[ebp-228],bl

0043323A   .  88>mov byte ptr ss:[ebp-227],bl

00433240   .  6A>push 1                                 ; /Arg3 = 00000001

00433242   .  68>push 0FF                               ; |Arg2 = 000000FF

00433247   .  8D>lea ecx,dword ptr ss:[ebp-228]         ; |

0043324D   .  51 push ecx                                ; |Arg1

0043324E   .  8D>lea ecx,dword ptr ss:[ebp-22C]         ; |

00433254   .  E8>call Photosho.00B72F30                  ; \Photosho.00B72F30

00433259   .  68>push Photosho.011648E8

0043325E   .  8D>lea ecx,dword ptr ss:[ebp-228]

00433264   .  E8>call Photosho.00B784E0

00433269   .  68>push Photosho.01203F7C

0043326E   .  8D>lea ecx,dword ptr ss:[ebp-440]

00433274   .  E8>call Photosho.00B781C0

00433279   .  8B>mov esi,eax

0043327B   .  68>push Photosho.01203F7C

00433280   .  8D>lea ecx,dword ptr ss:[ebp-544]

00433286   .  E8>call Photosho.00B781C0

0043328B   .  8B>mov ebx,eax

0043328D   .  68>push Photosho.01203F7C

00433292   .  8D>lea ecx,dword ptr ss:[ebp-33C]

00433298   .  E8>call Photosho.00B781C0

0043329D   .  50 push eax

0043329E   .  56 push esi

0043329F   .  53 push ebx

004332A0   .  8D>lea edx,dword ptr ss:[ebp-228]

004332A6   .  52 push edx

004332A7   .  E8>call Photosho.00BBAD70

004332AC   .  68>push 89

004332B1   .  E8>call Photosho.00BBE060

004332B6   .  83>add esp,14

004332B9   .  C7>mov dword ptr ss:[ebp-4],-1

004332C0   .  8D>lea ecx,dword ptr ss:[ebp-22C]

004332C6   .  E8>call Photosho.00B72800

004332CB   .  8B>mov ecx,dword ptr ss:[ebp-C]

004332CE   .  64>mov dword ptr fs:[0],ecx

004332D5   .  5F pop edi

004332D6   .  5E pop esi

004332D7   .  5B pop ebx

004332D8   .  8B>mov esp,ebp

004332DA   .  5D pop ebp

004332DB   .  C3 retn

004332DC   >  6A>push 0A

004332DE   .  E8>call Photosho.00C161A0

004332E3   .  E8>call Photosho.0040FE90

004332E8   .  E8>call Photosho.00C20380

004332ED   .  E8>call Photosho.0040FE90

004332F2   .  E8>call Photosho.0040FE90

004332F7   .  E8>call Photosho.00BA0340

004332FC   .  E8>call Photosho.0040FE90

00433301   .  68>push 0A4

00433306   .  E8>call Photosho.00B97E30

0043330B   .  83>add esp,8

0043330E   .  89>mov dword ptr ss:[ebp-54C],eax

00433314   .  C7>mov dword ptr ss:[ebp-4],3

0043331B   .  3B>cmp eax,ebx

0043331D   .  74>je short Photosho.0043332A

0043331F   .  8B>mov ecx,eax

00433321   .  E8>call Photosho.008449B0

00433326   .  8B>mov esi,eax

00433328   .  EB>jmp short Photosho.0043332C

0043332A   >  33>xor esi,esi

0043332C   >  89>mov dword ptr ss:[ebp-548],esi

00433332   .  C7>mov dword ptr ss:[ebp-4],-1

00433339   .  88>mov byte ptr ss:[ebp-124],bl

0043333F   .  8B>mov ecx,esi

00433341   .  E8>call Photosho.00844A90

00433346   .  68>push Photosho.00433430

0043334B   .  E8>call Photosho.00C3BFE8

00433350   .  68>push Photosho.00433440

00433355   .  E8>call Photosho.00C3BFD8

0043335A   .  83>add esp,8

0043335D   .  8B>mov edi,eax

0043335F   .  89>mov dword ptr ss:[ebp-120],edi

00433365   .  C7>mov dword ptr ss:[ebp-4],4

0043336C   .  C6>mov byte ptr ss:[ebp-4],5

00433370   .  8B>mov ecx,esi

00433372   .  E8>call Photosho.00846920                 ;  关键所在。

00433377   .  EB>jmp short Photosho.004333E1

00433379   .  C6>mov byte ptr ss:[ebp-124],1

00433380   .  8B>mov ecx,dword ptr ss:[ebp-230]

00433386   .  8D>lea eax,dword ptr ds:[ecx+10]

00433389   .  66>cmp word ptr ds:[eax],0

0043338D   .  74>je short Photosho.004333A6

0043338F   .  8D>lea edx,dword ptr ds:[ecx+18]

00433392   .  52 push edx

00433393   .  83>add ecx,14

00433396   .  51 push ecx

00433397   .  50 push eax

00433398   .  E8>call Photosho.00B7F760                 ;  所以这里就是真正调用“序列号过时”的 CALL，所以最直接的不让程序运行到这里

0043339D   .  83>add esp,0C

004333A0   .  B8>mov eax,Photosho.004333D3

004333A5   .  C3 retn
```

修改时间对比发现如果时间在 2003 之前，经过 `00433372   .  E8>call Photosho.00846920` 会最直接地到了下一句 `00433377   .  EB>jmp short Photosho.004333E1`

会跳过如果时间超过 2003 年走过 `00433372   .  E8>call Photosho.00846920` 出出现异常会到 `00433379   .  C6>mov byte ptr ss:[ebp-124],1`

问题就在 `00433372   .  E8>call Photosho.00846920`

…………………………………………………………………………………………………………………………

进入 `00433372   .  E8>call Photosho.00846920` ; 问题就在这里

…………………………………………………………………………………………………………………………

```
00846920  /$  55 push ebp

00846921  |.  8B>mov ebp,esp

00846923  |.  6A>push -1

00846925  |.  68>push Photosho.00EE1E7B                 ;  SE 句柄安装

0084692A  |.  64>mov eax,dword ptr fs:[0]

00846930  |.  50 push eax

00846931  |.  64>mov dword ptr fs:[0],esp

00846938  |.  83>sub esp,24

0084693B  |.  53 push ebx

0084693C  |.  56 push esi

0084693D  |.  57 push edi

0084693E  |.  89>mov dword ptr ss:[ebp-10],esp

00846941  |.  8B>mov esi,ecx

00846943  |.  E8>call Photosho.00876050

00846948  |.  33>xor ebx,ebx

0084694A  |.  89>mov dword ptr ss:[ebp-4],ebx

0084694D  |.  E8>call Photosho.0077D2E0                 ;  这里会异常跟进

00846952  |.  6A>push 4

00846954  |.  E8>call Photosho.00B97E30
```

……………………………………………………………………………………………………………………

进入 `0084694D  |.  E8>call Photosho.0077D2E0` ; 这里会异常跟进

…………………………………………………………………………………………………………………………

```
0077D3C2   .  F2>repne scas byte ptr es:[edi]

0077D3C4   .  F7>not ecx

0077D3C6   .  49 dec ecx

0077D3C7   .  88>mov byte ptr ds:[1248220],cl

0077D3CD   .  68>push 0FF

0077D3D2   .  8D>lea eax,dword ptr ss:[ebp-238]

0077D3D8   .  50 push eax

0077D3D9   .  68>push Photosho.01248221                 ;  ASCII "C:\Program Files\Adobe\Photoshop 7.0\"

0077D3DE   .  E8>call Photosho.00C3C010

0077D3E3   .  83>add esp,0C
```



```
0077D3E6   .  C6>mov byte ptr ds:[124831F],0  

0077D3ED   .  8B>mov ecx,dword ptr ds:[1235E50]     ;  Photosho.01235E58  

0077D3F3   .  E8>call Photosho.0077D250             ;  到这里还是会异常  

0077D3F8   .  8B>mov ecx,dword ptr ds:[1249550]     ;  Photosho.01249558  

0077D3FE   .  E8>call Photosho.0077D250  

0077D403   .  E8>call Photosho.00B468F0  

0077D408   .  6A>push 1  

0077D40A   .  8D>lea ecx,dword ptr ss:[ebp-12C]  

0077D410   .  E8>call Photosho.009DD670  

0077D415   .  C7>mov dword ptr ss:[ebp-4],1  

0077D41C   .  6A>push 1  

0077D41E   .  8D>lea ecx,dword ptr ss:[ebp-120]  
```

进入 `0077D3F3 . E8>call Photosho.0077D250`

```
0077D250  /$  56 push esi  

0077D251  |.  8B>mov esi,ecx  

0077D253  |.  8B>mov eax,dword ptr ds:[esi]  

0077D255  |.  FF>call dword ptr ds:[eax+20]         ;  还是有异常跟进  

0077D258  |.  C6>mov byte ptr ds:[esi+2C],1  

0077D25C  |.  66>mov cx,word ptr ds:[1256260]  

0077D263  |.  51 push ecx  

0077D264  |.  E8>call Photosho.00B95180  

0077D269  |.  83>add esp,4  

0077D26C  |.  5E pop esi  

0077D26D  \\.  C3 retn  
```

进入 `0077D255 |. FF>call dword ptr ds:[eax+20]`

并把时间调到 2003 与 2004 两次对比发现：

```
005D67B0   .  6A>push -1  

005D67B2   .  68>push Photosho.00E90A18             ;  SE 句柄安装  

005D67B7   .  64>mov eax,dword ptr fs:[0]  

005D67BD   .  50 push eax  

005D67BE   .  64>mov dword ptr fs:[0],esp  

005D67C5   .  51 push ecx  

005D67C6   .  56 push esi  

005D67C7   .  8B>mov esi,ecx  

005D67C9   .  8B>mov eax,dword ptr ds:[esi]  

005D67CB   .  FF>call dword ptr ds:[eax+2C]  

005D67CE   .  8A>mov al,byte ptr ds:[esi+51]  

005D67D1   .  84>test al,al  

005D67D3   .  74>je short Photosho.005D67E0         ;  跳  

005D67D5   .  8A>mov al,byte ptr ds:[esi+4B]  

005D67D8   .  84>test al,al  

005D67DA   .  75>jnz short Photosho.005D67E0  

005D67DC   .  B0>mov al,1  

005D67DE   .  EB>jmp short Photosho.005D67E2  

005D67E0   >  32>xor al,al  

005D67E2   >  8A>mov cl,byte ptr ds:[esi+4F]  

005D67E5   .  53 push ebx  

005D67E6   .  84>test cl,cl  

005D67E8   .  75>jnz short Photosho.005D67F5        ;  没  

005D67EA   .  8A>mov cl,byte ptr ds:[esi+50]  

005D67ED   .  84>test cl,cl  

005D67EF   .  75>jnz short Photosho.005D67F5        ;  没  

005D67F1   .  32>xor bl,bl  

005D67F3   .  EB>jmp short Photosho.005D67F7  

005D67F5   >  B3>mov bl,1  

005D67F7   >  84>test al,al  

005D67F9   .  74>je short Photosho.005D6808         ;  跳  

005D67FB   .  68>push FFFF9D22  

005D6800   .  E8>call Photosho.00B80070  

005D6805   .  83>add esp,4  

005D6808   >  84>test bl,bl  

005D680A   .  5B pop ebx  

005D680B   .  74>je short Photosho.005D6859         ;  跳  

005D680D   .  E8>call Photosho.00B6E090  

005D6812   .  50 push eax  

005D6813   .  6A>push -1  

005D6815   .  68>push Photosho.01187064             ;  ASCII "$$$/ErrorStrings/MissingEnigmaLibs=because certain required files were not found in the Adobe Photoshop 7.0 Required folder.  Please reinstall Photoshop to restore these files"  

005D681A   .  8D>lea ecx,dword ptr ss:[esp+10]  

005D681E   .  E8>call Photosho.00B72530  

005D6823   .  8D>lea ecx,dword ptr ss:[esp+4]  

005D6827   .  C7>mov dword ptr ss:[esp+10],0  

005D682F   .  51 push ecx  

005D6830   .  E8>call Photosho.00B800D0  

005D6835   .  83>add esp,4  

005D6838   .  8D>lea ecx,dword ptr ss:[esp+4]  

005D683C   .  C7>mov dword ptr ss:[esp+10],-1  

005D6844   .  E8>call Photosho.00B72800  

005D6849   .  5E pop esi  

005D684A   .  8B>mov ecx,dword ptr ss:[esp+4]  

005D684E   .  64>mov dword ptr fs:[0],ecx  

005D6855   .  83>add esp,10  

005D6858   .  C3 retn  

005D6859   >  8A>mov al,byte ptr ds:[esi+52]  

005D685C   .  84>test al,al  

005D685E    74>je short Photosho.005D686D          ;  没（这里与日期 2003 年与 2004 年不一样）关键点  

005D6860   .  68>push FFFF9D2C  

005D6865   .  E8>call Photosho.00B80070  

005D686A   .  83>add esp,4  
```

修改：`005D685E    74>je short Photosho.005D686D` `jmp` 即可！

`4dd466d0e1e0cc123afbe504da2462ca`
