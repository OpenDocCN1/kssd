
# HTML2TXT V4.1.050202 注册算法分析 + VB 注册机

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-443.htm`](https://www.pediy.com/kssd/pediy07/pediy7-443.htm)

【破文标题】：HTML2TXT V4.1.050202 注册算法分析 + VB 注册机  

【破文作者】：KuNgBiM[DFCG]  

【作者邮箱】：gb_1227@163.com  

【软件名称】：HTML2TXT V4.1.050202  

【软件大小】：676 KB  

【软件语言】：英文  

【软件类别】：国外软件 / 试用版 / 文字处理  

【整理时间】：2005-07-04  

【开 发 商】：[`www.infomedia.it/artic/Baccan/`](http://www.infomedia.it/artic/Baccan/)  

【下载地址】：[`www.skycn.com/soft/20971.html`](http://www.skycn.com/soft/20971.html)  

【保护方式】：注册码 + 启动 NAG + 功能限制 + 重启验证  

【编译语言】：Borland Delphi 6.0 - 7.0  

【调试环境】：WinXP、PEiD、W32Dasm、Ollydbg  

【破解日期】：2005-11-02  

【破解目的】：研究算法分析  

【作者声明】：初学 Crack，只是感兴趣，没有其他目的。失误之处敬请诸位大侠赐教！  

—————————————————————————————————  

## 破解过程

侦测：用 `PEiD` 查壳，无壳，Borland Delphi 6.0 - 7.0 编译。

试探：运行主程序注册，输入试炼码，确认！程序提示："Thanks. You registration code will be checked when you start this application again."

BTW:但正确注册关键提示不在此处!在"Unregistered Copy" 这个信息!

对症下药：`Ollydbg` 载入主程序，用查找字符串插件查找"Unregistered Copy" 这个信息!双击来到 `004A67C4` 处下断，一路 `F9` 运行，直到程序运行，先取消注册，然后再手动点击选项，输入试炼信息：

\*\*\*\*\*\*\*\*\*\*\*\* 试炼信息 \*\*\*\*\*\*\*\*\*\*\*\*\*

Registered Email：gb_1227@163.com

Registration Code：9876543210

\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*\*

```
004A67AC    55                push ebp                              ; 此处 F2 下断!F9 运行!
004A67AD    68 84684A00       push h2t.004A6884
004A67B2    64:FF30           push dword ptr fs:[eax]
004A67B5    64:8920           mov dword ptr fs:[eax],esp
004A67B8    C605 58DC4A00 0>  mov byte ptr ds:[4ADC58],0
004A67BF    B8 5CDC4A00       mov eax,h2t.004ADC5C
004A67C4    BA 9C684A00       mov edx,h2t.004A689C                  ; ASCII "Unregistered Copy"
004A67C9    E8 0ADEF5FF       call h2t.004045D8
004A67CE    8D45 F4           lea eax,dword ptr ss:[ebp-C]
004A67D1    B9 B8684A00       mov ecx,h2t.004A68B8                  ; 检查是否有"h2t.ini"注册文件!ASCII "h2t.ini"
004A67D6    8B15 54DC4A00     mov edx,dword ptr ds:[4ADC54]         ; 是否在安装目录下!ASCII "F:\\HTML2TXT\\"
004A67DC    E8 AFE0F5FF       call h2t.00404890
004A67E1    8B4D F4           mov ecx,dword ptr ss:[ebp-C]          ; ASCII "F:\\HTML2TXT\\h2t.ini"
004A67E4    B2 01             mov dl,1                              ; dl=54 ('T')
004A67E6    A1 48334200       mov eax,dword ptr ds:[423348]
004A67EB    E8 10CCF7FF       call h2t.00423400                     ; 取出注册使用的"Registered Email"
004A67F0    8BD8              mov ebx,eax
004A67F2    6A 00             push 0
004A67F4    8D45 FC           lea eax,dword ptr ss:[ebp-4]
004A67F7    50                push eax
004A67F8    B9 C8684A00       mov ecx,h2t.004A68C8                  ; ASCII "Email"
004A67FD    BA D8684A00       mov edx,h2t.004A68D8                  ; ASCII "Register"
004A6802    8BC3              mov eax,ebx
004A6804    8B30              mov esi,dword ptr ds:[eax]
004A6806    FF16              call dword ptr ds:[esi]               ; 取出注册使用的"Registration Code"
004A6808    6A 00             push 0
004A680A    8D45 F8           lea eax,dword ptr ss:[ebp-8]
004A680D    50                push eax
004A680E    B9 EC684A00       mov ecx,h2t.004A68EC                  ; ASCII "Code"
004A6813    BA D8684A00       mov edx,h2t.004A68D8                  ; ASCII "Register"
004A6818    8BC3              mov eax,ebx
004A681A    8B30              mov esi,dword ptr ds:[eax]
004A681C    FF16              call dword ptr ds:[esi]               ; 取固定的字符串"h2t"
004A681E    8D45 FC           lea eax,dword ptr ss:[ebp-4]
004A6821    BA FC684A00       mov edx,h2t.004A68FC                  ; ASCII "h2t"
004A6826    E8 21E0F5FF       call h2t.0040484C                     ; 连接 Registered Email+"h2t"
004A682B    8D55 E0           lea edx,dword ptr ss:[ebp-20]
004A682E    8B45 FC           mov eax,dword ptr ss:[ebp-4]
004A6831    E8 A6F8FFFF       call h2t.004A60DC                     ; 检测是否连接无误!
004A6836    8D45 E0           lea eax,dword ptr ss:[ebp-20]
004A6839    8D55 F0           lea edx,dword ptr ss:[ebp-10]
004A683C    E8 0FF9FFFF       call h2t.004A6150                     ; ★算法 CALL★F7 跟进!
004A6841    8B45 F0           mov eax,dword ptr ss:[ebp-10]         ; 获取真注册码!
004A6844    8B55 F8           mov edx,dword ptr ss:[ebp-8]          ; 获取假注册码!
004A6847    E8 44E1F5FF       call h2t.00404990                     ; 经典比较!这里就不用跟进了!
004A684C    75 14             jnz short h2t.004A6862                ; ★完美爆破点★  nop 掉即可!
004A684E    C605 58DC4A00 0>  mov byte ptr ds:[4ADC58],1
004A6855    B8 5CDC4A00       mov eax,h2t.004ADC5C
004A685A    8B55 FC           mov edx,dword ptr ss:[ebp-4]
004A685D    E8 76DDF5FF       call h2t.004045D8
004A6862    8BC3              mov eax,ebx                             ; 跳到这里就 OVER 了~
004A6864    E8 57CFF5FF       call h2t.004037C0
004A6869    33C0              xor eax,eax
004A686B    5A                pop edx
004A686C    59                pop ecx
004A686D    59                pop ecx
004A686E    64:8910           mov dword ptr fs:[eax],edx
004A6871    68 8B684A00       push h2t.004A688B
004A6876    8D45 F0           lea eax,dword ptr ss:[ebp-10]
004A6879    BA 04000000       mov edx,4
004A687E    E8 25DDF5FF       call h2t.004045A8
004A6883    C3                retn
```

........

\========================= 跟进 `004A683C` `E8 0FF9FFFF` `call h2t.004A6150` \=========================

```
004A6150    55                push ebp                                ; 跟进到这里!
004A6151    8BEC              mov ebp,esp
004A6153    83C4 E8           add esp,-18
004A6156    53                push ebx
004A6157    56                push esi
004A6158    57                push edi
004A6159    33C9              xor ecx,ecx
004A615B    894D EC           mov dword ptr ss:[ebp-14],ecx
004A615E    894D E8           mov dword ptr ss:[ebp-18],ecx
004A6161    8BF0              mov esi,eax
004A6163    8D7D F0           lea edi,dword ptr ss:[ebp-10]
004A6166    A5                movs dword ptr es:[edi],dword ptr ds:[esi]  ; 调用标准 MD5 第一个常数
004A6167    A5                movs dword ptr es:[edi],dword ptr ds:[esi]  ; 调用标准 MD5 第二个常数
004A6168    A5                movs dword ptr es:[edi],dword ptr ds:[esi]  ; 调用标准 MD5 第三个常数
004A6169    A5                movs dword ptr es:[edi],dword ptr ds:[esi]  ; 调用标准 MD5 第四个常数
004A616A    8BFA              mov edi,edx
004A616C    33C0              xor eax,eax
004A616E    55                push ebp
004A616F    68 EA614A00       push h2t.004A61EA
004A6174    64:FF30           push dword ptr fs:[eax]
004A6177    64:8920           mov dword ptr fs:[eax],esp
004A617A    8BC7              mov eax,edi
004A617C    E8 03E4F5FF       call h2t.00404584
004A6181    B3 10             mov bl,10
004A6183    8D75 F0           lea esi,dword ptr ss:[ebp-10]
004A6186    FF37              push dword ptr ds:[edi]
004A6188    8D45 EC           lea eax,dword ptr ss:[ebp-14]
004A618B    0FB616            movzx edx,byte ptr ds:[esi]
004A618E    C1EA 04           shr edx,4
004A6191    83E2 0F           and edx,0F
004A6194    0FB692 D0BD4A00   movzx edx,byte ptr ds:[edx+4ABDD0]
004A619B    E8 CCE5F5FF       call h2t.0040476C
004A61A0    FF75 EC           push dword ptr ss:[ebp-14]
004A61A3    8D45 E8           lea eax,dword ptr ss:[ebp-18]
004A61A6    0FB616            movzx edx,byte ptr ds:[esi]
004A61A9    80E2 0F           and dl,0F
004A61AC    0FB6D2            movzx edx,dl
004A61AF    0FB692 D0BD4A00   movzx edx,byte ptr ds:[edx+4ABDD0]
004A61B6    E8 B1E5F5FF       call h2t.0040476C
004A61BB    FF75 E8           push dword ptr ss:[ebp-18]
004A61BE    8BC7              mov eax,edi
004A61C0    BA 03000000       mov edx,3
004A61C5    E8 3AE7F5FF       call h2t.00404904
004A61CA    46                inc esi
004A61CB    FECB              dec bl
004A61CD    ^ 75 B7           jnz short h2t.004A6186
```
