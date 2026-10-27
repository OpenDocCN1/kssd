
# 代码段

```c
//KdDebuggerEnabled变量存储
BOOLEAN gKdDebuggerEnabled=TRUE;

//KdPitchDebugger变量存储
BOOLEAN gKdPitchDebugger=FALSE;

//转移win7中的KdDebuggerEnabled和KdPitchDebugger变量
void MoveVariable_Win7(IN BOOL b***)
{
    ULONG ulAddr, ulAddr2;

    //----------------------------改写KeUpdateRunTime中的KdDebuggerEnabled

    //得到原内核的KeUpdateRunTime地址
    ulAddr=GetOriginalProcAddr(L"KeUpdateRunTime");

    //定位KdDebuggerEnabled
    //特征码
    //83ec9082 803d6c29fb8300  cmp     byte ptr [nt!KdDebuggerEnabled (83fb296c)],0
    //83ec9089 7412            je      nt!KeUpdateRunTime+0x164 (83ec909d)
    //83ec908b a1745ff883      mov     eax,dword ptr [nt!KiPollSlot (83f85f74)]
    UCHAR szSig1[10] = {0x80, 0x3d, '?', '?', '?', '?', '?', 0x74, 0x12, 0xa1};

    //特征码定位KdDebuggerEnabled
    ulAddr=SearchCode((PUCHAR)ulAddr, szSig1, 10, 0x200);
    ulAddr+=2;

    //改写KdDebuggerEnabled变量
    PageProtect(FALSE);
    *(PULONG)ulAddr=(ULONG)&gKdDebuggerEnabled;
    PageProtect(TRUE);

    //----------------------------改写KdCheckForDebugBreak中的KdDebuggerEnabled

    //得到KdCheckForDebugBreak地址
    //特征码
    //83ec9098 e80c000000      call    nt!KdCheckForDebugBreak (83ec90a9)
    //83ec909d 5f              pop     edi
    UCHAR szSig2[6] = {0xe8, '?', '?', '?', '?', 0x5f};
    ulAddr=SearchCode((PUCHAR)ulAddr, szSig2, 6, 0x100);
    ulAddr=ulAddr+*(PULONG)((PUCHAR)ulAddr+1)+5;

    //记录KdCheckForDebugBreak函数地址 用于KdPitchDebugger变量的处理
    ulAddr2=ulAddr;

    //特征码定位KdDebuggerEnabled
    //83ec90b2 803d6c29fb8300  cmp     byte ptr [nt!KdDebuggerEnabled (83fb296c)],0
    //83ec90b9 7410            je      nt!KdCheckForDebugBreak+0x22 (83ec90cb)
    //83ec90bb e81f000000      call    nt!KdPollBreakIn (83ec90df)
    UCHAR szSig3[10] = {0x80, 0x3d, '?', '?', '?', '?', '?', 0x74, 0x10, 0xe8};
    ulAddr=SearchCode((PUCHAR)ulAddr, szSig3, 10, 0x100);
    ulAddr+=2;

    //改写KdDebuggerEnabled变量
    PageProtect(FALSE);
    *(PULONG)ulAddr=(ULONG)&gKdDebuggerEnabled;
    PageProtect(TRUE);

    //----------------------------改写KdCheckForDebugBreak中的KdPitchDebugger

    //定位KdPitchDebugger变量
    //由于KdPitchDebugger变量在函数开头偏移2处 所以直接偏移定位KdPitchDebugger
    //83eb30a9 803d270df68300  cmp     byte ptr [nt!KdPitchDebugger (83f60d27)],0
    //83eb30b0 7519            jne     nt!KdCheckForDebugBreak+0x22 (83eb30cb)
    ulAddr2+=2;

    //改写KdPitchDebugger变量
    PageProtect(FALSE);
    *(PULONG)ulAddr2=(ULONG)&gKdPitchDebugger;
    PageProtect(TRUE);

    //----------------------------改写KdPollBreakIn中的KdDebuggerEnabled

    //得到KdPollBreakIn地址
    //特征码
    //83ebe0bb e81f000000      call    nt!KdPollBreakIn (83ebe0df)
    //83ebe0c0 84c0            test    al,al
    UCHAR szSig4[7] = {0xe8, '?', '?', '?', '?', 0x84, 0xc0};
    ulAddr=SearchCode((PUCHAR)ulAddr, szSig4, 7, 0x100);
    ulAddr=ulAddr+*(PULONG)((PUCHAR)ulAddr+1)+5;

    //记录KdPollBreakIn函数地址 用于KdPitchDebugger变量的处理
    ulAddr2=ulAddr;

    //特征码定位KdDebuggerEnabled
    //83ebe0f7 885dff          mov     byte ptr [ebp-1],bl
    //83ebe0fa 381d6c79fa83    cmp     byte ptr [nt!KdDebuggerEnabled (83fa796c)],bl
    //83ebe100 0f84c0000000    je      nt!KdPollBreakIn+0xe7 (83ebe1c6)
    UCHAR szSig5[11] = {0x88, 0x5d, 0xff, 0x38, 0x1d, '?', '?', '?', '?', 0x0f, 0x84};
    ulAddr=SearchCode((PUCHAR)ulAddr, szSig5, 11, 0x100);
    ulAddr+=5;

    //改写KdDebuggerEnabled变量
    PageProtect(FALSE);
    *(PULONG)ulAddr=(ULONG)&gKdDebuggerEnabled;
    PageProtect(TRUE);

    //----------------------------改写KdPollBreakIn中的KdPitchDebugger

    //特征码定位KdPitchDebugger
    //83eb30e6 33db            xor     ebx,ebx
    //83eb30e8 381d270df683    cmp     byte ptr [nt!KdPitchDebugger (83f60d27)],bl
    //83eb30ee 7407            je      nt!KdPollBreakIn+0x18 (83eb30f7)
    UCHAR szSig6[10] = {0x33, 0xdb, 0x38, 0x1d, '?', '?', '?', '?', 0x74, 0x07};
    ulAddr2=SearchCode((PUCHAR)ulAddr2, szSig6, 10, 0x100);
    ulAddr2+=4;

    //改写KdPitchDebugger变量
    PageProtect(FALSE);
    *(PULONG)ulAddr2=(ULONG)&gKdPitchDebugger;
    PageProtect(TRUE);

}
```

