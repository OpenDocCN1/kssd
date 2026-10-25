
# W32Dasm 反汇编功能增强

> 原文：[`www.pediy.com/kssd/pediy03/forum159.htm`](https://www.pediy.com/kssd/pediy03/forum159.htm)

主页：`http://personal.dfminfo.com.cn/~kuangren/index.htm`

准备工作：`SOFTICE`, `W32DASM`, `HEXEDIT` (两份，一份 `HEXEDIT1.EXE` 为原版未加密，反汇编 `OK`；二份 `HEXEDIT2.EXE` 为 `ASPROTECT` 加密，无法反汇编，原因是文件头的信息 `.TEXT` 不能被 `W32DASM` 支持). 故先必须记录下两个 `HEXEDIT` 被反汇编后在 `W32DASM` 窗口中前至少 10 行的内容 (文件头包含的信息). 首先记录下每个 `FLAG`, 开始攻击

`.TEXT` `HEXEDIT1.EXE` `FLAG=60000020`

`.TEXT` `HEXEDIT2.EXE` `FLAG=C0000040` 其他信息因为文件被加密数据不会相同. 本人没有教好的方法，采用最笨的吧，监视 `EAX`, `EBX` 等或某内存的数值为 `60000020` (`C0000040`), 若监视成功，必然会进行分别判断处理!

载入 `SOFTICE`, 启动 `W32DASM`

打开反汇编文件 `HEXEDIT2.EXE` 点 `'OK'` 前 `BPX READFILE`, 要拦好几个地方，每拦一次注意是否有 `LOOP`, `BD *` 跳出循环，再 `BE *` 以免手指会按的发抖.

```
.0045705A: E996000000          jmp       .0004570F5  -------- (1)
.0045705F: 8B83A5726000       mov       eax,[ebx][0006072A5]
.00457065: 03C7               add       eax,edi
.00457067: 50                 push      eax
.00457068: 6BD61C             imul      edx,esi,01C ;" "
.0045706B: 03D3               add       edx,ebx
.0045706D: 81C26A491700       add       edx,00017496A ;"  Ij"
.00457073: 52                 push      edx
.00457074: E8E17B0500         call      .0004AEC5A  -------- (2)
.00457079: 8B8BA5726000       mov       ecx,[ebx][0006072A5]
.0045707F: 8B443924           mov       eax,[ecx][edi][00024]
```

好长的一条龙呀! 而且下的其他断点，下次再用无效，感谢 `BPX READFILE` 每次失败后很快又到此风水宝坻.

`D DS:ECX+EDI+24` 我发现记录下来的 `FLAG` 反复在此显示，有多少个 `FALG`? 就循环多少次.

```
.00457083: 8BD6               mov       edx,esi
.00457085: C1E203             shl       edx,003 ;" "
.00457088: 2BD6               sub       edx,esi
.0045708A: 89849382491700     mov       [ebx][edx]*4[000174982],eax
```

保存所有的 `FALG` 数值到另外的地方，下次必读此内存，看来反汇编过程就在下面不远了.

```
.00457091: 8B8BA5726000       mov       ecx,[ebx][0006072A5]
.00457097: 8B443910           mov       eax,[ecx][edi][00010]
.0045709B: 8BD6               mov       edx,esi
.0045709D: C1E203             shl       edx,003 ;" "
.004570A0: 2BD6               sub       edx,esi
.004570A2: 8984937A491700     mov       [ebx][edx]*4[00017497A],eax
.004570A9: 8B8BA5726000       mov       ecx,[ebx][0006072A5]
.004570AF: 8B443914           mov       eax,[ecx][edi][00014]
.004570B3: 8BD6               mov       edx,esi
.004570B5: C1E203             shl       edx,003 ;" "
.004570B8: 2BD6               sub       edx,esi
.004570BA: 8984937E491700     mov       [ebx][edx]*4[00017497E],eax
.004570C1: 8B8BA5726000       mov       ecx,[ebx][0006072A5]
.004570C7: 8B44390C           mov       eax,[ecx][edi][0000C]
.004570CB: 8BD6               mov       edx,esi
.004570CD: C1E203             shl       edx,003 ;" "
.004570D0: 2BD6               sub       edx,esi
.004570D2: 89849372491700     mov       [ebx][edx]*4[000174972],eax
.004570D9: 8B8BA5726000       mov       ecx,[ebx][0006072A5]
.004570DF: 8B443908           mov       eax,[ecx][edi][00008]
.004570E3: 8BD6               mov       edx,esi
.004570E5: C1E203             shl       edx,003 ;" "
.004570E8: 2BD6               sub       edx,esi
.004570EA: 89849376491700     mov       [ebx][edx]*4[000174976],eax
.004570F1: 83C728             add       edi,028 ;"("
.004570F4: 46                 inc       esi
```

`ESI` 做为计数器

```
.004570F5: 0FB78B381F6F00     movzx     ecx,w,[ebx][0006F1F38]
```

`ECX` 统计有多少个 `SECTION` (为 `MM`), 每个 `SECTION` 对应一个 `FLAG` (记录没有)? `JB 45705F` 循环 `MM` 次.

```
.004570FC: 3BF1               cmp       esi,ecx
.004570FE: 0F825BFFFFFF       jb        .00045705F  -------- (1)
```

循环结束，文件头的信息全部反汇编完毕就到此了，又跳到 `45711E`.

```
.00457104: 33F6               xor       esi,esi
.00457106: EB16               jmps      .00045711E  -------- (2)
.00457108: 8BC6               mov       eax,esi
.0045710A: C1E003             shl       eax,003 ;" "
.0045710D: 2BC6               sub       eax,esi
.0045710F: F684838549170020   test      b,[ebx][eax]*4[000174985],020
```

若 `JE` 成立跳到了 `45711D`, `ESI=ESI+1`, 我发现将不能正常反汇编 (啥也没做嘛!)，

蒙蒙蒙! 我更改此处的代码后连续反编译几个被加密的软件，蛙! 一切正常，收工吃饭.

```
.00457117: 7404               je        .00045711D  -------- (1)
.00457119: 8BFE               mov       edi,esi
.0045711B: EB0C               jmps      .000457129  -------- (2)
.0045711D: 46                 inc       esi
.0045711E: 0FB783381F6F00     movzx     eax,w,[ebx][0006F1F38]
.00457125: 3BF0               cmp       esi,eax
```

`EAX` 为 `SECTION` 计数器，好象是要继续进行个 `SECTION` 的反汇编了，`JB 457108` 循环 `MM` 次.

```
.00457127: 72DF               jb        .000457108  -------- (3)
```

下面的代码难道不是在干可恶的反编译的勾当?

```
.00457129: 8BD7               mov       edx,edi
.0045712B: C1E203             shl       edx,003 ;" "
.0045712E: 2BD7               sub       edx,edi
.00457130: 8B8C937A491700     mov       ecx,[ebx][edx]*4[00017497A]
.00457137: 898BBC376F00       mov       [ebx][0006F37BC],ecx
.0045713D: 6BC71C             imul      eax,edi,01C ;" "
.00457140: 03C3               add       eax,ebx
.00457142: 056A491700         add       eax,00017496A ;"  Ij"
.00457147: 50                 push      eax
.00457148: 8D938D6A1700       lea       edx,[ebx][000176A8D]
.0045714E: 52                 push      edx
.0045714F: E8067B0500         call      .0004AEC5A  -------- (4)
```

汉语功底不好，但是修改的地方已经指出来了. 在我的主页 `HTTP://PERSONAL.DFMINFO.COM.CN/~KUANGREN` 可以下载 `W32DASM8.93` (修改了不能调试中文信息的 `BUG`, 修改了不能调试 `VB` 中的信息的 `BUG`, 修改了不能静态反汇编的 `BUG`.)   `4dd466d0e1e0cc123afbe504da2462ca`
