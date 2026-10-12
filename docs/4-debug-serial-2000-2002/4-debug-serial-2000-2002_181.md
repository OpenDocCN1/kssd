
# AT2000

> 原文：[`www.pediy.com/kssd/pediy03/forum1072.htm`](https://www.pediy.com/kssd/pediy03/forum1072.htm)

AT2000 的不完全破解！

软件下载：`http://www.modernagri.com/at2000.zip` (2.5M)

破解者：大老

主页：`HTTP://DALAO2001.YEAH.NET`

日期：2001.6.19

软件是用 `sentinel` 软件狗保护，并用 `BTFNT＋wwwpack` 加壳

这个软件的反跟踪做的非常好！要动态跟踪要麻烦点！嘿嘿

## 第一步：先脱壳

用 `TRW2000` 载入文件

用 `G` 命令 `G 668E10`

再用 `MAKEPE` 生成脱壳后的文件！

## 第二步：跟踪算出注册码

这一步比较麻烦注意！要先运行程序！不能先运行 `TRW` 或 `ICE`，

如果你运行的话嘿嘿！你的机器！就 `REST`!

进入注册画面！这时的调试环境已经被破坏！

这时候要运行一个程序叫 `superbpm` 然后把它的 `erase` 的选项打开

软件下载版主，主页工具里有！

不然的话嘿嘿！死机！

这时候运行 `TRW` 就可以跟踪了！

设断点 `BPX 5EE72C`

```
* Possible Reference to String Resource ID=00001: "歃:"
                      |
:005EE71F 6A01            push 00000001
:005EE721 8B4DFC          mov ecx, dword ptr [ebp-04]
:005EE724 E8688F0900      call 00687691
:005EE729 8B4DFC          mov ecx, dword ptr [ebp-04]
:005EE72C E82C000000      call 005EE75D  =====>关键 CALL
:005EE731 85C0            test eax, eax
:005EE733 740C            je 005EE741
```

`CALLL` 里的关键比较

```
:005EE810 3B45F0          cmp eax, dword ptr [ebp-10]
:005EE813 7D1F            jge 005EE834
:005EE815 8B4DF8          mov ecx, dword ptr [ebp-08]
:005EE818 83C103          add ecx, 00000003
:005EE81B 51              push ecx
:005EE81C 8B4DEC          mov ecx, dword ptr [ebp-14]
:005EE81F 83C15C          add ecx, 0000005C
:005EE822 E8C9F3E3FF      call 0042DBF0
:005EE827 0FBED0          movsx edx, al
:005EE82A 8B45F4          mov eax, dword ptr [ebp-0C]
:005EE82D 03C2            add eax, edx
:005EE82F 8945F4          mov dword ptr [ebp-0C], eax
:005EE832 EBD0            jmp 005EE804
:005EE834 8B4DFC          mov ecx, dword ptr [ebp-04]
:005EE837 81E1FF000000    and ecx, 000000FF
:005EE83D 8B55F4          mov edx, dword ptr [ebp-0C]
:005EE840 81E2FF000000    and edx, 000000FF
:005EE846 3BCA            cmp ecx, edx =====>一定要 ecx=edx 的值
:005EE848 7404            je 005EE84E ====》关键
:005EE84A 33C0            xor eax, eax
:005EE84C EB05            jmp 005EE853

* Possible Reference to String Resource ID=00001: "歃:"
                      |
:005EE84E B801000000      mov eax, 00000001
:005EE853 8BE5            mov esp, ebp
:005EE855 5D              pop ebp
:005EE856 C3              ret
```

注册吗的第一位好像没有参加比较计算！

算出注册码后还是不能用！也许还要解狗！哈哈！！！！！

我的注册码是：

`USERID:AF1VGC-PH5RFW-D57QG4-IYP8KW-SF4EG`

`PASSWORD:1AAC70`

这就是关键的地方。什么意思不用我说了吧。xixi~~~

可怜我的爱机啊！被超度了 `n(n>10)` 次。

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00404FC6(U)
|
:00404FEF 1E              push ds
:00404FF0 06              push es
:00404FF1 668CD0          mov ax, ss
:00404FF4 668ED8          mov ds, ax
:00404FF7 668EC0          mov es, ax
:00404FFA BEC84F4000      mov esi, 00404FC8
:00404FFF 8975FC          mov dword ptr [ebp-04], esi
:00405002 B9EF4F4000      mov ecx, 00404FEF
:00405007 2BCE            sub ecx, esi
:00405009 8B3D806E7100    mov edi, dword ptr [00716E80]
:0040500F F3              repz
:00405010 A4              movsb
:00405011 0F014DF4        sidt [ebp-0C]
:00405015 8B5DF6          mov ebx, dword ptr [ebp-0A]
:00405018 83C308          add ebx, 00000008
:0040501B 8B03            mov eax, dword ptr [ebx]
:0040501D A3586A7100      mov dword ptr [00716A58], eax
:00405022 8B4B04          mov ecx, dword ptr [ebx+04]
:00405025 890D5C6A7100    mov dword ptr [00716A5C], ecx
:0040502B 8B45FC          mov eax, dword ptr [ebp-04]
:0040502E 668CC9          mov cx, cs
:00405031 FA              cli
:00405032 668903          mov word ptr [ebx], ax
:00405035 66894B02        mov word ptr [ebx+02], cx
:00405039 66C7430400EE    mov [ebx+04], EE00
:0040503F C1E810          shr eax, 10
:00405042 66894306        mov word ptr [ebx+06], ax
:00405046 FB              sti
:00405047 83C310          add ebx, 00000010
:0040504A 8B03            mov eax, dword ptr [ebx]
:0040504C A3506A7100      mov dword ptr [00716A50], eax
:00405051 8B4B04          mov ecx, dword ptr [ebx+04]
:00405054 890D546A7100    mov dword ptr [00716A54], ecx
:0040505A 8B45FC          mov eax, dword ptr [ebp-04]
:0040505D 668CC9          mov cx, cs
:00405060 FA              cli
:00405061 668903          mov word ptr [ebx], ax
:00405064 66894B02        mov word ptr [ebx+02], cx
:00405068 66C7430400EE    mov [ebx+04], EE00
:0040506E C1E810          shr eax, 10
:00405071 66894306        mov word ptr [ebx+06], ax
:00405075 FB              sti
:00405076 07              pop es
:00405077 1F              pop ds
:00405078 33C0            xor eax, eax
:0040507A 5F              pop edi
:0040507B 5E              pop esi
:0040507C 5B              pop ebx
:0040507D 8BE5            mov esp, ebp
:0040507F 5D              pop ebp
:00405080 C3              ret
```

`4dd466d0e1e0cc123afbe504da2462ca`
