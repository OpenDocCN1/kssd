
# 我的第二阶段第一题，这个题目解答

> 原文：[`www.pediy.com/kssd/pediy09/pediy09-280.htm`](https://www.pediy.com/kssd/pediy09/pediy09-280.htm)

> `seg000:00000000` `B5 33 F3 EF` `dd 0EFF333B5h` `; 特殊数,让最后计算结果刚好拷贝 0x0D(13)个 DWORD`

> `seg000:00000000` `; 此数用穷举可以得到。我的穷举代码见后面`

> `seg000:00000004` `00 00 00 00` `dd 0`

> `seg000:00000008` `00` `db 0`

> `seg000:00000009` `; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?`

> `seg000:00000009` `6A 00` `push 0` `; push 0`

> `seg000:0000000B` `E8 04 00 00 00` `call loc_14`

> `seg000:0000000B` `; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?`

> `seg000:00000010` `4F 4B 21 00` `aOk` `db 'OK!',0` `; push offset "OK!"`

> `seg000:00000014` `; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?`

> `seg000:00000014`

> `seg000:00000014` `loc_14:` `; CODE XREF: seg000:0000000Bp`

> `seg000:00000014` `E8 04 00 00 00` `call loc_1D` `; push 0`

> `seg000:00000014` `; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?`

> `seg000:00000019` `4F 4B 21 00` `aOk_0` `db 'OK!',0` `; push offset "OK!"`

> `seg000:0000001D` `; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?`

> `seg000:0000001D`

> `seg000:0000001D` `loc_1D:` `; CODE XREF: seg000:loc_14p`

> `seg000:0000001D` `6A 00` `push 0` `; push 0`

> `seg000:0000001F` `FF 15 4C 02 40 00` `call dword ptr ds:40024Ch` `; Call MessageBoxA`

> `seg000:00000025` `B8 E7 03 40 00` `mov eax, 4003E7h` `; 返回地址`

> `seg000:0000002A` `8B FC` `mov edi, esp`

> `seg000:0000002C` `33 C9` `xor ecx, ecx`

> `seg000:0000002E` `EB 04` `jmp short loc_34`

> `seg000:0000002E` `; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?`

> `seg000:00000030` `F5 02 40 00` `dd 4002F5h` `; 溢出时让执行的代码地址。我刚好再让 ret 一次到压的栈的参数里面去`

> `seg000:00000034` `; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?`

> `seg000:00000034`

> `seg000:00000034` `loc_34:` `; CODE XREF: seg000:0000002Ej`

> `seg000:00000034` `49` `dec ecx`

> `seg000:00000035` `F3 AF` `repe scasd` `; 搜索返回地址；我没有直接用 EBP,ESP 相对地址，是不是很傻啊？`

> `seg000:00000037` `8D 67 FC` `lea esp, [edi-4]`

> `seg000:0000003A` `8D 6F 18` `lea ebp, [edi+18h]` `; 修正 ESP,EBP`

> `seg000:0000003D` `C7 04 24 97 03 40 00` `mov dword ptr [esp], 400397h` `; 修改返回地址; 返回; 完成功能`

> `seg000:00000044` `33 FF` `xor edi, edi`

> `seg000:00000046` `C3` `retn`

```
ULONGLONG X;
ULONGLONG N1 = 0x78CC02A869948F1B;
ULONGLONG N2 = 0x5BE6FF82A5164785;
ULONGLONG K;
//X * N1 % N2 % 0x100000000 = XXXXXXXX0000000D

//K = 0x0D;
K = N1;
X = 0;
K = 0;
//
K = 0xB3D097F555179B6A;
X = 0x0000000453E8A6DE;
for (;;)
{
  if (((K % N2) & 0xFFFFFFFF) == 0x0D)
  {
    printf("%8X%08X\n", (DWORD)(X >> 32), (DWORD)X);
  }
  K = K + N1;//利用加法，加快计算速度
  X++;
  if (X == 0) break;
}
printf("Test\n");
```

这个题纯属送分题，我不知道，我一下提交了 1-32 种办法，可能还有 33,34 我忘记了）

付 32 种办法代码(Delphi 版)附件传不上来，就贴关键代码了

```
SendMessage(hWnd, WM_CLOSE, 0, 0);
SendMessage(hWnd, WM_COMMAND, IDOK, 0);
SendMessage(hWnd, WM_SYSCOMMAND, SC_CLOSE, 0);
SendMessage(hWnd, WM_KEYDOWN, VK_ESCAPE, 0);
PostMessage(hWnd, WM_CLOSE, 0, 0);
PostMessage(hWnd, WM_QUIT, 0, 0);
PostMessage(hWnd, WM_COMMAND, IDOK, 0);
PostMessage(hWnd, WM_SYSCOMMAND, SC_CLOSE, 0);
PostMessage(hWnd, WM_KEYDOWN, VK_ESCAPE, 0);
PostThreadMessage(GetWindowThreadProcessId(hWnd, nil), WM_QUIT, 0, 0);
SendNotifyMessage(hWnd, WM_CLOSE, 0, 0);
SendNotifyMessage(hWnd, WM_COMMAND, IDOK, 0);
SendNotifyMessage(hWnd, WM_SYSCOMMAND, SC_CLOSE, 0);
SendMessageTimeout(hWnd, WM_CLOSE, 0, 0, SMTO_ABORTIFHUNG, 100, nil);
SendMessageTimeout(hWnd, WM_COMMAND, IDOK, 0, SMTO_ABORTIFHUNG, 100, nil);
SendMessageTimeout(hWnd, WM_SYSCOMMAND, SC_CLOSE, 0, SMTO_ABORTIFHUNG, 100, nil);
```

```
SetForegroundWindow(hWnd);
keybd_event(VK_ESCAPE, 0, 0, 0);
keybd_event(VK_ESCAPE, 0, KEYEVENTF_KEYUP, 0);

SetForegroundWindow(hWnd);
keybd_event(VK_MENU, 0, 0, 0);
keybd_event(VK_F4, 0, 0, 0);
keybd_event(VK_MENU, 0, KEYEVENTF_KEYUP, 0);
keybd_event(VK_F4, 0, KEYEVENTF_KEYUP, 0);
```

```
if GetWindowThreadProcessId(hWnd, @dwProcessId) = 0 then Exit;
hSnap := CreateToolhelp32Snapshot(TH32CS_SNAPTHREAD, dwProcessId);
if hSnap = INVALID_HANDLE_VALUE then Exit;
te.dwSize := SizeOf(te);
if Thread32First(hSnap, te) then
begin
  repeat
    if te.th32OwnerProcessID <> dwProcessId then continue;
    dwThreadId := te.th32ThreadID;
    if dwThreadId <> 0 then
    begin
      hThread := OpenThread(THREAD_TERMINATE, False, dwThreadId);
      if hThread <> 0 then
      begin
        TerminateThread(hThread, 0);
        CloseHandle(hThread);
      end;
    end;
  until not Thread32Next(hSnap, te);
end;
CloseHandle(hSnap);
```

```
HOOK->ExitProcess(0);
HOOK->TerminateProcess(GetCurrentProcess(), 0);
```

```
HOOK->//调用 crackme 的函数->TerminateProcess(hProcess, 0);
dwProcessId := DWORD(-1);
asm
  pushad
  mov eax, 00404198h
  mov eax, [eax + 020h]//12FE90h
  mov ecx, [eax + 060h]
  mov edx, [ecx]
  lea eax, R
  push eax
  push 0
  push 0
  push 4
  lea eax, dwProcessId
  push eax
  push 0830020C0h
  call dword ptr [edx + 020h]
  popad
end;
```

```
HOOK->->//调用 crackme 的函数->DebugActiveProcess(dwProcessId);
```

```
HOOK->
hJob := CreateJobObject(nil, 'CRACKME.JOB');
if hJob = 0 then
begin
  CloseHandle(hProcess);
  Exit;
end;
if AssignProcessToJobObject(hJob, hProcess) then
begin
  TerminateJobObject(hJob, 0);
end;
CloseHandle(hProcess);
CloseHandle(hJob);
```

```
HOOK->RemoteThread
procedure StartRemoteThread(hProcess: THandle);
var
  VM: Pointer;
  hThread: THandle;
  dwThreadId: THandle;
  RemoteThreadCode: TRemoteThreadCode;
  dw: DWORD;
begin
  VM := VirtualAllocEx(hProcess, nil, $1000, MEM_COMMIT, PAGE_EXECUTE_READWRITE);
  if VM = nil then Exit;

  RemoteThreadCode.b0 := $6A;
  RemoteThreadCode.b1 := $00;
  RemoteThreadCode.b2 := $68;
  RemoteThreadCode.fnExitProcess := DWORD(GetProcAddress(GetModuleHandle(kernel32), 'ExitProcess'));
  RemoteThreadCode.b3 := $C3;
  RemoteThreadCode.b4 := $C2;
  RemoteThreadCode.b5 := $04;
  RemoteThreadCode.b6 := $00;
```




```
WriteProcessMemory(hProcess, VM, @RemoteThreadCode, SizeOf(RemoteThreadCode), dw);
hThread := CreateRemoteThread(hProcess, nil, 0, VM, nil, 0, dwThreadId);
if hThread <> 0 then
begin
  WaitForSingleobject(hThread, 500);
  CloseHandle(hThread);
end;
VirtualFreeEx(hProcess, VM, 0, MEM_RELEASE);
end;
```

```
HookOn(GetWindowThreadProcessId(hWnd, nil));
Sleep(500);
TerminateProcess(ReadProcessHandle(), 0);
HookOff();
CloseHandle(ReadProcessHandle());
```

```
HookOn(GetWindowThreadProcessId(hWnd, nil));
Sleep(500);
HookOff();
if GetWindowThreadProcessId(hWnd, @dwProcessId) = 0 then Exit;
```

```
hProcess := ReadProcessHandle();
if hProcess = 0 then Exit;
hJob := CreateJobObject(nil, 'CRACKME.JOB');
if hJob = 0 then
begin
  CloseHandle(hProcess);
  Exit;
end;
if AssignProcessToJobObject(hJob, hProcess) then
begin
  TerminateJobObject(hJob, 0);
end;
CloseHandle(hProcess);
CloseHandle(hJob);
```

HOOK->

```
HookOn(GetWindowThreadProcessId(hWnd, nil));
Sleep(500);
HookOff();
GetWindowThreadProcessId(hWnd, @dwProcessId);
if DbgUiConnectToDbg() < 0 then Exit;
hProcess := OpenProcess(PROCESS_ALL_ACCESS, False, dwProcessid);
DbgUiDebugActiveProcess(hProcess);
CloseHandle(hProcess);
```

```
HookOn(GetWindowThreadProcessId(hWnd, nil));
Sleep(500);
HookOff();
if GetWindowThreadProcessId(hWnd, @dwProcessId) = 0 then Exit;
```

```
hProcess := ReadProcessHandle();
if hProcess = 0 then Exit;
hJob := CreateJobObject(nil, 'CRACKME.JOB');
if hJob = 0 then
begin
  CloseHandle(hProcess);
  Exit;
end;
if AssignProcessToJobObject(hJob, hProcess) then
begin
  TerminateJobObject(hJob, 0);
end;
CloseHandle(hProcess);
CloseHandle(hJob);
```

```
HookOn(GetWindowThreadProcessId(hWnd, nil));
Sleep(200);
HookOff();
if GetWindowThreadProcessId(hWnd, @dwProcessId) = 0 then Exit;
hProcess := ReadProcessHandle();
if hProcess = 0 then Exit;

StartRemoteThread(hProcess);

CloseHandle(hProcess);
```

```
HookOn(GetWindowThreadProcessId(hWnd, nil));
Sleep(200);
HookOff();
GetWindowThreadProcessId(hWnd, @dwProcessId);
if DbgUiConnectToDbg() < 0 then Exit;
hProcess := ReadProcessHandle();
DbgUiDebugActiveProcess(hProcess);
CloseHandle(hProcess);
```

```
if GetWindowThreadProcessId(hWnd, @dwProcessId) <> 0 then
begin
  Len := $1000;
  GetMem(P, Len);
  while NtQuerySystemInformation(SystemProcessesAndThreadsInformation, P, Len, nil) = STATUS_INFO_LENGTH_MISMATCH do
  begin
    FreeMem(P);
    Len := Len * 2;
    GetMem(P, Len);
  end;
  P1 := PSYSTEM_PROCESSES(P);
  while P1^.NextEntryDelta <> 0 do
  begin
    if P1^.ProcessId = dwProcessId then
    begin
      //开始终止所有线程
      for I := 0 to P1^.ThreadCount - 1 do
      begin
        P2 := PSYSTEM_THREADS(@P1^.Threads);
        hThread := OpenThread(THREAD_TERMINATE , False, P2^.ClientId.UniqueThread);
        TerminateThread(hThread, 0);
        CloseHandle(hThread);
        Inc(P2);
      end;
      break;
    end;
    P1 := PSYSTEM_PROCESSES(DWORD(P1) + P1^.NextEntryDelta);
  end;
  FreeMem(P);
end;
```

`4dd466d0e1e0cc123afbe504da2462ca`
