
# [原创] 某Android模拟器的广告植入分析

这是一个做的还不错的模拟器，不过它的广告让人很烦，考虑先分析看看。

## 初步分析 `dnplayer.exe`

### 1. 反调试

它是带反调试的，使用`x64dbg`启动，会直接带调试器一起退出，添加了两项后，可以继续：

1. `SharpOD`：`anti_anti_debug`
2. `x64dbg`：获取调试权限

### 2. 静态分析

在启动的时候，会出现异常：

![图片描述](https://bbs.kanxue.com/upload/attach/202004/587311_DM3WDP637V4BPG5.png)

进入代码分析 `sub_449010`：

![图片描述](https://bbs.kanxue.com/upload/attach/202004/587311_ZEZY76F2NCVRHE9.png)

定位到上面的代码，并且发现它在加载文件 `launchConfig.data`，打开看看内容：

![图片描述](https://bbs.kanxue.com/upload/attach/202004/587311_B3CRZTQGYADY6VH.png)

看上去是去远端获取游戏资源的配置项（连接可以加载打开看）。

先把这部分 bypass 后再看看功能，发现启动还是有广告。

```
sub_4C7C50(_DWORD *this)
+--->show_root_window_sub_4D1930(__m128i *this) ## 初始化函数，不能去掉
+--->sub_448D30()
+---> sub_449010()
```

查看网络数据信息：

![图片描述](https://bbs.kanxue.com/upload/attach/202004/587311_JYE2JMVZAEWR3QV.png)

从调试中得到的请求地址信息：

![图片描述](https://bbs.kanxue.com/upload/attach/202004/587311_TAWUZFBTSMHRA97.png)

HTTP 相关的调用操作：

```
utility::CHttpDownloader::download2mem
utility::CHttpDownloader::waitFor
utility::CHttpDownloader::start
```

因为测试 `show_root_window_sub_4D1930` 函数不能被 bypass，但是它是主要的加载位置，而且广告也是在它内部加载的，所以考虑从它内部进行分析。

## 功能去除

### 1. 启动去广告 patch

```
show_root_window_sub_4D1930
{
    ...
    if ( sub_444BB0(&dword_5F8EF0, v48) ) // Bypass点，验证去掉了启动广告。
    {
        v49 = *(_DWORD **)(v47 + 1372);
        v50 = v49[3];
        if ( v50 )
        {
            if ( v50 != 2 )
            {
                LABEL_54:
                v52 = sub_518A50((_DWORD *)(v47 + 44), (int)L"loadingbk_hImage");
                ...
    }
    ...
}
```

### 2. 启动去掉 HTTP 外部请求

从请求数据可以发现它进行了很多远端操作，包括统计信息发送，版本检测等等，需要把这些全干掉；从 API 监控分析：

![图片描述](https://bbs.kanxue.com/upload/attach/202004/587311_N3S9HZMDA8GSCA3.png)

关键函数：`InternetCrackURL`，通过设置 API 断点，进行定位函数调用点。

![图片描述](https://bbs.kanxue.com/upload/attach/202004/587311_MARHVW9HCUQ2PKM.png)

定位到函数：`sub_437090(utility::CHttpDownloader *this)`

![图片描述](https://bbs.kanxue.com/upload/attach/202004/587311_ME2FSB2DTQ2ZHPW.png)

对上面的函数进行 Patch 可以解决一部分的请求，应该是有多处请求的位置，还剩下几个请求，其中比较可疑的是下面的请求：

![图片描述](https://bbs.kanxue.com/upload/attach/202004/587311_MRGJZ3GBS2UK364.png)

像是一个强制 APP 列表信息：

![图片描述](https://bbs.kanxue.com/upload/attach/202004/587311_S4QW3B2N2K8SFC8.png)

直接就有调用函数栈信息，从这里就能定位到调用点：`sub_489100 -> sub_47EFB0()`

但是 Patch 不太容易，`dnutility.dll` 是一个模块，可以去看看这个 HTTP 函数处理：

![图片描述](https://bbs.kanxue.com/upload/attach/202004/587311_36EXFRTV5DG9SU9.png)

分析可以看到，如果 Patch 掉上面的位置，可以 Patch 所有请求的发起。

![图片描述](https://bbs.kanxue.com/upload/attach/202004/587311_NGMDNMSYZKTDZKT.png)

Patch 后，发现请求一个也没有，而且测试并不影响后面的 APP 网络使用（应该是走的虚拟机网络）。

### 3. 去掉窗口下的固定广告

![图片描述](https://bbs.kanxue.com/upload/attach/202004/587311_ZYM6YDYU2P5PF8S.png)

猜测这个广告应该是一个窗口叠加产生的，它并不在启动的虚拟机内，但是如果点击就会调用命令进行对应的请求和安装？？？但是从模拟器内部看感觉不像，应该是对 Android 的功能设定的。

进入 Android 系统分析

#### 3.1 使用虚拟机加载虚拟磁盘

- `system.vmdk`：963.4MB，NTFS
- `data.vmdk`：128GB，NTFS
- `sdcard.vmdk`：128GB，NTFS

```
[root@localhost ~]# mkdir /tmp/android
[root@localhost ~]# cd /tmp/andorid
[root@localhost andorid]# mkdir system1 system2 data1 data2 sdcard1 sdcard2
[root@localhost andorid]# mount /dev/sdd1 system1
[root@localhost andorid]# mount /dev/sdd2 system2
[root@localhost andorid]# mount /dev/sdc1 data1
[root@localhost andorid]# mount /dev/sdc2 data2
[root@localhost andorid]# mount /dev/sdb1 sdcard
[root@localhost andorid]# mount /dev/sdb1 sdcard1
[root@localhost andorid]# mount /dev/sdb2 sdcard2
```

通过直接查找：

```
[root@localhost sdcard2]# grep "三国志" -r .
./data/com.android.launcher3/shared_prefs/ldmnq_preference.xml
./data/com.android.flysilkworm/files/__track_send_data_1585810935428
```

![图片描述](https://bbs.kanxue.com/upload/attach/202004/587311_386DH65Q8Y637BU.png)

通过删除文件，重启后测试，发现并没有什么变化，而且自己备份的文件还被删除了，说明这个地方的配置应该是有程序在自动生成，并不是配置文件，而且这个目录有点像是 APK 的应用数据目录，那么这个目录对应的是：`com.android.launcher3`，应该是定制了这个 APK。

```
[root@localhost sdcard2]# grep configCache -r .
Binary file ./dalvik-cache/x86/system@app@Launcher3@Launcher3.apk@classes.dex matches
```

进行文件分析：

```
[root@localhost sdcard2]# tree | grep Launcher3.apk
│ ├── system@app@Launcher3@Launcher3.apk@classes.art
│ ├── system@app@Launcher3@Launcher3.apk@classes.dex
[root@localhost sdcard2]# cd ../system2/
[root@localhost system2]# tree | grep Launcher3.apk
│ │ └── Launcher3.apk
[root@localhost system2]#
[root@localhost system2]# find . -name Launcher3.apk
./app/Launcher3/Launcher3.apk
```

考虑修改试试，先用 APKIDE 进行分析 & 打包：

![图片描述](https://bbs.kanxue.com/upload/attach/202004/587311_VMB334C6UAQN7RE.png)

![图片描述](https://bbs.kanxue.com/upload/attach/202004/587311_A4FNHRM5KRFFC64.png)

上传 APK，重启测试：

```
aosp:/system/app/Launcher3 # mount | grep /system
/dev/block/sda2 on /system type ext4 (ro,relatime,data=ordered)
aosp:/system/app/Launcher3 # mount -o rw,remount /system
aosp:/system/app/Launcher3 # mount | grep /system
/dev/block/sda2 on /system type ext4 (rw,relatime,data=ordered)
aosp:/system/app/Launcher3 # cp Launcher3.apk Launcher3.apk.bak
cp Launcher3.apk Launcher3.apk.bak
aosp:/system/app/Launcher3 # ls
Launcher3.apk Launcher3.apk.bak
```

![图片描述](https://bbs.kanxue.com/upload/attach/202004/587311_G98UWSAZS3WNKJA.png)

然后就发现已经成功去掉了内置广告：

![图片描述](https://bbs.kanxue.com/upload/attach/202004/587311_HSCQYZRXAFSJD7B.png)
