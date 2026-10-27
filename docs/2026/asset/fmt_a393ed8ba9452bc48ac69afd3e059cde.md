
``` 
;;;;;;;;;;;;;;IF;;;;;;;;;;;;;;;;;;

004B882C    81F7 26058919   xor edi,0x19890526                       ; API地址加密

NOP掉，edi里面是真实api

004B885E    893488          mov dword ptr ds:[eax+ecx*4],esi         ; 写入第一段shellcode的入口地址

直接存储真实api地址，所以改为：mov dword ptr ds:[eax+ecx*4],edi

;;;;;;;;;;;;;ELSE;;;;;;;;;;;;;;;;;

004B8965    81F7 26058919   xor edi,0x19890526                       ; edi为api地址

NOP掉，edi里面是真实api

004B899D    893488          mov dword ptr ds:[eax+ecx*4],esi          ; 将第一段shellcode入口地址存到00020000段

直接存储真实api地址，所以改为：mov dword ptr ds:[eax+ecx*4],edi
```

17. 然后入口点改为`000442A2H`，RVA改为`000AD000H`，大小为`1000`，获取导入表后把无效的删除，dump下来得到`dump_.exe`。

![](https://bbs.kanxue.com/upload/attach/201907/860174_WDYU46K69F86TRE.png)

18. 用IDA再次查看import，可以看到`177`个，和第6步的数量刚好对应。

![](https://bbs.kanxue.com/upload/attach/201907/860174_DKMSPJWGXE4FG5F.png)

19. 但是很遗憾，不知道为什么运行不了，可能功力不够，后劲不足。运行后只有声音，没有界面。使用PEID查壳发现区段多了一个`.mackt`段，这是impREC之后自动生成的。

![](https://bbs.kanxue.com/upload/attach/201907/860174_DSUFU463Y8EQSX4.png)

# 小结

这个程序的壳部分使用了`SMC`技术，可通过一本《加密与解密 第四版》系统学习脱壳技术和`SMC`以及编写`shellcode`代码，让自己的内力犹如九阳神功，后劲充足。

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

上传的附件：

- [九重妖塔.zip](javascript:void(0);)

  （590.42kb，39次下载）
