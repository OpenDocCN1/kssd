
最后以一个`FFFFFF`结束. 这就不是标准型的输入表结构, 应该是以`0`分隔和结束. 也就是说当来到`OEP`时, 将`7FFFFFFF`和`FFFFFF`修改为`0`, 再`dump`就`OK`了.

### 脱壳机的编写

单步执行前两条指令, 保存`EBP`中的值作为`pIAT`, 保存`[EBX+C]`为`OEP`, 直接在`OEP`处下硬件执行断点, `RUN`! 便中断在了`OEP`, 然后修复输入表, `dump`.

不过我不想编写脱壳机, 因为那样很花费时间. 而且在我发觉脚本的强大功能时, 我就更加不想再编写代码了, 我想体验一下脚本的强大魔力. 呵呵, 我的`OD`脚本处女作. 第一次写`OD`脚本, 相当不熟练, 写出来感觉很憋足.

```assembly
//文件名:FSG 脱壳.osc
//功能:脱 FSG2.0+的壳,主要是针对输入表不能修复的情况.

var   OEP
var   pIAT

Start:
       sto
       sto
       add       ebx, c
       mov      OEP, [ebx]
       sub       ebx, c
       mov      pIAT, ebp
       bphws   OEP, "x"
       run           //goooo  to oep

//来到 oep,修复输入表
@nextdll:
       mov eax,  pIAT
       mov eax, [eax]
       cmp eax, 0
       je  @over

@nextfunc:
       cmp      [eax], 7FFFFFFF
       je   @equ_7FFFFFFF
       cmp      [eax], FFFFFFFF
       je   @equ_FFFFFFFF
       add       eax, 4
       jmp @nextfunc

@equ_7FFFFFFF:
@equ_FFFFFFFF:
       mov      [eax], 0
       add       pIAT, 8
       jmp       @nextdll

@over:
       bphwc   OEP
ret

///dpe "c:\fsg_dumped.exe",OEP
```

//本来想用脚本直接 dump 到文件中, 可是用脚本 dump 出的文件不能运行, 只好运行到 OEP 再手动 Dump 了....
//最后到达 OEP 时若代码很乱, 就取消分析.

运行脚本后, 很快就能到`OEP`了, 然后`DUMP`处程序就`OK`了. 感觉很爽. 我一连测试了`5`个被`FSG2.0+`加过壳的程序(编写语言有: 汇编, `vc`, `delphi`, `VB`), 全部脱之全部`OK`.

然而相当不满意的是脱壳之后的程序要远远大于加壳前的程序, 例如一个`asm`编写的程序在加壳前只有`2.5Kb`, 加壳之后为`893`字节, 脱壳之后居然`25KB`, 使用`LordPE.EXE`重建一下还有`9.61kb`. 这一点相当有问题.

另: 发现`FSG 2.0`存在一个`BUG`: 有一个测试程序在加密后为`6.18k`(压缩比`95%`), 的确很强啊, 不过运行加密后的程序`CPU`就一直`100%`了, 程序却运行不起来了. 脱壳后的程序也是`CPU`一直`100%`.

我把这个原程序和加壳后的程序放在附件里了, 有兴趣的好手可以分析下, 告诉俺原因啊.

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) | | `FSG 2.0+.rar` \[解压密码：`PEDIY`\] | |
|---|---|---|---|