经过以上转移以后 再次清零 `KdDebuggerEnabled` 变量后,也可以 `ctrl+break` 断点到 `windbg` 中,第一块问题就基本解决了

## 第二块

正常情况下观察 `KiDebugRoutine`

```
1: kd> dd KiDebugRoutine
83fa2b20  841664f2 83f16683 00000000 0311870a
83fa2b30  00000bb8 00000011 5385d2ba d717548f
83fa2b40  83eb7d5c 00000000 00000191 83eb83a4
83fa2b50  986fa000 00000000 00000339 986fb02c
83fa2b60  00000100 00000000 00000000 83fa2b68
83fa2b70  00000340 00000340 00000007 00000000
83fa2b80  86cdb6b8 86cdb5f0 86ce3838 86ce39c8
83fa2b90  86ce3900 00000000 86d37040 00000000
1: kd> u 841664f2
nt!KdpTrap:
841664f2 8bff            mov     edi,edi
841664f4 55              push    ebp
841664f5 8bec            mov     ebp,esp
841664f7 51              push    ecx
841664f8 51              push    ecx
841664f9 8b4510          mov     eax,dword ptr [ebp+10h]
841664fc 33d2            xor     edx,edx
841664fe 813803000080    cmp     dword ptr [eax],80000003h
```

将其设置为 `KdpStub` 函数

