
# 转帖【梦醒时分t0db.myetang.com】翻译的：【破解Autodesk AUTOCAD 14 狗加密】 (15千字)

怎样破解Autodesk AUTOCAD 14 法语版的狗加密

--------------------------------------------------------------------------------

translate by 梦醒时分

yinqun2000@263.net

http://t0db.myetang.com

级别:中级-高级

所需工具:

`Soft ice 3.x`

你自己最喜欢的 编辑器 (我一直用 `Hexworkshop`)

汇编知识 (至少是基础)

`Wdasm89` (并不特别需要)

大脑 (你有一个 , 是吗 ? :p )

一些啤酒 :O ( 因为破解这个软件是要很长时间的!)

一些音乐 ;)

历史:

我是很久以前破解这个软件的... ;O

它对我来说很难,而且浪费了我几天时间.最难的部分就是找到检查狗的地方而且我不是用下面我教你的方法做的!那是一种很复杂的方法...我很蠢的破解了它,而且还常常崩溃!几天前,我完成了另一个注册机时觉得很烦.所以,我想到除了注册机的破解.那么那个狗保护的Autocad 14?!啊!那是狗保护!我讨厌狗,而且在这方面几乎没有什么经验...

狗保护是最贵的保护方法,这个软件的售价是$7000左右!(够开一个网吧的了)

让我们来破解它吧:)

BTW,我用好方法破解它大概用了 2 小时 (包括 反汇编: 最长的部分 :p) 这期间没有死机 ;)

## 1) 运行软件

好的.. 让我们来看看这个目标,我安装这个软件非常慢.. 该死的光盘!假设它非常快吧!安装好后,找到安装目录.有一个 `Acad.exe` 的可执行文件和其它很多dll文件...我运行这个软件,跳出一个对话框:

"ERREUR FATALE : Le system de securit?verrouillage materiel..."

这是法语, 意思是:

"FATAL ERROR : security system is missing! blablabla"

好的,它就是告诉我们最重要的硬件(\*狗\*)丢失!哈哈哈

那么... 让我们更深层的看看 ;) 我使用`softice`设置了在 `LPT1`上最常用的断点: (`BPIO -H 378 R`).

我们停在了这儿:

```
0028:CE5AA885  88442405          MOV
[ESP+05],AL      ; 这儿.
0028:CE5AA889  66C746680100      MOV    WORD PTR
[ESI+68],0001
0028:CE5AA88F  8A442405          MOV
AL,[ESP+05]
0028:CE5AA893  884615            MOV
  [ESI+15],AL
0028:CE5AA896  884614            MOV
  [ESI+14],AL
0028:CE5AA899  5E
POP    ESI
0028:CE5AA89A  83C404            ADD
  ESP,04
0028:CE5AA89D  C3
RET
```

我们是在 `chat_to_dongle` routine. 而且如果我们看下面的data部分,我们看到:`SENTINELXXX`...

这是什么啊?! 你应该知道有什么公司提供这种加狗的服务如: `Hasp`, `Sentinel`, `DesKEY`, `Activator/Unikey` 还有许多其它的.

因此,我们可以推断碰到的是一个 `SENTINEL` 狗加密!我们现在在 `SENTINEL.VXD` 文件... 回到文章中来,我们停在这儿:

```
0028:CE5AA885  88442405            MOV
  [ESP+05],AL  ;  这里..
0028:CE5AA889  66C746680100        MOV
WORD PTR [ESI+68],0001
0028:CE5AA88F  8A442405            MOV
  AL,[ESP+05]
0028:CE5AA893  884615              MOV
  [ESI+15],AL
0028:CE5AA896  884614              MOV
  [ESI+14],AL
```

我们要看是关于可执行文件的`CALL`,所以我们要按三四次`F12`,我们会在这儿:

