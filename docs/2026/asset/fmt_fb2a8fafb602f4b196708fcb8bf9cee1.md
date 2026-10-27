
# Folder Browser Control v1.0.10注册码的计算 (15千字)

控件名称：Folder Browser

软件授权：共享软件

下载地址：http://www.bitdebris.com/downloads/bdsfbrws.zip

控件简介：From the developer: "When all you want to do is select a folder. The Folder Browser Control provides an interface for selecting a folder to the user of your program. It implements a common feel of the Windows interface, a hierarchical tree structure representing the local and network file system, similar to the one found in the Windows Explorer. The Folder Browser Control acts as a drop-in replacement for the Common Dialog control when all you want to do is to select a folder."

目标程序：`bdsfbrws.ocx`，69632 bytes

破解工具：`W32Dasm` 8.93(打过VB支持补丁)，`SoftICE` 4.05

说明：本人想用VB编一个程序，需要用到只取得文件夹名称的控件，而VB中的Dirlist控件只能显示当前驱动器的文件夹，必须结合DriveList控件，但这样用起来又太不方便了，让人感觉太别扭，于是上网搜索，奋战了3个多小时，终于找到两个满意的，确实比微软的那个强，但都需要注册，前面我说过FolderView 3.0注册部分的计算过程，这次介绍Folder Browser注册部分的计算过程，该控件采用VB 6.0编程，Active编码方式(^\_^，如果是P-Code，我立即只有瞪眼的份)，我没有象FolderView那样立即写出注册部分的计算过程，就是因为它的注册码的计算含有幂和sin的计算，我从“五一”才开始利用一点时间练习用VB编程(我很讨厌VB，代码有臭又长，尤其讨厌该程序至少必须有一个更臭更长的DLL支持库，我希望以后转到DEPHI，我练习用VB编程也是因为现在很多程序都是用VB编的，要想破解容易点儿，了解它所用的编译语言会很有帮助的，当然，我学破解的部分目的是学习软件作者的编程思想)，对于VB编程中幂等浮点的计算的W32Dasm反汇编结果中push参数如何进行没有十分搞清楚(不过现在基本明白了――我是指的非P-Code形式)。另外，该控件用SmartCheck可以看到真正的注册码。好了，闲话少说，现在就说一下它注册部分的计算过程。

* Referenced by a CALL at Addresses:

|:110065E4 , :11006D6A , :1100831E

|

:11007AB0 55
  push ebp
