
# 斗地主 1.0

> 原文：[`www.pediy.com/kssd/pediy03/forum294.htm`](https://www.pediy.com/kssd/pediy03/forum294.htm)

斗地主 1.0 版

- 下载地址：`http://www.newhua.com.cn/down/LandOwner.zip`
- 破解难度：太容易 (下载了之后才知道)
- 破解关键：运行 `trw` 后，运行“斗地主”，填入注册信息之后，下：

```
:bpx hmemcpy
:g
```

之后，按 `F12` 十数次来到如下代码：

```
015F:0045E7A2  MOV      EAX,[EAX]       //d eax (你输入的注册码)
015F:0045E7A4  MOV      EDX,[EBP-04]     //d edx (正确的注册码)
015F:0045E7A7  CALL     00403C20         //注册码比较部分
015F:0045E7AC  JZ       0045E7BA         //不跳则 game over
015F:0045E7AE  MOV      EAX,[0046096C]
015F:0045E7B3  CALL     0043317C
015F:0045E7B8  JMP      SHORT 0045E7C6
015F:0045E7BA  MOV      EAX,[00460954]
015F:0045E7BF  CALL     0043317C
```

拿笔抄下注册码，去注册吧！

注意：这种保护太简单，注册码直接可在内存中找出！它的破解没什么大不了的，只要有耐心，初学者就可以搞定 :)

注册信息保存在注册表：

`HKEY_CURRENT_USER/SoftWare/Littlegrass/LandOwner`

十三少
China Cracking Group

2000.09.05 4dd466d0e1e0cc123afbe504da2462ca
