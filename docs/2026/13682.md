
# [原创] 震惊！一男子居然用杀软做出这种事情!

## 思路

利用杀软对感染型病毒的清除过程中对文件名过滤的不严格实现任意目录写任意文件.(包括杀软目录, 但杀软目录下的文件无法覆盖)

## 原理

```
//假设以下是杀软的清除代码
int fuckpolyransom(wchar_t* szdir)
{
//得到被感染文件的原始文件名
wchar_t* originalfilename = GetInfectedFileOriginalFileName(xxxx);
wchar_t* fulloriginalfilename = (wchar_t*)malloc(sizeof(wchar_t)*(32767));
//没有对originalfilename进行检查是否包含'\\','//', 导致了问题.
StringCbPrintfW(fulloriginalfilename,32767*sizeof(wchar_t),L"%s\\%s",szdir,originalfilename);
CreateFile(fulloriginalfilename,xxxx);
return 0
}
```

## 发现过程

最近搞`polyransom.a`. 这个感染型病毒会把目标文件整个包起来,然后执行时会释放, 而且它会保留原文件名信息.

之后我用各杀软清理了下被感染的文件,发现杀软会在当前目录下以原文件名创建恢复后的文件. 这时,我想到,如果保留的文件名中包含路径分隔符, 然后我再弄个目录符号链接(文件夹快捷方式应该也行), 岂不是可以利用杀软往任意目录下写任意文件了? 而且还是利用杀软的清除功能, 把内嵌的文件放到了我想要的位置!

那么,关键就在,杀软会不会对从感染体中获得的文件名信息进行检查. 经过试验发现, 不会,所以就有了这篇文章.

## 测试样本构造

随意取一个能查杀的`polyransom.a`样本, 将路径信息更改成`b\fuck.exe`

原始文件名:

![](upload/attach/202007/782787_VPRCCAX8A23VEFV.png)

可以看到文件名为`cuninst.exe`, 后面紧跟着文件的PE数据. 此处是解密后的数据.

重跑程序,来到解密之前, 将数据填成解密之后的数据. 修改解密为加密流程, 得到新文件名的解密前数据:

![](upload/attach/202007/782787_TC8ZKYJZ7HFRCPX.png)

`od`的显示有点bug, `\fuck.exe`前面的`b`也应该是被选中的. 得到的数据为:

![](upload/attach/202007/782787_DJ4XAXQUEA95JBG.png)

利用`010editor`修改文件数据:

![](upload/attach/202007/782787_4JFHUEYP2JJXBPF.png)

将修改完的文件跑下,看是否修改成功:

![](upload/attach/202007/782787_THEQY3TKDD9FSV2.png)

![](upload/attach/202007/782787_BBB324AKSCZKS65.png)

至此,样本构造完成.

## 杀软测试

### 卡巴斯基(kis20.0.14.1085_zh-hans-cn_full)

首先, 根据杀软安装目录,创建目录符号链接:

![](upload/attach/202007/782787_M2ZHEEHK43T82VF.png)

然后把构造的样本放到`C:`下, 选择查杀样本.

![](upload/attach/202007/782787_A4NBNWGJZD4ND8Q.png)

![](upload/attach/202007/782787_DVZ3G3PFG3B8ZT8.png)

![](upload/attach/202007/782787_9RYKR88PY49MQHF.png)

果然, 成功了.

### 360卫士 (2020-7-7版)

同理,测试,得到结果为:

![](upload/attach/202007/782787_W3CHX75MZNBD3S4.png)

### 360杀毒(2020-7-7版)

![](upload/attach/202007/782787_8P84PY8V3FZ5YFX.png)

同理,测试,得到结果为:

![](upload/attach/202007/782787_4M4D5S9UX6SH4WH.png)

## 应用场景

你在卡饭论坛上上传了一个这样的样本, 让大家帮你测测看能不能查杀, 于是....

## 其他

1. 创建符号链接可能会被拦截, 但这一步并非必须步骤, 只要把样本放到目标目录的上级目录, 或者利用`..\`相对路径(不过这点得看杀软是否使用的是`\\?\`开头支持超长路径的绝对路径了).
2. 一般人点了扫描之后,看到扫出病毒, 点击立即处理,然后看到样本消失,肯定会以为病毒被删除了,却没想到还有这种骚招数,这样隐蔽性就比较高.

## 附件密码infected

只需在样本所在目录下用`mklink /d b <targetdir>`,然后用杀软扫描样本, 就可以将文件释放到`targetdir`中.

![](upload/attach/202007/782787_WQCED8KTWH55EDA.jpg)
