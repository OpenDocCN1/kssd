
# `bootkit`过64位Windows 7用户密码

> 原文：[`www.pediy.com/kssd/pediy12/129167.html`](https://www.pediy.com/kssd/pediy12/129167.html)

代码中有很多垃圾代码，大家见谅。

代码：

```
#filename bian.bat
@set path=d:\Program Files\nasm;%path%
nasm passx64win7.asm -o loader7x64.bin
```

事先安装好`nasm`编译出的`bin`作为光盘启动文件。

代码：
