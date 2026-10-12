
# WinDbg 条件断点问题

> 原文：[`www.pediy.com/kssd/pediy11/123627.html`](https://www.pediy.com/kssd/pediy11/123627.html)

在 `WinDbg` 中这样下断点，没有提示错误，但是一执行就提示语法错误

```
bp kernel32!CreateFileA ".if($sicmp(poi(esp+4),\"E:\\its.tns\") == 0){}.else{gc;}"
```

望高手解答，先谢了

```
bp kernel32!CreateFileW "r $t1=poi(esp+4);as /mu $FileName $t1;.block{.if($sicmp(\"${$FileName}\",\"C:\\haha.txt\")==0){.echo hahaha}.else{.echo nonono}}"
```
