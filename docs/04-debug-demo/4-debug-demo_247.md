
# IncrediBuild 2.40 3.20 无限期试用的办法

> 原文：[`www.pediy.com/kssd/pediy10/64658.html`](https://www.pediy.com/kssd/pediy10/64658.html)

`IncrediBuild` 是一个很强的分布式编译工具，可以明显缩短大型项目编译时间，但是价格不菲。对于我这样的穷人来说，只能使用试用版。试用期限是 30 天，30 天到了即使删掉再安装仍然不能使用。给 `Xoreax` 写信申请延长试用期限，也没给答复，估计针对个人他们根本就不让延长试用。

令人郁闷的是，网上能找到的所有破解都是无效的。即使界面显示已经破解，但是时间一到，功能根本不正常。根本不会把编译任务分发给别人，只能本机编译了。

`IncrediBuild` 2.40 的 `License` 有 2 个文件 `CoordLicense.dat` 和 `AgentLicense.dat`，分别位于 `Coordinator` 和 `Agent` 安装目录下，这两个文件都是 `RSA` 数字签名过的，除非修改 `.exe` 文件中的解密密钥，否则没法伪造 `License` 文件。但既然网上能找到的破解都无法正常使用，所以肯定不容易搞定。对于 3.20 应该也大同小异。

`IncrediBuild` 在第一次运行的时候会向注册表中写入软件到期的时间。

2.40: `HKCR\Interface\{E9B0227F-437C-4F7A-86D9-2676B83F359F}\ProxyStubClsid32 = {M1-M2-M3-T1-T2}`

3.20: `HKCR\Interface\{B7348B5D-B65D-4BF5-AF63-A3135249ACA7}\ProxyStubClsid32 = {M1-M2-M3-T1-T2}`

卸载软件的时候并不会卸载这个注册表项，所以重新安装仍然不能使用。最简单的办法是卸载软件后手动删除这个注册表项，然后重新安装，就又可以继续试用。还有一种办法就是，我们定期更新上面这个注册表项的值，把时间往后推移。还好该软件时间算法并不复杂，很容易算出来。

比如说到期时间是 2008.5.30 日 23:59:59，可以写两行简单的代码：

```
COleDateTime DateTime(2008, 5, 30, 23, 59, 59);
DATE Date = (DATE)DateTime;
```

此时 `Date` 的值是 `39598.999988425923 (0x37BA E7FFDF55E340)`

`T1`: `37BA`

`T2`: `E7FFDF55E340`

`M1` = `37 * BA * E7 * FF = 23EAEB06`

`M2` = `DF * 55 = 4A0B`

`M3` = `E3 * 40 = 38C0`

这样我们就可以把注册表中上述键值改为：`{23EAEB06-4A0B-38C0-37BA-E7FFDF55E340}`

这样，软件到了 2008.5.31 00:00:00 才会过期。

更正一下，应该是修改:

```
[HKEY_CLASSES_ROOT\Interface\{B7348B5D-B65D-4BF5-AF63-A3135249ACA7}\ProxyStubClsid32]
@="{0F509010-1FF2-38C0-7D58-E5665E57E340}"
```

改为: `@="{23EAEB06-4A0B-38C0-37BA-E7FFDF55E340}"`

才是试用到 2008-5-30

延长时间的 `*.ib_lic` 可以用来分析 `Packet` 格式. 替换 `N/E` 后, 可以制造其他日期的 `Keygen` (需要补丁公钥, 和其他跑不出来 `PQ` 的 `RSA` 算法程序一样).

不过确实难以申请到, 找有安全的企业邮箱的人帮忙不知道是否可以申请来.

这个文件的格式是

```
tagLicHeader = packet record
  LicenseFileVersion: Integer; // 1
  LicenseFileLength: Integer; // whole ib_lic file size
  ContentChecksum: Int64; // unsigned byte checksum of content
end;
```

`content` 是密文, 用私钥进行 `RSA` 加密后得来, 明文的具体格式未知, 只知道是若干 `TLicensePacket` 的堆叠

`PDWORD(Packet[0])^ = 2; //format version`

`Packet[16]` 开始的 16 个字节是 `UUID`, 格式就是 `UUID` 并非字符串, 内容是 `{713F4A0C-C92A-4C75-9366-9ED08B15BEF4}`

接下来是个长度, 下面应该是各个 `Feathur` 的描述了, 不过没有试用 `lic`, 分析太耗时间.
