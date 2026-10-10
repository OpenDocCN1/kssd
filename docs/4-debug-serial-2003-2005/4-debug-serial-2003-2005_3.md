

# 《学生体质健康标准管理系统》中小学版

> 原文：[`www.pediy.com/kssd/pediy06/pediy6049.htm`](https://www.pediy.com/kssd/pediy06/pediy6049.htm)

《学生体质健康标准管理系统》中小学版破解手记   
整理日期： 2003 年 05 月 30 日   
破解菜鸟： 我是谁㊣   
软件授权： 共享软件   
使用平台： Win9x/Me/NT/2000/XP   
下载地址： http://www.cenoinfo.net/cenoinfo/download/Schsmis2.2Demo.exe   
加密方式： CDKEY 保护   
功能限制： 功能限制   
破解日期： 2003 年 05 月 29 日   
演示名称： 《\*\*\*\*\*\*\*\*\*\*\*管理系统》中、小学版    
版本号码： Ver 2.2    
应用范围： 各中、小学校及教育管理部门    
适用平台： Windows 9x/Me/2000/xp   
使用工具，Gold893 by  yaoyuan\[CCG\]，OLYYDBG1.09 版 PEID8 等，GOGOGO!!!!   
  
用 PEID8 查看安装后的程序为 DELPHI6.0 编写，发现程序反 WASD，DEDE2.5 等，只好用上 yaoyuan\[CCG\]的作品，反汇编找关键部位后用 OLLYDBG 载入跟踪后分析如下：   
:0045E9D3 E8D853FEFF              call 00443DB0 //关键调用跟进   
:0045E9D8 84C0                    test al, al //关键比较   
:0045E9DA 744B                    je 0045EA27 //跳就去死   
\* Possible StringData Ref from Code Obj ->"注册成功射射使用"   
                                  |   
:0045E9DC BA18EB4500              mov edx, 0045EB18   
  
\* Possible StringData Ref from Code Obj ->"信息"   
                                  |   
:0045E9E1 B8E4EA4500              mov eax, 0045EAE4   
:0045E9E6 E81144FAFF              call 00402DFC   
:0045E9EB A10C3E4600              mov eax, dword ptr \[00463E0C\]   
:0045E9F0 8B00                    mov eax, dword ptr \[eax\]   
:0045E9F2 83C02C                  add eax, 0000002C   
:0045E9F5 8B55FC                  mov edx, dword ptr \[ebp\-04\]   
  
\* Reference To: rtl70.System::::LStrAsg(void()), Ord:0000h   
                                  |   
:0045E9F8 E81327FAFF              Call 00401110   
:0045E9FD A1043E4600              mov eax, dword ptr \[00463E04\]   
:0045EA02 8B00                    mov eax, dword ptr \[eax\]   
:0045EA04 8B4DFC                  mov ecx, dword ptr \[ebp\-04\]   
  
\* Possible StringData Ref from Code Obj ->"CDKEY"   
                                  |   
:0045EA07 BA34EB4500              mov edx, 0045EB34   
:0045EA0C E80F82FBFF              call 00416C20   
:0045EA11 8BC3                    mov eax, ebx   
:0045EA13 E830030000              call 0045ED48   
:0045EA18 B201                    mov dl, 01   
:0045EA1A 8B8304030000            mov eax, dword ptr \[ebx+00000304\]   
:0045EA20 E8C732FAFF              call 00401CEC   
:0045EA25 EB1E                    jmp 0045EA45   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:0045E9DA(C)   
|   
  
\* Possible StringData Ref from Code Obj ->"CDKEY 错误"   
                                  |   
:0045EA27 BAF4EA4500              mov edx, 0045EAF4   
  
\* Possible StringData Ref from Code Obj ->"错误"   
                                  |   
:0045EA2C B808EB4500              mov eax, 0045EB08   
:0045EA31 E84644FAFF              call 00402E7C   
:0045EA36 A10C3E4600              mov eax, dword ptr \[00463E0C\]   
:0045EA3B 8B00                    mov eax, dword ptr \[eax\]   
:0045EA3D 83C02C                  add eax, 0000002C   
.~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~   
:00443DB0 55                      push ebp// 从上面 CALL 来到这儿，往下看   
:00443DB1 8BEC                    mov ebp, esp   
:00443DB3 83C4EC                  add esp, FFFFFFEC   
:00443DB6 53                      push ebx   
:00443DB7 56                      push esi   
:00443DB8 57                      push edi   
:00443DB9 8955F8                  mov dword ptr \[ebp\-08\], edx   
:00443DBC 8945FC                  mov dword ptr \[ebp\-04\], eax   
:00443DBF 8B45FC                  mov eax, dword ptr \[ebp\-04\]   
  
\* Reference To: rtl70.System::::LStrAddRef(void()), Ord:0000h   
                                  |   
:00443DC2 E891D3FBFF              Call 00401158   
:00443DC7 8B45F8                  mov eax, dword ptr \[ebp\-08\]   
  
\* Reference To: rtl70.System::::LStrAddRef(void()), Ord:0000h   
                                  |   
:00443DCA E889D3FBFF              Call 00401158   
:00443DCF 8D7DF4                  lea edi, dword ptr \[ebp\-0C\]   
:00443DD2 8D75F0                  lea esi, dword ptr \[ebp\-10\]   
:00443DD5 33C0                    xor eax, eax   
:00443DD7 55                      push ebp   
:00443DD8 687A454400              push 0044457A   
:00443DDD 64FF30                  push dword ptr fs:\[eax\]   
:00443DE0 648920                  mov dword ptr fs:\[eax\], esp   
:00443DE3 33DB                    xor ebx, ebx   
:00443DE5 56                      push esi   
:00443DE6 8BCF                    mov ecx, edi   
:00443DE8 8D55EC                  lea edx, dword ptr \[ebp\-14\]   
:00443DEB 8B45FC                  mov eax, dword ptr \[ebp\-04\]   
:00443DEE E805FFFFFF              call 00443CF8   
:00443DF3 8B45F8                  mov eax, dword ptr \[ebp\-08\]   
:00443DF6 50                      push eax   
:00443DF7 B930424600              mov ecx, 00464230   
:00443DFC BA2C424600              mov edx, 0046422C   
:00443E01 B828424600              mov eax, 00464228   
:00443E06 E881070000              call 0044458C   
:00443E0B 66A128424600            mov ax, word ptr \[00464228\]   
:00443E11 E89EFEFFFF              call 00443CB4   
:00443E16 8B55FC                  mov edx, dword ptr \[ebp\-04\]   
:00443E19 3A02                    cmp al, byte ptr \[edx\]//开始第一位真假比较   
:00443E1B 0F853E070000            jne 0044455F //别让他跳哦   
:00443E21 66A12C424600            mov ax, word ptr \[0046422C\]   
:00443E27 E888FEFFFF              call 00443CB4   
:00443E2C 8B55FC                  mov edx, dword ptr \[ebp\-04\]   
:00443E2F 3A4201                  cmp al, byte ptr \[edx+01\]//第二位真假比较    
:00443E32 0F8527070000            jne 0044455F//不能跳    
:00443E38 66A130424600            mov ax, word ptr \[00464230\]   
:00443E3E E871FEFFFF              call 00443CB4   
:00443E43 8B55FC                  mov edx, dword ptr \[ebp\-04\]   
:00443E46 3A4202                  cmp al, byte ptr \[edx+02\]//第三位真假比较    
:00443E49 0F8510070000            jne 0044455F//不能跳    
:00443E4F 837DEC00                cmp dword ptr \[ebp\-14\], 00000000//比较是否为零来决定后面的计算   
:00443E53 0F85BC010000            jne 00444015 //跳过去和 16 进制数 40 比较进行比较产生另一种第五到十六位真假码比较比较方法向下看(作者不会有病吧)    
  
  
:00443E59 8BD6                    mov edx, esi   
:00443E5B 8BC7                    mov eax, edi   
:00443E5D E85AFEFFFF              call 00443CBC   
:00443E62 B91A000000              mov ecx, 0000001A   
:00443E67 33D2                    xor edx, edx   
:00443E69 F7F1                    div ecx   
:00443E6B 8BC2                    mov eax, edx   
:00443E6D E842FEFFFF              call 00443CB4   
:00443E72 8B55FC                  mov edx, dword ptr \[ebp\-04\]   
:00443E75 3A4204                  cmp al, byte ptr \[edx+04\] //第五位真假比较   
:00443E78 0F85E1060000            jne 0044455F//不能跳   
:00443E7E 8BD6                    mov edx, esi   
:00443E80 8BC7                    mov eax, edi   
:00443E82 E835FEFFFF              call 00443CBC   
:00443E87 B91A000000              mov ecx, 0000001A   
:00443E8C 33D2                    xor edx, edx   
:00443E8E F7F1                    div ecx   
:00443E90 8BC2                    mov eax, edx   
:00443E92 E81DFEFFFF              call 00443CB4   
:00443E97 8B55FC                  mov edx, dword ptr \[ebp\-04\]   
:00443E9A 3A4205                  cmp al, byte ptr \[edx+05\]//第六位真假码比较   
:00443E9D 0F85BC060000            jne 0044455F//不能跳   
:00443EA3 8BD6                    mov edx, esi   
:00443EA5 8BC7                    mov eax, edi   
:00443EA7 E810FEFFFF              call 00443CBC   
:00443EAC B91A000000              mov ecx, 0000001A   
:00443EB1 33D2                    xor edx, edx   
:00443EB3 F7F1                    div ecx   
:00443EB5 8BC2                    mov eax, edx   
:00443EB7 E8F8FDFFFF              call 00443CB4   
:00443EBC 8B55FC                  mov edx, dword ptr \[ebp\-04\]   
:00443EBF 3A4206                  cmp al, byte ptr \[edx+06\]//第七位真假码比较   
:00443EC2 0F8597060000            jne 0044455F//不能跳   
:00443EC8 8BD6                    mov edx, esi   
:00443ECA 8BC7                    mov eax, edi   
:00443ECC E8EBFDFFFF              call 00443CBC   
:00443ED1 B91A000000              mov ecx, 0000001A   
:00443ED6 33D2                    xor edx, edx   
:00443ED8 F7F1                    div ecx   
:00443EDA 8BC2                    mov eax, edx   
:00443EDC E8D3FDFFFF              call 00443CB4   
:00443EE1 8B55FC                  mov edx, dword ptr \[ebp\-04\]   
:00443EE4 3A4207                  cmp al, byte ptr \[edx+07\]//第八位真假码比较   
:00443EE7 0F8572060000            jne 0044455F//不能跳   
:00443EED 8BD6                    mov edx, esi   
:00443EEF 8BC7                    mov eax, edi   
:00443EF1 E8C6FDFFFF              call 00443CBC   
:00443EF6 B91A000000              mov ecx, 0000001A   
:00443EFB 33D2                    xor edx, edx   
:00443EFD F7F1                    div ecx   
:00443EFF 8BC2                    mov eax, edx   
:00443F01 E8AEFDFFFF              call 00443CB4   
:00443F06 8B55FC                  mov edx, dword ptr \[ebp\-04\]   
:00443F09 3A4208                  cmp al, byte ptr \[edx+08\]//第九位真假码比较   
:00443F0C 0F854D060000            jne 0044455F//不能跳   
:00443F12 8BD6                    mov edx, esi   
:00443F14 8BC7                    mov eax, edi   
:00443F16 E8A1FDFFFF              call 00443CBC   
:00443F1B B91A000000              mov ecx, 0000001A   
:00443F20 33D2                    xor edx, edx   
:00443F22 F7F1                    div ecx   
:00443F24 8BC2                    mov eax, edx   
:00443F26 E889FDFFFF              call 00443CB4   
:00443F2B 8B55FC                  mov edx, dword ptr \[ebp\-04\]   
:00443F2E 3A4209                  cmp al, byte ptr \[edx+09\]//第十位真假码比较   
:00443F31 0F8528060000            jne 0044455F//不能跳   
:00443F37 8BD6                    mov edx, esi   
:00443F39 8BC7                    mov eax, edi   
:00443F3B E87CFDFFFF              call 00443CBC   
:00443F40 B91A000000              mov ecx, 0000001A   
:00443F45 33D2                    xor edx, edx   
:00443F47 F7F1                    div ecx   
:00443F49 8BC2                    mov eax, edx   
:00443F4B E864FDFFFF              call 00443CB4   
:00443F50 8B55FC                  mov edx, dword ptr \[ebp\-04\]   
:00443F53 3A420A                  cmp al, byte ptr \[edx+0A\]//第十一位真假码比较   
:00443F56 0F8503060000            jne 0044455F//不能跳   
:00443F5C 8BD6                    mov edx, esi   
:00443F5E 8BC7                    mov eax, edi   
:00443F60 E857FDFFFF              call 00443CBC   
:00443F65 B91A000000              mov ecx, 0000001A   
:00443F6A 33D2                    xor edx, edx   
:00443F6C F7F1                    div ecx   
:00443F6E 8BC2                    mov eax, edx   
:00443F70 E83FFDFFFF              call 00443CB4   
:00443F75 8B55FC                  mov edx, dword ptr \[ebp\-04\]   
:00443F78 3A420B                  cmp al, byte ptr \[edx+0B\]//第十二位真假码比较   
:00443F7B 0F85DE050000            jne 0044455F//不能跳   
:00443F81 8BD6                    mov edx, esi   
:00443F83 8BC7                    mov eax, edi   
:00443F85 E832FDFFFF              call 00443CBC   
:00443F8A B91A000000              mov ecx, 0000001A   
:00443F8F 33D2                    xor edx, edx   
:00443F91 F7F1                    div ecx   
:00443F93 8BC2                    mov eax, edx   
:00443F95 E81AFDFFFF              call 00443CB4   
:00443F9A 8B55FC                  mov edx, dword ptr \[ebp\-04\]   
:00443F9D 3A420C                  cmp al, byte ptr \[edx+0C\]//第十三位真假码比较   
:00443FA0 0F85B9050000            jne 0044455F//不能跳   
:00443FA6 8BD6                    mov edx, esi   
:00443FA8 8BC7                    mov eax, edi   
:00443FAA E80DFDFFFF              call 00443CBC   
:00443FAF B91A000000              mov ecx, 0000001A   
:00443FB4 33D2                    xor edx, edx   
:00443FB6 F7F1                    div ecx   
:00443FB8 8BC2                    mov eax, edx   
:00443FBA E8F5FCFFFF              call 00443CB4   
:00443FBF 8B55FC                  mov edx, dword ptr \[ebp\-04\]   
:00443FC2 3A420D                  cmp al, byte ptr \[edx+0D\]//第十四位真假码比较   
:00443FC5 0F8594050000            jne 0044455F//不能跳   
:00443FCB 8BD6                    mov edx, esi   
:00443FCD 8BC7                    mov eax, edi   
:00443FCF E8E8FCFFFF              call 00443CBC   
:00443FD4 B91A000000              mov ecx, 0000001A   
:00443FD9 33D2                    xor edx, edx   
:00443FDB F7F1                    div ecx   
:00443FDD 8BC2                    mov eax, edx   
:00443FDF E8D0FCFFFF              call 00443CB4   
:00443FE4 8B55FC                  mov edx, dword ptr \[ebp\-04\]   
:00443FE7 3A420E                  cmp al, byte ptr \[edx+0E\]//第十五位真假码比较   
:00443FEA 0F856F050000            jne 0044455F//不能跳   
:00443FF0 8BD6                    mov edx, esi   
:00443FF2 8BC7                    mov eax, edi   
:00443FF4 E8C3FCFFFF              call 00443CBC   
:00443FF9 B91A000000              mov ecx, 0000001A   
:00443FFE 33D2                    xor edx, edx   
:00444000 F7F1                    div ecx   
:00444002 8BC2                    mov eax, edx   
:00444004 E8ABFCFFFF              call 00443CB4   
:00444009 8B55FC                  mov edx, dword ptr \[ebp\-04\]   
:0044400C 3A420F                  cmp al, byte ptr \[edx+0F\]//第十六位真假码比较   
:0044400F 0F854A050000            jne 0044455F//不能跳   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00443E53(C)   
|   
:00444015 837DEC40                cmp dword ptr \[ebp\-14\], 00000040//第四位是否大写字母   
:00444019 0F85BC010000            jne 004441DB//不是就跳去和 80(P)进行比较产生另一种第五到十六位真假码比较比较方法同上(作者不会有病吧)    
:0044401F 8BD6                    mov edx, esi   
:00444021 8BC7                    mov eax, edi   
:00444023 E8A0FCFFFF              call 00443CC8   
:00444028 B91A000000              mov ecx, 0000001A   
:0044402D 33D2                    xor edx, edx   
:0044402F F7F1                    div ecx   
:00444031 8BC2                    mov eax, edx   
:00444033 E87CFCFFFF              call 00443CB4   
:00444038 8B55FC                  mov edx, dword ptr \[ebp\-04\]   
:0044403B 3A4204                  cmp al, byte ptr \[edx+04\]   
:0044403E 0F851B050000            jne 0044455F   
:00444044 8BD6                    mov edx, esi   
:00444046 8BC7                    mov eax, edi   
:00444048 E87BFCFFFF              call 00443CC8   
:0044404D B91A000000              mov ecx, 0000001A   
:00444052 33D2                    xor edx, edx   
:00444054 F7F1                    div ecx   
:00444056 8BC2                    mov eax, edx   
:00444058 E857FCFFFF              call 00443CB4   
:0044405D 8B55FC                  mov edx, dword ptr \[ebp\-04\]   
:00444060 3A4205                  cmp al, byte ptr \[edx+05\]   
:00444063 0F85F6040000            jne 0044455F   
:00444069 8BD6                    mov edx, esi   
:0044406B 8BC7                    mov eax, edi   
:0044406D E856FCFFFF              call 00443CC8   
:00444072 B91A000000              mov ecx, 0000001A   
:00444077 33D2                    xor edx, edx   
:00444079 F7F1                    div ecx   
:0044407B 8BC2                    mov eax, edx   
:0044407D E832FCFFFF              call 00443CB4   
:00444082 8B55FC                  mov edx, dword ptr \[ebp\-04\]   
:00444085 3A4206                  cmp al, byte ptr \[edx+06\]   
:00444088 0F85D1040000            jne 0044455F   
:0044408E 8BD6                    mov edx, esi   
:00444090 8BC7                    mov eax, edi   
:00444092 E831FCFFFF              call 00443CC8   
:00444097 B91A000000              mov ecx, 0000001A   
:0044409C 33D2                    xor edx, edx   
:0044409E F7F1                    div ecx   
:004440A0 8BC2                    mov eax, edx   
:004440A2 E80DFCFFFF              call 00443CB4   
:004440A7 8B55FC                  mov edx, dword ptr \[ebp\-04\]   
:004440AA 3A4207                  cmp al, byte ptr \[edx+07\]   
:004440AD 0F85AC040000            jne 0044455F   
:004440B3 8BD6                    mov edx, esi   
:004440B5 8BC7                    mov eax, edi   
:004440B7 E80CFCFFFF              call 00443CC8   
:004440BC B91A000000              mov ecx, 0000001A   
:004440C1 33D2                    xor edx, edx   
:004440C3 F7F1                    div ecx   
:004440C5 8BC2                    mov eax, edx   
:004440C7 E8E8FBFFFF              call 00443CB4   
:004440CC 8B55FC                  mov edx, dword ptr \[ebp\-04\]   
:004440CF 3A4208                  cmp al, byte ptr \[edx+08\]   
:004440D2 0F8587040000            jne 0044455F   
:004440D8 8BD6                    mov edx, esi   
:004440DA 8BC7                    mov eax, edi   
:004440DC E8E7FBFFFF              call 00443CC8   
:004440E1 B91A000000              mov ecx, 0000001A   
:004440E6 33D2                    xor edx, edx   
:004440E8 F7F1                    div ecx   
:004440EA 8BC2                    mov eax, edx   
:004440EC E8C3FBFFFF              call 00443CB4   
:004440F1 8B55FC                  mov edx, dword ptr \[ebp\-04\]   
:004440F4 3A4209                  cmp al, byte ptr \[edx+09\]   
:004440F7 0F8562040000            jne 0044455F   
:004440FD 8BD6                    mov edx, esi   
:004440FF 8BC7                    mov eax, edi   
:00444101 E8C2FBFFFF              call 00443CC8   
:00444106 B91A000000              mov ecx, 0000001A   
:0044410B 33D2                    xor edx, edx   
:0044410D F7F1                    div ecx   
:0044410F 8BC2                    mov eax, edx   
:00444111 E89EFBFFFF              call 00443CB4   
:00444116 8B55FC                  mov edx, dword ptr \[ebp\-04\]   
:00444119 3A420A                  cmp al, byte ptr \[edx+0A\]   
:0044411C 0F853D040000            jne 0044455F   
:00444122 8BD6                    mov edx, esi   
:00444124 8BC7                    mov eax, edi   
:00444126 E89DFBFFFF              call 00443CC8   
:0044412B B91A000000              mov ecx, 0000001A   
:00444130 33D2                    xor edx, edx   
:00444132 F7F1                    div ecx   
:00444134 8BC2                    mov eax, edx   
:00444136 E879FBFFFF              call 00443CB4   
:0044413B 8B55FC                  mov edx, dword ptr \[ebp\-04\]   
:0044413E 3A420B                  cmp al, byte ptr \[edx+0B\]   
:00444141 0F8518040000            jne 0044455F   
:00444147 8BD6                    mov edx, esi   
:00444149 8BC7                    mov eax, edi   
:0044414B E878FBFFFF              call 00443CC8   
:00444150 B91A000000              mov ecx, 0000001A   
:00444155 33D2                    xor edx, edx   
:00444157 F7F1                    div ecx   
:00444159 8BC2                    mov eax, edx   
:0044415B E854FBFFFF              call 00443CB4   
:00444160 8B55FC                  mov edx, dword ptr \[ebp\-04\]   
:00444163 3A420C                  cmp al, byte ptr \[edx+0C\]   
:00444166 0F85F3030000            jne 0044455F   
:0044416C 8BD6                    mov edx, esi   
:0044416E 8BC7                    mov eax, edi   
:00444170 E853FBFFFF              call 00443CC8   
:00444175 B91A000000              mov ecx, 0000001A   
:0044417A 33D2                    xor edx, edx   
:0044417C F7F1                    div ecx   
:0044417E 8BC2                    mov eax, edx   
:00444180 E82FFBFFFF              call 00443CB4   
:00444185 8B55FC                  mov edx, dword ptr \[ebp\-04\]   
:00444188 3A420D                  cmp al, byte ptr \[edx+0D\]   
:0044418B 0F85CE030000            jne 0044455F   
:00444191 8BD6                    mov edx, esi   
:00444193 8BC7                    mov eax, edi   
:00444195 E82EFBFFFF              call 00443CC8   
:0044419A B91A000000              mov ecx, 0000001A   
:0044419F 33D2                    xor edx, edx   
:004441A1 F7F1                    div ecx   
:004441A3 8BC2                    mov eax, edx   
:004441A5 E80AFBFFFF              call 00443CB4   
:004441AA 8B55FC                  mov edx, dword ptr \[ebp\-04\]   
:004441AD 3A420E                  cmp al, byte ptr \[edx+0E\]   
:004441B0 0F85A9030000            jne 0044455F   
:004441B6 8BD6                    mov edx, esi   
:004441B8 8BC7                    mov eax, edi   
:004441BA E809FBFFFF              call 00443CC8   
:004441BF B91A000000              mov ecx, 0000001A   
:004441C4 33D2                    xor edx, edx   
:004441C6 F7F1                    div ecx   
:004441C8 8BC2                    mov eax, edx   
:004441CA E8E5FAFFFF              call 00443CB4   
:004441CF 8B55FC                  mov edx, dword ptr \[ebp\-04\]   
:004441D2 3A420F                  cmp al, byte ptr \[edx+0F\]   
:004441D5 0F8584030000            jne 0044455F   
  
\* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
|:00444019(C)   
|   
:004441DB 817DEC80000000          cmp dword ptr \[ebp\-14\], 00000080 //第四位字母是否为 P   
:004441E2 0F85BC010000            jne 004443A4//不是跳过去和 C0(192)进行比较产生另一种第五到十六位真假码比较比较方法同上代码略过    
  
:004441E8 8BD6                    mov edx, esi   
:004441EA 8BC7                    mov eax, edi   
:004441EC E8E3FAFFFF              call 00443CD4   
:004441F1 B91A000000              mov ecx, 0000001A   
:004441F6 33D2                    xor edx, edx   
:004441F8 F7F1                    div ecx   
:004441FA 8BC2                    mov eax, edx   
:004441FC E8B3FAFFFF              call 00443CB4   
:00444201 8B55FC                  mov edx, dword ptr \[ebp\-04\]   
:00444204 3A4204                  cmp al, byte ptr \[edx+04\]   
:00444207 0F8552030000            jne 0044455F   
:0044420D 8BD6                    mov edx, esi   
:0044420F 8BC7                    mov eax, edi   
:00444211 E8BEFAFFFF              call 00443CD4   
:00444216 B91A000000              mov ecx, 0000001A   
:0044421B 33D2                    xor edx, edx   
:0044421D F7F1                    div ecx   
:0044421F 8BC2                    mov eax, edx   
:00444221 E88EFAFFFF              call 00443CB4   
:00444226 8B55FC                  mov edx, dword ptr \[ebp\-04\]   
:00444229 3A4205                  cmp al, byte ptr \[edx+05\]   
:0044422C 0F852D030000            jne 0044455F   
:00444232 8BD6                    mov edx, esi   
:00444234 8BC7                    mov eax, edi   
:00444236 E899FAFFFF              call 00443CD4   
:0044423B B91A000000              mov ecx, 0000001A   
:00444240 33D2                    xor edx, edx   
:00444242 F7F1                    div ecx   
:00444244 8BC2                    mov eax, edx   
:00444246 E869FAFFFF              call 00443CB4   
:0044424B 8B55FC                  mov edx, dword ptr \[ebp\-04\]   
:0044424E 3A4206                  cmp al, byte ptr \[edx+06\]   
:00444251 0F8508030000            jne 0044455F   
:00444257 8BD6                    mov edx, esi   
:00444259 8BC7                    mov eax, edi   
:0044425B E874FAFFFF              call 00443CD4   
:00444260 B91A000000              mov ecx, 0000001A   
:00444265 33D2                    xor edx, edx   
:00444267 F7F1                    div ecx   
:00444269 8BC2                    mov eax, edx   
:0044426B E844FAFFFF              call 00443CB4   
:00444270 8B55FC                  mov edx, dword ptr \[ebp\-04\]   
:00444273 3A4207                  cmp al, byte ptr \[edx+07\]   
:00444276 0F85E3020000            jne 0044455F   
:0044427C 8BD6                    mov edx, esi   
:0044427E 8BC7                    mov eax, edi   
:00444280 E84FFAFFFF              call 00443CD4   
:00444285 B91A000000              mov ecx, 0000001A   
:0044428A 33D2                    xor edx, edx   
:0044428C F7F1                    div ecx   
:0044428E 8BC2                    mov eax, edx   
:00444290 E81FFAFFFF              call 00443CB4   
:00444295 8B55FC                  mov edx, dword ptr \[ebp\-04\]   
:00444298 3A4208                  cmp al, byte ptr \[edx+08\]   
:0044429B 0F85BE020000            jne 0044455F   
:004442A1 8BD6                    mov edx, esi   
:004442A3 8BC7                    mov eax, edi   
:004442A5 E82AFAFFFF              call 00443CD4   
:004442AA B91A000000              mov ecx, 0000001A   
:004442AF 33D2                    xor edx, edx   
:004442B1 F7F1                    div ecx   
:004442B3 8BC2                    mov eax, edx   
:004442B5 E8FAF9FFFF              call 00443CB4   
:004442BA 8B55FC                  mov edx, dword ptr \[ebp\-04\]   
:004442BD 3A4209                  cmp al, byte ptr \[edx+09\]   
:004442C0 0F8599020000            jne 0044455F   
:004442C6 8BD6                    mov edx, esi   
:004442C8 8BC7                    mov eax, edi   
:004442CA E805FAFFFF              call 00443CD4   
:004442CF B91A000000              mov ecx, 0000001A   
:004442D4 33D2                    xor edx, edx   
:004442D6 F7F1                    div ecx   
:004442D8 8BC2                    mov eax, edx   
:004442DA E8D5F9FFFF              call 00443CB4   
:004442DF 8B55FC                  mov edx, dword ptr \[ebp\-04\]   
:004442E2 3A420A                  cmp al, byte ptr \[edx+0A\]   
:004442E5 0F8574020000            jne 0044455F   
:004442EB 8BD6                    mov edx, esi   
:004442ED 8BC7                    mov eax, edi   
:004442EF E8E0F9FFFF              call 00443CD4   
:004442F4 B91A000000              mov ecx, 0000001A   
:004442F9 33D2                    xor edx, edx   
:004442FB F7F1                    div ecx   
:004442FD 8BC2                    mov eax, edx   
:004442FF E8B0F9FFFF              call 00443CB4   
:00444304 8B55FC                  mov edx, dword ptr \[ebp\-04\]   
:00444307 3A420B                  cmp al, byte ptr \[edx+0B\]   
:0044430A 0F854F020000            jne 0044455F   
:00444310 8BD6                    mov edx, esi   
:00444312 8BC7                    mov eax, edi   
:00444314 E8BBF9FFFF              call 00443CD4   
:00444319 B91A000000              mov ecx, 0000001A   
:0044431E 33D2                    xor edx, edx   
:00444320 F7F1                    div ecx   
:00444322 8BC2                    mov eax, edx   
:00444324 E88BF9FFFF              call 00443CB4   
:00444329 8B55FC                  mov edx, dword ptr \[ebp\-04\]   
:0044432C 3A420C                  cmp al, byte ptr \[edx+0C\]   
:0044432F 0F852A020000            jne 0044455F   
:00444335 8BD6                    mov edx, esi   
:00444337 8BC7                    mov eax, edi   
:00444339 E896F9FFFF              call 00443CD4   
:0044433E B91A000000              mov ecx, 0000001A   
:00444343 33D2                    xor edx, edx   
:00444345 F7F1                    div ecx   
:00444347 8BC2                    mov eax, edx   
:00444349 E866F9FFFF              call 00443CB4   
:0044434E 8B55FC                  mov edx, dword ptr \[ebp\-04\]   
:00444351 3A420D                  cmp al, byte ptr \[edx+0D\]   
:00444354 0F8505020000            jne 0044455F   
:0044435A 8BD6                    mov edx, esi   
:0044435C 8BC7                    mov eax, edi   
:0044435E E871F9FFFF              call 00443CD4   
:00444363 B91A000000              mov ecx, 0000001A   
:00444368 33D2                    xor edx, edx   
:0044436A F7F1                    div ecx   
:0044436C 8BC2                    mov eax, edx   
:0044436E E841F9FFFF              call 00443CB4   
:00444373 8B55FC                  mov edx, dword ptr \[ebp\-04\]   
:00444376 3A420E                  cmp al, byte ptr \[edx+0E\]   
:00444379 0F85E0010000            jne 0044455F   
:0044437F 8BD6                    mov edx, esi   
:00444381 8BC7                    mov eax, edi   
:00444383 E84CF9FFFF              call 00443CD4   
:00444388 B91A000000              mov ecx, 0000001A   
:0044438D 33D2                    xor edx, edx   
:0044438F F7F1                    div ecx   
:00444391 8BC2                    mov eax, edx   
:00444393 E81CF9FFFF              call 00443CB4   
:00444398 8B55FC                  mov edx, dword ptr \[ebp\-04\]   
:0044439B 3A420F                  cmp al, byte ptr \[edx+0F\]   
:0044439E 0F85BB010000            jne 0044455F   
|:004441E2(C)   
|   
:004443A4 817DECC0000000          cmp dword ptr \[ebp\-14\], 000000C0//这个比较产生另一种第五到第十六位真假码比较   
:004443AB 0F85AC010000            jne 0044455D//这里跳过去就会将 BL 置 1，赖得去跟了，有兴趣的朋友可心跟一下。    
:004443B1 8BD6                    mov edx, esi   
:004443B3 8BC7                    mov eax, edi   
:004443B5 E82AF9FFFF              call 00443CE4   
:004443BA B91A000000              mov ecx, 0000001A   
:004443BF 33D2                    xor edx, edx   
:004443C1 F7F1                    div ecx   
:004443C3 8BC2                    mov eax, edx   
:004443C5 E8EAF8FFFF              call 00443CB4   
:004443CA 8B55FC                  mov edx, dword ptr \[ebp\-04\]   
:004443CD 3A4204                  cmp al, byte ptr \[edx+04\]   
:004443D0 0F8589010000            jne 0044455F   
:004443D6 8BD6                    mov edx, esi   
:004443D8 8BC7                    mov eax, edi   
:004443DA E805F9FFFF              call 00443CE4   
:004443DF B91A000000              mov ecx, 0000001A   
:004443E4 33D2                    xor edx, edx   
:004443E6 F7F1                    div ecx   
:004443E8 8BC2                    mov eax, edx   
:004443EA E8C5F8FFFF              call 00443CB4   
:004443EF 8B55FC                  mov edx, dword ptr \[ebp\-04\]   
:004443F2 3A4205                  cmp al, byte ptr \[edx+05\]   
:004443F5 0F8564010000            jne 0044455F   
:004443FB 8BD6                    mov edx, esi   
:004443FD 8BC7                    mov eax, edi   
:004443FF E8E0F8FFFF              call 00443CE4   
:00444404 B91A000000              mov ecx, 0000001A   
:00444409 33D2                    xor edx, edx   
:0044440B F7F1                    div ecx   
:0044440D 8BC2                    mov eax, edx   
:0044440F E8A0F8FFFF              call 00443CB4   
:00444414 8B55FC                  mov edx, dword ptr \[ebp\-04\]   
:00444417 3A4206                  cmp al, byte ptr \[edx+06\]   
:0044441A 0F853F010000            jne 0044455F   
:00444420 8BD6                    mov edx, esi   
:00444422 8BC7                    mov eax, edi   
:00444424 E8BBF8FFFF              call 00443CE4   
:00444429 B91A000000              mov ecx, 0000001A   
:0044442E 33D2                    xor edx, edx   
:00444430 F7F1                    div ecx   
:00444432 8BC2                    mov eax, edx   
:00444434 E87BF8FFFF              call 00443CB4   
:00444439 8B55FC                  mov edx, dword ptr \[ebp\-04\]   
:0044443C 3A4207                  cmp al, byte ptr \[edx+07\]   
:0044443F 0F851A010000            jne 0044455F   
:00444445 8BD6                    mov edx, esi   
:00444447 8BC7                    mov eax, edi   
:00444449 E896F8FFFF              call 00443CE4   
:0044444E B91A000000              mov ecx, 0000001A   
:00444453 33D2                    xor edx, edx   
:00444455 F7F1                    div ecx   
:00444457 8BC2                    mov eax, edx   
:00444459 E856F8FFFF              call 00443CB4   
:0044445E 8B55FC                  mov edx, dword ptr \[ebp\-04\]   
:00444461 3A4208                  cmp al, byte ptr \[edx+08\]   
:00444464 0F85F5000000            jne 0044455F   
:0044446A 8BD6                    mov edx, esi   
:0044446C 8BC7                    mov eax, edi   
:0044446E E871F8FFFF              call 00443CE4   
:00444473 B91A000000              mov ecx, 0000001A   
:00444478 33D2                    xor edx, edx   
:0044447A F7F1                    div ecx   
:0044447C 8BC2                    mov eax, edx   
:0044447E E831F8FFFF              call 00443CB4   
:00444483 8B55FC                  mov edx, dword ptr \[ebp\-04\]   
:00444486 3A4209                  cmp al, byte ptr \[edx+09\]   
:00444489 0F85D0000000            jne 0044455F   
:0044448F 8BD6                    mov edx, esi   
:00444491 8BC7                    mov eax, edi   
:00444493 E84CF8FFFF              call 00443CE4   
:00444498 B91A000000              mov ecx, 0000001A   
:0044449D 33D2                    xor edx, edx   
:0044449F F7F1                    div ecx   
:004444A1 8BC2                    mov eax, edx   
:004444A3 E80CF8FFFF              call 00443CB4   
:004444A8 8B55FC                  mov edx, dword ptr \[ebp\-04\]   
:004444AB 3A420A                  cmp al, byte ptr \[edx+0A\]   
:004444AE 0F85AB000000            jne 0044455F   
:004444B4 8BD6                    mov edx, esi   
:004444B6 8BC7                    mov eax, edi   
:004444B8 E827F8FFFF              call 00443CE4   
:004444BD B91A000000              mov ecx, 0000001A   
:004444C2 33D2                    xor edx, edx   
:004444C4 F7F1                    div ecx   
:004444C6 8BC2                    mov eax, edx   
:004444C8 E8E7F7FFFF              call 00443CB4   
:004444CD 8B55FC                  mov edx, dword ptr \[ebp\-04\]   
:004444D0 3A420B                  cmp al, byte ptr \[edx+0B\]   
:004444D3 0F8586000000            jne 0044455F   
:004444D9 8BD6                    mov edx, esi   
:004444DB 8BC7                    mov eax, edi   
:004444DD E802F8FFFF              call 00443CE4   
:004444E2 B91A000000              mov ecx, 0000001A   
:004444E7 33D2                    xor edx, edx   
:004444E9 F7F1                    div ecx   
:004444EB 8BC2                    mov eax, edx   
:004444ED E8C2F7FFFF              call 00443CB4   
:004444F2 8B55FC                  mov edx, dword ptr \[ebp\-04\]   
:004444F5 3A420C                  cmp al, byte ptr \[edx+0C\]   
:004444F8 7565                    jne 0044455F   
:004444FA 8BD6                    mov edx, esi   
:004444FC 8BC7                    mov eax, edi   
:004444FE E8E1F7FFFF              call 00443CE4   
:00444503 B91A000000              mov ecx, 0000001A   
:00444508 33D2                    xor edx, edx   
:0044450A F7F1                    div ecx   
:0044450C 8BC2                    mov eax, edx   
:0044450E E8A1F7FFFF              call 00443CB4   
:00444513 8B55FC                  mov edx, dword ptr \[ebp\-04\]   
:00444516 3A420D                  cmp al, byte ptr \[ed 

> :00444519 7544                    jne 0044455F   
> :0044451B 8BD6                    mov edx, esi   
> :0044451D 8BC7                    mov eax, edi   
> :0044451F E8C0F7FFFF              call 00443CE4   
> :00444524 B91A000000              mov ecx, 0000001A   
> :00444529 33D2                    xor edx, edx   
> :0044452B F7F1                    div ecx   
> :0044452D 8BC2                    mov eax, edx   
> :0044452F E880F7FFFF              call 00443CB4   
> :00444534 8B55FC                  mov edx, dword ptr \[ebp\-04\]   
> :00444537 3A420E                  cmp al, byte ptr \[edx+0E\]   
> :0044453A 7523                    jne 0044455F   
> :0044453C 8BD6                    mov edx, esi   
> :0044453E 8BC7                    mov eax, edi   
> :00444540 E89FF7FFFF              call 00443CE4   
> :00444545 B91A000000              mov ecx, 0000001A   
> :0044454A 33D2                    xor edx, edx   
> :0044454C F7F1                    div ecx   
> :0044454E 8BC2                    mov eax, edx   
> :00444550 E85FF7FFFF              call 00443CB4   
> :00444555 8B55FC                  mov edx, dword ptr \[ebp\-04\]   
> :00444558 3A420F                  cmp al, byte ptr \[edx+0F\]   
> :0044455B 7502                    jne 0044455F   
> ……………………………………………………………………………………………………….   
> \* Referenced by a (U)nconditional or (C)onditional Jump at Address:   
> |:004443AB(C)   
> |   
> :0044455D B301                    mov bl, 01//往下就只好去死了，    
> 至此，将所得到的真码连起我的 cdkey 是，CBED---AYQO—YMQA---WAQQ   
> 总结：程序的注册码前三位为固定的 CEB，第四位有四种变数，即和 0 比较、和 40 比较、和 80 比较、和以 C0 比较，以此来决定后面第五到十六位的比较方法。最后感谢好好学习妹妹陪我过夜。   
> game over   
> go to bad  4dd466d0e1e0cc123afbe504da2462ca