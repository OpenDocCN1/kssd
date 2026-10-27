
# [原创]wibu软授权(一)

> 本节将描述wibu软授权中可能用到的一些文件，在初次接触wibu软授权的伙伴可以通过本文快速找准自己的方向。
>
> 部分描述均为个人猜测，仅作参考。

## WibuCmLIF文件

WibuCmLIF文件又称为LIF文件，其记录了待授权软件的信息、配置、环境要求等。一般地，WibuCmLIF文件会跟随软件一同发布，所以在软授权过程中，我们首先接触的就是WibuCmLIF文件。

以下是某份WibuCmLIF文件的内容：

```ini
[WIBU-SYSTEMS Control File]
Guid={00070001-0000-1100-8005-0000C06B5161}
Description=CodeMeter License Transfer Generation Information
Version=1.00
Encoding=UTF-8
[Info]
CreationTime=2020-10-27 21:26:09
TimeStamp=1603805169
LicenseInformationSize=1204
[LicenseInformation]
UZ2c7xT77/9BygbMO0cf8CM9VG1gdIywuAHhAjqn9rIxvyeP15rvdGjLGEzcaxYhANuJX1np0X25fTEyK8X3CU5r/f2NaEUPqq8g
gwLb0AKenTv2/d6Q9njQExlKvhwHFDQXoPLg0Ak7fc/8GhzZu8Pxq0biHmg82ToBYI4WTTdDENQNuFzzSwiqabadsq163iKfuVIb
7qg5DAIoihMb1NHNSRYypkIM6cmZbFoWh4mBFgN5Al0/H5n3oDUqJhq1MY4t39JouPhJ0EXW3J6vEsneum6mnMROyJPNfcrFx9SR
tjeRZrB1crnOk2Bbcm7cQMiDOEIYKipYJw34YHRf5fCKRg5Ojhnoi2Q7bbis0ZAjBadl4a90ert/tGxUuKgvV2wn//q4KYxq1Cpx
6xvENAdpQnSIJfliUHUgOfm3/ML0mJSCXzY0psoI2E12nDXdDhNhFTVAcGRJO58xenxu2iHvpMH/DkbdG87SfCl/Rmz4pgJ68w1E
aN/2XHDg3VG0OMI9JsFEWA/M+RQ6ohMj9yJ/NiZug3ChW4x26/gQjLuyzVMEXJWuPPmFmmvcbc2Vg+5CNSkpcU24F8HcLmeP+WFu
i8d77K99Dl3oCsNoUmQaXbNzHEpwhkThXewmo9a+lHsHCAoswUZs6Q5ukxmrMZbMzW4IVtwmITeQ/j+O+LC4HZf3OHGf6LJoYXS4
D0IDLa/KcffMAXPi6cIIqT6nV/eH1g6kYzPde7eir8eoNbr+8QMXhKdBVRtzs/WgPbvmN4gWjlYgdYCI+MsEeJghBzxHX1V7fxOz
Bh+V1vunRbXQhOsX1dzHHMkKaiwHdzRWYKzr6BetVAkeJEyu7R1hG4aSlmFmCbZDTruY1rs/YckgqwQcYMgIMz3GK0M5vPUE2c5X
VPy/bIzrbpYGT+IbNoIv6GsOKYFL2xo253PxdOh8JX7+4uc8UhwCXcb1bMu8lLo9Fc1M0weNK2Gh+sGHgqCq4XJCOH/ImfVS+cDa
ml10fCyFc+an0GB7EwRaZTs7wW4Ic0fN9BwOq/sqCr7QBZCWRJ/kJGnzun1GbFqY9kRhqj0QmLZsFFSOLyjfQ35QAaFlVIIn5tkw
MLaqlZtrQISWsSa75pnWCpL2DvtPT3UKrYJ8e4ObLZB7W4Ho/AKAw4JDa4lykbXtaI19cm2OZMHU+fgG1x9AW0s2tKo+8Ilc76Un
FdVPYoIBcOnSZ2s8KbqdnCQ0Qx+vyaJZKctQ1OEACXlnQsky6Mazlmp8mfmBNfVfAfd9DpiL/rwv7FERPmGpT9QEuUNIpUeFIW1o
NODjIAm5GosMJk+mfFtKm3qelVl9c5mEpPchK/4bpWg2cVKj+GhXwdRkDSFOaS1PnM3Jncz4PXve+chit5uvXrtx1brD9E9S2F3c
Ysjd9GdsTezaSUcKQrXD3hb2uHovIDSVOVjiYHvPuzepnwXkYmLClG2o0EkuEjb5pcVmxLuKARYJzvkn3x07abrpTPqJLVnh/MT2
VrhdIwgnYJyZ6DvFHi0I/T+pe8Vv1WOc7cNrmfl+AXeQh3dWnq2fpAmPvB2Ypv0RQdIe3VcCK3Zri7oAbc52QqngonerSGz6+154
WAo5xw
```

我们可以看到，LIF文件的内容有点类似ini配置文件的结构，其中LicenseInformation节有逆向经验的伙伴很容易就能看出是BASE64编码的内容。

