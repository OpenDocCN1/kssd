
# StarForce 加密小窥

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-250.htm`](https://www.pediy.com/kssd/pediy07/pediy7-250.htm)

今天小弟 hook 一个函数，不小心却发现了 `StarForce` 超度虚拟光驱的一点秘密。

游戏名称《大富翁 7》，保护方式 `Starforce 3.3`，今天偶 hook `ScsiPortInitialize` 函数跟 `Starforce` 撞车，也许这对大家早就不是秘密了，可是对偶却是新鲜。所以贴上来让大家见笑。

`starforce` 保护文件的名称 `prosync1.sys`

这个文件属性如下：

该文件的说明

`StarForce Protection Synchronization Driver`

产品版本

`version 3.3`

`StarForce Protection System`

```asm
.text:000103E5 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?
.text:000103E8             align 10h
.text:000103F0 aScsiportinitia db 'ScsiPortInitialize',0 ; DATA XREF: sub_10411+A4o
.text:00010403             align 4
.text:00010404 aNtoskrnl_exe   db 'ntoskrnl.exe',0     ; DATA XREF: sub_10411+E4o
.text:00010411
.text:00010411 ; 圹圹圹圹圹圹圹圹圹?S U B R O U T I N E 圹圹圹圹圹圹圹圹圹圹圹圹圹圹圹圹圹圹圹圹?
.text:00010411
.text:00010411
.text:00010411 sub_10411       proc near           ; CODE XREF: start+85p
.text:00010411                 push    esi
.text:00010412                 push    0
.text:00010414                 push    656E6F4Eh
.text:00010419                 push    10h
.text:0001041B                 push    0
.text:0001041D                 push    0
.text:0001041F                 push    0
.text:00010421                 push    offset unk_111B0
.text:00010426                 call    sub_10340
.text:0001042B                 push    0
.text:0001042D                 push    656E6F4Eh
.text:00010432                 push    0Ch
.text:00010434                 push    0
.text:00010436                 push    0
.text:00010438                 push    0
.text:0001043A                 push    offset unk_11238
.text:0001043F                 call    sub_10340
.text:00010444                 push    offset unk_111A4
.text:00010449                 call    ds:KeInitializeSpinLock
.text:0001044F                 push    2Ch
.text:00010451                 push    0
.text:00010453                 push    offset dword_1120C
.text:00010458                 call    memset
.text:0001045D                 add     esp, 0Ch
.text:00010460                 mov     byte_11200, 0
.text:00010467                 mov     eax, offset dword_11288
.text:0001046C                 mov     dword_1128C, eax
.text:00010471                 mov     dword_11288, eax
.text:00010476                 mov     dword_11208, 0
.text:00010480                 call    sub_10871
.text:00010485                 mov     esi, eax
.text:00010487                 test    esi, esi
.text:00010489                 jz      short loc_104A5
.text:0001048B                 push    edi
.text:0001048C                 mov     edi, ds:ExDeleteNPagedLookasideList
.text:00010492                 push    offset unk_11238
.text:00010497                 call    edi ; ExDeleteNPagedLookasideList
.text:00010499                 push    offset unk_111B0
.text:0001049E                 call    edi ; ExDeleteNPagedLookasideList
.text:000104A0                 mov     eax, esi
.text:000104A2                 pop     edi
.text:000104A3                 pop     esi
.text:000104A4                 retn
.text:000104A5 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?
.text:000104A5
.text:000104A5 loc_104A5:                     ; CODE XREF: sub_10411+78j
.text:000104A5                 push    offset loc_11100
.text:000104AA                 call    sub_10ABA
.text:000104AF                 push    eax
.text:000104B0                 call    sub_10ACA
.text:000104B5                 push    offset aScsiportinitia ; "ScsiPortInitialize"
.text:000104BA                 push    eax
.text:000104BB                 mov     dword_111A0, eax
.text:000104C0                 call    sub_10AFA //这里是 hook "ScsiPortInitialize",
.text:000104C5                 push    0
.text:000104C7                 push    0
.text:000104C9                 push    0
.text:000104CB                 push    0
.text:000104CD                 mov     dword_111A8, eax
.text:000104D2                 call    PsGetVersion
.text:000104D7                 test    al, al
.text:000104D9                 jz      short loc_10500
.text:000104DB                 push    offset loc_110EE
.text:000104E0                 call    sub_10ABA
.text:000104E5                 push    offset nullsub_1
.text:000104EA                 mov     dword_11204, eax
.text:000104EF                 push    eax
.text:000104F0                 mov     eax, dword_111A0
.text:000104F5                 push    offset aNtoskrnl_exe ; "ntoskrnl.exe"
.text:000104FA                 push    eax
.text:000104FB                 call    sub_10C0A
.text:00010500
.text:00010500 loc_10500:                     ; CODE XREF: sub_10411+C8j
.text:00010500                 mov     ecx, dword_111A8
.text:00010506                 mov     edx, dword_111A0
.text:0001050C                 push    offset sub_10531 //hook "ScsiPortInitialize"的程序的位置，以后程序调用 ScsiPortInitialize 就先进 starfoce 的 sub_10531
.text:00010511                 push    ecx
.text:00010512                 push    edx
.text:00010513                 call    sub_10BAA
.text:00010518                 xor     eax, eax
.text:0001051A                 pop     esi
.text:0001051B                 retn
.text:0001051B sub_10411       endp ; sp = -10h
.text:0001051B
.text:0001051B ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?
.text:0001051C                 dd 90909090h
.text:00010520                 db 90h
.text:00010521 ; [00000003 BYTES: COLLAPSED FUNCTION nullsub_1. PRESS KEYPAD "+" TO EXPAND]
.text:00010524                 dd 3 dup(90909090h)
.text:00010530                 db 90h
.text:00010531

