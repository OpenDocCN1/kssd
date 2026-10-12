
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
