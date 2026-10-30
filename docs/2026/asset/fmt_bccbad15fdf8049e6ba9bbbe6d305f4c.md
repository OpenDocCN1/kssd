
```c
if ( a3 )
    *a3 = *(_DWORD *)(pool_ptr1 + 8);
if ( a4 )
    *a4 = *(_QWORD *)(pool_ptr1 + 0x10);
if ( a5 )
    *a5 = *(_DWORD *)(pool_ptr1 + 0x18);
if ( a6 )
    *a6 = *(_QWORD *)(pool_ptr1 + 0x20);
if ( a7 )
    *a7 = *(_QWORD *)(pool_ptr1 + 0x28);
if ( a8 )
    *a8 = *(_QWORD *)(pool_ptr1 + 0x30);
if ( a9 )
    *a9 = *(_DWORD *)(pool_ptr1 + 0x38);
if ( a10 )
    *a10 = *(_QWORD *)(pool_ptr1 + 0x40);
if ( a11 )
    *a11 = *(_QWORD *)(pool_ptr1 + 0x48);
if ( a12 )
    *a12 = *(_DWORD *)(pool_ptr1 + 0x50);
if ( a13 )
    *a13 = *(_QWORD *)(pool_ptr1 + 0x58);
if ( a14 )
    *a14 = *(_QWORD *)(pool_ptr1 + 0x60);
return 0i64;
```

返回到`cng!ConfigurationFunctionIoHandler`函数后，由于输入缓冲区的第二个四字节0x10400被截断为0x400，就会执行到这样的分支

```c
if ( a1 == 0x400 )
    return BCryptSetContextFunctionProperty(
        dwTable, // *a3 = *(_DWORD *)(pool_ptr1 + 8)
        pszContext, // *a4 = *(_QWORD *)(pool_ptr1 + 0x10)
        dwInterface, // *a5 = *(_DWORD *)(pool_ptr1 + 0x18)
        pszFunction, // *a6 = *(_QWORD *)(pool_ptr1 + 0x20)
        pszProperty, // *a8 = *(_QWORD *)(pool_ptr1 + 0x30)
        cbValue, // *a12 = *(_DWORD *)(pool_ptr1 + 0x50)
        pbValue); // *a13 = *(_QWORD *)(pool_ptr1 + 0x58)
```

## `cng!BCryptSetContextFunctionProperty`

跟进`cng!BCryptSetContextFunctionProperty`函数，继续分析此函数，`cng!ValidateTableId`所在的分支无法满足条件，利用`cbValue`和`pbValue`的值初始化`DestinationString`，利用此`DestinationString`调用`cng!CfgReg_Acquire`

```c
NTSTATUS __stdcall BCryptSetContextFunctionProperty(
    ULONG dwTable, // pool_ptr1_off_8
    LPCWSTR pszContext, // pool_ptr1_off_10
    ULONG dwInterface, // pool_ptr1_off_18
    LPCWSTR pszFunction, // pool_ptr1_off_20
    LPCWSTR pszProperty, // pool_ptr1_off_30
    ULONG cbValue, // pool_ptr1_off_50
    PUCHAR pbValue) // pool_ptr1_off_58
{
    ...
    *(_QWORD *)&DestinationString.Length = 0i64;
    v36 = 0;
    v34 = 0;
    v35 = 0;
    v37 = 0i64;
    v38 = 0i64;
    v39 = 0i64;
    DestinationString.Buffer = 0i64;
    v42 = 0i64;
    if ( !ValidateTableId(dwTable) // dwTable = 1, false
        || !(unsigned int)ValidateInterfaceId(dwInterface, 0) // dwInterface=3, false
        || !v13 // pszFunction, false
        || *v13 == (_WORD)v12 // *pszFunction != 0, false
        || !pszProperty // pszProperty, false
        || *pszProperty == (_WORD)v12 ) // *pszProperty != 0, false
    {
        v16 = 0x57;
        LABEL_36:
        if ( !v10 )
            goto LABEL_38;
        goto LABEL_37;
    }
    if ( cbValue && pbValue ) // true
    {
        *(_QWORD *)&DestinationString.Length = pbValue;
        LODWORD(DestinationString.Buffer) = cbValue;
        HIDWORD(DestinationString.Buffer) = cbValue;
        LODWORD(v42) = v12;
    }
    v14 = CfgReg_Acquire(v11, dwTable, 0x3001Fi64);
    ...
}
```

