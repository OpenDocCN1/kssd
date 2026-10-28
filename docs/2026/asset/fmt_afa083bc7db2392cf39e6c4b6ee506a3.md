
```
CONST FLT_OPERATION_REGISTRATION Callbacks[] = {

    { IRP_MJ_CREATE,              //Request的功能号

      0,                          //标志位，置0表示仅对/读写回调有用

      MyMinifiterPreOperation,    //Pre-Operation Fun

      MyMinifiterPostOperation }, //Post-Operation Fun



    { IRP_MJ_CREATE_NAMED_PIPE,

      0,

      MyMinifiterPreOperation,

      MyMinifiterPostOperation },



    { IRP_MJ_CLOSE,

      0,

      MyMinifiterPreOperation,

      MyMinifiterPostOperation },



    { IRP_MJ_READ,

      0,

      MyMinifiterPreOperation,

      MyMinifiterPostOperation },

    { IRP_MJ_OPERATION_END }   //最后一个元素必须存在否则过滤管理器不知道有多少个回调参数

```

## 4 编写一个自己的Minifilter

查资料发现，Vs2015及以上版本安装WDK组件过后都可以选择创建Minifilter项目，VS会自动生成框架，但其中还有一些需要设置的地方。接下来会详细介绍：

### 4.1 inf文件

