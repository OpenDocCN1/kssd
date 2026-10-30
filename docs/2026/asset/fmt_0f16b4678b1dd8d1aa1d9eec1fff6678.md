
# [原创]固件解密

[TOC]

## 前言

最近玩固件有点上头，但是遇到了一个固件什么解不出来，属实懵逼，于是百度了下，发现被加密了，于是赶紧学了一手，应该够用了，大概意思是借助中间迭代版本，来找其中解密方法，但是发现该产品好多固件都在用同一个解密程序，对此比较好奇，于是尝试了下该程序能否解密其他固件

## 了解

《物联网安全漏洞挖掘实战》书里面有一个固件解密例子`dir822`，我就不再分析了，直接简述吧，在下载`DIR822C1_FW315`固件时，有一份pdf文件，大致意思是从`DIR822C1_FW303`进行过渡过来的

![图片描述](upload/attach/202301/964032_GVTEPVMEZMZBKCJ.png)

如果你直接用`binwalk`解`DIR822C1_FW315`，结果如下

![图片描述](upload/attach/202301/964032_CWA6SSWG43GBQ9T.png)

使用`binwalk`解`DIR822C1_FW303`，解后查看`/etc/templates/hnap/StartFirmwareDownload.php`文件，第88行开始下载固件，代码我就不分析了，第114行是解密固件，可以看到用的是`encimg`程序

![图片描述](upload/attach/202301/964032_VJT4AYWC92FMPND.png)

用`find`定位了该程序位于`/usr/sbin/encimg`，在定位签名位于`/etc/config/image_sign`

![图片描述](upload/attach/202301/964032_BV73N6WSWC9Y268.png)

查看签名内容为`wrgac43s_dlink.2015_dir822c1`，最后执行`encimg -d -i DIR822C1_FW315WWb02.bin -s wrgac43s_dlink.2015_dir822c1`，在使用`binwalk`解就可以了

![图片描述](upload/attach/202301/964032_P4MSZRGWEYR4UER.png)

## 思考

浏览了下最新固件，发现也存在`encimg`程序，也有签名，此时有一个想法，会不会都用该程序对固件解密，只是签名不一样

![图片描述](upload/attach/202301/964032_HAQ47WQKX7QU46X.png)

于是又找到一个固件中间迭代版本`DIR842C1_FW302b03`，发现签名变动的位置只是`43s`和`822c1`，且字母都是小写

![图片描述](upload/attach/202301/964032_S58SRK9R972H7T7.png)

## 验证

于是就开始玩了起来，就解了两次密，发现第二次解密，和第一次长度不一样，于是又尝试错误签名解了两次，发现没有变化依旧是`6869168`，这不妥妥的判断依据

![图片描述](upload/attach/202301/964032_6GTXXATWX2FE3WS.png)

由于是用`qemu`进行的系统模拟，所以用`sh`脚本编写可能方便一点，于是临时在菜鸟教程学了一点语法

```sh
#!/bin/sh
num=`cat /bin/num` #从00-100
chr=`cat /bin/chr` #从a-z
a="te"
b="st"
echo "./usr/sbin/encimg -d -i $1 -s wrgac${a}_dlink.2015_dir$2"
echo "./usr/sbin/encimg -d -i $1 -s wrgac${a}${b}_dlink.2015_dir$2"
for i in $num
do
b=`./usr/sbin/encimg -d -i $1 -s wrgac${i}_dlink.2015_dir$2`
c=`./usr/sbin/encimg -d -i $1 -s wrgac${i}_dlink.2015_dir$2`
if [ "$b" != "$c" ];then
echo $i
break
fi
done
for i in $num
do
for j in $chr
do
b=`./usr/sbin/encimg -d -i $1 -s wrgac${i}${j}_dlink.2015_dir$2`
c=`./usr/sbin/encimg -d -i $1 -s wrgac${i}${j}_dlink.2015_dir$2`
if [ "$b" != "$c" ];then
echo $i.$j
break
fi
done
done
```

于是就尝试了一次爆破`dir822`最新固件，结果失败了，我怀疑是`encimg`有问题还是固件有问题，于是重新换了`encimg`，发现结果依旧失败，于是尝试换固件，结果好了，应该是`encimg`对固件解密次数达到一定次数就被限制了，至于根据什么限制的，得用`IDA`对`encimg`反汇编看，这里我就暂时跳过，于是我就把固件放到另一个目录下，每次解密完后，在重新复制固件

```sh
#!/bin/sh
num=`cat /bin/num`
chr=`cat /bin/chr`
a="te"
b="st"
echo "cp /bin/$1 ./"
echo "./usr/sbin/encimg -d -i $1 -s wrgac${a}_dlink.2015_dir$2"
echo "./usr/sbin/encimg -d -i $1 -s wrgac${a}${b}_dlink.2015_dir$2"
for i in $num
do
`cp /bin/$1 ./`
b=`./usr/sbin/encimg -d -i $1 -s wrgac${i}_dlink.2015_dir$2`
c=`./usr/sbin/encimg -d -i $1 -s wrgac${i}_dlink.2015_dir$2`
if [ "$b" != "$c" ];then
echo $i
break
fi
done
for i in $num
do
for j in $chr
do
`cp /bin/$1 ./`
b=`./usr/sbin/encimg -d -i $1 -s wrgac${i}${j}_dlink.2015_dir$2`
c=`./usr/sbin/encimg -d -i $1 -s wrgac${i}${j}_dlink.2015_dir$2`
if [ "$b" != "$c" ];then
echo $i.$j
break
fi
done
done
```

可以看出执行结果出现了`43.s`，在看之前的签名中间确实是`43s`

![图片描述](upload/attach/202301/964032_5FG97BE7AB2Q2WN.png)

脚本搞定后，于是尝试解`DIR842C1_FW313`，失败了并没有成功，以为是自己脚本问题，于是手工尝试用`822`的`encimg`解`842`的，发现两次相同，我以为固件有问题，于是乎尝试用`842`中间版本解`DIR842C1_FW313`，发现两次不同，应该是`encimg`问题

![图片描述](upload/attach/202301/964032_FT9H38ZS5K8JES6.png)

## 总结

随着更新，虽然文件相同，但是实现原理上应该多少有一点差别，才导致了失败。固件解密可以借助中间迭代版本，进行解密，或者不缺钱的话，可以直接买设备，利用`uart`方式进入固件，对其进行分析
