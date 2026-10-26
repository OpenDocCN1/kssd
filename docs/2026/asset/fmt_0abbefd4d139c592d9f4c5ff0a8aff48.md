
```6502 Assembly
0:000> !analyze -v
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
\* \*
\* Exception Analysis \*
\* \*
\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*
\*\*\* WARNING: Unable to verify checksum for Qt5Widgets.dll
\*\*\* WARNING: Unable to verify checksum for Qt5Gui.dll
\*\*\* WARNING: Unable to verify checksum for qwindows.dll
KEY\_VALUES\_STRING: 1
Key : AV.Fault
Value: Execute
Key : Analysis.CPU.mSec
Value: 952
Key : Analysis.Elapsed.mSec
Value: 3586
Key : Analysis.IO.Other.Mb
Value: 0
Key : Analysis.IO.Read.Mb
Value: 0
Key : Analysis.IO.Write.Mb
Value: 0
Key : Analysis.Init.CPU.mSec
Value: 374
Key : Analysis.Init.Elapsed.mSec
Value: 154065
Key : Analysis.Memory.CommitPeak.Mb
Value: 192
Key : Failure.Bucket
Value: SOFTWARE\_NX\_FAULT\_c0000005\_Qt5Core.dll!QMetaObject::cast
Key : Failure.Hash
Value: {8d8d331f-0b04-8cd4-6ec8-5928245527c2}
Key : Timeline.Process.Start.DeltaSec
Value: 102650
Key : WER.OS.Branch
Value: vb\_release
Key : WER.OS.Version
Value: 10.0.19041.1
FILE\_IN\_CAB: crashdump[2023\_11\_02\_09\_32\_29].dmp
CONTEXT: (.ecxr)
eax=71dedb94 ebx=012bd4d0 ecx=041f4358 edx=29eac848 esi=6cf55108 edi=041f4358
eip=7d830675 esp=010fd4b8 ebp=041f4358 iopl=0 nv up ei pl nz na po nc
cs=0023 ss=002b ds=002b es=002b fs=0053 gs=002b efl=00210202
7d830675 ?? ???
Resetting default scope
EXCEPTION\_RECORD: (.exr -1)
ExceptionAddress: 7d830675
ExceptionCode: c0000005 (Access violation)
ExceptionFlags: 00000000
NumberParameters: 2
Parameter[0]: 00000008
Parameter[1]: 7d830675
Attempt to execute non-executable address 7d830675
PROCESS\_NAME: LaserMaker.exe
EXECUTE\_ADDRESS: 7d830675
FAILED\_INSTRUCTION\_ADDRESS:
+0
7d830675 ?? ???
ERROR\_CODE: (NTSTATUS) 0xc0000005 - 0x%p 0x%p %s
EXCEPTION\_CODE\_STR: c0000005
EXCEPTION\_PARAMETER1: 00000008
EXCEPTION\_PARAMETER2: 7d830675
IP\_ON\_HEAP: 7d830675
The fault address in not in any loaded module, please check your build's rebase
log at <releasedir>\bin\build\_logs\timebuild\ntrebase.log for module which may
contain the address if it were loaded.
STACK\_TEXT:
WARNING: Frame IP not in any known module. Following frames may be wrong.
010fd4b4 683e2972 683e2960 041f4358 6cb9704a 0x7d830675
010fd4c0 6cb9704a 041f4358 010fd69c 0128c4f8 Qt5Core!QMetaObject::cast+0x12
010fd4e0 6cb60332 041f4358 010fd69c 041f4358 Qt5Widgets!QGestureManager::filterEvent+0x1a
010fd624 683defe1 041f4358 010fd69c 7fbd5a58 Qt5Widgets!QApplication::notify+0x192
010fd66c 6cb6453e 041f4358 010fd69c 1d00eff0 Qt5Core!QCoreApplication::notifyInternal2+0xb1
010fd6a4 6cb64b93 27316130 2975e7b8 00000001 Qt5Widgets!QApplicationPrivate::setPalette\_helper+0x1ee
010fd6bc 6cb5f170 010fd6d4 041f4a08 29eabdf8 Qt5Widgets!QApplicationPrivate::setSystemPalette+0x63
010fd6d8 6cb621f3 29880170 69a4d8d2 00000000 Qt5Widgets!initSystemPalette+0x70
010fd6e0 69a4d8d2 00000000 1d00eff0 29880170 Qt5Widgets!QApplicationPrivate::notifyThemeChanged+0x33
010fd6f0 69a378d1 29880170 fffffffe 012a76e8 Qt5Gui!QGuiApplicationPrivate::processThemeChanged+0x12
010fd710 6c7f1e72 00000024 00000113 6841fce8 Qt5Gui!QWindowSystemInterface::sendWindowSystemEvents+0x161
010fd7a0 75a90f4b 003c0398 00000113 fffffffe qwindows!qt\_plugin\_query\_metadata+0x15e2
010fd7cc 75a87efa 6841fa00 003c0398 00000113 user32!\_InternalCallWinProc+0x2b
010fd8b0 75a85c6a 6841fa00 00000000 00000113 user32!UserCallWinProcCheckWow+0x33a
010fd924 75a85a30 00000013 010ff5e4 6841f581 user32!DispatchMessageWorker+0x22a
010fd930 6841f581 010ff5ac 7fbd79d0 012a76e8 user32!DispatchMessageW+0x10
010ff6e4 6c7f1e55 00000024 043461a8 010ff784 Qt5Core!QEventDispatcherWin32::processEvents+0x3c1
010ff744 683dde8d 00000000 7fbd7ba8 0128a6c8 qwindows!qt\_plugin\_query\_metadata+0x15c5
010ff79c 0056fd52 6405bd91 0059844c 012e0ae8 Qt5Core!QCoreApplication::exec+0x14d
010ff864 00581f04 00000001 0129f0d8 00000000 LaserMaker!main+0x1c2
010ff8e4 771cfcc9 00eb7000 771cfcb0 010ff950 LaserMaker!WinMain+0xe4
010ff8f4 77377c6e 00eb7000 bd63b1a2 00000000 kernel32!BaseThreadInitThunk+0x19
010ff950 77377c3e ffffffff 77398c2d 00000000 ntdll!\_\_RtlUserThreadStart+0x2f
010ff960 00000000 00581478 00eb7000 00000000 ntdll!\_RtlUserThreadStart+0x1b
STACK\_COMMAND: ~0s; .ecxr ; kb
FAULTING\_SOURCE\_LINE: c:\users\qt\work\qt\qtbase\src\corelib\kernel\qmetaobject.cpp
FAULTING\_SOURCE\_FILE: c:\users\qt\work\qt\qtbase\src\corelib\kernel\qmetaobject.cpp
FAULTING\_SOURCE\_LINE\_NUMBER: 355
FAULTING\_SOURCE\_CODE:
No source found for 'c:\users\qt\work\qt\qtbase\src\corelib\kernel\qmetaobject.cpp'
SYMBOL\_NAME: qt5core!QMetaObject::cast+12
MODULE\_NAME: Qt5Core
IMAGE\_NAME: Qt5Core.dll
FAILURE\_BUCKET\_ID: SOFTWARE\_NX\_FAULT\_c0000005\_Qt5Core.dll!QMetaObject::cast
OS\_VERSION: 10.0.19041.1
BUILDLAB\_STR: vb\_release
OSPLATFORM\_TYPE: x86
OSNAME: Windows 10
IMAGE\_VERSION: 5.9.3.0
FAILURE\_ID\_HASH: {8d8d331f-0b04-8cd4-6ec8-5928245527c2}
Followup: MachineOwner
---------
```

![图片描述](upload/attach/202311/940598\_VPPGD4VTTFZDV94.webp)

3. 输入`~*kvn`命令，查看所有线程堆栈

生成如下结果：
