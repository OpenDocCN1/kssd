
# 去除广告(高手飘过)

> 原文：[`www.pediy.com/kssd/pediy11/104753.html`](https://www.pediy.com/kssd/pediy11/104753.html)

【文章标题】: 去除广告(高手飘过)  
【文章作者】: `suifengjz`  
【作者邮箱】: `75603171@qq.com`  
【作者主页】: `http://hi.baidu.com/suifengjz/`  
【作者 QQ 号】: `765603171`  
【软件大小】: `663 KB`  
【下载地址】: `http://bbs.pediy.com/showthread.php?t=102168&page=2`  
【编写语言】: `Borland Delphi 6.0 - 7.0`  
【使用工具】: `ODICY`  
【操作平台】: `xp2`  
【软件介绍】: 查询 QQ 在线状态  
【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

---

## 详细过程

**分析**

在一个进程中打开浏览器一般用 `ShellExecute` 或者 `CreateProcess`，在 `OD` 中查找参考 `ShellExecute` 然后都打算断点就跟到了 `ShellExecuteA` 是常用的打开网址的函数

`od` 载入，下断点 `bp ShellExecuteA`，`F9`

断在程序领空

```
7D610F40 >   8BFF             MOV EDI,EDI
7D610F42    55               PUSH EBP
7D610F43    8BEC             MOV EBP,ESP
7D610F45    83EC 3C          SUB ESP,3C
7D610F48    8B45 08          MOV EAX,DWORD PTR SS:[EBP+8]
7D610F4B    8365 E4 00       AND DWORD PTR SS:[EBP-1C],0
7D610F4F    8945 CC          MOV DWORD PTR SS:[EBP-34],EAX
7D610F52    8B45 0C          MOV EAX,DWORD PTR SS:[EBP+C]

0012FD0C   00474EBC  /CALL 到 ShellExecuteA 来自 小玩意.00474EB7 
0012FD10   00000000  |hWnd = NULL
0012FD14   00000000  |Operation = NULL
0012FD18   00DC58C8  |FileName = "http://bush.wfetch.com/"
0012FD1C   00404639  |Parameters = ""
0012FD20   00404639  |DefDir = ""
0012FD24   00000001  \IsShown = 1
0012FD28   0012FD34  指向下一个 SEH 记录的指针
0012FD2C   00474EC6  SE 处理器
0012FD30   0012FD64
0012FD34   0012FD6C  指向下一个 SEH 记录的指针
0012FD38   00474F09  SE 处理器
```

`Alt + F9`，返回程序领空，停在 `00474EBC` 处，向上找

```
00474E80      837D E8 00    CMP DWORD PTR SS:[EBP-18],0
00474E84      74 11         JE SHORT 小玩意.00474E97                  来到这里， 关键点修改为 jne
00474E86   .  53             PUSH EBX
00474E87   .  8B45 E8       MOV EAX,DWORD PTR SS:[EBP-18]
00474E8A   .  E8 A5F7F8FF   CALL 小玩意.00404634
00474E8F   .  50             PUSH EAX                             ; |CmdLine
00474E90   .  E8 3B19F9FF   CALL                            ; \\WinExec
00474E95   .  EB 25         JMP SHORT 小玩意.00474EBC                  这里执行才可以掉过广告，继续向上找
00474E97   >  53             PUSH EBX
00474E98   .  8B45 EC       MOV EAX,DWORD PTR SS:[EBP-14]
00474E9B   .  E8 94F7F8FF   CALL 小玩意.00404634
00474EA0   .  50             PUSH EAX
00474EA1   .  8B45 F0       MOV EAX,DWORD PTR SS:[EBP-10]
00474EA4   .  E8 8BF7F8FF   CALL 小玩意.00404634
00474EA9   .  50             PUSH EAX
00474EAA   .  8B45 F4       MOV EAX,DWORD PTR SS:[EBP-C]
00474EAD   .  E8 82F7F8FF   CALL 小玩意.00404634
00474EB2   .  50             PUSH EAX                             ; |FileName
00474EB3   .  6A 00          PUSH 0                               ; |Operation = NULL
00474EB5   .  6A 00          PUSH 0                               ; |hWnd = NULL
00474EB7   .  E8 C872FBFF   CALL                          ; \\ShellExecuteA 
00474EBC   >  33C0           XOR EAX,EAX
```

修改后，保存运行，收工。

---

【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

2010 年 01 月 08 日 0:00:54
