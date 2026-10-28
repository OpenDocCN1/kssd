
```C++
0018E584 07 02 00 00 00 A4 00 00 52 53 41 32 00 08 00 00 ...?.RSA2...
0018E594 01 00 01 00 73 F6 BE B8 C8 55 17 87 6A F8 0D 55 ..s鼍溉U噅?U
0018E5A4 CF FD 1A F8 1B 4B F4 17 FC 14 F4 D4 EE 25 DD C0 淆?K??粼?堇
0018E5B4 CD 0F A3 BB 63 32 AE 1D E3 47 34 2B CB D8 C5 FD ?；c2?鉍4+素琵
0018E5C4 90 69 2F C1 F3 0B BD 91 CB E1 96 26 03 52 E3 9C 恑/馏綉酸?R銣
0018E5D4 D2 F0 89 1A 36 66 87 FC 75 09 63 05 75 44 37 AF 茵?6f圏u.cuD7?
0018E5E4 E0 44 85 87 10 CF 0A 5B 89 4C 91 45 90 69 7F F8 郉厙?[塋慐恑?
0018E5F4 31 58 35 31 2B 0C 8B D8 C6 85 16 C2 8D C4 C1 EF 1X51+.嬝茀聧牧?
0018E604 EF 10 10 1D ED AA 53 E6 79 65 83 2D 36 4B C8 68 ?愍S鎦e?6K萮
0018E614 DD AD 99 02 2D 18 87 BE CB 08 F9 23 A4 27 6E DA 莪?-嚲???n?
0018E624 BA C1 E9 14 55 FF 61 E8 41 54 08 07 AD F5 8F C8 毫?Ua鐰T徣
0018E634 FB 83 5D 87 5D 09 67 71 B2 9F 7B A5 C3 5C 23 2F 麅]嘳.gq矡{ッ\#/
0018E644 1A E0 8C 6C 26 B1 99 39 6E 68 9F 20 1D FF DE 5C 鄬l&睓9nh?轡
0018E654 05 79 A5 72 72 32 4C CE BF 90 F5 F8 00 E4 8C E3 yr2L慰愼?鋵?
0018E664 F2 AE 33 FB 03 A4 DA 1C 5C EC D3 3E C7 12 FE B3 虍3?ぺ\煊>?
0018E674 6E A6 2A 9A 5C 00 00 00 00 00 00 00 00 00 00 00 n?歕...........
```

接下来的 五个线程回调函数就是整个程序的重点了

### `CreateResFile` 第一个线程回调函数 创建`00000000.res`文件

这个函数在工作路径创建了`00000000.res`这个文件 并且往里写入数据

![](upload/attach/201904/833057\_AGMEAFEZKWT35EC.png)

写入的内容如下 其中有0x8个字节的随机数 还有0x4个字节的当前时间

![](upload/attach/201904/833057\_WWDVEBY6ZGNP6JS.png)

### `CheckDky` 第二个线程函数 检测文件是否存在

![](upload/attach/201904/833057\_JGCGNSED4EJPURX.png)

这个函数的作用是每个五秒检测工作路径下是否存在`774F34B5.dky`这个文件 文件名应该是个随机数 不是固定的 如果存在的话会执行`sub_10003D10`这个函数 由于我的桌面上一直都没有存在 所以这个函数我也就没有去分析

### `EncryptAllFiles` 第三个线程函数 加密所有文件(重点)

这个函数是整个病毒程序最核心的函数 代码量最多 里里外外总共嵌套了十几层函数 我感觉比整个熊猫烧香的代码还要多 下面是这个函数的第一层

#### 核心加密函数第一层

循环检测是否有新的磁盘加入 如果有 则加密 没有就一直循环

![](upload/attach/201904/833057\_RC3GAE3HXE8DRF7.png)

#### 核心加密函数第二层

接下来进入到核心函数第二层

![](upload/attach/201904/833057\_GDQ2J6T684B3NZC.png)

第二层有三个比较重要的函数 每个都有各自的作用 我们重点解析第二个加密函数

##### `MovFileToTemp` 移动文件到临时目录下并重命名为`.WNCRTY`

