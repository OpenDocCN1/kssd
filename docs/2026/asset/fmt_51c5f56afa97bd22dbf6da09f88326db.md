
使用正常`TPM_ALG_HMAC`类型反序列化长度是对称的最多`0x30`,导致这个缓冲区之后的数据被泄露字节是没有问题的不存在泄露,但如果比如使用一种不匹配的反序列化类型比如`TPM_ALG_EDDSA`,由于输出缓冲区是共用的,会将`CryptHmacSign`结果`signature->signature.hmac.digest`前4个字节后的2个字节(`UINT16`可以由hash结果控制)作为反序列作为下个输出反序列化字节流的长度,这个长度可以是`UINT16`最大值65535,所以就导致了可能的越界读取信息泄露或者内存破坏等拒绝服务情况.但实际上在输出缓冲区反序列化时候会对输出字节流的最大长度判断即输出最大数据大小`Sign_Out`减去当前长度,如果这个长度很大比如65535,会导致结果是负数,进入不可预知的分支崩溃断言,导致`vmsp.exe`进程崩溃(`int 3`),如下显示是崩溃时的复现结果.但如果这个值未超过输出缓冲区的最大大小,比如果小于65535会根据这个值累加返回缓冲区大小长度,当然这个值可能导致最后的缓冲区`UINT16`大小溢出,则可能导致越过`TPM_ALG_HMAC`哈希结果的最大`0x30`大小导致这个缓冲区之后的数据被泄露,但是实际上这些数据都是0,只有溢出的数据才是泄露的部分,这个结果证明信息泄露的可能性和客户端可以读取实际上的越界信息泄露.`CVE-2025-2884`通过`bindiff`分析可以找到相同的补丁修复位置,手动在`windbg`把验证的代码绕过就可以触发到达漏洞的复现位置,复现效果为和未打补丁前相同,其他tpm通用平台漏洞复现也可以使用类似poc代码实现.

## 漏洞复现栈回溯

