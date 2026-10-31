
传入命令`/bin/sh`，1个`\x00`用来截断剩下的凑数，然后传入`sigreturn_addr`，接着调用`syscall`来执行手动设置的`fake sigframe`。

结果
