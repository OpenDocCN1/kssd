
```
005B34D2  |. 50           PUSH EAX
005B34D3  |. E8 C83C0800  CALL starryni.006371A0           ****比较看是否相等
005B34D8  |. 59           POP ECX
005B34D9  |. 59           POP ECX
005B34DA  |. 85C0         TEST EAX,EAX                     ****返回 0 表示相等
005B34DC  |. 74 0B        JE SHORT starryni.005B34E9
005B34DE  |. 8B45 18      MOV EAX,DWORD PTR SS:[EBP+18]
005B34E1  |. C600 08      MOV BYTE PTR DS:[EAX],8
005B34E4  |. E9 F6000000  JMP starryni.005B35DF
005B34E9  |> 8D7D CC      LEA EDI,DWORD PTR SS:[EBP-34]
005B34EC  |. 31C0         XOR EAX,EAX
005B34EE  |. 83C9 FF      OR ECX,FFFFFFFF
005B34F1  |. F2:AE        REPNE SCAS BYTE PTR ES:[EDI]
005B34F3  |. BB FEFFFFFF  MOV EBX,-2
005B34F8  |. 29CB         SUB EBX,ECX
005B34FA  |. 807D 97 00   CMP BYTE PTR SS:[EBP-69],0      ****不明标志
005B34FE  |. 74 29        JE SHORT starryni.005B3529
005B3500  |. FF75 10      PUSH DWORD PTR SS:[EBP+10]       ****名字
005B3503  |. E8 ED010000  CALL starryni.005B36F5           ****关键 call,计算名字检查和
```

`005B36F5` `PUSH` `EBP`  
`005B36F6` `MOV` `EBP` `ESP`  
`005B36F8` `PUSH` `EBX`  
`005B36F9` `PUSH` `ESI`  
`005B36FA` `PUSH` `EDI`  
`005B36FB` `SUB` `ESP` `24`  
`005B36FE` `MOV` `DWORD PTR SS:[EBP-30]` `75BCD15`  ****常数  
`005B3705` `PUSH` `1E`  
`005B3707` `LEA` `EAX` `DWORD PTR SS:[EBP-2C]`  
`005B370A` `PUSH` `EAX`  
`005B370B` `PUSH` `DWORD PTR SS:[EBP+8]`  
`005B370E` `CALL` `starryni.005B37BD`  ****这个函数见过了，就是串联字符串达到 `0x1E`  
`005B3713` `ADD` `ESP` `0C`  
`005B3716` `TEST` `AL` `AL`  
`005B3718` `JNZ` `SHORT` `starryni.005B3721`  
`005B371A` `MOV` `EAX` `4455EE33`  ****奇怪的返回值表示失败  
`005B371F` `JMP` `SHORT` `starryni.005B3754`  
`005B3721` `LEA` `EDI` `DWORD PTR SS:[EBP-2C]`  ****注册名字相连而成的长度为 `1E` 的串  
`005B3724` `XOR` `EAX` `EAX`  
`005B3726` `OR` `ECX` `FFFFFFFF`  
`005B3729` `REPNE` `SCAS` `BYTE PTR ES:[EDI]`  
`005B372B` `MOV` `ESI` `-2`  
`005B3730` `SUB` `ESI` `ECX`  
`005B3732` `XOR` `EBX` `EBX`  ****`ebx` 是索引 `index`  
`005B3734` `JMP` `SHORT` `starryni.005B374D`  
`005B3736` `/` `MOVZX` `ECX` `BYTE PTR SS:[EBP+EBX-2C]`  ****`ecx = ch`  
`005B373B` `|` `MOV` `EDX` `EBX`  ****`edx = index`  
`005B373D` `|` `IMUL` `EDX` `EBX`  ****`edx = index**2`  
`005B3740` `|` `IMUL` `EDX` `EBX`  ****`edx = index**3`  
`005B3743` `|` `IMUL` `EDX` `ECX`  ****`edx = index**3*ch`  
`005B3746` `|` `IMUL` `EDX` `ECX`  ****`edx = index**3*ch**2`  
`005B3749` `|` `ADD` `DWORD PTR SS:[EBP-30]` `EDX`  ****加到常数 `75BCD15` 上  
`005B374C` `|` `INC` `EBX`  
`005B374D` `CMP` `EBX` `ESI`  
`005B374F` `\` `JL` `SHORT` `starryni.005B3736`  
`005B3751` `MOV` `EAX` `DWORD PTR SS:[EBP-30]`  
`005B3754` `LEA` `ESP` `DWORD PTR SS:[EBP-C]`  
`005B3757` `POP` `EDI`  
`005B3758` `POP` `ESI`  
`005B3759` `POP` `EBX`  
`005B375A` `POP` `EBP`  
`005B375B` `RETN`  

这个过程计算名字的检查和，同样先把名字串联达到 `0x1E` 长度，然后计算：

```
checksum = sum(index**3*ch**2) + 0x75BCD15
```

```
005B3508  |. 59           POP ECX
005B3509  |. 89C6         MOV ESI,EAX                    ****名字的检查和
005B350B  |. 8D53 F8      LEA EDX,DWORD PTR DS:[EBX-8]
005B350E  |. 8D4D CC      LEA ECX,DWORD PTR SS:[EBP-34]   ****注册码前 4 部分相连并重排后的字符串
005B3511  |. 01CA         ADD EDX,ECX                    ****`edx` 指向该字符串的后面 8 个字符
005B3513  |. 52           PUSH EDX
005B3514  |. E8 6A040000  CALL starryni.005B3983          ****转换成 16 进制数码
005B3519  |. 59           POP ECX
005B351A  |. 39F0         CMP EAX,ESI                    ****结果与名字的检查和比较
005B351C  |. 74 0B        JE SHORT starryni.005B3529      ****必须相等，否则失败
005B351E  |. 8B45 18      MOV EAX,DWORD PTR SS:[EBP+18]
005B3521  |. C600 04      MOV BYTE PTR DS:[EAX],4
005B3524  |. E9 B6000000  JMP starryni.005B35DF
005B3529  |> 8D45 CF      LEA EAX,DWORD PTR SS:[EBP-31]  ****重排后注册码的第二部分
005B352C  |. 50           PUSH EAX
005B352D  |. E8 FF030000  CALL starryni.005B3931          ****转换成 16 进制数
005B3532  |. 59           POP ECX
005B3533  |. 0FB7C0       MOVZX EAX,AX                   只保留低位字
005B3536  |. 89C3         MOV EBX,EAX
005B3538  |. E8 2E010000  CALL starryni.005B366B          ****这个函数没有搞懂。看上去十分复杂，很多层次
                                                         ****子程序调用，但与名字及注册码均无关，最后返回
