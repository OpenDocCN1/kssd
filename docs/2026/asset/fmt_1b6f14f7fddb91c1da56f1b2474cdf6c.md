
由于没抽取，所以脱壳还是比较简单，这里说一句，本身用来加载的`dex`的落地文件是不规则的，个人猜测是通过`fread`和`fwrite`进行解密的，但是没去验证。

执行完`sub_15928`之后，对`maps`下`classes.jar`的地址进行`dump`就可以了。

![图片描述](https://bbs.kanxue.com/upload/attach/202103/762912_ZHNBBF3ATFQTFXG.png)

## 0x6 一些检测

`sub_17218`:签名校验

![图片描述](https://bbs.kanxue.com/upload/attach/202103/762912_5QPRQSVH7P9PE7M.png)

`sub_18DF4`:Hook Android日志

![图片描述](https://bbs.kanxue.com/upload/attach/202103/762912_762912_D3E4AK6FPXR7UFT.png)

`sub_49794`:检测FART

![图片描述](https://bbs.kanxue.com/upload/attach/202103/762912_PQAQDVWJ9UG2N2B.png)

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

最后于 2021-3-10 10:47

被GitRoy编辑，原因：修正一个错误

[#脱壳反混淆](https://bbs.kanxue.com/forum-161-1-122.htm)

上传的附件：

- [upx修复.pdf](javascript:void(0);)
  （988.83kb，353次下载）
- [IDA脚本.zip](javascript:void(0);)
  （41.86kb，190次下载）
- [test.apk](javascript:void(0);)
  （3.29MB，197次下载）
- [libSecShell_fix.so.zip](javascript:void(0);)
  （3.54MB，167次下载）
