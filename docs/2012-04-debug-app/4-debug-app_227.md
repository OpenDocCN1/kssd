
# 代码补丁实现扫雷秒杀

> 原文：[`www.pediy.com/kssd/pediy10/85841.html`](https://www.pediy.com/kssd/pediy10/85841.html)

关于扫雷秒杀的文章，看雪论坛上讨论的比较多，本文的部分地址也是来自看雪别人分析的结果，在这里对前辈们做的分析表示由衷的感谢。

以下工作均在 `Ollydbg` 下实现，不会 `OD` 的请先简单的学习下 `OD` 的使用再来看本文。

本文的思路是选取一个用处不大的菜单，修改菜单消息响应函数地址，使他指向我们自己编写的一个函数中，有点像 `HOOK`，我在这里选取的是使用帮助这个菜单，感觉没什么用处，就改这个了。

第一步，我们要找一块不用的空间来存放我们编写的代码，我这里使用的是 `01004A57`，后面是一大段的 `00`，所以我就用这个了，你也可以选取其他的空间，当然必须要能够存放的下我们编写的代码，不能把别人的代码给覆盖了。

在这里，我们要完成秒杀的功能，实际上就是自动完成扫雷的工作，这里有几个比较重要的地址：

`0x10056AC` 列的数目

`0x10056A8` 行的数目

【列+行\*0x20+0x1005340】的值如果为 `8F` 就表示存在雷

`01003512` 为关键 `CALL`，扫雷的函数地址。

编写以下代码到空白区域

```
01004A57  $  55            PUSH EBP  
01004A58  .  8BEC          MOV EBP,ESP                     ;保存堆栈  
01004A5A  .  A1 AC560001   MOV EAX,DWORD PTR DS:[10056AC]；获取到列的数目  
01004A5F  .  83C0 01       ADD EAX,1 ；列加 1  
01004A62  .  50            PUSH EAX ；保存列到堆栈中  
01004A63  .  8B0D A8560001 MOV ECX,DWORD PTR DS:[10056A8]；获取行的数目  
01004A69  .  83C1 01       ADD ECX,1；行加 1  
01004A6C  .  51            PUSH ECX ；保存行到堆栈中  
01004A6D  .  6A 00         PUSH 0 ；创建一个堆栈变量，保存外层循环次数（列记次变量）  
01004A6F  .  6A 00         PUSH 0 ；创建一个堆栈变量，保存内层循环次数（行记次变量）  
01004A71  .  33C0          XOR EAX,EAX ；eax 得到一个 0 值  
01004A73  .  8945 F0       MOV DWORD PTR SS:[EBP-10],EAX；将 0 放到堆栈中，初始值  
01004A76  >  8345 F0 01    ADD DWORD PTR SS:[EBP-10],1；循环开始，堆栈中变量加 1  
01004A7A  .  8B45 F0       MOV EAX,DWORD PTR SS:[EBP-10]；取出外层循环变量放到 eax 中  
01004A7D  .  90            NOP ；调试留下的一些空位置  
01004A7E  .  90            NOP  
01004A7F  .  90            NOP  
01004A80  .  3945 FC       CMP DWORD PTR SS:[EBP-4],EAX；比较外层循环是否结束  
01004A83  .  74 2E         JE SHORT winmine.01004AB3；循环完了就跳  
01004A85  .  33C9          XOR ECX,ECX；获取一个 0 值  
01004A87  .  894D F4       MOV DWORD PTR SS:[EBP-C],ECX；将 0 保存到堆栈变量中  
01004A8A  >  8345 F4 01    ADD DWORD PTR SS:[EBP-C],1；堆栈变量加 1，内层循环开始  
01004A8E  .  8B4D F4       MOV ECX,DWORD PTR SS:[EBP-C]；取出堆栈变量到 ecx  
01004A91  .  394D F8       CMP DWORD PTR SS:[EBP-8],ECX；比较内层循环是否结束  
01004A94  .^ 74 E0         JE SHORT winmine.01004A76；结束了就调到外层循环  
01004A96  .  8BF1          MOV ESI,ECX  
01004A98  .  C1E6 05      SHL ESI,5；esi ＝ esi×32  
01004A9B  .  F68430 405300> TEST BYTE PTR DS:[EAX+ESI+1005340],80；看该位置是否有雷，080h 就是有雷的标记  
01004AA3  .^ 75 E5         JNZ SHORT winmine.01004A8A；有雷就继续循环，没有雷就挖掉  
01004AA5  .  51            PUSH ECX；参数 2，行  
01004AA6  .  50            PUSH EAX；参数 1，列  
01004AA7  .  E8 66EAFFFF   CALL winmine.01003512；调用扫雷函数  
01004AAC  .  8B45 F0       MOV EAX,DWORD PTR SS:[EBP-10]；取出堆栈变量到 eax  
01004AAF  .^ EB D9         JMP SHORT winmine.01004A8A；跳到内层循环  
01004AB1  .^ EB C3         JMP SHORT winmine.01004A76；跳到外层循环  
01004AB3  >  5F            POP EDI；这里 2 个 pop 是为了平衡堆栈，因为原来的函数带 2 个参数  
01004AB4  .  5E            POP ESI  
01004AB5  .  C9            LEAVE  
01004AB6  .  C2 0800      RETN 8；返回  
```

这段代码的意思就是遍历雷区所有位置的方格，如果方格内部不存在雷，就调用函数 `01003512` 挖掉该位置。

由于我选取的是菜单使用帮助的消息响应，原来的函数有 2 个参数，因此这里最后要用 2 个 `pop`，中间的 `NOP` 是我在调试的时候留下的，你可以去掉，不过要记得修改跳转语句和 `CALL` 的地址。

第二步到消息响应的位置，修改消息响应函数，将这里的 `CALL 01003D76` 修改 `CALL 01004A57`。

```
01001F03  |. /EB 0A        JMP SHORT winmine.01001F0F  
01001F05  |> |6A 02        PUSH 2                              ;  Case 24F of switch 01001EDC，这里是第二个参数  
01001F07  |. |6A 01        PUSH 1；这里是第一个参数  
01001F09  |. |EB 04        JMP SHORT winmine.01001F0F  
01001F0B  |> |6A 00        PUSH 0                              ;  Case 24E of switch 01001EDC  
01001F0D  |. |6A 03        PUSH 3  
01001F0F  |> \\E8 432B0000   CALL winmine.01003D76；要将这里改成我们自己编写的 CALL 地址  
01001F14  |.   E9 90020000   JMP winmine.010021A9  
```

修改好了，就直接用 `OD` 复制到可执行文件，全部保存内容，就基本上做好了。

完成以上的工作后就是修改下菜单的资源了，使用 `PExplorer` 修改菜单使用帮助的标题为使用秒杀。

好了，现在可以来看看效果了，点下使用秒杀，就可以看到自己完成，时间为 0，雷的自动标记也已经做好了。

附件是修改好了扫雷，可以看看效果。

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || winmine.rar  
|