005B353D  |. 89DA         MOV EDX,EBX                    ****`0x135A`
005B353F  |. 29C2         SUB EDX,EAX                    ****第二部分的值减去这个 `135A`
005B3541  |. 8B45 14      MOV EAX,DWORD PTR SS:[EBP+14]
005B3544  |. 8910         MOV DWORD PTR DS:[EAX],EDX      ****如果结果小于 0 就会失败
005B3546  |. 0FBE45 98    MOVSX EAX,BYTE PTR SS:[EBP-68]  ****这里是注册码第二个字符，可能决定注册的版本
005B354A  |. 83E8 63      SUB EAX,63
005B354D  |. 74 26        JE SHORT starryni.005B3575
005B354F  |. 83E8 0B      SUB EAX,0B
005B3552  |. 74 56        JE SHORT starryni.005B35AA
005B3554  |. 83E8 04      SUB EAX,4
005B3557  |. 74 3B        JE SHORT starryni.005B3594
005B3559  |. 83E8 02      SUB EAX,2
005B355C  |. 74 07        JE SHORT starryni.005B3565
005B355E  |. 83E8 01      SUB EAX,1
005B3561  |. 74 47        JE SHORT starryni.005B35AA
005B3563  |. EB 4D        JMP SHORT starryni.005B35B2
005B3565  |> 8B45 14      MOV EAX,DWORD PTR SS:[EBP+14]   ****'t'，注册 15 天试用版
005B3568  |. 8338 00      CMP DWORD PTR DS:[EAX],0
005B356B  |. 7C 2F        JL SHORT starryni.005B359C
005B356D  |. 837D 1C 28   CMP DWORD PTR SS:[EBP+1C],28
005B3571  |. 7E 47        JLE SHORT starryni.005B35BA
005B3573  |. EB 27        JMP SHORT starryni.005B359C
005B3575  |> 8B45 14      MOV EAX,DWORD PTR SS:[EBP+14]   ****'c'，不明用处
005B3578  |. 8338 00      CMP DWORD PTR DS:[EAX],0
005B357B  |. 7C 1F        JL SHORT starryni.005B359C
005B357D  |. 837D 1C 14   CMP DWORD PTR SS:[EBP+1C],14
005B3581  |. 7E 02        JLE SHORT starryni.005B3585
005B3583  |. EB 17        JMP SHORT starryni.005B359C
005B3585  |> BA 14000000  MOV EDX,14
005B358A  |. 2B55 1C      SUB EDX,DWORD PTR SS:[EBP+1C]
005B358D  |. 8B45 14      MOV EAX,DWORD PTR SS:[EBP+14]
005B3590  |. 8910         MOV DWORD PTR DS:[EAX],EDX
005B3592  |. EB 26        JMP SHORT starryni.005B35BA
005B3594  |> 8B45 14      MOV EAX,DWORD PTR SS:[EBP+14]   ****'r'，好像是正式的 backyard 版
005B3597  |. 8338 00      CMP DWORD PTR DS:[EAX],0
005B359A  |. 7D 1E        JGE SHORT starryni.005B35BA
005B359C  |> 8B45 18      MOV EAX,DWORD PTR SS:[EBP+18]
005B359F  |. C600 03      MOV BYTE PTR DS:[EAX],3
005B35A2  |. 8B45 14      MOV EAX,DWORD PTR SS:[EBP+14]
005B35A5  |. 8320 00      AND DWORD PTR DS:[EAX],0
005B35A8  |. EB 35        JMP SHORT starryni.005B35DF
005B35AA  |> 8B45 14      MOV EAX,DWORD PTR SS:[EBP+14]   ****'n'和'u'，没看出有什么特别
005B35AD  |. 8320 00      AND DWORD PTR DS:[EAX],0
005B35B0  |. EB 08        JMP SHORT starryni.005B35BA
```
