
## 5.4.

```
1001CC58    | C3                                 | ret                                                                                                                                   |
```

```
0018A504    | 90                                 | nop                                                                                                                                   |
0018A505    | 90                                 | nop                                                                                                                                   |
0018A506    | 90                                 | nop                                                                                                                                   |
0018A507    | 90                                 | nop                                                                                                                                   |
0018A508    | 90                                 | nop                                                                                                                                   |
0018A509    | 90                                 | nop                                                                                                                                   |
0018A50A    | 90                                 | nop                                                                                                                                   |
0018A50B    | 90                                 | nop                                                                                                                                   |
0018A50C    | 90                                 | nop                                                                                                                                   |
0018A50D    | 90                                 | nop                                                                                                                                   |
0018A50E    | 90                                 | nop                                                                                                                                   |
0018A50F    | 90                                 | nop                                                                                                                                   |
0018A510    | 90                                 | nop                                                                                                                                   |
0018A511    | 90                                 | nop                                                                                                                                   |
0018A512    | 90                                 | nop                                                                                                                                   |
0018A513    | 90                                 | nop                                                                                                                                   |
0018A514    | 90                                 | nop                                                                                                                                   |
0018A515    | 90                                 | nop                                                                                                                                   |
0018A516    | 90                                 | nop                                                                                                                                   |
0018A517    | 90                                 | nop                                                                                                                                   |
0018A518    | DA D7                              | fcmovbe st(0),st(7)                                                                                                                   |
0018A51A    | D9 74 24 F4                        | fnstenv m28 ptr ss:[esp-C]                                                                                                            ......
```

不过样本`python`脚本中有个`nop_rop_chain_1`,在上面的流程中没有出现,改成`AAA...`也不影响.

主要是分析了下思路,实际自己去做肯定是要很多尝试的

![](https://bbs.kanxue.com/upload/attach/201806/761984_KTR27HY6BUZHWY5.jpg)

软件和脚本下载

524K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2W2P5s2m8D9L8$3W2@1i4K6u0V1k6r3u0Q4x3X3g2U0L8$3#2Q4x3V1k6W2P5s2m8D9L8$3W2@1M7#2)9J5c8U0b7@1y4U0f1^5i4K6u0r3

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于 2019-4-12 10:19

被树梢之上编辑

，原因：
