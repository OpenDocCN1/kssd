
# 简单实现防止 U 盘传染

> 原文：[`www.pediy.com/kssd/pediy10/99948.html`](https://www.pediy.com/kssd/pediy10/99948.html)

## 一 缘起
学校机房已经被弄得千疮百孔了，又没维护，U 盘一插进去，感染的隐藏的，导致无法正常使用了。

很烦，就想到自己弄个试下当做学习，网上这样的工具应该不少了，现在可真是防不胜防啊！先分享下自己的简单实现，呵呵，不足之处还请大牛多指导下才好啊！

## 二 实现原理简介
一般 ring3 下的感染是通过 `GetLogicalDrives` 然后在 `GetDriveType` 来判断是否存在移动设备的，所以就跟了下这两 API，

发现在我的 xp sp3 下实现的有点出乎意料，大致描述如下：

```
DWORD GetLogicalDrives(VOID)
{
  NtQueryInformationProcess((-1,0x17,ProcessInformation,36,NULL);//ProcessInformationClass=0x17 这个在 MSDN 中我没找到说明。
  return *(LPDWORD)((DWORD)ProcessInformation);
}

DWORD GetDriveTypeW(LPCTSTR lpRootPathName)
{
  BYTE i;
  ;对 lpRootPathName 的一些字符编码操作
  NtQueryInformationProcess((-1,0x17,ProcessInformation,36,NULL);
  i=*(LPBYTE)lpRootPathName-0x41;
  i=*(LPBYTE)((DWORD)ProcessInformation+4+i);
  switch(i)
  {
    case 2: //移动设备
    return 2;
    case 3: //硬盘
    return 3;
    case 5: //CD_ROM
    return 5;
    default:
    ;不是以上三种再根据 lpRootPathName 进一步的判断,后面的可以不管了，上面的就够了。
  }
}
```

既然这样就变简单了好多，我们只要 HOOK 这个 `NtQueryInformationProcess`，就可以实现目的了！

U 盘检测示例代码：

```
BOOL Detect_u(void)
{
  char DiskPath[5]="C:\\";
  DWORD DrivesBitmap;
  BYTE i;
  DrivesBitmap=GetLogicalDrives();
  for (i=0;i<24;i++)
  {
    DiskPath[0]='C'+i;
    if ((DrivesBitmap & 1)==1)
    {
      if (GetDriveType(DiskPath)==2)
        return TRUE;
    }
    DrivesBitmap>>=1;
  }
  return FALSE;
}
```

## 三 代码实现
这里 HOOK `NtQueryInformationProcess` 的方法用了比较简单的 SSDT hook,主要代码片段如下：

```
NTSTATUS __stdcall MyNtQueryInformationProcess(__in          HANDLE ProcessHandle,
                                                 __in          PROCESSINFOCLASS ProcessInformationClass,
                                                 __out         PVOID ProcessInformation,
                                                 __in          DWORD ProcessInformationLength,
                                                 __out_opt     LPDWORD ReturnLength
                                                 )
{
  NTSTATUS status;
  DWORD     i;
  status=RealNtQueryInformationProcess(ProcessHandle,ProcessInformationClass,ProcessInformation,ProcessInformationLength,ReturnLength);
  if (ProcessInformationClass==0x17)
  {
    KdPrint(("OriValue:0x%08X",*(LPDWORD)((DWORD)ProcessInformation+8)));
    *(LPDWORD)ProcessInformation<<=1; //GetLogicalDrives 函数返回值 drives bitmap <<1 错乱对应关系，相当于 HOOk GetLogicalDrives
    (DWORD)ProcessInformation+=4;
    for (i=0;i<26;i++)
    {
      if (*(LPBYTE)ProcessInformation==2)
      {
        KdPrint(("Address:0x%08X",ProcessInformation));
      //  KdPrint(("Have Ramovebel Device!"));
        *(LPBYTE)ProcessInformation=3; //移动设备改成硬盘，相当于 HOOk GetDriveType 两个都齐了！
      }
      (DWORD)ProcessInformation+=1;
    }
  KdPrint(("AffValue:0x%08X",*(LPDWORD)((DWORD)ProcessInformation-22)));
  }
  return status;
}
```

## 四 总结
这样虽然可以防住一些普通的感染，但是对比较 BT 的和 ring0 下的还是不行的，如果你在自己电脑上试驱动部分，

`(Addr=(DWORD)KeServiceDescriptorTable->ServiceTableBase+0x9A*4)` 这里面的 `NtQueryInformationProcess` 在 SSSDT 中的偏移 `0x9A` 需要改下，

还有其他平台下 `GetLogicalDrives` 和 `GetDriveTyped` 的实现可能就不是这样了，我的是 xp sp3！

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || `U_Try.rar` | |
|---|---|---|
|
