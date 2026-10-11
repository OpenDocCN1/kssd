
# PhotoShop 窗口置顶制作过程

> 原文：[`www.pediy.com/kssd/pediy06/pediy6898.htm`](https://www.pediy.com/kssd/pediy06/pediy6898.htm)

PhotoShop 窗口置顶补丁制作

作者：nbw

HP：`www.vxer.com`

GB：`[NE365][FCG][BCG][DFCG]`

`www.fcgchina.com` 欢迎您

有朋友说想让 PhotoShop 窗口有置顶功能，便于平时边看教程边练习。那么今天就做作看。我这里采用的是 PhotoShop 6.0.1 版本，没有装 Adobe Online 软件（这一点很重要），是中文版（或者汉化），还有，好像是 DB。

制作目标：

点击帮助菜单下面的新闻查看选项，改变窗口置顶属性（不明白？晕倒….）

## 一、剩余空间

这次首先说说剩余空间（这个最好说）。用我写的“剩余空间查看器”分析一下 `Photoshp.exe`。结果如下：

```
名称      RVA        OA         尺寸 D    可写否

.text   009eb752   009eb752    2222      否

.rdata  00b896fe   00b896fe    2306      否

.data   00cc8e90   00cc8e90  -585360    可

.rsrc   00cefb40   00c60b40    1216      否

.reloc  00dc1c5a   00d32c5a     934      否
```

有效剩余空间(字节 D)为: 6678

我就采用`.text` 区段的剩余空间，由于某些原因，这个区段并没有想象中好用，下面具体介绍。

## 二、获取主窗口句柄

由于 `SetWindowPos` 函数需要主窗口句柄作为参数，所以这里需要找到这个句柄，保存起来，共参数调用。

创建窗口一般用 `Createwindoexa` 函数.这个函数的返回值是被创建的窗口的句柄.故此,下断点 `bpx createwindowexa`.载入 `photoshop`,被数次中断.说明程序创建了不少子窗口.但是哪个是主窗口呢?第一个么?一般非也.

我的做法是,每次中断后,F12 执行完该函数,然后记下返回值(`eax`),这样会记下一堆句柄.然后程序完全加载后,察看窗口句柄,然后和刚才记录的那一堆比较.这样就知道哪次中断是主窗口的中断了.比如,我的记录如下:

`53C 544 54C 550` (再往后不用记录了,因为窗口已经出现了)

程序启动后,用窗口句柄察看器察看主窗口句柄,我发现工作区处的句柄为 `54C`,想当然地以为第三次中断便是创建主窗口.以至于做到最后发现句柄没有找到.郁闷...

我刚才找的 `54C` 只不过是 `photoshop` 的工作区,看起来比较"主"罢了.真正的主窗口是外面的框架所在的窗口.其实当时我也意识到了,只不过....

这样一来,就需要手动查找窗体的加载函数了.我依然利用断点 `bpx createwindoexa`,当被中断 4 次(或者 5 次吧?)后,主窗口显现.重新启动,当中断 3 次后,就 `F10` 单步运行.当经过 `0167:A9AE0B call A690D0` 的时候主窗口显现,说明这个函数用于加载窗口.其返回值明显不是窗口句柄.不妨再次启动,追进去看看,不久就发现了 `0167:A691BF call createwindoexa`.这个函数的返回值我记录下来,程序启动后察看,果然就是主窗口句柄.原来 `0167:A691BF` 处的 `call` 才是主窗口的创建函数.

不过令我郁闷的是开始下的中断 `bpx createwindowexa` 竟然拦截不到此处.我也不明白为什么.或者 `ps` 有反调试功能?!

### 变量处理

这里所谓的变量,就是说具有可写性质的的内存地址。但是我发现`.text` 区段虽然有大量剩余空间，但是不具有动态可写性，即使修改`.text` 区段的属性也无法得以实现。经过试验`.rdata` 区段具有可写性,不过事先必须用 `Pedit` 把该区段属性设置为可写。其剩余空间如下所示：

`.rdata 00b896fe 00b896fe 2306 否（修改为可写）`

确定保存窗口句柄的地址为：

`00b89700(OA)==00F89700(VA)`

`A691BF(VA) call ***** ;返回窗体 hwnd`

此处文件地址为:`6691BF H`

原来为:

```
6691BF   :       call       createwindoexa
6691C5   :       test       eax,eax
6691C7   :       mov       [esi+4],eax
6691CA   :       jne       6691D9
```

修改为:

```
6691C5 :       jmp       9EB8B0
```

覆盖了:

```
6691C5   :       test       eax,eax
6691C7   :       mov       [esi+4],eax
```

记下来,以后补上

新添加的代码：

```
9EB8B0(OA)   :       nop
                nop
                mov dword [00F89700],eax
                test       eax,eax
                mov       [esi+4],eax
                jmp       6691CA
```

## 三、菜单消息处理

当用户点击“帮助\\Adobe 新闻...”的时候，窗体的置顶属性发生变化。所以需要找到这个菜单的消息处理函数，我这里没有装 Adobe Online 软件，所以每次选中这个菜单，都会提示安装 Adobe Online，点其他几个菜单（常见问题，可下载内容等等）也会如此。所以这里比较麻烦，需要判断每个菜单。由于篇幅原因（不要砸鸡蛋阿...），这里就不说具体的寻找方法了，哪位不清楚咱们再细谈。具体结果如下：

`0167:00A697CE call ***** ;关键.如果此时 esi=FAC,则为“Adobe 新闻菜单”，所以需要在这里跳转。`

### 剩余空间

继续采用上面所述,文件地址 `oa:9EB900(va=DEB900)`

### 菜单处理位置

原来为:

`0167:00A697CD(VA)== 6697CD(OA)`

```
0167:00A697CD       56           push       esi
0167:00A697CE       E88D710100     call       .000680960
0167:00A697D3       83C404          add       esp,004
0167:00A697D6       33C0             xor       eax,eax
```

修改为:

```
0167:00A697CD       56           push       esi
                 jmp       9EB900
```

覆盖了:

```
0167:00A697CE       E88D710100     call       .000680960
```

记下来,以后补上

### 变量处理

`F89710` 处(`OA=B89710`)设定为标记,判断是否进行置顶。此标记也是 `SetWindowPos` 函数用到的参数。这里这个参数只允许是`-1` 或者`-2`，并且事先静态配置好。我这里配置为`-2`。标记处理流程如下：

```
if  dword ptr [F89710]=-1  then    ;窗口处于顶层
   [F89710]=-2
else  ;窗口一般状态
   [F89710]=-1
endif
```

### 高级语言演示

请注意下面的参数 `HWND_NOTOPMOST` 和 `HWND_TOPMOST` 就是我们刚才设定的标记

```
'*****窗体一般状态
Const SWP_NOMOVE = 2
Const SWP_NOSIZE = 1
Const HWND_NOTOPMOST = -2
SetWindowPos Form1.hwnd, HWND_NOTOPMOST, 0, 0, 0, 0, SWP_NOMOVE Or SWP_NOSIZE
'*****
'*****窗体最上
Const SWP_NOMOVE = 2
Const SWP_NOSIZE = 1
Const HWND_TOPMOST = -1
SetWindowPos Form1.hwnd, HWND_TOPMOST, 0, 0, 0, 0, SWP_NOMOVE Or SWP_NOSIZE
'*****
```

### 新添加的代码

`9EB900(oa)==DEB900(va)`

```
0167:DEB900:        cmp       esi,FAC
                jnz   _exit           ;不等,则跳走,不处理
                pushad
                mov       eax,dword [F89710] ;获取标志位
                cmp eax,-1                 ;ONTOP
                jz       _TONOR
                inc   eax              ;mov eax,-1
                mov dword [F89710],eax       ;save it
                jmp       _SetWin
   _TOPNOR(9EB920):
                dec  eax              ;mov eax,-2
                mov dword [F89710],eax       ;save it
   _SetWin(9EB927):
                push       3
                push       000
                push       000
                push       000
                push       000
                push       eax
                push       dword [F89700]        ;push       hwnd
                mov       eax,DEC8F0
                call  dword [eax]
                popad
                jmp       6697D3           ;下面的 call       .000680960 只不过用来
                                             调出他们的新闻，所以我们不需要用，直接回去就可以了
   _exit(9EB946):
                call       .000680960
                jmp       6697D3
```

到此基本完工，不是之处多原谅，有问题请再说……

4dd466d0e1e0cc123afbe504da2462ca