////////////////////////////////////////////////////////////////////////////////////////////////////////////
///由于 prosync1.sys 以后只要有 ScsiPortInitialize 调用 StarForce 就先接管进这下边的程序。你是虚拟光驱就难逃法眼。
///Starforce 可以在你虚拟光区驱动起来之前，这里先去探测你的 scsi 卡的寄存器。当你有 IDE 光驱连接的时候，StarForce
如果探测寄存器失败。这下就可以判断出你是虚拟光驱了。叫你换盘插入。嘿嘿，你只有拔掉 IDE 光驱线骗 StarForce，StarForce
//拔掉 IDE 光驱线后由于探测不到寄存器所以不敢错杀一千，可以让虚拟光驱通过这万里检查的第一步。后边的 DPM 的检测才是
//真正利害的！
.text:00010531 ; 圹圹圹圹圹圹圹圹圹?S U B R O U T I N E 圹圹圹圹圹圹圹圹圹圹圹圹圹圹圹圹圹圹圹圹?
.text:00010531
.text:00010531
.text:00010531 sub_10531       proc near           ; DATA XREF: sub_10411+FBo
.text:00010531
.text:00010531 arg_0           = dword ptr  10h
.text:00010531 arg_4           = dword ptr  14h
.text:00010531 arg_8           = dword ptr  18h
.text:00010531 arg_C           = dword ptr  1Ch
.text:00010531
.text:00010531                 push    ebp
.text:00010532                 push    esi
.text:00010533                 push    edi
.text:00010534                 push    offset unk_111B0
.text:00010539                 call    sub_10370
.text:0001053E                 mov     edi, eax
.text:00010540                 test    edi, edi
.text:00010542                 jnz     short loc_1054F
.text:00010544                 pop     edi
.text:00010545                 pop     esi
.text:00010546                 mov     eax, 0C000009Ah
.text:0001054B                 pop     ebp
.text:0001054C                 retn    10h
.text:0001054F ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?
.text:0001054F
.text:0001054F loc_1054F:                     ; CODE XREF: sub_10531+11j
.text:0001054F                 mov     eax, [esp+arg_C]
.text:00010553                 mov     ecx, [esp+arg_8]
.text:00010557                 mov     edx, [esp+arg_4]
.text:0001055B                 mov     esi, [esp+arg_0]
.text:0001055F                 push    eax
.text:00010560                 push    ecx
.text:00010561                 push    edx
```



```
.text:00010562                push    esi
.text:00010563                call    dword_111A8
.text:00010569                mov     ebp, eax
.text:0001056B                and     eax, 0C0000000h
.text:00010570                cmp     eax, 0C0000000h
.text:00010575                jnz     short loc_1058A
.text:00010577                push    edi
.text:00010578                push    offset unk_111B0
.text:0001057D                call    sub_103B0
.text:00010582                pop     edi
.text:00010583                mov     eax, ebp
.text:00010585                pop     esi
.text:00010586                pop     ebp
.text:00010587                retn    10h
.text:0001058A ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?

