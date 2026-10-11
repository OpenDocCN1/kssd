
# HEXEDIT2.54 显示汉字

> 原文：[`www.pediy.com/kssd/pediy03/forum160.htm`](https://www.pediy.com/kssd/pediy03/forum160.htm)

让 `HEXEDIT2.54` 显示汉字 湖北 wuhuashang

主页：http://personal.dfminfo.com.cn/~kuangren/index.htm

某些西文软件存在不能输入汉字或显示汉字的问题，可能原因是编写软件的作者没有考虑周全，我在此介绍一种调试修改的方法，以供大家参考。

准备工作：工具：`SOFTICE`, `HACKVIEW`, `HEXEDIT2.54`。用写字板写几个字如"THISISDEMO 我在演示"，保存后注销或启动计算机。

1.  载入 `SOFTICE`，用 `HEXEDIT` 打开刚才写的文件，您发现在窗口右栏仅显示 `THISISDEMO`，汉字全部为'.'，现在的任务是要显示正确的汉字。`CTRL+D` 切入 `SOFTICE`，下断点 `BPX TEXTOUT`，退出后更改字母 `T` 为 `M`，立即被 `SOFTICE` 拦截。
2.  `BC *`，按若干次 `F12` 回到 `HEXEDIT` 的领空。
    `S DS:0 L FFFFFFFF "MHISISDEM"`，大搜捕后共有好几个地址，真正的地址是 `DS:00XXXXXX`，而非 `DS:CXXXXXXX`，也就是小的地址为 `HEXEDIT` 实际存取的地址。
3.  `BPM DS:00XXXXXX W` (不是 `R/RW`，否则一直被 `SOFTICE` 拦截无法返回)。
    退出 `SOFTICE`，再将 `M` 改为 `T`，好!`SOFTICE` 成功拦截，可以猜想，程序下一步便要读该地址的内容，然后判别其中的字符串是否在 `20H-7FH` 之间，决定是否正常显示。
    `BPE 0` 修改断点为 `BPM DS:00XXXXXX R`，退出 `SOFTICE`，立即又被 `SOFTICE` 拦截，原来是用 `MOVSB` 把 `DS:00XXXXXX` 中的字符串复制到 `DS:00YYYYYY`。
4.  `BD *`
    `BPM DS:00YYYYYY R` 退出 `SOFTICE`，立即被拦截返回 `SOFTICE` 中，好漂亮!停止在 `CS:412616`

```assembly
.004125F2: C745FC00000000             mov     d,[ebp][-0004],000000000 ;"
.004125F9: E903000000                 jmp     .000412601
.004125FE: FF45FC                     inc     d,[ebp][-0004]
.00412601: 8B4510                     mov     eax,[ebp][00010]
.00412604: 3945FC                     cmp     [ebp][-0004],eax
.00412607: 0F8D20000000               jge     .00041262D
.0041260D: 8B45FC                     mov     eax,[ebp][-0004]
.00412610: 8B4D08                     mov     ecx,[ebp][00008]
.00412613: 8A0408                     mov     al,[eax][ecx]

说明:D DS:EAX+ECX  ---文章中的字符串!
.00412616: 50                         push    eax
.00412617: E894FFFFFF                 call    .0004125B0  -------- (1)
.0041261C: 83C404                     add     esp,004 ;" "
.0041261F: 8B4DFC                     mov     ecx,[ebp][-0004]
.00412622: 8B550C                     mov     edx,[ebp][0000C]
.00412625: 880411                     mov     [ecx][edx],al

说明:AL=?,若 AL 为可以显示的字符,则经过(1)CALL 处理后还是原来的 AL
若 AL 为不可显示(如汉字)字符,则经过(1)CALL 处理后 AL=2E('.'),原来汉字变成一个'.',此 CALL 有问题!
.00412628: E9D1FFFFFF                 jmp     .0004125FE
.0041262D: E900000000                 jmp     .000412632
.00412632: 5F                         pop     edi
.00412633: 5E                         pop     esi
.00412634: 5B                         pop     ebx
.00412635: C9                         leave
.00412636: C3                         retn

看 CALL 中的代码:
.004125B0: 55                         push    ebp
.004125B1: 8BEC                       mov     ebp,esp
.004125B3: 53                         push    ebx
.004125B4: 56                         push    esi
.004125B5: 57                         push    edi
.004125B6: 0FBE4508                   movsx   eax,b,[ebp][00008]
.004125BA: 83F820                     cmp     eax,020 ;" "
.004125BD: 0F8C1A000000               jl      .0004125DD  -------- (1)

比 20H 小,AL=2E ,也就是一个'.'
.004125C3: 0FBE4508                   movsx   eax,b,[ebp][00008]
.004125C7: 83F87E                     cmp     eax,07E ;"~"
.004125CA: 0F8F0D000000               jg      .0004125DD  -------- (2)

比 7EH 大,AL=2E ,也就是一个'.'
.004125D0: 8A4508                     mov     al,[ebp][00008]
.004125D3: E90C000000                 jmp     .0004125E4
.004125D8: E907000000                 jmp     .0004125E4
.004125DD: B02E                       mov     al,02E ;"."

你在此处想改头换面?
.004125DF: E900000000                 jmp     .0004125E4
.004125E4: 5F                         pop     edi
.004125E5: 5E                         pop     esi
.004125E6: 5B                         pop     ebx
.004125E7: C9                         leave
.004125E8: C3                         retn
.004125E9: 55                         push    ebp
.004125EA: 8BEC                       mov     ebp,esp
```

在 `HACKVIEW` 中改 `CS:4125BA` 之间的代码为

`CMP EAX,00`
`JL 4125DF`

或用 `HEXEDIT` 修改 `0F8C1A000000` 为 `0F8C1C000000`

保存退出。正是:汉字西文均显示,古怪字符变成'.'

版权所有，请不要修改原内容   4dd466d0e1e0cc123afbe504da2462ca
