
# ReGet Junior 2.0破解手记(三）

作者：囚童[FCG][BCG]

课题：（三）脱除ReGet Junior 2.0主程序文件`regetjr.exe`的壳

背景：PACKED WITH ASProtect v1.2

下载：`http://www.cn778.net/download/c_bexpress.zip`

工具：CASPR v1.012、EditPlus 2、language2000中文版v4.51、FileInfo2.40、TYP、BTW2000

接ReGet Junior 2.0破解手记(一）、(二）中，我们讲了如何解除ReGet Junior 2.0广告动画画面的方法三种，解除显示软件名称和版本号的BANNER的方法，并制作了内存补丁。这里，我们介绍如何给ReGet Junior 2.0主程序文件`regetjr.exe`脱壳。

`regetjr.exe`原有的尺寸为497K。用language2000中文版v4.51、FileInfo2.40和TYP都没能查出`regetjr.exe`是否被压缩过，但用BTW2000和TRW2000观察到的程序入口分别是`440A2A`和`400040`，据此可以判断：`regetjr.exe`是被压缩过的，怀疑是ASProtect v1.2。

① 用EditPlus建立`CASPR.ini`文件如下：

```ini
[Options]
Import=0x00000001
Export=0x00000001
TLS=0x00000001
Resource=0x00000001
Relocation=0x00000000
Overlay=0x00000001

[RVAs]
;IMAGE_IMPORT_DESCRIPTOR
IID=0x00000000
;IMAGE_IMPORT_OriginalFirstThunk_DATAs
IIOFTD=0x00000000
;IMAGE_IMPORT_BY_NAMEs
IIBN=0x00000000
;IMAGE_IMPORT_DESCRIPTOR.DLLNAME
IIDDN=0x00000000
;IMAGE_EXPORT_DIRECTORY
IED=0x00000000
;IMAGE_TLS_DIRECTORY
ITLSD=0x00000000
;IMAGE_RESOURCE_DIRECTORY
IRD=0x00000000
;IMAGE_RESOURCE_DATA
IRData=0x00000000
;IMAGE_RELOCATION
IR=0x00000000
```

② 用EditPlus建立批文件`Un-pack.BAT`如下：

```batch
CASPR -L regetjr.exe -P Output
```

③ 建立一个工作目录`Un-pack`。将下列文件拷入这个目录：`Caspr.exe`、`Caspr.ini`、`Caspr1.dll`、`Caspr2.dll`、`regetjr.exe`、`Un-pack.BAT`。

④ 在`Un-pack`目录下再建一个子目录`Output`。

⑤ 运行`Un-pack.BAT`。`Caspr`在`Output`目录中生成一个新文件`regetjr.ex_`，没有异常提示出现。

⑥ 将新文件`regetjr.ex_`以后缀EXE更名后，覆盖原有的`regetjr.exe`。

⑦ 用BTW2000复查程序入口点为`440A2A`。

⑧ 试运行新的`regetjr.exe`，没有异常提示出现，检验各项功能齐全。新的`regetjr.exe`尺寸为982K。脱壳成功。

相关文章：[ReGet Junior 2.0破解手记(一）](https://www.kanxue.com/kanxue424.htm) [ReGet Junior 2.0破解手记(二）](https://www.kanxue.com/kanxue425.htm) [ReGet Junior 2.0破解手记(三）](https://www.kanxue.com/kanxue426.htm)
