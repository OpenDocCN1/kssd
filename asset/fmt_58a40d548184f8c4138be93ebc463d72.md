
# 数据大师 3.17

> 原文：[`www.pediy.com/kssd/pediy04/kanxue294.htm`](https://www.pediy.com/kssd/pediy04/kanxue294.htm)

破解者：`nzg`

来自：`青岛`

破解时间：`2001-12-04`

破解工具：`unvfpexe 1.6` `测试版`

`vfp6.0`

## 软件简介

《数据大师》—通用信息（数据库）管理系统，是专门针对各行政、企事业单位不同管理需求而设计的能充分发挥操作人员创造力的傻瓜式通用信息管理软件。该软件用 `Vfp6.0` 设计，能用于任何单位的工资、人事、客户资料、固定资产、商务、学籍成绩、中介信息等等。

首先你要知道它使 `vfp` 编的，（压缩过，脱壳以后就可以看到，脱壳后也可以用 `unfoxall` 反编译！）

然后用 `unvfpexe 1.6` `测试版` 还原为`\*.app` 文件，然后就可以用写字板打开它，找到关键位置：

```
commandbutton 
Left = 232 
Height = 21 
Width = 70 
FontName = "宋体" 
FontSize = 9 
Caption = "现在注册" 
ToolTipText = "确认输入的注册码" 
Name = "Command1" 
if used('hd_dwqkk') 
    sele hd_dwqkk 
else 
    sele 0 
    use hd_dwqkk 
endi 
repl 单位名称 with thisform.text3.value 
repl 联系人 with thisform.text4.value 
repl 联系电话 with thisform.text5.value 
x=val(thisform.text1.value) 
zcm=bitxor(x,x/2+19620422) 
if val(thisform.text2.value)=zcm 
    （此处为关键，至于怎么比较，傻子也看得明白！其中 x 就是你的机器号。估计 19620422 是作者的生日，嗬嗬，好玩！！） 
    messagebox('注册成功，祝贺您获得《数据大师》永久使用权！',48,tytsck) 
clos all 
use hdcsk 
repl 注册码 with (thisform.text2.value) 
clos data all 
xtzcm_jc() 
rele thisform 
else 
    do form 联系注册表单 
    *messagebox('注册码不对！',48,tytsck) 
endi 
clos all 
```

好了！现在算法看明白了！如果你对 `vfp` 懂那么一点点！自己算号吧！

要是你像我一样连 `bitxor` 都不知道是干什么的，那么你继续往下看：

装一下 `vfp` 吧！

建立一个新的程序文件：

然后敲一下的东西：

```
x=.....(你的机器号） 
y=0 
y=bitxor(x,x/2+19620422) 
? y 
```

运行一下 `ok!!!` `y` 的值就是你想要得的注册码呀！

小弟对 `vfp` 一点也不理解，有不对之处一定恳请各位批评！

`青岛阿牛` `2001-12-04` `4dd466d0e1e0cc123afbe504da2462ca`