```
0028:CE5B35D6  50            PUSH
EAX
0028:CE5B35D7  55            PUSH
EBP
0028:CE5B35D8  57            PUSH
EDI
0028:CE5B35D9  E8D2FEFFFF    CALL    CE5B34B0
  ; 我们的 call :)
0028:CE5B35DE  C06C241F01    SHR    BYTE PTR [ESP+1F],01
;回到这儿.
```

在 `CALL` 中:

```
0028:CE5B35D6  50
  PUSH    EAX
0028:CE5B35D7  55
  PUSH    EBP
...
```

Ok,看看下面的代码并不能帮助我们什么 :( 我看过几篇关于狗加密的教程.我们做的这几步,应该是已经完成了.我想,在`CALL`后面,会有一个关于狗的 `CMP` ,如果值是和狗相同,那么就通过,否则就会加上一个错误的旗标...... 但是,我们是不幸运的,这里不是那么简单!的.通过和代码游戏(跟踪`call`,寻找一些比较的代码),我按很多次 `F12` ,但是找不到一点好东西!

BTW,我无从这该死的`Sentinel`的`VXD`中出来!按住 `F12`,可是什么都没有变!`VDX`使我陷入困境了......我希望回到 `Acad.exe` 文件中!我不希望破解狗硬件,大多数的时候,狗保护的弱点一般都是在目标本身!

我试了 `BPIO -h 378 R`

让我们在 I/O-port 上试试其它的断点 :

`378`  已经试过了

`3BC`  这个 ;)

`278`  这个也是 ;o)

OK,运行软件,但是却没有拦截 ;( 我的天啊!

因为它是一个 `VDX` ,所以我也试了 `bpx CreateFileA` ,但是走不了多久......

那么,为什么不反汇编它呢?运气好说不定能找到一些在字符串中找到一些关于出错的信息(不要做梦了,那是不可能有的!!:p)现在你能用工具中的啤酒了 ;) 因为我们要花很长时间去反汇编!!!! 那就是我为什么不用`IDA`的原因了! BTW, 我们的目标可执行文件大概是 7.24 mb!好了,现在你知道为什么要花这么多时间了吧..

喝过酒后,一觉醒来.

经过这么长的时间,终于反汇编好了! 首先,保存结果!我们不希望再等那么多时间,不是吗?如果你的电脑死机或者其它原因,你只有再等那么多时间了!再喝一次酒,呵呵!

完了吗? Ok!

我们愚蠢的想法是去找字符串,运气不佳,没有,其实是意料之中 :p 那么让我们想想!

我们怎样攻击它呢? 我们看看 Import 表吧!看看狗到底调用了什么函数 :) 我们可以找到一个 "l33t0" API 函数!哈哈,我想会有一些好东西了, 我们来看看内核......

太太太好了 :) 我找到一些有趣的东西了:

`DeviceIoControl`

从没有看过,但是名称,听起来非常好,不是吗?

呵呵!去除所有的断点只在`softice`中输入: `'bc \*'` 和: `'BPX DeviceIocontrol'`

现在运行我们的软件! 哈哈,它拦截下来了 :) 按`F12`,我们跳出了`dll`文件来到了`ACAD.exe` :) 让我们和代码玩耍吧.......

先看代码, 我按`F12`几次直到在一个好位置.下面是来自我的`winice.log` 的一部分:

```
:bl
00) BPX KERNEL32!DeviceIoControl  // Damn good :)
Break due to BPX KERNEL32!DeviceIoControl
Break due to BPX KERNEL32!DeviceIoControl
Break due to BPX KERNEL32!DeviceIoControl
Break due to BPX KERNEL32!DeviceIoControl
Break due to BPX KERNEL32!DeviceIoControl
Break due to BPX KERNEL32!DeviceIoControl
// 你在这可以看到我在找到好地方之前被拦截下来多少次
// 你用这个 bpx 的时候,你按 F5 3 次,一会有拦截下来,
// 你再按两次后停在了这儿 :
```
