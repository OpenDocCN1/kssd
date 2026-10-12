
0006F268 10003A60 |`Buffer` = `lqcyc52_.10003A60`

0006F26C 0006F274 \\`pBufSize` = `0006F274`

0006F268 100094AF /CALL 到 `RegCloseKey`

0006F26C 00000000 \\`hKey` = `NULL`

0006F268 1000AD2D /CALL 到 `PathRemoveFileSpecA`

0006F26C 10003A60 \\`Path` = `""`

0006F274 1000BA8D /CALL 到 `WinExec`

0006F278 10003A60 |`CmdLine` = `"\SoftMgr\SoftManager.exe"`

0006F27C 00000001 \\`ShowState` = `SW_SHOWNORMAL`

0006F690 1000A3A1 /CALL 到 `ExitProcess`

0006F694 00000000 \\`ExitCode` = `0`

然后回想第一个调用

0006F690 100096A0 /CALL 到 `GetModuleHandleA`

0006F694 10002150 \\`pModule` = `"EXPLORER.EXE"`

是什么用意？我们修改返回值，让他不为 `0`，然后在运行，发现过程有变：

0006F67C 1000B395 /CALL 到 `CreateThread`

0006F680 00000000 |`pSecurity` = `NULL`

0006F684 00000000 |`StackSize` = `0`

0006F688 10001320 |`ThreadFunction` = `usp10_1.10001320`

0006F68C 00000000 |`pThreadParm` = `NULL`

0006F690 00000000 |`CreationFlags` = `0`

0006F694 0006F6A4 \\`pThreadId` = `0006F6A4`

直接来看这个线程地址：

```
10001320 81EC 08010000 sub esp, 108 ; 终于是没 VM 过的代码了。。
10001326 A1 00300010 mov eax, dword ptr [10003000]
1000132B 33C4 xor eax, esp
1000132D 898424 04010000 mov dword ptr [esp+104], eax
10001334 53 push ebx
10001335 8B1D 50200010 mov ebx, dword ptr [10002050] ; kernel32.WinExec
1000133B 56 push esi
1000133C 8B35 40200010 mov esi, dword ptr [10002040] ; kernel32.Sleep
10001342 57 push edi
10001343 8B3D 48200010 mov edi, dword ptr [10002048] ; kernel32.GetWindowsDirectoryA
10001349 8DA424 00000000 lea esp, dword ptr [esp]
10001350 68 D0070000 push 7D0
10001355 FFD6 call esi
10001357 68 04010000 push 104
1000135C 8D4424 10 lea eax, dword ptr [esp+10]
10001360 50 push eax
10001361 FFD7 call edi
10001363 8D4424 0C lea eax, dword ptr [esp+C]
10001367 48 dec eax
10001368 8A48 01 mov cl, byte ptr [eax+1]
1000136B 40 inc eax
1000136C 84C9 test cl, cl
1000136E ^ 75 F8 jnz short 10001368
10001370 8B0D F8200010 mov ecx, dword ptr [100020F8] ；看下这里是什么数据
```

`100020F8` `5C 73 79 73 74 65 6D 64 65 62 75 67 2E 65 78 65` `\systemdebug.exe`

`10002108` `00 00 00 00 52 69 73 69 6E 67 00 00 52 61 76 5C` `....Rising..Rav\`

`10002118` `00 00 00 00 4B 69 6E 67 73 6F 66 74 00 00 00 00` `....Kingsoft....`

`10002128` `6B 69 6E 67 73 6F 66 74 00 00 00 00 4B 61 73 70` `kingsoft....Kasp`

`10002138` `65 72 73 6B 79 00 00 00 4B 50 50 33 00 00 00 00` `ersky...KPP3....`

`10002148` `6B 77 61 74 63 68 00 00 45 58 50 4C 4F 52 45 52` `kwatch..EXPLORER`

`10002158` `2E 45 58 45 00 00 00 00 6C 71 63 79 63 35 32 2E` `.EXE....lqcyc52.`

`10002168` `63 79 63 00 00 00 00 00 00 00 00 00 00 00 00 00` `cyc.............`

`.... ...`

```
1000139B 8D5424 10 lea edx, dword ptr [esp+10] ; C:\WINDOWS\systemdebug.exe
1000139F 52 push edx
100013A0 8848 10 mov byte ptr [eax+10], cl
100013A3 FFD3 call ebx ; kernel32.WinExec
```

运行 `systemdebug.exe` 上面已经分析出 `systemdebug.exe` 是个下载者

```
100013A5 68 00DD6D00 push 6DDD00 ; 然后每过 7200000 毫秒重复上面的操作
100013AA FFD6 call esi ; kernel32.Sleep
```

`lqcyc52.cyc` 的功能是替换 `usp10.dll`，代替 `usp10.dll` 工作，如果被 `explorer.exe` 载入时运行 `systemdebug.exe`

![](img/3620e97f8873804c5523b7c0d227ad76.jpg)
