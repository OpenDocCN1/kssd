
# morphine

> 原文：[`www.pediy.com/kssd/pediy06/pediy6719.htm`](https://www.pediy.com/kssd/pediy06/pediy6719.htm)

下载页面：[`bbs.pediy.com/showthread.php?s=&threadid=5830&perpage=15&pagenumber=1`](http://bbs.pediy.com/showthread.php?s=&threadid=5830&perpage=15&pagenumber=1) 或 点击此处本地下载。

软件简介：Morphine is very unique application for PE files encryption. Unlike other PE encryptors and compressors Morphine includes own PE loader which enables it to put whole source image to the `.text` section of new PE file. This one is very powerful because you can compress source file with your favourite compressor like UPX and then encrypt its output with Morphine. Another powerful thing here is polymorphic engine which always creates absolutely different decryptor for the new PE file. This mean if your favourite trojan horse is detected by an antivirus you can encrypt it with Morphine. You will not get the virus alert again.

【作者声明】：只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！

【调试环境】：`WinXP`、`Ollydbg`、`PEiD`、`PETools`

---

【脱壳过程】：

很少注意这个 `Morphine` 壳，`Morphine` 只是对 `Section Table` 动了手脚，没有加密输入表等。

感觉更像是一个简单的“伪装”壳罢了。看到论坛里这几天有兄弟讨论这个，所以抽点时间看了看。

`jingulong` 没时间写教程，我就来代写一下吧。

脱壳目标用 `csjwaman` 上传之 `Morphine` 加壳的“PE 文件分析器.exe”。

---

## 一、恢复 Section Table

```
005F125B     F8                clc
//进入 Ollydbg 后暂停在这
005F125C     85C9              test ecx,ecx
005F125E     F5                cmc
005F125F     57                push edi
005F1260     74 05             je short PE.005F1267
```

下断：`BP VirtualAlloc`

中断后取消断点，`Alt+F9` 返回

```
005F10FD     FFD3              call ebx
005F10FF     59                pop ecx
//返回这里
005F1100     85C0              test eax,eax
005F1102     75 13             jnz short PE.005F1117
005F1104     6A 40             push 40
005F1106     68 00100000       push 1000
005F110B     51                push ecx
005F110C     50                push eax
005F110D     FFD3              call ebx
005F110F     85C0              test eax,eax
005F1111     0F84 3A010000     je PE.005F1251
```

---

看看寄存器情况：

`EAX 00400000`
`ECX 77E5986B kernel32.77E5986B`
`EDX 7FFE0304`
`EBX 77E5980A kernel32.VirtualAlloc`
`ESP 0012F7A0`
`EBP 0012FFB0`
`ESI 005F18FB ASCII "CODE"`
`EDI 0012FDB8 ASCII "PE"`
`EIP 005F10FF PE.005F10FF`

转存 `005F18FB` 处看看，这里就是原来的段表信息：

```
005F18FB   43 4F 44 45 00 00 00 00 90 62 0B 00 00 10 00 00    CODE....恇
005F190B   00 64 0B 00 00 04 00 00 00 00 00 00 00 00 00 00    .d
005F191B   00 00 00 00 60 00 00 E0 44 41 54 41 00 00 00 00    ....\`.. DATA....
005F192B   98 22 00 00 00 80 0B 00 00 24 00 00 00 68 0B 00    ...?€
005F193B   00 00 00 00 00 00 00 00 00 00 00 00 60 00 00 E0    ............\`..?
005F194B   42 53 53 00 00 00 00 00 B5 0C 00 00 00 B0 0B 00    BSS.....?...?.
005F195B   00 00 00 00 00 8C 0B 00 00 00 00 00 00 00 00 00    .....?.........
005F196B   00 00 00 00 60 00 00 E0 2E 69 64 61 74 61 00 00    ....\`..?idata..
005F197B   7A 26 00 00 00 C0 0B 00 00 28 00 00 00 8C 0B 00    z&...?..(...?.
005F198B   00 00 00 00 00 00 00 00 00 00 00 00 60 00 00 E0    ............\`..?
005F199B   2E 74 6C 73 00 00 00 00 10 00 00 00 00 F0 0B 00    .tls........?.
005F19AB   00 00 00 00 00 B4 0B 00 00 00 00 00 00 00 00 00    .....?.........
005F19BB   00 00 00 00 60 00 00 E0 2E 72 64 61 74 61 00 00    ....\`..?rdata..
005F19CB   18 00 00 00 00 00 0C 00 00 02 00 00 00 B4 0B 00    ...........?.
005F19DB   00 00 00 00 00 00 00 00 00 00 00 00 60 00 00 F0    ............\`..?
005F19EB   2E 72 65 6C 6F 63 00 00 44 B1 00 00 00 10 0C 00    .reloc..D?....
005F19FB   00 B2 00 00 00 B6 0B 00 00 00 00 00 00 00 00 00    .?..?.........
005F1A0B   00 00 00 00 60 00 00 F0 2E 72 73 72 63 00 00 00    ....\`..?rsrc...
005F1A1B   00 14 02 00 00 D0 0C 00 00 14 02 00 00 68 0C 00    ...?....h..
005F1A2B   00 00 00 00 00 00 00 00 00 00 00 00 60 00 00 F0    ............\`..?
005F1A3B   00 00 00 00 00 00 00 00 00 00 00 00 00 F0 0E 00    .............?.
005F1A4B   00 00 00 00 00 7C 0E 00 00 00 00 00 00 00 00 00    .....|.........
005F1A5B   00 00 00 00 40 00 00 50 00 00 00 00 00 00 00 00    ....@..P........
```

`Section Table` (段表信息)为：

```
   Section Header  1
    Name:               CODE
    VirtualSize:        000B6290
    VirtualAddress:     00001000
    SizeOfRawData:      000B6400
    PointerToRawData:   00000400
    PointerToRelocations: 00000000
    PointerToLinenumbers: 00000000
    NumberOfRelocations: 0000
    NumberOfLinenumbers: 0000
    Characteristics:    E0000060

   Section Header  2
    Name:               DATA
    VirtualSize:        00002298
    VirtualAddress:     000B8000
    SizeOfRawData:      00002400
    PointerToRawData:   000B6800
    ...

   Section Header  3
    Name:               BSS
    VirtualSize:        00000CB5
    VirtualAddress:     000BB000
    SizeOfRawData:      00000000
    PointerToRawData:   000B8C00
    ...

   Section Header  4
    Name:               .idata
    VirtualSize:        0000267A
    VirtualAddress:     000BC000
    SizeOfRawData:      00002800
    PointerToRawData:   000B8C00
    ...

   Section Header  5
    Name:               .tls
    VirtualSize:        00000010
    VirtualAddress:     000BF000
    SizeOfRawData:      00000000
    PointerToRawData:   000BB400
    ...

   Section Header  6
    Name:               .rdata
    VirtualSize:        00000018
    VirtualAddress:     000C0000
    SizeOfRawData:      00000200
    PointerToRawData:   000BB400
    ...

   Section Header  7
    Name:               .reloc
    VirtualSize:        0000B144
    VirtualAddress:     000C1000
    SizeOfRawData:      0000B200
    PointerToRawData:   000BB600
    ...

   Section Header  8
    Name:               .rsrc
    VirtualSize:        00021400
    VirtualAddress:     000CD000
    SizeOfRawData:      00021400
    PointerToRawData:   000C6800
    ...
```

---

下面开始解压数据：

```
005F1117     8945 F4         mov dword ptr ss:[ebp-C],eax ; 00400000
005F111A     89C7            mov edi,eax
005F111C     8B75 08        mov esi,dword ptr ss:[ebp+8] ; PE.005F1703
005F111F     8B4E 3C        mov ecx,dword ptr ds:[esi+3C]
005F1122     81C1 F8000000  add ecx,0F8

//ECX=00000100+0F8=1F8      在这里修改吧
//005F1703+1F8＝005F18FB     块表信息开始在 005F18FB，不解压到 PE 中
//改为：ADD ECX,260         这样加大 Size 就可以了 ★

005F1128     F3:A4          rep movsb byte ptr es:[edi],byte ptr ds:[esi]
//ESI=005F1703+360＝005F1A63

005F112A     8B45 08        mov eax,dword ptr ss:[ebp+8]
005F112D     0340 3C        add eax,dword ptr ds:[eax+3C]
005F1130     0FB640 06      movzx eax,byte ptr ds:[eax+6]
005F1134     8D7D C8        lea edi,dword ptr ss:[ebp-38]
005F1137     57             push edi
005F1138     6A 0A          push 0A
005F113A     59             pop ecx

//别忘了要把 ESI 的值还原为 005F18FB  ★

005F113B     F3:A5          rep movsd dword ptr es:[edi],dword ptr ds:[esi]
005F113D     5F             pop edi
005F113E     8B57 14        mov edx,dword ptr ds:[edi+14]
005F1141     85D2           test edx,edx
005F1143     74 14          je short PE.005F1159
005F1145     56             push esi
005F1146     8B75 08        mov esi,dword ptr ss:[ebp+8]
005F1149     01D6           add esi,edx
005F114B     8B4F 10        mov ecx,dword ptr ds:[edi+10]
005F114E     8B57 0C        mov edx,dword ptr ds:[edi+C]
005F1151     8B7D F4        mov edi,dword ptr ss:[ebp-C]
005F1154     01D7           add edi,edx
005F1156     F3:A4          rep movsb byte ptr es:[edi],byte ptr ds:[esi]
```
