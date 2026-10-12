
`600251B0` `76D36051` `IPHLPAPI.GetAdaptersInfo`

`600251B4` `00000000`

### 第四步：DUMP 及修复重定位

用`LordPE`选中`Ollydbg`的`loaddll.exe`的进程，在下面的列表里选择`pcomm.dll`，然后完整脱壳，得到`dumped.dll`。

运行`ImportREC`，（注意：去掉“使用来自磁盘的 PE 部首”的选项）

选中`Ollydbg`的`loaddll.exe`的进程，然后点“选取 DLL”，选择`pcomm.dll`，填入`RVA=00025000`、`大小=000001B4`，点“Get Import”，得到输入表。改`OEP=0000FC55`，`FixDump`！

根据第一步得到的重定位表地址，用`WinHex`打开`dumped\_.dll`，复制`00051000`－`00051E4B`之间的 16 进制数值，另存为`1.bin`

运行看雪老师写的辅助修复 UPX 加壳 DLL 重定位表的工具`UPXAngela.exe`，打开`1.bin`，很快的提示`pediy.bin`文件创建成功！

（这里要告诉大家一下，我为什么选择看雪老师写的辅助修复 UPX 加壳 DLL 重定位表的工具`UPXAngela.exe` 呢，因为原来学习了 FLY 大侠写的这篇《用 Ollydbg 手脱 UPX 加壳的 DLL 》之后才用此工具）

============================================

大家可以对照一下：

这个是 UPX 加壳的 DLL

```
003B825E     8A07             mov al,byte ptr ds:[edi]//EDI=003B7318-当前基址 003B0000＝00007318  ★ 这就是重定位表的 RVA

003B8260     47              inc edi

003B8261     09C0            or eax,eax

003B8263     74 22           je short EdrLib.003B8287//重定位数据处理完毕则跳转

003B8265     3C EF           cmp al,0EF

003B8267     77 11           ja short EdrLib.003B827A

003B8269     01C3            add ebx,eax

003B826B     8B03            mov eax,dword ptr ds:[ebx]

003B826D     86C4            xchg ah,al//找到这里

003B826F     C1C0 10         rol eax,10

003B8272     86C4            xchg ah,al

003B8274     01F0            add eax,esi

003B8276     8903            mov dword ptr ds:[ebx],eax

003B8278     EB E2           jmp short EdrLib.003B825C

003B827A     24 0F           and al,0F

003B827C     C1E0 10         shl eax,10

003B827F     66:8B07         mov ax,word ptr ds:[edi]

003B8282     83C7 02         add edi,2

003B8285     EB E2           jmp short EdrLib.003B8269
```

这个是 NsPacK V3.7 加壳的 DLL

```
600553CA    8A07            MOV AL,BYTE PTR DS:[EDI]           ; EDI=60051000-当前基址 60000000＝00051000 , 这就是重定位表的 RVA 了

600553CC    47             INC EDI

600553CD    0BC0           OR EAX,EAX

600553CF    74 20          JE SHORT pcomm.600553F1             ; 重定位数据处理完，则跳转

600553D1    3C EF          CMP AL,0EF

600553D3    77 06          JA SHORT pcomm.600553DB

600553D5    03D8           ADD EBX,EAX

600553D7    010B           ADD DWORD PTR DS:[EBX],ECX

600553D9   ^ EB ED         JMP SHORT pcomm.600553C8

600553DB    24 0F          AND AL,0F

600553DD    C1E0 10        SHL EAX,10

600553E0    66:8B07        MOV AX,WORD PTR DS:[EDI]

600553E3    83C7 02        ADD EDI,2

600553E6    0BC0           OR EAX,EAX

600553E8   ^ 75 EB         JNZ SHORT pcomm.600553D5

600553EA    8B07           MOV EAX,DWORD PTR DS:[EDI]

600553EC    83C7 04        ADD EDI,4

600553EF   ^ EB E4         JMP SHORT pcomm.600553D5

600553F1    33DB           XOR EBX,EBX                          ; EDI=60051E4B,也就是重定位表的结束地址
```

============================================

我们就从`dumped\_.dll`中的`00051000`处开始。

用`WinHex`把`pediy.bin`（注：`pediy.bin`中的数据长度为`1C22` 比原先的数据长度要长，这就要注意写入时不能覆盖了有用的数据哟）中的 16 进制数值全部复制、写入到`dumped\_.dll`的`00051000`处。

用`LordPE`修正`dumped\_.dll`的重定位表`RVA=00051000`、`大小＝00001C22`，并保存。

用`OD`，及`Dll_LoadEx`载入都很正常。

脱壳完成。

--------------------------------------------------------------------------------
【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

2007 年 03 月 12 日 13:43:37