.text:0001058A loc_1058A:                        ; CODE XREF: sub_10531+44j
.text:0001058A                push    ebx
.text:0001058B                mov     ecx, offset unk_111A4
.text:00010590                call    ds:KfAcquireSpinLock
.text:00010596                push    esi
.text:00010597                mov     bl, al
.text:00010599                call    sub_10631
.text:0001059E                test    eax, eax
.text:000105A0                jnz     short loc_105CB
.text:000105A2                mov     eax, [esi+34h]
.text:000105A5                mov     ecx, [esi+74h]
.text:000105A8                test    eax, eax
.text:000105AA                mov     [edi+4], esi
.text:000105AD                mov     [edi+8], ecx
.text:000105B0                mov     [edi+0Ch], eax
.text:000105B3                mov     dword ptr [esi+74h], offset sub_10661
.text:000105BA                jz      short loc_105C3
.text:000105BC                mov     dword ptr [esi+34h], offset sub_107E1
.text:000105C3

.text:000105C3 loc_105C3:                        ; CODE XREF: sub_10531+89j
.text:000105C3                push    edi
.text:000105C4                call    sub_105F1
.text:000105C9                jmp     short loc_105D6
.text:000105CB ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?

.text:000105CB loc_105CB:                        ; CODE XREF: sub_10531+6Fj
.text:000105CB                push    edi
.text:000105CC                push    offset unk_111B0
.text:000105D1                call    sub_103B0
.text:000105D6

.text:000105D6 loc_005D6:                        ; CODE XREF: sub_10531+98j
.text:000105D6                mov     dl, bl
.text:000105D8                mov     ecx, offset unk_111A4
.text:000105DD                call    ds:KfReleaseSpinLock
.text:000105E3                pop     ebx
.text:000105E4                pop     edi
.text:000105E5                mov     eax, ebp
.text:000105E7                pop     esi
.text:000105E8                pop     ebp
.text:000105E9                retn    10h
.text:000105E9 sub_10531       endp
.text:000105E9

.text:000105E9 ; 哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪哪?

.text:000105EC                dd 90909090h
.text:000105F0                db 90h
.text:000105F1

.text:000105F1 ; 圹圹圹圹圹圹圹?S U B R O U T I N E 圹圹圹圹圹圹圹圹圹圹圹圹圹圹圹圹圹圹圹圹?
```

/////////////////////////////////////////////////////////////////////////////////////////////////////////////////

你也许会说了 `ScsiPortInitialize()` 这个函数这样重要吗？嘿嘿，对了。`ScsiPortInitialize` 是虚拟光驱的第一口呼吸。

`StarForce` hook 该函数的目的就是想把虚拟光驱扼杀在哭出第一声来之前，真是太惨了。连摇篮都不愿意给虚拟光驱准备。偶一直也不知道 `ScsiPortInitialize` 的内幕，最近不是 `M$` 的 `NT4-SRC` 泄露出来了吗？嘿嘿以下函数就是在 `NT4-SRC` 中带的这个函数的细节，有兴趣的朋友可以去看看。

```
/*++
Copyright (c) 1990  Microsoft Corporation

Module Name:

    port.c

Abstract:

    This is the NT SCSI port driver.  This file contains the initialization
    code.

Authors:

    Mike Glass
    Jeff Havens

Environment:

    kernel mode only

Notes:

    This module is a driver dll for scsi miniports.

Revision History:

ULONG
ScsiPortInitialize(
    IN PVOID Argument1,
    IN PVOID Argument2,
    IN PHW_INITIALIZATION_DATA HwInitializationData,
    IN PVOID HwContext OPTIONAL
    )

/*++
Routine Description:

    This routine initializes the port driver.

Arguments:

    Argument1 - Pointer to driver object created by system
    HwInitializationData - Miniport initialization structure
    HwContext - Value passed to miniport driver's config routine

Return Value:

    The function value is the final status from the initialization operation.

--*/

