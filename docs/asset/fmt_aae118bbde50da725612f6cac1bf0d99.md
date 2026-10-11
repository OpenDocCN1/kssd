
# 珠联跳棋 2.27

> 原文：[`www.pediy.com/kssd/pediy06/pediy6109.htm`](https://www.pediy.com/kssd/pediy06/pediy6109.htm)

## 基本信息

- **作者**：HAILDUZ[CCG]
- **软件名称**：珠联跳棋 享版 2.27
- **整理日期**：2003.6.8
- **文件大小**：178KB
- **软件授权**：共享软件
- **使用平台**：Win9x/Me/NT/2000/XP
- **个人网址**：`http://bmzhao.wx-e.com`

**软件简介**：“跳棋”是一种流行于全国棋类游戏，玩法简单，易于学习。

**加密方式**：注册码

**功能限制**：未注册信息提示

**难度**：容易

**破解工具**：`TRW2000`，`W32Dasm`，`PEiD 0.8`

**作者声明**：只是学习转贴请保持完整。

`PEiD 0.8` 检查可知，VC++ 编写且无壳。`TRW2000` 栽入，`BPX HMEMCPY`，`BC*`，`PMODULE`，`F10` 后：

### Part 1

```asm
* Possible Ref to Menu: MenuID_0080, Item: ""

                          |
:00402FE7 6A01                push 00000001
:00402FE9 E8FF330100           call 004163ED
:00402FEE 8D9E9C000000         lea ebx, dword ptr [esi+0000009C]  /停在这里，（d*ebx=输入注册码）
:00402FF4 8BCB                mov ecx, ebx
:00402FF6 E86FFF0000          call 00412F6A
:00402FFB 6A00                push 00000000
:00402FFD 8BCB                mov ecx, ebx
:00402FFF E8CFFE0000          call 00412ED3
:00403004 6A0A                push 0000000A
:00403006 8BF8                mov edi, eax
:00403008 6A00                push 00000000
:0040300A 6A00                push 00000000
:0040300C 8D44241C            lea eax, dword ptr [esp+1C]
:00403010 6A00                push 00000000
:00403012 50                  push eax
:00403013 6A0C                push 0000000C
:00403015 6A00                push 00000000

* Possible StringData Ref from Data Obj ->"c:\\"

                          |
:00403017 6888114200          push 00421188

* Reference To: KERNEL32.GetVolumeInformationA, Ord:0177h      /取硬盘 C:的信息。

                          |
:0040301C FF1560B14100        Call dword ptr [0041B160]
:00403022 8B4C2410            mov ecx, dword ptr [esp+10]  /ecx=硬盘 c:的 serial number
:00403026 85C9                test ecx, ecx
:00403028 752F                jne 00403059

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00403057(C)
|
:0040302A 8BC1                mov eax, ecx
:0040302C 33D2                xor edx, edx
:0040302E BD1A000000          mov ebp, 0000001A       
:00403033 F7F5                div ebp                        /硬盘 serial number 除以 0x1a   
:00403035 0FBE07             movsx eax, byte ptr [edi]         /依次取取注册码位 1 位....进 eax
:00403038 8B1495C0104200      mov edx, dword ptr [4*edx+004210C0]/依据前面余数查表（见 part2)得到检验初码
:0040303F 83C241              add edx, 00000041                /检验初码加 0x41 得到检验码
:00403042 47                  inc edi
:00403043 3BD0                cmp edx, eax                       /比较注册码与检验码
:00403045 7559                jne 004030A0   //不等则失败   
:00403047 B84FECC44E          mov eax, 4EC4EC4F
:0040304C F7E1                mul ecx
:0040304E C1EA03              shr edx, 03
:00403051 8BCA                mov ecx, edx
:00403053 894C2410            mov dword ptr [esp+10], ecx        /还原上面的商
:00403057 75D1                jne 0040302A                      /商为 0 结束成功

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00403028(C)
|
:00403059 E841650100          call 0041959F
:0040305E 8B0B                mov ecx, dword ptr [ebx]
:00403060 8B1524294200        mov edx, dword ptr [00422924]
:00403066 8B4004              mov eax, dword ptr [eax+04]
:00403069 51                  push ecx
:0040306A 8B0D28294200        mov ecx, dword ptr [00422928]
:00403070 52                  push edx
:00403071 51                  push ecx
:00403072 8BC8                mov ecx, eax
:00403074 E888560100          call 00418701
:00403079 6A00                push 00000000

* Possible StringData Ref from Data Obj ->"注册成功"

                          |
:0040307B 6850124200          push 00421250

* Possible StringData Ref from Data Obj ->"恭喜你，注册成功!"

                          |
:00403080 683C124200          push 0042123C
:00403085 8BCE                mov ecx, esi
:00403087 E8052C0100          call 00415C91
:0040308C 8BCE                mov ecx, esi

* Possible Ref to Menu: MenuID_0080, Item: ""

                          |
:0040308E C7465C01000000      mov [esi+5C], 00000001
:00403095 E82C110100          call 004141C6
:0040309A 5F                  pop edi
:0040309B 5E                  pop esi
:0040309C 5D                  pop ebp
:0040309D 5B                  pop ebx
:0040309E 59                  pop ecx
:0040309F C3                  ret

* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:00403045(C)
|
:004030A0 6A00                push 00000000

* Possible StringData Ref from Data Obj ->"注册失败"

                          |
:004030A2 6830124200          push 00421230

* Possible StringData Ref from Data Obj ->"注册码不对!"

                          |
:004030A7 6824124200          push 00421224
:004030AC 8BCE                mov ecx, esi
:004030AE E8DE2B0100          call 00415C91
:004030B3 8BCE                mov ecx, esi
:004030B5 E80C110100          call 004141C6
:004030BA 5F                  pop edi
:004030BB 5E                  pop esi
:004030BC 5D                  pop ebp
:004030BD 5B                  pop ebx
:004030BE 59                  pop ecx
:004030BF C3                  ret
```

### Part 2

**数据表**

```text
0030:004210C0h:0C 00 00 00 10 00 00 00 05 00 00 00 11 00 00 00
0030:004210D0h:06 00 00 00 15 00 00 00 19 00 00 00 09 00 00 00
0030:004210E0h:03 00 00 00 0F 00 00 00 0B 00 00 00 18 00 00 00
0030:004210F0h:0A 00 00 00 04 00 00 00 0E 00 00 00 02 00 00 00
0030:00421100h:17 00 00 00 08 00 00 00 01 00 00 00 16 00 00 00
0030:00421110h:12 00 00 00 13 00 00 00 07 00 00 00 00 00 00 00
0030:00421120h:0D 00 00 00 14
```

### 小结

该软件是将用户的 `c:` 盘序列号作为参数计算并查表得到检验码，逐位与注册码比较。

HAILDUZ[CCG] 2003.6.8

HAILDUZ@HOTMAIL.COM

`4dd466d0e1e0cc123afbe504da2462ca`
