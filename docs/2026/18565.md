
# [原创]Windows 所有程序崩溃漏洞

只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

受影响平台x86: `xp` `win7` 其它平台没测试

中枪控件: `Edit` `ThunderRT6TextBox` `ThunderRT6FormDC` `Chrome_OmniboxView` 等

我只重点拿`notepad.exe`的`edit`控件做测试.

分析发现`comctl32.dll`这个模块造成的崩溃.

全部都是GDI操作的.

`comctl32.dll`

----------------------------------------------

```
771DE49A  |.  E8 DFD4FFFF   call COMCTL32.771DB97E
771DE49F  |.  E9 C8030000   jmp COMCTL32.771DE86C
771DE4A4  |>  F646 62 40    test byte ptr ds:[esi+62], 40            ;  Case BD (EM_GETHANDLE) of switch 771DE460
771DE4A8  |.  56            push esi
771DE4A9  |.  74 0E         je short COMCTL32.771DE4B9
771DE4AB  |.  E8 DE36FCFF   call COMCTL32.771A1B8E
771DE4B0  |.  8B4E 0C       mov ecx, dword ptr ds:[esi+C]
771DE4B3  |.  C60408 00     mov byte ptr ds:[eax+ecx], 0
771DE4B7  |.  EB 0D         jmp short COMCTL32.771DE4C6
771DE4B9  |>  E8 D036FCFF   call COMCTL32.771A1B8E
771DE4BE  |.  8B4E 0C       mov ecx, dword ptr ds:[esi+C]
771DE4C1  |.  66:832448 00  and word ptr ds:[eax+ecx*2], 0           ;  // error, eax=0  and ecx = 0  其中EAX是wparn
771DE4C6  |>  56            push esi
771DE4C7  |.  E8 2F37FCFF   call COMCTL32.771A1BFB
771DE4CC  |.  8B06          mov eax, dword ptr ds:[esi]
771DE4CE  |.  E9 99030000   jmp COMCTL32.771DE86C
771DE4D3  |>  53            push ebx                                 ;  Case BB (EM_LINEINDEX) of switch 771DE460
771DE4D4  |.  56            push esi
771DE4D5  |.  E8 0ECDFFFF   call COMCTL32.771DB1E8
771DE4DA  |.  E9 8D030000   jmp COMCTL32.771DE86C
771DE4DF  |>  53            push ebx                                 ;  Case C1 (EM_LINELENGTH) of switch 771DE460
```

------------------------------------------------------------------------------------

试了一天没有找到利用方法.不过感觉应该是可以利用的. ![](https://bbs.kanxue.com/view/img/face/12.gif)

最近时间太忙了,等有时间再继续看看. ![](https://bbs.kanxue.com/view/img/face/50.gif)

本来是分析360程序的,发现了系统的这个BUG.

起初一直以为是360程序的BUG.

用`windbg`调试,发现,是系统造成的.

再没有安装360的电脑上测试,一样中枪.

剩下的大家看代码吧.

上个DEMO图.

![](https://bbs.kanxue.com/upload/attach/201305/532453_be4dm9d91sif7ay.jpg)

## 中招条件:

1. 程序有窗口.
2. 程序有使用 `edit` 一类的控件.(至于哪些控件中枪,得自测了.)
3. 即使程序有加载驱动,只要没有HOOK 消息,一样有效果.(如 `XueTr` `Kernel Detective` `360 Taskmgr` 等.) ![](https://bbs.kanxue.com/view/img/face/1.gif)

![](https://bbs.kanxue.com/upload/attach/201305/532453_5g5itpqqi8i0jem.jpg)

![](https://bbs.kanxue.com/upload/attach/201305/532453_5dadv0qvbby2n0l.jpg)

![](https://bbs.kanxue.com/upload/attach/201305/532453_8u6qpp456z3oy36.jpg)

![](https://bbs.kanxue.com/upload/attach/201305/532453_l8qfqmkvadzetll.jpg)

## 控制台版本:(此版本效果更佳.崩溃的机率更高)

![](https://bbs.kanxue.com/upload/attach/201305/532453_tidrpk32wvx5nsj.jpg)

/*本人博客:[3c5K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3y4F1j5X3I4G2k6%4y4Q4x3X3g2U0L8$3#2Q4x3V1k6T1K9X3u0D9j5%4u0S2j5$3E0W2k6l9`.`.](https://bbs.kanxue.com/elink@4e6K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3y4F1j5X3I4G2k6%4y4Q4x3X3g2U0L8$3#2Q4x3V1k6T1K9X3u0D9j5%4u0S2j5$3E0W2k6l9`.`.)

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [GDI.jpg](javascript:void(0);)
  （27.58kb，5次下载）
- [GDI漏洞_源码.7z](javascript:void(0);)
  （8.29kb，307次下载）
- [GDI漏洞_Bin.7z](javascript:void(0);)
  （87.06kb，290次下载）
- [XueTr.jpg](javascript:void(0);)
  （60.54kb，16次下载）
- [Kernel Detective.jpg](javascript:void(0);)
  （106.58kb，104次下载）
- [Kernel Detective2.jpg](javascript:void(0);)
  （91.39kb，9次下载）
- [TaskMgr.jpg](javascript:void(0);)
  （71.13kb，12次下载）
- [控制台版本.jpg](javascript:void(0);)
  （43.69kb，9次下载）
- [Console_Bin.7z](javascript:void(0);)
  （16.72kb，193次下载）
- [Console_Src.7z](javascript:void(0);)
  （9.32kb，229次下载）
