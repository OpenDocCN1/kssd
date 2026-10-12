
# 全球首发免疫鬼影病毒批处理

> 原文：[`www.pediy.com/kssd/pediy11/109264.html`](https://www.pediy.com/kssd/pediy11/109264.html)

官方网站：`http://www.nb12345.com/post/8.html`

```batch
del /f /q "c:\windows\system32\DRIVERS\atixx.sys"

md "c:\windows\system32\DRIVERS\atixx.sys"

echo y|cacls "c:\windows\system32\DRIVERS\atixx.sys" /t /c /d ""

del /f /q "c:\windows\system32\DRIVERS\atixi.sys"

md "c:\windows\system32\DRIVERS\atixi.sys"

echo y|cacls "c:\windows\system32\DRIVERS\atixi.sys" /t /c /d ""

md "c:\Program Files\MSDN"

echo y|cacls "c:\Program Files\MSDN" /t /c /d ""
```

保存以上代码为：`KGY.bat`

双击打开执行！

如果是网吧用户，请添加至开机批处理执行通道里面！
