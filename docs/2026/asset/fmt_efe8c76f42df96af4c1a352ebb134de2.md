
如果需要修改`/path/to/file`，使用如下代码

```python
rfs = RemoteStream()
system = ext4.Volume(rfs, offset=1024*1024)
targetfile = system.root.get_inode("path", "to", "file").open_read()
targetdata = open('editedfile', 'rb').read() # 本地读取修改后的数据
# 因为是在文件系统上直接覆写文件，修改后的文件大小一定要和原文件相同
targetfile.rewrite(targetdata)
```

该方式主要用于修改文件，浏览以及读取文件可以使用7-Zip打开system.img。

## 5. 获取root权限

### 5.1 修改系统apex包

能够修改system文件之后，目标是patch adbd，然而adbd还被封装在`/system/apex/com.android.adbd.apex`中。使用7-Zip拿出`com.android.adbd.apex`后发现可以直接解压。解压后发现其中有个文件是`apex_payload.img`，又是一个小型的ext4文件系统镜像，而adbd就在这个文件系统里。*（好家伙，俄罗斯套娃套起来了...）*

借助ext4模块可以编辑`apex_payload.img`中的adbd文件，但是改完了要怎么把`apex_payload.img`放回去？重新打包的话并不能保证`com.android.adbd.apex`文件的大小不变，也就无法再将其放回system文件系统里了。

这时发现这个apex文件压缩方式是仅存储，也就是并没有压缩，`apex_payload.img`的完整数据可以直接在这个文件中找到。因为patch adbd并不会改变该文件的大小，于是可以用python里的`replace`替换该文件，同时需要替换该文件的`crc32`。实验发现幸好android系统并不会验证在`/system/apex/`目录下已经安装了的apex包的签名。

### 5.2 patch adbd

再使用7-Zip将adbd从`apex_payload.img`拿出来，之后使用ida打开adbd。

adbd判断是否降权的依据一是调用`__android_log_is_debuggable`最终判断`ro.debuggable`，这个函数是导入函数，存在对其的包装函数，ida会将其自动识别出其名称为`.__android_log_is_debuggable`，使用G跳转到它，按`ctrl+alt+k`使用keypatch将前两行修改为

```
MOV X0, #1
RET
```

使用keypatch前需要已经安装keystone：`pip install keystone-engine`不然按此快捷键不会有反应。

![](upload/attach/202107/888604_H6UPY8XJCNMJTVY.png)

因为不想运行adb root留下`service.adb.root`属性痕迹，所以接下来还要patch掉对它的判断。

搜索对字符串`service.adb.root`的引用然后F5往下找找到判断逻辑，在if处按tab转回汇编代码会定位到条件跳转，为一行CBZ指令，将此CBZ改为B指令。

![](upload/attach/202107/888604_N2WJG9W2FB5GM3S.png)

![](upload/tmp/888604_6FE7G5QM33HY8Eupload/attach/202107/888604_HNMXKWZBTPY9DVD.png)

![](upload/attach/202107/888604_4YFUKNT2WW67GWB.png)

改完之后点击左上角菜单`Edit->Patch program->Apply patches to input file...`，如果没有备份可以选中弹窗左下角的Create backup，点击OK。

之后复制原`apex_payload.img`为`apex_payload.img.bak`后，将修改后的adbd放回`apex_payload.img`：

```python
import ext4
img = open("apex_payload.img", "rb+")
imgfs = ext4.Volume(img, offset=0)
target_adbd = imgfs.root.get_inode("bin", "adbd").open_read()
target_adbd.rewrite(open("adbd", "rb").read())
```

将`apex_payload.img`放回`com.android.adbd.apex`：

```python
import zlib
from pwn import *
apex = bytearray(open("com.android.adbd.apex", "rb").read())
org_img = open("apex_payload.img.bak", "rb").read()
org_crc = p32(zlib.crc32(org_img))
new_img = open("apex_payload.img", "rb").read()
new_crc = p32(zlib.crc32(new_img))
apex = apex.replace(org_img, new_img)
apex = apex.replace(org_crc, new_crc)
open("com.android.adbd.apex.new", "wb").write(apex)
```

