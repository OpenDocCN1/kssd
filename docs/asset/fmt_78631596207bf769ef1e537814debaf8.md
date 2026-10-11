
# 简单的堆栈平衡原理脚本

> 原文：[`www.pediy.com/kssd/pediy10/93100.html`](https://www.pediy.com/kssd/pediy10/93100.html)

初学脚本，省下反复设置删除硬件断点的麻烦了，为自己增加一贴![](img/e71d9210a215d5b66755d038631e0c83.jpg "恐怖!")

代码:

```
VAR temp
STO
EOB break
BPHWS esp,"r"
mov temp,esp
run
break:
BPHWC temp
log eip
CMT eip,"Success!I must quit now!"
ret
```

下载:

上传的附件

| ![](img/0f24970173caf34cf302ee22e88648bc.jpg) || `esp` 定律.txt |
