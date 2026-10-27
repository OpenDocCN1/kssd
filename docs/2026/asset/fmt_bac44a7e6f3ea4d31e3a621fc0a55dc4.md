
`003710D6` `popad`

`003710D7` `mov dword ptr ss:[esp+1C],edi`

`003710DB` `popad`

`003710DC` `jmp eax`  // 前往 `OEP` 处理的花样 `00AA0000`

`003710DE` `popad`

`003710DF` `retn`

// 里面有很多花花，可以用 `Esp` 定律找 `Stolen Code`，也可以根据堆栈 “我猜我猜我猜猜猜”

```
00AA0000   push 401CDD //★★★
00AA0005   neg esi
00AA0008   cmp edi,edi
00AA000A   shld edi,eax,cl
00AA08E9   push dword ptr fs:[0]  //★★★
00AA08EF   mov esi,AB3919AB
00AA08F5   inc esi
00AA08F7   ror esi,3E
00AA11E6   mov dword ptr fs:[0],esp  //★★★
00AA11EC   not esi
00AA11EE   test esi,8BBE848D
00AA11F4   imul edi,esi,1DC84EAC
00AA1AEB   mov ecx,edi
00AA1AED   and edi,FFFF0000
00AA1AF3   sub ecx,edi
00AA1AF5   rep stos byte ptr es:[edi] // 移“花”植“草”
00AA1AF7   popad
00AA1AF8   popfd
00AA1AF9   push 401011               // 压入伪 OEP
00AA1AFE   retn
```

// 当 `eip` 等于 `401011` 时堆栈的情况，明显是安装了 `seh` `401CDD`

```
0012FFBC    0012FFE0   指针到下一个 SEH 记录
0012FFC0    00401CDD   SE 句柄
0012FFC4    77E5EB69   返回到 kernel32.77E5EB69
```

// `OEP`定为 `401000` 吧，上面找到的长度要大，套不了，作如下修复刚好用完 `00` 字节

```
00401000    >push 401CDD
00401005    >xor eax,eax
00401007    >push dword ptr fs:[eax]
0040100A    >mov dword ptr fs:[0],esp
```

哈哈，这次脱壳后又使我再次从壳盲的人群中挤了出来！

另外，我很想知道很多早期的壳在`98`下检测出`OD`调试器来，是怎么做的？

```
00401C05   and eax,FFFF0000
00401C0A   mov dword ptr ss:[ebp+4097DF],eax
00401C10   mov eax,dword ptr ss:[ebp+407889]
00401C16   and eax,FFFF0000
00401C1B   cmp word ptr ds:[eax],5A4D         //到达这里的指令， 很明显是只适用 NT 的获取模块基址的方式，好像不适用 98
00401C20   je short 00401C29                                  ; ASLOAD.00401C29
00401C22   sub eax,10000
00401C27   jmp short 00401C1B                                 ; ASLOAD.00401C1B
00401C29   mov dword ptr ss:[ebp+409543],eax
00401C2F   mov dword ptr ss:[ebp+40952B],eax
```

Written by askformore

06.01.2005

Andy Law Advertising: 想要...就系你既...![](https://bbs.kanxue.com/view/img/face/27.gif)

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
