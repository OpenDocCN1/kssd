
# Security Administrator 7.1

> 原文：[`www.pediy.com/kssd/pediy03/forum1178.htm`](https://www.pediy.com/kssd/pediy03/forum1178.htm)

今天工作不太忙，继续我的作业。呵呵，不然`FCG`的老大说我作业通不过那可惨了！

连上华军软件园里，来到最近更新，发现目标：`This is "Security Administrator 7.1"`一个安全工具。呵呵

URL：`http://jx163.onlinedown.net/down/secagent.zip`

软件不大，只人八百多`K`

## 第一步

先得安装。呵呵。

## 第二步

运行软件。马上就跳出了要你注册的画面。得，注册就注册吧。写入注册码`"456456456"`，点`"register"`小弟 E 文不好，但直觉认为这个是注册的按钮。：）因为他离写注册码的地方最近。呵呵。

跳出出错的对话框。嘿嘿，有出错的对话框就成。呵呵。下面在`SICE`中下断点：`bpx hmemcpy`。

回到程序后，点那个`"register"`，马上被拦。老样子，按`F12`连按 12 下，来到了这里：

```asm
0167:004888B6  MOV      EAX,[EBP-08]    <---这里 EAX 是我们的假注册码
0167:004888B9  LEA      EDX,[EBP-04]
0167:004888BC  CALL     0046FF44
0167:004888C1  CMP      DWORD [EBP-04],BYTE +00
0167:004888C5  JNZ      004888D4
0167:004888C7  LEA      EAX,[EBP-04]
0167:004888CA  MOV      EDX,00488920
0167:004888CF  CALL     00403D20
0167:004888D4  MOV      EAX,[004B5AA4]
0167:004888D9  MOV      EAX,[EAX]
0167:004888DB  MOV      EDX,[EBP-04]
0167:004888DE  CALL     0049CEE8    <----此 call 一按 F10 就死了。没得玩了。得跟进去看看
```

结果那个`AL`一改为`1`就注册成功了，但在第二次启动之时。。。。。。。还是没注册的，得，又得花时间再看看。

就到那个改`AL`值上的那个`call`里去转转，说不定有收获呀。：）

```asm
0167:0049CEE8  PUSH     EBP
0167:0049CEE9  MOV      EBP,ESP
0167:0049CEEB  PUSH     ECX
0167:0049CEEC  PUSH     EBX
0167:0049CEED  MOV      [EBP-04],EDX
0167:0049CEF0  MOV      EBX,EAX
0167:0049CEF2  MOV      EAX,[EBP-04]
0167:0049CEF5  CALL     004040BC
0167:0049CEFA  XOR      EAX,EAX
0167:0049CEFC  PUSH     EBP
0167:0049CEFD  PUSH     DWORD 0049CF80
0167:0049CF02  PUSH     DWORD [FS:EAX]
0167:0049CF05  MOV      [FS:EAX],ESP
0167:0049CF08  MOV      EDX,[EBP-04]
0167:0049CF0B  MOV      EAX,EBX
0167:0049CF0D  CALL     00495F50
0167:0049CF12  TEST     AL,AL      <---这里看一下，啊呀，AL 的值是 0 呀，改 1 得了
0167:0049CF14  JZ       0049CF52
```

就在这个位置继续跟踪：

```asm
0167:00495F50  PUSH     EBP
0167:00495F51  MOV      EBP,ESP
0167:00495F53  PUSH     ECX
0167:00495F54  PUSH     EBX
0167:00495F55  PUSH     ESI
0167:00495F56  MOV      [EBP-04],EDX
0167:00495F59  MOV      ESI,EAX
0167:00495F5B  MOV      EAX,[EBP-04]
0167:00495F5E  CALL     004040BC
0167:00495F63  XOR      EAX,EAX
0167:00495F65  PUSH     EBP
0167:00495F66  PUSH     DWORD 004960BC
0167:00495F6B  PUSH     DWORD [FS:EAX]
0167:00495F6E  MOV      [FS:EAX],ESP
0167:00495F71  XOR      EBX,EBX
0167:00495F73  MOV      BYTE [004B6E18],00
0167:00495F7A  MOV      EAX,004B6E1C
0167:00495F7F  CALL     00403C88
0167:00495F84  CMP      DWORD [EBP-04],BYTE +00
0167:00495F88  JZ       NEAR 004960A6
0167:00495F8E  MOV      EAX,[EBP-04]
0167:00495F91  CALL     00403F08
0167:00495F96  CMP      EAX,BYTE +0C  <--一直来到这里，一看原来注册码要大于 12  得，跳出去重输吧。再继续下去
0167:00495F99  JZ       00495FA7
0167:00495F9B  MOV      EAX,[EBP-04]
0167:00495F9E  CMP      BYTE [EAX],38  <--把注册码的第一位与 ASCII 的 38 比，也就是与数字 8 比
0167:00495FA1  JNZ      NEAR 004960A6 <--不是就跳到死亡地带！！
0167:00495FA7  MOV      EAX,[EBP-04]
0167:00495FAA  CMP      BYTE [EAX],38
0167:00495FAD  JNZ      NEAR 004960A6
0167:00495FB3  MOV      EAX,[EBP-04]
0167:00495FB6  CMP      BYTE [EAX+02],33  <--把注册码的第三位与 ASCII 的 33 比，也就是与数字 3 比
0167:00495FBA  JNZ      NEAR 004960A6  <--不是就跳到死亡地带！！
0167:00495FC0  MOV      EAX,[EBP-04]
0167:00495FC3  CMP      BYTE [EAX+03],31  <--把注册码的第四位与 ASCII 的 31 比，也就是与数字 1 比
0167:00495FC7  JNZ      NEAR 004960A6  <--不是就跳到死亡地带！！
0167:00495FCD  MOV      EAX,[EBP-04]
0167:00495FD0  CMP      BYTE [EAX+04],39  <--把注册码的第五位与 ASCII 的 39 比，也就是与数字 9 比
0167:00495FD4  JNZ      NEAR 004960A6  <--不是就跳到死亡地带！！
0167:00495FDA  MOV      EAX,[EBP-04]
0167:00495FDD  CMP      BYTE [EAX+08],30  <--把注册码的第九位与 ASCII 的 30 比，也就是与数字 0 比
0167:00495FE1  JNZ      NEAR 004960A6  <--不是就跳到死亡地带！！
0167:00495FE7  MOV      EAX,[EBP-04]
0167:00495FEA  CMP      BYTE [EAX+09],35  <--把注册码的第十位与 ASCII 的 35 比，也就是与数字 5 比
0167:00495FEE  JNZ      NEAR 004960A6  <--不是就跳到死亡地带！！
0167:00495FF4  MOV      EAX,[EBP-04]
0167:00495FF7  CMP      BYTE [EAX+0A],53  <--把注册码的第十一位与 ASCII 的 53 比，也就是与字母 S 比
0167:00495FFB  JNZ      NEAR 004960A6  <--不是就跳到死亡地带！！
0167:00496001  MOV      EAX,[EBP-04]
0167:00496004  CMP      BYTE [EAX+0B],45  <--把注册码的第十二位与 ASCII 的 45 比，也就是与字母 E 比
0167:00496008  JNZ      NEAR 004960A6  <--不是就跳到死亡地带！！
0167:0049600E  MOV      EAX,004B6E1C
0167:00496013  MOV      EDX,[EBP-04]
0167:00496016  CALL     00403CDC
0167:0049601B  MOV      BYTE [004B6E18],01
0167:00496022  MOV      BL,01
0167:00496024  XOR      EDX,EDX
0167:00496026  MOV      EAX,[ESI+04F4]
```

记一下，注册码的这几位是：`8?319???05SE`，哎，那还有几位呢？？？我再找！一直翻了好几遍都找不到，没办法，只好先把这么位写上先：`8031900005SE`，一点注册，呵呵，成了，原来他只判断这几位就成了。其他的，你想怎么填都可以。呵呵。要暴破的话，将以上`JNZ`的跳转改成不跳就成了。还有，将那几位的`ASCII`码改成我们喜欢的数字也成。呵呵。因为这个软件也没有特殊的`ID`或用户名之类的东东，觉得可能是只有唯一的注册码。

不知小弟的过程有没有错，请各位大虾指点指点。

`My oicq:8895546`

`My Email:plycy@8848.net 4dd466d0e1e0cc123afbe504da2462ca`
