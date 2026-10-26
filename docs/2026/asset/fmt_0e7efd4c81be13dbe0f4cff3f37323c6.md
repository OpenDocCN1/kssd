
# XnView V1.90.3 多语言(中文)绿色版

【软件名称】XnView V1.90.3 多语言(中文)绿色版

【资源来自】`霏凡软件aa8K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4N6%4N6#2)9J5k6h3y4J5M7$3E0&6i4K6u0W2j5$3!0E0i4K6u0r3k6r3g2X3j5i4g2D9N6q4)9J5k6h3S2@1L8h3I4Q4c8e0y4Q4z5o6m8Q4z5o6t1`。

【应用平台】Win2000

【作者邮箱】chubing6143@sina.com

【使用工具】`peid`, `OllyDbg1.10`

【软件限制】注册码

【破解工具】`OllyDbg v1.10`

【软件简介】非常棒的图像查看程序。支持150种图片格式， 除一般的查看、浏览、幻灯显示等功能外，还自带30多面滤镜，方便编辑修改； 可以批量转换文件格式，创建缩略图并生成网页，还可自己制作GIF， 小巧实用。 新增Xnview Shell Extension下载，可为您的鼠标右键增加图片预览功能！

关于XnView的优势.

1. 打开图片速度极快
2. 功能齐全,且很实用
3. 基本不占资源
4. 界面美观,支持换肤
5. 其具有的右键预览功能非常实用(可通过提供的`XnShellEx.exe`自主安装)
6. 体积小,不占什么空间(相对ACDSEE)

【破解过程】该软件注册过程非常清晰，如果想爆破太简单了，但是对于我等菜鸟跟踪算法写注册机到不失为一个不错的例子。

友情提示：跟踪算法时如果担心自己记不住，最好自己用高级语言实现一遍，记笔记了。

利用`PEID`察看该软件是`ASPack 2.12b -> Alexey Solodovnikov`加壳，利用`OD`自己手动脱壳，脱壳过程在此就不再赘述了。脱壳后发现是`VC`下的程序，通过下`BP MessageBox`可以定位到下面关键地方：

