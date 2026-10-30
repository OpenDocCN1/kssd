
# [原创]D-Link DIR-645路由器溢出分析

[TOC]

## 漏洞介绍

该漏洞是CGI脚本在处理`authentication.cgi`请求，来读取POST参数中的`password`参数的值时造成的缓冲区溢出。

## 固件提取文件系统

固件下载：`ftp://ftp2.dlink.com/PRODUCTS/DIR-645/REVA/DIR-645\_FIRMWARE\_1.03.ZIP`

![图片描述](upload/attach/202108/913238\_96HNCC775ASJ2X8.png)

## qemu+IDA调试分析

1,`run_cgi.sh`脚本：

```bash
#!/bin/bash
# 待执行命令
# sudo ./run\_cgi.sh `python -c "print 'uid=A21G&password='+'A'\*0x600"` "uid=A21G"
INPUT="$1" # 参数1，uid=A21G&password=1160个A
TEST="$2" # 参数2，uid=A21G
LEN=$(echo -n "$INPUT" | wc -c) # 参数1的长度
PORT="1234" # 监听的调试端口
# 用法错误则提示
if [ "$LEN" == "0" ] || [ "$INPUT" == "-h" ] || [ "$UID" != "0" ]
then
echo -e "\nUsage: sudo $0 \n"
exit 1
fi
# 复制qemu-mipsel-static到本目录并重命名，注意是static版本
cp $(which qemu-mipsel-static) ./qemu
echo $TEST
# ｜ 管道符：前者输出作为后者输入
# chroot 将某目录设置为根目录（逻辑上的）
echo "$INPUT" | chroot . ./qemu -E CONTENT\_LENGTH=$LEN -E CONTENT\_TYPE="application/x-www-form-urlencoded" -E REQUEST\_METHOD="POST" -E REQUEST\_URI="/authentication.cgi" -E REMOTE\_ADDR="127.0.0.1" -g $PORT /htdocs/web/authentication.cgi
echo 'run ok'
rm -f ./qemu # 删除拷贝过来的执行文件
```

2,调试目标程序需要匹配正确。（有知道原因的可以跟帖回复，也方便大家览阅）

![图片描述](upload/attach/202108/913238\_3B9H3FTDT2VMB7W.png)

3，IDA分析，追踪问题函数

![图片描述](upload/attach/202108/913238\_GVQ926B9HEQVGE5.png)

4，填充数据调试

![图片描述](upload/attach/202108/913238\_F7SMR9DWF59NA3N.png)

IDA调试参考：

![图片描述](upload/attach/202108/913238\_XKXH7W84SDABFYF.png)

获得`&ra`在栈上的地址（这是非子叶函数的性质）：

![图片描述](upload/attach/202108/913238\_M6Z3R3JQ28S37CM.png)

F8执行观察，直到栈上保存`&ra`的数据内容发送变化（可猜测这里可能时溢出点）：

![图片描述](upload/attach/202108/913238\_H8CJRKFPAJDYFRP.png)

注意：为了防止后面可能出现二次溢出，或则其他处溢出才是真正影响被程序被控制的位置，我们继续F8执行观察。

程序异常结束了，发现时`a1`寄存器的值是栈上的，大概猜测一下是我们填充的值太大影响到了这位置上的值。

5，看看`a1`正常的内容读取:

![图片描述](upload/attach/202108/913238\_3N75TUVY6Q4T6BS.png)

缩短填充内容的长度，重新调试：

![图片描述](upload/attach/202108/913238\_2425MFY6WY4JM3C.png)

程序走到`authenticationcgi_main`的返回位置才退出：

如果需要看到更明显的步骤，可以自己找到此处再下个断点

![图片描述](upload/attach/202108/913238\_7MFKJWV7CP3C634.png)

![图片描述](upload/attach/202108/913238\_ZNQ2FGURQNRKWBZ.png)

结论：真实溢出位置就是`read()`函数引起的。

6，分析`read()`函数上下文传入传出数据。

先到`read()`函数跳转处分析参数的来源与目的地：

![图片描述](upload/attach/202108/913238\_BBWG6GCGVJ3E5BR.png)

分析方法：由于MIPS是流水线执行指令顺序，寻找参数先到函数跳转处先向下查找参数，然受再向上查找参数。

![图片描述](upload/attach/202108/913238\_GQDU76CYET3F4E7.png)

最终得到`read()`函数原型：

`read(fileno(stdin), var\_430, atoi(getenv("CONTENT\_LENGTH")))`

7，注：`var_430`计算大小方式：

根据栈中变量的顺序去计算

![图片描述](upload/attach/202108/913238\_BA3EW99JGMWDMVQ.png)

至此漏洞定位分析完，起始后面还有些危险函数可能存在危险溢出点需要验证，不过方法都无非是构造数据填充加上调试观察构造的数据位置。由于后面的函数都达不到溢出，所以就不附上步骤了。

