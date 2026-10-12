
# 木马克星 1120

> 原文：[`www.pediy.com/kssd/pediy06/pediy6375.htm`](https://www.pediy.com/kssd/pediy06/pediy6375.htm)

软件介绍：

- **软件名称**：木马克星（`iparmor`）
- **整理日期**：2003.11.20
- **最新版本**：1120
- **文件大小**：2.5M
- **软件授权**：共享软件
- **使用平台**：`Win9x/Me/NT/2000`
- **发布公司**：`http://www.luosoft.com/`

**软件简介**：

可以查杀 5021 种国际木马，112 种电子邮件木马，保证查杀冰河类文件关联木马，`oicq` 类寄生木马，`icmp` 类幽灵木马，网络神偷类反弹木马。内置木马防火墙，任何黑客试图与本机建立连接，都需要 `Iparmor` 确认，不仅可以查杀木马，更可以查黑客。

**下载地址**：`http://www.luosoft.com/downcn.htm`

**用户名**：`leozem[YCG]`
**假序列号**：`8792492`
**真序列号**：`493756985`

\---------------------------
**破解人**：`leozem[YCG]`，转贴请注明出处。

**工具**：`ollydbg`，`pw32dasmgold`

首先用 `PW32` 打开木马克星

字串参考“软件已经被成功注册”，双击

然后再用 `OD` 打开木马克星

```
:00568447 E8742EEAFF           call 0040B2C0
:0056844C 8B55FC               mov edx, dword ptr [ebp-04]----注册名进 EDX
:0056844F 8BC6                 mov eax, esi
:00568451 E8C6E3EDFF           call 0044681C
:00568456 8D55F0               lea edx, dword ptr [ebp-10]
:00568459 8BB3D8020000         mov esi, dword ptr [ebx+000002D8]

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:005683EA(C)
|

:0056845F 8BC6                 mov eax, esi
:00568461 E876E3EDFF           call 004467DC
:00568466 8B45F0               mov eax, dword ptr [ebp-10]-----序列号进 EAX
:00568469 8D55F4               lea edx, dword ptr [ebp-0C]
:0056846C E84F2EEAFF           call 0040B2C0
:00568471 8B55F4               mov edx, dword ptr [ebp-0C]
:00568474 8BC6                 mov eax, esi
:00568476 E8A1E3EDFF           call 0044681C
:0056847B 8D95E8FEFFFF         lea edx, dword ptr [ebp+FFFFFEE8]
:00568481 8B83E0020000         mov eax, dword ptr [ebx+000002E0]
:00568487 E850E3EDFF           call 004467DC
:0056848C 8B85E8FEFFFF         mov eax, dword ptr [ebp+FFFFFEE8]
:00568492 8D95ECFEFFFF         lea edx, dword ptr [ebp+FFFFFEEC]
:00568498 E8072BEAFF           call 0040AFA4-----小写变大写
:0056849D 8B95ECFEFFFF         mov edx, dword ptr [ebp+FFFFFEEC]
:005684A3 8D85F0FEFFFF         lea eax, dword ptr [ebp+FFFFFEF0]
:005684A9 B9FF000000           mov ecx, 000000FF
:005684AE E8CDC6E9FF           call 00404B80
:005684B3 8D95F0FEFFFF         lea edx, dword ptr [ebp+FFFFFEF0]
:005684B9 8B83D0020000         mov eax, dword ptr [ebx+000002D0]
:005684BF E848BFF1FF           call 0048440C-----生成关键码的 CALL，F7 追入得关键数 1D6E1D4F
:005684C4 8D95E4FEFFFF         lea edx, dword ptr [ebp+FFFFFEE4]
:005684CA 8B83D8020000         mov eax, dword ptr [ebx+000002D8]
:005684D0 E807E3EDFF           call 004467DC
:005684D5 8B85E4FEFFFF         mov eax, dword ptr [ebp+FFFFFEE4]
:005684DB 50                   push eax
:005684DC 8B83D0020000         mov eax, dword ptr [ebx+000002D0]
:005684E2 8B8024020000         mov eax, dword ptr [eax+00000224]
:005684E8 05EA040000           add eax, 000004EA----EAX=1D6E1D4F+4EA=1D6E2239
:005684ED 99                   cdq
:005684EE 33C2                 xor eax, edx
:005684F0 2BC2                 sub eax, edx-------求绝对值
:005684F2 8D95E0FEFFFF         lea edx, dword ptr [ebp+FFFFFEE0]
:005684F8 E83F30EAFF           call 0040B53C----将 1D6E2239 转成十进制
:005684FD 8B95E0FEFFFF         mov edx, dword ptr [ebp+FFFFFEE0]----EDX=493756985(注册码)
:00568503 58                   pop eax-------假码出贱
:00568504 E8ABC7E9FF           call 00404CB4---比较注册码的 CALL,再追
:00568509 0F85E5000000          jne 005685F4----关键跳转
:0056850F 6A00                  push 00000000
:00568511 8D85DCFEFFFF          lea eax, dword ptr [ebp+FFFFFEDC]
:00568517 50                    push eax
:00568518 8D95D8FEFFFF          lea edx, dword ptr [ebp+FFFFFED8]
.......
.......

* Possible StringData Ref from Code Obj ->"注册成功"

:005685A0 B8CC865600           mov eax, 005686CC
:005685A5 E81A84F0FF           call 004709C4
:005685AA EB0A                 jmp 005685B6
........
........

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00568509(C)
|

:005685F4 803D1DB7590000        cmp byte ptr [0059B71D], 00
:005685FB 740C                  je 00568609

* Possible StringData Ref from Code Obj ->"注册失败!"

:005685FD B80C875600           mov eax, 0056870C
:00568602 E8BD83F0FF           call 004709C4
:00568607 EB0A                 jmp 00568613
```

以下是算法部分

```
:0048440C 53                   push ebx
:0048440D 56                   push esi
:0048440E 57                   push edi
:0048440F 81C400FFFFFF          add esp, FFFFFF00
:00484415 8BF2                 mov esi, edx
:00484417 8D3C24               lea edi, dword ptr [esp]
:0048441A 33C9                 xor ecx, ecx
:0048441C 8A0E                 mov cl, byte ptr [esi]

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:00484433(U)
|

:0048444C 8BC3                 mov eax, ebx
:0048444E E805010000           call 00484558-------再 F7 追入
:00484453 81C400010000         add esp, 00000100
:00484459 5F                   pop edi
:0048445A 5E                   pop esi
:0048445B 5B                   pop ebx
```

```
:00484558 53                   push ebx
:00484559 83C4B8               add esp, FFFFFFB8
:0048455C 8BD8                 mov ebx, eax
:0048455E 33C0                 xor eax, eax
:00484560 8A4324               mov al, byte ptr [ebx+24]
:00484563 40                   inc eax
:00484564 83F846               cmp eax, 00000046----注册名位数和 70 比较
:00484567 7F0B                 jg 00484574--大于或等于 70 就跳到 484574
:00484569 C64403242A           mov [ebx+eax+24], 2A--小于 70 位用*（2A）补上
:0048456E 40                   inc eax---位数加 1
:0048456F 83F847               cmp eax, 00000047------和 71 比较
:00484572 75F5                 jne 00484569-----不等继续循环
:00484578 8A4C0324             mov cl, byte ptr [ebx+eax+24]
:0048457C 880A                 mov byte ptr [edx], cl
:0048457E 40                   inc eax
:0048457F 42                   inc edx
:00484580 83F847               cmp eax, 00000047
:00484583 75F3                 jne 00484578
:00484585 8BCC                 mov ecx, esp
:00484587 8B932C020000          mov edx, dword ptr [ebx+0000022C]
:0048458D 8BC3                 mov eax, ebx
:0048458F E87CFFFFFF           call 00484510----再 F7 跟入
:00484594 898324020000          mov dword ptr [ebx+00000224], eax
:0048459A 33C0                 xor eax, eax
:0048459C 8A8324010000          mov al, byte ptr [ebx+00000124]
:004845A2 40                   inc eax
:004845A3 83F846               cmp eax, 00000046
```

```
:00484510 53                   push ebx
:00484511 56                   push esi
:00484512 57                   push edi
:00484513 83C4B8               add esp, FFFFFFB8
:00484516 8BF1                 mov esi, ecx
:00484518 8D3C24               lea edi, dword ptr [esp]
:0048451B B911000000           mov ecx, 00000011
:00484520 F3                   repz---将注册名位数 OB 放在注册名前
```



## 汇编指令分析

```
:00484521 A5                 movsd   

:00484522 66A5               movsw   

:00484524 A4                 movsb   

:00484525 B147               mov cl, 47   

:00484527 8BC4               mov eax, esp\------BLEOZEM\[YCG\]，B 为注册名位数 11   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:0048454D(C)   
|   

:00484529 8BDA               mov ebx, edx\---EBX\=EDX\=EFCA99(罗建斌给的)   

:0048452B C1EB08             shr ebx, 08-----EBX\=EFCA   

:0048452E 81E3FFFFFF00       and ebx, 00FFFFFF----EBX\=EFCA AND FFFFFF=EFCA   

:00484534 0FB630             movzx esi, byte ptr \[eax\]---循环取变化后注册名(字符)的 ASCALL 码进 ESI   

变化后注册名等于 BLEOZEM\[YCG\]\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*共 71 位,第 1 位是输入的注册名的长度,后面用\*补足.   

:00484537 33D6               xor edx, esi\-----EDX\=EDX XOR SISI\=EFCA99 XOR B=EFCA92   

:00484539 81E2FF000000       and edx, 000000FF----EDX\=92   

:0048453F 8B149530F05700     mov edx, dword ptr \[4\*edx+0057F030\]--根据 EDX 取下面的数,共 256 个.   

:00484546 33DA               xor ebx, edx\---EBX\=EFCA XOR 1E01F268=1E011DA2   

:00484548 8BD3               mov edx, ebx\---EDX\=1E011DA2   

:0048454A 40                 inc eax\-------EAX\=EAX+1   

:0048454B FEC9               dec cl\--------CL\=CL\-1   

:0048454D 75DA               jne 00484529---CL 不等 0 就继续循环   

:0048454F 8BC2               mov eax, edx\-------EAX\=1D6E1D4F(关键数)   

:00484551 83C448             add esp, 00000048   

:00484554 5F                 pop edi   

:00484555 5E                 pop esi   

:00484556 5B                 pop ebx   

:00484557 C3                 ret   

:004845A6 7F0E               jg 004845B6   

\-----------------------------让 0048453F 中 EDX 取的数------------------   

77073096 EE0E612C 990951BA 076DC419 706AF48F E963A535 9E6495A3 0EDB8832   
79DCB8A4 E0D5E91E 97D2D988 09B64C2B 7EB17CBD E7B82D07 90BF1D91 00000000   
6AB020F2 F3B97148 84BE41DE 1ADAD47D 6DDDE4EB F4D4B551 83D385C7 136C9856   
646BA8C0 FD62F97A 8A65C9EC 14015C4F 63066CD9 FA0F3D63 8D080DF5 3B6E20C8   
4C69105E D56041E4 A2677172 3C03E4D1 4B04D447 D20D85FD A50AB56B 35B5A8FA   
42B2986C DBBBC9D6 ACBCF940 32D86CE3 45DF5C75 DCD60DCF ABD13D59 26D930AC   
51DE003A C8D75180 BFD06116 21B4F4B5 56B3C423 CFBA9599 B8BDA50F 2802B89E   
5F058808 C60CD9B2 B10BE924 2F6F7C87 58684C11 C1611DAB B6662D3D 76DC4190   
01DB7106 98D220BC EFD5102A 71B18589 06B6B51F 9FBFE4A5 E8B8D433 7807C9A2   
0F00F934 9609A88E E10E9818 7F6A0DBB 086D3D2D 91646C97 E6635C01 6B6B51F4   
1C6C6162 856530D8 F262004E 6C0695ED 1B01A57B 8208F4C1 F50FC457 65B0D9C6   
12B7E950 8BBEB8EA FCB9887C 62DD1DDF 15DA2D49 8CD37CF3 FBD44C65 4DB26158   
3AB551CE A3BC0074 D4BB30E2 4ADFA541 3DD895D7 A4D1C46D D3D6F4FB 4369E96A   
346ED9FC AD678846 DA60B8D0 44042D73 33031DE5 AA0A4C5F DD0D7CC9 5005713C   
270241AA BE0B1010 C90C2086 5768B525 206F85B3 B966D409 CE61E49F 5EDEF90E   
29D9C998 B0D09822 C7D7A8B4 59B33D17 2EB40D81 B7BD5C3B C0BA6CAD EDB88320   
9ABFB3B6 03B6E20C 74B1D29A EAD54739 9DD277AF 04DB2615 73DC1683 E3630B12   
94643B84 0D6D6A3E 7A6A5AA8 E40ECF0B 9309FF9D 0A00AE27 7D079EB1 F00F9344   
8708A3D2 1E01F268 6906C2FE F762575D 806567CB 196C3671 6E6B06E7 FED41B76   
89D32BE0 10DA7A5A 67DD4ACC F9B9DF6F 8EBEEFF9 17B7BE43 60B08ED5 D6D6A3E8   
A1D1937E 38D8C2C4 4FDFF252 D1BB67F1 A6BC5767 3FB506DD 48B2364B D80D2BDA   
AF0A1B4C 36034AF6 41047A60 DF60EFC3 A867DF55 316E8EEF 4669BE79 CB61B38C   
BC66831A 256FD2A0 5268E236 CC0C7795 BB0B4703 220216B9 5505262F C5BA3BBE   
B2BD0B28 2BB45A92 5CB36A04 C2D7FFA7 B5D0CF31 2CD99E8B 5BDEAE1D 9B64C2B0   
EC63F226 756AA39C 026D930A 9C0906A9 EB0E363F 72076785 05005713 95BF4A82   
E2B87A14 7BB12BAE 0CB61B38 92D28E9B E5D5BE0D 7CDCEFB7 0BDBDF21 86D3D2D4   
F1D4E242 68DDB3F8 1FDA836E 81BE16CD F6B9265B 6FB077E1 18B74777 88085AE6   
FF0F6A70 66063BCA 11010B5C 8F659EFF F862AE69 616BFFD3 166CCF45 A00AE278   
D70DD2EE 4E048354 3903B3C2 A7672661 D06016F7 4969474D 3E6E77DB AED16A4A   
D9D65ADC 40DF0B66 37D83BF0 A9BCAE53 DEBB9EC5 47B2CF7F 30B5FFE9 BDBDF21C   

CABAC28A 53B39330 24B4A3A6 BAD03605 CDD70693 54DE5729 23D967BF B3667A2E   

C4614AB8 5D681B02 2A6F2B94 B40BBE37 C30C8EA1 5A05DF1B 2D02EF8D 004C3E50   

\----------------------------------------------------------------------------   
```

以下是比较注册码部分:

`********************************************************************从 00568504 追入********************************************************************`

```
:00404CB4 53                 push ebx   

:00404CB5 56                 push esi   

:00404CB6 57                 push edi   

:00404CB7 89C6               mov esi, eax\-----假码进 ESI   

:00404CB9 89D7               mov edi, edx\-----真码进 EDI   

:00404CBB 39D0               cmp eax, edx\-----比较真假码的地址是不是一样   

:00404CBD 0F848F000000       je 00404D52-----一样就玩完   

:00404CC3 85F6               test esi, esi\----假码是否存在   

:00404CC5 7468               je 00404D2F------不存在就玩完   

:00404CC7 85FF               test edi, edi\----真码是否存在   

:00404CC9 746B               je 00404D36------不存在就玩完   

:00404CCB 8B46FC             mov eax, dword ptr \[esi\-04\]--假码位数进 EAX   

:00404CCE 8B57FC             mov edx, dword ptr \[edi\-04\]--真码位树进 EDX   

:00404CD1 29D0               sub eax, edx\----真减假   

:00404CD3 7702               ja 00404CD7-----真大于假则跳   

:00404CD5 01C2               add edx, eax\----假码位数进 EDX   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00404CD3(C)   
|   

:00404CD7 52                 push edx\-------假码位数进见   

:00404CD8 C1EA02             shr edx, 02----假码位数右移两位   

:00404CDB 7426               je 00404D03----小于或等于则跳   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00404CF9(C)   
|   

:00404CDD 8B0E               mov ecx, dword ptr \[esi\]----取假码的 ASCII 码的前 4 位进 ECX   

:00404CDF 8B1F               mov ebx, dword ptr \[edi\]----取真码的 ASCII 码的前 4 位进 EDX   

:00404CE1 39D9               cmp ecx, ebx\-------比较真假 ASCII 码   

:00404CE3 7558               jne 00404D3D-------不等则跳   

:00404CE5 4A                 dec edx\--------真码位数减 1   

:00404CE6 7415               je 00404CFD-----等 0 则跳   

:00404CE8 8B4E04             mov ecx, dword ptr \[esi+04\]---取假码的 ASCII 码的 5 到 8 位进 ECX   

:00404CEB 8B5F04             mov ebx, dword ptr \[edi+04\]---取真码的 ASCII 码的 5 到 8 位进 EDX   

:00404CEE 39D9               cmp ecx, ebx\-------比较真假 ASCII 码   

:00404CF0 754B               jne 00404D3D-------不等则跳   

:00404CF2 83C608             add esi, 00000008----ESI 指向假码第九位   

:00404CF5 83C708             add edi, 00000008----EDI 指向真码第九位   

:00404CF8 4A                 dec edx\------EDX 减 1   

:00404CF9 75E2               jne 00404CDD------不等 0 跳上去再比   

:00404CFB EB06               jmp 00404D03------等 0 跳下去   

* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00404CE6(C)   
|   

:00404CFD 83C604             add esi, 00000004   

:00404D00 83C704             add edi, 00000004   

* Referenced by a (U)nconditional or (C)onditional Jump at Addresses:   
|:00404CDB(C), :00404CFB(U)   
|   

:00404D03 5A                 pop edx\------假码位数出贱   

:00404D04 83E203             and edx, 00000003----EDX\=9 AND 3=1   

:00404D07 7422               je 00404D2B------等 0 则跳   

:00404D09 8B0E               mov ecx, dword ptr \[esi\]----最后一位假码的 ASCII 进 ECX   

:00404D0B 8B1F               mov ebx, dword ptr \[edi\]----最后一位真码的 ASCII 进 EBX   

:00404D0D 38D9               cmp cl, bl\------比较 CL 与 BL   

:00404D0F 7541               jne 00404D52----不等则玩完   

:00404D11 4A                 dec edx\---------EDX 减 1   

:00404D12 7417               je 00404D2B------等于 0 就注册正确了   
```



---------------------------破解人:leozem[YCG],转贴请注名出处.-------------------------------

`4dd466d0e1e0cc123afbe504da2462ca`