```
> !analyze -v
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
\* \*
\* Exception Analysis \*
\* \*
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
KEY\_VALUES\_STRING: 1
Key : Analysis.CPU.Sec
Value: 0
Key : Analysis.DebugAnalysisProvider.CPP
Value: Create: 8007007e on DESKTOP-9IQIGSF
Key : Analysis.DebugData
Value: CreateObject
Key : Analysis.DebugModel
Value: CreateObject
Key : Analysis.Elapsed.Sec
Value: 38
Key : Analysis.Memory.CommitPeak.Mb
Value: 67
Key : Analysis.System
Value: CreateObject
Key : Timeline.OS.Boot.DeltaSec
Value: 4128
Key : Timeline.Process.Start.DeltaSec
Value: 1698
NTGLOBALFLAG: 0
APPLICATION\_VERIFIER\_FLAGS: 0
EXCEPTION\_RECORD: (.exr -1)
ExceptionAddress: 00007ffec98ff2a2 (TpmEngUM!BYTE\_Array\_Marshal+0x00000000000000ce)
ExceptionCode: 80000003 (Break instruction exception)
ExceptionFlags: 00000000
NumberParameters: 1
Parameter[0]: 0000000000000000
FAULTING\_THREAD: 0000280c
PROCESS\_NAME: vmsp.exe
ERROR\_CODE: (NTSTATUS) 0x80000003 - { }
EXCEPTION\_CODE\_STR: 80000003
EXCEPTION\_PARAMETER1: 0000000000000000
STACK\_TEXT:
00000182`5440e270 00000000`0000003e : 00007ffe`c995018b 00007ffe`c994cfa0 00007ffe`c990efb7 00000182`5440e414 : TpmEngUM!BYTE\_Array\_Marshal+0xce
00000182`5440e278 00007ffe`c995018b : 00007ffe`c994cfa0 00007ffe`c990efb7 00000182`5440e414 00007ffe`c98ff337 : 0x3e
00000182`5440e280 00007ffe`c994cfa0 : 00007ffe`c990efb7 00000182`5440e414 00007ffe`c98ff337 00000000`00000002 : TpmEngUM!s\_requestBuffer+0x2b
00000182`5440e288 00007ffe`c990efb7 : 00000182`5440e414 00007ffe`c98ff337 00000000`00000002 00000182`5440e350 : TpmEngUM!s\_actionOutputBuffer
00000182`5440e290 00007ffe`c99072cb : 00000182`5440e410 00007ffe`c994cfa0 00000182`5440e410 00007ffe`c98f0000 : TpmEngUM!TPM2\_Sign+0xfb
00000182`5440e330 00007ffe`c98f5d7b : 00000000`0000015d 00000000`ffff0101 00007ffe`c995018b 00007ffe`c99501c9 : TpmEngUM!CommandDispatcher+0x1a3f
00000182`5440e3b0 00007ffe`c98f5135 : 00000000`00001000 00007ffe`d88d6605 00007ffe`d7ea5e00 00007ffe`d7ea5cbd : TpmEngUM!ExecuteCommand+0x34b
00000182`5440e490 00007ff6`b0c48fff : 00000000`00000069 00000182`5440eb20 00007ff6`b0c5a950 00007ffe`d886b4c6 : TpmEngUM!VTpmExecuteCommand+0xd5
00000182`5440e4f0 00000000`00000069 : 00000182`5440eb20 00007ff6`b0c5a950 00007ffe`d886b4c6 00000000`00001000 : 0x00007ff6`b0c48fff
00000182`5440e4f8 00000182`5440eb20 : 00007ff6`b0c5a950 00007ffe`d886b4c6 00000000`00001000 00000182`5440e880 : 0x69
00000182`5440e500 00007ff6`b0c5a950 : 00007ffe`d886b4c6 00000000`00001000 00000182`5440e880 00007ff6`00001000 : 0x00000182`5440eb20
00000182`5440e508 00007ffe`d886b4c6 : 00000000`00001000 00000182`5440e880 00007ff6`00001000 00008ce1`373b1774 : 0x00007ff6`b0c5a950
00000182`5440e510 00000182`5423e098 : 00000000`00001000 00000182`5440eb20 00000182`54264b80 00007ffe`d89218c6 : RPCRT4!Ndr64OutInit+0x1da
00000182`5440e580 00000000`00001000 : 00000182`5440eb20 00000182`54264b80 00007ffe`d89218c6 00000000`00000000 : 0x00000182`5423e098
00000182`5440e588 00000182`5440eb20 : 00000182`54264b80 00007ffe`d89218c6 00000000`00000000 00000182`5440e9a0 : 0x1000
00000182`5440e590 00000182`54264b80 : 00007ffe`d89218c6 00000000`00000000 00000182`5440e9a0 00000000`00000000 : 0x00000182`5440eb20
00000182`5440e598 00007ffe`d89218c6 : 00000000`00000000 00000182`5440e9a0 00000000`00000000 00000000`0000000e : 0x00000182`54264b80
00000182`5440e5a0 00000000`00000000 : 00000000`00000000 00000000`00000000 00000000`00000000 00000000`00000000 : RPCRT4!Invoke+0x26
```

## 漏洞复现

笔者漏洞poc采用uefi程序复现,出于安全原因笔者不能提供完整的poc代码,下图是笔者在的Win11 24h2物理机上使用嵌套虚拟化模式成功复现了CVE的利用效果

![](upload/attach/202510/609565_QFM77UNYSEB6SMV.webp)

## 相关引用

<a href="elink@ab2K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2U0N6X3g2Q4x3X3g2G2M7X3N6Q4x3V1k6o6g2V1g2d9k6h3y4G2M7X3c8Q4x3@1k6A6k6q4)9K6c8p5y4h3c8g2)9J5k6o6t1H3x3U0g2Q4x3X3b7J5z5o6R3@1">`CVE-2025-2884`</a>

<a href="elink@934K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6E0K9h3y4J5L8%4y4G2k6Y4c8Q4x3V1k6E0M7#2)9J5k6s2c8H3L8g2)9J5k6o6t1H3i4K6u0V1M7X3g2X3i4K6u0r3N6s2u0W2k6g2)9J5c8X3#2S2K9h3^5`.">TPM源码</a>

<a href="elink@451K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6K6N6r3g2X3j5h3&6T1k6i4u0Y4k6i4u0Q4x3V1k6D9K9h3u0@1M7r3#2K6i4K6u0r3j5$3!0E0L8h3W2@1i4K6u0r3x3o6c8T1x3X3b7^5k6e0W2S2k6X3x3H3j5e0W2T1y4X3u0X3k6X3f1#2y4U0u0S2x3U0y4W2y4e0S2U0x3r3c8W2x3e0p5#2x3K6u0V1x3b7`.`.">补丁</a>

