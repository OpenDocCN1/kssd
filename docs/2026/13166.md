
# [原创]企业wx数据之初探秘

前面看多了大家分析个人wx数据的相关信息。

想起来看看企业wx的是否可以同样搞搞。

1. 于是思路简单些，先在`od`里面搜索`database`字符串，看是否有发现。

   果然发现一处：

   ![](https://bbs.kanxue.com/upload/attach/202004/580134_7RWPAJR9GAGFRUK.jpg)

   再看此处的汇编，发现是这样的：

   ![](https://bbs.kanxue.com/upload/attach/202004/580134_YGUERZV7QC4HMR2.jpg)

2. 看起来很像是打开数据库的地方，寻着此处往上在函数顶下断。

   然后，运行，扫码登录，在断下后，单步运行。

   发现堆栈中有数据库的路径：

   ![](https://bbs.kanxue.com/upload/attach/202004/580134_9RHUY3VG8KBPXYP.jpg)

3. 继续往下跟，发现一处有路径用到的地方：

   ![](https://bbs.kanxue.com/upload/attach/202004/580134_WMRFXPRBYA5TFT3.jpg)

   `ecx`数据是这样：

   ![](https://bbs.kanxue.com/upload/attach/202004/580134_7C9W9HHS7BCFU2P.jpg)

4. 继续单步往下，运行完后，`eax`里面是这样：

   ![](https://bbs.kanxue.com/upload/attach/202004/580134_WTQ8QQ3FEK7DZZW.jpg)

   点进去去看后看起来像个结构体：

   ![](https://bbs.kanxue.com/upload/attach/202004/580134_DG8YKJZ3YSVES6C.jpg)

   很像存放了数据库句柄的。

5. 继续一路往下，直到要跳过`Open database failed.`字符串处。发现比较了`cmp dword ptr ds:[edi+0x4],0x0`，可能在`edi+0x4`里面存了句柄之类，`edi+0x4`里面的内容和上面的`eax`一样：

   ![](https://bbs.kanxue.com/upload/attach/202004/580134_VCGGTHT2WZHTDMA.jpg)

   `jmp`后，此函数基本结束。

6. `F9`后，会再次断下在此`call`，不过是个新的数据库路径。

   观察刚才疑似密钥处，发现：

   ![](https://bbs.kanxue.com/upload/attach/202004/580134_W52XRWJH4UAJUR6.jpg)

   这个密钥似乎是16位的，而个人版的是32位的。

7. 取了这个密钥用个人版本的解密代码稍作修改后尝试。

   果然解密成功。是sqlit的数据库：

   ![](https://bbs.kanxue.com/upload/attach/202004/580134_WUFUQXQEMD8EH47.jpg)

   解密的是`user.db`，不过可惜，这个数据需要链接密码才可以使用，不能直接查看。

8. 本来想解密直接看看数据库的用户好友信息，看来失败告终。

9. 查看用户数据目录下面，发现它的有些`log`表是没有加密的，可以直接打开，也不要解密。

   ![](https://bbs.kanxue.com/upload/attach/202004/580134_MVQPKAVP8XQK69F.jpg)

至此，数据库研究告一段落。

企业wx逆向难度比较大，比个人版的大很多。有兴趣的小伙伴可以搞搞。也可以一起交流下。目前我还在研究中。

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

最后于 2021-10-15 11:07 被zzg令狐编辑，原因：[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)
