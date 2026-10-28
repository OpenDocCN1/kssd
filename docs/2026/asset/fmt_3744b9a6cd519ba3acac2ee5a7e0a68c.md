
`NdisFilterHandle`句柄。如果一个状态指示是一个 OID 请求相关的（下层请求一个 OID 下层要做相应的状态指示），那么状态的`DestinationHandle`和`RequestId`成员要设置成上层的 OID 请求包携带的数据。

`Filter Driver`调用`NdisFIndicateStatus`后，`NDIS`会调用相邻上层的状态指示函数（`ProtocolStatusEx`或`FilterStatus`）。

## 3 安装 NDIS Filter Drivers

这节主要介绍如何安装 NDIS Filter Driver 的信息。Filter Driver 安装不同于中间层过滤驱动，配置管理器把 Filter Driver 安装成自启动的服务类似于协议驱动。对于每一个微端口适配器（Miniport Adapter）为 NDIS 提供一个 Filter Module 的列表。Filter Driver 不像中间层过滤驱动那样联结一个虚拟设备（或虚拟微端口(virtual miniport)）。

要安装一个 Filter Driver 需要提供一个 INF 的配置文件。配置管理器从这个配置文件中读取相关的配置信息并复制到注册表。与此不同的是中间层过滤驱动，它有两个配置文件，一个提供上边缘协议驱动的接口配置信息，另一个提供下边缘虚拟微端口驱动的接口配置信息。

Filter Driver 的 INF 配置文件类似于中间层过滤驱动的协议 INF 配置文件（Prolocol INF File），它没有微端口 INF 配置文件（Miniport INF File）。

下面的小节将介绍如何为一个 Filter Driver 生成一个 INF 文件。

### 3.1 指定 Filter Driver 的绑定关系

在一个网络驱动的 INF 文件中，`UpperRanget`条目列出可能的上边缘绑定和`LowerRange`条目列出可能的上边缘绑定。这些条目包含都是系统预定义的值。

对 Filter Driver 来说，你必须设置`UpperRange`和`LowerRange`条目分别为`noupper`和`nolower`。下面示例了一个 Filter Driver 的 INF 文件的条目。

```
HKR, Ndi\Interfaces,UpperRange,,"noupper"
HKR, Ndi\Interfaces,LowerRange,,"nolower"
```

在一个 Filter Driver 的 INF 文件中，`FilterMediaTypes`条目定义了 Filter Driver 要绑定的驱动，它指定了 Filter Driver 要服务的媒体类型。下面图示列出在一个 Filter Driver 的 INF 文件中的`FilterMediaTypes`定义及`FilterMediaTypes`可取值的列表。

```
HKR, Ndi\Interfaces, FilterMediaTypes,,"ethernet"
```

`FilterMediaTypes`可取值的列表，它和`LowerRange`的一样。

![](https://bbs.kanxue.com/upload/attach/201402/203861_pya15wo6m1cgiyn.png)

![](https://bbs.kanxue.com/upload/attach/201402/203861_gi5mkn08addfbrw.png)

当计算机加载一个 Filter Driver 时，Filter Driver 要被插入到已经存在的协议到适配器的绑定（Protocol‐to‐Adapter Banding），这时就依赖`FilterMediaTypes`提供一个列表来决定插入到哪些协议到适配器的绑定上。

### 3.2 Filter Driver 的 INF 文件设置

一个 Filter Driver 的 INF 设置指定了它的特性，例如：可以指定这个驱动是 Modifying 还是 Monitoring 的，是 Mandatory 还是 Optional 的。此处，在 INF 文件中还可以指定 Filter Driver 的 FilterModule 的通用配置参数和其它有具体微端口适配口相关联的细节信息。Filter Drivr 还要指定一个唯一的过滤器识别 GUID（FID GUID）

下面两个小节分别示例如何配置 Monitoring 和 Modifying 两种类型的 Filter Driver。

#### 3.2.1 Monitoring Filter Driver 的 INF 文件配置

下面 Filter Driver 的安装节指定它为一个 Monitoring 类型的 Filter Driver:

1) 设置 INF 文件的`Class`条目为`NetService`（同时要指定`ClassGUID`）下面是一个示例。

```
Class       = NetService
ClassGUID      = {4D36E974‐E325‐11CE‐BFC1‐08002BE10318}
```

2) 必须设置安装节（DDInstall,这个名字不 INF 文件中可能有不同）的`Characteristics`条目，下边示例一个设置

