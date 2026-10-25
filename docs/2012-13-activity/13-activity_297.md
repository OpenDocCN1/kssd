
# 我贴一个第三阶段和木马驱动无关的 ring3 方法

> 原文：[`www.pediy.com/kssd/pediy10/75907.html`](https://www.pediy.com/kssd/pediy10/75907.html)

大家爆，我也爆吧。。。

这个是我第三阶段的第一个 ring3 方法，没分析驱动就直接用了，试了很好用，不过需要重启

重启前：
找到木马文件对应硬盘上的扇区，直接写扇区破坏木马文件

重启后：
批处理删掉木马尸体和注册表垃圾

重启前的 `DeleteFileBySection` 代码见附件

`delfile.cpp`

代码:
```
#include <windows.h>
#include <stdio.h>
#include "DeleteFileBySection.h"

char szSysdir[MAX_PATH];
char szfile[MAX_PATH];

int main()
{
  CDeleteFileBySection delfile;
  GetSystemDirectory(szSysdir,MAX_PATH);
  strcpy(szfile,szSysdir);
  strcat(szfile,"\\\\drivers\\\\HBKernel32.sys");
  if(delfile.DeleteFile(szfile))
    printf("delfile Success! : %s\\n\\n",szfile);
  else
    printf("delfile Failed! : %s\\n\\n",szfile);

  strcpy(szfile,szSysdir);
  strcat(szfile,"\\\\System.exe");
  if(delfile.DeleteFile(szfile))
    printf("delfile Success! : %s\\n\\n",szfile);
  else
    printf("delfile Failed! : %s\\n\\n",szfile);

  strcpy(szfile,szSysdir);
  strcat(szfile,"\\\\HBQQXX.dll");
  if(delfile.DeleteFile(szfile))
    printf("delfile Success! : %s\\n\\n",szfile);
  else
    printf("delfile Failed! : %s\\n\\n",szfile);

  printf("Done! Reboot system!\\n");
  getchar();
  return 0;
}
```

重启后的批处理：

`clean up.bat`

代码:
```
reg delete HKEY_LOCAL_MACHINE\\SYSTEM\\CurrentControlSet\\Services\\HBKernel32 /f
reg delete HKEY_LOCAL_MACHINE\\SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\Run /v HBService32 /f
reg add "HKEY_LOCAL_MACHINE\\SOFTWARE\\Microsoft\\Windows NT\\CurrentVersion\\Windows" /v AppInit_DLLs /t REG_SZ /d "" /f
del /f %SYSTEMROOT%\\system32\\drivers\\HBKernel32.sys
del /f %SYSTEMROOT%\\system32\\system.exe
del /f %SYSTEMROOT%\\system32\\HBQQXX.dll
```

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || DeleteFileBySection.rar |
