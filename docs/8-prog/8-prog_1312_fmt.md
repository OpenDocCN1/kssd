
# `Ring3` 下设置文件安全权限来简单保护文件防删\[有码\]～～～

> 原文：[`www.pediy.com/kssd/pediy11/113744.html`](https://www.pediy.com/kssd/pediy11/113744.html)

思路就是把要保护的文件夹设置成不允许删除和不允许删除子目录和文件，详细设置方法请看附件里的代码和注释。在 `XP`/`Vista`/`Windows7` 下面测试有效，可以阻止 `DeleteFile`，`ZwDeleteFile` 的函数的删除~~~

![](img/12bc272499adca74223dbb65a87f60ea.jpg)

上传的附件

| ![](img/1baac1a3eff5ac15dae6f33dccc134d9.jpg) || `FileProtect.zip`
|
