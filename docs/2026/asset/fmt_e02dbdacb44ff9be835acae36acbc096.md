
# [原创]蒙泰 Debug Api补丁源码

看雪早就有蒙泰5.0电子版全版，

该文章介绍于179K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4m8W2k6r3W2&6i4K6u0W2j5$3!0E0i4K6u0r3j5X3u0K6K9s2c8E0L8q4)9J5c8V1u0n7f1K6c8Q4x3V1k6C8j5h3&6^5N6h3f1I4x3o6W2Q4x3X3g2Z5N6r3@1`.

小弟不才，发现一个重要问题，该版软件有一个很大的暗桩，就是这个软件居然还是电子版纹设计软件，属于防伪设计的。市面价格28万。去买的话，他们销售会反馈10万的回扣（这就是国产软件的猫腻）。主要用于小型印刷中心和有防伪印刷需求的中小公司。

它是插不同的狗 表现为不同的软件版本。

其中版纹是藏的最深的一个暗桩。

其它的电子版各个版本都属于便宜货。

=======================================================

站在前人的肩膀上：

## 关键点：

`00436BCA  |. 85C0           TEST EAX,EAX`

`00436BCC  |. 74 34          JE SHORT Out.00436C02`

`00436BCE  |. 68 503C7D5A    PUSH 5A7D3C50                            ; /Arg5 = 5A7D3C50`

`00436BD3  |. 8D8D FCFDFFFF  LEA ECX,DWORD PTR SS:[EBP-204]           ; |`

`00436BD9  |. 51             PUSH ECX                                 ; |Arg4`

`00436BDA  |. 8D85 F4FDFFFF  LEA EAX,DWORD PTR SS:[EBP-20C]           ; |`

`00436BE0  |. 50             PUSH EAX                                 ; |Arg3`

`00436BE1  |. 8D55 FC        LEA EDX,DWORD PTR SS:[EBP-4]             ; |`

`00436BE4  |. 52             PUSH EDX                                 ; |Arg2`

`00436BE5  |. 8D8D F8FDFFFF  LEA ECX,DWORD PTR SS:[EBP-208]           ; |`

`00436BEB  |. 51             PUSH ECX                                 ; |Arg1`

`00436BEC  |. E8 BD271A00    CALL DTPW.EXE.005D93AE                   ; \DTPW.EXE.005D93AE`

最关键的点是

`LEA ECX,DWORD PTR SS:[EBP-204]`

中的`(EBP-204)`指向内存中的数据，根据有狗或者无狗 反复查看判断，这个点的数据非常关键(事后证明也就是这个关键点)，

无狗数据： `{0x70,0x37,0xd3,0x00,0x00,0x00,0xD2,0x00,0x02,0x00,0x00,0x00,0x00,0x00,0x00,0x00}`;

有狗数据:  `{0x30,0x43,0x8E,0x24,0xD4,0xA6,0xC6,0x6C,0xCA,0xB6,0xC3,0xA7,0xBD,0x98,0xC9,0xC9}`;

然后在 `00436BEC  |. E8 BD271A00    CALL Out.005D93AE                        ; \Out.005D93AE`

所以在soft-ice中 幸亏的把数据手写到纸片上，然后把数据在辛苦的添到ebp-204指向的内存 ，运行

嘻嘻 无狗也是蒙泰版纹。

最后看了看没有合适的内存补丁软件，最后自己写了个VC的程序 其中利用 Debug Api。

程序很长时间没有维护了，所以很乱。大家慢慢看。

```c
#include "windows.h"

HINSTANCE                GhInstance;

//HANDLE                MutexHandle;

STARTUPINFO                gstStartUp;

PROCESS_INFORMATION        gpsInfo;

DEBUG_EVENT                gDBEvent;

CONTEXT                        gContext;

DWORD                        dwIdOld;

//BYTE                        buffer[2]={0x90,0x90};

//BYTE                        MYEIP1CODE[5]={0XE8,0X4F,0X00,0X00,0X00};//call xxxx

//BYTE                        MYEIP2CODE[2]={0x74,0x34};//jz +34

BYTE                        INT3CODE[1]={0XCC};

BYTE                        OldCodeBca[2]={0x85,0xC0};//test eax,eax

//BYTE                        StartCode[1]={0xa1};

unsigned long                StartEip=0x00401000;// 0xa1=>cc

unsigned long                MyEip1=0x00669E00;

unsigned long                MyEip2=0x00436BCA;

unsigned long                MyEip3=0x014dfdb0; //ebp-204// 98 is 0x014bfd94

long                        DoSth=0;

//BYTE                        TarGetData[16]={0x70,0x37,0xd3,0x00,0x00,0x00,0xD2,0x00,0x02,0x00,0x00,0x00,0x00,0x00,0x00,0x00};

