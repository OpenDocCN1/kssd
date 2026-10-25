
# 如何从一个文件的 `FILE_OBJECT` 得到该文件的完整路径？

> 原文：[`www.pediy.com/kssd/pediy10/60777.html`](https://www.pediy.com/kssd/pediy10/60777.html)

从 `FILE_OBJECT` 结构体中的 `Filename` 属性可以得到该文件的文件名，例如：`abc.exe`

另外从 `FILE_OBJECT` 结构体中的 `RelatedFileObject` 结构体下的 `Filename` 还可以进一步得到该文件的相对路径，例如：`\\AAAA\\BBBB\\CCCC`

这样综合起来，我只能得到该文件没有盘符的路径：`\\AAAA\\BBBB\\CCCC\\abc.exe`

因此我的问题是，从 `FILE_OBJECT` 结构体中如何得到该文件所在的盘符信息？谢谢！

多谢！

通过 `ObQueryNameString(file, fileNameInformation, 1024, &retSize);`

最终得到了该文件的物理路径,例如：`\\Device\\HarddiskVolume1\\AAAA\\BBBB\\CCCC\\abc.exe`

基本上这就差不多了，`\\Device\\HarddiskVolume1` 就是 `c:`

另外，`ObQueryNameString` 函数需要在驱动中引入一个头文件：`#include "ntifs.h"`

这里顺便上传上来！如何和 `ntddk.h` 中的定义重复了，直接把 `ntifs.h` 中的定义注释掉就好了！

`ntifs.rar`

多谢 5 楼！同时这两天看了看 sysinternal 的 `FileMon` 和 `RegMon`，颇有收获！

本问题，从 `FILE_OBJECT` 到该文件的完整路径的方法，我个人总结如下：

代码:

```
//相关变量
HANDLE filehandle=0;
PFILE_OBJECT file=0;
PFILE_OBJECT relatedfile=0;
POBJECT_HANDLE_INFORMATION info=0; 
char filefullname_c[1024];
ANSI_STRING filefullname_a; 
UNICODE_STRING filefullname_u;
int relatedfilelength;
POBJECT_NAME_INFORMATION     fileNameInformation;
ULONG retSize;

//先得到盘符
RtlVolumeDeviceToDosName(file->DeviceObject,&filefullname_u);
RtlUnicodeStringToAnsiString(&filefullname_a,&filefullname_u,1);
strncpy(filefullname_c,filefullname_a.Buffer,filefullname_a.Length); 
filefullname_c[filefullname_a.Length]='\0';
relatedfilelength=filefullname_a.Length;

//相对路径
relatedfile=file->RelatedFileObject;
RtlUnicodeStringToAnsiString(&filefullname_a,&relatedfile->FileName,1);
strncat(filefullname_c,filefullname_a.Buffer,filefullname_a.Length); 
filefullname_c[relatedfilelength+filefullname_a.Length]='\0';
relatedfilelength+=filefullname_a.Length;

//文件名
RtlUnicodeStringToAnsiString(&filefullname_a,&file->FileName,1);
strcat(filefullname_c,"\\");
strncat(filefullname_c,filefullname_a.Buffer,filefullname_a.Length);
filefullname_c[relatedfilelength+filefullname_a.Length+1]='\0';
```

最终的文件路径就放在：`filefullname_c`
