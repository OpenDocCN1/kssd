
# delphi 字符串参考脚本

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-146.htm`](https://www.pediy.com/kssd/pediy09/pediy09-146.htm)

> `OD` 自身以及 `ULTRA STRING PLUGIN` 都不能正确获取 delphi 程序的字符串
>
> 给严重依赖字符串参考的菜鸟们带来莫大的痛苦
>
> 下面是我今天破解过程中一个副产品
>
> 首次写 `ODBGScript` 脚本，在经历期间一些摸索后，处女作出来了：

代码:

```
var addr
var str0
var slen
var numb
var logfile

mov addr,00401000
mov numb,0

cmp $VERSION,"1.53"
jb odbgver

mov logfile, "g:\\download\\s.log"
eval "\\r\\n 解析进程中 delphi 字符串脚本 by 快雪时晴\\r\\n\\r\\n"
wrt logfile, $RESULT

loop_0:

find addr,#FFFFFFFF#
cmp $RESULT,0
je exit_0
mov addr,$RESULT
add addr,4
mov slen,\[addr\]

//太长串了，不大可能，或许 00 到了空白区
cmp slen,ff
ja next_0
//太短串了，也没多大意义
cmp slen,5
jb next_0

//string NULL,ONLY 1 BYTE
mov str0,\[addr+slen+4\]
and str0,000000FF
cmp str0,00
jne next_0

add addr,4
inc numb
readstr \[addr\],slen
eval "#{numb}# \[{addr}\]"+$RESULT
//log $RESULT
wrta logfile,$RESULT
next_0:
jmp loop_0

odbgver:
msg "ODBGScript 版本号应大于 v1.53（ReadStr）"

exit_0:
eval "共找到{numb}个 Delphi 字符串参考." 
msg $RESULT

ret
```

我的结果：

引用: `#1# [40E090]C++ Exception`
`#2# [40EBF8]m/d/yy`
`#3# [40EC08]mmmm d, yyyy`
`#4# [40EC50] AMPM`
`#5# [40EC60]AMPM`
`#6# [40EC7C]:mm:ss`
......
......
`#BC2# [76DA90]#Reminder title`
`#BC3# [76DAAC]%DAYS`
`#BC4# [76DCD8]#DAYS USED`
`#BC5# [76DD08]#Trial Expired`
`#BC6# [76DD20]%MAXDAYS`
`#BC7# [76DD40]#Days Used`
`#BC8# [76DE34]##Info Trial Expired`
.......

有用就拿去![](img/c367a558167d303a470cbda3563c57f3.jpg "使眼色")

鼓掌！

4dd466d0e1e0cc123afbe504da2462ca
