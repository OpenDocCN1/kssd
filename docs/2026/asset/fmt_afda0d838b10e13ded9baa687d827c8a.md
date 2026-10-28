
# 补丁代码分析

```
0105B065    8BF8            MOV EDI,EAX                        ；把BKillProcess函数的地址保存到EDI
0105B067    85FF            TEST EDI,EDI                       ；判断EDI是否为0
0105B069    75 06           JNZ SHORT DIY任务?0105B071         ；不为0就跳到0105B071继续执行
0105B06B    61              POPAD                        ；如果为0，就说明获得BKillProcess函数的地址，恢复环境，
0105B06C  - E9 240EFBFF     JMP DIY任务?0100BE95               ；恢复环境后，跳到默认消息处理流程，继续执行
0105B071    58              POP EAX                            ；把“PID”弹出栈
0105B072    60              PUSHAD                             ；保存当前的环境
0105B073    90              NOP
0105B074    90              NOP
0105B075    83F8 04         CMP EAX,4                          ；判断是不是SYSTEM.EXE进程
0105B078   /0F84 81000000   JE DIY任务?0105B0FF                ；是则转恢复现场
0105B07E    50              PUSH EAX                           ；ProcessId进栈
0105B07F    6A 00           PUSH 0                             ；|Inheritable = FALSE
0105B081    68 10040000     PUSH 410                           ； |Access = VM_READ|QUERY_INFORMATION
0105B086    FF15 14110001   CALL DWORD PTR DS:[<&kernel32.OpenProces>; kernel32.OpenProcess
0105B08C    8985 00FAFFFF   MOV DWORD PTR SS:[EBP-600],EAX     ；将OpenProcess打开的句柄保存
0105B092    83A5 F8F9FFFF 0>AND DWORD PTR SS:[EBP-608],0       ；将字符串保存地址清零
0105B099    85C0            TEST EAX,EAX                       ；判断EAX是否为0即OpenProcess是否成功
0105B09B    74 62           JE SHORT DIY任务?0105B0FF          ；不成功跳到恢复现场
0105B09D    68 04010000     PUSH 104
0105B0A2    8D85 F4FBFFFF   LEA EAX,DWORD PTR SS:[EBP-40C]
0105B0A8    50              PUSH EAX                           ；传入保存结果地址
0105B0A9    FFB5 00FAFFFF   PUSH DWORD PTR SS:[EBP-600]        ；OpenProcess句柄
0105B0AF    E8 9475FBFF     CALL DIY任务?01012648              ；获取进程映像路径的函数
0105B0B4    85C0            TEST EAX,EAX                       ；EAX保存查询到的字符串WORD数
0105B0B6    74 31           JE SHORT DIY任务?0105B0E9          ；没有查询到则转CloseHandle
0105B0B8    8D3C00          LEA EDI,DWORD PTR DS:[EAX+EAX]     ；转为字节数保存到EDI
0105B0BB    8D47 02         LEA EAX,DWORD PTR DS:[EDI+2]       ；字符串结尾0000需要两个字节
0105B0BE    50              PUSH EAX                           ； /Size
0105B0BF    6A 40           PUSH 40                            ； |Flags = LPTR
0105B0C1    FF15 38110001   CALL DWORD PTR DS:[<&kernel32.LocalAlloc>; kernel32.LocalAlloc
0105B0C7    85C0            TEST EAX,EAX                       ；判断是否申请成功
0105B0C9    8985 F8F9FFFF   MOV DWORD PTR SS:[EBP-608],EAX     ；将申请到的空间首地址保存
0105B0CF    74 18           JE SHORT DIY任务?0105B0E9          ；不成功则执行CloseHandle
0105B0D1    8BCF            MOV ECX,EDI                        ；字节数存放到ECX
0105B0D3    8BF8            MOV EDI,EAX                        ；字符串复制目的地址放入EDI
0105B0D5    8BC1            MOV EAX,ECX                        ；复制字节数保存在EAX
0105B0D7    C1E9 02         SHR ECX,2                          ；字节数/4=按DWORD复制的次数
0105B0DA    8DB5 F4FBFFFF   LEA ESI,DWORD PTR SS:[EBP-40C]     ；字符串复制源地址存入ESI
0105B0E0    F3:A5           REP MOVS DWORD PTR ES:[EDI],DWORD PTR DS:[ESI] ；执行字符串双字复制
0105B0E2    8BC8            MOV ECX,EAX                        ；复制字节数保存在ECX
0105B0E4    83E1 03         AND ECX,3                          ；计算剩余字节数,即按字节复制次数
0105B0E7    F3:A4           REP MOVS BYTE PTR ES:[EDI],BYTE PTR DS:[ESI]  ；执行字符串字节复制
0105B0E9    FFB5 00FAFFFF   PUSH DWORD PTR SS:[EBP-600]               ；OpenProcess句柄
0105B0EF    FF15 5C110001   CALL DWORD PTR DS:[<&kernel32.CloseHandl> ；CloseHandle
0105B0F5    8B85 F8F9FFFF   MOV EAX,DWORD PTR SS:[EBP-608]     ；读取保存的字符串地址
0105B0FB    85C0            TEST EAX,EAX                       ；判断关闭句柄是否成功
0105B0FD    75 07           JNZ SHORT DIY任务?0105B106         ；成功关闭句柄就跳到0105B106继续执行
0105B0FF    61              POPAD                              ；否则就恢复环境
0105B100    61              POPAD                              ；恢复环境，因为pushad两次
0105B101  - E9 8F0DFBFF     JMP DIY任务?0100BE95               ；跳到默认的消息处理流程
0105B106    61              POPAD                              ；恢复环境
0105B107    8B85 F8F9FFFF   MOV EAX,DWORD PTR SS:[EBP-608]     ；把进程的映像路径保存到EAX
0105B10D    50              PUSH EAX                           ；把映像路径压栈
0105B10E    FFD3            CALL EBX                           ；调用“GetFileNamebyPath”函数
0105B110    56              PUSH ESI                           ；把HMODULE参数压栈
0105B111    FF15 D8100001   CALL DWORD PTR DS:[<&kernel32.FreeLibrar>; kernel32.FreeLibrary
0105B117    61              POPAD                              ；恢复环境
0105B118  - E9 780DFBFF     JMP DIY任务?0100BE95               ；跳转到默认的消息处理流程
```

