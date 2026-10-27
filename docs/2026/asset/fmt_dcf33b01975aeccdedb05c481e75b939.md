
winsecurity文件多为53字节和68字节，此处仅描述53字节结构，68字节未作分析。

winsecurity文件前32字节为对应wbc文件名的sha256，接下来的17字节为供wbc文件使用的数据（有效数据），最后4字节为数据类型。本人认为68字节的文件与53字节文件结构类似，只是有效数据的长度不同。

有效数据的用途还待挖掘。

# CodeMeterLin

CodeMeterLin是wibu软授权架构的核心程序，用于验证、生成软授权相关信息，采用C/S架构提供服务。

CodeMeterLin默认情况下位于`/usr/sbin`，服务端端口为`22350`。

CodeMeterLin默认情况下在后台运行（daemon模式），可以使用`-f`参数使其在前台运行。

## 解除反调试

在daemon模式下，原先的父进程会被终止，而fork出来的子进程将被init进程继承，如果动态跟踪的话，到daemon函数处就会终止，具体可查看daemon函数的手册，所以要调试CodeMeterLin最好添加`-f`参数。

运行起来的CodeMeterLin最终会在函数`sub_4F8410`的while中循环，该while语句仅在`dword_B4FC50`为0时终止。`dword_B4FC50`是检测CodeMeterLin运行环境的变量，一旦CodeMeterLin认为自身处于不安全的环境中，该值就会赋值为0，从而结束进程。

```c
__int64 __fastcall sub_4F8410(__int64 a1, __int64 a2)
{
__pid_t v2; // eax
__int64 v3; // rax
__int64 v4; // rax
char v5; // al
char v6; // dl
sub_4F8270(a1);
v2 = getpid();
byte_B736E4 = 0;
dword_B73254 = v2;
byte_B73250 = 0;
sub_4C8F70(&dword_B73700, 0LL, 1LL);
dword_B4FC50 = 1;
signal(10, handler);
signal(2, handler);
signal(15, handler);
signal(1, handler);
signal(12, handler);
v3 = sub_43EA80();
v4 = sub_43E950(v3);
sub_526940(a2, v4);
msleep(100);
signal(10, handler);
signal(12, handler);
byte_B736E4 = 1;
msleep(600);
v5 = 0;
v6 = 0;
byte_B73250 = 0;
if ( dword_B4FC50 )
{
while ( 1 )
{
if ( v5 || !v6 )
{
usleep(1500000u);
system("/usr/bin/codemeter-info -SMmp > /dev/null");
byte_B73250 = 0;
sub_4D0DE0(0x12u);
if ( byte_B73250 )
{
byte_B73250 = 0;
sub_4D0DE0(0x12u);
}
}
msleep(800);
if ( !dword_B4FC50 ) //若运行环境不安全则退出while循环
break;
v5 = byte_B73250;
v6 = 1;
}
}
dword_B4FC50 = 0;
*(_BYTE *)(qword_B70410 + 122) = 0;
msleep(1000);
return 1LL;
}
```

接下来的事情就简单了，你可以发挥你的想象来控制`dword_B4FC50`不变为0。

交叉引用寻找`dword_B4FC50`的调用点，修改`dword_B4FC50`的赋值语句，这样做仅修改一个字节即可。

```c
void set_env_unsafe()
{
dword_B4FC50 = 0; //set 1 to disable env check
nullsub_16();
}
```

交叉引用`set_env_unsafe`可以发现有很多情况会导致运行环境不安全，这里就不展开细述，有兴趣可以自己去调一调。

> 本节将描述wibu软授权中可能用到的一些文件，在初次接触wibu软授权的伙伴可以通过本文快速找准自己的方向。
>
> 部分描述均为个人猜测，仅作参考。

# WibuCmLIF文件

WibuCmLIF文件又称为LIF文件，其记录了待授权软件的信息、配置、环境要求等。一般地，WibuCmLIF文件会跟随软件一同发布，所以在软授权过程中，我们首先接触的就是WibuCmLIF文件。