```
[Install]
AddReg=Inst_Ndi
Characteristics=0x40000
NetCfgInstanceId="{5cbf81bd‐5055‐47cd‐9055‐a76b2b4e3697}"
Copyfiles = NdisMon.copyfiles.sys
```

`0x40000`这个是指`NCF_LW_FILTER`集合，一个 Filter Driver 不设置为`NCF_FILTER(0x400)`这个标记

3) 设置`NetCfgInstanceId`条目，这个 Id 和 Filter Driver 中调用`NdisFRegisterFilterDriver`是指定 Filter Driver 的唯一名保持一样。它可以用`Uuidgent.exe`来生成。下面示例这个条目的配置。

```
[Install]
AddReg=Inst_Ndi
Characteristics=0x40000
NetCfgInstanceId="{5cbf81bd‐5055‐47cd‐9055‐a76b2b4e3697}"
Copyfiles = NdisMon.copyfiles.sys
```

4) 一个安装节（DDInstall,这个名字不 INF 文件中可能有不同）必须指定一个`AddReg`条目用于配置`Ndi`键的信息，在`Ndi`节下面必须指定`Service`条目。下面是一个示例。

```
[Install]
AddReg=Inst_Ndi
Characteristics=0x40000
NetCfgInstanceId="{5cbf81bd‐5055‐47cd‐9055‐a76b2b4e3697}"
Copyfiles = NdisMon.copyfiles.sys
[Inst_Ndi]
HKR, Ndi,Service,,"NdisMon"
HKR, Ndi,CoServices,0x00010000,"NdisMon"
HKR, Ndi,HelpText,,%NdisMon_HelpText%
HKR, Ndi,FilterClass,, compression
HKR, Ndi,FilterType,0x00010001,0x00000002
HKR, Ndi\Interfaces,UpperRange,,"noupper"
HKR, Ndi\Interfaces,LowerRange,,"nolower"
HKR, Ndi\Interfaces, FilterMediaTypes,,"ethernet"
HKR, Ndi,FilterRunType, 0x00010001, 1
```

5) Filter Driver 的安装节必须指定`FilterType`和`FilterRunType`条目，下面示例了将 Filter Driver 配置为 Monitoring 且 Mandatory 的配置方法

```
[Inst_Ndi]
HKR, Ndi,Service,,"NdisMon"
HKR, Ndi,CoServices,0x00010000,"NdisMon"
HKR, Ndi,HelpText,,%NdisMon_HelpText%
HKR, Ndi,FilterClass,, compression
HKR, Ndi,FilterType,0x00010001,0x00000001
HKR, Ndi\Interfaces,UpperRange,,"noupper"
HKR, Ndi\Interfaces,LowerRange,,"nolower"
HKR, Ndi\Interfaces, FilterMediaTypes,,"ethernet"
HKR, Ndi,FilterRunType, 0x00010001, 1
```

`FilterType`为`0x00000001`说是一个 Monitoring 类型的 Filter Driver，`FilterRunType`为`1`是以 Mandatory 加载，为`2`是以 Optional 方式加载。

6) 下面示例如何为一个 Filter Driver 在系统中添加一个服务。

```
[Install.Services]
AddService=NdisMon,,NdisMon_Service_Inst

[NdisMon_Service_Inst]
DisplayName          = %NdisMon_Desc%
ServiceType          = 1 ;SERVICE_KERNEL_DRIVER
StartType              = 1 ;SERVICE_SYSTEM_START
ErrorControl        = 1 ;SERVICE_ERROR_NORMAL
ServiceBinary      = %12%\NdisMon.sys
LoadOrderGroup    = NDIS
Description          = %NdisMon_Desc%
AddReg                    = Common.Params.reg
```

7）INF 文件中要在`CoService`条目中为 Filter Driver 指定主服务名（即上一步创建的服务），下面示例。

```
[Inst_Ndi]
HKR, Ndi,Service,,"NdisMon"
HKR, Ndi,CoServices,0x00010000,"NdisMon"
HKR, Ndi,HelpText,,%NdisMon_HelpText%
HKR, Ndi,FilterClass,, compression
HKR, Ndi,FilterType,0x00010001,0x00000002
HKR, Ndi\Interfaces,UpperRange,,"noupper"
HKR, Ndi\Interfaces,LowerRange,,"nolower"
HKR, Ndi\Interfaces, FilterMediaTypes,,"ethernet"
HKR, Ndi,FilterRunType, 0x00010001, 1
```

8）在 INF 文件中定`FilterClass`的值，指定 Filter Driver 在驱动栈上的顺序（位置），但是 Monitoring 的 Filter Driver 不必指定，它固定加载到离微端口适配口最近的位置。