```asm
00534470    .  81EC 68010000     sub esp,168
00534476    .  8D4424 68         lea eax,dword ptr ss:[esp+68]
0053447A    .  56                push esi
0053447B    .  8BB424 70010000   mov esi,dword ptr ss:[esp+170]
00534482    .  57                push edi
00534483    .  8B3D 78A66600     mov edi,dword ptr ds:[<&USER32.GetDlgI>;  USER32.GetDlgItemTextA
00534489    .  68 00010000       push 100                               ; /Count = 100 (256.)
0053448E    .  50                push eax                               ; |Buffer
0053448F    .  68 D0070000       push 7D0                               ; |ControlID = 7D0 (2000.)
00534494    .  56                push esi                               ; |hWnd
00534495    .  FFD7              call edi                               ; \用户名
00534497    .  8D4C24 10         lea ecx,dword ptr ss:[esp+10]
0053449B    .  6A 20             push 20                                ; /Count = 20 (32.)
0053449D    .  51                push ecx                               ; |Buffer
0053449E    .  68 D1070000       push 7D1                               ; |ControlID = 7D1 (2001.)
005344A3    .  56                push esi                               ; |hWnd
005344A4    .  FFD7              call edi                               ; \注册码
005344A6    .  8A4424 70         mov al,byte ptr ss:[esp+70]
005344AA    .  84C0              test al,al
005344AC    .  0F84 3A010000     je dumped.005345EC                     ;  判断用户名是否为空
005344B2    .  8A4424 10         mov al,byte ptr ss:[esp+10]
005344B6    .  84C0              test al,al
005344B8    .  0F84 2E010000     je dumped.005345EC                     ;  判断注册码是否为空
005344BE    .  8D5424 08         lea edx,dword ptr ss:[esp+8]
005344C2    .  8D4424 70         lea eax,dword ptr ss:[esp+70]
005344C6    .  52                push edx
005344C7    .  50                push eax
005344C8    .  E8 F3AEF9FF       call dumped.004CF3C0                   ;  用户名运算
005344CD    .  8D4C24 18         lea ecx,dword ptr ss:[esp+18]
005344D1    .  51                push ecx
005344D2    .  E8 7C660200       call dumped.0055AB53                   ;  注册码运算
005344D7    .  8B4C24 14         mov ecx,dword ptr ss:[esp+14]
005344DB    .  83C4 0C           add esp,0C
005344DE    .  3BC8              cmp ecx,eax                            ;  用户名运算结果与注册码运算结果进行比较
005344E0       74 5D             je short dumped.0053453F               ;  关键跳转
005344E2    .  A1 A0B16E00       mov eax,dword ptr ds:[6EB1A0]
005344E7    .  8D5424 30         lea edx,dword ptr ss:[esp+30]
005344EB    .  6A 40             push 40                                ; /Count = 40 (64.)
005344ED    .  52                push edx                               ; |Buffer
005344EE    .  68 93130000       push 1393                              ; |RsrcID = STRING "Invalid registration"
005344F3    .  50                push eax                               ; |hInst => 10000000
005344F4    .  FF15 10A76600     call dword ptr ds:[<&USER32.LoadString>; \LoadStringA
005344FA    .  6A 10             push 10                                ; /Style = MB\_OK|MB\_ICONHAND|MB\_APPLMODAL
005344FC    .  8D4C24 34         lea ecx,dword ptr ss:[esp+34]          ; |
00534500    .  68 EC316E00       push dumped.006E31EC                   ; |Title = ""
00534505    .  51                push ecx                               ; |Text
00534506    .  56                push esi                               ; |hOwner
00534507    .  FF15 98A66600     call dword ptr ds:[<&USER32.MessageBox>; \MessageBoxA
0053450D    .  68 D0070000       push 7D0                               ; /ControlID = 7D0 (2000.)
00534512    .  56                push esi                               ; |hWnd
00534513    .  FF15 90A66600     call dword ptr ds:[<&USER32.GetDlgItem>; \GetDlgItem
00534519    .  50                push eax                               ; /hWnd
0053451A    .  FF15 D0A66600     call dword ptr ds:[<&USER32.SetFocus>] ; \SetFocus
00534520    .  68 EC316E00       push dumped.006E31EC                   ; /Text = ""
00534525    .  68 D1070000       push 7D1                               ; |ControlID = 7D1 (2001.)
0053452A    .  56                push esi                               ; |hWnd
0053452B    .  FF15 8CA66600     call dword ptr ds:[<&USER32.SetDlgItem>; \SetDlgItemTextA
00534531    .  5F                pop edi
00534532    .  B8 01000000       mov eax,1
00534537    .  5E                pop esi
00534538    .  81C4 68010000     add esp,168
0053453E    .  C3                retn
0053453F    >  8D5424 70         lea edx,dword ptr ss:[esp+70]
00534543    .  68 00010000       push 100
00534548    .  52                push edx
00534549    .  68 D0070000       push 7D0
0053454E    .  56                push esi
0053454F    .  FFD7              call edi
00534551    .  8D4424 10         lea eax,dword ptr ss:[esp+10]
00534555    .  6A 20             push 20
00534557    .  50                push eax
00534558    .  68 D1070000       push 7D1
0053455D    .  56                push esi
0053455E    .  FFD7              call edi
00534560    .  8D4C24 70         lea ecx,dword ptr ss:[esp+70]
00534564    .  51                push ecx
00534565    .  68 3CF06B00       push dumped.006BF03C                   ;  ASCII "LicenseName"
0053456A    .  6A 00             push 0
0053456C    .  E8 CF49F7FF       call dumped.004A8F40
00534571    .  8D5424 1C         lea edx,dword ptr ss:[esp+1C]
00534575    .  52                push edx
00534576    .  68 2CF06B00       push dumped.006BF02C                   ;  ASCII "LicenseNumber"
0053457B    .  6A 00             push 0
0053457D    .  E8 BE49F7FF       call dumped.004A8F40
00534582    .  A1 A8B16E00       mov eax,dword ptr ds:[6EB1A8]
00534587    .  83C4 18           add esp,18
0053458A    .  C705 C8B16E00 010>mov dword ptr ds:[6EB1C8],1
00534594    .  6A 01             push 1                                 ; /Flags = MF\_BYCOMMAND|MF\_GRAYED|MF\_STRING
00534596    .  68 F2000000       push 0F2                               ; |ItemID = F2 (242.)
0053459B    .  50                push eax                               ; |/hWnd => 001C0320 ('XnView v1.90.3  - [浏览器 - D...',class='XmainClass')
0053459C    .  FF15 30A76600     call dword ptr ds:[<&USER32.GetMenu>]  ; |\GetMenu
005345A2    .  50                push eax                               ; |hMenu
005345A3    .  FF15 E4A66600     call dword ptr ds:[<&USER32.EnableMenu>; \EnableMenuItem
005345A9    .  8B15 A0B16E00     mov edx,dword ptr ds:[6EB1A0]          ;  xnviewzh.10000000
005345AF    .  8D4C24 30         lea ecx,dword ptr ss:[esp+30]
005345B3    .  6A 40             push 40                                ; /Count = 40 (64.)
005345B5    .  51                push ecx                               ; |Buffer
005345B6    .  68 94130000       push 1394                              ; |RsrcID = STRING "Registration successful.

Thank you for purchasing XnView."

005345BB    .  52                push edx                               ; |hInst => 10000000
```
