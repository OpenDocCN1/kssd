
# Folder Browser Control v1.0.10

> 原文：[`www.pediy.com/kssd/pediy03/forum1045.htm`](https://www.pediy.com/kssd/pediy03/forum1045.htm)

控件名称：`Folder Browser`

软件授权：共享软件

下载地址：`http://www.bitdebris.com/downloads/bdsfbrws.zip`

控件简介：From the developer: "When all you want to do is select a folder. The `Folder Browser Control` provides an interface for selecting a folder to the user of your program. It implements a common feel of the Windows interface, a hierarchical tree structure representing the local and network file system, similar to the one found in the Windows Explorer. The `Folder Browser Control` acts as a drop-in replacement for the Common Dialog control when all you want to do is to select a folder."

目标程序：`bdsfbrws.ocx`，69632 bytes

破解工具：`W32Dasm` 8.93(打过 VB 支持补丁)，`SoftICE` 4.05

说明：本人想用 VB 编一个程序，需要用到只取得文件夹名称的控件，而 VB 中的 `Dirlist` 控件只能显示当前驱动器的文件夹，必须结合 `DriveList` 控件，但这样用起来又太不方便了，让人感觉太别扭，于是上网搜索，奋战了 3 个多小时，终于找到两个满意的，确实比微软的那个强，但都需要注册，前面我说过 `FolderView` 3.0 注册部分的计算过程，这次介绍 `Folder Browser` 注册部分的计算过程，该控件采用 VB 6.0 编程，Active 编码方式(^\_^，如果是 P-Code，我立即只有瞪眼的份)，我没有象 `FolderView` 那样立即写出注册部分的计算过程，就是因为它的注册码的计算含有幂和 sin 的计算，我从“五一”才开始利用一点时间练习用 VB 编程(我很讨厌 VB，代码有臭又长，尤其讨厌该程序至少必须有一个更臭更长的 DLL 支持库，我希望以后转到 DEPHI，我练习用 VB 编程也是因为现在很多程序都是用 VB 编的，要想破解容易点儿，了解它所用的编译语言会很有帮助的，当然，我学破解的部分目的是学习软件作者的编程思想)，对于 VB 编程中幂等浮点的计算的 `W32Dasm` 反汇编结果中 push 参数如何进行没有十分搞清楚(不过现在基本明白了——我是指的非 P-Code 形式)。另外，该控件用 `SmartCheck` 可以看到真正的注册码。好了，闲话少说，现在就说一下它注册部分的计算过程。

\* Referenced by a CALL at Addresses:

```
|:110065E4  , :11006D6A  , :1100831E  

|   

:11007AB0 55                  push ebp   

:   

:   

:   

:11007B26 8B45E0              mov eax, dword ptr [ebp-20]     ====> RegCode   

\* Reference To: MSVBVM60.__vbaLenBstr, Ord:0000h   

                          |   

:11007B29 8B1D1C100011       mov ebx, dword ptr [1100101C]   

:11007B2F 50                push eax   

:11007B30 FFD3             call ebx   

:11007B32 8B4DEC           mov ecx, dword ptr [ebp-14]     ====> UserName   

:11007B35 8BD0             mov edx, eax   

:11007B37 F7DA             neg edx   

:11007B39 1BD2             sbb edx, edx   

:11007B3B 51               push ecx   

:11007B3C F7DA             neg edx   

:11007B3E 89955CFFFFFF     mov dword ptr [ebp+FFFFFF5C], edx   

:11007B44 FFD3             call ebx   

:11007B46 8B955CFFFFFF     mov edx, dword ptr [ebp+FFFFFF5C]   

:11007B4C F7D8             neg eax   

:11007B4E 1BC0             sbb eax, eax   

:11007B50 F7D8             neg eax   

:11007B52 85D0             test eax, edx   

:11007B54 0F84F1000000     je 11007C4B   

:11007B5A 8B45DC           mov eax, dword ptr [ebp-24]     ====> CompanyName   

:11007B5D 50               push eax   

:11007B5E FFD3             call ebx   

:11007B60 85C0             test eax, eax   

:11007B62 0F8EA1000000     jle 11007C09   

:11007B68 8D5588           lea edx, dword ptr [ebp-78]   

:11007B6B 6A01            push 00000001                     ====> 取一个字符   

:11007B6D 8D45C8           lea eax, dword ptr [ebp-38]   

:11007B70 8D4DDC           lea ecx, dword ptr [ebp-24]   

:11007B73 BE08400000       mov esi, 00004008   

:11007B78 52               push edx   

:11007B79 50               push eax   

:11007B7A 894D90           mov dword ptr [ebp-70], ecx   

:11007B7D 897588           mov dword ptr [ebp-78], esi   

\* Reference To: MSVBVM60.rtcLeftCharVar, Ord:0269h   

                          |   

:11007B80 FF1568110011     Call dword ptr [11001168]       ====> 取单位名左第一个字符   

:11007B86 8B4DEC           mov ecx, dword ptr [ebp-14]   

:11007B89 8D8568FFFFFF     lea eax, dword ptr [ebp+FFFFFF68]   

:11007B8F 894D80           mov dword ptr [ebp-80], ecx   

:11007B92 6A01            push 00000001                     ====> 取一个字符   

:11007B94 8D4DA8           lea ecx, dword ptr [ebp-58]   

:11007B97 8D55DC           lea edx, dword ptr [ebp-24]   

:11007B9A 50               push eax   

:11007B9B 51               push ecx   

:11007B9C C78578FFFFFF08000000  mov dword ptr [ebp+FFFFFF78], 00000008   

:11007BA6 899570FFFFFF     mov dword ptr [ebp+FFFFFF70], edx   

:11007BAC 89B568FFFFFF     mov dword ptr [ebp+FFFFFF68], esi   

\* Reference To: MSVBVM60.rtcRightCharVar, Ord:026Bh   

                          |   

:11007BB2 FF1574110011     Call dword ptr [11001174]       ====> 取单位名右第一个字符   

\* Reference To: MSVBVM60.__vbaVarCat, Ord:0000h   

                          |   

:11007BB8 8B35E4100011     mov esi, dword ptr [110010E4]   

:11007BBE 8D55C8           lea edx, dword ptr [ebp-38]   

:11007BC1 8D8578FFFFFF     lea eax, dword ptr [ebp+FFFFFF78]   

:11007BC7 52               push edx   

:11007BC8 8D4DB8           lea ecx, dword ptr [ebp-48]   

:11007BCB 50               push eax   

:11007BCC 51               push ecx   

:11007BCD FFD6             call esi                          ====> 复制左侧字符   

:11007BCF 50               push eax   

:11007BD0 8D55A8           lea edx, dword ptr [ebp-58]   

:11007BD3 8D4598           lea eax, dword ptr [ebp-68]   

:11007BD6 52               push edx   

:11007BD7 50               push eax   

:11007BD8 FFD6             call esi                          ====> 复制右侧字符   

:11007BDA 50               push eax   

:   

:   

:   

:11007C11 8B55E0           mov edx, dword ptr [ebp-20]     ====> NewRegName   

:11007C14 8D4514           lea eax, dword ptr [ebp+14]     ====> eax=0x18DEB=101867   

:11007C17 52               push edx   

:11007C18 8D4DE8           lea ecx, dword ptr [ebp-18]     ====> Username   

:11007C1B 50               push eax   

:11007C1C 51               push ecx   

:11007C1D E89EFBFFFF       call 110077C0                    ====> 计算注册码   

:11007C22 8BD0             mov edx, eax   

:11007C24 8D4DD8           lea ecx, dword ptr [ebp-28]   

\* Reference To: MSVBVM60.__vbaStrMove, Ord:0000h   

                          |   

:11007C27 FF1570110011     Call dword ptr [11001170]   

:11007C2D 50               push eax                          ====> SoftICE 下 d eax 即可看到真正注册码   

\* Reference To: MSVBVM60.__vbaStrCmp, Ord:0000h   

                          |   

:11007C2E FF159C100011     Call dword ptr [1100109C]       ====> 比较两个注册码是否相等   

:11007C34 8BF0             mov esi, eax                      ====> 改为 xor esi, esi (机器码 33F6)即可暴力破解   

:   

:   

:   

:11007C94 C3                ret   

\* Referenced by a CALL at Address:   

|:11007C1D  

|   

:110077C0 55                  push ebp   

:   

:   

:   

:1100780C 8B4508              mov eax, dword ptr [ebp+08]   

:1100780F 8B08                mov ecx, dword ptr [eax]   

:11007811 51                push ecx   

\* Reference To: MSVBVM60.__vbaLenBstr, Ord:0000h   

                          |   

:11007812 FF151C100011       Call dword ptr [1100101C]       ====> 计算 NewRegName 的长度   

:11007818 8BC8              mov ecx, eax   

\* Reference To: MSVBVM60.__vbaI2I4, Ord:0000h   

                          |   

:1100781A FF15A8100011       Call dword ptr [110010A8]   

\* Reference To: MSVBVM60.__vbaStrMove, Ord:0000h   

                          |   

:11007820 8B1D70110011       mov ebx, dword ptr [11001170]   

:11007826 898574FFFFFF       mov dword ptr [ebp+FFFFFF74], eax   

:1100782C BF01000000         mov edi, 00000001   
```



# 汇编代码分析

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| `:11007A29(U)`

```
:11007831 663BBD74FFFFFF             cmp di, word ptr [ebp+FFFFFF74]
:11007838 0F8FF0010000               jg 11007A2E

:1100783E 8B5508                     mov edx, dword ptr [ebp+08]     ====> NewRegName
:11007841 8D4DB8                     lea ecx, dword ptr [ebp-48]
:11007844 0FBFC7                     movsx eax, di
:11007847 8955A0                     mov dword ptr [ebp-60], edx
:1100784A 51                         push ecx
:1100784B 898568FFFFFF               mov dword ptr [ebp+FFFFFF68], eax
:11007851 50                         push eax
:11007852 8D5598                     lea edx, dword ptr [ebp-68]
:11007855 8D45A8                     lea eax, dword ptr [ebp-58]
:11007858 52                         push edx
:11007859 50                         push eax
:1100785A C745C001000000             mov [ebp-40], 00000001
:11007861 C745B802000000             mov [ebp-48], 00000002
:11007868 C7459808400000             mov [ebp-68], 00004008
```

\* Reference To: `MSVBVM60.rtcMidCharVar`, Ord:0278h

|

```
:1100786F FF1584100011               Call dword ptr [11001084]       ====> 从 NewRegName 中取一个字符
:11007875 8D4DA8                     lea ecx, dword ptr [ebp-58]
:11007878 8D55D0                     lea edx, dword ptr [ebp-30]
:1100787B 51                         push ecx
:1100787C 52                         push edx
```

\* Reference To: `MSVBVM60.__vbaStrVarVal`, Ord:0000h

|

```
:1100787D FF15E0100011               Call dword ptr [110010E0]       ====> 转为数值型
:11007883 50                         push eax
```

\* Reference To: `MSVBVM60.rtcAnsiValueBstr`, Ord:0204h

|

```
:11007884 FF1534100011               Call dword ptr [11001034]       ====> ascii 转为字符
:1100788A 8D4DD0                     lea ecx, dword ptr [ebp-30]
:1100788D 8BF0                       mov esi, eax                   ====> 放到 esi 中
```

\* Reference To: `MSVBVM60.__vbaFreeStr`, Ord:0000h

|

```
:1100788F FF1588110011               Call dword ptr [11001188]
:11007895 8D45A8                     lea eax, dword ptr [ebp-58]
:11007898 8D4DB8                     lea ecx, dword ptr [ebp-48]
:1100789B 50                         push eax
:1100789C 51                         push ecx
:1100789D 6A02                       push 00000002
```

\* Reference To: `MSVBVM60.__vbaFreeVarList`, Ord:0000h

|

```
:1100789F FF1528100011               Call dword ptr [11001028]
:110078A5 8B5508                     mov edx, dword ptr [ebp+08]
:110078A8 83C40C                     add esp, 0000000C
:110078AB DB8568FFFFFF               fild dword ptr [ebp+FFFFFF68]  ====> st0=1
:110078B1 8B02                       mov eax, dword ptr [edx]
:110078B3 50                         push eax
:110078B4 DD9D60FFFFFF               fstp qword ptr [ebp+FFFFFF60]
```

\* Reference To: `MSVBVM60.__vbaLenBstr`, Ord:0000h

|

```
:110078BA FF151C100011               Call dword ptr [1100101C]       ====> NewRegName 的长度
:110078C0 89855CFFFFFF               mov dword ptr [ebp+FFFFFF5C], eax
:110078C6 8B4D0C                     mov ecx, dword ptr [ebp+0C]    ====> ecx=0x18DEB
:110078C9 DB855CFFFFFF               fild dword ptr [ebp+FFFFFF5C]
:110078CF 83EC08                     sub esp, 00000008
:110078D2 DD9D54FFFFFF               fstp qword ptr [ebp+FFFFFF54]  ====> 长度放到此
:110078D8 DD8560FFFFFF               fld qword ptr [ebp+FFFFFF60]   ====> st0=1
:110078DE 833D00C0001100             cmp dword ptr [1100C000], 00000000
:110078E5 7508                       jne 110078EF
:110078E7 DCB554FFFFFF               fdiv qword ptr [ebp+FFFFFF54]  ====> 1/len(NewRegName)
:110078ED EB11                       jmp 11007900
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| `:110078E5(C)`

```
:110078EF FFB558FFFFFF               push dword ptr [ebp+FFFFFF58]
:110078F5 FFB554FFFFFF               push dword ptr [ebp+FFFFFF54]
```

\* Reference To: `MSVBVM60._adj_fdiv_m64`, Ord:0000h

|

```
:110078FB E8D49AFFFF                 Call 110013D4
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| `:110078ED(U)`

```
:11007900 DFE0                       fstsw ax
:11007902 A80D                       test al, 0D
:11007904 0F858F010000               jne 11007A99
:1100790A DD1C24                     fstp qword ptr [esp]           ====> 作为幂
:1100790D DB01                       fild dword ptr [ecx]           ====> [ecx]=18DEB=101867
:1100790F DD9D4CFFFFFF               fstp qword ptr [ebp+FFFFFF4C]
:11007915 8B9550FFFFFF               mov edx, dword ptr [ebp+FFFFFF50]
:1100791B 8B854CFFFFFF               mov eax, dword ptr [ebp+FFFFFF4C]
:11007921 52                         push edx
:11007922 50                         push eax
```

\* Reference To: `MSVBVM60.__vbaPowerR8`, Ord:0000h

|

```
:11007923 FF1524110011               Call dword ptr [11001124]      ====> 101867 的幂
:11007929 DD0578120011               fld qword ptr [11001278]       ====> st0=3.14159265359(用 pi 表示)
:1100792F 833D00C0001100             cmp dword ptr [1100C000], 00000000
:11007936 7508                       jne 11007940
:11007938 DC3570120011               fdiv qword ptr [11001270]      ====> pi/180
:1100793E EB11                       jmp 11007951
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| `:11007936(C)`

```
:11007940 FF3574120011               push dword ptr [11001274]
:11007946 FF3570120011               push dword ptr [11001270]
```

\* Reference To: `MSVBVM60._adj_fdiv_m64`, Ord:0000h

|

```
:1100794C E8839AFFFF                 Call 110013D4
```

\* Referenced by a (U)nconditional or (C)onditional Jump at Address:

| `:1100793E(U)`

```
:11007951 83EC08                     sub esp, 00000008
:11007954 DEC9                       fmulp st(1), st(0)
:11007956 DFE0                       fstsw ax
:11007958 A80D                       test al, 0D
:1100795A 0F8539010000               jne 11007A99
:11007960 DD1C24                     fstp qword ptr [esp]           ====> 放到[esp]
```

\* Reference To: `MSVBVM60.rtcSin`, Ord:0246h

|

```
:11007963 FF1504100011               Call dword ptr [11001004]     ====> sin
:11007969 DD9D7CFFFFFF               fstp qword ptr [ebp+FFFFFF7C]
:1100796F DD857CFFFFFF               fld qword ptr [ebp+FFFFFF7C]
:11007975 DC0D68120011               fmul qword ptr [11001268]     ====> 乘以 255
:1100797B D9E1                       fabs                            ====> 取绝对值
:1100797D DFE0                       fstsw ax
:1100797F A80D                       test al, 0D
:11007981 0F8512010000               jne 11007A99
```

\* Reference To: `MSVBVM60.__vbaR8IntI2`, Ord:0000h

|

```
:11007987 FF1560110011               Call dword ptr [11001160]     ====> 取整
:1100798D 33F0                       xor esi, eax                  ====> 与所取字符异或
:1100798F 8D5598                     lea edx, dword ptr [ebp-68]
:11007992 8D45B8                     lea eax, dword ptr [ebp-48]
:11007995 8D4DD4                     lea ecx, dword ptr [ebp-2C]
:11007998 52                         push edx
:11007999 50                         push eax
:1100799A 8975D4                     mov dword ptr [ebp-2C], esi
:1100799D 894DA0                     mov dword ptr [ebp-60], ecx
:110079A0 C7459802400000             mov [ebp-68], 00004002
```

\* Reference To: `MSVBVM60.rtcHexVarFromVar`, Ord:023Dh

|



:110079A7 `FF1510110011` Call `dword ptr [11001110]` ====> 计算结果转为 16 进制

:11007A1D `B801000000` `mov eax, 00000001` ====> 准备下一个字符

:11007A22 `6603C7` `add ax, di`

:11007A25 `7077` `jo 11007A9E`

:11007A27 `8BF8` `mov edi, eax`

:11007A29 `E903FEFFFF` `jmp 11007831`

计算过程：

`NewRegName=left(company,1)+UserName+right(company,1);`

`for i=1 to len(NewRegName);`

`Regcode=Regcode+Hex(Asc(Mid(NewRegName, i, 1)) Xor Abs(Int(Sin(101867^(i/len(NewRegName))*(3.14159265359/180))*255)))`

`next i`

上面的注册适用于英文名字注册，如果适合中英文名字，改为：

`Regcode=Regcode+Right(Hex(Asc(Mid(NewRegName, i, 1)) Xor Abs(Int(Sin(101867^(i/len(NewRegName))*(3.14159265359/180))*255))),4)`

注册器已经制作成功 `4dd466d0e1e0cc123afbe504da2462ca`
