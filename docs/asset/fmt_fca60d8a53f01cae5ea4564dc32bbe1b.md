
```
01080087    B9 08003F00     mov     ecx, 3F0008                   ; 上一步得到的 IAT 表的起始地址
 0108008C    3901            cmp     dword ptr [ecx], eax
 0108008E    74 05           je      short 01080095
 01080090    83C1 04        add     ecx, 4
 01080093  ^ EB F7          jmp     short 0108008C
 01080095    8BD1            mov     edx, ecx
 01080097    81C2 00100000   add     edx, 1000
 0108009D    8902            mov     dword ptr [edx], eax
 0108009F    83C4 1C         add     esp, 1C
 010800A2    3E:8B0424       mov     eax, dword ptr ds:[esp]
 010800A6    83E8 06         sub     eax, 6
 010800A9    66:C700 FF25    mov     word ptr [eax], 25FF          ; 更改成 Jmp 类型
 010800AE    83C0 02         add     eax, 2
 010800B1    8910            mov     dword ptr [eax], edx
 010800B3    90              nop
 010800B4    90              nop
 010800B5    813D 04003F00 1cmp     dword ptr [3F0004], 408218      ; OEP，作为搜索的结束地址
 010800BF    74 2C           je      short 010800ED
 010800C1    8B0D 04003F00   mov     ecx, dword ptr [3F0004]        ; xpsmoker.00401000
 010800C7    66:8139 FF15    cmp     word ptr [ecx], 15FF           ; 对比是否是要修改的 Call
 010800CC    75 17           jnz     short 010800E5
 010800CE    8BC1            mov     eax, ecx                       ; 是则
 010800D0    83C0 02         add     eax, 2
 010800D3    8138 80224E00   cmp     dword ptr [eax], 4E2280        ; 比较 Call 的 Addr 是不是 004E2280
 010800D9    75 0A           jnz     short 010800E5
 010800DB    83C0 04         add     eax, 4                          ; 是则
 010800DE    A3 04003F00     mov     dword ptr [3F0004], eax        ; 赋予真实的 IAT 地址
 010800E3    FFE1            jmp     ecx                             ; 跳回下一个地址继续
 010800E5    FF05 04003F00   inc     dword ptr [3F0004]             ; xpsmoker. 00401000
 010800EB  ^ EB C8          jmp     short 010800B5
 010800ED  - EB FE          jmp     short 010800ED                  ; F2，Shift+F9 后,全部还原
 010800EF    90              nop
 010800F0    90              nop
 010800F1    90              nop
 010800F2    90              nop
```

????????????????????????????

取消断点，在 `OEP` 处新建 `EIP`，看看现在的 `IAT` 表

