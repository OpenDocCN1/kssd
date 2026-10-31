
# 关于unicorn去搞VMP的iat那点事

首先感谢！浩哥！龟哥！二木哥！小路哥哥！黎叔！存哥！曾哥！荣神！醒哥！砍老板！震哥！2st师傅！枪手师傅！hook师傅！VF师傅！彭指导！等等大师傅们的教导！！！！

其实看了很多帖子，关于`unicorn`去搞vmp的事情，最开始来源于大表哥的github项目`unicorn_pe`，于是自己也研究了一下`unicorn`和`capstone`的东西感觉很好用！于是乎准备写一下，一起来学！呜呜呜，本人技术比较垃圾，写的可能大师傅们几乎都会，所以不喜勿喷，留情！

## VMP寻找OEP

其实关于vmp去找OEP的这一步我觉得是很关键的，因为在最开始加完vmp后，可以发现对应的text段，data段都是空的

![image-20211227101512991](upload/attach/202112/873515_ECFQXVEN4YTEUJ9.png)

![image-20211227101529683](upload/attach/202112/873515_N5CJ4G8E52SJ5SZ.png)

可以看到是没有数据的，vmp在运行后，会动态的解密，这样text段就会有数据了

一般我们写代码都会用到像编译器这种东西，用什么vs呀，易语言呀，vc++ 6.0呀 等等等的，他们这些东西编译出来的都会有一点框架，所以我们一般对相应的遇见的第一个api下段就可以找到入口点，一般的api也就是`GetVersion`，`GetSystemTimeAsFileTime`，如果下段后的栈回溯在text段内，那么我们继续回溯即可

可以看到相应的可以对上了，我们直接溯到`call` `jmp`的位置进行`dump`即可

![image-20211227102849337](upload/attach/202112/873515_7EJHAKBTC84843E.png)

## VMP寻找iat

我们先分析没有加壳的代码

![image-20211223104539097](upload/attach/202112/873515_QTPU7W3K4KR5RKC.png)

可以发现`call`为`FF 15 call`（当然还有`FF 25`）（`mov reg,iat call reg / jmp reg`）

这里以`FF 15 call`为例子，这里面都变成了`E8 call`，因为要保持原来的6个字节的问题 所以一般都是`push reg call xxxx / call xxxx ret`

![image-20211223104817486](upload/attach/202112/873515_XRVCNV6XTEMNTFH.png)

以第一个`push eax call sub_4FBE6B`为例子

```asm
.vmp00:004FBE6B 90                                      nop                     ; No Operation
.vmp00:004FBE6C 9F                                      lahf                    ; Load Flags into AH Register
.vmp00:004FBE6D 98                                      cwde                    ; AX -> EAX (with sign)
.vmp00:004FBE6E 58                                      pop eax
.vmp00:004FBE6F E9 62 5A EC FF                          jmp loc_3C18D6
.vmp00:003C18D6 87 04 24                                xchg eax, [esp-4+arg_0] ; Exchange Register/Memory with Register
.vmp00:003C18D9 E9 F5 D8 F2 FF                          jmp loc_2EF1D3      ; Jump
.vmp00:002EF1D3 50                                      push eax
.vmp00:002EF1D4 B8 3F 15 28 00                          mov eax, 28153Fh
.vmp00:002EF1D9 E9 2F 45 16 00                          jmp loc_45370D      ; Jump
.vmp00:0045370D 8B 80 51 E1 00 00                       mov eax, [eax+0E151h]
.vmp00:00453713 8D 80 CD 4A 08 4C                       lea eax, [eax+4C084ACDh] ; Load Effective Address
.vmp00:00453719 87 04 24                                xchg eax, [esp+0]    ; Exchange Register/Memory with Register
.vmp00:0045371C E9 8C 35 E4 FF                          jmp nullsub_32      ; Jump
.vmp00:00296CAD C3                                      retn                    ; Return Near from Procedure
```

可以看到上面的流程最后一个`retn`又因为`xchg`交换了`eax`和`esp`的内存，所以可以判定出iat的地址有关系的几句汇编是

```asm
mov     eax, 28153Fh
mov     eax, [eax+0E151h]
lea     eax, [eax+4C084ACDh]
xchg    eax, [esp+0]
```

`iat = [28153F+0E151]+4C084ACD`

![image-20211223105335161](upload/attach/202112/873515_YET7K3FBUC9CWZB.png)

可以看到表示的没有问题

## iat脚本

这里我使用的是`unicorn`来获取到对应的api，因为在恶意样本的操作的时候，一定会用到api做一些恶意的功能，也会对分析来说多了一点帮助

使用方法：

用`x64dbg`转到对应的段的内存布局进行文件`dump`

将文件放在项目的目录下，以及将当前环境的`reg`的值填入（这里我延用了周壑老师的代码，进行了修改）

![image-20211223105529537](upload/attach/202112/873515_3QVFTUMCBC3BP7E.png)

因为周壑老师说的是抛异常的形式，那么我就按照异常的形式来捕获iat，因为对于api的这些内存的位置，我是没有map到内存中的，所以遇到iat就会出现异常

因为我看了很多地方都是`E8 call`，而且在text段中`call`的位置为vmp段，所以直接暴搜`E8 call`即可，把这些位置记录下来然后遍历即可，根据ldr以及pe的解析就可以搞出来iat啦！

效果：

![image-20211223105749933](upload/attach/202112/873515_P9MFHTZPHXWE96W.png)

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

[#VM保护](https://bbs.kanxue.com/forum-4-1-4.htm)