```
:
:
:
:11007B26 8B45E0
mov eax, dword ptr [ebp-20]    ====> RegCode
* Reference To: MSVBVM60.__vbaLenBstr, Ord:0000h
            |
:11007B29 8B1D1C100011            mov ebx, dword
ptr [1100101C]
:11007B2F 50
  push eax
:11007B30 FFD3
  call ebx
:11007B32 8B4DEC
mov ecx, dword ptr [ebp-14]    ====> UserName
:11007B35 8BD0
  mov edx, eax
:11007B37 F7DA
  neg edx
:11007B39 1BD2
  sbb edx, edx
:11007B3B 51
  push ecx
:11007B3C F7DA
  neg edx
:11007B3E 89955CFFFFFF            mov dword ptr
[ebp+FFFFFF5C], edx
:11007B44 FFD3
  call ebx
:11007B46 8B955CFFFFFF            mov edx, dword
ptr [ebp+FFFFFF5C]
:11007B4C F7D8
  neg eax
:11007B4E 1BC0
  sbb eax, eax
:11007B50 F7D8
  neg eax
:11007B52 85D0
  test eax, edx
:11007B54 0F84F1000000            je 11007C4B
:11007B5A 8B45DC
mov eax, dword ptr [ebp-24]    ====> CompanyName
:11007B5D 50
  push eax
:11007B5E FFD3
  call ebx
:11007B60 85C0
  test eax, eax
:11007B62 0F8EA1000000            jle 11007C09
:11007B68 8D5588
lea edx, dword ptr [ebp-78]
:11007B6B 6A01
  push 00000001
  ====> 取一个字符
:11007B6D 8D45C8
lea eax, dword ptr [ebp-38]
:11007B70 8D4DDC
lea ecx, dword ptr [ebp-24]
:11007B73 BE08400000              mov esi,
00004008
:11007B78 52
  push edx
:11007B79 50
  push eax
:11007B7A 894D90
mov dword ptr [ebp-70], ecx
:11007B7D 897588
mov dword ptr [ebp-78], esi
* Reference To: MSVBVM60.rtcLeftCharVar, Ord:0269h
            |
:11007B80 FF1568110011            Call dword ptr
[11001168]      ====> 取单位名左第一个字符
:11007B86 8B4DEC
mov ecx, dword ptr [ebp-14]
:11007B89 8D8568FFFFFF            lea eax, dword
ptr [ebp+FFFFFF68]
:11007B8F 894D80
mov dword ptr [ebp-80], ecx
:11007B92 6A01
  push 00000001
  ====> 取一个字符
:11007B94 8D4DA8
lea ecx, dword ptr [ebp-58]
:11007B97 8D55DC
lea edx, dword ptr [ebp-24]
:11007B9A 50
  push eax
:11007B9B 51
  push ecx
:11007B9C C78578FFFFFF08000000    mov dword ptr [ebp+FFFFFF78], 00000008
:11007BA6 899570FFFFFF            mov dword ptr
[ebp+FFFFFF70], edx
:11007BAC 89B568FFFFFF            mov dword ptr
[ebp+FFFFFF68], esi
* Reference To: MSVBVM60.rtcRightCharVar, Ord:026Bh
            |
:11007BB2 FF1574110011            Call dword ptr
[11001174]      ====> 取单位名右第一个字符
* Reference To: MSVBVM60.__vbaVarCat, Ord:0000h
            |
:11007BB8 8B35E4100011            mov esi, dword
ptr [110010E4]
:11007BBE 8D55C8
lea edx, dword ptr [ebp-38]
:11007BC1 8D8578FFFFFF            lea eax, dword
ptr [ebp+FFFFFF78]
:11007BC7 52
  push edx
:11007BC8 8D4DB8
lea ecx, dword ptr [ebp-48]
:11007BCB 50
  push eax
:11007BCC 51
  push ecx
:11007BCD FFD6
  call esi
    ====> 复制左侧字符
:11007BCF 50
  push eax
:11007BD0 8D55A8
lea edx, dword ptr [ebp-58]
:11007BD3 8D4598
lea eax, dword ptr [ebp-68]
:11007BD6 52
  push edx
:11007BD7 50
  push eax
:11007BD8 FFD6
  call esi
    ====> 复制右侧字符
:11007BDA 50
  push eax
:
:
:
:11007C11 8B55E0
mov edx, dword ptr [ebp-20]    ====> NewRegName
:11007C14 8D4514
lea eax, dword ptr [ebp+14]    ====> eax=0x18DEB=101867
:11007C17 52
  push edx
:11007C18 8D4DE8
lea ecx, dword ptr [ebp-18]    ====> Username
:11007C1B 50
  push eax
:11007C1C 51
  push ecx
:11007C1D E89EFBFFFF              call 110077C0
                ====> 计算注册码
:11007C22 8BD0
  mov edx, eax
:11007C24 8D4DD8
lea ecx, dword ptr [ebp-28]
* Reference To: MSVBVM60.__vbaStrMove, Ord:0000h
            |
:11007C27 FF1570110011            Call dword ptr
[11001170]
:11007C2D 50
  push eax
    ====> SoftICE下d eax即可看到真正注册码
* Reference To: MSVBVM60.__vbaStrCmp, Ord:0000h
            |
:11007C2E FF159C100011            Call dword ptr
[1100109C]      ====> 比较两个注册码是否相等
:11007C34 8BF0
  mov esi, eax
====> 改为xor esi, esi (机器码33F6)即可暴力破解
:
:
:
:11007C94 C3
  ret
* Referenced by a CALL at Address:
|:11007C1D
|
:110077C0 55
  push ebp
:
:
:
:1100780C 8B4508
mov eax, dword ptr [ebp+08]
:1100780F 8B08
  mov ecx, dword ptr [eax]
:11007811 51
  push ecx
* Reference To: MSVBVM60.__vbaLenBstr, Ord:0000h
            |
:11007812 FF151C100011            Call dword ptr
[1100101C]      ====> 计算NewRegName的长度
:11007818 8BC8
  mov ecx, eax
* Reference To: MSVBVM60.__vbaI2I4, Ord:0000h
            |
:1100781A FF15A8100011            Call dword ptr
[110010A8]
* Reference To: MSVBVM60.__vbaStrMove, Ord:0000h
            |
:11007820 8B1D70110011            mov ebx, dword
ptr [11001170]
:11007826 898574FFFFFF            mov dword ptr
[ebp+FFFFFF74], eax
:1100782C BF01000000              mov edi,
00000001
* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:11007A29(U)
|
:11007831 663BBD74FFFFFF          cmp di, word ptr
[ebp+FFFFFF74]
:11007838 0F8FF0010000            jg 11007A2E
:1100783E 8B5508
mov edx, dword ptr [ebp+08]    ====> NewRegName
:11007841 8D4DB8
lea ecx, dword ptr [ebp-48]
:11007844 0FBFC7
movsx eax, di
:11007847 8955A0
mov dword ptr [ebp-60], edx
:1100784A 51
  push ecx
:1100784B 898568FFFFFF            mov dword ptr
[ebp+FFFFFF68], eax
:11007851 50
  push eax
:11007852 8D5598
lea edx, dword ptr [ebp-68]
:11007855 8D45A8
lea eax, dword ptr [ebp-58]
:11007858 52
  push edx
:11007859 50
  push eax
:1100785A C745C001000000          mov [ebp-40], 00000001
:11007861 C745B802000000          mov [ebp-48], 00000002
:11007868 C7459808400000          mov [ebp-68], 00004008
* Reference To: MSVBVM60.rtcMidCharVar, Ord:0278h
            |
:1100786F FF1584100011            Call dword ptr
[11001084]      ====> 从NewRegName中取一个字符
:11007875 8D4DA8
lea ecx, dword ptr [ebp-58]
:11007878 8D55D0
lea edx, dword ptr [ebp-30]
:1100787B 51
  push ecx
:1100787C 52
  push edx
* Reference To: MSVBVM60.__vbaStrVarVal, Ord:0000h
            |
:1100787D FF15E0100011            Call dword ptr
[110010E0]      ====> 转为数值型
:11007883 50
  push eax
* Reference To: MSVBVM60.rtcAnsiValueBstr, Ord:0204h
            |
:11007884 FF1534100011            Call dword ptr
[11001034]      ====> ascii转为字符
:1100788A 8D4DD0
lea ecx, dword ptr [ebp-30]
:1100788D 8BF0
  mov esi, eax
====> 放到esi中
* Reference To: MSVBVM60.__vbaFreeStr, Ord:0000h
            |
:1100788F FF1588110011            Call dword ptr
[11001188]
:11007895 8D45A8
lea eax, dword ptr [ebp-58]
:11007898 8D4DB8
lea ecx, dword ptr [ebp-48]
:1100789B 50
  push eax
:1100789C 51
  push ecx
:1100789D 6A02
  push 00000002
* Reference To: MSVBVM60.__vbaFreeVarList, Ord:0000h
            |
:1100789F FF1528100011            Call dword ptr
```