<a href="elink@b18K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6U0j5Y4N6S2L8X3M7#2x3o6g2Q4x3V1k6e0k6h3y4#2M7X3g2C8k6i4u0F1k6h3I4u0g2f1#2p5k6h3u0#2k6H3`.`.">笔者工具</a>

## 参与贡献

作者来自ZheJiang Guoli Security Technology,邮箱cbwang505@hotmail.com

## 引用

> 这篇文章的目的是介绍Hyper-V平台TPM组件服务进程`vmsp.exe`相关的通用TPM漏洞`CVE-2025-2884`分析与复现.

目录

- [引用](https://bbs.kanxue.com/#引用)

- [简介](https://bbs.kanxue.com/#简介)

- [复现环境](https://bbs.kanxue.com/#复现环境)

- [嵌套虚拟化模式下用户模式隔离进程IUM调试工具](https://bbs.kanxue.com/#嵌套虚拟化模式下用户模式隔离进程ium调试工具)

- [嵌套虚拟化模式下用户模式隔离进程IUM调试工具使用方法](https://bbs.kanxue.com/#嵌套虚拟化模式下用户模式隔离进程ium调试工具使用方法)

- [笔者工具使用效果](https://bbs.kanxue.com/#笔者工具使用效果)

- [Hyper-V平台通用TPM漏洞CVE-2025-2884分析](https://bbs.kanxue.com/#hyper-v平台通用tpm漏洞cve-2025-2884分析)

- [漏洞复现栈回溯](https://bbs.kanxue.com/#漏洞复现栈回溯)

- [漏洞复现](https://bbs.kanxue.com/#漏洞复现)

- [相关引用](https://bbs.kanxue.com/#相关引用)

- [参与贡献](https://bbs.kanxue.com/#参与贡献)

## 简介

文章结合了笔者魔改的用户模式隔离进程调试工具和逆向代码及调试结果分析了通用TPM漏洞`CVE-2025-2884`的利用过程和漏洞成因.

## 复现环境

Windows 11 24h2 canary preview 启用嵌套虚拟化模式

## 嵌套虚拟化模式下用户模式隔离进程IUM调试工具

Hyper-V 的 虚拟信任级别（VTL）是Windows的Hyper-V 虚拟化技术中用于隔离不同安全域的安全机制.VTL1 是其中较高的特权级别,其特性如下

VTL1 的特权高于 VTL0（最低权限级别）,在虚拟安全模式（Virtual Secure Mode）中,VTL1 用于运行安全内核和隔离用户模式（IUM isolated user mode process）的代码，这些程序通过系统调用与 VTL0 中的内核交互.常见IUM进程有`LsaIso.exe`,`vmsp.exe`等.

开启虚拟安全模式可在 Windows 安全中心>设备安全>核心隔离详细信息>中找到内存完整性启用后重启计算机就能开启.

在VTL0 域中即使获取了内核级的范围权限 ‌,也无法对VTL1中内存进行操作.这种设计可抵御内核级攻击,保护用户密码哈希、BitLocker加密密钥等机密信息.VTL1 的安全内核模式由`securekernel.exe`承载用于调度用户IUM模式系统调用.

笔者使用最新版的[`LiveCloudKd`](https://bbs.kanxue.com/elink@9a4K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6Y4k6i4u0Z5j5i4u0@1x3o6q4Q4x3V1k6x3K9i4k6W2b7$3I4G2N6h3c8w2k6q4)9J5c8Y4c8J5k6h3g2Q4x3V1k6E0j5i4y4@1k6i4t1`.)工具找到了一种可以嵌套虚拟化模式下调试虚拟机内用户模式隔离进程方法,原理在于区别虚拟机分区中虚拟安全模式隔离了跨VTL内存的访问权限,

对于虚拟机的父分区仍然可以调用内核模式hypercall或`winhv.sys`驱动api直接操作虚拟机线性物理内存,`LiveCloudKd`工具为我提供了一个签名的驱动`hvmm.sys`通过应用层api就能实现这些操作,即使在虚拟机中这片保护的物理内存被划分给VTL1级别,仍然可以在父分区使用物理地址对这片虚拟机内的内存进行访问.如果要调试用户IUM模式进程就需要patch一下

`securekernel!SkpsIsProcessDebuggingEnabled`函数,在新版中这个函数被内联在`IumInvokeSecureService`系统调用实现中,简单描述下特征

|  |  |

| --- | --- |
