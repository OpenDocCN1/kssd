
# Thebat!139脱壳详情及对Asprotect加壳保护的一点小结

小弟本想脱thebat141的壳的，但小弟下载的thebat141却没有加壳，没办法，还是拿thebat139开刀了。

如果小弟下面的内容有错漏的地方，请给小弟指正。

小弟的脱壳方法和冰毒的不同，他的方法我用不了，我用`Procdump32`脱不了thebat的壳。

废话少说，let's go!

## 所用工具

所用工具：`soft-ice405`、`icedump6015`、`procdump1.6.2`、`ultraedit5.0`。

首先加载`softice`，再加载`icedump`。运行`procdump32`，点击`PE Editor`，选择`thebat.exe`文件，我们要记一下数据了：`size of image:00314000` `image base:00400000`，

点击`sections`，记下`.idata`的数据：`virtual siza:00003000` `virtual offset:0022E000` ，这样我们
可以知道`.idata`在内存中的地址为：`0062E000`，长度为：`3000`

启动`Symbol Loader`，载入`thebat.exe`（加载过`icedump`后，`softice`可以成功拦截`thebat`），点击一下`Load`，马上被拦截，如下：

```
XXX:006FF001    PUSHAD  **第一个PUSHAD指令**
XXX:006FF002    CALL  006FF008  按F8进入
XXX:006FF007    NOP
XXX:006FF008    POP  EBP 按F10一直来到：
XXX:006FF0D6    CALL  006FF0DE  按F8进入
```

在这之后的一段程序要走的小心一点，我建议用F8，如果离开了thebat的就按一下F12，然后再按F8，一直来到：

```
XXX:006FFA3B    PUSHAD  **第二个PUSHAD指令**    改按F10一直来到：
XXX:006FFB6C    POPAD    **和第二个PUSHAD指令是一对**
XXX:006FFB6D    POP EBP
XXX:006FFB6E    RET  0008
```

这个`CALL`走完了，来到：

```
XXX:006FF09F    MOV  ECX，EAX  按F10来到：
XXX:006FF0D6    CALL  006FF0DE  按F8进入，然后一直按F10来到：
XXX:006FFA3B    PUSHAD  **第三个PUSHAD指令** （难道又回去了？当然不是。）按F10直到：
XXX:006FFB6C    POPAD    **和第三个PUSHAD指令是一对**
XXX:006FFB6D    POP  EBP
XXX:006FFB6E    RET  0008
```

走完这个`CALL`来到：

```
XXX:006FF1D3    PUSH  04
XXX:006FF1D5    PUSH  00001000  按F10一直到：
XXX:006FF218    CALL  006FF220  按F8进入，之后按F10一直到：
XXX:006FF2A2    CALL  006FF2AA  这里开始一直按F8，如果离开了thebat，就按一下F12，再按F8来到：
XXX:0099E001    PUSHAD  **第四个PUSHAD指令** F10一直来到：
XXX:0099E5C5    POPAD    **和第四个PUSHAD指令是一对**
XXX:0099E5C6    JNP  009E5D0
XXX:0099E5C8    MOV  EAX,00000001
XXX:0099E5CD    RET  000C
XXX:0099E5D0    PUSH  00990B60
XXX:0099E5D5    RET  终于走完了这个CALL，来到这里：
XXX:00990B60    PUSH  EBP 按F10来到：
XXX:00990B8A    CALL  00983DC4  F8进入，F10来到：
XXX:00990956    CALL  009904F4  F8进入，F10来到：
XXX:00990A11    CALL  00990110
XXX:00990A16    XOR  EAX，EAX    （这里和冰毒一样，我可省了不少时间:-)）
```

这里下： `PAGEIN D 62E000 3000 D:\IDATA.BIN` 得到完整的`.idata`部分

继续F10来到：

```
XXX:00990B43    CALL  009904C8  F8进入，来到这里：
XXX:009904D5    POPAD  **和第一个PUSHAD指令是一对**
XXX:009904D6    PUSH  EAX
XXX:009904D7    RET  （这里冰毒用Procdump32脱壳。可以吗？我怎么不行？谁可以就教教我。）
```

走完这里来到：

```
XXX:0061C528  PUSH  EBP
```

来到这里后`Soft-Ice`中显示着‘`THEBAT！CODE+0021B526`’，有‘`CODE`’的字样，我们可以知道thebat是用Broland写，成的,而且我们已经进入了主程序，可以脱壳了。

下： `PAGEIN D 400000 314000 D:\THEBAT.EXE` 得到脱壳后的`thebat.exe`文件

用`ultraedit5.0`打开`thebat.exe`及`idata.bin`，将`idata.bin`的内容替换到`thebat.exe`的`22E000--230FFF`中。

这样得到的程序还不能用，启动`Procdump32`修改文件`Sections`。

将每一项`Section`的 `Raw Size ==> Virtual Size` ； `Raw Offset ==> Virtual Offset`

将程序入口改成：`0021C528`（`61C528-400000=21C528`）

我记得冰毒说这样脱壳的程序不行，当然了！因为他漏了最重要的一步（趁冰毒不在，说了他不少坏话，大家可不要告诉他呦:-D）：

点击`Procdump32`中的 `PE Editor-->Directory`，将 `Import Table` 的值改成：`0022E000`

自此程序已脱壳成功，我们来小结一下：

小弟通过对几个用`Asprotect`加壳的软件脱壳，对`Asprotect`加的壳有的一定了解，发现了一点共有的特征，用`Asprotect`加壳的软件会出现3到4对‘`PUSHAD--POPAD`’指令（用`Aspack`或`Upx`加壳的软件只会出现一对）

在走完这几对‘`PUSHAD--POPAD`’指令后，我们便进入到主程序了，可以开始脱壳。脱壳后的文件还要还原`PE`文件头，要将完整的‘`.idata`’或‘`.rdata`’替换到脱壳后的文件中，至于到底是‘`.idata`’还是‘`.rdata`’就要大家自己去判断了，最后修改文件的`Sections`、`Entry Point`、`Import Table`各项。这样也就脱壳成功了。

baoleigz（暴雷）
