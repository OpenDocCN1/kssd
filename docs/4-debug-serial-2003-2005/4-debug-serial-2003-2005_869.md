
# 环球卫星网络电视(网络加速版) 算法分析

> 原文：`www.pediy.com/kssd/pediy07/pediy7-164.htm`

*   软件版本 `V7.9`
*   软件类别 视频工具
*   软件性质 共享
*   文件大小 `2.25M`
*   系统平台 `Win9x/2K/XP/2003`
*   界面预览 有
*   整理日期 `2005-03-11`
*   软件评价 `★ ★ ★ ★ ★`
*   下载地址：`http://download.21cn.com/list.php?id=46540`

## 软件简介

环球卫星网络电视软件是一款集网络电视收看、网络电台收听，以及本地音视频播放为一体的休闲娱乐软件。环球卫星网络电视软件吸取其他网络电视精华，抛弃其缺点，收视率高，清晰，流畅，是其它软件不及的一处大优点。使用环球卫星网络电视软件，您可收看到国内外近 300 个电视台，其中包括很多本地有线电视都没有的港台、国外的精彩电视台，收听 200 百个广播电台，精彩的电视片断，明星写真，此外还可以用它来播放本地硬盘里的歌曲，每周更新 1 次，自动升级。

用 `PEID` 查看无壳，`DELPHI` 编写的程序，一般 `DELPHI` 编写的程序我喜欢先看字串有没有发现，好了 `OD` 装入字串查找 `'注册码无效'`

找到关键字串双击在无效提示的前面下断，开始分析

