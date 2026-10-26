
- 运行，输入`name`，输出相应的序列号，测试，成功

  ![image-20200215180624176](https://bbs.kanxue.com/upload/attach/202002/868592_GGS5D38F3U33P96.jpg)

## 2.6-网络验证

正确序列号使用一段时间后就会报错，因为其远程连接服务器进行了网络验证

1. 标志位判断，直接`JMP`跳过

   ![image-20200215180644023](https://bbs.kanxue.com/upload/attach/202002/868592_NTBWWZNKKURF7DX.jpg)

2. 网络验证函数内部，使其一直返回1（也就是验证正确才返回的值

   ![image-20200215180700592](https://bbs.kanxue.com/upload/attach/202002/868592_B3PXCYF4WZAFC39.jpg)

# 03-总结

1. 边分析边测试的方法
2. 逆向真的是体力活，说难也没太难，就得有耐心
3. 要细心，失之毫厘，差之千里

# 04-参考

1. 15PB视频课程-逆向工程实战之软件算法分析

PS：流水账的写法，图片说明较少，简单记录为日后翻看方便

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于 2020-2-16 14:36 被21Gun5编辑，原因：[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)