LIF文件的使用方式很简单，运行`cmu -i -f xxx.WibuCmLIF`即可。运行后，将在wibu的软授权系统加载该LIF文件并且生成与本次加载相关的文件。值得注意的时，每次加载生成的相关文件都不相同，所以程序的说明文档(如果有)会提示不要重复运行该命令。以下是使用某份LIF文件运行该命令的输出，其中的Serial number每次运行该命令都会不同。

```
cmu - CodeMeter Universal Support Tool.
Version 6.60 of 2017-Dec-18 (Build 2869) for Linux
Copyright (C) 2007-2017 by WIBU-SYSTEMS AG. All rights reserved.
The file contains 1 Update:
CmActLtLicense binding information: FirmCode 6000934
Execute Update ...
The file contains 1 Update:
CmActLtLicense update: Serial number 130-704886716, FirmCode 6000934.
--> successful
1 successful update done
```

## WibuCmRaC文件

WibuCmRaC文件又称RAC文件，它是我们接触到的第二个文件，它与当前机器的环境、状态等信息强关联。

该文件由命令`cmu -c6000934 -s130-704886716 -f context.WibuCmRaC`生成，其中的-c和-s参数由导入LIF文件时确定。RAC文件的结构和LIF文件类似。以下为运行命令结果

```
cmu - CodeMeter Universal Support Tool.
Version 6.60 of 2017-Dec-18 (Build 2869) for Linux
Copyright (C) 2007-2017 by WIBU-SYSTEMS AG. All rights reserved.
Write CmFAS for 130-704886716 for FirmItem 6000934
```

RAC文件需要提交到软件供应商处进行激活，软件供应商将返回一个RAU文件作为激活文件。

## WibuCmRaU文件

WibuCmRaU文件又称RAU文件，它是软件供应商提供的激活文件，只有LIF文件与RAC文件相匹配才能导入成功，之后就能正常使用程序。

RAU文件的导入方式和LIF文件类似，只要运行命令`cmu -i -f xxx.WibuCmRaU`即可。

LIF文件、RAC文件和RAU文件的核心内容(base64编码那部分内容)均使用asn1编码，而asn1定义位于`CodeMeterLin`中。利用asn1定义可以使用如`asn1c`(c/c++平台)、`asn1tools`(python平台)进行解析，目前除RAU文件的部分内容外，其余文件本人都进行了解析并可以正确获取内容。

RAU文件的部分内容需要使用CmAct证书的私钥解密，而CmAct证书的私钥与机器绑定(或者说与导入LIF时的环境绑定)，故需要能够接触真实运行的机器才能获取CmAct证书的私钥。只有RAU文件没有实际机器是无法解密该文件的，初次接触的小伙伴需要注意一下。

由于本人只有一份RAU文件，并且该RAU文件尚未过期，所以本人不会公开这个RAU文件。

## wbc文件

wbc文件是LIF文件导入时生成的文件，主要包含了系统特征的哈希值，并且描述了RAC文件中CmAct证书私钥的生成参数，默认情况下位于`/var/lib/CodeMeter/CmAct`。

```ini
[WIBU-SYSTEMS Control File]
Guid={000B0002-0000-1100-8005-0000C06B5161}
Description=WIBU-SYSTEMS CmAct Inventory
Version=1.00
Encoding=UTF-8
[Inventory]
Nonce=5B953B8DE30C80383851A1C4A0DCB4C08B5EA6DB5B57503C9F766B7EE75303B6
RedundancyData=4B76D4A6ADA098349EE74931AD4686F251B512998CB181346527C4625B2BA8ADEA16AEA16335F3ACC84F1C4E532D5DCC8EEE91901D9C7BCBC71506F16FF71CFBE893EBDA3FE75A0AFE8F94FB05150C26A2611DFDD9A369B326B493C2EF03A49FFA83743C2100
Version0=6
Version1=60
Version2=2869
Version3=0
Heuristic=2
Flags=0
ItemCount=13
[Item\_0]
ID=2B65AB32E1F52D647E2A6208CC3C5AD6F7E686FF6E71DE0030B2CF8F15C8AD06
Position=0
Length=67
Params=AB1A401EC3FBA4070FA7CA569EB44DE8
[Item\_1]
ID=ABBB3C53DBAB3E9545CE8E030BC4936830983EFC0E68784272E70E2DECA7A64F
Position=67
Length=112
Params=8B4FC28F523E3B201B0A21075DA160C1
[Item\_2]
ID=9B7AC9C0CCB23CE9B70E6962CBFC67A11A6399B2E3375DBAB367A6170F5359D9
Position=179
Length=42
Params=2B0E51706F32DC52323AF862ABABFE3A
[Item\_3]
ID=3BF9F76D91B518204C05927CE7EE6E500016239F4AC1F42B7D9943DBAAC043B6
Position=221
Length=35
Params=6B40611626E60FC67B30626610B878FD
[Item\_4]
ID=CBA5E1C03BC3D91F2848E79A3039C2158CE3956493B8D8E6658FFBE4DBA90756
Position=256
Length=28
Params=9BE631605A441A9B12E509782946EB78
[Item\_5]
ID=BBBB1E63F5980CC754646CA9041C2A2060F32701932ABCD1869933AB33174C66
Position=284
Length=28
Params=6B815996E45B0B54A80720D84B7F060E
[Item\_6]
ID=FB0B8A33EF4007CF54B577927E3957826C4C88B57971565CD1CD183C5B046540
Position=312
Length=28
Params=6B4B5DA0A249605BBE8FFA83ED843DA2
[Item\_7]
ID=8BA2DA67D3094E04F5810492B75FC5922A51865B38D66D33801FDA1FA08B8AE0
Position=340
Length=28
Params=1B88A2D99C44F94561C03248732EB525
[Item\_8]
ID=6BFD47ABA498564E0A8633AA00E08F3BBF2B92008BD7E3EDADCAE57A23BD7D1F
Position=368
Length=21
Params=6B10A5347FD4FCEE48C90BC1C4CA1E12
[Item\_9]
ID=5B8AF905FDBCD470CE3F87CDB02071C5D53B3A9DEA8784B77DC38AA2FEAD508A
Position=389
Length=21
Params=FB2238548392D8B0683ED68C372D80D2
[Item\_10]
ID=5BDC9C95A56535C52C34D0093DCAAFE19188748619136DF7F671FCC43E9C4CE0
Position=410
Length=45
Params=0B71A077E2DC717B14EE02E0FE904B7F
[Item\_11]
ID=6B96BDE884536CDE60A3944FA37EE56BE6FECD499A8BFDE37F42B101139B3DD2
Position=455
Length=42
Params=6B321A1A5B402142C827648B5447CEE6
[Item\_12]
ID=2B718715B76200DB0DB8B0FC2177A4B8BCC1ACF2999FF82F2E1B76499852427D
Position=497
Length=15
Params=DB7548975ACC0CD4FA7BC69F78B20164
```

