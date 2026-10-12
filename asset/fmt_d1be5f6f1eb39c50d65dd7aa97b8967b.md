
# AceReader V4.5

> 原文：[`www.pediy.com/kssd/pediy05/pediy50499.htm`](https://www.pediy.com/kssd/pediy05/pediy50499.htm)

目标软件：`AceReader V4.5`

软件大小：787 KB

软件语言：英文

软件类别：国外软件 / 共享版 / 电子阅读

应用平台：`Win9x/NT/2000/XP`

下载地址：`http://www.skycn.com/soft/9585.html`

**软件介绍**

为改进你的阅读技巧，`AceReader` 包含两种模式：眼扫描训练模式和中心文本模式。在眼扫描训练模式中，文本从显示区域卷绕穿过，迫使你的眼睛象正常阅读那样移动。该软件可以作为一种教学工具来使用，通过设置在更高的速度档上的练习，可以使你不论在计算机上还是在书本上都能提高阅读速度。软件能通过不同等级水平的阅读理解来跟踪你的进度。`AceReader` 可以帮你抛弃每个词都在脑子里发音的阅读习惯，从而从整体上改进你的阅读技巧和速度。在中心文本模式中，文本显示在文本区的中央。这种模式可作为一种高效率的阅读工具来使用，从而使阅读更加容易和迅速，因为你的视线无需移动。你可以用该软件作快速阅读、略读文档中或剪贴板上的文字。`AceReader` 的设计是面向所有年龄段的用户的，从需要在现在这个信息时代阅读得更加迅速的繁忙的行政人员，到刚刚学会怎样读书的小孩子都可以使用它。

**破解过程**

`Hmemcpy`，设断，易来到：

```
PUSH    EBX
PUSH    ESI
PUSH    EDI
MOV     [EBP-04],ECX
MOVSX   EAX,BYTE [EBP+0C]    //查表。
MOV     ECX,09               //0x9 送入 ECX。
ADD     EAX,BYTE +02         //ECX+0x2。
CDQ
IDIV    ECX                  //除以 ECX。
MOV     EAX,[EBP+08]         //用户名送入 EAX。
MOVSX   EAX,BYTE [EAX]       //取用户名的第一位。
MOV     ECX,0B               //0xB 送入 ECX。
LEA     EAX,[EAX+EDX+08]     //EAX+EDX+08 的值保存到 EAX。
CDQ
IDIV    ECX                  //除以 ECX。
LEA     EAX,[EDX+70]         //EDX+0x70 的值保存到 EAX。
MOV     ECX,[EBP+10]         //EBP+0x10 的值送入 ECX。
MOV     [ECX],AL             //AL 的值日送入 ECX。
MOVSX   EAX,BYTE [EBP+0C]    //查表。
MOV     ECX,05               //0x5 送入 ECX。
ADD     EAX,BYTE +09         //EAX+0x9。
CDQ
IDIV    ECX                  //除以 ECX。
MOV     EAX,[EBP+08]         //用户名送入 EAX。
MOVSX   EAX,BYTE [EAX+01]    //取用户名的第二位。
LEA     EAX,[EAX+EDX+2C]     //将 EAX+EDX+2C 的值保存到 EAX。
CDQ
XOR     EAX,EDX              //EAX 与 EDX 做异或运算。
SUB     EAX,EDX              //EAX-EDX。
AND     EAX,BYTE +07         //EAX 与 0x7 做与运算。
XOR     EAX,EDX              //EAX 与 EDX 做异或运算。
SUB     EAX,EDX              //EAX-EDX。
ADD     EAX,BYTE +41         //EAX+0x41。
MOV     ECX,[EBP+10]         //EBP+0x10 的值送入 ECX。
MOV     [ECX+01],AL          //AL 的值日送入 ECX+0x1 处。
MOVSX   EAX,BYTE [EBP+0C]    //查表。
MOV     ECX,03               //0x3 送入 ECX。
ADD     EAX,BYTE +09         //EAX+0x3。
CDQ
IDIV    ECX                  //除以 ECX。
MOV     EAX,[EBP+08]         //将用户名送入 EAX。
MOVSX   EAX,BYTE [EAX+02]    //取用户名第三位。
ADD     EAX,EDX              //EAX+EDX。
CDQ
XOR     EAX,EDX              //EAX 与 EDX 做异或运算。
SUB     EAX,EDX              //EAX-EDX。
AND     EAX,BYTE +07         //EAX 与 0x7 做与运算。
XOR     EAX,EDX              //EAX 与 EDX 做异或运算。
SUB     EAX,EDX              //EAX-EDX。
ADD     EAX,BYTE +32         //EAX+0x32。
MOV     ECX,[EBP+10]         //EBP+10 处的值送入 ECX。
MOV     [ECX+02],AL          //将 AL 的值送入 ECX+0x2 处。
MOVSX   EAX,BYTE [EBP+0C]    //查表。
ADD     EAX,BYTE +04         //EAX+0x4。
CDQ
XOR     EAX,EDX              //EAX 与 EDX 做异或运算。
SUB     EAX,EDX              //EAX-EDX。
AND     EAX,BYTE +07         //EAX 与 0x7 做与运算。
XOR     EAX,EDX              //EAX 与 EDX 做异或运算。
SUB     EAX,EDX              //EAX-EDX。
MOV     ECX,[EBP+08]         //用户名送入 ECX。
MOVSX   ECX,BYTE [ECX+03]    //取用户名的第四位。
MOV     EBX,05               //将 Ox5 送入 EBX。
LEA     EAX,[ECX+EAX+02]     //将 ECX+EAX+02 的值保存到 EAX。
CDQ
IDIV    EBX                  //除以 EBX。
LEA     EAX,[EDX+4A]         //EDX+0x4A 的值保存到 EAX。
MOV     ECX,[EBP+10]         //EBP+10  处的值送入 ECX。
MOV     [ECX+03],AL          //将 AL 的值送入 ECX+03 处。
MOVSX   EAX,BYTE [EBP+0C]    //查表。
MOV     ECX,06               //将 0x6 送入 ECX。
ADD     EAX,BYTE +03         //EAX+0x3。
CDQ
IDIV    ECX                  //除以 ECX。
MOV     EAX,[EBP+08]         //用户名送入 EAX。
MOVSX   EAX,BYTE [EAX+04]    //取用户名的第五位。
MOV     ECX,0B               //将 0xB 送入 ECX。
LEA     EAX,[EAX+EDX+07]     //将 EAX+EDX+07 的值保存到 EAX。
CDQ
IDIV    ECX                  //除以 ECX。
LEA     EAX,[EDX+61]         //将 EDX+61 的值保存到 EAX。
MOV     ECX,[EBP+10]         //将 EBP+10 的值送入 ECX。
MOV     [ECX+04],AL          //将 AL 送入 ECX+04 处。
MOVSX   EAX,BYTE [EBP+0C]    //查表。
ADD     EAX,BYTE +06         //EAX+0x6。
CDQ
XOR     EAX,EDX              //EAX 与 EDX 做异或运算。
SUB     EAX,EDX              //EAX-EDX。
AND     EAX,BYTE +07         //EAX 与 0x7 做与运算。
XOR     EAX,EDX              //EAX 与 EDX 做异或运算。
SUB     EAX,EDX              //EAX-EDX。
MOV     ECX,[EBP+08]         //用户名送入 ECX。
MOVSX   ECX,BYTE [ECX+05]    //取用户名的第六位。
LEA     EAX,[ECX+EAX+07]     //将 ECX+EAX+07 的值保存到 EAX。
CDQ
XOR     EAX,EDX              //EAX 与 EDX 做异或运算。
SUB     EAX,EDX              //EAX-EDX。
AND     EAX,BYTE +07         //EAX 与 0x7 做与运算。
XOR     EAX,EDX              //EAX 与 EDX 做异或运算。
SUB     EAX,EDX              //EAX-EDX。
ADD     EAX,BYTE +32         //EAX+0x32。
MOV     ECX,[EBP+10]         //将 EBP+10 处的值送入 ECX。
MOV     [ECX+05],AL          //将 AL 的值送入 ECX+05 处。
MOVSX   EAX,BYTE [EBP+0C]    //查表。
ADD     EAX,BYTE +04         //EAX+0x4。
CDQ
XOR     EAX,EDX              //EAX 与 EDX 做异或运算。
SUB     EAX,EDX              //EAX-EDX。
AND     EAX,BYTE +03         //EAX 与 0x3 做与运算。
XOR     EAX,EDX              //EAX 与 EDX 做异或运算。
SUB     EAX,EDX              //EAX-EDX。
MOV     ECX,[EBP+08]         //用户名送入 ECX。
MOVSX   ECX,BYTE [ECX+06]    //取用户名第七位。
LEA     EAX,[ECX+EAX+01]     //将 ECX+EAX+01 的值保存到 EAX。
CDQ
XOR     EAX,EDX              //EAX 与 EDX 做异或运算。
SUB     EAX,EDX              //EAX-EDX。
AND     EAX,BYTE +01         //EAX 与 0x1 做与运算。
XOR     EAX,EDX              //EAX 与 EDX 做异或运算。
SUB     EAX,EDX              //EAX-EDX。
ADD     EAX,BYTE +6D         //EAX+0x6D。
MOV     ECX,[EBP+10]         //将 EBP+10 出的值送入 ECX。
MOV     [ECX+06],AL          //将 AL 的值送入 ECX+06 处。
MOVSX   EAX,BYTE [EBP+0C]    //查表。
MOV     ECX,09               //0x9 送入 ECX。
ADD     EAX,BYTE +09         //EAX+0x9。
CDQ
IDIV    ECX                  //除以 ECX。
MOV     EAX,[EBP+08]         //用户名送入 EAX。
MOVSX   EAX,BYTE [EAX+07]    //取用户名第八位。
LEA     EAX,[EAX+EDX+03]     //将 EAX+EDX+03 的值保存到 EAX。
CDQ
XOR     EAX,EDX              //EAX 与 EDX 做异或运算。
SUB     EAX,EDX              //EAX-EDX。
AND     EAX,BYTE +07         //EAX 与 0x7 做与运算。
XOR     EAX,EDX              //EAX 与 EDX 做异或运算。
SUB     EAX,EDX              //EAX-EDX。
ADD     EAX,BYTE +32         //EAX+0x32。
MOV     ECX,[EBP+10]         //将 EBP+10 处的值送入 ECX。
MOV     [ECX+07],AL          //将 AL 的值送入 ECX+07 处。
MOVSX   EAX,BYTE [EBP+0C]    //查表。
INC     EAX                  //EAX+1。
CDQ
XOR     EAX,EDX              //EAX 与 EDX 做异或运算。
SUB     EAX,EDX              //EAX-EDX。
AND     EAX,BYTE +03         //EAX 与 0x3 做与运算。
XOR     EAX,EDX              //EAX 与 EDX 做异或运算。
SUB     EAX,EDX              //EAX-EDX。
MOV     ECX,[EBP+08]         //用户名送入 ECX。
MOVSX   ECX,BYTE [ECX+08]    //取用户名第九位。
LEA     EAX,[ECX+EAX+09]     //将 ECX+EAX+09 的值保存到 EAX。
CDQ
XOR     EAX,EDX              //EAX 与 EDX 做异或运算。
SUB     EAX,EDX              //EAX-EDX。
AND     EAX,BYTE +01         //EAX 与 0x1 做与运算。
XOR     EAX,EDX              //EAX 与 EDX 做异或运算。
SUB     EAX,EDX              //EAX-EDX。
ADD     EAX,BYTE +6D         //EAX+0x6D。
MOV     ECX,[EBP+10]         //将 EBP+10 处的值送入 ECX。
MOV     [ECX+08],AL          //将 AL 的值送入 ECX+08。
MOVSX   EAX,BYTE [EBP+0C]    //查表。
ADD     EAX,BYTE +08         //EAX+0x8。
CDQ
XOR     EAX,EDX              //EAX 与 EDX 做异或运算。
```
