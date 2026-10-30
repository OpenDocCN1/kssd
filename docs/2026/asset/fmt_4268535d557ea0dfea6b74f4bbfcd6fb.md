
`int` `SkpsIsProcessDebuggingEnabled()` `{` `int` `editto1`=`0`; `int` `Policy` = `SkpspFindPolicy((__int64)v560,` `2`,`7`, (__int64)&BugCheckParameter2_4, (__int64)v554);` `if` `( Policy <` `0` `|| BugCheckParameter2_4 >=` `4` `)` `{` `DbgPrint(` `"%hs: SkpspFindPolicy returned 0x%x and DebugEnable is %d.\n"`,` `"SkpsIsProcessDebuggingEnabled"`,` `Policy,` `BugCheckParameter2_4);` `}` `else` `if` `( BugCheckParameter2_4 )` `{` `if` `( BugCheckParameter2_4`== `1` `)` `{` `editto1` = `1`; `}` `else` `if` `( BugCheckParameter2_4`== `2` `)` `{` `editto1` = `(unsigned` `int`)`SkIsSecureBootEnabled()` == `0x80430006`; `}` `else` `{` `editto1` = `BYTE1(*v560) &` `1`; `}` `}` `if` `( editto1 )` `{` `//editto1改成1开启调试` `SkpsEnableDebugging((unsigned __int64)v560, a1[16]);` `}` |

`securekernel`默认的调试策略被保存在其镜像策略配置中,直接修改配置或patch文件二进制会导致`securekernel`签名验证失败,重启虚拟机蓝屏,所以这种方式是不可行.但是对于运行时的代码只需要匹配包含特征的代码patch掉`SkpsEnableDebugging`条件始终为`true`就可以.

在这里我们需要得到3个关键的数据:

1. 搜索到这片代码所在的物理页面地址的映射
2. 找到一个立即触发的`securekernel`函数地址`HvGetVpRegisters`通过这个`hypercall`得到虚拟地址`RIP`和`cr3`
3. 找到`securekernel`进程虚拟机地址和物理页面地址的映射

一个最常见的可以稳定触发的`securekernel`函数笔者找到的是`IumInvokeSecureService`这个函数,通过`SharpDisasm`这个工具反编译目标得到函数返回指令`ret`地址,然后不断搜索匹配的个函数所在页面的二进制数据,直到匹配到一个合适的物理页面,然后把指令`ret`改成如下无限循环汇编代码

|  |  |

| --- | --- |

| 1 2 3 4 | `spin_loop:` `pause ; <- 关键指令` `jmp spin_loop` `ret` |

修改内存后并不会引起虚拟机蓝屏,查询虚拟机所有`vcpu`的`RIP`通过`HvGetVpRegisters`这个`hypercall`,如果`rip`的低位与这个指令的低`16`位匹配,这样我们得到了一个符合条件的虚拟地址`gva`和匹配的物理地址`gpa`,就可以计算出页表页目录的基址`cr3`(也是通过`hypercall`)等.也同样通过`SharpDisasm`汇编指令匹配`SkpsIsProcessDebuggingEnabled`字符串,

得到要`patch`代码的地址根据`securekernel`计算得出的基址,加上偏移量转为物理地址后`patch`目标汇编代码,这样我们就成功绕过并启用调试用户`IUM`模式进程.

## 嵌套虚拟化模式下用户模式隔离进程IUM调试工具使用方法

1. 下载最新版的[LiveCloudKd](https://bbs.kanxue.com/elink@62fK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6Y4k6i4u0Z5j5i4u0@1x3o6q4Q4x3V1k6x3K9i4k6W2b7$3I4G2N6h3c8w2k6q4)9J5c8Y4u0W2L8r3g2S2M7$3g2K6i4K6u0r3N6r3q4Y4i4K6u0r3N6U0y4Q4x3X3f1H3i4K6u0W2x3q4)9J5k6e0t1H3x3U0f1H3z5o6t1@1)
2. 复制所有文件到:`"C:\Program Files\Windows Kits\10\Debuggers\x64"`
3. 修改`"C:\Program Files\Windows Kits\10\Debuggers\x64\cfg\RegParam.reg"` 其中`WinDbgPath`改成`"C:\Program Files\Windows Kits\10\Debuggers\x64\"`
4. 安装[vc运行库x64版](https://bbs.kanxue.com/elink@766K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6S2K9$3q4Q4x3X3g2E0M7#2)9J5c8Y4k6K6i4K6u0r3x3e0N6Q4x3V1k6J5k6h3I4W2j5i4y4W2i4K6u0r3N6X3y4Q4y4h3k6J5k6h3c8A6M7%4c8Q4x3X3g2^5y4U0c8Q4x3X3g2W2P5r3f1`.)
5. `regsvr32 "C:\Program Files\Windows Kits\10\Debuggers\x64\ExdiHvSrv.dll"`
6. `"C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe" -exec bypass -Command "Set-VMProcessor -VMName YourVMName -ExposeVirtualizationExtensions $true"`其中`YourVMName`是你的虚拟机名字,启用嵌套虚拟化模式
7. 复制虚拟机内`C:\Windows\System32\securekernel.exe`文件到:`"C:\Program Files\Windows Kits\10\Debuggers\x64"`和笔者工具同目录
8. 虚拟机运行后运行笔者工具,看到打印出`securekernel.exe`基址即为成功

## 笔者工具使用效果

笔者工具使采用`.net`程序实现,笔者工具无需禁用安全启动或者修改`bcdedit`配置,下图是笔者在的`Win11 24h2`物理机上嵌套虚拟化模式下调试用户模式隔离进程`vmsp.exe`效果

![](https://bbs.kanxue.com/upload/attach/202510/609565_KS6T4KMKHK5GPC2.webp)

## Hyper-V平台通用TPM漏洞CVE-2025-2884分析

`TPM`模块实现`2.0`(Trusted Platform Module）是可信平台模块的第二代版本,是一种安全密码处理器的国际标准,`TPM`模块实现可以是`cpu`上的加密芯片固件程序或者`Hyper-V`等虚拟机平台`TPM`组件模拟服务进程如`vmsp.exe`.笔者`poc`采用`uefi`程序实现,初始化`TPM`客户端通过与`Hyper-V`服务的共享物理页面 `PcdTpmBaseAddress = 0xFED40000`获取适配的`TPM`版本模式,

使用字段的结构地址进行所有的`TPM`通信数据包交互,有固定的输入输出缓冲区大小设置和操作返回等读写功能.

通用`TPM`漏洞`CVE-2025-2884`补丁发布与今年6月所影响包含所有共享`tpm`组件代码的固件及服务端应用程序,基于笔者的`IUM`模式进程调试工具我们得到了一种可以调试`vmsp.exe`进程的方法,下面我我们就通过代码结合调试分析下这个漏洞.

该漏洞存在于`CryptHmacSign`函数中,补丁前后的代码分析如下:

|  |  |

| --- | --- |

| 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31 32 33 | `static UINT64 PcdTpmBaseAddress` = `0xFED40000`; `//入口TPM2_Sign` `TPM_RC` `CryptSign(OBJECT*` `signKey,` `/` `IN: signing key` `TPMT_SIG_SCHEME*` `signScheme,` `/` `IN: sign scheme.` `TPM2B_DIGEST*` `digest,` `/` `IN: The digest being signed` `TPMT_SIGNATURE*` `signature` `/` `OUT: signature` `)` `{` `signature->sigAlg` = `signScheme->scheme;` `signature->signature.any.hashAlg` = `signScheme->details.any.hashAlg;` `if(signKey->publicArea.type==TPM_ALG_KEYEDHASH)` `{` `static TPM_RC CryptHmacSign(TPMT_SIGNATURE*` `signature,` `/` `OUT: signature` `OBJECT*` `signKey,` `/` `IN: HMAC key sign the` `hash` `TPM2B_DIGEST*` `hashData` `/` `IN:` `hash` `to be signed` `)` `{` `HMAC_STATE hmacState;` `UINT32 digestSize;` `//这行是补丁代码` `if (signature->sigAlg ==` `TPM_ALG_HMAC)` `{` `digestSize` = `CryptHmacStart2B(&hmacState,` `signature->signature.any.hashAlg,` `&signKey->sensitive.sensitive.bits.b);` `CryptDigestUpdate2B(&hmacState.hashState, &hashData->b);` `CryptHmacEnd(&hmacState, digestSize, (BYTE*)&signature->signature.hmac.digest);` `return` `TPM_RC_SUCCESS;` `}` `return` `TPM_RC_SCHEME;` `}` `}` |

到达目标的漏洞代码需要调用`Tpm2_CreatePrimary`函数创建一个`signKey`,具体方法需要预定义一个`key`参数模板`publicArea`,

成功后在返回结果可以得到它的句柄,根据`tpm`文档构造参数传入调用`TPM2_Sign`,预先验证参数的`signScheme`和`signKey`的相关字段匹配成功后,

该函数对于传入的`signKey->publicArea.type`如果`hash`类型为`TPM_ALG_KEYEDHASH`会调用`CryptHmacSign`进行对传入的数据进行`hash`也就是签名,

根据其中的`signature->signature.any.hashAlg`算法可以是`TPM_ALG_SHA1`到`TPM_ALG_SHA512`等,`hash`数据的签名结果大小在`0x10-0x30`之间,而且可以通过传入的`hashData`控制,

这个函数默认是给`HMAC`算法使用,但实际类型可以由用户控制,导致签名的结果数据反序列化时存在对`signature->sigAlg`类型的混肴,进行另外一种类型对其反序列化,可能允许攻击者读取该签名的结果缓冲区末尾后最多65535个字节.

|  |  |

| --- | --- |

| 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31 32 33 34 35 36 37 38 39 40 41 42 43 44 45 46 47 48 49 50 51 52 53 54 55 56 57 58 59 60 | `UINT16` `BYTE_Array_Marshal(BYTE*` `source, BYTE**` `buffer``, INT32*` `size, INT32 count)` `{` `if(buffer != 0)` `{` `if((size == 0) || ((*size -= count) >= 0))` `{` `memcpy(*buffer, source, count);` `*buffer += count;` `}` `pAssert((size == 0) || (*size >= 0));` `}` `if(count >= INT16_MAX){` `{` `//poc位置` `DebugBreak();` `}}` `return` `((UINT16)count);` `}` `UINT16` `TPM2B_ECC_PARAMETER_Marshal(TPM2B_ECC_PARAMETER*` `source, BYTE**` `buffer``, INT32*` `size)` `{` `UINT16 result` = `(UINT16)(result` + `UINT16_Marshal((UINT16*)&(source->t.size),` `buffer``, size));` `if(source->t.size == 0)` `return` `result;` `return` `(UINT16)(result` + `BYTE_Array_Marshal((BYTE*)&(source->t.buffer),` `buffer`,` `size,` `(INT32)source->t.size));` `}` `UINT16` `TPMS_SIGNATURE_ECC_Marshal(TPMS_SIGNATURE_ECC*` `source, BYTE**` `buffer``, INT32*` `size)` `{` `UINT16 result` = `(UINT16)(result` + `TPMI_ALG_HASH_Marshal(` `(TPMI_ALG_HASH*)&(source->hash),` `buffer``, size));` `return` `=` `(UINT16)(result` + `TPM2B_ECC_PARAMETER_Marshal(` `(TPM2B_ECC_PARAMETER*)&(source->signatureR),` `buffer``, size));` `...` `}` `UINT16` `TPMU_SIGNATURE_Marshal(` `TPMU_SIGNATURE*` `source, BYTE**` `buffer``, INT32*` `size, UINT32 selector)` `{s` `switch(selector=signature->sigAlg)` `{` `//HMAC算法使用类型` `case TPM_ALG_HMAC:` `return` `TPMT_HA_Marshal((TPMT_HA*)&(source->hmac),` `buffer``, size);` `...` `//对signature->sigAlg类型的混肴` `case TPM_ALG_EDDSA:...` `return` `TPMS_SIGNATURE_ECC_Marshal(` `(TPMS_SIGNATURE_EDDSA*)&(source->eddsa),` `buffer``, size);` `case ..` `}` |
