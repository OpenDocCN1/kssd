
# 呼吸主页秀

> 原文：`www.pediy.com/kssd/pediy06/pediy6105.htm`

软件大小： 1213 KB

软件语言： 简体中文

软件类别： 国产软件 / 共享版 / 网页辅助

应用平台： `Win9x/NT/2000/XP`

加入时间： 2002-02-05 15:34:44

下载次数： 27068

推荐等级： \*\*\*\*

软件下载：`http://count.skycn.com/softdown.php?id=960&url=http://js\-http.skycn.net/down/okayle32.exe`

## 软件介绍

呼吸『OK 了！』主页秀是呼吸小秘——主页特效篇 1.2 版的升级版本。软件在呼吸小秘书主页特效篇基础上作了较大改动，增加了大量的特效代码，改变了其原有的设计风格，具有独特的窗口预览功能，无需使用 `IE` 进行特效预览（部分特效受窗口设计的限制仍然使用 `IE` 观看）。软件继承原有风格，简单易用，不需要太多的网页知识便能轻松上手，软件内置代码编辑器，能将代码轻松的插入 `htm` 文件中。软件中每一个插件都具有相应的提示信息，使操作者使用更加方便。除特效生成外还有网页颜色捕获工具、软件特效代码智能升级、自制插件代码管理、免费计数器、留言本申请等。

## 破解工具

`TRW2000`, `Windows` 自带计算器

## 作者声明

初学破解，仅作学习交流之用，失误之处敬请大侠赐教

`Delphi` 编写，无壳。

填入用户名: `ShenGe` 和注册码: `52093241386514845512`, 用 `S` 大法定位，来到如下代码：

