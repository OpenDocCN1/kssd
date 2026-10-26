
# [原创] AAA AceCapture 1.95 算法分析

【文章标题】: AAA AceCapture 1.95 算法分析

【文章作者】: qifeon

【软件名称】: AAA AceCapture 1.95

【下载地址】: 自己搜索下载

【使用工具】: `od`,`peid`

【操作平台】: `winxp sp2`

【作者声明】: 只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教!

--------------------------------------------------------------------------------

## 【详细过程】

一、 `peid` 查壳，显示无壳，编程语言为：Microsoft Visual C++ 6.0。

二、试运行软件，输入“qifeon,12345-67890”。有错误提示。“registration code is invalid”。

同时注意注册码部分无法输入字母，即只能为数字。

三、`od` 载入程序，查找错误提示字符串。

`Ultra String Reference`, 条目 280

`Address=0040FE47`

`Disassembly=push 0046E074`

`Text String=registration code is invalid!`

双击来到

```
0040FE45  |.  57            push    edi
0040FE46  |.  57            push    edi
0040FE47  |.  68 74E04600   push    0046E074                         ;  registration code is invalid!
0040FE4C  |.  EB 22         jmp     short 0040FE70                       返回处
0040FE4E  |>  837D EC 01    cmp     dword ptr [ebp-14], 1
0040FE52  |.  75 21         jnz     short 0040FE75
0040FE54  |.  57            push    edi
0040FE55  |.  57            push    edi
0040FE56  |.  68 58E04600   push    0046E058                         ;  thank you for supporting!
0040FE5B  |.  E8 34AB0200   call    0043A994
0040FE60  |.  8BCE          mov     ecx, esi
0040FE62  |.  E8 57E70100   call    0042E5BE
```

********************************************************************************

向上找到段首

********************************************************************************
