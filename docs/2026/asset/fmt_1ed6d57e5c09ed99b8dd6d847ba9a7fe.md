
```
004017F8   .  50            PUSH EAX
004017F9   .  8BCE          MOV ECX,ESI
004017FB   .  E8 50050000   CALL CrackMe_.00401D50
00401DE8   .  53            PUSH EBX                                 ; /hTemplateFile => NULL
00401DE9   .  68 80000000   PUSH 0x80                                ; |Attributes = NORMAL
00401DEE   .  6A 03         PUSH 0x3                                 ; |Mode = OPEN_EXISTING
00401DF0   .  53            PUSH EBX                                 ; |pSecurity => NULL
00401DF1   .  53            PUSH EBX                                 ; |ShareMode => 0
00401DF2   .  68 000000C0   PUSH 0xC0000000                          ; |Access = GENERIC_READ|GENERIC_WRITE
00401DF7   .  68 58D34200   PUSH CrackMe_.0042D358                   ; |FileName = "\\.\vmxdrv"
00401DFC   .  FF15 88324200 CALL DWORD PTR DS:[<&KERNEL32.CreateFile>; \CreateFileA
00401E02   .  8BF8          MOV EDI,EAX
00401E04   .  83FF FF       CMP EDI,-0x1
00401E07   .  0F84 CE000000 JE CrackMe_.00401EDB
00401E0D   .  8D4424 1C     LEA EAX,DWORD PTR SS:[ESP+0x1C]
00401E11   .  53            PUSH EBX                                 ; /pOverlapped => NULL
00401E12   .  50            PUSH EAX                                 ; |pBytesReturned
00401E13   .  8D8C24 380200>LEA ECX,DWORD PTR SS:[ESP+0x238]         ; |
00401E1A   .  68 00010000   PUSH 0x100                               ; |OutBufferSize = 100 (256.)
00401E1F   .  51            PUSH ECX                                 ; |OutBuffer
00401E20   .  53            PUSH EBX                                 ; |InBufferSize => 0x0
00401E21   .  53            PUSH EBX                                 ; |InBuffer => NULL
00401E22      68 0C202200   PUSH 0x22200C
00401E27   .  57            PUSH EDI                                 ; |hDevice
00401E28   .  FF15 8C324200 CALL DWORD PTR DS:[<&KERNEL32.DeviceIoCo>; \DeviceIoControl
00401E2E   .  3BC3          CMP EAX,EBX
00401E30   .  0F84 A5000000 JE CrackMe_.00401EDB
00401E36   .  56            PUSH ESI
00401E37   .  8D5424 34     LEA EDX,DWORD PTR SS:[ESP+0x34]
00401E3B   .  55            PUSH EBP
00401E3C   .  52            PUSH EDX
00401E3D   .  E8 4E9B0000   CALL CrackMe_.0040B990
00401E42   .  83C4 0C       ADD ESP,0xC
00401E45   .  885C34 30     MOV BYTE PTR SS:[ESP+ESI+0x30],BL
00401E49   .  8D4424 18     LEA EAX,DWORD PTR SS:[ESP+0x18]
00401E4D   .  46            INC ESI
00401E4E   .  53            PUSH EBX                                 ; /pOverlapped
00401E4F   .  50            PUSH EAX                                 ; |pBytesWritten
00401E50   .  8D4C24 38     LEA ECX,DWORD PTR SS:[ESP+0x38]          ; |
00401E54   .  56            PUSH ESI                                 ; |nBytesToWrite
00401E55   .  51            PUSH ECX                                 ; |Buffer
00401E56   .  57            PUSH EDI                                 ; |hFile
00401E57   .  FF15 90324200 CALL DWORD PTR DS:[<&KERNEL32.WriteFile>>; \WriteFile
00401E5D   .  85C0          TEST EAX,EAX
00401E5F   .  74 73         JE XCrackMe_.00401ED4
00401E61   .  8D5424 14     LEA EDX,DWORD PTR SS:[ESP+0x14]
00401E65   .  53            PUSH EBX                                 ; /pOverlapped
00401E66   .  52            PUSH EDX                                 ; |pBytesRead
00401E67   .  8D8424 380100>LEA EAX,DWORD PTR SS:[ESP+0x138]         ; |
00401E6E   .  6A 10         PUSH 0x10                                ; |BytesToRead = 10 (16.)
00401E70   .  50            PUSH EAX                                 ; |Buffer
00401E71   .  57            PUSH EDI                                 ; |hFile
00401E72   .  FF15 94324200 CALL DWORD PTR DS:[<&KERNEL32.ReadFile>] ; \ReadFile
```

//离开核心算法程序，回到主流程

```
00401829   .  E8 F2000000   CALL CrackMe_.00401920                   ;  对驱动返回的HASH再进行1次MD5
0040182E   .  6A 0A         PUSH 0xA
00401830   .  8D4424 18     LEA EAX,DWORD PTR SS:[ESP+0x18]
00401834   .  6A 02         PUSH 0x2
00401836   .  50            PUSH EAX
00401837   .  8D4C24 1C     LEA ECX,DWORD PTR SS:[ESP+0x1C]
0040183B   .  E8 38420100   CALL CrackMe_.00415A78                   ;  将MD5结果从第3位开始截取10位
```

//将截取的10位字符与内置的`"888aeda4ab"`进行比较，相同则成功

