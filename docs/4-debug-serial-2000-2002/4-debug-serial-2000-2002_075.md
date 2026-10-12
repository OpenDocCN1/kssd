
# Macro Magic 4.1

> 原文：[`www.pediy.com/kssd/pediy02/forum123.htm`](https://www.pediy.com/kssd/pediy02/forum123.htm)

**软件名称**：`Macro Magic 4.1N`
**注册方式**：注册码

首先请出我们的工具 `Trw2000`

下指令 `bpx hmemcpy`

下指令 `bd *`

下指令 `pmodule`

来到这里

```
015F:0045D577 8B45D8                MOV       EAX,[EBP-28]          这里放你的输入的注册码

015F:0045D57A 50                    PUSH      EAX

015F:0045D57B FF75F8                PUSH      DWORD [EBP-08]

015F:0045D57E FF75F4                PUSH      DWORD [EBP-0C]

015F:0045D581 8D45E4                LEA       EAX,[EBP-1C]

015F:0045D584 E807050000            CALL      0045DA90

015F:0045D589 8B55E4                MOV       EDX,[EBP-1C]

015F:0045D58C 58                    POP       EAX                   这里放正确的注册码

015F:0045D58D E81E68FAFF            CALL      00403DB0

015F:0045D592 0F8551010000         JNZ       NEAR 0045D6E9
```

不到两分钟，就破了。是不是很简单。

`4dd466d0e1e0cc123afbe504da2462ca`