9）必须为 Filter 指定绑定的信息

```
[Inst_Ndi]
HKR, Ndi,Service,,"NdisMon"
HKR, Ndi,CoServices,0x00010000,"NdisMon"
HKR, Ndi,HelpText,,%NdisMon_HelpText%
HKR, Ndi,FilterClass,, compression
HKR, Ndi,FilterType,0x00010001,0x00000002
HKR, Ndi\Interfaces,UpperRange,,"noupper"
HKR, Ndi\Interfaces,LowerRange,,"nolower"
HKR, Ndi\Interfaces, FilterMediaTypes,,"ethernet"
HKR, Ndi,FilterRunType, 0x00010001, 1
```

10）为 Filter Driver 指定通用参数，这些参数可以是与具体网络适配器相关的也可以是具体实例（Filter Module）相关的，下面为示例。

```
[Common.Params.reg]

HKR, FilterDriverParams\DriverParam,  ParamDesc,    , "Driverparam for Mon"
HKR, FilterDriverParams\DriverParam,  default,    , "5"
HKR, FilterDriverParams\DriverParam,  type,       , "int"
HKR, FilterAdapterParams\AdapterParam,     ParamDesc,    , "Adapterparam for Mon"
HKR, FilterAdapterParams\AdapterParam,  default,    , "10"
HKR, FilterAdapterParams\AdapterParam,  type,   , "int"
HKR, FilterInstanceParams\InstanceParam,        ParamDesc, , "Instance param for lwf"
HKR, FilterInstanceParams\InstanceParam, default, , "15"
HKR, FilterInstanceParams\InstanceParam, type, , "int"
```

#### 3.2.2 Modifying Filter Driver 的 INF 文件配置

下面 Filter Driver 的安装节指定它为一个 Modifying 类型的 Filter Driver:

1) 设置 INF 文件的`Class`条目为`NetService`（同时要指定`ClassGUID`）下面是一个示例。

```
Class     = NetService
ClassGUID      = {4D36E974‐E325‐11CE‐BFC1‐08002BE10318}
```

2) 必须设置安装节（DDInstall,这个名字不 INF 文件中可能有不同）的`Characteristics`条目，下边示例一个设置

```
[Install]
AddReg=Inst_Ndi
Characteristics=0x40000
NetCfgInstanceId="{5cbf81bd‐5055‐47cd‐9055‐a76b2b4e3697}"
Copyfiles = NdisMon.copyfiles.sys
```

`0x40000`这个是指`NCF_LW_FILTER`集合，一个 Filter Driver 不设置为`NCF_FILTER(0x400)`这个标记

3) 设置`NetCfgInstanceId`条目，这个 Id 和 Filter Driver 中调用`NdisFRegisterFilterDriver`是指定 Filter Driver 的唯一名保持一样。它可以用`Uuidgent.exe`来生成。下面示例这个条目的配置。

```
[Install]
AddReg=Inst_Ndi
Characteristics=0x40000
NetCfgInstanceId="{5cbf81bd‐5055‐47cd‐9055‐a76b2b4e3697}"
Copyfiles = NdisMon.copyfiles.sys
```

4) 一个安装节（DDInstall,这个名字不 INF 文件中可能有不同）必须指定一个`AddReg`条目用于配置`Ndi`键的信息，在`Ndi`节下面必须指定`Service`条目。下面是一个示例。

```
[Install]
AddReg=Inst_Ndi
Characteristics=0x40000
NetCfgInstanceId="{5cbf81bd‐5055‐47cd‐9055‐a76b2b4e3697}"
Copyfiles = NdisMon.copyfiles.sys
[Inst_Ndi]
HKR, Ndi,Service,,"NdisLwf"
HKR, Ndi,CoServices,0x00010000,"NdisLwf"
HKR, Ndi,HelpText,,%NdisLwf_HelpText%
HKR, Ndi,FilterClass,, compression
HKR, Ndi,FilterType,0x00010001,0x00000002
HKR, Ndi\Interfaces,UpperRange,,"noupper"
HKR, Ndi\Interfaces,LowerRange,,"nolower"
HKR, Ndi\Interfaces, FilterMediaTypes,,"ethernet"
HKR, Ndi,FilterRunType, 0x00010001, 1
```

5）Filter Driver 的安装节必须指定`FilterType`和`FilterRunType`条目，下面示例了将 Filter Driver 配置为 Monitoring 且 Mandatory 的配置方法