BYTE                        WriteData[16]={0x30,0x43,0x8E,0x24,0xD4,0xA6,0xC6,0x6C,0xCA,0xB6,0xC3,0xA7,0xBD,0x98,0xC9,0xC9};

BYTE                        ReadCodeData[128];

BYTE                        ReadBuffer[10];

char                        szFileName[20]={".\\out.exe"};    // 目标文件名 Path.dat

long                        LastErrorCode;

char                        LpErrorText[40];

char                        LpDesCripeText[2048];

bool                        WhileDoFlag;

bool                        LoaderBkFlag;

int  APIENTRY WinMain(HINSTANCE hInstance,

                      HINSTANCE hPrevInstance,

                      LPSTR     lpCmdLine,

                      int       nCmdShow)

{

        GhInstance=hInstance;

        SetLastError(0);

        GetStartupInfo(&gstStartUp);

        if (CreateProcess(szFileName,NULL,NULL,NULL,FALSE,DEBUG_ONLY_THIS_PROCESS,0,0,&gstStartUp,&gpsInfo)==0)

                {

                LastErrorCode=GetLastError();

                wsprintf(LpErrorText,"创建Dun-packed.exe失败,检查目标程序是否存在ErrorCode=%2d",LastErrorCode);

                MessageBox(NULL,LpErrorText,"",MB_ICONERROR);

                return 0;

                }

        WhileDoFlag=true;

        SetLastError(0);

        while(WhileDoFlag)//调试循环

                {

                WaitForDebugEvent(&gDBEvent,INFINITE);

                gContext.ContextFlags=CONTEXT_FULL;

                GetThreadContext(gpsInfo.hThread,&gContext);

                LastErrorCode=GetLastError();

                switch(gDBEvent.dwDebugEventCode)

                        {

                        case EXCEPTION_DEBUG_EVENT://1debug 事件

                                if (gDBEvent.u.Exception.ExceptionRecord.ExceptionCode==EXCEPTION_BREAKPOINT)//断点

                                        {

                                        if (LoaderBkFlag==true)//第一次断点

                                                {

                                                gContext.ContextFlags= CONTEXT_FULL;

                                                GetThreadContext(gpsInfo.hThread,&gContext);

                                                gContext.Dr0=MyEip2;

                                                gContext.Dr7=0x0000-0000;

                                                SetThreadContext(gpsInfo.hThread,&gContext);

                                                //gContext.EFlags=gContext.EFlags | 0x100;//or regFlag,100h

                                                //SetThreadContext(gpsInfo.hThread,&gContext);

                                                //VirtualProtectEx(gpsInfo.hProcess,(void *)gContext.Eip,50,PAGE_READWRITE,&dwIdOld);

                                                //VirtualProtectEx(hProcess, lpAddr, 1, dwOldProt, &dwNewProt);

                                                if ((LastErrorCode=GetLastError())!=0)

                                                        {

                                                        wsprintf(LpErrorText,"GVirtualProtectEx出错了=%d",LastErrorCode);

                                                        MessageBox(0,LpErrorText,"出错",MB_OK);

                                                        }

                                                //ReadProcessMemory(gpsInfo.hProcess,(void *)StartEip,&ReadCodeData,sizeof(ReadCodeData),NULL);

                                                if ((LastErrorCode=GetLastError())!=0)

                                                        {

                                                        wsprintf(LpErrorText,"ReadProcessMemory出错了=%d",LastErrorCode);

                                                        MessageBox(0,LpErrorText,"出错",MB_OK);

                                                        }

                                                //wsprintf(LpDesCripeText,"补丁结果Eip=%4x,Code=%x %x %x

%x",gContext.Eip,ReadCodeData[0],ReadCodeData[1],ReadCodeData[2],ReadCodeData[3]);

                                                //MessageBox(0,LpDesCripeText,"第一次中断!",MB_OK);

                                                LoaderBkFlag=false;

                                                DoSth=1;

                                                }

                                        else if (LoaderBkFlag==false)

                                                {

                                                if (DoSth==1 && gContext.Eip==(MyEip2+1))

                                                        {

                                                        ReadProcessMemory(gpsInfo.hProcess,(void

*)gContext.Eip,&ReadCodeData,sizeof(ReadCodeData),NULL);

                                                        //wsprintf(LpDesCripeText,"Before Path

Eip=%4x,Code=%02x%02x%02x%02x",gContext.Eip,ReadCodeData[0],ReadCodeData[1],ReadCodeData[2],ReadCodeData[3]);

                                                        //MessageBox(0,LpDesCripeText,"第2次中断!",MB_OK);
```
