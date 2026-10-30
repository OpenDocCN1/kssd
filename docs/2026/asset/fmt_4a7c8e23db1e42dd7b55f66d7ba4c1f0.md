
```
kd> g
nt!IoFreeMdl: 83c826fa
8bff            mov     edi,edi
kd> kb
# ChildEBP RetAddr      Args to Child
00 80e5c9ec 8e3a6eb0     8757da60 00000000 8e38989f nt!IoFreeMdl
01 80e5ca08 8e3898ac     00000000 00000001 0ed23fa8 afd!AfdReturnTpInfo+0xad
02 80e5ca44 8e38abba     0ed23f00 000120c3 8e38aa8c afd!AfdTliGetTpInfo+0x89
03 80e5caec 8e38f2bc     890ad7c0 87610030 80e5cb14 afd!AfdTransmitPackets+0x12e
04 80e5cafc 83c43047     87610030 8913b7b8 8913b7b8 afd!AfdDispatchDeviceControl+0x3b
05 80e5cb14 83e199d5     890ad7c0 8913b7b8 8913b894 nt!IofCallDriver+0x63
06 80e5cb34 83e1bdc8     87610030 890ad7c0 00000000 nt!IopSynchronousServiceTail+0x1f8
07 80e5cbd0 83e22d9d     87610030 8913b7b8 00000000 nt!IopXxxControlFile+0x6aa
08 80e5cc04 83c4987a     00000050 00000000 00000000 nt!NtDeviceIoControlFile+0x2a
<Intermediate frames may have been skipped due to lack of complete unwind>
09 80e5cc04 778770b4 (T) 00000050 00000000 00000000 nt!KiFastCallEntry+0x12a
<Intermediate frames may have been skipped due to lack of complete unwind>
0a 0051f498 77875864 (T) 75a0989d 00000050 00000000 ntdll!KiFastSystemCallRet
0b 0051f49c 75a0989d     00000050 00000000 00000000 ntdll!ZwDeviceIoControlFile+0xc
0c 0051f4fc 75d2a671     00000050 000120c3 00156200 KERNELBASE!DeviceIoControl+0xf6
0d 0051f528 001517be     00000050 000120c3 00156200 kernel32!DeviceIoControlImplementation+0x80
WARNING: Stack unwind information not available. Following frames may be wrong.
0e 0051f718 00151396     00000012 00220ffc 00220fec CVE_2014_1767+0x17be
0f 0051f7b8 75d33c45     00000000 00000000 75d33c45 CVE_2014_1767+0x1396
10 0051f7d0 778937f5     7ffde000 77c39b57 00000000 kernel32!BaseThreadInitThunk+0xe
11 0051f810 778937c8     001514c0 7ffde000 00000000 ntdll!__RtlUserThreadStart+0x70
12 0051f828 00000000     001514c0 7ffde000 00000000 ntdll!_RtlUserThreadStart+0x1b
kd> g
// 此处如果再g一次，便可以得到本文一开始的BSOD现场
*** Fatal System Error:
0x000000c2 (0x00000007,0x00001097,0x08B50005,0x8757DA60)
Break instruction exception - code 80000003 (first chance)
A fatal system error has occurred.
Debugger entered on first try; Bugcheck callbacks have not been invoked.
A fatal system error has occurred.
For analysis of this file, run !analyze -v
nt!RtlpBreakWithStatusInstruction:
83c6cd00 cc int     3
kd> !analyze -v
// 通过下文的堆栈分析可以发现现场和本文一开始一致
Connected to Windows 7 7601 x86 compatible target at (Tue Mar 22 14:02:00.608 2022 (UTC + 8:00)), ptr64 FALSE
Loading Kernel Symbols
...............................................................
.................................................................
.
Press ctrl-c (cdb, kd, ntsd) or ctrl-break (windbg) to abort symbol loads that take too long.
Run !sym noisy before .reload to track down problems loading symbols.
.........
Loading User Symbols
................
Loading unloaded module list
...............
**********************************************************************************
*
*                        Bugcheck Analysis
*
**********************************************************************************

BAD_POOL_CALLER (c2)
The current thread is making a bad pool request. Typically this is at a bad IRQL level or double freeing the same allocation, etc.
Arguments:
Arg1: 00000007, Attempt to free pool which was already freed
Arg2: 00001097, Pool tag value from the pool header
Arg3: 08b50005, Contents of the first 4 bytes of the pool header
Arg4: 8757da60, Address of the block of pool being deallocated

Debugging Details:
------------------

**********************************************************************************
***
***
***
Either you specified an unqualified symbol, or your debugger
***
***
doesn't have full symbol information. Unqualified symbol
***
***
resolution is turned off by default. Please either specify a
***
***
fully qualified symbol module!symbolname, or enable resolution
***
***
of unqualified symbols by typing ".symopt- 100". Note that
***
***
enabling unqualified symbol resolution with network symbol
***
***
server shares in the symbol path may cause the debugger to
***
***
appear to hang for long periods of time when an incorrect
***
***
symbol name is typed or the network symbol server is down.
***
***
***
For some commands to work properly, your symbol path
***
***
must point to .pdb files that have full type information.
***
***
***
Certain .pdb files (such as the public OS symbols) do not
***
***
contain the required information. Contact the group that
***
***
provided you with these symbols if you need this command to
***
***
work.
***
***
***
Type referenced: kernel32!gpServerNlsUserInfo
***
***
**********************************************************************************

KEY_VALUES_STRING: 1
Key  : Analysis.CPU.mSec
Value: 5296
Key  : Analysis.DebugAnalysisManager
Value: Create
Key  : Analysis.Elapsed.mSec
Value: 12813
Key  : Analysis.Init.CPU.mSec
Value: 7218
Key  : Analysis.Init.Elapsed.mSec
Value: 127788
Key  : Analysis.Memory.CommitPeak.Mb
Value: 147
Key  : WER.OS.Branch
Value: win7sp1_rtm
Key  : WER.OS.Timestamp
Value: 2010-11-19T18:50:00Z
Key  : WER.OS.Version
Value: 7.1.7601.17514
BUGCHECK_CODE:  c2
BUGCHECK_P1: 7
BUGCHECK_P2: 1097
BUGCHECK_P3: 8b50005
BUGCHECK_P4: ffffffff8757da60
POOL_ADDRESS: 8757da60 Nonpaged pool
FREED_POOL_TAG:  Mdl_
PROCESS_NAME:  CVE-2014-1767.exe
STACK_TEXT:
80e5c54c 83ce6589     00000003 611ba6c5 00000065 nt!RtlpBreakWithStatusInstruction
80e5c59c 83ce7085     00000003 8757da58 000001ff nt!KiBugCheckDebugBreak+0x1c
80e5c960 83d2cc4e     000000c2 00000007 00001097 nt!KeBugCheck2+0x68b
80e5c9d8 83c8276a     8757da60 00000000 89107380 nt!ExFreePoolWithTag+0x1b2
80e5c9ec 8e3a6eb0     8757da60 00000000 8e38989f nt!IoFreeMdl+0x70
80e5ca08 8e3898ac     00000000 00000001 0ed23fa8 afd!AfdReturnTpInfo+0xad
80e5ca44 8e38abba     0ed23f00 000120c3 8e38aa8c afd!AfdTliGetTpInfo+0x89
80e5caec 8e38f2bc     890ad7c0 87610030 80e5cb14 afd!AfdTransmitPackets+0x12e
80e5cafc 83c43047     87610030 8913b7b8 8913b7b8 afd!AfdDispatchDeviceControl+0x3b
80e5cb14 83e199d5     890ad7c0 8913b7b8 8913b894 nt!IofCallDriver+0x63
80e5cb34 83e1bdc8     87610030 890ad7c0 00000000 nt!IopSynchronousServiceTail+0x1f8
80e5cbd0 83e22d9d     87610030 8913b7b8 00000000 nt!IopXxxControlFile+0x6aa
80e5cc04 83c4987a     00000050 00000000 00000000 nt!NtDeviceIoControlFile+0x2a
<Intermediate frames may have been skipped due to lack of complete unwind>
80e5cc04 778770b4 (T) 00000050 00000000 00000000 nt!KiFastCallEntry+0x12a
<Intermediate frames may have been skipped due to lack of complete unwind>
0051f498 77875864 (T) 75a0989d 00000050 00000000 ntdll!KiFastSystemCallRet
0051f49c 75a0989d     00000050 00000000 00000000 ntdll!ZwDeviceIoControlFile+0xc
0051f4fc 75d2a671     00000050 000120c3 00156200 KERNELBASE!DeviceIoControl+0xf6
// 此时control code为0x120c3
0051f528 001517be     00000050 000120c3 00156200 kernel32!DeviceIoControlImplementation+0x80
WARNING: Stack unwind information not available. Following frames may be wrong.
0051f718 00151396     00000012 00220ffc 00220fec CVE_2014_1767+0x17be
0051f7b8 75d33c45     00000000 00000000 75d33c45 CVE_2014_1767+0x1396
0051f7d0 778937f5     7ffde000 77c39b57 00000000 kernel32!BaseThreadInitThunk+0xe
0051f810 778937c8     001514c0 7ffde000 00000000 ntdll!__RtlUserThreadStart+0x70
0051f828 00000000     001514c0 7ffde000 00000000 ntdll!_RtlUserThreadStart+0x1b

SYMBOL_NAME:  afd!AfdReturnTpInfo+ad
MODULE_NAME: afd
IMAGE_NAME:  afd.sys
STACK_COMMAND:  .thread ; .cxr ; kb
FAILURE_BUCKET_ID: 0xc2_7_Mdl__afd!AfdReturnTpInfo+ad
OS_VERSION: 7.1.7601.17514
BUILDLAB_STR:  win7sp1_rtm
OSPLATFORM_TYPE:  x86
OSNAME:  Windows 7
FAILURE_ID_HASH:  {7fe1e721-1d80-7be3-9354-8d3b5b5ab1ef}
Followup:     MachineOwner
```