—————————————————————————————————————————————

至此，所有的补丁代码完毕，保存后，任务管理器就可以运行了。（这里我只调用了“`GetFileNamebyPath`”函数,因为我把"`BKillProcess`"这个函数在“`GetFileNamebyPath`”末尾调用了，前面已经说了。也许你会问我又获得了"`BKillProcess`"地址做什么，呵呵，因为我刚开始的思路是分别调用每个函数，所有就获得了"`BKillProcess`"函数的地址，后来在实际操作中，我发现分两次调用，我就会破坏堆栈，在地址“`0105B118`”执行完后，来到下面的地址开始执行，执行完 “`0100BE97`”，程序就不知道该到哪里执行了。`EIP`被破坏了。所有我调用一次函数。如果谁知道这个原因，请帮我解释一下，谢谢！）

```
0100BE95   > \5E            POP ESI                                  ;  分支 0100BDAA 默认案例
0100BE96   .  5D            POP EBP
0100BE97   .  C2 0800       RETN 8
```

—————————————————————————————————————————————

我又郁闷了，写了好久，刚要发布，突然发现我的QQ变成灰色的了，我断网了。无奈之下只好睡觉了。明天早晨在发布吧，谨以此文纪念我不幸的一天！

说句题外话，马上要过年了。大家都注意安全，男同胞们少喝点酒，喝酒千万别开车，被抓到酒后驾车的抓，一次扣12分。

女同胞们别自己走夜路。什么？你是单身，好办，你找我吧，免费的“护花使者”，呵呵^_^！

最后祝大家在新的一年里面能有新的气色，政府官员：步步高升，生意人：财源滚滚，学生：学业有成，老人：福如东海，男人：彩旗飘飘，女人：找一个外面没有彩旗的好老公。祝所有人：笑口常开，心想事成，身体健康，万事如意。

最后祝我自己明年能领个媳妇回家过大年！工资争取翻一番，能翻2翻更好，呵呵，多多益善！

--------------------------------------------------------------------------------

## 【经验总结】

耐心，对一些重复性高，比较细致的工作，必须要有耐心。

头脑，要学以致用，把日常生活和学习联系起来。多动脑，多动手。

注意字符编码。微软里面的东西都是 Unicode 的，这么说不知道准确不准确，好处是支持65535个字符，支持各种符号的显示。

--------------------------------------------------------------------------------

## 【版权声明】

转载请注明作者并保持文章的完整,文章出自看雪论坛。 谢谢!

2010年02月07日 01:21:57

本代码的着色效果由`xTiNt`自动完成

下载`xTiNt`

**附件下载:**

- [DIY 让我们的任务管理器再智能一点.rar](https://bbs.kanxue.com/attach-download-38872.htm)
- [[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

上传的附件：

- [效果图.jpg](javascript:void(0);)
- [DIY 让我们的任务管理器再智能一点.rar](javascript:void(0);)