```
0167:004AB72A  CALL     0042FCE8
                       <--取输入的假码
0167:004AB72F  MOV      EAX,[EBP-0C]
                       <--EAX="52093241386514845512"
0167:004AB732  CALL     00403F10
                       <--取假码的长度
0167:004AB737  CMP      EAX,BYTE +14
                       <--比较假码是否为 20 位
0167:004AB73A  JNZ      NEAR 004ABCB2
0167:004AB740  LEA      EDX,[EBP-10]
0167:004AB743  MOV      EAX,[EBX+0300]
0167:004AB749  CALL     0042FCE8
                       <--取用户名
0167:004AB74E  MOV      EAX,[EBP-10]
                       <--EAX=ShenGeAA
0167:004AB751  CALL     00403F10
0167:004AB756  TEST     EAX,EAX
                       <--判断有无输入用户名
0167:004AB758  JNG      NEAR 004ABCB2
0167:004AB75E  LEA      EAX,[EBP-04]
0167:004AB761  PUSH     EAX
0167:004AB762  LEA      EDX,[EBP-18]
0167:004AB765  MOV      EAX,[EBX+0300]
0167:004AB76B  CALL     0042FCE8
0167:004AB770  MOV      EAX,[EBP-18]
0167:004AB773  LEA      EDX,[EBP-14]
0167:004AB776  CALL     0040890C
0167:004AB77B  LEA      EAX,[EBP-14]
0167:004AB77E  MOV      EDX,004ABD50
                       <--004ABD50 中的值在内存单元中为
                        20 20 20 20 20 20 20 20
0167:004AB783  CALL     00403F18
0167:004AB788  MOV      EAX,[EBP-14]
0167:004AB78B  MOV      ECX,08
0167:004AB790  MOV      EDX,01
0167:004AB795  CALL     00404118
0167:004AB79A  LEA      EAX,[EBP-08]
                       <--此时 EAX 在内存单元中的值为
                       53 68 65 6E 47 65 20 20
0167:004AB79D  PUSH     EAX
0167:004AB79E  LEA      EDX,[EBP-1C]
0167:004AB7A1  MOV      EAX,[EBX+0310]
0167:004AB7A7  CALL     0042FCE8
0167:004AB7AC  MOV      EAX,[EBP-1C]
                       <--EAX="52093241386514845512"
0167:004AB7AF  PUSH     EAX
0167:004AB7B0  LEA      EAX,[EBP-20]
0167:004AB7B3  MOV      ECX,[004B7FA4]
                       <--ECX="7558"
0167:004AB7B9  MOV      EDX,[EBP-04]
                       <--EDX="ShenGe"
0167:004AB7BC  CALL     00403F5C
                       <--字符串连接
0167:004AB7C1  MOV      EDX,[EBP-20]
                       <--EDX="ShenGe  7558"
0167:004AB7C4  XOR      ECX,ECX
0167:004AB7C6  POP      EAX
                       <--EAX="52093241386514845512"
0167:004AB7C7  CALL     0049FAC0
                       <--关键的 Call,跟进
0167:004AB7CC  MOV      EAX,004B7FE4
0167:004AB7D1  MOV      EDX,[EBP-08]
0167:004AB7D4  CALL     00403CE4
---------------------------------------------------
0167:004AB7D9  LEA      EAX,[EBP-24]
0167:004AB7DC  PUSH     EAX
0167:004AB7DD  MOV      ECX,06
0167:004AB7E2  MOV      EDX,01
0167:004AB7E7  MOV      EAX,[004B7FE4]
                       <--EAX 中为用户名与注册码运算的值,
                       上面那个关键 Call 中[EBP-18]中的值
                       我得到的为:(内存中)
                       08 51 F1 15 E9 0C 90 F0 CF
0167:004AB7EC  CALL     00404118
0167:004AB7F1  MOV      EAX,[EBP-24]
                       <--即从上个 Call 得到的字中自
                       第 1 个位置取 6 个字节到 EAX 中，我得到的为
                       08 51 F1 15 E9 0C
0167:004AB7F4  MOV      EDX,[004B7FA8]
                       <--EDX="161863"
0167:004AB7FA  CALL     00404020
                       <--比较是否相等
0167:004AB7FF  JNZ      NEAR 004ABCB2
--------------------------------------------
这段程序功能详见上面跟进那个关键 Call 中注释，不再啰嗦了!

0167:004AB805  LEA      EAX,[EBP-28]
0167:004AB808  PUSH     EAX
0167:004AB809  MOV      ECX,06
0167:004AB80E  MOV      EDX,01
0167:004AB813  MOV      EAX,[004B7FE4]
0167:004AB818  CALL     00404118
0167:004AB81D  MOV      EAX,[EBP-28]
0167:004AB820  MOV      EDX,[004B7FA8]
0167:004AB826  CALL     00404020
0167:004AB82B  JNZ      004AB844
--------------------------------------------
0167:004AB82D  PUSH     DWORD 004B7FB0
0167:004AB832  MOV      ECX,03
0167:004AB837  MOV      EDX,07
0167:004AB83C  MOV      EAX,[EBP-08]
0167:004AB83F  CALL     00404118
0167:004AB844  LEA      EAX,[EBP-2C]
0167:004AB847  PUSH     EAX
0167:004AB848  MOV      ECX,06
0167:004AB84D  MOV      EDX,01
0167:004AB852  MOV      EAX,[004B7FE4]
0167:004AB857  CALL     00404118
0167:004AB85C  MOV      EAX,[EBP-2C]
0167:004AB85F  MOV      EDX,[004B7FA8]
0167:004AB865  CALL     00404020
0167:004AB86A  JNZ      004AB891
-----------------------------------------
0167:004AB86C  PUSH     DWORD 004B7FB4
0167:004AB871  LEA      EDX,[EBP-30]
0167:004AB874  MOV      EAX,[EBX+0310]
0167:004AB87A  CALL     0042FCE8
0167:004AB87F  MOV      EAX,[EBP-30]
0167:004AB882  MOV      ECX,04
0167:004AB887  MOV      EDX,01
0167:004AB88C  CALL     00404118
0167:004AB891  LEA      EAX,[EBP-34]
0167:004AB894  PUSH     EAX
0167:004AB895  MOV      ECX,06
0167:004AB89A  MOV      EDX,01
0167:004AB89F  MOV      EAX,[004B7FE4]
0167:004AB89A  MOV      EDX,01
0167:004AB89F  MOV      EAX,[004B7FE4]
0167:004AB8A4  CALL     00404118
0167:004AB8A9  MOV      EAX,[EBP-34]
0167:004AB8AC  MOV      EDX,[004B7FA8]
0167:004AB8B2  CALL     00404020
0167:004AB8B7  JNZ      004AB8C0
-----------------------------------------------
0167:004AB8B9  MOV      BYTE [004B7FA1],01
                       <--[004B7FA1]=01,置已注册成功标志?
0167:004AB8C0  LEA      EAX,[EBP-38]
0167:004AB8C3  PUSH     EAX
0167:004AB8C4  MOV      ECX,06
0167:004AB8C9  MOV      EDX,01
0167:004AB8CE  MOV      EAX,[004B7FE4]
0167:004AB8D3  CALL     00404118
0167:004AB8D8  MOV      EAX,[EBP-38]
0167:004AB8DB  MOV      EDX,[004B7FA8]
0167:004AB8E1  CALL     00404020
0167:004AB8E6  JNZ      004AB8F6
------------------------------------------------
0167:004AB8E8  MOV      EAX,[EBX+031C]
0167:004AB8EE  MOV      EDX,[EAX]
0167:004AB8F0  CALL     NEAR [EDX+CC]
0167:004AB8F6  LEA      EAX,[EBP-3C]
0167:004AB8F9  PUSH     EAX
0167:004AB8FA  MOV      ECX,06
0167:004AB8FF  MOV      EDX,01
0167:004AB904  MOV      EAX,[004B7FE4]
0167:004AB909  CALL     00404118
0167:004AB90E  MOV      EAX,[EBP-3C]
0167:004AB911  MOV      EDX,[004B7FA8]
0167:004AB917  CALL     00404020
0167:004AB91C  JNZ      004AB934
------------------------------------------------
0167:004AB91E  MOV      EAX,[EBX+031C]
0167:004AB924  MOV      EAX,[EAX+0208]
0167:004AB92A  MOV      EDX,004ABD64
0167:004AB92F  MOV      ECX,[EAX]
0167:004AB931  CALL     NEAR [ECX+34]
0167:004AB934  LEA      EAX,[EBP-40]
0167:004AB937  PUSH     EAX
0167:004AB938  MOV      ECX,06
0167:004AB93D  MOV      EDX,01
0167:004AB942  MOV      EAX,[004B7FE4]
0167:004AB947  CALL     00404118
0167:004AB94C  MOV      EAX,[EBP-40]
0167:004AB94F  MOV      EDX,[004B7FA8]
0167:004AB955  CALL     00404020
0167:004AB95A  JNZ      004AB970
------------------------------------------------
0167:004AB95C  MOV      EAX,[EBX+031C]
```
