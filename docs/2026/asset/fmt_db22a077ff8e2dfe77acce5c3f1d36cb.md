
这种方式的优点是不会对系统或者通信本身产生任何伤害，因为它没有做任何劫持，而是实打实的解密了，解密的[原理](<https://bbs.kanxue.com/elink@6c0K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6T1L8%4g2F1k6r3q4J5P5g2)9J5c8Y4N6A6M7X3g2K6K9r3q4J5K9#2)9J5c8X3u0D9L8$3u0Q4x3V1k6E0j5i4y4@1k6i4u0Q4x3V1k6W2M7r3q4F1i4K6u0r3k6r3W2K6M7$3g2U0N6r3!0J5M7#2)9J5c8Y4m8S2j5$3E0W2N6q4)9J5k6s2y4K6L8q4)9J5k6s2g2@1K9h3I4K6i4K6u0W2j5#2)9J5x3@1H3@1x3e0j5@1>)是利用了[NSS Key Log](<https://bbs.kanxue.com/elink@6f3K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1k6i4k6W2L8r3!0H3k6i4u0Q4x3X3g2E0L8%4A6A6L8r3I4S2i4K6u0W2L8%4u0Y4i4K6u0r3k6h3&6Q4x3X3c8g2f1#2)9J5c8X3c8G2j5%4y4Q4x3V1k6y4L8%4A6A6L8r3I4S2i4K6u0r3f1s2u0G2K9X3g2U0N6s2y4Q4x3V1k6z5f1#2y4Q4x3V1k6w2k6i4W2Q4y4h3k6x3L8$3N6Q4y4h3k6r3L8%4u0E0j5i4b7>)，对于启用了`SSL-Pinning`的客户端效果更好。缺点是无法指定进程抓包，而且操作起来有一定的不便行，在分析客户端的过程中，一旦有一个新的域名出现，就需要先用Chrome访问一遍才能解密。

本文将采用第二种方法来进行抓包。

> Proxifier+Charles

Charles使用https中间人劫持技术，所以它同样可以解密https流量，但是它只对使用了`wininet`系列api的流量有效，而我们的网盘客户端使用的是`libcurl`，所以需要借助Proxifier来将客户端的流量重定向到Charles，从而实现https解密。

- 添加Proxifier的Proxy Server

  ![Proxy Servers](https://bbs.kanxue.com/upload/attach/202009/903028_7DWKT4DGHU8GBW8.png)

- 添加Proxifier的Proxification Rules

  ![Proxification Rules](https://bbs.kanxue.com/upload/attach/202009/903028_TNS8PTW52CM8T96.png)

- Charles启用SSL代理

  ![SSL Proxying Settings](https://bbs.kanxue.com/upload/attach/202009/903028_VBC54KR8XKFY4E4.png)

到这里，我们的环境准备工作就做完了，只要启动客户端就能看到成功解密的https流量。不过由于客户端默认以管理员权限运行，为了避免不必要的麻烦，还是使用Resource Hacker修改下manifest文件，以普通用户运行。

![Resource Hacker](https://bbs.kanxue.com/upload/attach/202009/903028_2TAS3Z68KX994K4.png)

修改高亮部分由`requireAdministrator`修改为`asInvoker`，然后点击工具栏的编译按钮，最后保存即可。

到了激动人心的时刻，启动客户端！

![启动客户端](https://bbs.kanxue.com/upload/attach/202009/903028_ZBDU7HDST242CZX.png)

## 登录

输入账号密码，点击登录按钮，我们看到了这样的数据

![登录请求头](https://bbs.kanxue.com/upload/attach/202009/903028_ZPSY75WR9XZF8XD.png)

![登录请求数据](https://bbs.kanxue.com/upload/attach/202009/903028_NY6FD9XFGPQCUF3.png)

`app_id`看起来应该是标识不同渠道的，很可能是固定值；`password`是32位16进制字符串，猜测是密码的md5值，我找了个工具试了下，结果还真是-v-；`timestamp`是毫秒单位的时间戳，这个很明显，没什么好说的；`platfrom`看起来是系统信息拼接的字符串，虽然信息比较多，但是这里也能看出个大概，`[app_id]+[exe version]+[mac]+pc+[computer name]+[system version]`，当然尚不能完全确定，暂时先放一边；`sign`和`password`类似，看起来也是md5值，但具体是谁的md5值就不得而知了，尝试了把所有参数拼接为字符串计算md5结果也不对。

到这里，从抓包的角度来说，我们已经走不下去了，因为`platform`和`sign`参数搞不定的话流程就断了。

## 签名算法分析

是时候了，来吧！

### 定位函数位置

为了方便分析，首先将客户端修改为静态基址。

![静态基址](https://bbs.kanxue.com/upload/attach/202009/903028_Z7SPRGHHWTR5CCG.png)

这样，我们就可以将OD的基址和IDA的基址设置为一致，即使重启进程也不用重新设置了。

![Rebase Section](https://bbs.kanxue.com/upload/attach/202009/903028_SQ9ZMKREFNHD58A.png)

通过前面的分析，我们知道登录地址为`/loginServer/login`，在IDA中所搜字符串，只有一处即`aHttpsPanApiBit`，按`x`查找引用，仍然只有一处（由于我已经分析过，所以改了函数名）。

![aHttpsPanApiBit](https://bbs.kanxue.com/upload/attach/202009/903028_GRB2F56AH5DZ6G3.png)