以下是某份WibuCmLIF文件的内容：

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19  20  21  22  23  24  25  26  27  28  29 | `[WIBU``-``SYSTEMS Control` `File``]`  `Guid``=``{``00070001``-``0000``-``1100``-``8005``-``0000C06B5161``}`  `Description``=``CodeMeter License Transfer Generation Information`  `Version``=``1.00`  `Encoding``=``UTF``-``8`    `[Info]`  `CreationTime``=``2020``-``10``-``27` `21``:``26``:``09`  `TimeStamp``=``1603805169`  `LicenseInformationSize``=``1204`    `[LicenseInformation]`  `UZ2c7xT77``/``9BygbMO0cf8CM9VG1gdIywuAHhAjqn9rIxvyeP15rvdGjLGEzcaxYhANuJX1np0X25fTEyK8X3CU5r``/``f2NaEUPqq8g`  `gwLb0AKenTv2``/``d6Q9njQExlKvhwHFDQXoPLg0Ak7fc``/``8GhzZu8Pxq0biHmg82ToBYI4WTTdDENQNuFzzSwiqabadsq163iKfuVIb`  `7qg5DAIoihMb1NHNSRYypkIM6cmZbFoWh4mBFgN5Al0``/``H5n3oDUqJhq1MY4t39JouPhJ0EXW3J6vEsneum6mnMROyJPNfcrFx9SR`  `tjeRZrB1crnOk2Bbcm7cQMiDOEIYKipYJw34YHRf5fCKRg5Ojhnoi2Q7bbis0ZAjBadl4a90ert``/``tGxUuKgvV2wn``/``/``q4KYxq1Cpx`  `6xvENAdpQnSIJfliUHUgOfm3``/``ML0mJSCXzY0psoI2E12nDXdDhNhFTVAcGRJO58xenxu2iHvpMH``/``DkbdG87SfCl``/``Rmz4pgJ68w1E`  `aN``/``2XHDg3VG0OMI9JsFEWA``/``M``+``RQ6ohMj9yJ``/``NiZug3ChW4x26``/``gQjLuyzVMEXJWuPPmFmmvcbc2Vg``+``5CNSkpcU24F8HcLmeP``+``WFu`  `i8d77K99Dl3oCsNoUmQaXbNzHEpwhkThXewmo9a``+``lHsHCAoswUZs6Q5ukxmrMZbMzW4IVtwmITeQ``/``j``+``O``+``LC4HZf3OHGf6LJoYXS4`  `D0IDLa``/``KcffMAXPi6cIIqT6nV``/``eH1g6kYzPde7eir8eoNbr``+``8QMXhKdBVRtzs``/``WgPbvmN4gWjlYgdYCI``+``MsEeJghBzxHX1V7fxOz`  `Bh``+``V1vunRbXQhOsX1dzHHMkKaiwHdzRWYKzr6BetVAkeJEyu7R1hG4aSlmFmCbZDTruY1rs``/``YckgqwQcYMgIMz3GK0M5vPUE2c5X`  `VPy``/``bIzrbpYGT``+``IbNoIv6GsOKYFL2xo253PxdOh8JX7``+``4uc8UhwCXcb1bMu8lLo9Fc1M0weNK2Gh``+``sGHgqCq4XJCOH``/``ImfVS``+``cDa`  `ml10fCyFc``+``an0GB7EwRaZTs7wW4Ic0fN9BwOq``/``sqCr7QBZCWRJ``/``kJGnzun1GbFqY9kRhqj0QmLZsFFSOLyjfQ35QAaFlVIIn5tkw`  `MLaqlZtrQISWsSa75pnWCpL2DvtPT3UKrYJ8e4ObLZB7W4Ho``/``AKAw4JDa4lykbXtaI19cm2OZMHU``+``fgG1x9AW0s2tKo``+``8Ilc76Un`  `FdVPYoIBcOnSZ2s8KbqdnCQ0Qx``+``vyaJZKctQ1OEACXlnQsky6Mazlmp8mfmBNfVfAfd9DpiL``/``rwv7FERPmGpT9QEuUNIpUeFIW1o`  `NODjIAm5GosMJk``+``mfFtKm3qelVl9c5mEpPchK``/``4bpWg2cVKj``+``GhXwdRkDSFOaS1PnM3Jncz4PXve``+``chit5uvXrtx1brD9E9S2F3c`  `Ysjd9GdsTezaSUcKQrXD3hb2uHovIDSVOVjiYHvPuzepnwXkYmLClG2o0EkuEjb5pcVmxLuKARYJzvkn3x07abrpTPqJLVnh``/``MT2`  `VrhdIwgnYJyZ6DvFHi0I``/``T``+``pe8Vv1WOc7cNrmfl``+``AXeQh3dWnq2fpAmPvB2Ypv0RQdIe3VcCK3Zri7oAbc52QqngonerSGz6``+``154`  `WAo5xw` |