![](upload/attach/201904/833057\_Q659F6ACF5NFDNW.png)

这个函数单独创建了一个线程 将一部分文本文件移动到临时目录下 并进行重命名

下面是这个函数循环结束后我的临时文件夹下的文件 这些文件并没有加密 是可以直接通过修改后缀名恢复

![](upload/attach/201904/833057\_RXE64UBJ5Y9F3H9.png)

##### `FillDisk` 在回收站创建一个文件 并且循环写入数据直到磁盘空间不足

![](upload/attach/201904/833057\_N87UBNTMETPY5RB.png)

这个函数就比较有意思了 它会在在`$RECYCLE`下创建一个名为`hibsys.WNCRYT`的文件 并设置属性为隐藏 然后循环往这个文件写入数据 直到磁盘空间不足跳出循环

当这个函数结束的时候 这个文件居然有39个G

![](upload/attach/201904/833057\_HEJWBRDQH93U6H4.png)

##### `EncryptFile` 加密磁盘上的所有文件

![](upload/attach/201904/833057\_Z82KXMYM6DUWBWC.png)

这个函数对磁盘和路径做了一个判断 然后又调用了另外一个函数 我们需要再次进入到这个函数

#### 核心加密函数第三层

![](upload/attach/201904/833057\_JRD7U2FWQBHRNNX.png)

进到第三层之后又有一个函数 这个函数会遍历并且加密所有的文件 而且是递归调用的 然后我们再次进入这个函数

#### 核心加密函数第四层

![](upload/attach/201904/833057\_A4DBZW8XU6FNA33.png)

![](upload/attach/201904/833057\_2U8S96NP2UQJQ6D.png)

这个函数首先遍历所有的文件 对文件夹和文件执行不同的操作 并且对后缀名进行过滤

首先跳过 `@Please_Read_Me@.txt`，`@WanaDecryptor@.exe.lnk`， `@WanaDecryptor@.bmp`

具体类型如下

0 没有后缀以及其他类型后缀

1 `.exe`, `.dll`

4 `.WNCRYT`

5 `.WNCYR`

6 `.WNCRY`

2.

```C++
“.doc”".docx”".xls”".xlsx”".ppt”".pptx”".pst”".ost”".msg”".eml”".vsd”".vsdx”
“.txt”".csv”".rtf”".123″”.wks”".wk1″".pdf”".dwg”".onetoc2″”.snt”".jpeg”".jpg”
```

3.

```C++
“.docb”".docm”".dot”".dotm”".dotx”".xlsm”".xlsb”".xlw”".xlt”".xlm”".xlc”".xltx”".xltm”".pptm”".pot”".pps”".ppsm”".ppsx”".ppam”".potx”".potm”
“.edb”".hwp”".602″”.sxi”".sti”".sldx”".sldm”".sldm”".vdi”".vmdk”".vmx”".gpg”".aes”".ARC”".PAQ”".bz2″”.tbk”".bak”".tar”".tgz”".gz”".7z”".rar”
“.zip”".backup”".iso”".vcd”".bmp”".png”".gif”".raw”".cgm”".tif”".tiff”".nef”".psd”".ai”".svg”".djvu”".m4u”".m3u”".mid”".wma”".flv”".3g2″”.mkv”
“.3gp”".mp4″”.mov”".avi”".asf”".mpeg”".vob”".mpg”".wmv”".fla”".swf”".wav”".mp3″”.sh”".class”".jar”".java”".rb”".asp”".php”".jsp”".brd”".sch”
“.dch”".dip”".pl”".vb”".vbs”".ps1″”.bat”".cmd”".js”".asm”".h”".pas”".cpp”".c”".cs”".suo”".sln”".ldf”".mdf”".ibd”".myi”".myd”".frm”".odb”".dbf”
“.db”".mdb”".accdb”".sql”".sqlitedb”".sqlite3″”.asc”".lay6″”.lay”".mml”".sxm”".otg”".odg”".uop”".std”".sxd”".otp”".odp”".wb2″”.slk”".dif”".stc”
“.sxc”".ots”".ods”".3dm”".max”".3ds”".uot”".stw”".sxw”".ott”".odt”".pem”".p12″”.csr”".crt”".key”".pfx”".der”
```

