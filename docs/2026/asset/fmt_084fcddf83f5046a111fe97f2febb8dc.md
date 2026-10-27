
# [原创]某浏览器和输入法软件base64加解密

没啥可说的，主要是体力劳动。调试器挂载跟踪`recv()`函数的处理流程，稍嫌麻烦，即可得到解密算法。

总共有3步：

1. url编码解码
2. base64解码
3. 异或解密

异或算法如下：

```c
void decrypt(char * data, int filesize, char * dst) {
    char * key = "Z#jM0NeVv#wMDG9+8rwzxVsti80A=j5a.op";
    if (dst == 0)
    {
        dst = new char[filesize + 1024];
        memset(dst, 0, filesize + 1024);
    }
    int i = 0, j = 0;
    for (; i < filesize; i++, j++)
    {
        int k = j % 0x23;
        dst[i] = data[i] ^ key[k];
    }
}
```

c++代码和样本、以及程序都在附件压缩包中

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于 2024-2-3 15:56 被 satadrover 编辑，原因：

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)

上传的附件：

- [`test.rar`](javascript:void(0);)（5.87MB，33次下载）