我们可以看到，LIF文件的内容有点类似ini配置文件的结构，其中LicenseInformation节有逆向经验的伙伴很容易就能看出是BASE64编码的内容。

LIF文件的使用方式很简单，运行`cmu -i -f xxx.WibuCmLIF`即可。运行后，将在wibu的软授权系统加载该LIF文件并且生成与本次加载相关的文件。值得注意的时，每次加载生成的相关文件都不相同，所以程序的说明文档(如果有)会提示不要重复运行该命令。以下是使用某份LIF文件运行该命令的输出，其中的Serial number每次运行该命令都会不同。

|  |  |

| --- | --- |

| 1  2  3  4  5  6  7  8  9  10  11  12 | `cmu` `-` `CodeMeter Universal Support Tool.`  `Version` `6.60` `of` `2017``-``Dec``-``18` `(Build` `2869``)` `for` `Linux`  `Copyright (C)` `2007``-``2017` `by WIBU``-``SYSTEMS AG.` `All` `rights reserved.`    `The` `file` `contains` `1` `Update:`  `CmActLtLicense binding information: FirmCode` `6000934`    `Execute Update ...`  `The` `file` `contains` `1` `Update:`  `CmActLtLicense update: Serial number` `130``-``704886716``, FirmCode` `6000934.`  `-``-``> successful`  `1` `successful update done` |

# WibuCmRaC文件

WibuCmRaC文件又称RAC文件，它是我们接触到的第二个文件，它与当前机器的环境、状态等信息强关联。

该文件由命令`cmu -c6000934 -s130-704886716 -f context.WibuCmRaC`生成，其中的`-c`和`-s`参数由导入LIF文件时确定。RAC文件的结构和LIF文件类似。以下为运行命令结果

|  |  |

| --- | --- |

| 1  2  3  4  5 | `cmu` `-` `CodeMeter Universal Support Tool.`  `Version` `6.60` `of` `2017``-``Dec``-``18` `(Build` `2869``)` `for` `Linux`  `Copyright (C)` `2007``-``2017` `by WIBU``-``SYSTEMS AG.` `All` `rights reserved.`    `Write CmFAS` `for` `130``-``704886716` `for` `FirmItem` `6000934` |

RAC文件需要提交到软件供应商处进行激活，软件供应商将返回一个RAU文件作为激活文件。

# WibuCmRaU文件

WibuCmRaU文件又称RAU文件，它是软件供应商提供的激活文件，只有LIF文件与RAC文件相匹配才能导入成功，之后就能正常使用程序。

RAU文件的导入方式和LIF文件类似，只要运行命令`cmu -i -f xxx.WibuCmRaU`即可。

LIF文件、RAC文件和RAU文件的核心内容(base64编码那部分内容)均使用asn1编码，而asn1定义位于CodeMeterLin中。利用asn1定义可以使用如`asn1c`(c/c++平台)、`asn1tools`(python平台)进行解析，目前除RAU文件的部分内容外，其余文件本人都进行了解析并可以正确获取内容。

RAU文件的部分内容需要使用CmAct证书的私钥解密，而CmAct证书的私钥与机器绑定(或者说与导入LIF时的环境绑定)，故需要能够接触真实运行的机器才能获取CmAct证书的私钥。只有RAU文件没有实际机器是无法解密该文件的，初次接触的小伙伴需要注意一下。

由于本人只有一份RAU文件，并且该RAU文件尚未过期，所以本人不会公开这个RAU文件。

# wbc文件

wbc文件是LIF文件导入时生成的文件，主要包含了系统特征的哈希值，并且描述了RAC文件中CmAct证书私钥的生成参数，默认情况下位于`/var/lib/CodeMeter/CmAct`。

|  |  |

| --- | --- |
