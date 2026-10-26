
## 十二. 读入 IAT 信息

`053C3608`    `8B45 D8`         `MOV EAX,DWORD PTR SS:[EBP-28]`                             ; 已处理的长度
`053C35F0`    `8B4D DC`         `MOV ECX,DWORD PTR SS:[EBP-24]`                             ; `EXERe.053C4100`
`053C357C`    `57`              `PUSH EDI`
`053C3A48`    `6A 04`           `PUSH 4`
`053C35BA`    `8D3408`          `LEA ESI,DWORD PTR DS:[EAX+ECX]`
`053C3B0E`    `8D45 E4`         `LEA EAX,DWORD PTR SS:[EBP-1C]`
`053C3B62`    `50`              `PUSH EAX`
`053C3B4C`    `A1 88A03B05`     `MOV EAX,DWORD PTR DS:[53BA088]`
`053C3B22`    `56`              `PUSH ESI`                                                  ; `EXERe.053EB014`
`053C3B3A`    `FF35 D8A03B05`   `PUSH DWORD PTR DS:[53BA0D8]`
`053C3AE3`    `F7D0`            `NOT EAX`
`053C3AF7`    `FFD0`            `CALL EAX`                                                  ; `ReadProcessMemory(4 byte)`
`053C3AA9`    `57`              `PUSH EDI`
`053C3B04`    `8D45 F0`         `LEA EAX,DWORD PTR SS:[EBP-10]`
`053C3ACA`    `6A 04`           `PUSH 4`
`053C3A91`    `50`              `PUSH EAX`
`053C3AB5`    `8D46 04`         `LEA EAX,DWORD PTR DS:[ESI+4]`
`053C3A61`    `50`              `PUSH EAX`                                                  ; `EXERe.053EB018`
`053C3AD7`    `A1 88A03B05`     `MOV EAX,DWORD PTR DS:[53BA088]`
`053C3A79`    `FF35 D8A03B05`   `PUSH DWORD PTR DS:[53BA0D8]`
`053C3986`    `F7D0`            `NOT EAX`
`053C39B1`    `FFD0`            `CALL EAX`                                                  ; `ReadProcessMemory (4 byte)`
`053C3997`    `FF75 E4`         `PUSH DWORD PTR SS:[EBP-1C]`                                ; uncompressed size = `0F54`
`053C39FE`    `A1 B4A03B05`     `MOV EAX,DWORD PTR DS:[53BA0B4]`
`053C3A14`    `F7D0`            `NOT EAX`
`053C3A2F`    `6A 08`           `PUSH 8`
`053C39CA`    `FF35 D4A03B05`   `PUSH DWORD PTR DS:[53BA0D4]`
`053C39EB`    `FFD0`            `CALL EAX`                                                  ; `ntdll.RtlAllocateHeap`
`053C395A`    `FF75 F0`         `PUSH DWORD PTR SS:[EBP-10]`                                ; compressed size = `095F`
`053C3A1B`    `8945 EC`         `MOV DWORD PTR SS:[EBP-14],EAX`
`053C3A08`    `A1 B4A03B05`     `MOV EAX,DWORD PTR DS:[53BA0B4]`
`053C3972`    `6A 08`           `PUSH 8`
`053C38A4`    `FF35 D4A03B05`   `PUSH DWORD PTR DS:[53BA0D4]`
`053C3882`    `F7D0`            `NOT EAX`
`053C388F`    `FFD0`            `CALL EAX`                                                  ; `ntdll.RtlAllocateHeap`
`053C383D`    `8BD8`            `MOV EBX,EAX`
`053C382C`    `57`              `PUSH EDI`
`053C381C`    `895D B4`         `MOV DWORD PTR SS:[EBP-4C],EBX`
`053C3850`    `FF75 F0`         `PUSH DWORD PTR SS:[EBP-10]`                                ; `095F size`
`053C3862`    `A1 88A03B05`     `MOV EAX,DWORD PTR DS:[53BA088]`
`053C386E`    `83C6 08`         `ADD ESI,8`                                                 ; 跳过 8 字节
`053C3912`    `53`              `PUSH EBX`
`053C392A`    `56`              `PUSH ESI`                                                  ; `EXERe.053EB01C`
`053C393C`    `FF35 D8A03B05`   `PUSH DWORD PTR DS:[53BA0D8]`
`053C38D7`    `F7D0`            `NOT EAX`
`053C38F3`    `FFD0`            `CALL EAX`                                                  ; `ReadProcessMemory`
`053C3902`    `8B45 F0`         `MOV EAX,DWORD PTR SS:[EBP-10]`
`053C38E0`    `33F6`            `XOR ESI,ESI`                                               ; `EXERe.053EB01C`
`053C38B9`    `83C0 F8`         `ADD EAX,-8`
`053C38C3`    `897D DC`         `MOV DWORD PTR SS:[EBP-24],EDI`
`053BFDC3`    `85C0`            `TEST EAX,EAX`
`053BFDE3`   `/0F86 31010000`   `JBE EXERe.053BFF1A`
`053BFDCB`    `8B041E`          `MOV EAX,DWORD PTR DS:[ESI+EBX]`                            ; 循环解压
`053BFE2F`    `8B4C33 04`       `MOV ECX,DWORD PTR DS:[EBX+ESI+4]`
`053BFDD9`    `83C6 08`         `ADD ESI,8`
`053BFDEE`    `3BC1`            `CMP EAX,ECX`
`053BFDF9`    `8945 E8`         `MOV DWORD PTR SS:[EBP-18],EAX`
`053BFE05`    `894D F8`         `MOV DWORD PTR SS:[EBP-8],ECX`
`053BFE13`   `/0F85 6F2B0000`   `JNZ EXERe.053C2988`
`053C2988`    `8B45 DC`         `MOV EAX,DWORD PTR SS:[EBP-24]`
`053C2953`    `8B4D EC`         `MOV ECX,DWORD PTR SS:[EBP-14]`
`053C2D9C`    `FF75 E8`         `PUSH DWORD PTR SS:[EBP-18]`
`053C2DD0`    `03C1`            `ADD EAX,ECX`
`053C2DDA`    `B9 70083B05`     `MOV ECX,EXERe.053B0870`
`053C2DEC`    `50`              `PUSH EAX`
`053C2DB2`    `8D041E`          `LEA EAX,DWORD PTR DS:[ESI+EBX]`
`053C2DC0`    `FF75 F8`         `PUSH DWORD PTR SS:[EBP-8]`
`053C2BA7`    `50`              `PUSH EAX`
`053C2B8C`    `FF15 A0283C05`   `CALL DWORD PTR DS:[53C28A0]`                               ; `EXERe.053A53E1`  解压(`compressed`, `size1`, `uncompress`, `size2`)
`053BFFC7`    `8B45 E8`         `MOV EAX,DWORD PTR SS:[EBP-18]`
`053BFFBE`    `0375 F8`         `ADD ESI,DWORD PTR SS:[EBP-8]`
`053BFF0C`    `0145 DC`         `ADD DWORD PTR SS:[EBP-24],EAX`
`053BFEE3`    `8B45 F0`         `MOV EAX,DWORD PTR SS:[EBP-10]`
`053BFED8`    `83C0 F8`         `ADD EAX,-8`
`053BFF02`    `3BF0`            `CMP ESI,EAX`
`053BFF3F`  `^0F82 86FEFFFF`   `JB EXERe.053BFDCB`

## 十三. 处理 IAT

`053BFF20`    `FF75 EC`         `PUSH DWORD PTR SS:[EBP-14]`                                ; 解压后的 IAT 信息
`053BFF2E`    `8D4D 8C`         `LEA ECX,DWORD PTR SS:[EBP-74]`                             ; `12FF4C`
`053BFF70`    `FF15 F9333C05`   `CALL DWORD PTR DS:[53C33F9]`                               ; `EXERe.053A486D`, 初步解密

`0012FF4C`   `00000007`  ....  ; 7 个 DLL
`0012FF50`   `00146240`  .)..  ; 开始地址, 每个DLL 4 + 4 + 40h*4 + 4 + 4 + 4 byte { A, B, C, D, E, F }
`0012FF54`   `001469cc`  T1..  ; 结束地址

其中 `A` = 该 DLL 中 API 个数
    `B` = 加壳前 FirstThunk 位置 (RVA)
    `C` = DLL Name
