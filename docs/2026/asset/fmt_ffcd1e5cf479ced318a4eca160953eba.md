
# [原创]某APP密码加密算法分析

## 前言

记录一下，最近在逆向某app过程中遇到的登录密码加密算法，样本就不给出了，欢迎交流，只做技术探讨，侵权联系我删除。

## 正文

反编译apk文件，通过一系列的定位，最终可以定位到密码的加密位置在

![图片描述](upload/attach/202112/892597\_YKFZGAJVMAPGAMR.png)

上面`sM2Ciphertext`是最后提交到服务器的密码值，可以看出调用了`getSM2Ciphertext`方法获取加密后的密码原文。跟踪调用关系，很容易找到了最后是调用了native层的`getOutput`函数，如下

![图片描述](upload/attach/202112/892597\_EHPR88QYUBE6QY4.png)

由于该函数的具体实现在so里面，我们先对该函数hook一下，看一下参数和返回值。

![图片描述](upload/attach/202112/892597\_N8PGJYVFQAA2YB2.png)

可以看到第一个参数是157845242和我们的密码无关，第四个参数是`779e3c9ccda4...|392157c3fb5dd...`这样的一个字符串，记住这个字符串很重要，后面会说到。通过抓包发现函数的返回值确实是提交到服务器的值，那么猜测可能在调用`getOutput`获取加密密码之前，一定是调用了其他的native方法，将密码参数添加到了so层，这样才能调用`getOutput`生成密码。然后我们又hook了`addtText`方法，并在密码框输入123456

![图片描述](upload/attach/202112/892597\_28HM6EMUBF3MCFF.png)

发现，每次输入时，`addText`的第三个参数发生变化，最后三个int参数是递增关系，这里我们可以知道，java层对我们的输入字符做了某种映射，使其最后添加到so层的不是原字符串。作者首先考虑的是使用unidbg模拟调用，但是模拟调用之前，首先要找到映射关系，还要处理初始化函数调用关系等。so... 放弃了。

换种思路，我们直接在so层找到加密算法，于是作者找到了了lib目录下`libPassGuard.so`，也可以hook动态注册函数，找到native方法注册在哪个so.

打开so发现在导出函数中没有找到`getOutput`函数，那么这是动态注册了

![图片描述](upload/attach/202112/892597\_QEHEUBVJHU98G66.png)

可以看出`JNI_Onload`方法中`v5`是动态注册的函数地址

![图片描述](upload/attach/202112/892597\_V75J7UQSRVTP7X8.png)

于是我们找到了上面的函数，`getOutput`的函数不是很复杂，如下

![图片描述](upload/attach/202112/892597\_EC6RCTPU5HRSSNY.png)

我们先hook `AndroidPassGuard::GetOutput`看一下

![图片描述](upload/attach/202112/892597\_9VUAMJTS9M2T2BR.png)

![图片描述](upload/attach/202112/892597\_84JZDNARHX9PZK8.png)

`v12`是我们从java层传进去的第二个字符串，事实上这是`PassGuardEncrypt`在初始时产生的随机hash值，`retval`就是返回给java的最终密码加密值。

我们继续跟踪调用，最后发现

![图片描述](upload/attach/202112/892597\_C98ZAESCEPJHBBK.png)

我们hook `GetRealTextex`发现

![图片描述](upload/attach/202112/892597\_HZNZBBNNB9VHW2W.png)

`AndroidPassGuard::PassGuardEdit::GetRealTextex`在这里将映射后的密码恢复成我们输入的密码了，这就很好办了。最后真实密码给了`MultiEncrypt`进行加密，看一下函数内部

![图片描述](upload/attach/202112/892597\_K8JB4K4KZSWBYHS.png)

然后又调用了`Encrypt`方法，继续跟进去

![图片描述](upload/attach/202112/892597\_C8XB8YB6MD4KBW7.png)

发现根据不同的`a1`值，最终调用的不同的加密算法，包括sm2、sm4、rsa等，而`a1`又是该函数的第一个参数，我们继续hook `Encrypt`方法得到了`a1 = 0x4`，那么很明显最终是调用了`realsm2`加密算法。我们继续hook `realsm2`算法看一下。

![图片描述](upload/attach/202112/892597\_VXTEB9QRP53CPMW.png)

`realsm2`的第一个参数是返回值，就是我们的加密结果了，第二个参数`779e3c9ccda4...|392157c3fb5dd...`是我们从java层传进去的，`123456`就是我们的明文密码了。

这里我们可以认为加密算法就是sm2了，但是公钥却不是`779e3c9ccda4...|392157c3fb5dd...`，一般来说，国密加密算法，公钥以04开头，且不会出现在字符串中出现`|`。

再继续看`realsm2`内部

![图片描述](upload/attach/202112/892597\_67PE4AUY4U6RD2C.png)

这里不hook了，累，直接说分析结果吧，在函数内部根据`|`分割字符串得到两个字符串，再使用`BN_hex2bn`将其转成对应的两个字节数组，然后在`EC_key_from_BN`函数内部，如下

![图片描述](upload/attach/202112/892597\_67GJNM3NPXWEEWG.png)

被当作仿射坐标，生成sm2加密公钥。当然这个公钥是个结构体，不是简单的字符串，不好直接hook，但是我们知道了仿射坐标就是java层传进去的字符串，至此就可以对原文加密了。

有了上面的分析，其实就可以加密了，作者找了一份网上公开的java版sm2加密算法，但是网上的代码一般是直接给定了04开头字符串公钥，而不是仿射坐标，如下

![图片描述](upload/attach/202112/892597\_CN432DM88E4KYJU.png)

阅读源码`decodePoint`源码可知，对给定的公钥

![图片描述](upload/attach/202112/892597\_A83JP467GGB66BD.png)

生成了仿射坐标，那么就很办了，我们直接做如下修改

![图片描述](upload/attach/202112/892597\_HEPDMFXKWAUR77N.png)

其中`point_x`就是`779e3c9ccda4...`，`point_y`是`392157c3fb5dd...`，这样来设置用户密钥。

最后放出加密结果

![图片描述](upload/attach/202112/892597\_7MVVRGZ9YRWH9B3.png)

## 结语

到这里基本验证了算法的可行性，但是各位看官都明白作者的java版sm2加密算法，还不一定能在在模拟请求中使用，因为服务器可能验证不通过。本文作者只对加密算法做了探讨，事实上该算法最后也通过了服务器验证，这里可以做个快速验证：用xposed hook替换掉java层`getSM2Ciphertext`函数，使用我们自己的java加密算法对密码原文进行加密，如果能成功登录，说明算法是正确的。

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)

[#逆向分析](https://bbs.kanxue.com/forum-161-1-118.htm)

[#协议分析](https://bbs.kanxue.com/forum-161-1-120.htm)

[#HOOK注入](https://bbs.kanxue.com/forum-161-1-125.htm)