用`com.android.adbd.apex.new`覆写`/system/apex/com.android.adbd.apex`：

```python
rfs = RemoteStream()
system = ext4.Volume(rfs, offset=1024*1024)
apex = system.root.get_inode("system", "apex", "com.android.adbd.apex").open_read()
apex.rewrite(open("com.android.adbd.apex.new", "rb").read())
```

### 5.3 patch selinux

之后还需要修改selinux策略使得未降权的adbd能够正常运行。该系统版本是release版本，selinux策略中不包含对`u:r:su:s0`的一系列允许策略。如果只是单纯关闭selinux，不仅会降低系统安全性，而且很容易被app发现你没有开启selinux，可能会开心地从设备上读取各种信息并送入可疑设备名单，所以这里选择patch selinux策略。

system的主要selinux策略文件在`/system/etc/selinux/plat_sepolicy.cil`，使用编辑器打开发现是文本文件，并且里面有大量自动生成的注释，这就在不能改变文件大小的条件下给我提供了可操作的空间，可以删掉注释并把自定义的selinux策略添加进去。

cil文件中基本的selinux策略语法为`(行为 主体 客体 (类别 (属性集)))`说明当主体对客体的某些类别的某些属性产生访问事件时所采取的行为。

之后问题是对`u:r:su:s0`制限解除的策略代码要怎么写，首先要找一些参考资料。

aosp中完整的sepolicy策略项目在 <mark class="encrypted">5caK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2L8X3c8J5L8$3W2V1i4K6u0W2k6$3!0G2k6$3I4W2M7$3!0#2M7X3y4W2i4K6u0W2j5$3!0E0i4K6u0r3M7r3I4S2N6r3k6G2M7X3#2Q4x3V1k6K6P5i4y4@1k6h3#2Q4x3V1k6K6k6i4m8G2L8r3W2U0P5b7`.`.</mark> ,在该项目目录下可以找到`prebuilts/api/系统api版本/public/su.te`，此文件中将dontaudit改为allow后就是我们需要的制限解除代码。

然而从te到cil还需要一层编译转换。看了看文档后发现可以选择手动编译；

te文件中类似调用函数的宏定义在`system/sepolicy/prebuilts/api/系统api版本/public/global_macros`里，类别的属性集定义在`system/sepolicy/prebuilts/api/系统api版本/private/access_vectors`，有了这两个文件就可以进行手动展开。比如：

```
allow su self:capability_class_set *;
```

可以在`global_macros`中找到`define('capability_class_set', '{ capability capability2 cap_usersns cap2_usersns }')`

展开为

```
allow su self:capability *;
allow su self:capability2 *;
allow su self:cap_usersns *;
allow su self:cap2_usersns *;
```

`capability`等类的属性集在`access_vectors`可以找到其定义

```
common cap
{
chown
dac_override
...
}
class capability
inherits cap
```

就可以继续展开并转化为cil语法

```
; allow su self:capability_class_set *;
(allow su self (capability (chown dac_override dac_read_search fowner fsetid kill setgid setuid setpcap linux_immutable net_bind_service net_broadcast net_admin net_raw ipc_lock ipc_owner sys_module sys_rawio sys_chroot sys_ptrace sys_pacct sys_admin sys_boot sys_nice sys_resource sys_time sys_tty_config mknod lease audit_write audit_control setfcap)))
(allow su self (capability2 (mac_override mac_admin syslog wake_alarm block_suspend audit_read)))
(allow su self (cap_usersns (chown dac_override dac_read_search fowner fsetid kill setgid setuid setpcap linux_immutable net_bind_service net_broadcast net_admin net_raw ipc_lock ipc_owner sys_module sys_rawio sys_chroot sys_ptrace sys_pacct sys_admin sys_boot sys_nice sys_resource sys_time sys_tty_config mknod lease audit_write audit_control setfcap)))
(allow su self (cap2_usersns (mac_override mac_admin syslog wake_alarm block_suspend audit_read)))
```