```assembly
004CB9F3   >  8D55 F0      LEA EDX,DWORD PTR SS:[EBP-10]
004CB9F6   .  8B45 FC      MOV EAX,DWORD PTR SS:[EBP-4]
004CB9F9   .  8B80 0003000>MOV EAX,DWORD PTR DS:[EAX+300]
004CBA04   .  E8 1476F7FF  CALL 环球卫星.00443018
004CBA08   .  74 1E        JE SHORT 环球卫星.004CBA28        ; 跳向注册码无效
004CBA0A   .  8D55 EC      LEA EDX,DWORD PTR SS:[EBP-14]
004CBA0D   .  8B45 FC      MOV EAX,DWORD PTR SS:[EBP-4]    ; 0188e42c 入 EAX
004CBA10   .  8B80 0003000>MOV EAX,DWORD PTR DS:[EAX+300]  ; 01889AE0 入 EAX
004CBA16   .  E8 FD75F7FF  CALL 环球卫星.00443018
004CBA1B   .  8B45 EC      MOV EAX,DWORD PTR SS:[EBP-14]   ; 假码的地址给 EAX
004CBA1E   .  E8 BD89F3FF  CALL 环球卫星.004043E0
004CBA23   .  83F8 08      CMP EAX,8                         ; 注册码位数跟 8 比
004CBA26   .  7E 30        JLE SHORT 环球卫星.004CBA58       ; 注册码必需小于等于 8 就跳
004CBA28   >  B8 20BD4C00  MOV EAX,环球卫星.004CBD20
004CBA2D   .  E8 B631F6FF  CALL 环球卫星.0042EBE8
004CBA32   .  8B45 FC      MOV EAX,DWORD PTR SS:[EBP-4]
004CBA35   .  8B80 0003000>MOV EAX,DWORD PTR DS:[EAX+300]
004CBA3B   .  33D2         XOR EDX,EDX
004CBA3D   .  E8 0676F7FF  CALL 环球卫星.00443048
004CBA42   .  8B45 FC      MOV EAX,DWORD PTR SS:[EBP-4]
004CBA45   .  8B80 0003000>MOV EAX,DWORD PTR DS:[EAX+300]
004CBA4B   .  8B10         MOV EDX,DWORD PTR DS:[EAX]
004CBA4D   .  FF92 C400000>CALL DWORD PTR DS:[EDX+C4]
004CBA53   .  E9 23020000  JMP 环球卫星.004CBC7B
004CBA58   >  8D45 E4      LEA EAX,DWORD PTR SS:[EBP-1C]
004CBA5B   .  50           PUSH EAX
004CBA5C   .  8D55 E0      LEA EDX,DWORD PTR SS:[EBP-20]
004CBA5F   .  8B45 FC      MOV EAX,DWORD PTR SS:[EBP-4]
004CBA62   .  8B80 0803000>MOV EAX,DWORD PTR DS:[EAX+308]
004CBA68   .  E8 AB75F7FF  CALL 环球卫星.00443018
004CBA6D   .  8B45 E0      MOV EAX,DWORD PTR SS:[EBP-20]   ; 机器码入 EAX
004CBA70   .  B9 06000000  MOV ECX,6
004CBA75   .  BA 01000000  MOV EDX,1
004CBA7A   .  E8 C18BF3FF  CALL 环球卫星.00404640          ; 取机器码前六位
004CBA7F   .  8B4D E4      MOV ECX,DWORD PTR SS:[EBP-1C]   ; 机器码前 6 位入 ECX
004CBA82   .  8D45 E8      LEA EAX,DWORD PTR SS:[EBP-18]
004CBA85   .  BA 38BD4C00  MOV EDX,环球卫星.004CBD38        ; ASCII "0x"
004CBA8A   .  E8 9D89F3FF  CALL 环球卫星.0040442C          ; 在机器码前六位的前面加 0x 我们称为 a
004CBA8F   .  8B45 E8      MOV EAX,DWORD PTR SS:[EBP-18]   ; a 给 EAX
004CBA92   .  E8 D1D3F3FF  CALL 环球卫星.00408E68
004CBA97   .  8BF0         MOV ESI,EAX
004CBA99   .  33C0         XOR EAX,EAX
004CBA9B   .  55           PUSH EBP
004CBA9C   .  68 32BC4C00  PUSH 环球卫星.004CBC32
004CBAA1   .  64:FF30      PUSH DWORD PTR FS:[EAX]
004CBAA4   .  64:8920      MOV DWORD PTR FS:[EAX],ESP
004CBAA7   .  8D55 DC      LEA EDX,DWORD PTR SS:[EBP-24]
004CBAAA   .  8B45 FC      MOV EAX,DWORD PTR SS:[EBP-4]
004CBAAD   .  8B80 0003000>MOV EAX,DWORD PTR DS:[EAX+300]
004CBAB3   .  E8 6075F7FF  CALL 环球卫星.00443018
004CBAB8   .  8B45 DC      MOV EAX,DWORD PTR SS:[EBP-24]   ; 假注册码入 EAX
004CBABB   .  E8 A8D3F3FF  CALL 环球卫星.00408E68          ; 把注册码转为十六进制的 CALL 称为 b
004CBAC0   .  8BD8         MOV EBX,EAX                      ; B 给 EBX
004CBAC2   .  8BC3         MOV EAX,EBX                      ; EBX 给 EAX
004CBAC4   .  2BC6         SUB EAX,ESI                      ; 注册码的十六进制-机器码前 6 位
004CBAC6   .  3B05 B4F34D0>CMP EAX,DWORD PTR DS:[4DF3B4]   ; 值等于 000C0F1E（790302）可能是作者生日就成功
004CBACC   .  74 59        JE SHORT 环球卫星.004CBB27       ; 等于就正确了
```

## 总结

这个软件算法很简单。

注册码的十六进制 - 机器码的前六位 = `000C0F1E` (也就是 `790302`) 就成功了。

如机器码为 `BDC4E5E0`。

找出对应的注册码：`BDC4E5 + 000C0F1E` = (注册码的十六进制)，把这转成十进制就是真的注册码。

我的机器码为 `BDC4E5E0`，对应的注册码为 `13227011`。

`4dd466d0e1e0cc123afbe504da2462ca`
