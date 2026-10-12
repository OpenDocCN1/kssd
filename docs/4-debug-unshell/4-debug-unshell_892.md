
# PE-Pack

> 原文：[`www.pediy.com/kssd/pediy05/pediy50224.htm`](https://www.pediy.com/kssd/pediy05/pediy50224.htm)

## PE-Pack 脱壳手记

前天在国外某网站下载了一个注册机制作工具(类似于国内的注册机生成器),某些功能很有特色,所以想汉化了自己用,用 `PEiD` 查看加的什么壳,原来是 `PE-Pack 1.0` 加壳的,当即用 `trw` 载入,跟踪到入口,`makepe`,结果不能使用,用 `ImportREC` 可以修复,却不能跨平台运行。于是在网上找了一下,找到一个叫 `Depepack` 的程序,很容易就把壳脱了,当时也没有在意,今天无意中打开《论坛精华 4》,看到 `娃娃[CCG]` 写的《`PEPack1.0` 脱壳手记》,有几点疑问:

1. 他 `makepe` 出来的文件可以跨平台!?为了验证他的话,我用了 `1.13`,`1.22`,`1.23` 的 `makepe` 还是搞不定.不知他用的是什么版本.
2. 写了个 `PROCDUMP1.62` 的 `PEPack` 脱壳脚本,但是在我这里不能用.

于是决定自己写一篇,各位高手可以对照一下

### 1. 关于程序的 OEP

娃娃写了很长的篇幅介绍如何找到 `OEP`。以下是我找 `Oep` 的总结：

```
0187:0040D000 60                  PUSHA
0187:0040D001 E800000000          CALL    0040D006
0187:0040D006 5D                  POP     EBP
0187:0040D007 83ED06              SUB     EBP,BYTE +06
0187:0040D00A 80BD3E05000001      CMP     BYTE [EBP+053E],01
0187:0040D011 0F8448020000        JZ      NEAR 0040D25F            (NO JUMP)    //直接 G 到 0040D25F 看看

....

0187:0040D25F 8B8557050000        MOV     EAX,[EBP+0557]
0187:0040D265 03858B050000        ADD     EAX,[EBP+058B]
0187:0040D26B 8944241C            MOV     [ESP+1C],EAX
0187:0040D26F 61                  POPA
0187:0040D270 FFE0                JMP     EAX                     //是不是很熟悉,直接跳到 OEP 去了

....

0187:004010CC 55                  PUSH    EBP                     //这里就是 Oep,可以 makepe 但是生成的文件却不能使用!
0187:004010CD 8BEC                MOV     EBP,ESP
0187:004010CF 83EC44             SUB     ESP,BYTE +44

.....
```

### 2. 关于如何使脱壳的文件可以跨平台运行

我用 `Notepad.exe` 试遍了 `PE-Pack` 的所有参数,可是就找不到一个可以直接 `makepe` 出来的,莫非是我功力不够,还请高手指教。

以下是 `PE-Pack` 的参数:

```
Usage:    pepack  filename  [/-,options]

~~~~~
Options:               a option can start with '/', '-' or ','
~~~~~~~  -? -h            this short help
               -o             write to output.exe
               -rs-           do not pack RESOURCES
               -ver           also pack versioninfo
               -icn           also pack ICON
               -aln           set file align to 512
               -spd           faster compression
```

没有办法,只有自己动手了,这里也有个建议,不要只看别人的经验,最好是自己动手。

运行脱壳后的程序,出现错误提示:

```
链接文件 newpe.exe

到不存在的输出 output.exe:  伞

(注:output.exe 是未脱壳的文件)
```

哈,让我们看看到底是怎么了

看看正常程序的导入表:

- `shell32.dll`
- `kernel32.dll`
- `user32.dll`
- `gdi32.dll`
- `advapi32.dll`
- `condlg32.dll`

再看看 `makepe` 出来的错误文件:

- `output.exe`    //注意这里,它里面的 `Api` 显示为?
- `shell32.dll`
- `kernel32.dll`
- `user32.dll`
- `gdi32.dll`
- `advapi32.dll`
- `condlg32.dll`

于是大胆删除了导入表里面的 `output.exe`

再运行程序,这次正常了,也可以跨平台运行了。

## 后记

如果感兴趣的朋友可以自己动手试一试,也不知道在你们那里是不是和我的一样,我用的加壳工具是 `PE-PaCK 1.0` 版及他的 `win` 界面 `sPEP10`。

本人才疏学浅,如果有什么错误和疏漏,还请大家指正.谢谢!

另:那个做注册机的程序虽然脱壳了,但是汉化却没有完成,只因为没有时间,哪位朋友有兴趣可以汉化一下,不要忘了给我一份。

这是脱壳后程序的下载地址: [`http://flyfancy.my.west163.com/pmaker.rar`](http://flyfancy.my.west163.com/pmaker.rar) (`413K`) `4dd466d0e1e0cc123afbe504da2462ca`
