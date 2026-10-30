
# [原创]精心构造代码，使 OMAPClock 超频设置永久有效

用 OMAP CPU 的用户可能知道 `OMAP Clock` 可以超频。但由于 OMAP CPU 本身的特性，在超频之后如果设备关闭（设备不使用，一般是两分钟后），再次开启时，频率又恢复成默认的了。

`OMAPCLock` 本身没有对这些行为作出相关操作，导致 `OMAP Clock` 用的非常不爽。

我稍微分析了一下 `OMAP Clock` 的代码，感觉精心构造一小段的代码，应该可以解决这些问题。方案如下：

1. 在原来程序提示设置频率的 `MessageBox` 处加入代码，总共 11 条指令，刚刚好够空间。
2. 由于增加导入表不方便，干脆取消掉 `About` 显示功能，然后修改 `LoadResource`、`DialogBoxindirectParam` 和 `EndDialog` 的导入函数为 `DeleteFileW`、`SHCreateShortcut` 和 `CeRunAppAtEven`，这样我们就可以调用这三个函数了。
2. 修改代码，调用 `SHCreateShortcut` 创建快捷方式（之前还要调用 `DeleteFile` 掉快捷方式否则无法创建，我晕倒！）。`\Windows\Startup\OMAPClock.lnk`，指向 `\Windows\OMAPClock.exe -clock <频率>`
3. 调用 `CeRunAppAtEnent` 清除掉所有的 `\Windows\Startup\OMAPClock.lnk` 事件。（否则事件通知会越来越多）
3. 调用 `CeRunAppAtEnent` 设置在设备唤醒时调用 `\Windows\Startup\OMAPClock.lnk`。
4. `CeRunAppEnent` 正好返回 `R0` 为 1，所以会继续以设置主频。

经过一天下午的调试分析和实际操作，终于解决了 `OMAPClock` 的这个问题。

## 调试代码片段

```
.text:0001165C sub_1165C                               ; CODE XREF: sub_11358+30p
.text:0001165C                                         ; sub_11B8C+C0p
.text:0001165C
.text:0001165C szTarget        = -0x198
.text:0001165C arg_8           =  8
.text:0001165C
.text:0001165C                 MOV     R12, SP
.text:00011660                 STMFD   SP!, {R0-R3}
.text:00011664                 STMFD   SP!, {R12,LR}
.text:00011668                 SUB     SP, SP, #0x190
.text:0001166C                 MOV     R2, R1          ; wchar_t *
.text:00011670                 ADD     R3, SP, #0x198+arg_8 ; va_list
.text:00011674                 MOV     R1, #0xC8       ; size_t
.text:00011678                 ADD     R0, SP, #0x198+szTarget ; wchar_t *
.text:0001167C                 BL      _vsnwprintf
.text:0001167C
.text:00011680                 LDR     R0, =s_WindowsStartu ; lpFileName
.text:00011684                 BL      DeleteFileW
.text:00011684
.text:00011688                 LDR     R0, =s_WindowsStartu ; szShortcut
.text:0001168C                 ADD     R1, SP, #0x198+szTarget ; szTarget
.text:00011690                 BL      SHCreateShortcut
.text:00011690
.text:00011694                 MOV     R1, #0          ; lWhichEvent
.text:00011698                 LDR     R0, =s_WindowsStartu ; pwszAppName
.text:0001169C                 BL      CeRunAppAtEvent
.text:0001169C
.text:000116A0                 MOV     R1, #0xB        ; lWhichEvent
.text:000116A4                 LDR     R0, =s_WindowsStartu ; pwszAppName
.text:000116A8                 BL      CeRunAppAtEvent
.text:000116A8
.text:000116AC                 ADD     SP, SP, #0x190
.text:000116B0                 LDMFD   SP, {SP,PC}
.text:000116B0
.text:000116B0 ; End of function sub_1165C
.text:000116B0
.text:000116B0 ; ---------------------------------------------------------------------------
.text:000116B4                 DCD unk_1480C
.text:000116B8 ; LPCWSTR lpFileName
.text:000116B8 lpFileName      DCD s_WindowsStartu     ; DATA XREF: sub_1165C+24r
.text:000116B8                                         ; sub_1165C+2Cr
.text:000116B8                                         ; sub_1165C+3Cr
.text:000116B8                                         ; sub_1165C+48r
.text:000116B8                                         ; "\\Windows\\Startup\\OMAPClock.lnk"
```

