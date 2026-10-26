
```
00442781     8BEC                       mov ebp,esp
00442783     8B45 08                    mov eax,dword ptr ss:[ebp+8]
00442786     A3 BC564400                mov dword ptr ds:[4456BC],eax  //这个就是Aspack2.12存放注册用户名的地方，我现在的Key是小黑，这里当然没有内容，先记住，找这个要多调试Asprotect程序找规律。
0044278B     5D                         pop ebp
0044278C     C2 0400                    retn 4
0044278F     90                         nop
00442790     55                         push ebp       //当前位置
00442791     8BEC                       mov ebp,esp
00442793     8B45 08                    mov eax,dword ptr ss:[ebp+8]
00442796     A3 FC484400                mov dword ptr ds:[4448FC],eax  //剩余日期
0044279B     8B45 0C                    mov eax,dword ptr ss:[ebp+C]
0044279E     A3 00494400                mov dword ptr ds:[444900],eax //总共日期
004427A3     5D                         pop ebp
004427A4     C2 0800                    retn 8
```

继续 `Shift+F9` 两次来到最后一次异常。

```
009900E1     3100                       xor dword ptr ds:[eax],eax
009900E3     64:8F05 00000000           pop dword ptr fs:[0]
009900EA     58                         pop eax
009900EB     833D DC399900 00           cmp dword ptr ds:[9939DC],0
009900F2     74 14                      je short 00990108
009900F4     6A 0C                      push 0C
009900F6     B9 DC399900                mov ecx,9939DC
009900FB     8D45 F8                    lea eax,dword ptr ss:[ebp-8]
009900FE     BA 04000000                mov edx,4
00990103     E8 94C3FFFF                call 0098C49C
00990108     FF75 FC                    push dword ptr ss:[ebp-4]
0099010B     FF75 F8                    push dword ptr ss:[ebp-8]
0099010E     8B45 F4                    mov eax,dword ptr ss:[ebp-C]
00990111     8338 00                    cmp dword ptr ds:[eax],0
00990114     74 02                      je short 00990118
00990116     FF30                       push dword ptr ds:[eax]
00990118     FF75 F0                    push dword ptr ss:[ebp-10]
0099011B     FF65 EC                    jmp dword ptr ss:[ebp-14]
0099011E     5F                         pop edi
0099011F     5E                         pop esi
00990120     5B                         pop ebx
00990121     8BE5                       mov esp,ebp
00990123     5D                         pop ebp
00990124     C3                         retn
```

```
0044289C     55                         push ebp  //早期的Asprotect善良，可以在入口脱壳,先写注册名。
```

命令行：

`dd 4456BC` 内容为 `00983431`，有些 Asprotect 校验这里验证壳是否存在。

寻找空地址：

`445FDC` 写入 `Mr.David`，然后修改 `4456BC` 内容为 `445FDC`。

```
0044289D     8BEC                       mov ebp,esp
0044289F     83C4 F4                    add esp,-0C
004428A2     E8 4D0BFCFF                call ASPack.004033F4
004428A7     E8 0C21FCFF                call ASPack.004049B8
004428AC     E8 6354FCFF                call ASPack.00407D14
004428B1     E8 D6C6FCFF                call ASPack.0040EF8C
004428B6     E8 BDC7FCFF                call ASPack.0040F078
004428BB     E8 08E7FCFF                call ASPack.00410FC8
004428C0     E8 974DFDFF                call ASPack.0041765C
004428C5     E8 B61AFEFF                call ASPack.00424380
004428CA     E8 F986FEFF                call ASPack.0042AFC8
004428CF     E8 E899FEFF                call ASPack.0042C2BC
004428D4     E8 639AFEFF                call ASPack.0042C33C
004428D9     E8 CEA5FEFF                call ASPack.0042CEAC
004428DE     E8 95ABFEFF                call ASPack.0042D478
004428E3     E8 30BAFEFF                call ASPack.0042E318
004428E8     E8 6FBAFEFF                call ASPack.0042E35C
004428ED     E8 FABAFEFF                call ASPack.0042E3EC
004428F2     E8 69E1FEFF                call ASPack.00430A60
004428F7     E8 A4AFFFFF                call ASPack.0043D8A0
004428FC     A1 30564400                mov eax,dword ptr ds:[445630]
00442901     E8 D210FEFF                call ASPack.004239D8
00442906     BA 38294400                mov edx,ASPack.00442938                      ; ASCII "ASPack"
```

