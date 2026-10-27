
# [原创][开源]NTQ相关AntiDebug及Od反AntiDebug

上篇帖子：

PEB相关AntiDebug及Od反AntiDebug：

<http://bbs.pediy.com/showthread.php?p=1403899#post1403899>

由于判断逻辑都一样，所以只在第一个贴图贴代码![](https://bbs.kanxue.com/view/img/face/1.gif)

工具：VS2012 OD

- `NtQueryInformationProcess()`是一个可以同时在 R0 及 R3 运行的函数，它的主要作用是查看进程相关的各种信息。
- 根据想要查看信息类别不同，我们可以给其第二个参数`ProcessInformationClass`传的值也就不同。
- 根据`ProcessInformationClass`的类别可知，此函数可以查看 60 余种进程相关的信息。

## `ProcessDebugPort`

`ProcessDebugPort`可以获取目标进程的调试端口，如果目标进程未处于调试状态，此端口为`0`，否则为`0xFFFFFFFF`。

```
bool NTAPI_processDebugPort( )
{
  int nDebugPort = 0;
  NtQueryInformationProcess(GetCurrentProcess( ), // 目标进程句柄
              ProcessDebugPort, // 查询的信息类型
              &nDebugPort, // 输出查询信息
              sizeof(nDebugPort), // 输出信息的变量类型大小
              NULL// 实际返回数据大小
              );
  return  nDebugPort == 0xffffffff ? 1 : 0;
}
```

### OD 反 AntiDebug

拖入 OD，找`MAIN`函数：

![](https://bbs.kanxue.com/upload/attach/201511/607869_x3oclza55iu7c1h.png)

找关键跳[红色高亮]：

```
CPU Disasm

地址        十六进制数据            指令                                       注释

00E414E0    55              PUSH EBP                                 ; UNICODE "匎G"
00E414E1    8BEC            MOV EBP,ESP
00E414E3    81EC C0000000   SUB ESP,0C0
00E414E9    53              PUSH EBX
00E414EA    56              PUSH ESI
00E414EB    57              PUSH EDI
00E414EC    8DBD 40FFFFFF   LEA EDI,[EBP-0C0]
00E414F2    B9 30000000     MOV ECX,30
00E414F7    B8 CCCCCCCC     MOV EAX,CCCCCCCC
00E414FC    F3:AB           REP STOS DWORD PTR ES:[EDI]
00E414FE    E8 4DFBFFFF     CALL 00E41050
00E41503    0FB6C0          MOVZX EAX,AL
[COLOR="Black"][B]00E41506    85C0            TEST EAX,EAX
00E41508    74 1F           JE SHORT 00E41529
00E4150A    8BF4            MOV ESI,ESP
00E4150C    6A 00           PUSH 0
00E4150E    68 5858E400     PUSH OFFSET 00E45858                     ; UNICODE "提示"
00E41513    68 6058E400     PUSH OFFSET 00E45860                     ; UNICODE "被调试"
00E41518    6A 00           PUSH 0
00E4151A    FF15 8C93E400   CALL DWORD PTR DS:[<&USER32.MessageBoxW>
00E41520    3BF4            CMP ESI,ESP
00E41522    E8 28FCFFFF     CALL 00E4114F
00E41527    EB 1D           JMP SHORT 00E41546
00E41529    8BF4            MOV ESI,ESP
00E4152B    6A 00           PUSH 0
00E4152D    68 5858E400     PUSH OFFSET 00E45858                     ; UNICODE "提示"
00E41532    68 6C58E400     PUSH OFFSET 00E4586C                     ; UNICODE "正常运行"
00E41537    6A 00           PUSH 0
00E41539    FF15 8C93E400   CALL DWORD PTR DS:[<&USER32.MessageBoxW>
00E4153F    3BF4            CMP ESI,ESP
00E41541    E8 09FCFFFF     CALL 00E4114F
00E41546    33C0            XOR EAX,EAX[/B][/COLOR]
00E41548    5F              POP EDI
00E41549    5E              POP ESI
00E4154A    5B              POP EBX
00E4154B    81C4 C0000000   ADD ESP,0C0
00E41551    3BEC            CMP EBP,ESP
00E41553    E8 F7FBFFFF     CALL 00E4114F
00E41558    8BE5            MOV ESP,EBP
00E4155A    5D              POP EBP
00E4155B    C3              RETN

```

## `ProcessDebugObjectHandle`

`ProcessDebugObjectHandle`可以获取目标进程的调试对象的句柄，如果未处于调试状态则获取的值为`NULL`。

```
bool NQIP_ProcessDebugObjectHandle()
{
  HANDLE hProcessDebugObjectHandle=0;
  NtQueryInformationProcess(GetCurrentProcess(),        //目标（当前）进程句柄
              (PROCESSINFOCLASS)0x1e,        //查询信息类型
              &hProcessDebugObjectHandle,      //输出查询信息
              sizeof(hProcessDebugObjectHandle),  //查询类型大小
              NULL);                //实际返回大小
  return hProcessDebugObjectHandle?false:true;
}
```

## `ProcessDebugFlag`

`ProcessDebugFlag`可以获取目标进程的调试标记，如果处于调试状态其值为`0`，否则为`1`。

```
bool NQIP_ProcessDebugFlag()
{
  BOOL bProcessDebugFlag=0;
  NtQueryInformationProcess(GetCurrentProcess(),    //目标（当前）进程句柄
              (PROCESSINFOCLASS)0x1f,    //查询信息类型
              &bProcessDebugFlag,      //输出查询信息
              sizeof(bProcessDebugFlag),  //查询类型大小
              NULL);            //实际返回大小
  return bProcessDebugFlag?false:true;
}
```

## 更新一个：`ProcessBasicInformation`

`ProcessBasicInformation`可以获取指定进程的父进程 PID，我们可以将其与`ExPlorer.exe`的 PID 进行对比。如果不匹配则证明此进程不是双击运行的。

```
bool NQIP_CheckParentProcess()
{
	struct PROCESS_BASIC_INFORMATION
	{
		ULONG ExitStatus;				//进程返回码
		PPEB PebbaseAddress;			//PEB地址
		ULONG AffinityMask;				//CPU亲和性掩码
		LONG BasePriority;				//基本优先级
		ULONG UniqueProcessId;			//本进程PID
		ULONG InheritedFromUniqueProcessId;//父进程PID
	}stcProcInfo;

	NtQueryInformationProcess(GetCurrentProcess(),
								ProcessBasicInformation,
								&stcProcInfo,
								sizeof(stcProcInfo),
								NULL);

	DWORD ExplorerPID=0;
	DWORD CurrentPID=stcProcInfo.InheritedFromUniqueProcessId;

	GetWindowThreadProcessId(FindWindow(L"Progman",NULL),&ExplorerPID);

	return ExplorerPID==CurrentPID?false:true;
}
```

---

## 源码

[源码(已更新).rar](https://bbs.kanxue.com/attach-download-101563.htm)

[传递专业知识、拓宽行业人脉——看雪讲师团队等你加入！！](https://bbs.kanxue.com/thread-275828.htm)

[#调试逆向](https://bbs.kanxue.com/forum-4-1-1.htm)

### 上传的附件

- [8.png](javascript:void(0);)（22.06kb，11次下载）
- [源码(已更新).rar](javascript:void(0);)（16.20kb，128次下载）
