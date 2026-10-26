
```
001D3A22  /$  55            PUSH EBP

001D3A23  |.  8BEC          MOV EBP,ESP

001D3A25  |.  6A FF         PUSH -0x1

001D3A27  |.  68 29983000   PUSH WnSkinIn.00309829

001D3A2C  |.  64:A1 0000000>MOV EAX,DWORD PTR FS:[0]

001D3A32  |.  50            PUSH EAX

001D3A33  |.  B8 34180000   MOV EAX,0x1834

001D3A38  |.  E8 73540000   CALL WnSkinIn.001D8EB0

001D3A3D  |.  A1 E4903600   MOV EAX,DWORD PTR DS:[0x3690E4]

001D3A42  |.  33C5          XOR EAX,EBP

001D3A44  |.  8945 F0       MOV DWORD PTR SS:[EBP-0x10],EAX

001D3A47  |.  53            PUSH EBX

001D3A48  |.  56            PUSH ESI

001D3A49  |.  57            PUSH EDI

001D3A4A  |.  50            PUSH EAX

001D3A4B  |.  8D45 F4       LEA EAX,DWORD PTR SS:[EBP-0xC]

001D3A4E  |.  64:A3 0000000>MOV DWORD PTR FS:[0],EAX

001D3A54  |.  8BF9          MOV EDI,ECX

001D3A56  |.  8B75 08       MOV ESI,DWORD PTR SS:[EBP+0x8]

001D3A59  |.  8D85 F8EDFFFF LEA EAX,DWORD PTR SS:[EBP-0x1208]

001D3A5F  |.  68 E8030000   PUSH 0x3E8

001D3A64  |.  33DB          XOR EBX,EBX

001D3A66  |.  53            PUSH EBX

001D3A67  |.  50            PUSH EAX

001D3A68  |.  E8 83E71000   CALL WnSkinIn.002E21F0

001D3A6D  |.  68 08020000   PUSH 0x208

001D3A72  |.  8D85 C8F5FFFF LEA EAX,DWORD PTR SS:[EBP-0xA38]

001D3A78  |.  53            PUSH EBX

001D3A79  |.  50            PUSH EAX

001D3A7A  |.  E8 71E71000   CALL WnSkinIn.002E21F0

001D3A7F  |.  68 08020000   PUSH 0x208

001D3A84  |.  8D85 E8FDFFFF LEA EAX,DWORD PTR SS:[EBP-0x218]

001D3A8A  |.  53            PUSH EBX

001D3A8B  |.  50            PUSH EAX

001D3A8C  |.  E8 5FE71000   CALL WnSkinIn.002E21F0

001D3A91  |.  68 08020000   PUSH 0x208

001D3A96  |.  8D85 D0F7FFFF LEA EAX,DWORD PTR SS:[EBP-0x830]

001D3A9C  |.  53            PUSH EBX

001D3A9D  |.  50            PUSH EAX

001D3A9E  |.  E8 4DE71000   CALL WnSkinIn.002E21F0

001D3AA3  |.  68 08020000   PUSH 0x208

001D3AA8  |.  8D85 D8F9FFFF LEA EAX,DWORD PTR SS:[EBP-0x628]

001D3AAE  |.  53            PUSH EBX

001D3AAF  |.  50            PUSH EAX

001D3AB0  |.  E8 3BE71000   CALL WnSkinIn.002E21F0

001D3AB5  |.  83C4 3C       ADD ESP,0x3C

001D3AB8  |.  837E 14 08    CMP DWORD PTR DS:[ESI+0x14],0x8

001D3ABC  |.  72 02         JB SHORT WnSkinIn.001D3AC0

001D3ABE  |.  8B36          MOV ESI,DWORD PTR DS:[ESI]

001D3AC0  |>  68 98C83000   PUSH WnSkinIn.0030C898                                 ;  -install

001D3AC5  |.  E8 B0301100   CALL WnSkinIn.002E6B7A

001D3ACA  |.  40            INC EAX

001D3ACB  |.  8D0446        LEA EAX,DWORD PTR DS:[ESI+EAX*2]

001D3ACE  |.  50            PUSH EAX

001D3ACF  |.  8D85 F8EDFFFF LEA EAX,DWORD PTR SS:[EBP-0x1208]

001D3AD5  |.  50            PUSH EAX

001D3AD6  |.  E8 4C301100   CALL WnSkinIn.002E6B27

001D3ADB  |.  8D85 F8EDFFFF LEA EAX,DWORD PTR SS:[EBP-0x1208]

001D3AE1  |.  50            PUSH EAX

001D3AE2  |.  8D85 C8E7FFFF LEA EAX,DWORD PTR SS:[EBP-0x1838]

001D3AE8  |.  50            PUSH EAX

001D3AE9  |.  E8 022BFFFF   CALL WnSkinIn.001C65F0

001D3AEE  |.  83BD DCE7FFFF>CMP DWORD PTR SS:[EBP-0x1824],0x8

001D3AF5  |.  8D85 C8E7FFFF LEA EAX,DWORD PTR SS:[EBP-0x1838]

001D3AFB  |.  895D FC       MOV DWORD PTR SS:[EBP-0x4],EBX

001D3AFE  |.  0F4385 C8E7FF>CMOVNB EAX,DWORD PTR SS:[EBP-0x1838]

001D3B05  |.  50            PUSH EAX

001D3B06  |.  8D85 E8FDFFFF LEA EAX,DWORD PTR SS:[EBP-0x218]

001D3B0C  |.  50            PUSH EAX

001D3B0D  |.  E8 46301100   CALL WnSkinIn.002E6B58

001D3B12  |.  BE F4010000   MOV ESI,0x1F4

001D3B17  |.  8D85 D4F3FFFF LEA EAX,DWORD PTR SS:[EBP-0xC2C]

001D3B1D  |.  56            PUSH ESI

001D3B1E  |.  53            PUSH EBX

001D3B1F  |.  50            PUSH EAX

001D3B20  |.  E8 CBE61000   CALL WnSkinIn.002E21F0

001D3B25  |.  56            PUSH ESI

001D3B26  |.  8D85 E0F1FFFF LEA EAX,DWORD PTR SS:[EBP-0xE20]

001D3B2C  |.  53            PUSH EBX

001D3B2D  |.  50            PUSH EAX

001D3B2E  |.  E8 BDE61000   CALL WnSkinIn.002E21F0

001D3B33  |.  8D85 D4F3FFFF LEA EAX,DWORD PTR SS:[EBP-0xC2C]

001D3B39  |.  50            PUSH EAX

001D3B3A  |.  8D85 E8FDFFFF LEA EAX,DWORD PTR SS:[EBP-0x218]

001D3B40  |.  50            PUSH EAX

001D3B41  |.  E8 3037FFFF   CALL WnSkinIn.001C7276

001D3B46  |.  8D85 E0F1FFFF LEA EAX,DWORD PTR SS:[EBP-0xE20]

001D3B4C  |.  50            PUSH EAX

001D3B4D  |.  8D85 D4F3FFFF LEA EAX,DWORD PTR SS:[EBP-0xC2C]

001D3B53  |.  50            PUSH EAX

001D3B54  |.  E8 6338FFFF   CALL WnSkinIn.001C73BC

001D3B59  |.  83C4 44       ADD ESP,0x44

001D3B5C  |.  8D85 E8FDFFFF LEA EAX,DWORD PTR SS:[EBP-0x218]

001D3B62  |.  50            PUSH EAX

001D3B63  |.  8D85 E0F1FFFF LEA EAX,DWORD PTR SS:[EBP-0xE20]

001D3B69  |.  50            PUSH EAX

001D3B6A  |.  E8 2121FFFF   CALL WnSkinIn.001C5C90

001D3B6F  |.  59            POP ECX

001D3B70  |.  59            POP ECX

001D3B71  |.  E8 43A9FFFF   CALL WnSkinIn.001CE4B9

001D3B76  |.  8BC8          MOV ECX,EAX

001D3B78  |.  E8 0D07FFFF   CALL WnSkinIn.001C428A

001D3B7D  |.  50            PUSH EAX                                               ; /Arg3

001D3B7E  |.  8D85 D8F9FFFF LEA EAX,DWORD PTR SS:[EBP-0x628]                       ; |

001D3B84  |.  68 FCBD3000   PUSH WnSkinIn.0030BDFC                                 ; |%sAllSkin\

001D3B89  |.  50            PUSH EAX                                               ; |Arg1

001D3B8A  |.  E8 1001FFFF   CALL WnSkinIn.001C3C9F                                 ; \WnSkinIn.001C3C9F

001D3B8F  |.  83C4 0C       ADD ESP,0xC

001D3B92  |.  8D85 D8F9FFFF LEA EAX,DWORD PTR SS:[EBP-0x628]

001D3B98  |.  50            PUSH EAX                                               ; /Path

001D3B99  |.  FF15 F0A33000 CALL DWORD PTR DS:[<&SHLWAPI.PathFileExistsW>]         ; \PathFileExistsW

001D3B9F  |.  85C0          TEST EAX,EAX

001D3BA1  |.  75 0E         JNZ SHORT WnSkinIn.001D3BB1

001D3BA3  |.  53            PUSH EBX                                               ; /pSecurity

001D3BA4  |.  8D85 D8F9FFFF LEA EAX,DWORD PTR SS:[EBP-0x628]                       ; |

001D3BAA  |.  50            PUSH EAX                                               ; |Path

001D3BAB  |.  FF15 18A33000 CALL DWORD PTR DS:[<&KERNEL32.CreateDirectoryW>]       ; \CreateDirectoryW

001D3BB1  |>  BE 08020000   MOV ESI,0x208

001D3BB6  |.  8D85 F0EBFFFF LEA EAX,DWORD PTR SS:[EBP-0x1410]

001D3BBC  |.  56            PUSH ESI

001D3BBD  |.  53            PUSH EBX

001D3BBE  |.  50            PUSH EAX

001D3BBF  |.  E8 2CE61000   CALL WnSkinIn.002E21F0

001D3BC4  |.  68 28BE3000   PUSH WnSkinIn.0030BE28                                 ;  .wnpf

001D3BC9  |.  8D85 F0EBFFFF LEA EAX,DWORD PTR SS:[EBP-0x1410]

001D3BCF  |.  68 C0C83000   PUSH WnSkinIn.0030C8C0                                 ;  [1]%s

001D3BD4  |.  50            PUSH EAX

001D3BD5  |.  E8 C500FFFF   CALL WnSkinIn.001C3C9F

001D3BDA  |.  8D85 F0EBFFFF LEA EAX,DWORD PTR SS:[EBP-0x1410]

001D3BE0  |.  50            PUSH EAX

001D3BE1  |.  8D85 E8FDFFFF LEA EAX,DWORD PTR SS:[EBP-0x218]

001D3BE7  |.  50            PUSH EAX

001D3BE8  |.  E8 58111100   CALL WnSkinIn.002E4D45

001D3BED  |.  83C4 20       ADD ESP,0x20

001D3BF0  |.  85C0          TEST EAX,EAX

001D3BF2  |.  74 18         JE SHORT WnSkinIn.001D3C0C

001D3BF4  |.  33C9          XOR ECX,ECX

001D3BF6  |.  66:8908       MOV WORD PTR DS:[EAX],CX

001D3BF9  |.  8D85 E8FDFFFF LEA EAX,DWORD PTR SS:[EBP-0x218]

001D3BFF  |.  68 28BE3000   PUSH WnSkinIn.0030BE28                                 ;  .wnpf

001D3C04  |.  50            PUSH EAX

001D3C05  |.  E8 1D2F1100   CALL WnSkinIn.002E6B27

001D3C0A  |.  59            POP ECX

001D3C0B  |.  59            POP ECX

001D3C0C  |>  8D85 E8FDFFFF LEA EAX,DWORD PTR SS:[EBP-0x218]

001D3C12  |.  50            PUSH EAX

001D3C13  |.  8D85 C8F5FFFF LEA EAX,DWORD PTR SS:[EBP-0xA38]

001D3C19  |.  50            PUSH EAX

001D3C1A  |.  E8 392F1100   CALL WnSkinIn.002E6B58

001D3C1F  |.  8D85 C8F5FFFF LEA EAX,DWORD PTR SS:[EBP-0xA38]

001D3C25  |.  68 28BE3000   PUSH WnSkinIn.0030BE28                                 ;  .wnpf

001D3C2A  |.  50            PUSH EAX

001D3C2B  |.  E8 15111100   CALL WnSkinIn.002E4D45

001D3C30  |.  83C4 10       ADD ESP,0x10

001D3C33  |.  85C0          TEST EAX,EAX

001D3C35  |.  74 05         JE SHORT WnSkinIn.001D3C3C

001D3C37  |.  33C9          XOR ECX,ECX

001D3C39  |.  66:8908       MOV WORD PTR DS:[EAX],CX

001D3C3C  |>  8D85 E8FDFFFF LEA EAX,DWORD PTR SS:[EBP-0x218]

001D3C42  |.  50            PUSH EAX

001D3C43  |.  8D85 D8F9FFFF LEA EAX,DWORD PTR SS:[EBP-0x628]

001D3C49  |.  50            PUSH EAX

001D3C4A  |.  8D85 D0F7FFFF LEA EAX,DWORD PTR SS:[EBP-0x830]

001D3C50  |.  68 DCA83000   PUSH WnSkinIn.0030A8DC                                 ;  %s%s

001D3C55  |.  50            PUSH EAX

001D3C56  |.  E8 4400FFFF   CALL WnSkinIn.001C3C9F

001D3C5B  |.  56            PUSH ESI

001D3C5C  |.  8D85 E8E9FFFF LEA EAX,DWORD PTR SS:[EBP-0x1618]

001D3C62  |.  53            PUSH EBX

001D3C63  |.  50            PUSH EAX

001D3C64  |.  E8 87E51000   CALL WnSkinIn.002E21F0

001D3C69  |.  56            PUSH ESI

001D3C6A  |.  8D85 E0E7FFFF LEA EAX,DWORD PTR SS:[EBP-0x1820]

001D3C70  |.  53            PUSH EBX

001D3C71  |.  50            PUSH EAX

001D3C72  |.  E8 79E51000   CALL WnSkinIn.002E21F0

001D3C77  |.  8D85 F8EDFFFF LEA EAX,DWORD PTR SS:[EBP-0x1208]

001D3C7D  |.  50            PUSH EAX

001D3C7E  |.  8D85 E8E9FFFF LEA EAX,DWORD PTR SS:[EBP-0x1618]

001D3C84  |.  50            PUSH EAX

001D3C85  |.  E8 CE2E1100   CALL WnSkinIn.002E6B58

001D3C8A  |.  8D85 D0F7FFFF LEA EAX,DWORD PTR SS:[EBP-0x830]

001D3C90  |.  50            PUSH EAX

001D3C91  |.  8D85 E0E7FFFF LEA EAX,DWORD PTR SS:[EBP-0x1820]

001D3C97  |.  50            PUSH EAX

001D3C98  |.  E8 BB2E1100   CALL WnSkinIn.002E6B58

001D3C9D  |.  8D85 E8E9FFFF LEA EAX,DWORD PTR SS:[EBP-0x1618]

001D3CA3  |.  50            PUSH EAX

001D3CA4  |.  E8 E6601100   CALL WnSkinIn.002E9D8F

001D3CA9  |.  8BF0          MOV ESI,EAX                                            ; |

001D3CAB  |.  8D85 E0E7FFFF LEA EAX,DWORD PTR SS:[EBP-0x1820]                      ; |

001D3CB1  |.  50            PUSH EAX                                               ; |Arg1

001D3CB2  |.  E8 D8601100   CALL WnSkinIn.002E9D8F                                 ; \WnSkinIn.002E9D8F

001D3CB7  |.  83C4 40       ADD ESP,0x40

001D3CBA  |.  8985 C0E7FFFF MOV DWORD PTR SS:[EBP-0x1840],EAX

001D3CC0  |.  8D8D C7E7FFFF LEA ECX,DWORD PTR SS:[EBP-0x1839]

001D3CC6  |.  E8 C9E7FFFF   CALL WnSkinIn.001D2494

001D3CCB  |.  33C0          XOR EAX,EAX

001D3CCD  |.  8D8D C7E7FFFF LEA ECX,DWORD PTR SS:[EBP-0x1839]

001D3CD3  |.  40            INC EAX

001D3CD4  |.  8845 FC       MOV BYTE PTR SS:[EBP-0x4],AL

001D3CD7  |.  8D85 C6E7FFFF LEA EAX,DWORD PTR SS:[EBP-0x183A]

001D3CDD  |.  50            PUSH EAX                                               ; /Arg2

001D3CDE  |.  8D85 F8EDFFFF LEA EAX,DWORD PTR SS:[EBP-0x1208]                      ; |

001D3CE4  |.  50            PUSH EAX                                               ; |Arg1

001D3CE5  |.  E8 3C380000   CALL WnSkinIn.001D7526                                 ; \WnSkinIn.001D7526

001D3CEA  |.  84C0          TEST AL,AL

001D3CEC  |.  75 18         JNZ SHORT WnSkinIn.001D3D06

001D3CEE  |.  3885 C6E7FFFF CMP BYTE PTR SS:[EBP-0x183A],AL

001D3CF4  |.  0F84 56010000 JE WnSkinIn.001D3E50

001D3CFA  |.  8BCF          MOV ECX,EDI

001D3CFC  |.  E8 6DF1FFFF   CALL WnSkinIn.001D2E6E

001D3D01  |.  E9 5C010000   JMP WnSkinIn.001D3E62

001D3D06  |>  53            PUSH EBX                                               ; /FailIfExists

001D3D07  |.  8D85 D0F7FFFF LEA EAX,DWORD PTR SS:[EBP-0x830]                       ; |

001D3D0D  |.  50            PUSH EAX                                               ; |NewFileName

001D3D0E  |.  8D85 F8EDFFFF LEA EAX,DWORD PTR SS:[EBP-0x1208]                      ; |

001D3D14  |.  50            PUSH EAX                                               ; |ExistingFileName

001D3D15  |.  FF15 88A23000 CALL DWORD PTR DS:[<&KERNEL32.CopyFileW>]              ; \CopyFileW

001D3D1B  |.  FFB5 C0E7FFFF PUSH DWORD PTR SS:[EBP-0x1840]

001D3D21  |.  8BD8          MOV EBX,EAX

001D3D23  |.  56            PUSH ESI

001D3D24  |.  E8 92671100   CALL WnSkinIn.002EA4BB

001D3D29  |.  59            POP ECX

001D3D2A  |.  59            POP ECX

001D3D2B  |.  33C9          XOR ECX,ECX

001D3D2D  |.  85C0          TEST EAX,EAX

001D3D2F  |.  6A 01         PUSH 0x1

001D3D31  |.  58            POP EAX

001D3D32  |.  0F44C8        CMOVE ECX,EAX

001D3D35  |.  8D85 D0F7FFFF LEA EAX,DWORD PTR SS:[EBP-0x830]

001D3D3B  |.  6A 00         PUSH 0x0                                               ; /Arg2 = 00000000

001D3D3D  |.  898D C0E7FFFF MOV DWORD PTR SS:[EBP-0x1840],ECX                      ; |

001D3D43  |.  8D8D C7E7FFFF LEA ECX,DWORD PTR SS:[EBP-0x1839]                      ; |

001D3D49  |.  50            PUSH EAX                                               ; |Arg1

001D3D4A  |.  E8 94230000   CALL WnSkinIn.001D60E3                                 ; \WnSkinIn.001D60E3

001D3D4F  |.  E8 65A7FFFF   CALL WnSkinIn.001CE4B9

001D3D54  |.  8BC8          MOV ECX,EAX

001D3D56  |.  E8 2005FFFF   CALL WnSkinIn.001C427B

001D3D5B  |.  8B3D 1CA33000 MOV EDI,DWORD PTR DS:[<&KERNEL32.WritePrivateProfileSt>;  kernel32.WritePrivateProfileStringW

001D3D61  |.  8BF0          MOV ESI,EAX

001D3D63  |.  56            PUSH ESI                                               ; /FileName

001D3D64  |.  68 84AD3000   PUSH WnSkinIn.0030AD84                                 ; |1

001D3D69  |.  68 ECC73000   PUSH WnSkinIn.0030C7EC                                 ; |IsModifyUIStyle

001D3D6E  |.  68 0CC83000   PUSH WnSkinIn.0030C80C                                 ; |SetModify

001D3D73  |.  FFD7          CALL EDI                                               ; \WritePrivateProfileStringW

001D3D75  |.  56            PUSH ESI                                               ; /FileName

001D3D76  |.  68 84AD3000   PUSH WnSkinIn.0030AD84                                 ; |1

001D3D7B  |.  68 20C83000   PUSH WnSkinIn.0030C820                                 ; |IsModifyUIStyle64

001D3D80  |.  68 0CC83000   PUSH WnSkinIn.0030C80C                                 ; |SetModify

001D3D85  |.  FFD7          CALL EDI                                               ; \WritePrivateProfileStringW

001D3D87  |.  56            PUSH ESI                                               ; /FileName

001D3D88  |.  68 84AD3000   PUSH WnSkinIn.0030AD84                                 ; |1

001D3D8D  |.  68 44C83000   PUSH WnSkinIn.0030C844                                 ; |IsTempModifyUIStyle

001D3D92  |.  68 0CC83000   PUSH WnSkinIn.0030C80C                                 ; |SetModify

001D3D97  |.  FFD7          CALL EDI                                               ; \WritePrivateProfileStringW

001D3D99  |.  56            PUSH ESI                                               ; /FileName

001D3D9A  |.  BE 84AD3000   MOV ESI,WnSkinIn.0030AD84                              ; |1

001D3D9F  |.  56            PUSH ESI                                               ; |String => "1"

001D3DA0  |.  68 6CC83000   PUSH WnSkinIn.0030C86C                                 ; |IsTempModifyUIStyle64

001D3DA5  |.  68 0CC83000   PUSH WnSkinIn.0030C80C                                 ; |SetModify

001D3DAA  |.  FFD7          CALL EDI                                               ; \WritePrivateProfileStringW

001D3DAC  |.  68 08020000   PUSH 0x208

001D3DB1  |.  8D85 E0FBFFFF LEA EAX,DWORD PTR SS:[EBP-0x420]

001D3DB7  |.  6A 00         PUSH 0x0

001D3DB9  |.  50            PUSH EAX

001D3DBA  |.  E8 31E41000   CALL WnSkinIn.002E21F0

001D3DBF  |.  83C4 0C       ADD ESP,0xC

001D3DC2  |.  E8 F2A6FFFF   CALL WnSkinIn.001CE4B9

001D3DC7  |.  8BC8          MOV ECX,EAX

001D3DC9  |.  E8 BC04FFFF   CALL WnSkinIn.001C428A

001D3DCE  |.  50            PUSH EAX

001D3DCF  |.  8D85 E0FBFFFF LEA EAX,DWORD PTR SS:[EBP-0x420]

001D3DD5  |.  68 A4C73000   PUSH WnSkinIn.0030C7A4                                 ;  %sConfig\

001D3DDA  |.  50            PUSH EAX

001D3DDB  |.  E8 BFFEFEFF   CALL WnSkinIn.001C3C9F

001D3DE0  |.  8D85 E0FBFFFF LEA EAX,DWORD PTR SS:[EBP-0x420]                       ; |

001D3DE6  |.  68 C8A93000   PUSH WnSkinIn.0030A9C8                                 ; |Config.ini

001D3DEB  |.  50            PUSH EAX                                               ; |Arg1

001D3DEC  |.  E8 362D1100   CALL WnSkinIn.002E6B27                                 ; \WnSkinIn.002E6B27

001D3DF1  |.  83C4 14       ADD ESP,0x14

001D3DF4  |.  8D85 E0FBFFFF LEA EAX,DWORD PTR SS:[EBP-0x420]

001D3DFA  |.  50            PUSH EAX

001D3DFB  |.  56            PUSH ESI

001D3DFC  |.  68 B8C73000   PUSH WnSkinIn.0030C7B8                                 ;  IsUseSkin

001D3E01  |.  BE ACA63000   MOV ESI,WnSkinIn.0030A6AC                              ;  Setting

001D3E06  |.  56            PUSH ESI

001D3E07  |.  FFD7          CALL EDI

001D3E09  |.  8D85 E0FBFFFF LEA EAX,DWORD PTR SS:[EBP-0x420]

001D3E0F  |.  50            PUSH EAX

001D3E10  |.  8D85 C8F5FFFF LEA EAX,DWORD PTR SS:[EBP-0xA38]

001D3E16  |.  50            PUSH EAX

001D3E17  |.  68 CCC73000   PUSH WnSkinIn.0030C7CC                                 ;  SkinName

001D3E1C  |.  56            PUSH ESI

001D3E1D  |.  FFD7          CALL EDI

001D3E1F  |.  8D85 E0FBFFFF LEA EAX,DWORD PTR SS:[EBP-0x420]

001D3E25  |.  50            PUSH EAX

001D3E26  |.  68 84AD3000   PUSH WnSkinIn.0030AD84                                 ;  1

001D3E2B  |.  68 FCC83000   PUSH WnSkinIn.0030C8FC                                 ;  InstallSkin

001D3E30  |.  56            PUSH ESI

001D3E31  |.  FFD7          CALL EDI

001D3E33  |.  85DB          TEST EBX,EBX

001D3E35  |.  74 0F         JE SHORT WnSkinIn.001D3E46

001D3E37  |.  33DB          XOR EBX,EBX

001D3E39  |>  53            PUSH EBX

001D3E3A  |.  68 88C73000   PUSH WnSkinIn.0030C788                                 ;  安装皮肤

001D3E3F  |.  68 14C93000   PUSH WnSkinIn.0030C914                                 ;  安装皮肤成功

001D3E44  |.  EB 15         JMP SHORT WnSkinIn.001D3E5B

001D3E46  |>  33DB          XOR EBX,EBX

001D3E48  |.  399D C0E7FFFF CMP DWORD PTR SS:[EBP-0x1840],EBX

001D3E4E  |.^ 75 E9         JNZ SHORT WnSkinIn.001D3E39

001D3E50  |>  53            PUSH EBX

001D3E51  |.  68 88C73000   PUSH WnSkinIn.0030C788                                 ;  安装皮肤

001D3E56  |.  68 94C73000   PUSH WnSkinIn.0030C794                                 ;  安装皮肤失败

001D3E5B  |>  53            PUSH EBX                                               ; |hOwner

001D3E5C  |.  FF15 08A43000 CALL DWORD PTR DS:[<&USER32.MessageBoxW>]              ; \MessageBoxW
```