```
1: kd> ed KiDebugRoutine KdpStub
1: kd> dd KiDebugRoutine
83fa2b20  83f16983 83f16683 00000000 0311870a
83fa2b30  00000bb8 00000011 5385d2ba d717548f
83fa2b40  83eb7d5c 00000000 00000191 83eb83a4
83fa2b50  986fa000 00000000 00000339 986fb02c
83fa2b60  00000100 00000000 00000000 83fa2b68
83fa2b70  00000340 00000340 00000007 00000000
83fa2b80  86cdb6b8 86cdb5f0 86ce3838 86ce39c8
83fa2b90  86ce3900 00000000 86d37040 00000000
1: kd> u 83f16983
nt!KdpStub:
83f16983 8bff            mov     edi,edi
83f16985 55              push    ebp
83f16986 8bec            mov     ebp,esp
83f16988 53              push    ebx
83f16989 56              push    esi
83f1698a 8b7510          mov     esi,dword ptr [ebp+10h]
83f1698d 33db            xor     ebx,ebx
83f1698f 813e03000080    cmp     dword ptr [esi],80000003h
```

`Windbg` `F5` 运行起来,接着 `ctrl+break` 发现 `VM` 死机了,看来这个影响还蛮大的,重启后 `IDA` 打开内核搜索全部 `KiDebugRoutine`,发现调用它的代码全部都在 `KiDispatchException` 中,反汇编 `KiDispatchException` 搜索 `KiDebugRoutine` 相关代码,过程和第一块一样,发现以下三个结果:

```
83f04027 ff15208bfb83    call    dword ptr [nt!KiDebugRoutine (83fb8b20)]
83f0404f ff15208bfb83    call    dword ptr [nt!KiDebugRoutine (83fb8b20)]
83f040c8 ff15208bfb83    call    dword ptr [nt!KiDebugRoutine (83fb8b20)]
```

看来先前修改了 `KiDebugRoutine`,造成的死机就是在 `KiDispatchException` 中出现的问题,现在编程解决问题,思路和上一块一样,转移 `KiDebugRoutine` 变量,部分代码如下:

```c
//KiDebugRoutine变量存储
ULONG gKiDebugRoutine=0;

//转移win7中的KiDebugRoutine变量
void MoveKiDebugRoutine_Win7(IN BOOL b***)
{
    //得到KdpTrap地址
    ULONG ulAddr=GetOriginalProcAddr(L"KdpTrap");

    //设置自定义的KiDebugRoutine变量
    gKiDebugRoutine=ulAddr;

    //得到原内核KiDispatchException地址
    ulAddr=GetOriginalProcAddr(L"KiDispatchException");

    //---------------------------------------第一处KiDebugRoutine
    //特征码定位KiDebugRoutine
    //83eb5027 ff15209bf683    call    dword ptr [nt!KiDebugRoutine (83f69b20)]
    //83eb502d 84c0            test    al,al
    UCHAR szSig[8] = {0xff, 0x15, '?', '?', '?', '?', 0x84, 0xc0};
    ulAddr=SearchCode((PUCHAR)ulAddr, szSig, 8, 0x200);
    if(!ulAddr)
    {
      KdPrint(("\n[MoveKiDebugRoutine_Win7]: The signature is not found in KiDispatchException"));
      return;
    }
    ulAddr+=2;

    //改写KiDebugRoutine变量
    PageProtect(FALSE);
    *(PULONG)ulAddr=(ULONG)&gKiDebugRoutine;
    PageProtect(TRUE);

    //---------------------------------------第二处KiDebugRoutine
    //特征码与前边一样
    ulAddr=SearchCode((PUCHAR)ulAddr, szSig, 8, 0x200);
    ulAddr+=2;
    //改写KiDebugRoutine变量
    PageProtect(FALSE);
    *(PULONG)ulAddr=(ULONG)&gKiDebugRoutine;
    PageProtect(TRUE);

    //---------------------------------------第三处KiDebugRoutine
    //特征码与前边一样
    ulAddr=SearchCode((PUCHAR)ulAddr, szSig, 8, 0x200);
    ulAddr+=2;

    //改写KiDebugRoutine变量
    PageProtect(FALSE);
    *(PULONG)ulAddr=(ULONG)&gKiDebugRoutine;
    PageProtect(TRUE);
}
```

这样一来第二块问题也基本解决了,这个和第一块原理一样,接下来就是第三块了

## 第三块

反汇编 `KdSendPacket`
