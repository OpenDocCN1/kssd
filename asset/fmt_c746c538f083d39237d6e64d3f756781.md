
# OFFLINE EXPLORER 1.0

> 原文：[`www.pediy.com/kssd/pediy03/forum998.htm`](https://www.pediy.com/kssd/pediy03/forum998.htm)

《 OFFLINE EXPLORER 1.0 》的注册码破解   

目标：OFFLINE EXPLORER 1.0   

一个新的且容易使用的离线浏览软件，速度比 `Teleport` 快，且支持拖放。`Offline Explorer` 把网站上网页整个拖回来是一件既方便又经济实惠的作法，特别是阅读网上的电子读物时更加重要。现在流行的 `Teleport` 最大只能有十个进程，而 `WebZIP` 功能虽强，但 BUG 多多，新兴离线浏览器 `Offline Explorer` 不但没有了十个进程的限制，而且界面优秀一级棒，功能多多。 

作者：`RATARICE[BCG]` 

工具：`TRW2000` 1.22 

### 过程

一、 运行软件，点 `help-->about offline explorer-->register`。 

填上 `registration user:RATARICE[BCG]` 

`registration keys:87654321` （瞎填的！） 

点“OK”，弹出错误框。 

二、 软件的分析到此为止，启动 `TRW2000`！ 

下 `bpx hmemcpy`。 

再点“OK”，被拦。 

下 `bd *`；`pmodule`；按 7 次 `F12`（8 次出错）。代码如下： 

```
015F:004AA461 CALL 00433784
015F:004AA466 MOV EAX,[EBP-04]
015F:004AA469 PUSH EAX
015F:004AA46A LEA EDX,[EBP-08]
015F:004AA461 CALL 00433784
015F:004AA466 MOV EAX,[EBP-04]
015F:004AA469 PUSH EAX
015F:004AA46A LEA EDX,[EBP-08]
015F:004AA46D MOV EAX,[EBX+02E0]
015F:004AA473 CALL 00433784
015F:004AA478 MOV EDX,[EBP-08]
015F:004AA47B MOV EAX,[004BBA38]
015F:004AA480 MOV EAX,[EAX]
015F:004AA482 POP ECX
015F:004AA483 CALL 004B1C10 -------------->关键，F8 进入。
015F:004AA488 TEST AL,AL
015F:004AA48A JZ 004AA4F6 -------------->跳则死，上面有问题。
```

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\* 

进入后，按 `F10` 一直到下面： 

```
015F:004B1C75 MOV EAX,[EBP-08]
015F:004B1C78 CALL 00403FDC
015F:004B1C7D SETZ BL
015F:004B1C80 XOR EAX,EAX
015F:004B1C82 POP EDX
015F:004B1C83 POP ECX
015F:004B1C84 POP ECX
015F:004B1C85 MOV [FS:EAX],EDX
015F:004B1C88 PUSH DWORD 004B1CA2
015F:004B1C8D LEA EAX,[EBP-10]
015F:004B1C90 MOV EDX,04
015F:004B1C75 MOV EAX,[EBP-08]
015F:004B1C78 CALL 00403FDC
015F:004B1C7D SETZ BL
015F:004B1C80 XOR EAX,EAX
015F:004B1C82 POP EDX
015F:004B1C83 POP ECX
015F:004B1C84 POP ECX
015F:004B1C85 MOV [FS:EAX],EDX
015F:004B1C88 PUSH DWORD 004B1CA2
015F:004B1C8D LEA EAX,[EBP-10]
015F:004B1C90 MOV EDX,04
015F:004B1C95 CALL 00403C64 ------------------>此处有问题，按 F8。
015F:004B1C9A RET
```

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\* 

进入后看到： 

```
015F:00403C62 MOV EAX,EAX
015F:00403C64 PUSH EBX
015F:00403C65 PUSH ESI
015F:00403C66 MOV EBX,EAX
015F:00403C68 MOV ESI,EDX
015F:00403C6A MOV EDX,[EBX]
015F:00403C6C TEST EDX,EDX ------------------->下 d edx 可看到真的注
015F:00403C6E JZ 00403C89  册码！
015F:00403C70 MOV DWORD [EBX],00
```

### 三、总结

`registration user:RATARICE[BCG]` 

`registration keys:564363-515233443-JH`  `4dd466d0e1e0cc123afbe504da2462ca`