te文件中还有一种语法是`typeattribute su 名称`，对应到cil中会变成`(typeattributeset 名称 (类型集))`在类型集中要将su添加进去。

结合这两种修改方式，可以将`su.te`中的基本策略展开为`su.cil`，并将`typeattribute`定义写进`typeattribute.txt`（见附件）

最终写出修改`plat_sepolicy.cil`的代码：

```python
import ext4
from pwn import *
class RemoteStream:
def __init__(self) -> None:
self.remote = remote('127.0.0.1', 4096)
def read(self, size) -> bytes:
self.remote.send(p32(0) + p64(size))
return self.remote.recvn(size)
def seek(self, offset, whence):
self.remote.send(p32(1) + p64(offset))
def write(self, data):
self.remote.send(p32(2) + p64(len(data)))
self.remote.send(data)
def tell(self) -> int:
self.remote.send(p32(3) + p64(0))
return u64(self.remote.recvn(8))
def open(self, path):
self.remote.send(p32(4) + p64(len(path)))
self.remote.send(path)
def flush(self):
pass
def zipcil():
orgcil = open('plat_sepolicy.cil').readlines()
newcil = []
for line in orgcil:
line = line.strip()
if line.startswith(';'):
pass
elif len(line):
newcil.append(line.encode('latin-1'))
open('plat_sepolicy_ziped.cil', 'wb').write(b'\n'.join(newcil))
def build_newcil():
oldlen = len(open('plat_sepolicy.cil').read())
sepolicy = open('plat_sepolicy_ziped.cil').read()
typeattributes = open('typeattribute.txt').readlines()
for line in typeattributes:
attr_name = line.strip().split(' ')[-1]
pre = "typeattributeset " + attr_name + " ("
end = ")"
start = sepolicy.find(pre) + len(pre)
end = sepolicy.find(end, start)
if start == len(pre)-1:
sepolicy = sepolicy + '\n(typeattributeset ' + attr_name + " (su))"
else:
groups = sepolicy[start:end].split(' ')
if 'su' not in groups:
sepolicy = sepolicy[:start] + 'su ' + sepolicy[start:]
payload = open('su.cil').read()
sepolicy = sepolicy + '\n' + payload + '\n;'
sepolicy += ' '* (oldlen - len(sepolicy))
open('plat_sepolicy_patched.cil', 'wb').write(sepolicy.encode('latin-1'))
def rwrite():
s = RemoteStream()
system = ext4.Volume(s, offset=1024*1024)
sepolicy = system.root.get_inode("system", "etc", "selinux", "plat_sepolicy.cil").open_read()
sepolicy.rewrite(open('plat_sepolicy_patched.cil', 'rb').read())
zipcil()
build_newcil()
rwrite()
```

运行之后重启并进入android系统，运行adb shell：

![](upload/attach/202107/888604_HNMXKWZBTPY9DVD.png)

## 6. 将system挂载为可写

获取到root权限后发现仍然无法将system挂载为可写，一番查阅发现该系统镜像使用了`EXT4_FEATURE_RO_COMPAT_SHARED_BLOCKS`特性。虽然可以通过使用`RemoteStream`大法强行更改ext4 superblock结构的`s_feature_ro_compat`去掉`EXT4_FEATURE_RO_COMPAT_SHARED_BLOCKS`(0x4000)从而可以挂载为可写，但是要删除原有文件释放空间的话仍然会出现问题。因为该特性正如其名`SHARED_BLOCKS`，各个文件之间相同内容的块（ext4下块大小为4096）被合并了，在文件驱动无视该特性删除文件时，其他含有相同内容的文件也会被破坏。

> 可以通过如下代码去掉`EXT4_FEATURE_RO_COMPAT_SHARED_BLOCKS`并将系统挂载为可写：
