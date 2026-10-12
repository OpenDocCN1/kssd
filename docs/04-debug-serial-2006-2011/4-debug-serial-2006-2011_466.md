
# Bluefox MOV Converter 2.10.08.1225 算法分析

> 原文：[`www.pediy.com/kssd/pediy10/80083.html`](https://www.pediy.com/kssd/pediy10/80083.html)

## 基本信息

**【破文标题】** Bluefox MOV Converter 2.10.08.1225 算法分析  
**【破文作者】** `creantan`  
**【作者邮箱】** `creantan@126.com`  
**【破解工具】** `PEiD`, `OD`  
**【破解平台】** `Windows XP sp2`  
**【软件名称】** Bluefox MOV Converter 2.10.08.1225  
**【软件大小】** `4569KB`  
**【软件语言】** 英文  
**【软件类别】** 国外软件/视频处理  
**【更新时间】** `2009-1-6`  
**【原版下载】** 自己找一下  
**【保护方式】** 注册码  
**【软件简介】** Bluefox MOV Converter 是一款录影转换软件  
**【破解声明】** 我是一只小菜鸟，偶得一点心得，愿与大家分享：）

## 破解内容

`BlueFoxStudio_Video_Mov.exe` 查壳，显示：Microsoft Visual C++ 6.0

观察下注册流程。。要填入用户名和密码。。随便输入出现错误提示，下 `bp MessageBoxA` 断点，找到下面注册算法：

```
0041A491   .  50           push    eax
0041A492   .  6A 00        push    0
0041A494   .  E8 652F0300 call    0044D3FE
0041A499   .  50           push    eax
0041A49A   .  E8 71CAFEFF call    00406F10                             ;  b-mov-converter
0041A49F   .  8D4C24 3C    lea     ecx, dword ptr [esp+3C]
0041A4A3   .  8D5424 60    lea     edx, dword ptr [esp+60]
0041A4A7   .  51           push    ecx
0041A4A8   .  52           push    edx
0041A4A9   .  E8 A2E3FeFF call    00408850                             ;  取“b-mov-converter”MD5 值=6622b32ece23a5be86356074ebd23cfd
```

```
0041A58E   .  8D4C24 10    lea     ecx, dword ptr [esp+10]
0041A592   .  C68424 5C020> mov     byte ptr [esp+25C], 2
0041A59A   .  E8 8E2A0300 call    0044D02D                             ;  取前四位假码
0041A59F   .  8B5424 24    mov     edx, dword ptr [esp+24]             ;  与 32 位 MD5 值的前两位和最后两位连接的字符串"66fd"进行比较
0041A5A3   .  8B4424 28    mov     eax, dword ptr [esp+28]
0041A5A7   .  52           push    edx
0041A5A8   .  50           push    eax
0041A5A9   .  E8 E1E00100 call    0043868F                             ;  验证是否相等
0041A5AE   .  83C4 08      add     esp, 8
0041A5B1   .  85C0         test    eax, eax
0041A5B3   .  74 04        je      short 0041A5B9                       ;  不等跳向错误提示
0041A5B5   .  33C0         xor     eax, eax
0041A5B7   .  EB 3E        jmp     short 0041A5F7
```



0041A5C7 的汇编代码块：
```
0041A5C7 . 68 00010000 push 100
0041A5CC . F3:AB rep stos dword ptr es:\[edi\]
0041A5CE . 8D8C24 580100>lea ecx, dword ptr \[esp+158\]
0041A5D5 . 51 push ecx
0041A5D6 . 50 push eax
0041A5D7 . 8BCD mov ecx, ebp
0041A5D9 . E8 202E0300 call 0044D3FE
0041A5DE . 50 push eax ; 取假码
0041A5DF . E8 2CC9FEFF call 00406F10
0041A5E4 . 83C4 0C add esp, 0C
0041A5E7 . 8D9424 540100>lea edx, dword ptr \[esp+154\]
0041A5EE . 8D4E 5C lea ecx, dword ptr \[esi+5C\]
0041A5F1 . 52 push edx
0041A5F2 . E8 39E5FFFF call 00418B30 ; 关键算法
{
00418B30 /$ 81EC 88050000 sub esp, 588
00418B36 |. B9 20000000 mov ecx, 20
00418B3B |. 33C0 xor eax, eax
00418B3D |. 56 push esi
00418B3E |. 57 push edi
00418B3F |. 8DBC24 900000>lea edi, dword ptr \[esp+90\]
00418B46 |. 68 28C94700 push 0047C928
00418B4B |. F3:AB rep stos dword ptr es:\[edi\]
00418B4D |. B9 20000000 mov ecx, 20
00418B52 |. 8DBC24 140200>lea edi, dword ptr \[esp+214\]
00418B59 |. F3:AB rep stos dword ptr es:\[edi\]
00418B5B |. B9 20000000 mov ecx, 20
00418B60 |. 8D7C24 14 lea edi, dword ptr \[esp+14\]
00418B64 |. F3:AB rep stos dword ptr es:\[edi\]
00418B66 |. B9 20000000 mov ecx, 20
00418B6B |. 8DBC24 140100>lea edi, dword ptr \[esp+114\]
00418B72 |. F3:AB rep stos dword ptr es:\[edi\]
00418B74 |. B9 20000000 mov ecx, 20
00418B79 |. 8DBC24 940100>lea edi, dword ptr \[esp+194\]
00418B80 |. F3:AB rep stos dword ptr es:\[edi\]
00418B82 |. B9 20000000 mov ecx, 20
00418B87 |. 8DBC24 940200>lea edi, dword ptr \[esp+294\]
00418B8E |. F3:AB rep stos dword ptr es:\[edi\]
00418B90 |. 8BBC24 980500>mov edi, dword ptr \[esp+598\]
00418B97 |. 57 push edi
00418B98 |. E8 13060200 call 004391B0 ; //判断假码中是否存在“-”
00418B9D |. 83C4 08 add esp, 8
00418BA0 |. 85C0 test eax, eax
00418BA2 |. 75 0E jnz short 00418BB2
00418BA4 |. 5F pop edi
00418BA5 |. 83C8 FF or eax, FFFFFFFF
00418BA8 |. 5E pop esi
00418BA9 |. 81C4 88050000 add esp, 588
00418BAF |. C2 0400 retn 4
00418BB2 |> 8A0F mov cl, byte ptr \[edi\]
00418BB4 |. 33F6 xor esi, esi
00418BB6 |. 80F9 2D cmp cl, 2D
00418BB9 |. 74 18 je short 00418BD3
00418BBB |. 8D9424 900000>lea edx, dword ptr \[esp+90\]
00418BC2 |. 8BC7 mov eax, edi
00418BC4 |. 2BD7 sub edx, edi
00418BC6 |> 880C02 /mov byte ptr \[edx+eax\], cl
00418BC9 |. 8A48 01 |mov cl, byte ptr \[eax+1\]
00418BCC |. 46 |inc esi
00418BCD |. 40 |inc eax
00418BCE |. 80F9 2D |cmp cl, 2D ; 碰到“-”结束
00418BD1 |.^ 75 F3 \\jnz short 00418BC6 ; 取第一段假码
00418BD3 |> 68 28C94700 push 0047C928
00418BD8 |. 57 push edi
00418BD9 |. 46 inc esi
00418BDA |. E8 D1050200 call 004391B0
00418BDF |. 83C4 08 add esp, 8
00418BE2 |. 85C0 test eax, eax
00418BE4 |. 75 0E jnz short 00418BF4 ; 判断第一个“-”后是否还有假码
00418BE6 |. 5F pop edi
00418BE7 |. 83C8 FF or eax, FFFFFFFF ; 没有的话 eax=FFFFFFFF
00418BEA |. 5E pop esi
00418BEB |. 81C4 88050000 add esp, 588
00418BF1 |. C2 0400 retn 4 ; 返回。。提示注册码长度不正确
00418BF4 |> 8A043E mov al, byte ptr \[esi+edi\]
00418BF7 |. 3C 2D cmp al, 2D
00418BF9 |. 74 13 je short 00418C0E
00418BFB |. 8D4C24 10 lea ecx, dword ptr \[esp+10\]
00418BFF |> 84C0 /test al, al ; 取假码第二部分
00418C01 |. 74 0B |je short 00418C0E
00418C03 |. 46 |inc esi
00418C04 |. 8801 |mov byte ptr \[ecx\], al
00418C06 |. 41 |inc ecx
00418C07 |. 8A043E |mov al, byte ptr \[esi+edi\]
00418C0A |. 3C 2D |cmp al, 2D ; 碰到“-”结束
00418C0C |.^ 75 F1 \\jnz short 00418BFF
00418C0E |> 803C3E 00 cmp byte ptr \[esi+edi\], 0 ; 判断第二个“-”后是否还有假码
00418C12 |. 75 0E jnz short 00418C22
00418C14 |. 5F pop edi
00418C15 |. 83C8 FF or eax, FFFFFFFF ; 没有的话 eax=FFFFFFFF
00418C18 |. 5E pop esi
00418C19 |. 81C4 88050000 add esp, 588
00418C1F |. C2 0400 retn 4 ; 返回。。提示注册码长度不正确
00418C22 |> 8D4424 0C lea eax, dword ptr \[esp+C\]
00418C26 |. 8D4C3E 01 lea ecx, dword ptr \[esi+edi+1\]
00418C2A |. 50 push eax
00418C2B |. 68 70B24700 push 0047B270 ; ASCII "%d"
00418C30 |. 51 push ecx
00418C31 |. E8 42050200 call 00439178 ; //取第三段假码
00418C36 |. 83C4 0C add esp, 0C
00418C39 |. 83F8 01 cmp eax, 1 ; 如果第三段不是数字的话就退出。。出现错误提示
00418C3C |. 73 0E jnb short 00418C4C
00418C3E |. 5F pop edi
00418C3F |. 83C8 FF or eax, FFFFFFFF
00418C42 |. 5E pop esi
00418C43 |. 81C4 88050000 add esp, 588
00418C49 |. C2 0400 retn 4
00418C4C |> 8B4424 0C mov eax, dword ptr \[esp+C\] ; 注册码第三部分值赋给 eax 进行计算。。。。。。
00418C50 |. 8D90 CF040000 lea edx, dword ptr \[eax+4CF\] ; edx=eax+0x4cf
00418C56 |. 8D88 4E040000 lea ecx, dword ptr \[eax+44E\] ; ecx=eax+0x44e
00418C5C |. 69C0 E67A0200 imul eax, eax, 27AE6 ; eax*=0x27ae6
00418C62 |. 895424 08 mov dword ptr \[esp+8\], edx
00418C66 |. DB4424 08 fild dword ptr \[esp+8\]
00418C6A |. 894C24 08 mov dword ptr \[esp+8\], ecx
00418C6E |. DB4424 08 fild dword ptr \[esp+8\]
00418C72 |. 894424 08 mov dword ptr \[esp+8\], eax
00418C76 |. DEC9 fmulp st(1), st ; edx 和 ecx 中值相乘
00418C78 |. DB4424 08 fild dword ptr \[esp+8\]
00418C7C |. DEC1 faddp st(1), st ; 乘的结果加上 EAX 值
00418C7E |. D9FA fsqrt ; 开方
00418C80 |. DC0D 20774600 fmul qword ptr \[467720\] ; 与常数 1112 相乘
00418C86 |. E8 DDF90100 call 00438668
00418C8B |. 50 push eax
00418C8C |. 8D9424 140100>lea edx, dword ptr \[esp+114\]
00418C93 |. 68 24C94700 push 0047C924 ; ASCII "%ld"
00418C98 |. 52 push edx
00418C99 |. E8 70FE0100 call 00438B0E
00418C9E |. 8DBC24 1C0100>lea edi, dword ptr \[esp+11C\] ; 取得上面计算结果长整形下面比较用
00418CA5 |. 83C9 FF or ecx, FFFFFFFF
00418CA8 |. 33C0 xor eax, eax
00418CAA |. 83C4 0C add esp, 0C
00418CAD |. F2:AE repne scas byte ptr es:\[edi\]
00418CAF |. F7D1 not ecx
00418CB1 |. 83C1 FE add ecx, -2
00418CB4 |. 8D8C0C 100100>lea ecx, dword ptr \[esp+ecx+110\]
00418CBB |> 8A11 /mov dl, byte ptr \[ecx\]
00418CBD |. 49 |dec ecx
00418CBE |. 889404 900100>|mov byte ptr \[esp+eax+190\], dl
00418CC5 |. 40 |inc eax
00418CC6 |. 83F8 06 |cmp eax, 6 ; 从长整形数个位开始往高位取六位数
00418CC9 |.^ 72 F0 \\jb short 00418CBB
00418CCB |. 53 push ebx
00418CCC |. 55 push ebp
00418CCD |. 8D7C24 18 lea edi, dword ptr \[esp+18\]
00418CD1 |. 83C9 FF or ecx, FFFFFFFF ; //////////////////////////////////////////
00418CD4 |. 33C0 xor eax, eax
00418CD6 |. 33DB xor ebx, ebx
00418CD8 |. 33ED xor ebp, ebp ; 这一段大家应该很眼熟了。。。取长度用的
00418CDA |. F2:AE repne scas byte ptr es:\[edi\]
00418CDC |. F7D1 not ecx
00418CDE |. 49 dec ecx ; /////////////////////////////////////////////////
00418CDF |. 74 59 je short 00418D3A
00418CE1 |. 8DB424 980300>lea esi, dword ptr \[esp+398\]
00418CE8 |> 8A441C 18 /mov al, byte ptr \[esp+ebx+18\]
00418CEC |. 3C 41 |cmp al, 41 ; 与‘A’比较
00418CEE |. 7C 2B |jl short 00418D1B
00418CF0 |. 3C 5A |cmp al, 5A ; 与‘Z’比较
00418CF2 |. 7F 27 |jg short 00418D1B
00418CF4 |. 0FBEC8 |movsx ecx, al
00418CF7 |. B8 67666666 |mov eax, 66666667
00418CFC |. 83C6 04 |add esi, 4
00418CFF |. F7E9 |imul ecx
00418D01 |. C1FA 02 |sar edx, 2
00418D04 |. 8BC2 |mov eax, edx
00418D06 |. C1E8 1F |shr eax, 1F
00418D09 |. 03D0 |add edx, eax
00418D0B |. 8BC1 |mov eax, ecx
00418D0D |. 8956 FC |mov dword ptr \[esi-4\], edx
00418D10 |. B9 0A000000 |mov ecx, 0A
00418D15 |. 99 |cdq
00418D16 |. 45 |inc ebp
00418D17 |. F7F9 |idiv ecx
00418D19 |. EB 06 |jmp short 00418D21
00418D1B |> 0FBED0 |movsx edx, al
00418D1E |. 83EA 30 |sub edx, 30 ; edx-30
00418D21 |> 8916 |mov dword ptr \[esi\], edx
00418D23 |. 43 |inc ebx
00418D24 |. 8D7C24 18 |lea edi, dword ptr \[esp+18\]
00418D28 |. 83C9 FF |or ecx, FFFFFFFF
00418D2B |. 33C0 |xor eax, eax
00418D2D |. 45 |inc ebp
00418D2E |. 83C6 04 |add esi, 4
00418D31 |. F2:AE |repne scas byte ptr es:\[edi\]
00418D33 |. F7D1 |not ecx
00418D35 |. 49 |dec ecx
00418D36 |. 3BD9 |cmp ebx, ecx
00418D38 |.^ 72 AE \\jb short 00418CE8
00418D3A |> 33DB xor ebx, ebx
00418D3C |. 33C0 xor eax, eax
00418D3E |. 3BEB cmp ebp, ebx
00418D40 |. 76 38 jbe short 00418D7A
00418D42 |. 8DB424 980200>lea esi, dword ptr \[esp+298\]
00418D49 |. 8DBC24 180200>lea edi, dword ptr \[esp+218\]
00418D50 |. 8D8C24 980300>lea ecx, dword ptr \[esp+398\]
00418D57 |> A8 01 /test al, 1 ; 关键点。。将注册码第二部分分解成两部分
00418D59 |. 75 0A |jnz short 00418D65 ; al 做奇偶标志
00418D5B |. 8A11 |mov dl, byte ptr \[ecx\] ; 奇数位的做为第一部分。。偶数位的做第二部分
00418D5D |. 80C2 30 |add dl, 30
00418D60 |. 8817 |mov byte ptr \[edi\], dl
00418D62 |. 47 |inc edi
00418D63 |. EB 0D |jmp short 00418D72
00418D65 |> 83F8 0C |cmp eax, 0C
00418D68 |. 73 08 |jnb short 00418D72
00418D6A |. 8A11 |mov dl, byte ptr \[ecx\]
00418D6C |. 80C2 30 |add dl, 30
00418D6F |. 8816 |mov byte ptr \[esi\], dl
00418D71 |. 46 |inc esi
00418D72 |> 40 |inc eax
00418D73 |. 83C1 04 |add ecx, 4
00418D76 |. 3BC5 |cmp eax, ebp
00418D78 |.^ 72 DD \\jb short 00418D57 ; 关键
00418D7A |> B9 20000000 mov ecx, 20
00418D7F |. 33C0 xor eax, eax
00418D81 |. 8DBC24 180300>lea edi, dword ptr \[esp+318\]
00418D88 |. 33ED xor ebp, ebp
00418D8A |. F3:AB rep stos dword ptr es:\[edi\]
00418D8C |. 8DBC24 980000>lea edi, dword ptr \[esp+98\]
00418D93 |. 83C9 FF or ecx, FFFFFFFF
00418D96 |. F2:AE repne scas byte ptr es:\[edi\]
00418D98 |. F7D1 not ecx
00418D9A |. 49 dec ecx
00418D9B |. 74 63 je short 00418E00
00418D9D |> 0FBE842C 9800>/movsx eax, byte ptr \[esp+ebp+98\] ; 用“66fd”计算
00418DA5 |. 50 |push eax
00418DA6 |. 8D4C24 14 |lea ecx, dword ptr \[esp+14\]
00418DAA |. 68 70B24700 |push 0047B270 ; ASCII "%d"
00418DAF |. 51 |push ecx
00418DB0 |. 895C24 1C |mov dword ptr \[esp+1C\], ebx
00418DB4 |. E8 55FD0100 |call 00438B0E
00418DB9 |. 8D7C24 1C |lea edi, dword ptr \[esp+1C\] ; 5454102100 固定值
00418DBD |. 83C9 FF |or ecx, FFFFFFFF
00418DC0 |. 33C0 |xor eax, eax
00418DC2 |. 83C4 0C |add esp, 0C
00418DC5 |. F2:AE |repne scas byte ptr es:\[edi\]
00418DC7 |. F7D1 |not ecx
00418DC9 |. 2BF9 |sub edi, ecx
00418DCB |. 8D9424 180300>|lea edx, dword ptr \[esp+318\]
00418DD2 |. 8BF7 |mov esi, edi
00418DD4 |. 8BFA |mov edi, edx
00418DD6 |. 8BD1 |mov edx, ecx
00418DD8 |. 83C9 FF |or ecx, FFFFFFFF
00418DDB |. F2:AE |repne scas byte ptr es:\[edi\]
00418DDD |. 8BCA |mov ecx, edx
00418DDF |. 4F |dec edi
00418DE0 |. C1E9 02 |shr ecx, 2
00418DE3 |. F3:A5 |rep movs dword ptr es:\[edi\], dword ptr \[esi\]
00418DE5 |. 8BCA |mov ecx, edx
00418DE7 |. 83E1 03 |and ecx, 3
00418DEA |. 45 |inc ebp
00418DEB |. F3:A4 |rep movs byte ptr es:\[edi\], byte ptr \[esi\]
00418DED |. 8DBC24 980000>|lea edi, dword ptr \[esp+98\]
00418DF4 |. 83C9 FF or ecx, FFFFFFFF
00418DF7 |. F2:AE repne scas byte ptr es:\[edi\]
00418DF9 |. F7D1 not ecx
00418DFB |. 49 dec ecx
00418DFC |. 3BE9 cmp ebp, ecx
00418DFE |.^ 72 9D \\jb short 00418D9D
00418E00 |> 8D8424 180200>lea eax, dword ptr \[esp+218\]
00418E07 |. 6A 06 push 6 ; 比较 6 位
00418E09 |. 8D8C24 1C0300>lea ecx, dword ptr \[esp+31C\] ; "5454102100"
00418E10 |. 50 push eax ; 假码第二部分奇数位
00418E11 |. 51 push ecx
00418E12 |. E8 29030200 call 00439140 ; 比较是否相等
00418E17 |. 83C4 0C add esp, 0C
00418E1A |. 85C0 test eax, eax
00418E1C |. 5D pop ebp
00418E1D |. 5B pop ebx
00418E1E |. 75 2E jnz short 00418E4E ; 不等跳向错误提示
00418E20 |. 8D9424 900100>lea edx, dword ptr \[esp+190\] ; 上面计算出的那个长整形从个位往高位取出的 6 位数
00418E27 |. 6A 06 push 6 ; 比较 6 位
00418E29 |. 8D8424 940200>lea eax, dword ptr \[esp+294\] ; 假码第二部分偶数位
00418E30 |. 52 push edx
00418E31 |. 50 push eax
00418E32 |. E8 09030200 call 00439140 ; 比较是否相等
00418E37 |. 83C4 0C add esp, 0C
00418E3A |. 85C0 test eax, eax
00418E3C |. 75 10 jnz short 00418E4E ; 不等跳向错误提示
00418E3E |. 5F pop edi
00418E3F |. B8 01000000 mov eax, 1
00418E44 |. 5E pop esi
00418E45 |. 81C4 88050000 add esp, 588
00418E4B |. C2 0400 retn 4
00418E4E |> 5F pop edi
00418E4F |. 33C0 xor eax, eax
00418E51 |. 5E pop esi
00418E52 |. 81C4 88050000 add esp, 588
00418E58 \\. C2 0400 retn 4
}
```



# 算法总结

1. 注册算法使用注册码，用户名不参与运算。
2. 注册码分三部分用 `-` 分开。
3. 注册码第一部分前 4 个字符必须为 `66fd`，第三部分必须为数字。

# 算法注册机

代码:

```cpp
void CKeyDlg::OnKeyGen() 
{

  char serial[]="66fd-5454102100-";
  CString result;
  int part3;
  double eax,edx,ecx;
  double temp=1112;

  srand( (unsigned)time( NULL ) );
  part3=rand()%10000;
  m_serial.Format("%s%d",serial,part3);
     
  eax=part3;
  edx=eax+0x4cf;
  ecx=eax+0x44e;
  eax*=0x27ae6;
  part3=sqrt((edx*ecx)+eax)*temp;

  int b=(int)part3;
  result.Format("%d",b);
  int len=result.GetLength()-1;
  int j=6;
  for(int i=0;i<6;i++)
  {
     
    m_serial.Insert(j,result.Mid((len-i),1));
    j+=2;
  }

  UpdateData(false);
}
```

# 注册信息

保存在 `HKEY_LOCAL_MACHINE\SOFTWARE\Bluefox Studio\b-mov-converter`

提供一组注册码：`66fd-5549504913032100-2622`

# 版权声明

破文是学习的手记，兴趣是成功的源泉；本破文纯属技术交流, 转载请注明作者并保持文章的完整, 谢谢!
