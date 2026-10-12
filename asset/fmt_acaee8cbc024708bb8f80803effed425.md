
# RegEditor V1.2

> 原文：[`www.pediy.com/kssd/pediy03/forum790.htm`](https://www.pediy.com/kssd/pediy03/forum790.htm)

软件名称：`RegEditor V1.2`

软件分类：`DOWNME` 首页=>软件下载=>系统工具=>注册表管理

软件设计：`A. Chabanenko.`

软件主页：`http://www.utils32.com/products.htm`

联系邮件：`nit@aip.mk.ua`

授权方式：共享软件

更新时间：`2001-04-05`

操作系统：`Windows 95/98/NT/2000`

试用限制：`30` 天

购买金额：`$20.00`

软件大小：`451KB`

星级评定：★★★★★

图片预览：点击这里查看软件图片

软件简介：`RegEditor` 是一个注册表编辑工具，它可以用于 `Registry 4 (REGEDIT4)` 的文件格式对 `Windows 95/98` 的注册表进行修改。`RegEditor` 还支持多语种的系统。

`http://www.downme.com/download/3864regeditr.zip`

------------------------------------------------------------------------------------

这些过程是总结小弟这几年来的 `Cracker` 经验之谈!也是最完整一篇了!

## 破解密码篇

谁说明破解没有直径？不过不叫直径而叫经验罢了？

只要你多看一些别人的经验有“拿来主义”总结分析

得到变成自己的新经验就是直径了：

### 旧方法

`F12`---->`17` 次

`F8`----->`15` 次

结果:`32` 次

### 常用法

`pmodule`

`F10`----->`72` 次

结果:`72` 次

### 个人法

`pmodule`

`F12`---->`10` 次

`F10`---->`8` 次

结果:`18` 次

就能来到这个错误 call!

```
0167:0047A64F  CALL    00454F50
0167:0047A654  TEST    AL,AL
0167:0047A656  JZ      0047A681
0167:0047A658  MOV     EAX,[EBP-04]
0167:0047A65B  CALL    004701DC//------->关键 call!
0167:0047A660  TEST    AL,AL
0167:0047A662  JZ      0047A677
0167:0047A664  MOV     EAX,0047A6EC
0167:0047A669  CALL    00454F2C
0167:0047A66E  MOV     EAX,EBX
0167:0047A670  CALL    0044AF84
0167:0047A675  JMP     SHORT 0047A681
0167:0047A677  MOV     EAX,0047A710
0167:0047A67C  CALL    00454F2C//--------->出来错误提示！
```

看到没有我个人方法多么的快？我一向破解也是采用这样的方法，

只要 `pmodule` 能用，把旧方法和常用法方法结合就是一种新方法也是新一种经验?

为什么还要用 `F12` 了，小弟没有什么编程的技术只是在学校学过 `Foxbase2.5`！

`ret` 是主要流程，也是主要的得的干骨！什么还不明白？

就用一颗树干，你从这里开始找就能快速的找到枝叶（分支流程）！

哪如何找到树干了？你把要用 `F12` 了记一下出现第一次提示的次数

-1 再用 `F10` 就找到枝叶！小弟用此方法不到一分钟就把注册码破解出来！

------------------------------------------------------------------------------------

```
:004701DC 55  push ebp//------------->1
```

------------------------------------------------------------------------------------

`EAX=00C0A168=78787878`

------------------------------------------------------------------------------------

```
:00403ED8 53  push ebx
:00403ED9 56  push esi
:00403EDA 57  push edi
:00403EDB 89C6  mov esi, eax
:00403EDD 89D7  mov edi, edx
:00403EDF 39D0  cmp eax, edx//-------->2
```

------------------------------------------------------------------------------------

`SN=EAX=61C8-064B-FE37`

`EDI=EDX=00C0A168=78787878`

------------------------------------------------------------------------------------

```
:00403EE1 0F848F000000  je 00403F76
:00403EE7 85F6  test esi, esi
:00403EE9 7468  je 00403F53
:00403EEB 85FF  test edi, edi
:00403EED 746B  je 00403F5A
:00403EEF 8B46FC  mov eax, dword ptr [esi-04]
:00403EF2 8B57FC  mov edx, dword ptr [edi-04]
:00403EF5 29D0  sub eax, edx
:00403EF7 7702  ja 00403EFB
:00403EF9 01C2  add edx, eax
:004701DD 8BEC  mov ebp, esp
:004701DF 83C4EC  add esp, FFFFFFEC
:004701E2 33D2  xor edx, edx
:004701E4 8955F0  mov dword ptr [ebp-10], edx
:004701E7 8955EC  mov dword ptr [ebp-14], edx
:004701EA 8945FC  mov dword ptr [ebp-04], eax
:004701ED 8B45FC  mov eax, dword ptr [ebp-04]
:004701F0 E8873DF9FF  call 00403F7C
```

------------------------------------------------------------------------------------

```
:0047020E 8D55F0  lea edx, dword ptr [ebp-10]//----->3
```

`ID=EAX=00CF4F8=6600-0183-F9FF`

`ss:[6DF364]=00C0F5B4=SN=61C8-064B-FE37`

------------------------------------------------------------------------------------

```
:0047020E 8D55F0  lea edx, dword ptr [ebp-10]
:00470211 E8D2FBFFFF  call 0046FDE8
:00470216 8B45F0  mov eax, dword ptr [ebp-10]//------>4
```

------------------------------------------------------------------------------------

`ID=EDX=00CF4F8=6600-0183-F9FF`

`ss:[6DF364]=00C0F5B4=SN=61C8-064B-FE37`

------------------------------------------------------------------------------------

```
:00470219 8B55FC  mov edx, dword ptr [ebp-04]//------>5
```

------------------------------------------------------------------------------------

`SN=EAX=61C8-064B-FE37`

`ss:[6DF370]=00C0A168=78787878`

------------------------------------------------------------------------------------

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004701A4(C)
|
:0047021C E8B73CF9FF  call 00403ED8//----------->6
```

------------------------------------------------------------------------------------

`EDX=00C0A168=78787878`

`EAX=00C0F5B4=SN=61C8-064B-FE37`

--------------------------------------------------------------------------------

```
:00470221 0F9445FB  sete byte ptr [ebp-05]
:00470225 807DFB00  cmp byte ptr [ebp-05], 00
:00470229 7469  je 00470294
:0047022B B201  mov dl, 01
:0047022D A1C0EE4600  mov eax, dword ptr [0046EEC0]
:00470232 E889EDFFFF  call 0046EFC0
:00470237 8945F4  mov dword ptr [ebp-0C], eax
:0047023A 33C0  xor eax, eax
:0047023C 55  push ebp
:0047023D 688D024700  push 0047028D
:00470242 64FF30  push dword ptr fs:[eax]
:00470245 648920  mov dword ptr fs:[eax], esp
:00470248 BA02000080  mov edx, 80000002
:0047024D 8B45F4  mov eax, dword ptr [ebp-0C]
:00470250 E80BEEFFFF  call 0046F060
:00470255 33C9  xor ecx, ecx
:00470257 BAD0024700  mov edx, 004702D0
:0047025C 8B45F4  mov eax, dword ptr [ebp-0C]
:0047025F E864EEFFFF  call 0046F0C8
:00470264 84C0  test al, al
:00470266 740F  je 00470277
:00470268 B920034700  mov ecx, 00470320
:0047026D 33D2  xor edx, edx
:0047026F 8B45F4  mov eax, dword ptr [ebp-0C]
:00470272 E845F4FFFF  call 0046F6BC
:00470211 E8D2FBFFFF  call 0046FDE8
:00470216 8B45F0  mov eax, dword ptr [ebp-10]
:00470219 8B55FC  mov edx, dword ptr [ebp-04]
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:004701A4(C)
|
:0047021C E8B73CF9FF  call 00403ED8
:00470221 0F9445FB  sete byte ptr [ebp-05]
:00470225 807DFB00  cmp byte ptr [ebp-05], 00
:00470229 7469  je 00470294
:0047022B B201  mov dl, 01
:0047022D A1C0EE4600  mov eax, dword ptr [0046EEC0]
:00470232 E889EDFFFF  call 0046EFC0
:00470237 8945F4  mov dword ptr [ebp-0C], eax
:0047023A 33C0  xor eax, eax
:0047023C 55  push ebp
:0047023D 688D024700  push 0047028D
:00470242 64FF30  push dword ptr fs:[eax]
:00470245 648920  mov dword ptr fs:[eax], esp
:00470248 BA02000080  mov edx, 80000002
:0047024D 8B45F4  mov eax, dword ptr [ebp-0C]
:00470250 E80BEEFFFF  call 0046F060
:00470255 33C9  xor ecx, ecx
:00470257 BAD0024700  mov edx, 004702D0
:0047025C 8B45F4  mov eax, dword ptr [ebp-0C]
```
