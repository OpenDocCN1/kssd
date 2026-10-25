
```
0040333E  |.  8D5424 14     LEA EDX,DWORD PTR SS:[ESP+14]               ;  指向前4字节

00403342  |.  8BCD          MOV ECX,EBP

00403344  |.  52            PUSH EDX                                    ; /Arg1

00403345  |.  895C24 18     MOV DWORD PTR SS:[ESP+18],EBX               ; |

00403349  |.  E8 82000000   CALL Bmp2TxtO.004033D0                      ; 取C:盘的volumeSerialNumber

004033D0  /$  83EC 08       SUB ESP,8

004033D3  |.  56            PUSH ESI

004033D4  |.  8B7424 10     MOV ESI,DWORD PTR SS:[ESP+10]

004033D8  |.  85F6          TEST ESI,ESI

004033DA  |.  75 09         JNZ SHORT Bmp2TxtO.004033E5

004033DC  |.  33C0          XOR EAX,EAX

004033DE  |.  5E            POP ESI

004033DF  |.  83C4 08       ADD ESP,8

004033E2  |.  C2 0400       RETN 4

004033E5  |>  6A 00         PUSH 0                                   ; /pFileSystemNameSize = Null

004033E7  |.  8D4424 08     LEA EAX,DWORD PTR SS:[ESP+8]             ; |

004033EB  |.  6A 00         PUSH 0                                   ; |pFileSystemNameBuffer = Null

004033ED  |.  8D4C24 10     LEA ECX,DWORD PTR SS:[ESP+10]            ; |

004033F1  |.  50            PUSH EAX                                 ; |pFileSystemFlags = 0012F5E4

004033F2  |.  8D5424 1C     LEA EDX,DWORD PTR SS:[ESP+1C]            ; |

004033F6  |.  51            PUSH ECX                                 ; |pMaxFilenameLength = 0012F5E8

004033F7  |.  52            PUSH EDX                                 ; |pVolumeSerialNumber = 0012F5F0

004033F8  |.  6A 00         PUSH 0                                   ; |MaxVolumeNameSize = 0

004033FA  |.  6A 00         PUSH 0                                   ; |VolumeNameBuffer = Null

004033FC  |.  68 1C734000   PUSH Bmp2TxtO.0040731C                   ; |RootPathName = "c:\
```
