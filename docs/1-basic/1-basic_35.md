

# 一个汇编指令：JL，所引发的深思

> 原文：[`www.pediy.com/kssd/pediy10/84113.html`](https://www.pediy.com/kssd/pediy10/84113.html)

一个汇编指令：JL，所引发的深思  
在 RCE 论坛上看到，值得学习，所以引用到这里了！  
帖子的地址是：http://www.woodmann.com/forum/showthread.php?t=12464。  
主题全文引用如下(作者是：nezumi-lab  )：引用:  
months ago Bow Sineath (a very clever reverser!) asked me: “does JL \[jump is less\] instruction check ZF flag?” I said: “well, give me a second to think, well, it’s supposed to check it, otherwise it would act like JLE \[jump if less or equal\] and besides, JL is synonym of JNGE (jump if not great or equal), so JL should check ZF!“.  
  
but, according to Intel’ manuals JL and JNGE check only if SF != OF. CMOVL/CMOVNGE work the same way. at that time I thought that it’s just a documentation bug and even pointed this out in my presentation on HITB 2008 conference.  
  
fragment of Intel' manual  
  
  
but I was wrong!!! I have checked it and found out that JL/JNGE does not check ZF flag!!! to do this I wrote extremely simple POC (if you’re too lazy to type, download source and binary):  
  
\_\_asm  
{  
mov eax, 002C2h ; S = 1, O = 0, Z = 1  
push eax  
popfd  
jl jump\_is\_taken ; ==>  
mov p, offset noo  
jump\_is\_taken:  
}   
  
  
mov eax, 2C2h/push eax/popfd set SF with ZF and clear OF. so, SF != OF, but ZF is set. what CPU is going to do? easy to check with Olly! just load the program and start tracing. ops!!! JL is taken!!! JL ignores ZF!!! x86emu (plug-in for IDA-Pro) acts the same. didn’t check other emulators yet.  
  
well, it’s interesting. why JL (and similar commands) ignores ZF?! guess, normal CPU command (like TEST/CMP/XOR/etc) never set ZF if result is less, so JL just ignores it. but… if we set flags manually or use other tricks… it becomes a real trap!!! consider the listing above and ask your co-worker: is the jump taken or not? I’m sure, some of them will answer: of course, the jump is not going to be taken! a good anti-reversing trick!!! I just wonder - how software is still working on buggy hardware.  
   JLOlly.gif (21.6 KB)  
  
2009-3-2 20:55 三楼的回复也不错，一并引用下来（作者是：deroko ）：引用:  
supposed to check Z flag? In intel manual it says it's not supposed to check it and it's logical, it only deals with signed comparasion. You can't get S if you use cmp on 2 negative numbers which are the same, -1 for example, but you will get S flags if you compare 0FFFFFFFE(-2) and 0FFFFFFFF(-1), it's lower. also try for example this : 0FFFFFFFF (-1) compared with 1, you will get S flag as -1 is lower then 1, but CF will be cleared as in unsigned comparasion 0FFFFFFFFh is bigger then 1.   
so it's not a bug really

上传的附件
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || JZ.rar  
 |