总结为下面的枚举

```C++
enum FILE\_TYPE
{
FILE\_TYPE\_NULL = 0,
FILE\_TYPE\_EXEDLL,
FILE\_TYPE\_DOC,
FILE\_TYPE\_DOCEX,
FILE\_TYPE\_WNCRYT, //.wncryt
FILE\_TYPE\_WNCYR, //.wncyr
FILE\_TYPE\_WNCRY //.wncry
}
```

#### 核心加密函数第五层

![](upload/attach/201904/833057\_2E5VNASC7ZEJQGA.png)

这个函数首先会调用`sub_10002E70`这个函数 根据这个函数返回值的不同 执行不同的加密策略 对加密策略做一个总结

1. 在枚举文件中，`cmd=1`，会对普通文件直接加密为`.WNCRY`，不再加入链表，大文件处理为`.WNCYR`，以及其他未作处理文件继续加入链表等待处理

2. 枚举完成后，`cmd`从2-4，每个`cmd`遍历都遍历加密文件 `cmd=2`，加密`FILE_TYPE_DOCEX`普通文件为`.WNCRY`（移出链表），以及`FILE_TYPE_DOCEX`大文件为`.WNCYR` `cmd=2`, 删除`.WNCRYT`

3. `cmd=3`, 加密链表中所有文件（移出链表）

4. `cmd=4`, 加密可能剩余链表中的文件

虽然操作不同 但是加密函数是同一个 接下来再次进入`EncryptFiles`

#### 核心加密函数第六层

![](upload/attach/201904/833057\_8F4XC6S4XYCVK4V.png)

这个加密函数首先做了一些准备工作 获取文件的后缀名 然后跟`.WNCRY`做判断 如果比较成功 则不加密 接着做了一个字符串拼接 然后才开始加密文件 那么我们还要再往里跟一层

#### 核心加密函数第七层

接下来是这个程序的核心的加密算法了 他加密的步骤如下

1. 读取文件前0x8个字节的内容 跟`WANACRY!`作比较

![](upload/attach/201904/833057\_HD8BTTQSJQSFHQ3.png)

2. 使用原文件名+`.WNCRYT` 创建一个新文件 创建的时候这个文件并没有任何内容

![](upload/attach/201904/833057\_6URVU3YRCE35FAU.png)

3. 对创建的文件写入数据

![](upload/attach/201904/833057\_B253D3U2CU4PVTW.png)

4. 读取原文件 将加密后的文件内容写入到创建的文件

![](upload/attach/201904/833057\_FDZUGMTS6WQM2F5.png)

至此 病毒的加密函数就分析完成

### `StartTaskdl` 第四个线程回调函数 以隐藏的方式启动`taskdl.exe`

![](upload/attach/201904/833057\_927T2WC8UN2Y8S8.png)

这个函数每隔三秒以隐藏的方式启动`taskdl.exe` 那么接下来还需要对`taskdl.exe`进行分析

### `StartExeAndSetReg` 第五个线程回调函数 启动`taskse.exe`和`@WanaDecryptor@.exe`并且修改注册表

![](upload/attach/201904/833057\_B2JPY7W8SVBQCPV.png)

这个函数每隔三秒会启动工作路径的`taskse.exe`和`@WanaDecryptor@.exe`并且使用`cmd`设置注册表启动项

#### 启动`taskse.exe`

![](upload/attach/201904/833057\_2ZEDDTJA59SFJNX.png)

这个函数以命令行的方式启动`taskse.exe` 参数为为`C:\Users\GuiShou\Desktop\@WanaDecryptor@.exe` 这个参数需要暂时记住 待会分析`taskse.exe`的时候需要用到

并且以命令行的方式显式启动`@WanaDecryptor@.exe` 这个是病毒的解密器

#### 设置注册表启动项

![](upload/attach/201904/833057\_XW2T36SGU9Q7WFR.png)