- 根据漏洞描述，POST提交数据时，并不是任意格式的数据都能造成缓存区溢出，需要`id=XX&&password=XX`形式的格式。

验证分析：

![图片描述](upload/attach/202108/913238\_B7UPU3J3GZQXMGA.png)

程序异常退出在此处，分析：

![图片描述](upload/attach/202108/913238\_ZYF546MJU4WUA6W.png)

在向上分析，发现数据最终来源与`$s2`相关的数据，双击进入，发现固定格式，读取后面数据为`strlen`服务：

![图片描述](upload/attach/202108/913238\_7WV8B3HXGZH5SVJ.png)

更改回要求的形式获得结果：

![图片描述](upload/attach/202108/913238\_HKPV7PQWS2T24HM.png)

![图片描述](upload/attach/202108/913238\_6RAX27GEM357VJG.png)

## 漏洞利用

1，调试确定偏移

这里分享个更方便的脚本`patter.pl`脚本生成构造数据:

```perl
#!/usr/bin/perl -w
use strict;
# Generate/Search Pattern (gspattern.pl) v0.2
# Scripted by Wasim Halani (washal)
# Visit me at https://securitythoughts.wordpress.com/
# Thanks to hdm and the Metasploit team
# Special thanks to Peter Van Eeckhoutte(corelanc0d3r) for his amazing Exploit Development tutorials
# This script is to be used for educational purposes only.
my $ustart = 65;
my $uend = 90;
my $lstart = 97;
my $lend = 122;
my $nstart = 0;
my $nend = 9;
my $length ;
my $string = "";
my ($upper, $lower, $num);
my $searchflag = 0;
my $searchstring;
sub credits(){
print "\nGenerate/Search Pattern \n";
print "Scripted by Wasim Halani (washal)\n";
print "https://securitythoughts.wordpress.com/\n";
print "Version 0.2\n\n";
}
sub usage(){
credits();
print " Usage: \n";
print " gspattern.pl \n";
print " Will generate a string of given length. \n";
print "\n";
print " gspattern.pl \n";
print " Will generate a string of given length,\n";
print " and display the offsets of pattern found.\n";
}
sub generate(){
credits();
$length = $ARGV[0];
#print "Generating string for length : " .$length . "\n";
if(length($string) == $length){
finish();
}
#looping for the uppercase
for($upper = $ustart; $upper <= $uend;$upper++){
$string =$string.chr($upper);
if(length($string) == $length){
finish();
}
#looping for the lowercase
for($lower = $lstart; $lower <= $lend;$lower++){
$string =$string.chr($lower);
if(length($string) == $length){
finish();
}
#looping for the numeral
for($num = $nstart; $num <= $nend;$num++){
$string = $string.$num;
if(length($string) == $length){
finish();
}
$string = $string.chr($upper);
if(length($string) == $length){
finish();
}
if($num != $nend){
$string = $string.chr($lower);
}
if(length($string) == $length){
finish();
}
}
}
}
}
sub search(){
my $offset = index($string,$searchstring);
if($offset == -1){
print "Pattern '".$searchstring."' not found\n";
exit(1);
}
else{
print "Pattern '".$searchstring."' found at offset(s) : ";
}
my $count = $offset;
print $count." ";
while($length){
$offset = index($string,$searchstring,$offset+1);
if($offset == -1){
print "\n";
exit(1);
}
print $offset ." ";
$count = $count + $offset;
}
print "\n";
exit(1);
}
sub finish(){
print "String is : \n".$string ."\n\n";
if($searchflag){
search();
}
exit(1);
}
if(!$ARGV[0]){
usage();
#print "Going into usage..";
}
elsif ($ARGV[1]){
$searchflag = 1;
$searchstring = $ARGV[1];
generate();
#print "Going into pattern search...";
}
else {
generate();
#print "Going into string generation...";
}
```

用法：

2,`patter.pl`脚本使用方法

有两种操作模式：

1) 只提供一个参数，即要生成的字符串的长度

(`./ gspattern.pl [length of string]`)

2) 字符串的长度和要找到偏移量的模式提供

（`./ gspattern.pl [字符串长度] [搜索模式]`）

注（搜索模式）：获得要计算偏移溢出位置的hex值，转化为ASCII码。（记住一定要根据大小端序来输入，下面步骤中已举例）

3，

生成构造数据（我直接写入文件了，它把description也一块写入了，需要进去删除下）：

```
./pattern.pl 1160 > test\_auth
```

调试确定需要的偏移位置值：

```
sudo ./run\_cgi.sh `python -c "print 'uid=A21G&password='+open('test\_auth','r').read(1160)"` "uid=A21G"
```

![图片描述](upload/attach/202108/913238\_DM4S4N6D757W3HV.png)

将`0x38684237` 转成对应ASCII码：`8hB7`

![图片描述](upload/attach/202108/913238\_QMABCEBKKRT754A.png)

4，构造ROP参考：[家用路由器漏洞挖掘实例分析](https://bbs.pediy.com/thread-268623.htm)

5，POC
