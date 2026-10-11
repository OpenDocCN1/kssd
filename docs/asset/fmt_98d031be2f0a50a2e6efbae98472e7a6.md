
# 本机突破.net Framework 强名称验证 1.1

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-33.htm`](https://www.pediy.com/kssd/pediy08/pediy8-33.htm)

由于`.net`下没有动态调试工具，所以很多时候都是修改文件，加入自己的代码来执行（比如在特定位置`dump`）。但是，对于强名称验证，修改将导致验证失败。如果删除强名称或者用其他密钥对签名，会导致`publickey`不同，程序里一个`AssemblyName.GetPublicKey`就死翘了。

所以，能蒙混过系统的验证最好。`StrongName`强名称相关的操作在`mscorsn.dll`里，里面有三个导出函数是验证相关的：

- `StrongNameSignatureVerification`
- `StrongNameSignatureVerificationEx`（调用`StrongNameSignatureVerification`）
- `StrongNameSignatureVerificationFromImage`

为了保险起见，把`StrongNameSignatureVerification`和`StrongNameSignatureVerificationFromImage`都patch掉:

`StrongNameSignatureVerification`

```
xor eax,eax
inc eax
retn 0ch
```

`StrongNameSignatureVerificationFromImage`

```
xor eax,eax
inc eax
retn 10h
```

保存，替换原来的`mscorsn.dll`。运行一个修改过的具有强名称的`.net`可执行文件，成功～ 记得没事的时候要改回去哦，改回去要安全一些的。

有什么错误缺漏还请大虾们指正，谢谢。

注 以上环境为`.net`Framework 1.1

Bi11\[CCG\]

1 Feb 2006

4dd466d0e1e0cc123afbe504da2462ca
