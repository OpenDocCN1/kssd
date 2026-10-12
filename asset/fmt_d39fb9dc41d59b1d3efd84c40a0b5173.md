
# 答案提交 by lelfei

> 原文：[`www.pediy.com/kssd/pediy12/133327.html`](https://www.pediy.com/kssd/pediy12/133327.html)

## 第 1 题

原始内容：`style=background-image:url(javascript:alert(String.fromCharCode(88,83,83)));//`

加密密钥：`testtesttesttesttest`

## 第 2 题

IE 下：

```
<BODY BACKGROUND="javascript:alert('XSS')">
```

## 第 3 题

网页地址包含字符串"`bho0day`"

## 第 4 题

单机测试时受文件名长度限制，基本放弃

## 第 5 题

环境没搭出来，基本放弃

## 第 6 题

调用 `VidSetTextColor` 并将返回值写入到 `Irp.UserBuffer` 时没有验证该地址，因此可以使用这个调用向任意地址写入数据

## 第 7 题

可以绕过内存保护直接向任意物理内存地址写入数据

上传的附件

| ![](img/67af4f1ef9c98996f43e89e749c7dcf5.jpg) || lelfei.rar |
