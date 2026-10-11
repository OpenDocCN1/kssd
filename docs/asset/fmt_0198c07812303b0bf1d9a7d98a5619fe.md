
# CASPR 1.0 说明书

> 原文：[`www.pediy.com/kssd/pediy05/pediy50447.htm`](https://www.pediy.com/kssd/pediy05/pediy50447.htm)

`CASPR` v1.100
一个非常酷的 `ASProducts` 脱壳器
(c) 2000,2001 Coded by `SAC/UG2001!`
所有权利保留

## I. 说明

`ASProducts` 产品包括 `ASPack` 和 `ASProtect`。

`ASPack` 是一个非常好的 `win32` 程序加壳程序，自从 `v2.11` 版本后 `ASPack` 一直使用 `Win95.Marburg` 的算法器。

`ASProtect` 是一个非常强劲的 `win32` 程序保护器，它包含了压缩、加密、反调试代码、反汇编、校验和检测、反 `dump` 等功能，它使用非常高效的加密算法如 `Blowfish`、`Twofish`、`TEA` 等等，并且使用著名的公开密钥算法 `RSA1024` - 一个世界上非常著名的公开密钥算法理论。

算法 -- 作为产生注册号的手段，它通过 `API` 钩子包含输入表钩子和输出表钩子与应用程序进行通信。然而，自从 `v1.1` 版本后，`ASProtect` 使用了 `Win95.Marburg` 的算法器，`ASProtect` V1.11c 版加入了反 `API` 钩子代码和 `BPE32` 的算法器。

现在一个大的问题是：它究竟是一个商业保护器还是一个超级病毒？

`CASPR` 是一个非常好的 `ASProducts` 的脱壳器，它是一群很有经验的 `cracker` 们设计出来（可不是什么新手），非常抱歉，它的源代码不能提供。

## II. 功能

* 支持 `ASProtect` v0.9xb - v1.2x 和 `ASPack` v1.00b - v2.11x
* 自解密和自反压缩代码
* 输入表重建器
* 输出表重建器
* 资源搜索器
* x86 处理器模拟
* 兼容 `Win32`(Win9x/Me/NT/2k/XP)

## III. 用法

`CASPR` 没有 `GUI` 界面,你必须在命令行下运行它:

```
CASPR \[-|/S\] \[-|/L\] <input> \[\[-|/P\] output\]

       -S        扫描子目录下的文件
       -L        打开导入文件目录下的 CASPR.INI 文件
       input     可以包含通配符(*和?)
       -P        输出路径名
       output    如果输入文件包含多个文件，输出是文件名（不是路径），那么输出仅对第一个文件有效
```

`CASPR.INI` 包含如下内容:

```ini
[Options]
Import=0x80000707
       Bit 31: 忽略输入表
   Bits 12-30: 未使用
       Bit 11: IIDDN_RVA 是可用的.(位 3 必须被设置)
       Bit 10: IIBN_RVA 是可用的.
       Bit 9 : IIOFTD_RVA 是可用的
       Bit 8 : IID_RVA 是可用的，不过现在你可以忽略它
     Bits 4-7: 未使用
       Bit 3 : 生成另一个 form，所有 DLL 动态链接库名被放在一起
       Bit 2 : 生成 original thunk 数据
       Bit 1 : 生成缩短的输入表
       Bit 0 : 在最后一个块中放上除 IAT 外的输入表

Export=0x80000101
       Bit 31: 忽略输出表
    Bits 9-30: 未使用
       Bit 8 : IED_RVA 是可用的
     Bits 1-7: 未使用
       Bit 0 : 把输出表放到最后一个块中

TLS=0x80000101
       Bit 31: 忽略 TLS
    Bits 9-30: 未使用
       Bit 8 : ITLSD_RVA 是可用的
     Bits 1-7: 未使用
       Bit 0 : 不为 TLS 进行搜索. 把 TLS 放入最后一个块中

Resource=0x80000301
       Bit 31: 忽略资源
   Bits 10-30: 未使用
       Bit 9 : IRData_RVA 是可用的
       Bit 8 : IRD_RVA 是可用的，在后面它会被用到
     Bits 1-7: 未使用
       Bit 0 : 不重建资源，把可移动的资源放入最后一个块中

Relocation=0x80000102
       Bit 31: 忽略重定位
   Bits 10-30: 未使用
       Bit 8 : IR_RVA 是可用的，在后面它会被用到
     Bits 2-7: 未使用
       Bit 1 : 如果重定位不存在，那么就不要设置 IMAGE_FILE_RELOCS_STRIPPED
       Bit 0 : 未使用

Overlay=0x00000001
   Bits 1-31: 未使用
       Bit 0 : 复制 overlay
```

*注意: 一些选项仅对 `ASProtect` 提供*

```ini
[RVAs]
   IID=0x00000000
       IMAGE_IMPORT_DESCRIPTOR 的起始地址
       现在你可以忽略它

IIOFTD=0x00000000
       IMAGE_IMPORT_OriginalFirstThunk_DATAs 的起始地址

  IIBN=0x00000000
       IMAGE_IMPORT_BY_NAMES 的起始地址

 IIDDN=0x00000000
       IMAGE_IMPORT_DESCRIPTOR.DLLNAME 的起始地址

   IED=0x00000000
       IMAGE_EXPORT_DIRECTORY 的起始地址

 ITLSD=0x00000000
       IMAGE_TLS_DIRECTORY 的起始地址

   IRD=0x00000000
       IMAGE_RESOURCE_DIRECTORY 的起始地址
       在后面会用到

IRData=0x00000000
       IMAGE_RESOURCE_DATA 的起始地址

    IR=0x00000000
       IMAGE_RELOCATION 的起始地址
       在后面会用到
```

*注意: 所有的 `RVA` 值必须定位于基址（`ImageBase`）和 `ASPack`/`ASProtect` 的块之间，否则他们将会被忽略。开动你的大脑，设置它们时请千万小心！*

## IV. 已知 BUGS

1. 块名和块属性可能会被错误删除或修改，你仅可凭自己的经验发现它们出现问题
2. 如果被保护程序使用了 `ASProtect` 的 `API` ，请等待到我的 `ASProtect`'s `API` 模拟器结束或者你自己手工去修复它们
   (看看 `ASProtect` 的帮助和实例是非常有帮助的)
3. 如果被保护程序的一个执行代码被加密，除非你得到了一个真实的 `key`，否则你不能发现它们(这可不是我的 `bug`)
4. `ASProducts` 有许多不同的版本，`CASPR` 不能全部检测到它们，如果你发现一个未知版本，请给我打电话或发电子邮件。

*注意：我想收集 `ASProducts` 的所有版本，请能帮帮我？*

## V. 计划

* `ASProtect` 的 `API`(输入钩子和输出钩子) 模拟器 - 为初学者使用的 `CASPR` `GUI` 界面版本。

## VI. 感谢

`Christoph Gabler`, `Snow Panther`, `ReDragon` and all `UG` 成员们,
`EliCZ`(Win32 Pioneer), `Aaron`, `totnak`, `hying`, `A.E.`, `Roman A.`, `sNoOFy`, `zadira`,
`YMY`, `Aleph`, `A. Ikonnikov`, `Centurion`, `BOT-CK`, `trip commando`,`Nikolay K.`,
`TiMeRiDeR` 和更多的幕后人员。

特别感谢 `Alexey Solodovnikov`: 请你更加努力工作!

## VII. 和我联系

电子邮件: `SAC@heremail.com`
主页: `http://SAC.mainpage.net`

## VIII. 历史 (与 ASProducts 之战)

* `+` 新增的
* `-` 除去的
* `!` 升级的
* `?` 改进的

05.25.2001  v1.100 beta1  非常对不起，让那些给我帮助的朋友们久等了，因为我太懒了！

```
! 供应许多 ASProtect 的旧版本和一些新版本
  (谢谢 Centurion, BOT-CK, trip commando,Nikolay K.和许多其他人员)
! 支持 4 个其它 ASPack 版本
  (谢谢 A. Ikonnikov ,YMY 和 TiMeRiDeR)
! 输出表重建器
+ 命令行参数: -S
+ 选项: Overlay
?小的 bugs
```

*注  : 以前的一些历史忽略了！*

经译者测试，此版本的 `ASProtect` 壳适用范围：`ASProtect` 1.1-1.2r

欢迎访问译者网站：`http://poorss.onchina.net`

rgbsky 译
2002.10.15  `4dd466d0e1e0cc123afbe504da2462ca`
