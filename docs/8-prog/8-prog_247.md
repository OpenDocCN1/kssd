
# Hide idt Hook By DrxHOOK

> 原文：[`www.pediy.com/kssd/pediy12/144052.html`](https://www.pediy.com/kssd/pediy12/144052.html)

前段时间看了 `zhouws` 的[hide idt hook src](http://bbs.pediy.com/showthread.php?t=142627),突然以前自己也瞎搞了一个隐藏 `IDT HOOK` 的玩意,差不多快烂了,于是整理一下也发布出来.  

原理:与 `zhouws` 不同, 这个主要 `Pass` 检测, `DrX Hook` + `代码流离`  

`lkd> u KiTrap01`  

`nt!KiTrap01:`  

```
8054311c 6a00            push    0  
8054311e 66c74424020000  mov     word ptr [esp+2],0  
80543125 55             push    ebp  
80543126 53             push    ebx  
80543127 56             push    esi  
80543128 57             push    edi  
80543129 0fa0           push    fs  
8054312b bb30000000     mov     ebx,30h  
```

`A` 为 `8054311e`, `B` 为 `80543125`  

首先在 `B` 处设 `Inline Hook`,这里我使用 `int 0C7` 做代理跳转到 `NewDBEntryAB`, 然后在 `A` 下读写硬件断点,注意读写跟执行硬件断点是分离  

当 `A` 处被读取的时候,`NewDBEntryAB` 会捉捕到,然后清除断点,还原 `INLINE HOOK`,接着在 `A` 处设 `INLINE HOOK`,在 `B` 处设读断点  

同理,`B` 处被读取的时候,`A` 处 `Inline HOOK`, `B` 处读断点,这样循环执行,总的来说就是跟检测代码躲猫猫.  

效果如下:  

![](img/1421e516d14225aaf12a9c47cd56a805.jpg)  

不过 `WINDBG u KiTrap01` 貌似没被 `PASS`,代码正常运行,弄了好久都搞不明白,求指教![](img/d23858dd221cce728a3a61b46badb044.jpg "恐怖!")  

![](img/99962d6341a73f761785e6bf72ffbd77.jpg)  

附上代码,环境 `VS2008`,`WinDDK 7600.16385.1`  

系统 `XP SP2`

上传的附件

| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) || `HideIDTHook.rar` |
