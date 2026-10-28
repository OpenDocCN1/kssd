
![图片描述](upload/attach/202203/625952_WV2R2K2XXAPRSWQ.png)

看一下英伟达显卡的PCIe配置空间

1. 先确定总线地址（总线号.设备号.功能号）

![图片描述](upload/attach/202203/625952_DTFWUQ2Q2R3369C.png)

windbg执行如下命令

![图片描述](upload/attach/202203/625952_6E4J3M4R6FZD62Q.png)
![图片描述](upload/attach/202203/625952_XJBB6UPDRN8UKK8.png)
![图片描述](upload/attach/202203/625952_5N4CJ324CDA5BZ5.png)
![图片描述](upload/attach/202203/625952_CN5ZG4VEGGZGZBC.png)
![图片描述](upload/attach/202203/625952_R4B9MMM8VJAQU9J.png)

英伟达真是强，把微软干的事，他自己干了一遍。

### 综上：

1. 代码是真实无误的
2. 英伟达对驱动有着很高的追求（代码里有很多微软的影子，很明显有微软帮忙）
3. 代码对现有中国windows GPU驱动开发有着巨大的研究和学习价值，值得业内人士深挖
4. 驱动代码是研究英伟达显卡最好的的资料
5. 时机成熟可自行编译，调试

以上分析是粗略分析，只是为了抛砖引玉，希望有更多的人来研究这份代码。

里面还有海量的信息供大家挖掘。相信可以找到很多有趣的信息。

BTC挖矿让英伟达飞起来了，当然也与英伟达在并行计算机领域的强大技术能力有关。这次国外黑客把英伟达的代码公开给大家。让中国人有机会一睹英伟达的显卡驱动的芳容。

此贴就重点来研究一下英伟达windows驱动代码。显卡驱动对许多人来说还是比较神密的。首先做PC显卡的那就三家：nvidia,intel,amd。这就造成了windows显卡驱动几乎没什么人接触，研究。即使是做游戏辅助的人也很少观注显卡驱动本身。

本帖就以号称显卡技术巅峰的nvidia驱动为研究对象进行剖析，以此来抛砖引玉。

源代码下载链接就不提供了，大家可以自行谷歌，非常好找。

剖析的方式包括分析源代码，ida逆向binary驱动，windbg调试硬件，相互佐证代码的真实性。

如下图所示，为了安全起见，建议放在vmware虚拟机进行解压，分析，原因是黑客可能在某些脚本里植入有病毒（当然不去执行里面的代码是没问题的）