## `cng!CfgReg_Acquire`

进一步跟进`cng!CfgReg_Acquire`，发现`cng!VerifyRegistryAccess`函数尝试对`System\CurrentControlSet\Control\Cryptography\Configuration\Local`注册表项执行操作，`cng!VerifyRegistryAccess`内部调用`cng!KeRegOpenKey`尝试打开注册表表项

![CVE-2020-17087-9.png](https://bbs.kanxue.com/upload/attach/202203/924781_RUQ53MX822CCSGJ.jpg)

根据`cng!VerifyRegistryAccess`的返回值为5得知打开失败，函数将5返回给`cng!BCryptSetContextFunctionProperty`

![CVE-2020-17087-10.png](https://bbs.kanxue.com/upload/attach/202203/924781_5RNNSBG9FHUZGDN.jpg)

紧接着在`cng!BCryptSetContextFunctionProperty`的执行流程大体如下，分别使用`pool1_ptr+0x100`、`pool1_ptr+0x200`、`pool1_ptr+0x400`处的字符初始化三个新的`UnicodeString`

```c
...
    v14 = CfgReg_Acquire(v11, dwTable, 0x3001Fi64); // v14 = 5
    v16 = v14;
    if ( v14 )
    {
        v17 = v14 == 5; // v17 = true
        goto LABEL_43;
    }

    ...
    LABEL_43:
    if ( !v17 )
        goto LABEL_49;
    }

    *(_QWORD *)&DestinationString.Length = 0i64;
    DestinationString.Buffer = 0i64;
    *(_QWORD *)&v40.Length = 0i64;
    v40.Buffer = 0i64;
    *(_QWORD *)&v43.Length = 0i64;
    v43.Buffer = 0i64;

    RtlInitUnicodeString(&DestinationString, pszContext); // pool_ptr1_off_10
    RtlInitUnicodeString(&v40, pszFunction); // pool_ptr1_off_20
    RtlInitUnicodeString(&v43, pszProperty); // pool_ptr1_off_30
```

## `cng!CfgAdtReportFunctionPropertyModification`

在初始化三个`UnicodeString`后，进入下面的分支调用`cng!CfgAdtReportFunctionPropertyModification`函数

![CVE-2020-17087-11.png](https://bbs.kanxue.com/upload/attach/202203/924781_FZK9D6GY2D5E9QC.jpg)

在调试器中查看一下函数的参数，最少有10个，IDA反编译出来的结果中有11个

![CVE-2020-17087-12.png](https://bbs.kanxue.com/upload/attach/202203/924781_K82QBQ75GW73KXE.jpg)

从上图中可以看到，在`rsp+40`的位置存储了0x2aab，这个数值来自于`pool_ptr1 + 0x50`的位置，查看一下寄存器`r14w`的数据来源，可以看到函数序言部分将`pool_ptr1 + 0x50`处的值赋给`r14d`

![CVE-2020-17087-13.png](https://bbs.kanxue.com/upload/attach/202203/924781_82DAWZ2P2TH52PS.jpg)

`cng!CfgAdtReportFunctionPropertyModification`函数内部调用最终的漏洞函数`cng!CfgAdtpFormatPropertyBlock`，调试器中查看一下`cng!CfgAdtpFormatPropertyBlock`的参数，可以看到第二个参数来自于`cng!CfgAdtReportFunctionPropertyModification`的第九个参数，数值为0x2aab

![CVE-2020-17087-14.png](https://bbs.kanxue.com/upload/attach/202203/924781_4R72MFY6NV6E6M6.jpg)

## `cng!CfgAdtpFormatPropertyBlock`

由于漏洞原理在开篇提及，此处就对一些关键位置查看下。查看一下`cng!BCryptAlloc`的参数，果然溢出为2

![CVE-2020-17087-15.png](https://bbs.kanxue.com/upload/attach/202203/924781_HEPZUBUN7NKAPMN.jpg)

调用完该函数申请到的池块大小为0x20字节

![CVE-2020-17087-16.png](https://bbs.kanxue.com/upload/attach/202203/924781_RGJB6CTS94DDBG2.jpg)

然而循环的次数为0x2aab次，每次向池块中写入6字节，最终就会覆写到相邻的内存，最终导致BSOD
