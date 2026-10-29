
### 小结

登录接口的`userpwd`和`_sign`已经分析完了。比较简单，就一个大写`md5`，还没有任何魔改，熟悉算法的佬应该很快就反应过来的。可惜俺不是。

### 前言

这次分享一个iOS端*车之家的登录接口的参数分析（仅供学习，严禁干坏事=。=）。本文为新手项，大佬请跳过。

### 准备

- iOS 12.5.5
- `frida` 14.0.0
- `ipa` 5rG96L2m5LmL5a62 11.33.5（爱思助手下载）

### 抓包

1. 抓包使用 `charles`，请自行安装并配置证书。
2. 抓取登陆接口，点击账号登陆。使用假账密测试抓包`123456` / `123456`，能够抓包成功。

[![pCZQ3E6.png](https://bbs.kanxue.com/upload/attach/202306/928251_F2AEAR297FNSPFK.jpg)](https://bbs.kanxue.com/elink@2dcK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6K6k6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6H3b7#2A6c8x3@1f1$3)

### 分析

1. 登录页面需要输入3个信息，分别是账号 / 密码 / 验证码，对应字段`logincode` / `userpwd` / `validcode`。
   - `logincode`为一个字符串，是输入账号加一个`*%3*`做前缀。
   - `userpwd`是一个加密字符串，需要待分析。
   - `validcode`是验证码原文。

2. 经过多次抓包分析，其他字段为一些设备信息，可以保持不变。`_timestamp`是一个10位的时间戳，`_sign`参数每次均改变，需要待分析。

3. 分析前将包含`Mach-O`文件的，后缀名为`*.app*`的文件夹从爱思助手导出。

[![pCZlEqI.png](https://bbs.kanxue.com/upload/attach/202306/928251_RTSEW956CYCGMRE.jpg)](https://bbs.kanxue.com/elink@a0aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6K6k6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6H3b7#2A6D9c8i4q4u0)

4. 检查`Mach-O`文件是否需要脱壳。
   - 找到`Mach-O`文件。
     |  |
     | --- |
     | `file` `Payload/Autohome.app/*` `| grep Mach-O`  `Payload/Autohome.app/Autohome: Mach-O 64-bit executable arm64` |
   - 使用`otool`查看是否脱壳。1: 未脱壳；0: 脱壳。
     |  |
     | --- |
     | `otool -l Payload/Autohome.app/Autohome | grep crypt`  `cryptoff 16384`  `cryptsize 28295168`  `cryptid 1` |
   - 使用[frida-ios-dump](https://bbs.kanxue.com/elink@c8fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6m8L8r3!0F1k6f1#2G2L8X3E0W2P5g2)9J5c8X3k6J5K9h3c8S2i4K6u0V1K9h3!0K6i4K6u0V1k6s2g2E0M7l9`.`.)砸壳。
     - 需要`app`完全启动才能砸，如果砸壳中遇到阻塞请重试。
     - 砸完会生成一个`ipa`，直接解压即可。
   - 再次使用`otool`检测会发现`cryptid 1`变为`cryptid 0`，即砸壳完成。

5. 开始分析`userpwd`参数。用`ida`将`Mach-O`文件`Autohome`打开，搜索字符串`userpwd`。

[![pCZl4QH.png](https://bbs.kanxue.com/upload/attach/202306/928251_VCVXVTRTZXKG2HU.jpg)](https://bbs.kanxue.com/elink@adfK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6A6L8h3N6K6k6g2)9J5k6h3y4G2L8g2)9J5c8X3W2Q4x3V1k6H3b7#2A6D9y4q4q4t1)

6. 点击第一个进入，`tab`键转换伪代码。

|  |
| --- |