```
00401876   .  68 6CD14200   PUSH CrackMe_.0042D16C                   ;  ASCII "888aeda4ab"
0040187B   .  51            PUSH ECX
0040187C   .  E8 309F0000   CALL CrackMe_.0040B7B1                   ;  关键比较
00401881   .  83C4 08       ADD ESP,0x8
00401884   .  85C0          TEST EAX,EAX
00401886   .  75 09         JNZ XCrackMe_.00401891                   ;  关键跳
```

## 五、小结RING3主程序算法流程：

```
1、输入字符：“123456”
2、转小写字符：“123456”
3、倒置字符：“654321”
4、驱动返回HASH：“17f73e13bad89038392692e82d63b17d” （类似MD5）
5、MD5(HASH)：“7c766e2a1ca057c7b0e91f935edaad73”
6、从第3位开始取10位：“766e2a1ca0”
7、与 “888aeda4ab” 比较
```

//显然只要再搞清楚第4步骤驱动里头在做什么就可以弄清楚算法了

## 六、驱动动态调试分析

//接下来该分析驱动，参考第三步静态分析的关键几个`MajorFunction`，开启`windbg`双机调试，断点：

```
bu vmxdrv+0x5A8 --> IRP_MJ_Read
bu vmxdrv+0x71A --> IRP_MJ_Device_Control
bu vmxdrv+0x61C --> IRP_MJ_Write
bu vmxdrv+0x685 --> Algorithm
```

//重新开始运行未修改过的主程序，断下`IRP_MJ_Device_Control`：

```
f8c0071a 8bff            mov     edi,edi
f8c0071c 55              push    ebp
f8c0071d 8bec            mov     ebp,esp
f8c0071f 51              push    ecx
f8c00720 a1c014c0f8      mov     eax,dword ptr [vmxdrv+0x14c0 (f8c014c0)]
f8c00725 33c5            xor     eax,ebp
f8c00727 8945fc          mov     dword ptr [ebp-4],eax
f8c0072a 56              push    esi
f8c0072b 8b750c          mov     esi,dword ptr [ebp+0Ch]
f8c0072e 8b4660          mov     eax,dword ptr [esi+60h]
f8c00731 8b400c          mov     eax,dword ptr [eax+0Ch]
f8c00734 2d04202200      sub     eax,222004h
```

//参考第三节程序，`push 0x222004`对应选择分支

//`IRP_MJ_Device_Control` 内核反调试及其他

```
int __stdcall sub_1071A(int a1, PIRP Irp)
{
  switch ( *(_DWORD *)(Irp->Tail.Overlay.PacketType + 12) )
  {
    case 0x222004:  //该分支定时并多次在流程中调用，包含内核反调试和暗桩
      dword_114D8 = 1;  //暗桩，必须为1，否则跟踪不到后续算法
      dword_114E0 = (int)IoGetCurrentProcess();
      sub_10486();  //DebugPort清零反调试
      break;
    case 0x222008:
      DbgPrint("%s\n", Irp->AssociatedIrp.IrpCount);
      break;
    case 0x22200C:
      DbgPrint("bye!\n");
      break;
    default:
      DbgPrint("unkonw code!\n");
      break;
  }
  Irp->IoStatus.Status = 0;
  Irp->IoStatus.Information = 0;
  IofCompleteRequest(Irp, 0);
  return 0;
}
```

```
f8c00774 ff158013c0f8    call    dword ptr [vmxdrv+0x1380 (f8c01380)]
nt!IoGetCurrentProcess:
804ef678 64a124010000    mov     eax,dword ptr fs:[00000124h] fs:0030:00000124=8211b4f0
804ef67e 8b4044          mov     eax,dword ptr [eax+44h]
eax=82148708
f8c0077a a3e014c0f8      mov     dword ptr [vmxdrv+0x14e0 (f8c014e0)],eax ds:0023:f8c014e0=00000000
f8c0077f e802fdffff      call    vmxdrv+0x486 (f8c00486)
```

//跟进`call vmxdrv+0x486 (f8c00486)`，`and dword ptr [eax+0BCh],0`对`DebugPort`清零，反调试

```
f8c00486 ff158013c0f8    call    dword ptr [vmxdrv+0x1380 (f8c01380)] ds:0023:f8c01380={nt!IoGetCurrentProcess (804ef678)}
f8c0048c 8b0de014c0f8    mov     ecx,dword ptr [vmxdrv+0x14e0 (f8c014e0)]
f8c00492 8bd0            mov     edx,eax
f8c00494 eb0f            jmp     vmxdrv+0x4a5 (f8c004a5)
f8c00496 8b8088000000    mov     eax,dword ptr [eax+88h]
f8c0049c 2d88000000      sub     eax,88h
f8c004a1 3bc2            cmp     eax,edx
f8c004a3 740b            je      vmxdrv+0x4b0 (f8c004b0)
f8c004a5 3bc1            cmp     eax,ecx
f8c004a7 75ed            jne     vmxdrv+0x496 (f8c00496)  //第一遍跟踪没有跳
f8c004a9 83a0bc00000000  and     dword ptr [eax+0BCh],0   //DebugPort清零，反调试
f8c004b0 c3              ret
```

//将`[f8c004a9]`的`83`修改为`C3`，跳过`DebugPort`清零反调试
