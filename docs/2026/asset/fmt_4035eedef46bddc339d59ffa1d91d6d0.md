
# [原创]初试so文件解密

## 前言

这里是2W班5月的习题。题目中对一个native函数进行了加密，导致IDA无法看到运行时的代码。加密方式来自于论坛比较久的帖子ThomasKing大佬的[简单粗暴的so加解密实现](https://bbs.pediy.com/thread-191649.htm)。照搬了大佬的代码，然后修改了一下加密方式即可完成解密。

通过本题学习到了so文件的格式解析，基础的加解密思路。

## 解题过程

### 步骤1：尝试用Fart脱壳

![](https://bbs.kanxue.com/upload/attach/202005/571058_FACF3KTVFJPGZRZ.jpg)

顺利完成，查找相关dex

```
sailfish:/data/data/com.kanxue.test
# grep -ril "MainActivity" ./*.txt
./2203744_classlist.txt
./2203744_classlist_execute.txt
```

拷贝出2203744大小的dex

### 步骤2：查看Java层代码

![](https://bbs.kanxue.com/upload/attach/202005/571058_JFP9EHU4NZWWDK9.jpg)

`OnCreate`去绑定事件，`test`函数进行的字符串比较

### 步骤3：查看so文件中test的代码

```
addr_RegisterNatives:
test
addr_RegisterNatives:
(Ljava/lang/Object;)Z
addr_RegisterNatives
func: 0x8dc5
```

`test`是动态注册的，注册地址是`0x8dc5`

![](https://bbs.kanxue.com/upload/attach/202005/571058_6P5QYDPWSQKKBYN.jpg)

反汇编代码比较奇怪

查看下`sub_8930`

![](https://bbs.kanxue.com/upload/attach/202005/571058_PXTWAR4Y7QHB83W.jpg)

这里获取so地址，然后查找`ooxx`函数的偏移

![](https://bbs.kanxue.com/upload/attach/202005/571058_79SKYJ76PWBXZ83.jpg)

查找`libnative-lib.so`内存中地址

![](https://bbs.kanxue.com/upload/attach/202005/571058_RCPPT37VK8VDNND.jpg)

看这段代码和so文件加密帖子(<https://bbs.pediy.com/thread-191649.htm>)给出的demo非常像,只是函数加密的方式不一样，具体思路可以直接去查看原帖。

### 步骤4：解密so文件

加密是找到函数偏移和长度，进行位运算，所以解密也就按照一样的方式还原即可。

下图是运行效果

![](https://bbs.kanxue.com/upload/attach/202005/571058_4NQHFBHDWSWANDW.jpg)

解密后再次反编译，解密方法参考帖子给出的代码，代码在windows下读取so文件会出现读取不准确，不知道为什么。最终在ubuntu下运行正常。

![](https://bbs.kanxue.com/upload/attach/202005/571058_MC9B2EHMEH8FEX2.jpg)

直接比对的，没加密。使用frida查看`1C04C`就是`kanxuetest`。

输入效验正确。

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [解密so.zip](javascript:void(0);)（23.00kb，58次下载）
- [2、找出flag.apk](javascript:void(0);)（2.44MB，35次下载）
