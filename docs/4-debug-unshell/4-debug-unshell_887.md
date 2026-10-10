

# PELOCKnt v2.04 壳

> 原文：[`www.pediy.com/kssd/pediy01/forum055.htm`](https://www.pediy.com/kssd/pediy01/forum055.htm)

http://www2.freeshine.com/macd/jsgj/downloads/znzdat3.macd 下载后将 macd 改为 zip 530K 中的 znzmate.exe 不知是哪种软件给加的壳，太厉害了。只要加载了 ICE 或 TRW2000 就立刻当机，用 ICE 欺骗工具也不行。 

答复：

用 FileInfo 查看是用 PELOCKnt v2.04 加的密，但到 playtools 站点下载 PEUNLOCK-NT 解密却不成功。 > 卡巴杀导入表，两字节地址指向 kernel32.dll 字符串就杀，而修改这两字节指向的字符串（不是 kernel32.dll）就不杀，但是这两字节必须指向 kernel32.dll 字符串。。这种情况该怎么免杀，请高手指点。。。谢谢 

Znzmate.exe 显然是用 PELOCKnt 2.04 加密的，但是用 PEUNLOCK 解密时却提示程序不是 PELOCKnt 加密的。因为 PELOCKnt 并无更新的版本，而 UCF 出的 PEUNLOCK 是可以对付所有版本的 PELOCKnt 的。事情便有些蹊跷。PEUNLOCK 带有源程序，不妨研究一下: 

cmp word ptr \[PEheader+0f6h\], 1000h ; check if its PELOCKNT-ed, this   
jnz NoPELock ; is a propertary flag for PELOCK-NT 

好的，用 PELOCKnt 加密一个小程序，在 HEX 编辑器中观察对比 

Znzmate: 0000 0100 1010 5045 4C4F 434B 6E74 ......PELOCKnt   
正常情况: 0000 0100 0010 5045 4C4F 434B 6E74 ......PELOCKnt 

这就是程序作者的小花招。知道如何做了吗？把 1010 改成 0010，存盘。再试着用 PEUNLOCK 解密，OK！不过程序还有第二层壳，用 ASPACK 1.07b 加的壳，用 UnAspack 可以自动脱壳，但脱壳后的程序运行时却出错。好在手动脱壳也很简单。用 ProcDUMP 脱壳并修改程序入口值即可。 

\--------------------------------------------------------------------------------