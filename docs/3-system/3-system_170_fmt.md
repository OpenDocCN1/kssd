
# 手工打造小 PE 总结

> 原文：[`www.pediy.com/kssd/pediy12/144699.html`](https://www.pediy.com/kssd/pediy12/144699.html)

## 简介

我很荣幸通过了入学考试并进入了科锐学习,目前我正在第三阶段学习 `PE` 知识点,在这里自己作一个小小总结,以备自己以后查看. 本文 `PE` 头指的是 `nt headers` 与 `section headers`;

**所需要环境:** `Redasm 2.2.1.9`, `Winhex` 即可!

## 源代码与链接选项

首先源代码如下:

```
.386

.model flat,stdcall
option casemap:none
include    windows.inc
include    user32.inc
includelib  user32.lib
.code

      szClassName       db  'CR10',0
      start:

      invoke MessageBox,NULL, offset szClassName, NULL, MB_OK
      ret

end  start
```

`Redasm` -> 工程 -> 工程选项

原配置:
`5,O,$B\\LINK.EXE /SUBSYSTEM:WINDOWS /RELEASE /VERSION:4.0 /LIBPATH:"$L" /OUT:"$5",3`

换成:
`5,O,$B\\LINK.EXE /SUBSYSTEM:WINDOWS /RELEASE /VERSION:4.0 /LIBPATH:"$L" /OUT:"$5" /merge:".rdata"=".text" /align:4  ,3`

其实多加了两个链接选项,即 `/merge:".rdata"=".text"` `/align:4`

默认情况下,如果不改链接选项的话,链接器产生两个节区,`.rdata`,与`.text.`,为了减少 `PE` 节表,所以利用 `merge` 将 `.rdata` 节合并到 `.text` 节. 而 `align` 选项是将文件对齐与内存对齐设为 `4`, 这个值对我们节区数据开始位置起到至关重要的一个值.因为文件默认对齐值是 `200`,那么我们节区必须模 `200` 地址开始,对减少 `PE` 大小干扰是可想可知的.

编译链接程序成功后,用 `winhex` 查看如下:图如下 1:

![](img/57066ace59369b36a9a1fcd3264ec90b.jpg)

## 优化 DOS 头与 PE 头

在 `DOS` 头中,其实有用的字段只有 `e_lfanew`:位于 `3C` 处,指向 `PE` 头偏移 和 `0` 偏移的 `e_magic:DOS` 标志 .其它大部份成员均没用到,那么是否可以将 `PE` 头到 `DOS` 头里来呢放到这里来,放在什么位置呢.

看下图 2:

![](img/f2113809f2d49bc06046462a50635b6a.jpg)

将 `PE` 头移动到 `DOS` 头里偏移 `4` 处,为什么是这里呢,因为对于 `DOS` 头来说,位于 `3c` 位置是 `e_lfanew` 字段,非常重要,指明了 `PE` 头偏移. 但此时的 `3C` 位置,对于 `PE` 头来说是是 `optional header` 的 `SectionAlienment` 字段,这是我们内存中对齐值.恰恰 也是 `4`.

在此图,也标明了大部份不重要字段,都用了 `CC` 填充,这为后面数据重合起到不可忽视的引导作用.

`PE` 头还有一处能减少,就是数据目录项,将默认的 `16` 项,改成了 `3` 项,影响 `optinal headers` 大小,即字段 `fileheaders.SizeOfOptinal.`,字段:`optinal headers.NumberOfRvaAndSizes` 数据目录有关,即数据目录项数,经测试这个值也不太重要,只设的不离谱.图如下 3:

![](img/af7359e1bf113aacdacbb3fed821b81e.jpg)

## 数据前移与字段重用

把 `PE` 数据目录项减少后,重要两点:`text` 节区数据,与输入表数据就要前移了,

图如下 4:

![](img/163d0af9cdd3d787fbdb6abbed9c8e1d.jpg)

为了方便 `RVA` 到 `FA` 转变,将节区内映射到 `RVA :0` 处,对应文件偏 移 `0` 处.

位于 `84` 处,是输入表项,设为 `e0` 为我们输入表数据.在这个表中 填上相对应的位置就可以了.

然后,我们看到,有大量的 `CC` 字段末使用,所以我们可以将后面的数据移到前面 有 `CC` 地方 去.

`B4` 到 `BB` 七个 `byte` 可利用.

- `B4,B5`:`text.Relocation Number` 不重要
- `B6,B7`:`text.Linenumbers Number` 不重要
- `B8-BB`:则是 `text` 的节区属性, 只有最高四位比较重要.`F` 表示可读写执行,共享.

所以我们可以把这个七个字节存导入的 `dll` 名称,即 `user32.dll` .这样当然我们放不下.所以导入 `dll` 名称,可以减短为 `user32` 即可,这个想法,源自于我记的调用 `loadlibrary` 函数时,不需要扩展名.相应的改变 输入表字段 `name(7c 处)` 值为:`B4` 指向导入的 `dll` 名称;

同样的道理,在 `PE` 头中 `C-17` 都是不要重字段,这里正好可以放下我们导入函数名 `MessageboxA`.

所以我们可以把输入表 `FirstThunk` 指向的 `IMAGE_IMPORT_BY_NAME` 项移到这里来, 因为在 `IMAGE_IMPORT_BY_NAM` 中,他导入的 `ID` 是不重要的,即他的 `ID` 可以与位于 `A` 处:`file head.NumberOfSections` 节区个数重叠. 随后 `C-17` 正好存放导入函数名 `MessageboxA`; 当然相应的改变输入表项 `FirstThunk` 指向的 `IMAGE_THNUK_DATA` 指向导入的函数名.

同样,可以把 `Messagebox` 的参数字符串,可以放到 `4e-53` 处.

当把这些工作做完后,由于我们改动了输入表 `IAT` 的位置,改动了 `Messagebox` 参数的字符串的位置,这些位置在指令都是硬性编译好的,如果直接运行,程序肯定会崩的,所以我们还需要改动指令的地址,有点类似给指令重新定位吧.这里就截个图吧.图如下 5:

![](img/0bb13e55163006724a2a6ac28aeef940.jpg)

这样我们就把所有的数据压到 `PE` 头里了,对 `BC` 后面的数据,可以全部删除掉了.我们的 `PE` 也就只有 `188` 字节大小了.

## 进一步优化节区表位置

还记的前面我说的 `file head` 中字段 `SizeofOptionalHeader.`吗?它决定了 `option header` 头大小,同时,它还有一个隐藏属性,它还决定了节区表的开始位置. 这一点对我们现在还能把 `PE` 压小,是至关重要的信息. 从上图中可以看到,如果我们能把 `.text` 节表再上移到上面大片的 `CC` 空间中,那么就能把 `PE` 打造的更小些.

那么 `.text` 节表前移到什么位置呢?

`file head` 中字段 `SizeofOptionalHeader = 18`,也就是 `.text` 节表起位置在 `34` 处. 为什么选择这个位置,原因有以下几个:

1. `text` 节名处 `34-3B`,不重要,而 `38-3B` 正是 `optional header` 头中的 `ImageBase` 字段,非常重要.所以它们重叠是合理的.
2. `3C` 处是 `option header` 中 `SectionAlignment` 内存对齐为 `4`,3C 对于 `.text` 节表中的 `Virtual Size`,对于它来说不重要.所以 它们能重叠.
3. 对于 `40` 来说是 `option header` 中 `FileAlignment` 文件对齐为 `4`,此对于 `.text` 节表来说正是起始映射的地址 `virtual addr ess`,对于它来说不重要.所以它们能重叠.

经过测试 `.text` 节表中文件大小(`raw size`),也可以全填充 `cc`.

同时为调整 `"CR10~"` 参数,与导入 `dll` 名位置.

`. 将"CR10~"字符串调整到 62-67` 位置.正好命中了 `optional header` 中 `DllCharacteristics,SizeOfstackReserve` 字段.这两个 字段恰好重叠,过了 `PE` 检查,实属运气成份!

同理导入库名 `"user32"` 一样的道理,调整到 `68-6E` 处.

这样我们还能把导入表前提到 `70`.

对于导入表来说重要的两个字段分别是

- `Name` 位于 `7C` 处.对于 `optional header` 来说是导出表目录,可随便值,他们可重叠.
- `FirstThunk` 位于 `80` 处.于 `optional header` 来说是导出表大小,一样可随便值,所以能重叠.

## 指令重组与最终压缩

当然移动了字符串,导入库名,这些都要在指令跟导入表做相应的指针变动.下面就是最重要的把指令打散分别嵌入离散的空间中去.(这里最主要的就是利用短跳 `jmp` 指令,将这些代码连接起来)

代码:

```
6A 00 6A 00 68 4E 00 40 00 6A 00 FF 15 7C 00 40 00 C3
```

反汇编如下:

```
00400065  >push    0
00400067   push    0
00400069   push    0040004E                     ; ASCII "CR10!"
0040006E   push    0
00400070   call    dword ptr [<&USER32.MessageBoxA>]     ; user32.MessageBoxA
00400076   retn
```

图如下 6:

![](img/05f188563a103f41acea320b7fce1a99.jpg)

还有一个重点:

对于导入表数据目录来说,只要留一个导入表起始位置字段即可,其它的均可删除,默认为 `0` 嘛.至此 `PE` 文件节约到 `133` 字节了!

终于勉强算是自己对 `PE` 字段一个肤浅的总结.

2011-11-29 19:29

科锐十期学生

米汤

`手工打造小 PE 样例.rar` 附件中含从大到小的 `exe` 各个大小版本.附带 `winhex pos` 文件