![](https://bbs.kanxue.com/upload/attach/202009/768770_WPKZZQ3E2PR7FVY.png)

其中`Class` `ClassGuid`是需要设置的，当然也可以用注释中的缺省值。

![](https://bbs.kanxue.com/upload/attach/202009/768770_H4CAD4H5M4FKJX2.png)

倒数第二行的`Altitude`值是微过滤器的层级码，具体内容可以参照上面的链接

这里可以设置为`370030`表示层中的位置

### 4.2 Callback设置

![](https://bbs.kanxue.com/upload/attach/202009/768770_BV6AKSNZ3MK4XSF.png)

Vs生成的框架默认是没有注册任何回调的，可以根据自己需要在`//TODO`注释后声明自己要过滤的操作和回调函数。也可以偷懒直接吧`if 0`修改成`if 1`直接过滤所有操作。当然此声明中Vs提供了默认的回调函数，也可以自行选择修改

### 4.3 回调函数

Vs已经为我们生成好了默认的回调函数

```
FLT_PREOP_CALLBACK_STATUS

FsFilter1PreOperation (          //FsFilter1是驱动名称

    _Inout_ PFLT_CALLBACK_DATA Data,

    _In_ PCFLT_RELATED_OBJECTS FltObjects,

    _Flt_CompletionContext_Outptr_ PVOID *CompletionContext

    )



FLT_POSTOP_CALLBACK_STATUS

FsFilter1PostOperation (

    _Inout_ PFLT_CALLBACK_DATA Data,

    _In_ PCFLT_RELATED_OBJECTS FltObjects,

    _In_opt_ PVOID CompletionContext,

    _In_ FLT_POST_OPERATION_FLAGS Flags

    )

```

至此，一个简单的Mnifilter驱动就已经能够成功生成并能够进行加载了（BTW，x64还得自行签名或者强制禁用驱动签名）

## 5 实现简单的过滤操作

### 5.1 过滤文件

废话不多说，直接上代码！

```
FLT_PREOP_CALLBACK_STATUS

MyMinifiterPreOperation (

    _Inout_ PFLT_CALLBACK_DATA Data,

    _In_ PCFLT_RELATED_OBJECTS FltObjects,

    _Flt_CompletionContext_Outptr_ PVOID *CompletionContext

    )

{

    NTSTATUS status;

    PFLT_FILE_NAME_INFORMATION nameInfo;



    UNREFERENCED_PARAMETER(FltObjects);

    UNREFERENCED_PARAMETER(CompletionContext);

    PAGED_CODE();

    __try {

        status = FltGetFileNameInformation(Data,

            FLT_FILE_NAME_NORMALIZED |

            FLT_FILE_NAME_QUERY_DEFAULT,

            &nameInfo);

        if (NT_SUCCESS(status)) {

            FltParseFileNameInformation(nameInfo);

            if (wcsstr(nameInfo->Name.Buffer, L"calc.exe") //过滤计算器

                Data->IoStatus.Status = STATUS_ACCESS_DENIED;//填写拒绝

                Data->IoStatus.Information = 0;

                FltReleaseFileNameInformation(nameInfo);

                return FLT_PREOP_COMPLETE;



            //release resource

            FltReleaseFileNameInformation(nameInfo);

        }

    }

    __except (EXCEPTION_EXECUTE_HANDLER) {

        DbgPrint("EXCEPTION_EXECUTE_HANDLER\n");

    }



    return FLT_PREOP_SUCCESS_WITH_CALLBACK;

}

```

这样`calc.exe`就不能读取、删除、覆盖、重命名和运行了，这是基于书中的代码实现的功能，也是一个简单的过滤操作的实现。

### 5.2 文件重定向

也是查阅了相关资料，自己实现了打开读写`1.txt`实际上会重定向另一目录下的`2.txt`

```
FLT_PREOP_CALLBACK_STATUS

MyMinifiterPreOperation (

    _Inout_ PFLT_CALLBACK_DATA Data,

    _In_ PCFLT_RELATED_OBJECTS FltObjects,

    _Flt_CompletionContext_Outptr_ PVOID *CompletionContext

    )

{

    NTSTATUS status;

    PFLT_FILE_NAME_INFORMATION nameInfo;



    UNREFERENCED_PARAMETER(FltObjects);

    UNREFERENCED_PARAMETER(CompletionContext);



    UNICODE_STRING desFilename；

    RtlCreateUnicodeString(&desFilename, L"\\DEVICE\\HARDDISKVOLUME3\\USERS\\User\\DESKTOP\\2.TXT");//为了方便测试路径我写死了，实际操作也很简单仅仅只是字符串的拼接

    PFILE_OBJECT FileObject;

    PAGED_CODE();

    __try {



        status = FltGetFileNameInformation(Data,

            FLT_FILE_NAME_NORMALIZED |

            FLT_FILE_NAME_QUERY_DEFAULT,

            &nameInfo);

        if (NT_SUCCESS(status)) {

            FltParseFileNameInformation(nameInfo);

            if (wcsstr(nameInfo->Name.Buffer, L"1.txt")) {

                FileObject = Data->Iopb->TargetFileObject;

                FileObject->FileName = desFilename;//替换成目标文件的路径。

                Data->IoStatus.Information = IO_REPARSE;

                Data->IoStatus.Status = STATUS_REPARSE;

                Data->Iopb->TargetFileObject->RelatedFileObject = NULL;

                FltSetCallbackDataDirty(Data)；

                FltReleaseFileNameInformation(nameInfo);

                return FLT_PREOP_COMPLETE;

            }

            //release resource

            FltReleaseFileNameInformation(nameInfo);

        }

    }

    __except (EXCEPTION_EXECUTE_HANDLER) {

        DbgPrint("EXCEPTION_EXECUTE_HANDLER\n");

    }

    return FLT_PREOP_SUCCESS_WITH_CALLBACK;

}

```

## 6 Some interesting thing

在上面两个功能的过滤中，我都采用了`if(wcsstr(nameInfo->Name.Buffer, L"1.txt")`的判断，我只判定子串中是否含有“1.txt”字符串，以前搞过web安全的我突然想起来当年IIS 6.0的解析漏洞，文件夹的命名是可以使用1.txt这样的名称的，如果开启过滤，那会对目录产生怎样的影响？对此我进行了实验。

实验的结果是，整个目录包括目录里的文件操作都被过滤掉了。

## 7 结语

至此，是我对Minifilter的初步学习，不足之处是对Minifilter与应用层的通信还未曾涉及，在今后的时间里会不断学习各种过滤操作，学习应用层通讯，打造一个完善的Minifilter。新人初次发帖，难免会有瑕疵，还请各位大佬多多指正。

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
