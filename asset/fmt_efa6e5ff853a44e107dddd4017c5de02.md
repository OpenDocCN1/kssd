
```
0050700C  55                 PUSH EBP   
0050700D  C3                 RETN   
0050700E  E8 01000000       CALL JETCAR.00507014   
00507013  EB 5D             JMP SHORT JETCAR.00507072   
00507015  BB EDFFFFFF       MOV EBX,-13   
0050701A  03DD              ADD EBX,EBP   
0050701C  81EB 00701000    SUB EBX,107000   
00507022  83BD 22040000 00 CMP DWORD PTR SS:[EBP+422],0   
00507029  899D 22040000    MOV DWORD PTR SS:[EBP+422],EBX   
```

计算 `MOV` 指令地址

`507001+399=50739A`

到指令 `50739A` 看看

```
0050738F  8B95 22040000    MOV EDX,DWORD PTR SS:[EBP+422]   
00507395  E9 EBfeFFFF      JMP JETCAR.00507285   
0050739A  B8 056D0700      MOV EAX,76D05<==================这儿就是 OEP 的 VA 地址   
0050739F  50               PUSH EAX   
005073A0  0385 22040000   ADD EAX,DWORD PTR SS:[EBP+422]==========>ImageBase   
005073A6  59               POP ECX   
005073A7  0BC9             OR ECX,ECX   
005073A9  8985 A8030000   MOV DWORD PTR SS:[EBP+3A8],EAX   
005073AF  61               POPAD   
```

计算 OEP

`OEP=ImageBase+[EP+39A]`

`=400000+[507001+39A]`

`=400000+76D05`

`=476D05`

==========================================================================

==========================================================================

正确。再来使用 `TRW2000` 脱一个看看如何，使用 `Aspack` 压缩 `UltraEdit`

```
004D3001 >60              PUSHAD   
004D3002  E8 03000000     CALL UEDIT32.004D300A   
004D3007  E9 EB045D45     JMP 45AA34F7   
004D300C  55              PUSH EBP   
004D339A  B8 D0850400     MOV EAX,485D0   
004D339F  50              PUSH EAX   
```

```
:u 4d339A   

:bpx 4485d0   

:g   

:bc *   

:pedump c:\mm.exe   
```

`ok`，`mm.exe` 脱壳完成

==========================================================================

`ASPack` 的壳，不是自身，采用了 `SEH` 等技术，不能用此方法脱壳

```
========================================================================= 

=                inside Pandora's Box - iPB 

=                Open Cracking Group  - OCG 

= 

=                            DiKeN/iPB 

========================================================================= 
```

比如 `NetCaptor 7.0`

加壳后的入口为 `5AC001`。在 `SoftICE` 中在 `5AC001` 处停下，然后不停向下翻屏就可以看到进入 OEP 的指令。这样写 `ProcDump` 的脚本也很方便。

```
001B:005AC3A9  MOV       [EBP+000003A8],EAX    //计算出来的 OEP   
001B:005AC3AF  POPAD   
001B:005AC3B0  JNZ       005AC3BA   
001B:005AC3B2  MOV       EAX,00000001   
001B:005AC3B7  RET       000C   
001B:005AC3BA  PUSH      00000000             //计算出来的 OEP 将覆盖这个值    
001B:005AC3BF  RET  4dd466d0e1e0cc123afbe504da2462ca
```
