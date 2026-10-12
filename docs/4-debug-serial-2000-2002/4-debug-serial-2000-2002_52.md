
# EXESCOPE5.12

> 原文：[`www.pediy.com/kssd/pediy01/forum038.htm`](https://www.pediy.com/kssd/pediy01/forum038.htm)

第一次写破解文章，不对之处还望指教。

工具：`TRW2000`

目标：`EXESCOPE5.12`

这是一个功能强大的汉化软件，先用 `TRW2000` `LOAD` 入 `EXESCOPE`，只要你找到下面的代码：

```
:00484A6F CALL 00403BA4
       CMP EAX, BYTE+0A     第一位是否为 A
       JNZ 00484AA0
       MOV EAX, [EBP-04]
       CMP BYTE [EAX], 41   是否为 A
       JNZ 00484AA0
       MOV EAX, [EBP-04]
       MOVZX EAX, BYTE [EAX+08] 取第九位
       MOV EDX, [EBP-04]
       MOVZX EDX, BYTE [EDX+09] 取第十位
       ADD EAX, EDX         第九位加第十位
       MOV ECX, 0A          此处注意 ECX=？？
       CDQ
       IDIV ECX             除 ECX
       CMP EDX, BYTE+04     余数是否为 4
       JNZ 00484AA0
```

从上可知密码为：`AXXXXXXX？？`。只要第一位为 `A`，第九位加第十位除 `ECX` 余数为 `4` 即可注册，名字可随便填，只要第九位加第十位等于十进制的 `8`，就可注册。