![图片描述](https://bbs.kanxue.com/upload/attach/202203/625952_7S95VWYFWHTSB8H.png)
![图片描述](https://bbs.kanxue.com/upload/attach/202203/625952_NBSSAFBHN9PWFB9.png)

第1步，先把代码载入source insight里

![图片描述](https://bbs.kanxue.com/upload/attach/202203/625952_KZGXBNYEMZVWCJM.png)

第2步，找出驱动的入口点`DriverEntry`

![图片描述](https://bbs.kanxue.com/upload/attach/202203/625952_QPJ59JEP9UFFQFM.png)
![图片描述](https://bbs.kanxue.com/upload/attach/202203/625952_UE9WC23NQ5EH3WE.png)
![图片描述](https://bbs.kanxue.com/upload/attach/202203/625952_Y3TQNHSTH287Z3A.png)

很明显代码是用C++编写，也用到了很多的类及模板

![图片描述](https://bbs.kanxue.com/upload/attach/202203/625952_YH3EXWF8RYTNN6G.png)

初始化部分代码很长，分段分析

![图片描述](https://bbs.kanxue.com/upload/attach/202203/625952_J6YU3W7V7YV9VUA.png)
![图片描述](https://bbs.kanxue.com/upload/attach/202203/625952_RVP6W47VPZWGPEA.png)
![图片描述](https://bbs.kanxue.com/upload/attach/202203/625952_CN56VE7JNZJHQNT.png)
![图片描述](https://bbs.kanxue.com/upload/attach/202203/625952_Z936P595893D6EN.png)
![图片描述](https://bbs.kanxue.com/upload/attach/202203/625952_UHFNEXEJHSAQVGE.png)
![图片描述](https://bbs.kanxue.com/upload/attach/202203/625952_KDFWGBHR6R3V5S5.png)
![图片描述](https://bbs.kanxue.com/upload/attach/202203/625952_6QSSUX38ZH9CED4.png)
![图片描述](https://bbs.kanxue.com/upload/attach/202203/625952_65SJNJQ5VVSH73M.png)
![图片描述](https://bbs.kanxue.com/upload/attach/202203/625952_6HMSSR5DHEJQQP7.png)
![图片描述](https://bbs.kanxue.com/upload/attach/202203/625952_86V63BXKZ5BDKPX.png)
![图片描述](https://bbs.kanxue.com/upload/attach/202203/625952_BP8D7F2MCGBMWKY.png)
![图片描述](https://bbs.kanxue.com/upload/attach/202203/625952_E5ZXZ5KDJFNRTWW.png)
![图片描述](https://bbs.kanxue.com/upload/attach/202203/625952_Z8XSWD9FCA4XNAG.png)

现在来看一下最重要的函数`DxgkInitialize`的具体流程

![图片描述](https://bbs.kanxue.com/upload/attach/202203/625952_JXU2RKDYRD3UTE4.png)
![图片描述](https://bbs.kanxue.com/upload/attach/202203/625952_8MZXMRHCFNJ7GE8.png)
![图片描述](https://bbs.kanxue.com/upload/attach/202203/625952_EQY637QMAEG2E3S.png)
![图片描述](https://bbs.kanxue.com/upload/attach/202203/625952_GZGTBFN7DCTKZS4.png)

流程结束

## 现在来看一下nvdia真实显卡驱动的二进制分析

配置好调试环境，用windbg连上

![图片描述](https://bbs.kanxue.com/upload/attach/202203/625952_7UCB24SRFXUCM9B.png)

定位驱动

![图片描述](https://bbs.kanxue.com/upload/attach/202203/625952_UJEDN2RX5S3JS47.png)

``` 
4: kd> lm

start end module name

fffff802`4b000000 fffff802`4b01d000 NDProxy (deferred)
fffff802`4b020000 fffff802`4b048000 AgileVpn (deferred)
fffff802`4b050000 fffff802`4b071000 rasl2tp (deferred)
fffff802`4b080000 fffff802`4b0a1000 raspptp (deferred)
fffff802`4b0b0000 fffff802`4b0cc000 raspppoe (deferred)
fffff802`4b0d0000 fffff802`4b0df000 ndistapi (deferred)
fffff802`4b0e0000 fffff802`4b11b000 ndiswan (deferred)
fffff802`4b120000 fffff802`4b168000 winnat (deferred)
fffff802`4b170000 fffff802`4b18b000 WdNisDrv (deferred)
fffff802`4b190000 fffff802`4b1b6000 MpKslDrv (deferred)
fffff802`4b1c0000 fffff802`4b1cf000 terminpt (deferred)
fffff802`4b1d0000 fffff802`4b2a6000 peauth (deferred)
fffff802`4b2b0000 fffff802`4b2b9000 SangforDnsDrv (deferred)
fffff802`4b2c0000 fffff802`4b2cf000 SangforTcpDrv (deferred)
fffff802`4b2d0000 fffff802`4b2e5000 tcpipreg (deferred)
fffff802`4b2f0000 fffff802`4b3b7000 srv2 (deferred)
fffff802`4b3c0000 fffff802`4b3d2000 xlwfp (deferred)
fffff802`4b3e0000 fffff802`4b3fc000 rassstp (deferred)
fffff802`4bec0000 fffff802`4bec6000 hal (deferred)
fffff802`4bed0000 fffff802`4bf32000 kd_02_10ec (deferred)
fffff802`4bf40000 fffff802`4bf89000 kdcom (deferred)
fffff802`4bf90000 fffff802`4bfb7000 tm (deferred)
fffff802`4bfc0000 fffff802`4bfda000 PSHED (deferred)
fffff802`4bfe0000 fffff802`4bfeb000 BOOTVID (deferred)
fffff802`4bff0000 fffff802`4bffe000 cmimcext (deferred)
fffff802`4c60e000 fffff802`4d654000 nt (pdb symbols) C:\Program Files (x86)\Windows Kits\10\Debuggers\x64\sym\ntkrnlmp.pdb\118018959D8D7CA5AAB45B75AED5A9761\ntkrnlmp.pdb
fffff802`4d800000 fffff802`4d86b000 CLFS (deferred)
fffff802`4d870000 fffff802`4d984000 clipsp (deferred)
fffff802`4d990000 fffff802`4d9ff000 FLTMGR (deferred)
fffff802`4da00000 fffff802`4da29000 ksecdd (deferred)
fffff802`4da30000 fffff802`4da91000 msrpc (deferred)
fffff802`4daa0000 fffff802`4dab1000 werkernel (deferred)
fffff802`4dac0000 fffff802`4dacc000 ntosext (deferred)
fffff802`4dad0000 fffff802`4dbb5000 CI (deferred)
fffff802`4dbc0000 fffff802`4dc7b000 cng (deferred)
fffff802`4dc80000 fffff802`4dd21000 VerifierExt (deferred)
fffff802`4dd30000 fffff802`4de01000 Wdf01000 (deferred)
fffff802`4de10000 fffff802`4de23000 WDFLDR (deferred)
fffff802`4de30000 fffff802`4de3f000 SleepStudyHelper (deferred)
fffff802`4de40000 fffff802`4de51000 WppRecorder (deferred)
fffff802`4de60000 fffff802`4de86000 acpiex (deferred)
fffff802`4de90000 fffff802`4dee6000 mssecflt (deferred)
fffff802`4def0000 fffff802`4df0a000 SgrmAgent (deferred)
fffff802`4df10000 fffff802`4dfdc000 ACPI (deferred)
fffff802`4dfe0000 fffff802`4dfec000 WMILIB (deferred)
fffff802`4dff0000 fffff802`4dffb000 IntelTA (deferred)
fffff802`4e020000 fffff802`4e08b000 intelpep (deferred)
fffff802`4e090000 fffff802`4e0a7000 WindowsTrustedRT (deferred)
fffff802`4e0b0000 fffff802`4e0bb000 WindowsTrustedRTProxy (deferred)
fffff802`4e0c0000 fffff802`4e0d4000 pcw (deferred)
fffff802`4e0e0000 fffff802`4e0f9000 MSDMFilt (deferred)
fffff802`4e100000 fffff802`4e10b000 msisadrv (deferred)
fffff802`4e110000 fffff802`4e187000 pci (deferred)
fffff802`4e190000 fffff802`4e1a5000 vdrvroot (deferred)
fffff802`4e1b0000 fffff802`4e1df000 pdc (deferred)
fffff802`4e1e0000 fffff802`4e1f9000 CEA (deferred)
fffff802`4e200000 fffff802`4e231000 partmgr (deferred)
fffff802`4e240000 fffff802`4e2eb000 spaceport (deferred)
fffff802`4e2f0000 fffff802`4e309000 volmgr (deferred)
fffff802`4e310000 fffff802`4e373000 volmgrx (deferred)
fffff802`4e380000 fffff802`4e39e000 mountmgr (deferred)
fffff802`4e3a0000 fffff802`4e3d2000 storahci (deferred)
fffff802`4e3e0000 fffff802`4e494000 storport (deferred)
fffff802`4e4a0000 fffff802`4e4cd000 stornvme (deferred)
fffff802`4e4d0000 fffff802`4e4ec000 EhStorClass (deferred)
fffff802`4e4f0000 fffff802`4e50a000 fileinfo (deferred)
fffff802`4e510000 fffff802`4e550000 Wof (deferred)
fffff802`4e560000 fffff802`4e5ce000 WdFilter (deferred)
fffff802`4e5d0000 fffff802`4e8a9000 Ntfs (deferred)
fffff802`4e8b0000 fffff802`4e8bd000 Fs_Rec (deferred)
fffff802`4e8c0000 fffff802`4ea2f000 ndis (deferred)
fffff802`4ea30000 fffff802`4eac8000 NETIO (deferred)
fffff802`4ead0000 fffff802`4eb02000 ksecpkg (deferred)
fffff802`4eb10000 fffff802`4edfc000 tcpip (deferred)
fffff802`4ee00000 fffff802`4ee7f000 fwpkclnt (deferred)
fffff802`4ee80000 fffff802`4eeb0000 wfplwfs (deferred)
fffff802`4eec0000 fffff802`4eed0000 VmsProxy (deferred)
fffff802`4eee0000 fffff802`4ef04000 vmbkmclr (deferred)
```
