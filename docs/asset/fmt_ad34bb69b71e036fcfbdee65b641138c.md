
# `ida` 6.1 局域网检测补丁

> 原文：[`www.pediy.com/kssd/pediy12/134981.html`](https://www.pediy.com/kssd/pediy12/134981.html)

代码:

```
0052C7C0  /$  81EC 9C030000                   SUB     ESP,39C
0052C7C6  |.  A1 F82B6200                   MOV     EAX,DWORD PTR DS:[622BF8]
0052C7CB  |.  33C4                         XOR     EAX,ESP
0052C7CD  |.  898424 98030000              MOV     DWORD PTR SS:[ESP+398],EAX
0052C7D4  |.  803D 146B6200 00             CMP     BYTE PTR DS:[626B14],0
0052C7DB  |.  0F85 EB000000                JNZ     idaq1.0052C8CC
0052C7E1  |.  8D0424                       LEA     EAX,DWORD PTR SS:[ESP]
0052C7E4  |.  50                          PUSH    EAX                         ; /pWSAData
0052C7E5  |.  6A 02                       PUSH    2                         ; |RequestedVersion = 2 (2.0.)
0052C7E7  |.  E8 FA700000                 CALL    <JMP.&WSOCK32.#115>          ; \WSAStartup
0052C7EC  |.  85C0                        TEST    EAX,EAX
0052C7EE  |.  74 17                       JE      SHORT idaq1.0052C807
0052C7F0  |>  32C0                        XOR     AL,AL
0052C7F2  |.  8B8C24 98030000             MOV     ECX,DWORD PTR SS:[ESP+398]
```

![](img/8b6e90fa5d14000bd17ec91023e36787.jpg)

![](img/e51bd7c9df97aa5f3fd60744a62794a6.jpg)

其实就是 patch 掉了 `WSAStartup` 函数后的跳转,附送源代码

上传的附件

| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) || Dialog.rar
|
