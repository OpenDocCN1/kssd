
# 可以找到IAT (2千字)

我在此连接下载Maple2.0 URL=`http://www.shellfishsoft.com/maple.zip`

Comment 电脑使用的文字处理程序非常多，从简单的只能编辑纯文字，到复杂的可编辑排版的都有，而Maple的特点则是可以对文件建立阶层式的结构，就像书籍章节段落的安排一样，让文件看起来阶层分明，有条不紊。在Maple视窗的左侧便是阶层的树状图，使用者可以加入无限个节点与次节点，代表上下的阶层分类。右侧则是显示该阶层的内容，支持完整的格式化文字，如字型、尺寸、颜色、对齐等功能，并可插入图片或物件，或者连结其他的文件。亦支持鼠标拖、放的功能。

```
Calculating hash of 1034240 bytes file E:\Maple\maple.exe...
MD5        : 7F36CC2CDA4A8450B4802D9B3D59F336
CRC-32      : E3799E3B
---------------------------------------------------------------------------------------
```

工具TRW2000,Imprec,

henryw 大侠的发现叫人震惊，我自己跟了一下，结果和他差不多^=^

从跟踪的代码看来，初步估计是Asprotect1.2(不是Asprotect1.2 se)。

1)先讲讲 OEP>>

程序於 `FC9D5D JMP EAX` (此时`EAX==401000`), 再从`401000 JMP 401012` (可能是`OEP`)，最后，会跳到`5DA614` (也可能是`OEP`)。经试验验证，`OEP == 401012` (与冲击波的一样，它真厉害呀！)

2)ImportTable>>

IAT 的RVA 是 `412164` , length 是 `11c4` (指在`ImpREC`中。注意，若是`revirgin`则是`11c0`)。其中，的IAT之间有很大的跳跃。

ImpREC1.2beta2 中的`IAT AutoSearch`失效。在`ImpREC` 中添入`OEP`，`IAT`，`Length`后,直接按`GetImport`。

经修复后只剩下一个`rva`没修好，它是`KERNEL32!GetProcAddress` (这是capsr1012告诉我的，我没去跟)。

修复后的程序可以正常运行。

3)原来CASPR1012 可以对这个程序脱壳，这说明我的假设是没错的-->是ASprotect 1.2 !!!! (这个年代竟还有人用这东东^\_^)

4)追踪过程中，无意间发现代码如下，哪位知道是何用途？？

```
017F:00FB0970  CALL    00FADC78
017F:00FB0975  CMP      DWORD [00FB3578],BYTE +00
017F:00FB097C  JZ      00FB098F
017F:00FB097E  MOV      EAX,[00FB3558]
017F:00FB0983  CALL    00FA3400
017F:00FB0988  PUSH    EAX
017F:00FB0989  CALL    MAPLE!@GetRegistrationInformation$qqspc
017F:00FB098F  CMP      DWORD [00FB3598],BYTE +00
017F:00FB0996  JZ      00FB09A3
017F:00FB0998  PUSH    DWORD 00FAC730
017F:00FB099D  CALL    NEAR [00FB3598]
017F:00FB09A3  CMP      DWORD [00FB3558],BYTE +00
017F:00FB09AA  JNZ      00FB09B5
017F:00FB09AC  CMP      DWORD [00FB3598],BYTE +00
017F:00FB09B3  JZ      00FB09
```

5)有谁知道这个程序是如何知道:是否已经过了30天的试用期？