wbc文件每一个Item节表示一个系统特征，当且仅当所有系统特征的哈希值与Item节的ID所对应时，该wbc文件才被认为是有效的。

每个Item节的`Position`和`Length`参数用于描述CmAct证书私钥的组成结构，所有Item节的`Length`参数加起来必须为512(比特)。`Params`字段具体作用未知。

wbc文件通常以`<firmcode>_<CmActSerial-ID>.wbc`的形式命名，firmcode在导入LIF文件时我们就遇到过，CmActSerial-ID通常由40个字符(20个字节)构成，为了便于输入，CmActSerial-ID可以缩略为Serial number。

以`6000934_820034d3a3fa6e51ce48f11c54f3e47b9c569e6d.wbc`为例，firmcode为6000934，CmActSerial-ID为820034d3a3fa6e51ce48f11c54f3e47b9c569e6d，Serial number为130-2726490540。

```python
>>> import hashlib
>>>
>>> CmActSerial\_ID = "820034d3a3fa6e51ce48f11c54f3e47b9c569e6d"
>>> part1 = int.from\_bytes(bytes.fromhex(CmActSerial\_ID)[0:2], "little")
>>> part2 = int.from\_bytes(hashlib.sha256(bytes.fromhex(CmActSerial\_ID)).digest()[0:4], "little")
>>> print("Serial number: %d-%d" % (part1, part2))
Serial number: 130-2726490540
```

wbc文件更细节的使用方式见后续章节。

## WibuCmActDyn文件

WibuCmActDyn文件又称DYN文件，同样位于`/var/lib/CodeMeter/CmAct`，命名规则与wbc文件相同。

DYN文件一份被AES128-CBC加密的文件，`IV`固定为全0，而AES密钥需要使用CmAct证书的私钥解密。

DYN文件从asn1定义上看可以记录很多信息，如激活时间、过期时间、批量授权等。

此处我的DYN文件却没有这么多信息，原因未知。

```xml
<sw-specs>
<creator-name> CodeMeter Runtime </creator-name>
<creator-version>
<sfl> 1 </sfl>
<sfh> 1 </sfh>
<feature-flags> 0 </feature-flags>
</creator-version>
<required-version>
<sfl> 1 </sfl>
<sfh> 1 </sfh>
<feature-flags> 0 </feature-flags>
</required-version>
</sw-specs>
<cmact-serial-id> 82 00 34 D3 A3 FA 6E 51 CE 48 F1 1C 54 F3 E4 7B 9C 56 9E 6D </cmact-serial-id>
<clocks>
<box-time> 579681158 </box-time>
<certified-time> 552660672 </certified-time>
</clocks>
<tag> 2 </tag>
<pi-dynamics>
</pi-dynamics>
```

## WibuCmActLic文件

WibuCmActLic文件又称LIC文件，同样位于`/var/lib/CodeMeter/CmAct`，命名规则与wbc文件相同。

LIC文件与LIF文件大同小异，LIC文件多了一些与本次LIF导入相关的字段，如`cmact-serial-id`、CmAct证书等。

## winsecurity文件

winsecurity文件位于`/var/spool/ctmp`下，文件名为随机构造。
