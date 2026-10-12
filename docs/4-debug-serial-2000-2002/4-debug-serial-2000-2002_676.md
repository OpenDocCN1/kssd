
# 商务邮件专家 2.0

> 原文：[`www.pediy.com/kssd/pediy03/forum584.htm`](https://www.pediy.com/kssd/pediy03/forum584.htm)

## 作者信息
- 作者：孙 锋
- Email：`sffs@263.net`
- 主页：`http://sffs.6to23.com`

## 工具
`SoftIce 4.05`

## 步骤
1. 随便输入注册码：`1212121212`。
2. `Ctrl+D` 调出 `SoftIce`，下 `bpx hmemcpy`。
3. `F5` 返回程式，点确定，被 `SoftIce` 拦截到。
4. `F12` 共 12 次，然后一直 `F10` 到下面的位置：

```assembly
0167:004BF3BD  MOV EDX,[EDX+00000D3C]
0167:004BF3CC  CALL 004042E8     ;---------->D EDX 得到正确注册码：+6T)L_0n
0167:004BF3C8  JNZ 004BF446
```

5. 整理重新注册，成功升级为正式版。

`<---------完--------> 4dd466d0e1e0cc123afbe504da2462ca`
