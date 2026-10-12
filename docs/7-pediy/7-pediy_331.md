
# Windows 取色对话框补煎计划 (安装程序的故事)

> 原文：[`www.pediy.com/kssd/pediy10/57428.html`](https://www.pediy.com/kssd/pediy10/57428.html)

上周搞了一个 `flexLM` 的程序, 打算发破解笔记的时候, 抓软件新版发现 `Lz0` 放了一个注册机, 还是开源的.

所以岁末第一篇只好改这个了.

话说从 `Windows 3.1` 开始我就对 `Windows` 的取色框欲求不满了, 曾经热切盼望 `2000` 和 `XP` 的取色框有所改观, 到了 `Windows 2008` 的时候我终于彻底失望了.

这个取色框的缺点就不用我一一说了吧, 很多软件里面, 提供自己的选色功能外, 都会提供自由选色功能, 绝大多数小软件都会乖乖的使用系统的这个取色框, 一个如时间机器般的莫名其妙的存在.

首先不能输入十六进色值就可以毙了他了, 更不要说只有 `HSB` 模式而且只是其中 `B` 柱条一种了, 而且这个模式下面取色区的显示还有 `bug`.

终于某一年, 心存不满的我看到了当时很绚的 `Whistler` 并且失望后, 无意中装了一个 `Photoshop`(做网页用).

当然看到鸟可耻的 `photoshop` 的取色框, 而且开始暗暗期待 `M$` 把取色框改这样了.

花开两陀, 各表一枝. 后来随着俺开始做软件, 也要用到取色功能, 自然为了向 `Adobe` 致敬, 俺克隆了一个 92% 相似度的取色框.

又后来, 随着俺发现 `LordPE` 等工具可以添加导入表, 也就开始将其制成一个 `dll`, 提供和系统取色框相同的接口, 用来替换使用了系统对话取色框的程序.

这么平安无事的过了几年, 除了 `vista` 和 `2008` 带来的失望外, 我只是偶尔在群里面上抱怨一下. 直到某天世纪的群里面有人说看雪坛子上有个 `M$` 安全聚会, 问俺要不要去演示改造取色框的故事, 这么着俺就造了一个话题, 蹭了一个名额去了.

其实之前我也有分享我的修改, 只不过多半是给做设计的朋友. 因为 `Fireworks`, `Dreamweaver` 等软件除了在固定的色票选取以外, 要自由选色, 也会跟其他小公司软件一样无畏地弹出系统的取色框, 毫不悔改, 毫不以为耻. 有鸟 `Adobe` 的取色框后, 这些一蹶不振的软件配起色来又重新焕发鸟青春, 代价就是俺每次都要拿到软件都要修改一遍, 还有就是还给别人的时候, 多了一个 `libpicker.dll`.

这里先声明一句, 此对话框完全不包括 `Adobe` 公司任何源代码, (这也要得搞得到啊).

那次的话题就是将系统的 `comdlg32.dll` 里面提供的取色框功能替代成自己的. 取代和改变系统 `API` 的功能, 俺用过的有若干方法,

其一, 各种全局挂钩的方式, 这或许是最无聊的办法了吧, 连普通用户都晓得这么干, 多半是流氓软件普及的结果吧.

比较洁净点的, 在程序目录放置空壳 `comdlg32.dll`, 其他函数全部指向 `system32` 目录下的 `comdlg32`, 仅仅实现 `ChooseColorW` 的功能. (这办法是宇宙杰出青年告诉的, 不过用户看到后还是觉得郁闷吧. )

其三, 在系统的 `comdlg32.dll` 里面插入 `libpicker.dll` 的导入项, 然后修改 `ChooseColorW` 的入口, 调用新的取色框. 这个跟前面的修改需要美化的程序没有大的区别, 同样会在系统目录留下把柄, 被人指摘.

厄, 果然最后就是比较绿色的办法, 跟系统 `dll` 进行合体, 溶解在 `comdlg32.dll` 的体内, 无声无息, 今天要表的段子就是这段大和谐了.

因为播出规制的原因, 先用一个非系统 `dll`, `MFC42u.dll` 来做演示, 好吧我马上承认了其实是上次的话题就是这个比较跟 `M$` 没有关系的 `MFC` 的运行库, `comdlg32.dll` 的修改虽然和这个一样, 但是俺没有写笔记, 预计留给明年的自动修改工具发布贴了~~

选这个 `dll` 是因为开始打算做一个合并取色框 `dll` 到 `exe` 的演示作为话题的第一部分, 结果发现这个 `exe` 本身没有调用 `Win32 API`, 是使用 `MFC` 的 `dll` 的功能, 就拿来作 `dll` 互相百合的例子了.

各位请看这个 `MFC42u.dll`

![](img/94f368f4923206d06e25409b48a5d40d.jpg)

我打算将 `libpicker` 的段拆出来, 合并到 `MFC42u.dll` 里面, `Libpicker` 的 `text` 插入到 `.data` 结尾的 `107000` 处, 资源段和重定位段进行合并, 放在两者的后面.

当然俺的 `dll` 的第一个段起始 `RVA` 并不是 `7D307000`, 这本身没法是个合法的基址, 有看官可能要说了, 设置链接器属性可以将基址和首段的起始位置结合起来得到 `7D307000`, 那前提是要合并的 `dll` 是我写得, 很多情况只是凑巧了.

不过 `dll` 身为女儿身, 最美妙的地方是带了一个重定位表, 根据这个东西我们完全可以把 `dll` 里面的段都给重定位到完全不合理的地址去.

原 `MFC` 的载入基址是 `7D200000`,

`(7D200000+107000) - (400000+1000) = 7CF06000`

俺打开 `Rejacker`, 使用固态重定位功能将文件重定位这个不伦不类的地址, 并且将 `CODE`, `DATA`, `BSS` 几个段合并到原 `dll` 中.

然后两个 `dll` 里面的各种非代码部分的段和表需要合并, 计有资源表, 导入表, 重定位表, 而 `libpicker` 的导出表就不需要了, 内部消化了.

在以前合并 `exe`/`dll` 时候, 我通常会用 `Rejacker` 导出原始程序的重定位表, 其中要合并入受方的模块使用表地址重定位功能导出, 然后粘贴在一起, 用 `ImpRec` 直接重建一个新的, 不过在这个 `MFC` 的 `dll` 面前, `ImpRec` 倒下了, 重建后的 `dll` 完全不能载入.

那好吧我只能重建 `Descriptor` 来干了. 2 个模块都各有 10 个描述项(10 个导入的 `dll`), 合并在一起就是 `10*20+10*20 = 21*20 = 420` 字节.

如果用手来量就是 `C8(@F2598) + C8(@17200) + 14(结束) = 1A4`, 尺寸是一样的.

![](img/db89421284abb6aec5caebbf4b70481d.jpg)

粘贴下来后, 后面的 `C8` 里面修正一下抠过来后导致的偏差, `106000`(107000-1000?), 这个可以用 `010` 的批量计算完成的.

每个记录中的最后的 `0190DC` 倒不需要修正.

然后找到内部使用 `ChooseColorW` 的地方, 修改为调用自身的

`7D31BD68`(合并后的 `libpicker` 新取色框功能)

代码:

```
7D24A471   .  50            PUSH EAX                     ; /pChooseColor
7D24A472   .  FF15 A817307D CALL NEAR DWORD PTR DS:\[7D3017A8\]     ; \\ChooseColorW
```

这里 `4A472+2` 处有一个重定位, 正好在重定位表里面注释掉.

最后一步需要人工操作的貌似只有合并 `OEP` 了吧, 在 `dll` 被调用的时候, 先后初始化两个 `dll`, 其中任何一个 `dll` 失败就返回失败.

写了一段代码, 用 `skypatch` 打到留出的空隙处.

```
push ebp
mov ebp,esp
push ebx
push esi
push edi
mov ebx, [ebp+10]    // lpReserved
mov esi, [ebp+C]     // fdwReason
mov edi, [ebp+8]     // hinstDLL
push ebx
push esi
push edi
call mfcoep
push eax
push ebx
push esi
push edi
call pickeroep
pop ebx
and eax, ebx
and eax,1
pop edx
pop edi
pop esi
pop ebx
leave
retn 0c
```

代码:

```
7D31FF30 >/$  55            PUSH EBP
7D31FF31  |.  8BEC          MOV EBP, ESP
7D31FF33  |.  53            PUSH EBX
7D31FF34  |.  56            PUSH ESI
7D31FF35  |.  57            PUSH EDI
7D31FF36  |.  8B5D 10       MOV EBX, DWORD PTR [EBP+10]
7D31FF39  |.  8B75 0C       MOV ESI, DWORD PTR [EBP+C]
7D31FF3C  |.  8B7D 08       MOV EDI, DWORD PTR [EBP+8]
7D31FF3F  |.  53            PUSH EBX                     ; /pReserved
7D31FF40  |.  56            PUSH ESI                     ; |CallReason
7D31FF41  |.  57            PUSH EDI                     ; |hDLLInstance
7D31FF42  |.  E8 8A54FBFF   CALL <mfcoep>                 ; \\Assumed DllEntryPoint
7D31FF47  |.  50            PUSH EAX
7D31FF48  |.  53            PUSH EBX                     ; /pReserved
7D31FF49  |.  56            PUSH ESI                     ; |CallReason
7D31FF4A  |.  57            PUSH EDI                     ; |hDLLInstance
7D31FF4B  |.  E8 2CC0FFFF   CALL <pickeroep>             ; \\Assumed DllEntryPoint
7D31FF50  |.  5B            POP EBX
7D31FF51  |.  21D8          AND EAX, EBX
7D31FF53  |.  83E0 01       AND EAX, 1
7D31FF56  |.  5F            POP EDI
7D31FF57  |.  5E            POP ESI
7D31FF58  |.  5B            POP EBX
7D31FF59  |.  C9            LEAVE
7D31FF5A  \\.  C2 0C00       RETN 0C
```

将原 `dll` 的重定位表注释掉刚才那一行, 然后拼接上 `Libpicker` 的重定位表, 用 `makereloc` 做成新的 `reloc` 段, 粘上去.

`LibPicker` 的重定位表是使用重定位重定位表导出功能导出的, 格式跟 `relox` 等工具兼容.

![](img/d034cae2b91676ae4321a7c1103d558f.jpg)

`od` 跑了一下合体后的, 爆在 `tlsalloc`, `call [tlsalloc]` 里面存放的地址并没有在载入时候被 `PE Loader` 给填成新的.

回过神看了下原来这个 `MFC42u.dll` 是做过 `bind` 的, `OrginalFirstThunk` 填 `PIMAGE_THUNK_DATA_NAMED` 名称, `FirstThunk` 填写 `IAT` 首地址.

而不使用 `bind` 的程序的每个导入项, 导入前 `FirstThunk` 填 `PIMAGE_THUNK_DATA_NAMED`, 导入后该处被占用为 `IAT`.

需要 2 倍大小的 `Picker` 的 `IAT` 作为新的 `OrginalFirstThunk` 的存放处, 原充当名称的 `FirstThunk` 安心充当 `IAT`.

于是开出 `$288` 的大小来存放 `OrginalFirstThunk`.

并且将 `OrginalFirstThunk` 的值设置为 `FirstThunk+E84 (11FF60-11F0DC)`

`Kernel32`, `OrginalFirstThunk` `11FF60`, `FirstThunk` `11F0DC`

![](img/79709f8d337f7fc8fc8fee2adcc40885.jpg)

这里我不禁要赞美一下 `010Editor`, 对于这种临时处理, 完全不用自己写工具或者脚本, 给出操作类型的长度, 给出跳过的大小(结构大小-操作数据长度), 一下就批量完毕了, 嗷嗷嗷.

这个对话框的意思就是

代码:

```
for I := 0 to Length(Descriptor) - 1 do
begin
  INC(Descriptor\[I\].OrginalfirstThunk, $E84);
end;
```

多爽啊. 记得我以前还在 `010` 里面写过脚本作类似的事情, 亏大了.

加工后的 `dll` 还被我扩大了 `Bound Import` 所在的头部, 然后用 `bind` 加工, 无事的平淡启动了.

点击选色后, 按照约定弹出了新的取色框.

合并后的导入表:

![](img/ca92b882588d51b34e9282b4c956389a.jpg)

合并后的资源:

![](img/f61bd90f473ee32e9103f5412f025775.jpg)

合并后的 `Bound Import`:

![](img/83e21e2fe8996e0534ed2394a146fec6.jpg)

最后全家福, `MFC` 和 `KOL`(一套 `API` 浅封装库)的无畏合体啊

![](img/a3a80a633bd26c9d1aa4c682f7ee36b8.jpg)

`"test" / \\ <`

占楼贴相关附件

后话: 安装程序的故事的标题是看 `Tango Icon Patcher` 时候想到的, 安装程序来提供对系统 `dll` 的处理, 并且提供 `reloader` 功能来适应 `hotfix` 和 `service pack` 的安装, 当然他修改的是资源, 用的是 `reshacker` 的脚本方式进行的替换, 而我所作的, 完全脚本化起来就要比较复杂, 只能等来年将 `Rejacker` 强健并脚本化后, 这个才能成为正标题吧.

选用的例子的测试程序, `mspaint`, 来自 `windows2000` 中文版.

附件: [`mspaint_lite.rar`](http://bbs.pediy.com/showpost.php?p=397958&postcount=2)

您可以观察 `vista` 的 `mspaint`, 只是面板位置动了一下, 实在不容易啊, 连 `2008` 中字体视图(文件夹)的安装字体还是跟取色框同期的 `3.1` 风格文件打开框呢, 特征是盘符, 路径和文件名都是单独的选择框, 啥时候去色对话框也能这么改一下呢...

从当时的对大将说得作战来说, 应该是失败了, 其中还提到 `.net` 的另外一个 `bug`, `M$` 的人问为啥不去报告, 俺就讲了以前报告 `apploc` 的 `bug` 的一次情况, `M$` 回信说 `bug` 确实存在, 不过这东西不是系统的一部分不给修.

上面的附件包含了添加重定位表和合并两种方式.

如果你跑不起来, 可能你的系统不是 `windows 2k3` 中文版, 重新 `bind` 一次好了.

按理说是 `bind` 的时候没有排除不该绑的 `dll`, 可是 `bind.exe` 的参数说明比较诡异, 排除选项用了几次不知为啥没有效果.

说起参数说明, `chkdsk` 的参数说明更诡异

  `volume Specifies the drive letter (followed by a colon),
                mount point, or volume name.`

我问了 `N` 多牛人, 每人都以为是卷名就是卷名, 可是谁都试不出来 `volume name` 怎么用, 而 `mount point` 按照 `diskpart` 里面的 `point` 弄出来也完全不行, 直到在 `Acronis Disk Director` 里面检查没有挂载的分区时候, 才抓出来他是这么的格式 `\\?\Volume{44f250d3-d206-11da-8451-00e04c03d25a}`, 这个应该是 `volume name` 了, 偶地神哪, 这是 `name` 么??

`bind` 我是执行两遍的,

`bind -o -u MFC42u.dll`

`bind -u MFC42u.dll`

不然第二次执行加入 `boundimport` 是不会提示 `binding MFC42u.dll` 的, 文件也不会更新, 为啥我也不知道, 那参数也没告诉我...

上面有使用过的 `rejacker` 几个月前传到了 `sourceforge`, 以前是私下分享代码, 干脆改了改 `bug` 开源了, 教程准备翻译一篇合并 `skin` 程序到 `unaspack` 到 `sf` 上面, 却迟迟没有动手. 果然没有好人们愿意加入这种项目啊...
