
# FoxitReaderPro 广告条分析笔记

## 汇编片段 1

```
005570A6  |.  897D D8       mov     [ebp-28], edi                             ;  80A2 菜单ID号...
005570A9  |.  FF50 0C       call    [eax+C]                                   ;  a.0056ddd6   比较之处
005570AC  |.  395D FC       cmp     [ebp-4], ebx
005570AF  |.  74 3E         je      short 005570EF
005570B1  |.  895D 08       mov     [ebp+8], ebx
005570B4  |>  8B06          mov     eax, [esi]
005570B6  |.  53            push    ebx
005570B7  |.  53            push    ebx
005570B8  |.  8BCE          mov     ecx, esi
005570BA  |.  FF75 08       push    dword ptr [ebp+8]
005570BD  |.  57            push    edi
005570BE  |.  FF50 0C       call    [eax+C]       ------------------------》关键函数：
005570C1  |>  5F            pop     edi
005570C2  |.  5E            pop     esi
005570C3  |.  5B            pop     ebx
005570C4  |.  C9            leave
005570C5  |.  C2 0800       retn    8
```

## 第六站直接亲密

.....

我写这累了,相比你看这也累了,我直接给出结局来！

原来的时候我是下`USER32.dll.ShowWindow`断点可是广告条根本就不吃这一套,断不下来的！还有当点击广告条的时候会进入它官方网站,这时候又下的`SHELL32.ShellExecute`断点等也没有断下来.只能断个`MessageBox`,感觉交互信息太少,还有它的菜单ID迟迟不比较等等！直到最后跟到这里才恍然大悟...

原来`FoxitReaderPro`是用的`InsertMenuA`插入广告条

用`DeleteMenu`删除广告条

用`DrawMenuBar`来刷新广告条的颜色,你会看到红,绿等色的交替！

用`SetRectEmpty`来清理广告条的矩形对象

## 汇编片段 2

```
0040C9C0  /$  56            push    esi
0040C9C1  |.  57            push    edi
0040C9C2  |.  8BF9          mov     edi, ecx
0040C9C4  |.  8B47 1C       mov     eax, [edi+1C]
0040C9C7  |.  50            push    eax                                       ; /hWnd
0040C9C8  |.  FF15 44175900 call    [<&USER32.GetMenu>]                       ; \GetMenu
0040C9CE  |.  50            push    eax
0040C9CF  |.  E8 E5D21400   call    00559CB9
0040C9D4  |.  8BF0          mov     esi, eax
0040C9D6  |.  85F6          test    esi, esi
0040C9D8      0F85 90000000 jnz     0040CA6E
0040C9DE  |.  8B4424 0C     mov     eax, [esp+C]
0040C9E2  |.  83F8 05       cmp     eax, 5
0040C9E5  |.  75 40         jnz     short 0040CA27
0040C9E7  |.  68 00410000   push    4100
0040C9EC  |.  56            push    esi
0040C9ED  |.  8BCF          mov     ecx, edi
0040C9EF  |.  E8 3CFBFFFF   call    0040C530
0040C9F4  |.  83F8 FF       cmp     eax, -1                                   ;  打开 advise....
0040C9F7      75 61         jnz     short 0040CA5A
0040C9F9  |.  8B4E 04       mov     ecx, [esi+4]
0040C9FC  |.  68 0C875D00   push    005D870C                                  ; /pNewItem = "PDF 编?,AD,"?,F7,""
0040CA01  |.  68 00410000   push    4100                                      ; |NewItemID = 4100 (16640.)
0040CA06  |.  68 00450000   push    4500                                      ; |Flags = MF\_BYPOSITION|MF\_ENABLED|MF\_STRING|MF\_OWNERDRAW|MF\_HELP
0040CA0B  |.  50            push    eax                                       ; |ItemID
0040CA0C  |.  51            push    ecx                                       ; |hMenu
0040CA0D  |.  FF15 28175900 call    [<&USER32.InsertMenuA>]                   ; \InsertMenuA
0040CA13  |.  8B4F 1C       mov     ecx, [edi+1C]
0040CA16  |.  51            push    ecx                                       ; /hWnd
0040CA17  |.  FF15 30175900 call    [<&USER32.DrawMenuBar>]                   ; \DrawMenuBar
0040CA1D      5F            pop     edi
0040CA1E      B8 01000000   mov     eax, 1
0040CA23  |.  5E            pop     esi
0040CA24  |.  C2 0400       retn    4
0040CA27  |>  85C0          test    eax, eax
0040CA29  |.  75 2F         jnz     short 0040CA5A
0040CA2B  |.  68 00410000   push    4100
0040CA30  |.  56            push    esi
0040CA31  |.  8BCF          mov     ecx, edi
0040CA33  |.  E8 F8FAFFFF   call    0040C530
0040CA38  |.  83F8 FF       cmp     eax, -1
0040CA3B  |.  74 1D         je      short 0040CA5A
0040CA3D  |.  8B56 04       mov     edx, [esi+4]
0040CA40  |.  68 00040000   push    400                                       ; /Flags = MF\_BYPOSITION|MF\_ENABLED|MF\_STRING
0040CA45  |.  50            push    eax                                       ; |ItemId
0040CA46  |.  52            push    edx                                       ; |hMenu
0040CA47  |.  FF15 34175900 call    [<&USER32.DeleteMenu>]                    ; \DeleteMenu
0040CA4D  |.  8D87 28020000 lea     eax, [edi+228]
0040CA53  |.  50            push    eax                                       ; /pRect
0040CA54  |.  FF15 B4175900 call    [<&USER32.SetRectEmpty>]                  ; \SetRectEmpty
0040CA5A  |>  8B4F 1C       mov     ecx, [edi+1C]
0040CA5D  |.  51            push    ecx                                       ; /hWnd
0040CA5E  |.  FF15 30175900 call    [<&USER32.DrawMenuBar>]                   ; \DrawMenuBar
0040CA64  |.  5F            pop     edi
0040CA65  |.  B8 01000000   mov     eax, 1
0040CA6A  |.  5E            pop     esi
0040CA6B  |.  C2 0400       retn    4
0040CA6E  |>  5F            pop     edi
0040CA6F  |.  33C0          xor     eax, eax
0040CA71  |.  5E            pop     esi
0040CA72  \.  C2 0400       retn    4
```

任务完成了,该找出来的都出来,到这里你稍微一动鼠标,广告就`over`掉了......

## 总结

末了：

`FoxitReaderPro` 的做的广告条正是以菜单栏为父窗口,因此为什么当跟踪的时候始终没有看到影射出一个`CWnd`类或者是`CFarme`类的`C++`对象,还有那些`ShowWindow`等窗体的断点API根本不管用的！

另外`FoxitReaderPro`还有一个有意思的地方就是当你点击广告条的时候,它会启动浏览器然后加载它的主页地址.我一开始用的是`SHELL32.ShellExecute`断点,不管用！它用的`KERNEL32.CreateProcess`,它把官方URL作为参数启动IE,当触发事件的时候就执行了`CreateProcess`.....

好累了,直到写完了我才吃了点东西！

Copyright © 2000 - 2006 PEdiy.com All Rights Reserved.By KanXue Studio

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