{
    PDRIVER_OBJECT    driverObject = Argument1;
    ULONG             slotNumber = 0;
    ULONG             functionNumber = 0;
    PDEVICE_EXTENSION deviceExtension = NULL;
    NTSTATUS          returnStatus = STATUS_DEVICE_DOES_NOT_EXIST;
    NTSTATUS          status;
```

不过对付这东西没办法了吗？现在不是有 `starFuck` 了吗？名字取的不错，问我，我也不知道其是啥原理，因为我没用过，但我想可以通过这样的方法躲过这个检查，就是在 `starfoce prosync1.sys` 驱动起来之前建立一个驱动，让它成为“虚拟光驱的虚拟光驱”。当 `prosync1.sys` 在 hook 处理 `ScsiPortInitialize()` 的时候返回一些端口数据来骗 `starforce`，不过 `daemon-tools` 和 `alcochol` 好象没这样处理。谁知道呢？`daemon` 的 `DPM` 数据精度探测处理骗过 `starforce` 非常利害。

偶发这个帖子只是想抛砖引玉 `StarFOrce` 在驱动中就有 5-6 个拦路虎，望各位大大多多给予指正。其中处理虚拟光驱的一点点，

Machoman
2005/4/5

新版本的 `StarForce` 已经把系统的 `IDE`、`SCSI` 严密监视起来了，之所以 `USB` 易驱能够对付现在的新版本的 `StarForce` 是由于 `StarForce` 研究人员还没搞定 `USB` 驱动的原因。

目前不脱壳对付 `StarForce` 新版本的方法我个人感觉是有两个：

1. 绕过系统的设备驱动层直接读扇区，突破 `StarForce` 的监视。但这一点谈何容易，首先要绕过文件系统，然，文件很可能在硬盘上是离散分布的，做到不通过文件系统打开文件本来就很难。然后还要采用 `out` 指令直接读扇区，这样可以绕过 `StarForce` 的检查。
2. 早于 `StarForce` 驱动起来之前保存 `idt` 干掉 `StarForce`，这又回到 `Dos` 年代破解的关键问题，谁先启动谁先有优先权利。

个人拙见，定有错误，但我发现各位高手对 `starforce` 谈论比较保守。希望大家多指教。技术的东西保留着没用的。

根据个人研究发现 `starforce` 是一种非常危险的保护方式。它为游戏厂家或者 `starforce` 获得游戏玩家的所有信息（它把 `IDE`、`SCSI` 以后 `USB` 存储设备完全监视起来了。你能有安全可言吗），这也是在欧美国家不敢用 `StarForce` 加密的原因。感觉 `StarForce` 善于耍阴招而已。比较违法的。

论实际的技术 `SecuRom New` 新版本在介质加密技术硬工夫上是远超 `StarForce` 的。

`StarForce` 确实开启了光盘加密的一个坏方向，牺牲安全保证加密。

謝謝 `machoman` 兄的分析。干掉 `IDT` 是不可行的，因為 `starforce` 是用 `int1` 來進入 `Ring0` 運行光驅檢查和主程序解密，連 `DeviceIoControl` 和 `sysenter` 都不用，最可惡的是他們在 `Ring0` 也執行 `int1` 和 `int3` 及自己用 `Memory Breakpoint`。不過這不是問題，最大問題是 `starforce` 用偽代碼來運行 `DPC` 及 `StartIO` 等，非常難以跟踨。我只是看到 `starforce` 時常在等某個 `Kernel Mutext` 及 `DPC`，所以要由 `starforce driver` 入手破解是非常困難。據小弟了解，即使是 `SafeDisc` 及 `SecuRom` 只是用驅動程序來運行光驅檢查，沒有改掉 `IDT` 及內核代碼，即使 `SecuRom7` 也是用 `Service` 來運行。

其實我覺得 `starforce` 的 `pcode` 已經夠強。沒必要干改內核代碼這一步，這已經涉及到 `Windows` 的穏定及操作。

東西太剛強是很容易折斷的。所以當他們的保護做到變成鐡銿之時也是他們未日的來臨，因為那時恐怕已經沒有多個系統能夠穏定的運行 `starforce` 了。

云兄，感谢您参与讨论，偶对 `StarForce` 的 `Pcode` 和驱动采用伪代码运行确实不知。谢谢你的知讯。对于脱壳偶是一点不了解，感觉 `StarForce` 确实有入铁桶了。既然您说干 `idt` 是不行的，我想第一种直接文件读取的方式是否可行，不通过系统，直接读 `io Port`，这需要一个简单的文件系统，没有采取分段链表连接的文件系统，从 `StarFuck` 的实现功能看来，`StarForce` 还没 `BT` 到把 `io port` 都封住，采取直接 `io` 操作不知道是否可行躲过其磁盘监视。

感谢 `machoman` 的回答。小弟直覺認為 `starforce` 不會直接用 `IO port`，因為 `SCSI`、`USB`、`SATA` 等很多都須要用驅動程式來運作，不是一兩個 `IO port` 可以解決的。可以肯定的是他們用 `IRP packet` 來讀光驅，再行個 `DPC` 來查光驅是否正確，由於大部份代碼都是 `pcode`，我們唯一能做的就是由 `IRP stack` 下手，因為 `DT` 和 `Alcohol` 是在這裏模擬 `RMPS` 的。但是我對 `Daemon Tools` 怎樣運作模擬 `RMPS` 不是很了解，所以不知要怎樣下手。相信 `starforce` 是非常了解 `Daemon Tools` 的運作，在 `SCSI` 的 `IRP stack` 動手腳才能 `Blacklist DT`。我們可以在 `Starforce` 動手之後 `IO` 完結之前把他撥亂反正，讓 `RMPS` 資料能夠順利通過檢查。

不過很明顯做比說還困難千萬倍，否則早就有人干了。相信 `DT` 的開發者也知道這一點，不難理解 `Daemon Tools 4` 要等這麼久（已差不多一年了）還是未出。

補充一下關於 `Starforce Nightmare` 的運作，相信他是用 `SetupDi` 等 `API` 來運作，很明顯新版的 `Starforce` 封銷了 `Device Notification` 之類的回饋信息或 `Device` 控制等從根本上廢掉 `SN` 的功能，所以看來一定要由驅動程式入手，而且要非常精於 `CDROM` 的 `Low-level` 運作及內核 `IRP` 的運用才能破掉 `starforce`。恐怕只有 `DT` 的開發者和精於寫 `CDROM` 驅動程式之人員才懂得做，以小弟現時的道行還真遠遠未夠編這方面的程式。

> 引用：
>
> * * *
>
> *最初由 云淡风輕 发布*
>
> **RPMS 是 Recordable Media Physical Signature.就是光碟上特有的物理鑑別信息. DPM 是 Data Position Measurement.就是用來讀取這些鑑別信息的技術. 至於詳細是怎樣運作，小弟就不知道了。**
>
> * * *

谢谢云兄的解释。就我的理解 `DPM` 其实就是对光盘上不同的扇区段采取不同的“线密度”的方式存储，这个“线密度”保存在 `DPM` 中成为 4 字节的值，当检测的时候读不同位置采取不同的“线密度”延迟就可以做到 `DPM` 的模拟。



RPMS 的原理小第目前还没研究出眉目，好象大概也是这个意思，不过是刻在光盘上。但 RPMS 有一个好处就是可以支持 USB 接口，这好象是现在对付新版的唯一方法，StarForce Nightmare 作者在上边提到的更好方法实现起来难度太大(自己写文件系统、设备驱动、)，非小第所能为。就连 Daemon-tools 目前也没有解决

```
4dd466d0e1e0cc123afbe504da2462ca
```
