

# pelock 未完全分析

> 原文：[`www.pediy.com/kssd/pediy10/87059.html`](https://www.pediy.com/kssd/pediy10/87059.html)

看到有人发这个壳  
  
我趁好在学脱壳 就看了看  
  
对于我这个刚接触壳的人来说  这个壳显得比较困难  
  
目标就是这个  
http://bbs.pediy.com/showthread.php?t=85830  
  
下载在这里  
http://bbs.pediy.com/attachment.php?attachmentid=24788&d=1239328199  
  
  
那个什么 范范 love  弄好了  叫他写教程  他也不写  
  
那我就菜鸟亲自动手吧  
  
oep  
  
这个壳 oep 还不算难  
  
就内存访问断点  或者最后一次异常后再内存断点都行.  
  
只用内存访问断的话 要 pe 段 代码段来回的段个五六次就到了  
  
到了 OEP 后  
/\*40C9A0\*/  call    0040DAE8  
/\*40C9A5\*/  jmp     00920D0D  
/\*40C9AA\*/  mov     eax, edi  
/\*40C9AC\*/  call    0040DC40  
  
明显入口被偷了  
这是堆栈  
0012FFB0   00401D60  
0012FFB4   00000060  
  
那么入口处可以给补上  
push 60  
push 00401D60  
  
补上后 OEP 为 40c999  
  
/////////////////////////////  
不过 IAT 还是加密的  
找到一个 401000  
dd 401000  
00401000  003E0000  ASCII "h",LF  
00401004  003E0033  
00401008  003E0084  
0040100C  003E010B  
00401010  003E015B  
00401014  003E01B5  
00401018  003E020B  
0040101C  003E023E  
00401020  003E026F  
00401024  003E0281  
00401028  003E02E5  
0040102C  003E031F  
  
  
确实加密了  
下个内存断点看下吧  
/\*38444E\*/  MOV DWORD PTR DS:\[ECX\],EbX  
  
在这里给填充了  
而此时的 eax 却是正确的 iat，可以改为  
MOV DWORD PTR DS:\[ECX\],EaX  
  
可这样改会有 crc 校验  我也没找到校验在哪  
  
  
就比猫画虎写了一点垃圾脚本 跑一下吧  
tmp1:  
          mov tmpreg, ebx  
          mov ebx, eax  
          sti  
          mov ebx, tmpreg  
          bprm 0038444e,1  
          cmp eax,77d3b144  
          run  
          jnz tmp1  
          mov ebx, tmpreg  
          sti  
          bprm 00384450,1  
  
我这脚本是走到 MOV DWORD PTR DS:\[ECX\],EbX 时才能运行的  
  
cmp eax,77d3b144 这里是最后一个函数的比较  
  
处理完后 IAT 全部都正常了  
  
可是还有一些 jmp 不知道怎么改  
  
/\*40C9A0\*/  CALL 1.0040DAE8  
/\*40C9A5\*/  JMP 003F0D0D   ////象这样的被替换的  
/\*40C9AA\*/  MOV EAX,EDI  
/\*40C9AC\*/  CALL 1.0040DC40  
/\*40C9B1\*/  MOV DWORD PTR SS:\[EBP-18\],ESP  
/\*40C9B4\*/  MOV ESI,ESP  
/\*40C9B6\*/  MOV DWORD PTR DS:\[ESI\],EDI  
/\*40C9B8\*/  PUSH ESI  
/\*40C9B9\*/  JMP 003F0D25    ////象这样的被替换的  
  
  
  
我就进行到这了  
那些 jmp 不知道怎么处理   也许可以用补区段来处理  
  
可我也没补过  不会补   愿哪个大大有闲工夫给我们这些刚入门的菜鸟上一课  
  
把我未完成的补充完整.