函数将获取到的随机的字母+数字设置到注册表启动项 但是在我的机器上是设置失败的 因为命令敲错了 正确的命令应该是这个

![](upload/attach/201904/833057\_A7J5Q35FV9VSHG5.png)

### `RepeatOperation` 一些重复操作

这个是整个病毒程序最后一个函数了

![](upload/attach/201904/833057\_2NFSNKHVYHVW5VJ.png)

这个函数有三个函数比较重要

#### 1. `RunBat` 创建并启动批处理脚本

![](upload/attach/201904/833057\_PPSB6JS55YZHKTP.png)

这个函数会检测`@WanaDecryptor@.exe.lnk`是否存在 如果不存在 就创建一个批处理脚本 并且将命令写入到`.bat`脚本 这个脚本的作用仅仅的给`@WanaDecryptor@.exe`创建一个快捷方式

#### 2.`CreateReadMe` 创建`@Please_Read_Me@.txt`

![](upload/attach/201904/833057\_3A689S6TCSD8ZWF.png)

这个函数首先检测工作路径下的`@Please_Read_Me@.txt`是否存在 如果不存在 就从`r.wrny`中读取内容 并写入到`@Please_Read_Me@.txt` 这个是病毒的勒索文档

#### 3.`EncryptOtherUsersFiles` 加密其他所有用户的文件

![](upload/attach/201904/833057\_STFVVAXZUX99R3U.png)

![](upload/attach/201904/833057\_78TWUF99QFSZC6N.png)

这个函数的作用是获取windows所有的用户名 并检测是否和当前的用户名一致 如果不一致 就加密那个用户的所有文件 用的还是刚刚我们分析过的加密函数

剩下的就都是一些重复操作了 至此病毒的核心程序就分析完成

## 对`taskdl.exe`的分析

接下来我们来对病毒程序释放的`taskdl.exe`做一个详细的分析 `taskdl.exe`的主函数如下

![](upload/attach/201904/833057\_GU636394SP9H6NA.png)

那么从上面可以看到这个exe的代码量比起其他两个来说可以说是简单的多的多的多了 其中最核心的函数只有一个 就是上面的`DeleteFile`这个函数 接下来进入到这个函数进行详细分析

### `DeleteFile` 删除回收站和临时目录下的`.WNCRY`文件

![](upload/attach/201904/833057\_JDYUVB36H9NK6HE.png)

### `GetRecyclePathOrTempPath`

首先是`GetRecyclePathOrTempPath`这个函数 这个函数在病毒的主程序里面也有 它会返回两个路径 一个是回收站路径 也就是`D:\$RECYCLE` 第二个是系统盘的临时文件夹的路径 `C:\Users\GuiShou\AppData\Local\Temp`

接着这个函数会循环两次 然后对这两个文件夹做处理 由于我的虚拟机环境只有一个C盘和D盘 所以我并不知道如果多一个盘会出现什么情况

### 遍历文件

接着函数会调用`FindFirstFileW`这个函数查找目标文件夹中所有`.WNCRYT` 结尾的文件 第一次循环我这里失败直接返回 因为我的回收站并没有`.WNCRYT` 结尾的文件 但是我们要分析另外一条分支 所以我在临时文件夹下创建了这么几个文件来方便调试

![](upload/attach/201904/833057\_U78YG2W7RSZRHBE.png)

如果`FindFirstFileW`调用成功 就会走下面的分支

![](upload/attach/201904/833057\_PU53D3STCXPGZF9.png)

首先这个它会遍历所有的`.WNCRY`文件 并且将文件的完整路径和长度存储到一个容器中 如下图

![](upload/attach/201904/833057\_F7W7474JG5HFEDW.png)

### 删除文件

当文件遍历结束 会调用`DeleteFileW`删除所有的`.WNCRY`后缀的文件

![](upload/attach/201904/833057\_JC8Z5WJ6UKG22DT.png)

### 收尾 释放内存

接着 这个函数的功能就算是完成了

![](upload/attach/201904/833057\_KW27GXDZM4E98BS.png)

函数末尾循环将存放文件的完整路径和长度的容器清空 然后释放内存
