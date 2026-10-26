
`0055F61F 03F8 add edi,eax`

`0055F621 8A9D 06244000 mov bl,byte ptr ss:[ebp+402406]`

`0055F627 B9 0A000000 mov ecx,0A`

`0055F62C AC lods byte ptr ds:[esi]`

`0055F62D 32C3 xor al,bl`

`0055F62F AA stos byte ptr es:[edi] ; 中断在这里`

`0055F630 ^ E2 FA loopd short UnPacked.0055F62C`

`0055F632 83EF 0A sub edi,0A`

`0055F635 57 push edi`

`0055F636 8DB5 EF1B4000 lea esi,dword ptr ss:[ebp+401BEF]`

`0055F63C 33F7 xor esi,edi`

`0055F63E 74 19 je short UnPacked.0055F659`

`0055F640 90 nop`

`0055F641 90 nop`

`0055F642 90 nop`

`0055F643 90 nop`

`0055F644 8B7424 24 mov esi,dword ptr ss:[esp+24]`

`0055F648 83EE 04 sub esi,4`

`0055F64B AD lods dword ptr ds:[esi]`

`0055F64C 81EF 16244000 sub edi,UnPacked.00402416`

`0055F652 2BFD sub edi,ebp`

`0055F654 03C7 add eax,edi`

`0055F656 8946 FC mov dword ptr ds:[esi-4],eax`

`0055F659 5F pop edi`

`0055F65A 57 push edi`

`0055F65B 33C9 xor ecx,ecx`

`0055F65D 83F9 08 cmp ecx,8`

`0055F660 74 0E je short UnPacked.0055F670`

`0055F662 90 nop`

`0055F663 90 nop`

`0055F664 90 nop`

`0055F665 90 nop`

`0055F666 8B448C 04 mov eax,dword ptr ss:[esp+ecx*4+4]`

`0055F66A 89048C mov dword ptr ss:[esp+ecx*4],eax`

`0055F66D 41 inc ecx`

`0055F66E ^ EB ED jmp short UnPacked.0055F65D`

`0055F670 893C8C mov dword ptr ss:[esp+ecx*4],edi`

`0055F673 60 pushad ; 从下面开始nop`

`0055F674 90 nop`

`0055F675 90 nop`

`0055F676 90 nop`

`0055F677 90 nop`

`0055F678 90 nop`

`0055F679 90 nop`

`0055F67A 90 nop`

`0055F67B 90 nop`

`0055F67C 90 nop`

`0055F67D 90 nop`

`0055F67E 90 nop`

`0055F67F 90 nop`

`0055F680 90 nop`

`0055F681 90 nop`

`0055F682 90 nop`

`0055F683 90 nop`

`0055F684 90 nop`

`0055F685 90 nop`

`0055F686 90 nop`

`0055F687 90 nop`

`0055F688 90 nop`

`0055F689 90 nop`

`0055F68A 90 nop`

`0055F68B 90 nop`

`0055F68C 90 nop`

`0055F68D 90 nop`

`0055F68E 90 nop`

`0055F68F 90 nop`

`0055F690 90 nop`

`0055F691 90 nop`

`0055F692 90 nop`

`0055F693 90 nop`

`0055F694 90 nop`

`0055F695 90 nop`

`0055F696 90 nop`

`0055F697 90 nop`

`0055F698 90 nop`

`0055F699 90 nop`

`0055F69A 90 nop`

`0055F69B 90 nop`

`0055F69C 90 nop`

`0055F69D 90 nop`

`0055F69E 90 nop`

`0055F69F 90 nop`

`0055F6A0 90 nop`

`0055F6A1 90 nop`

`0055F6A2 90 nop`

`0055F6A3 90 nop`

`0055F6A4 90 nop`

`0055F6A5 90 nop`

`0055F6A6 90 nop`

`0055F6A7 90 nop`

`0055F6A8 90 nop`

`0055F6A9 90 nop`

`0055F6AA 90 nop`

`0055F6AB 90 nop`

`0055F6AC 90 nop`

`0055F6AD 90 nop`

`0055F6AE 61 popad ; nop到这为止`

`0055F6AF 61 popad`

`0055F6B0 C3 retn`

```
E8 0C EF 00 00 8B 44 24 20 33 C9 8B 9C 8D 69 32 40 00 03 9D 1F FC 40 00 3B C3 74 07 90 90 90 90
41 EB E8 8D B5 49 61 40 00 B8 0A 00 00 00 F7 E1 03 F0 8D BD EF 1B 40 00 0F B6 84 0D B1 26 40 00
FE C0 88 84 0D B1 26 40 00 3C 20 EB 13 90 90 90 90 8B BD 23 FC 40 00 B8 0A 00 00 00 F7 E1 03 F8
8A 9D 06 24 40 00 B9 0A 00 00 00 AC 32 C3 AA E2 FA 83 EF 0A 57 8D B5 EF 1B 40 00 33 F7 74 19 90
90 90 90 8B 74 24 24 83 EE 04 AD 81 EF 16 24 40 00 2B FD 03 C7 89 46 FC 5F 57 33 C9 83 F9 08 74
0E 90 90 90 90 8B 44 8C 04 89 04 8C 41 EB ED 89 3C 8C 60 90 90 90 90 90 90 90 90 90 90 90 90 90
90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90 90
90 90 90 90 90 90 90 90 90 90 90 90 90 61 61 C3
```

另外还需要修改 `Code Replace Function'Start`

```
0055F416 60 pushad
0055F417 E9 A5010000 jmp UnPacked.0055F5C1 ; jmp到Code Replace的关键点
0055F41C 66:D3D3 rcl bx,cl
0055F41F 50 push eax
0055F420 E8 01000000 call UnPacked.0055F426
```

保存以上3处，程序可以运行了：）

我们还可以稍作优化，`nop`掉 `0055F41C~0055F5C0` 中的所有代码

### 五、后话

`Code Replace` 就这么处理完了！

还可以提供一个思路:不用 `ImportREC` 来修复，这样就没有最后的 `.mackt` 区段了，这个在 fly 的文章中早有介绍，

如何操作请查看 fly 的精华文章，我就不累述了。压缩包中的 `CryptoKeysU.exe` 就是没有用 `ImportREC` 来修复的文件，

文件相对小一些，而且要相对完美些。因为如果有的程序多加了很多保护的话，它会检测是否有 `.perplex`、入口是否在

`.perplex` 所在区段等等

最后用 `Resource Binder 2.3` 优化下文件吧。

另外，那位兄台有什么好的办法可以使脱壳文件不需要 `.perplex` 区段？望告之，谢谢！

--------------------------------------------------------------------------------

【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

2007年03月24日 wynney 06:46:01

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

上传的附件：

- [code replace.part1.rar](javascript:void(0);)

（976.56kb，124次下载）
