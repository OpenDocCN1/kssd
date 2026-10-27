
![](https://bbs.kanxue.com/upload/attach/202104/814951_JWWV2RMD8BU2TGJ.jpg)

```
not 00007fff`800fffff = FFFF80007FF00000
bsf     rcx, r8   从右向左扫描字或双字操作数oprd2中第一个含"1"的位， 并把扫描到的第一个含'1'的位的位号送操作数rcx[0x14]
StartPosition=0x542&~0xF+rcx=0x554
VirtualAddr = (StartPosition + 8 \* (VmStruct->BitMap.Buffer - gVmBitMapBase))  \* 64K

7。VAD继续赋值
BaseAddress = VirtualAddr;
EndArress = mapParameter->ViewSize + VirtualAddr - 1;
StartVpn = BaseAddress >> 12;
EndVpn  = EndArress >> 12;
vad->Core.StartingVpnHigh = BaseAddress >> 0x2C;
vad->Core.EndingVpnHigh  =  EndArress >> 0x2C;
vad->Core.StartingVpn =StartVpn;
vad->Core.EndingVpn = EndVpn;
PteOffset += (ULONG)(Vad->EndingVpn - Vad->StartingVpn)
vad->LastContiguousPte = MiComputeContiguousSubsectionPte(SubSection, PteOffset);
if (ControlArea->FilePointer == NULL)
{
MiInsertSharedCommitNode(ControlArea, TargetProcess, a3)
}

8。MiInsertVadCharges(vad, TargetProcess);主要实现如下：
PsChargeProcessNonPagedPoolQuota
PspChargeQuota
RtlSetBitsEx(VmStruct, StartingIndex, NumberToSet);  //修改工作集里虚拟地址的bitmap

9。MiGetWsAndInsertVad(vad);
1.将VAD加入二叉树。
2.添加VAD参数。
vad->VadsProcess = (Process | 1);
vad->ViewLinks.Flink=&ControlArea->ListHead;
vad->ViewLinks.Blink=&ControlArea->ListHead;
ControlArea->ListHead.Flink=&vad->ViewLinks;
ControlArea->ListHead.Blink=&vad->ViewLinks;

10。VM工作集里的bitMap，需要将0x542+1
segment->FirstMappedVa=BaseAddress;
MiAdvanceVadHint(StartVpn, EndVpn, vmStruct); //VM工作集做一些设置。
```

![](https://bbs.kanxue.com/upload/attach/202104/814951_3EU68T5QZC83RK6.jpg)

完成

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)

[#系统底层](https://bbs.kanxue.com/forum-4-1-2.htm)