```
00407D50   $- FF25 58103F00 jmp     dword ptr [3F1058]            ;  MSVBVM60.__vbaChkstk
00407D56   $- FF25 74103F00 jmp     dword ptr [3F1074]            ;  MSVBVM60.__vbaExceptHandler; 异常处理程序
00407D5C   >- FF25 7C103F00 jmp     dword ptr [3F107C]            ;  MSVBVM60.__vbaFPException
00407D62   .- FF25 20123F00 jmp     dword ptr [3F1220]            ;  MSVBVM60._adj_fdiv_m16i
00407D68   $- FF25 24123F00 jmp     dword ptr [3F1224]            ;  MSVBVM60._adj_fdiv_m32
00407D6E   .- FF25 28123F00 jmp     dword ptr [3F1228]            ;  MSVBVM60._adj_fdiv_m32i
00407D74   $- FF25 2C123F00 jmp     dword ptr [3F122C]            ;  MSVBVM60._adj_fdiv_m64
00407D7A   .- FF25 30123F00 jmp     dword ptr [3F1230]            ;  MSVBVM60._adj_fdiv_r
00407D80   .- FF25 34123F00 jmp     dword ptr [3F1234]            ;  MSVBVM60._adj_fdivr_m16i
00407D86   .- FF25 38123F00 jmp     dword ptr [3F1238]            ;  MSVBVM60._adj_fdivr_m32
00407D8C   .- FF25 3C123F00 jmp     dword ptr [3F123C]            ;  MSVBVM60._adj_fdivr_m32i
00407D92   .- FF25 40123F00 jmp     dword ptr [3F1240]            ;  MSVBVM60._adj_fdivr_m64
00407D98   .- FF25 44123F00 jmp     dword ptr [3F1244]            ;  MSVBVM60._adj_fpatan
00407D9E   .- FF25 48123F00 jmp     dword ptr [3F1248]            ;  MSVBVM60._adj_fprem
00407DA4   .- FF25 4C123F00 jmp     dword ptr [3F124C]            ;  MSVBVM60._adj_fprem1
00407DAA   .- FF25 50123F00 jmp     dword ptr [3F1250]            ;  MSVBVM60._adj_fptan
00407DB0   .- FF25 18103F00 jmp     dword ptr [3F1018]            ;  MSVBVM60._CIatan
00407DB6   .- FF25 1C103F00 jmp     dword ptr [3F101C]            ;  MSVBVM60._CIcos
00407DBC   .- FF25 20103F00 jmp     dword ptr [3F1020]            ;  MSVBVM60._CIexp
00407DC2   .- FF25 24103F00 jmp     dword ptr [3F1024]            ;  MSVBVM60._CIlog
00407DC8   .- FF25 28103F00 jmp     dword ptr [3F1028]            ;  MSVBVM60._CIsin
00407DCE   .- FF25 2C103F00 jmp     dword ptr [3F102C]            ;  MSVBVM60._CIsqrt
00407DD4   .- FF25 30103F00 jmp     dword ptr [3F1030]            ;  MSVBVM60._CItan
00407DDA   .- FF25 54123F00 jmp     dword ptr [3F1254]            ;  MSVBVM60._allmul
00407DE0   .- FF25 08103F00 jmp     dword ptr [3F1008]            ;  MSVBVM60.DllFunctionCall
00407DE6   $- FF25 DC113F00 jmp     dword ptr [3F11DC]            ;  MSVBVM60.__vbaVarLateMemSt
00407DEC   $- FF25 3C113F00 jmp     dword ptr [3F113C]            ;  MSVBVM60.__vbaR8Str
00407DF2   $- FF25 88113F00 jmp     dword ptr [3F1188]            ;  MSVBVM60.__vbaStrR8
00407DF8   $- FF25 B4113F00 jmp     dword ptr [3F11B4]            ;  MSVBVM60.__vbaVarCmpGt
00407DFE   $- FF25 30133F00 jmp     dword ptr [3F1330]            ;  MSVBVM60.rtcRound
00407E04   $- FF25 98113F00 jmp     dword ptr [3F1198]            ;  MSVBVM60.__vbaStrVarCopy
00407E0A   $- FF25 04113F00 jmp     dword ptr [3F1104]            ;  MSVBVM60.__vbaLenVar
00407E10   $- FF25 6C103F00 jmp     dword ptr [3F106C]            ;  MSVBVM60.__vbaEnd
00407E16   $- FF25 A4103F00 jmp     dword ptr [3F10A4]            ;  MSVBVM60.__vbaFpI4
00407E1C   $- FF25 38113F00 jmp     dword ptr [3F1138]            ;  MSVBVM60.__vbaR4Var
00407E22   $- FF25 F0103F00 jmp     dword ptr [3F10F0]            ;  MSVBVM60.__vbaLateIdCallSt
00407E28   $- FF25 F8123F00 jmp     dword ptr [3F12F8]            ;  MSVBVM60.rtcInputCharCountVar
00407E2E   $- FF25 B4123F00 jmp     dword ptr [3F12B4]            ;  MSVBVM60.rtcFileLength
00407E34   $- FF25 74113F00 jmp     dword ptr [3F1174]            ;  MSVBVM60.__vbaStrErrVarCopy
00407E3A   $- FF25 7C113F00 jmp     dword ptr [3F117C]            ;  MSVBVM60.__vbaStrI2
00407E40   $- FF25 9C123F00 jmp     dword ptr [3F129C]            ;  MSVBVM60.rtcKillFiles
00407E46   $- FF25 E4103F00 jmp     dword ptr [3F10E4]            ;  MSVBVM60.__vbaInputFile
00407E4C   $- FF25 B8123F00 jmp     dword ptr [3F12B8]            ;  MSVBVM60.rtcEndOfFile
00407E52   $- FF25 84103F00 jmp     dword ptr [3F1084]            ;  MSVBVM60.__vbaFileClose
00407E58   $- FF25 34113F00 jmp     dword ptr [3F1134]            ;  MSVBVM60.__vbaPrintFile
00407E5E   $- FF25 88103F00 jmp     dword ptr [3F1088]            ;  MSVBVM60.__vbaFileOpen
00407E64   $- FF25 4C103F00 jmp     dword ptr [3F104C]            ;  MSVBVM60.__vbaCastObj
00407E6A   $- FF25 C4123F00 jmp     dword ptr [3F12C4]            ;  MSVBVM60.rtcSetFileAttr
00407E70   $- FF25 C0123F00 jmp     dword ptr [3F12C0]            ;  MSVBVM60.rtcGetFileAttr
00407E76   $- FF25 1C133F00 jmp     dword ptr [3F131C]            ;  MSVBVM60.rtcEnvironVar
00407E7C   $- FF25 88123F00 jmp     dword ptr [3F1288]            ;  MSVBVM60.rtcLowerCaseVar
00407E82   $- FF25 04133F00 jmp     dword ptr [3F1304]            ;  MSVBVM60.rtcMidCharVar
00407E88   $- FF25 E4123F00 jmp     dword ptr [3F12E4]            ;  MSVBVM60.rtcVarBstrFromAnsi
00407E8E   $- FF25 8C123F00 jmp     dword ptr [3F128C]            ;  MSVBVM60.rtcTrimVar
00407E94   $- FF25 94123F00 jmp     dword ptr [3F1294]            ;  MSVBVM60.rtcSpaceVar
00407E9A   $- FF25 FC123F00 jmp     dword ptr [3F12FC]            ;  MSVBVM60.rtcGetObject
```
