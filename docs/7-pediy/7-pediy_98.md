
# Win Commander 5.1

> 原文：[`www.pediy.com/kssd/pediy05/pediy50269.htm`](https://www.pediy.com/kssd/pediy05/pediy50269.htm)

这个最好用的工具在 0828 出的新版 5.1，前天我才知道，0day 已经出来了，还是想自己还破一下~~~

还是 `aspack` 压的，`trw` 的 `pedump` 就可脱壳，文件的 `CRC` 校验很烦，没时间。事实上可以用 `smc` 搞定。

在壳中有一段 `rep movsd` 代码，向下不远是个判断是否解压完毕的语句，在这之后让他 `jmp` 到空地去(事实少的可怜，可以加一个段)，代码如下：

(机器正在疯狂下载中，没机会再改了)

```assembly
0167:005CF5D0 60           PUSHA                ；保存寄存器

0167:005CF5D1 E800000000   CALL   005CF5D6
0167:005CF5D6 5D           POP    EBP
0167:005CF5D7 89E8         MOV    EAX,EBP
0167:005CF5D9 2C36         SUB    AL,36         ；取得数据区的首址。原来是把数据放前面，代码跟在后面，这样不大好，最好顺序换一下，然后 add al, XX；另，最好是对 eax 操作，避免进位丢失。

0167:005CF5DB 89C6         MOV    ESI,EAX
0167:005CF5DD 31C9         XOR    ECX,ECX
0167:005CF5DF AC           LODSB               ；取长度
0167:005CF5E0 84C0         TEST   AL,AL
0167:005CF5E2 740D         JZ     005CF5F1      ；是否完毕

0167:005CF5E4 88C1         MOV    CL,AL         ；长度到 cl, 255 字节足够补一段了~~~
0167:005CF5E6 AD           LODSD
0167:005CF5E7 89C7         MOV    EDI,EAX       ；取目标地址
0167:005CF5E9 AC           LODSB
0167:005CF5EA AA           STOSB
0167:005CF5EB FEC9        DEC    CL
0167:005CF5ED 75FA        JNZ    005CF5E9      ；之所以没用 rep movsb 是因为非法，莫名其妙
0167:005CF5EF EBEE        JMP    SHORT 005CF5DF

0167:005CF5F1 61           POPA                ；准备返回
0167:005CF5F2 6800800000   PUSH   DWORD 8000   ；改跳转处原有语句
0167:005CF5F7 E9B2FBFFFF   JMP    005CF1AE      ；跳回去这样也可以，最好是用 jmp ebp+XXXX 这样的相对寻址
0167:005CF5FC 0000         ADD    [EAX],AL
0167:005CF5FE 0000         ADD    [EAX],AL
```

要补的数据区这样来定义，有通用性：

每个小节第一字节为长度，接下来四字节为要修改的地址，比如要改的 `004013c4` 就是 `C4 13 40 00`；再下来就是改后的内容，长度由第一字节决定，一定要数对了~~~ 然后跟着下一段，最后以 `0` 为结束标志。

大致如此，对它的 `CRC` 这样解决：

拷贝 `wincmd32.exe` 为另一份，随便起个名，比如 `WC.OLD`~~~~

破解时在 `createfilea` 前面是传参数，让 `eax` 指向你改过的路径加 `wc.old` 就行了，这样它就会去读 `wc.old` 了，呵呵，真乖给你买糖吃。接下来就随便了，改注册名什么的，只要空间够用，任你怎么作。

enjoy!

< `4dd466d0e1e0cc123afbe504da2462ca`