然后 `Loadpe` 脱壳，运行 `ImportREC`，选择这个进程。把 `OEP` 改为 `4289C`，点 `IT AutoSearch`，点 `Get Import`，用“追踪层次1”修复，然后用 `Asprotect1.22` 插件修复剩下8个指针，`FixDump`，脱壳就完成了。

## 2. SDK修复

`OD` 载入脱壳程序，`F9` 运行出错。

堆栈友好提示

```
0012FFB0    0044291B  返回到 dumped\_.<ModuleEntryPoint>+7F 来自 0098C9A0 //访问壳，右键反汇编中跟随。
0012FFB4    0012FFE0  指针到下一个 SEH 记录
0012FFB8    00403330  SE 句柄
0012FFBC    0012FFC0
0012FFC0    0012FFF0
0012FFC4    77E614C7  返回到 kernel32.77E614C7
```

```
00442915   |.  FF15 0C494400            call dword ptr ds:[44490C]     //这里有问题,去原程序看看。
0044291B   |.  A1 30564400              mov eax,dword ptr ds:[445630] //来到这里
00442920   |.  E8 5311FEFF              call dumped\_.00423A78
00442925   |.  E8 D21BFCFF              call dumped\_.004044FC
0044292A   |.  8BE5                     mov esp,ebp
0044292C   |.  5D                       pop ebp
0044292D   \.  C3                       retn
```

主程序跟踪路线

```
00442915     FF15 0C494400              call dword ptr ds:[44490C]        //都是F7避免跑飞
0098C9A0     833D A8359900 00           cmp dword ptr ds:[9935A8],0
0098C9A7     74 06                      je short 0098C9AF
0098C9A9     FF15 A8359900              call dword ptr ds:[9935A8]  ; ASPack.004427A8 //验证壳是否存在,就是处于pre-dip中
004427A8     B9 B8564400                mov ecx,ASPack.004456B8
004427AD     BA 08E54300                mov edx,ASPack.0043E508
004427B2     A1 30564400                mov eax,dword ptr ds:[445630]
004427B7     E8 2C12FEFF                call ASPack.004239E8
004427BC     C3                         retn
```

那么先修复：

```
00442915   |.  FF15 0C494400            call dword ptr ds:[44490C]     //这里有问题,去原程序看看。
```

原地址：`0098C9A0`

修改为：`004427A8`

复制到程序，重启 `OD`。

`F9` 出错，这次是主功能了，精神些，代码也多了些。

```
0012FE00    0043F1BE  返回到 dumped\_.0043F1BE 来自 0098C8F4  //右键反汇编中跟随。
0012FE04    0012FE58  指针到下一个 SEH 记录
0012FE08    0043F4D0  SE 句柄
0012FE0C    0012FE50
0012FE10    0012CD78
0012FE14    0043E508  dumped\_.0043E508
0012FE18    009620C4
0012FE1C    00000000
0012FE20    0096BF88  ASCII "D:\Down\Aspack V2.12\dumped\_.exe"
```

```
0043F1B8    .  FF15 04494400            call dword ptr ds:[444904]
0043F1BE    .  E9 83000000              jmp dumped\_.0043F246 //返回这里，右键删除分析
0043F1C3       26                       db 26                                        ;  CHAR '&'
0043F1C4       F4                       db F4
0043F1C5       CC                       int3
0043F1C6       46                       db 46                                        ;  CHAR 'F'
0043F1C7       F2                       db F2
0043F1C8       B8                       db B8
0043F1C9       A1                       db A1
0043F1CA       34                       db 34                                        ;  CHAR '4'
0043F1CB       0E                       db 0E
0043F1CC       BD                       db BD
0043F1CD       59                       db 59                                        ;  CHAR 'Y'
0043F1CE       6B                       db 6B                                        ;  CHAR 'k'
0043F1CF       0F                       db 0F
0043F1D0       05                       db 05
```

```
0043F1B8     FF15 04494400              call dword ptr ds:[444904] //跟踪原程序，这里只是一个简单的ret返回，如果这样修复只是Demo版，当然不能让我们满足。
0043F1BE     E9 83000000                jmp dumped\_.0043F246  //这里根本没有解码。
0043F1C3     26:F4                      hlt
0043F1C5     CC                         int3
0043F1C6     46                         inc esi
0043F1C7     F2:                        prefix repne:
0043F1C8     B8 A1340EBD                mov eax,BD0E34A1
0043F1CD     59                         pop ecx
0043F1CE     6B0F 05                    imul ecx,dword ptr ds:[edi],5
0043F1D1     F8                         clc
0043F1D2     58                         pop eax
0043F1D3     DBD8                       fcmovnu st,st
```