理论上 `OMAPClock` 支持 TI `OMAP CPU` 的 Smartphone 和 Pocket PC。但后来发现 Smartphone 的 `CeRunAppAtEvent` 的 Wakeup 事件竟然无效，又是一个弱智 API，Windows Mobile Team 的这些头脑进水的垃圾工程师，总是让我们有不断的“惊喜”发现——Smartphone 不支持唤醒事件，换句话说，新增功能不支持 Smartphone，Smrtphone SDK 的文档上没说，但是全世界的论坛都在说这样的问题（ffcK9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6i4m8U0M7X3g2$3K9h3g2%4i4K6u0W2j5$3!0Q4x3X3g2#2K9#2)9J5c8X3k6G2M7Y4g2E0M7#2)9J5c8Y4c8Z5M7X3g2S2k6q4)9J5k6o6t1@1y4U0M7&6z5e0m8Q4x3X3g2H3K9s2m8Q4c8f1k6Q4b7V1y4Q4z5p5y4Z5N6s2c8H3i4K6y4m8i4K6u0r3i4K6u0r3N6%4N6%4i4K6u0W2M7r3c8S2M7%4c8J5k6h3g2@1i4K6u0W2j5$3!0E0i4K6u0r3k6X3!0J5N6h3#2K6i4K6u0r3M7$3S2G2N6%4c8Z5M7X3g2S2k6q4)9J5k6i4m8Z5M7q4)9K6c8Y4c8Q4x3@1b7K6x3U0x3@1y4W2!0q4c8W2!0n7b7#2)9^5z5g2!0q4x3#2)9^5x3q4)9^5x3R3`.

由于以上代码修改的原因，此版本 `OMAPClock` 具有以下局限和特性：

1. 取消了 `About` 对话框（不影响使用）。
2. 取消了 设置频率的警告（不影响使用）。
3. 文件夹 `\Windows\Startup` 必须存在。
4. `OMAPClock` 程序路径必须是 `\Windows\OMAPClock.exe`。
5. 新增功能无法支持 Smartphone。

以上要求很容易满足（几乎所有系统中默认就是这样，只要把我修改的 `OMAPClock.exe` 放在 `\Windows` 中即可），所以这次修改可以看成“完美”了，哈哈……

注意，以上关机代码，仅针对 `DOPOD 830` 有效，不同的设备的关机 `KernelIOCode` 不同，我调试出了很多机器的 `Code`，但都忘了记录下来。有心人可以分析一下一 `HTC` 机器的 `PowerOffWarning.exe` 可得出。另外下面是我 `CeleCmd` 中关机和硬启的函数部分，已经有一些特定设备的关机和硬启功能（`ASUS P527`、`HTC Diamond`、`HTC Prophet`）：

```c
// SHUTdown

#include <PM.h>

#define IOCTL_HAL_REBOOT		0x101003C

#define IOCTL_HAL_SHUTDOWN		0x1012000

#define IOCTL_COLDBOOT_P527		0x1012004

#define IOCTL_COLDBOOT_Prophet	0x1012594

#define IOCTL_COLDBOOT_Diamond	0x1012048

extern "C" BOOL WINAPI SetCleanRebootFlag();

extern "C" BOOL WINAPI GwesPowerOffSystem();

extern "C" BOOL WINAPI ExitWindowsEx(UINT uFlags, DWORD dwReason);

extern "C" BOOL KernelIoControl(DWORD dwIoControlCode, PVOID pInBuf, DWORD nInBufSize, PVOID pOutBuf, DWORD nOutBufSize, PDWORD pBytesReturned);

HRESULT SHUT(PCTSTR ptzCmd)

{

	BOOL bResult;

	switch (UChrToUpper(*ptzCmd))

	{

	case 'S':

		// HTC

		KernelIoControl(IOCTL_HAL_SHUTDOWN, NULL, 0, NULL, 0, NULL);



	case 'G':

		bResult = GwesPowerOffSystem();

		break;



	case 'P':

		bResult = ExitWindowsEx(EWX_POWEROFF, 0);

		break;



	case 'K':

		keybd_event(VK_OFF, 0, KEYEVENTF_SILENT, 0);

		keybd_event(VK_OFF, 0, KEYEVENTF_SILENT | KEYEVENTF_KEYUP, 0);

		bResult = TRUE;

		break;



	case 'H':

		// HTC

		KernelIoControl((ptzCmd[1] == '0') ? UStrToInt(ptzCmd + 1) : IOCTL_COLDBOOT_Diamond, NULL, 0, NULL, 0, NULL);



	case 'C':

		SetCleanRebootFlag();



	case 'R':

		bResult = KernelIoControl(IOCTL_HAL_REBOOT, NULL, 0, NULL, 0, NULL);

		break;



	default:

		bResult = ExitWindowsEx(EWX_REBOOT, 0);

		break;

	}



	return !bResult;

}
```

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
