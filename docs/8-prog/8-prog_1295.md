
# 问个 `ZwWriteFile` 函数失败的问题

> 原文：[`www.pediy.com/kssd/pediy11/114194.html`](https://www.pediy.com/kssd/pediy11/114194.html)

`ZwWriteFile` 函数为啥子会失败呢?

先上代码:

```c
//这是打开逻辑磁盘哈数

HANDLE OpenLogicalDisk(CHAR lpszDiskName, ACCESS_MASK  DesiredAccess)

{

    NTSTATUS ntStatus;

    IO_STATUS_BLOCK iostatus;

    OBJECT_ATTRIBUTES objectAttributes;

    UNICODE_STRING DiskUnicodeString;

    HANDLE hfile;

    ANSI_STRING DiskNameString;

//    _asm int 3

    switch(lpszDiskName)

    {

    case 'C':

    //    RtlInitUnicodeString(&DiskUnicodeString, L"\\\\??\\\\PHYSICALDRIVE0");

        RtlInitUnicodeString(&DiskUnicodeString, L"\\\\??\\\\C:");

        break;

    case 'D':

        RtlInitUnicodeString(&DiskUnicodeString, L"\\\\??\\\\D:");

        break;

    case 'E':

        RtlInitUnicodeString(&DiskUnicodeString, L"\\\\??\\\\E:");

        break;

    case 'F':

        RtlInitUnicodeString(&DiskUnicodeString, L"\\\\??\\\\F:");

        break;

    case 'G':

        RtlInitUnicodeString(&DiskUnicodeString, L"\\\\??\\\\G:");

        break;

    case 'H':

        RtlInitUnicodeString(&DiskUnicodeString, L"\\\\??\\\\H:");

        break;

    case 'I':

        RtlInitUnicodeString(&DiskUnicodeString, L"\\\\??\\\\I:");

        break;

    }

    //初始化 objectAttributes

    InitializeObjectAttributes(&objectAttributes,

                             &DiskUnicodeString,

                             OBJ_CASE_INSENSITIVE,//对大小写敏感

                             NULL,

                             NULL);

    //打开文件

//    _asm int 3

    ntStatus = ZwCreateFile(&hfile,

                          DesiredAccess,

                           &objectAttributes,

                           &iostatus,

                           NULL,

                           FILE_ATTRIBUTE_NORMAL,

                           FILE_SHARE_READ|FILE_SHARE_WRITE,

                           FILE_OPEN,  //打开文件,如果不存在,则返回错误

                           FILE_SYNCHRONOUS_IO_NONALERT,

                           NULL,

                           0);

    if (NT_SUCCESS(ntStatus))

    {

        KdPrint(("Open file successfully.\n"));

    //    ZwClose(hfile);

       return hfile;

    }

    else

    {

        KdPrint(("Open file failed.\n"));

       return NULL;

    }

    return NULL;

}

//这是写入逻辑磁盘函数

BOOL WriteLogicalDiskSector(HANDLE hfile, PIoWriteDiskInBuffer pWriteSectorIn,\

                         PIoWriteDiskOutBuffer pWriteSectorOut)

{

    NTSTATUS ntStatus;

    IO_STATUS_BLOCK iostatus;

    //_asm int 3

    //读取文件

    ntStatus =    ZwWriteFile(hfile,

                      NULL,

                       NULL,

                       NULL,

                       &iostatus,

                       (PVOID)pWriteSectorIn->szWriteDiskBuffer,

                       (ULONG)pWriteSectorIn->readnum*512,

                       (PLARGE_INTEGER)&pWriteSectorIn->secStart,

                       NULL

                       );

// _asm int 3

     KdPrint(("Write offset %d  \n", pWriteSectorIn->secStart));

    KdPrint(("Btyes should be Writed  %d  \n", pWriteSectorIn->readnum*512));

if (NT_SUCCESS(ntStatus))  //写入数据成功

{

     pWriteSectorOut->Retn = TRUE;

     KdPrint(("The program really Write %d bytes \n", iostatus.Information));

     return TRUE;

}

else

{

     pWriteSectorOut->Retn = FALSE;

     KdPrint(("Write Data failed\n"));

     return FALSE;

}

}

//------------------------------------------------------------
```

下面是数据结构:

```c
//定义 IOCTL_WRITESECTOR

typedef struct IoWriteDiskInBuffer

{

    char szDiskName;    //读取的磁盘名

    BYTE readnum;        //读取扇区数

    DWORD secStart;       //扇区起始位置

    DWORD secEnd;         //扇区结束位置

    unsigned char szWriteDiskBuffer[4*512];

}IoWriteDiskInBuffer, *PIoWriteDiskInBuffer;

typedef struct IoWriteSectorOutBuffer

{

    BOOL Rtn;         //写入失败或成功

}IoWriteDiskOutBuffer, *PIoWriteDiskOutBuffer;

//-----------------------------------------------------------
```

最后是结果: 当 `secStart` = 0; `secEnd` = 0;的时候能够成功改写数据,

但是当 `secStart` = 0; `secEnd` = 512;就写入失败了, `secStart` = 512; `secEnd` = 512;还是写入失败,然后我就想不明白了![](img/9b2aec7f3d4bb041b2d419a59832192f.jpg "恐怖!"),哪位大牛来指教下吧... 不胜感激

上传的附件

| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) || 应用程序文件.rar

|

| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) || 驱动文件.rar

|
