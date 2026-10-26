
在 `Import REConstructor v1.4.2+` 的 `Attach to an Active Process` 窗口中选取加壳程序的进程，有 2 个进程选上面的一个，然后在下方的 `oep` 处填入 `000BA697`，点 `IAI AutoSearch`，再点 `Get Imports`，然后点 `Auto Trace`，然后点看还有多少地址是 `NO`，剩下的事就是手动找这些函数了。

`hying` 大哥说过 `CreateEventW` `GetStringTypeA` 如果发现这两个函数连续在一起的话，上面一个该改为 `GetStringTypeW` 手工修复修入表后，程序应能运行了，好像也就不用注册了，因为注册部分在壳中，壳脱了注册也就不用了。这程序的输入表我是手工找到一部分，然后参照 `hying` 大哥给我的 `Konvertor 3.12A` 的输入表修复的。先用 `importREC` 试试，输入我们找到的正确的 `OEP`: `004BA697`，注意输入 `004BA697-00400000=000BA697` 有很多指针无效。

这里现不要 `bpx GetModuleHandleA`，否则会被发现的，我想可能在 `unpack` 前程序会检测，所以要是能在程序开始 `unpack` 后再设断多好，怎么做呢，这样，先 `bpx SetProcessWorkingSetSize`

断下来之后再 `bpx GetModuleHandleA` 按 `F12` 没几次就到了如下

```
001B:01085B86  FF15AC800A01        CALL      [KERNEL32!GetModuleHandleA]

001B:01085B8C  394508              CMP       [EBP+08],EAX

001B:01085B8F  7507                JNZ       01085B98

001B:01085B91  B968B40A01          MOV       ECX,010AB468

001B:01085B96  EB52                JMP       01085BEA

001B:01085B98  393D68BA0A01        CMP       [010ABA68],EDI

001B:01085B9E  B968BA0A01          MOV       ECX,010ABA68

001B:01085BA3  0F8491000000        JZ        01085C3A  ======>magic jump!

001B:01085BA9  8B35581C0B01        MOV       ESI,[010B1C58]

001B:01085BAF  A1085D0B01          MOV       EAX,[010B5D08]

001B:01085BB4  F6410801            TEST      BYTE PTR [ECX+08],01

001B:01085BB8  740E                JZ        01085BC8

001B:01085BBA  8B507C              MOV       EDX,[EAX+7C]

001B:01085BBD  335070              XOR       EDX,[EAX+70]

001B:01085BC0  33501C              XOR       EDX,[EAX+1C]

001B:01085BC3  F6C280              TEST      DL,80

001B:01085BC6  7513                JNZ       01085BDB

001B:01085BC8  8B507C              MOV       EDX,[EAX+7C]

001B:01085BCB  335078              XOR       EDX,[EAX+78]

001B:01085BCE  335068              XOR       EDX,[EAX+68]

001B:01085BD1  335004              XOR       EDX,[EAX+04]

001B:01085BD4  3316                XOR       EDX,[ESI]

001B:01085BD6  395508              CMP       [EBP+08],EDX

001B:01085BD9  740C                JZ        01085BE7

001B:01085BDB  83C10C              ADD       ECX,0C

001B:01085BDE  83C604              ADD       ESI,04

001B:01085BE1  3939                CMP       [ECX],EDI

001B:01085BE3  75CF                JNZ       01085BB4
```

这里面有几个跳转，其中最后一个 `01085BB4` 是往上跳的。我们看一个就可发现：

`001B:01085BA3  0F8491000000  JZ 01085C3A`，这个跳正好可跳出这个循环。只要改这个跳，就不会破坏引入表了。再来，注意刚回到那段代码时，就下 `a 01085BA3` 将 `JZ 01085C3A` 改成 `JMP 01085C3A`，清除所有断点，按 `F5` 运行，程序运行后打开 `importREC`，选中程序，注意选择上面那个进程，入口填入 `OEP: 004BA697`，`-->RVA 005AC000 SIZE 00000214-->获得输入信息"。

如果还有无效的大可放心的 `cut`，因为它们本来就不是有用指针了。然后修复脱壳后的文件。`OK`，大功告成。脱壳后的程序，运行正常，圆满成功。
