
# Cpukiller 2.0

> 原文：[`www.pediy.com/kssd/pediy03/forum323.htm`](https://www.pediy.com/kssd/pediy03/forum323.htm)

献给初学者 破解 `Cpukiller 2.0`

软件名称：`Cpukiller 2.0`

软件大小：`500kb`

软件授权：共享软件

使用平台：`Win95/98/NT`

下载地址：自己找吧，哈哈

软件简介：能减慢某些程序运行太快的缺点!

解密日前：2000 年 9 月 17 日

解密工具：`Trw2000 1.22`

用 `bpx hmemcpy` 拦下，下 `pmodule`

从这里出来......

```
0167:0043F4E7  MOV      EAX,[EBP-14]----------载入姓名
0167:0043F4EA  LEA      EDX,[EBP-10]
0167:0043F4ED  CALL   0043F114--------------计算注册码
0167:0043F4F2  MOV      EAX,[EBP-10]----------过了这一行
0167:0043F4F5  PUSH     EAX-------------------下 d eax 可见注册码
0167:0043F4F6  LEA      EDX,[EBP-14]
0167:0043F4F9  MOV      EAX,[EBP-04]
0167:0043F4FC  MOV      EAX,[EAX+0214]
0167:0043F502  CALL   0041C3B0--------------比较注册码
0167:0043F507  MOV      EDX,[EBP-14]
0167:0043F50A  POP      EAX
0167:0043F50B  CALL   00403D24
0167:0043F510  JNZ      NEAR 0043F5BA
0167:0043F516  mov      dl, 01
0167:0043F518  mov      eax, dword ptr [0043B828]
0167:0043F51D  call   0043B95C
0167:0043F522  mov      dword ptr [ebp-08], eax
........
```

怎么样，简单吗?练练手吧! (我只用了 1 分钟不到，哈哈)不过写可残了

姓名:`gce` 注册码:`KASAKIWMIASISKIA`

!有没有高手能写一下它的注册码生成过程!

欢迎光临:`http://www.luosz.cn.gs`

与我联系:`gcehi@kali.com.cn` `4dd466d0e1e0cc123afbe504da2462ca`
