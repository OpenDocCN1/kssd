

# 跳棋大战 V2.28

> 原文：[`www.pediy.com/kssd/pediy06/pediy6074.htm`](https://www.pediy.com/kssd/pediy06/pediy6074.htm)

对象：跳棋大战 V2.28  
作者：lordor\[BCG\]  
目的：初学破解，属技术交流，无其它目的，请不要任意散布或用用商业用途。  
工具：softice,w32Dasm,ollydbg 

试炼码：  
机器码：558109930　　  
注册码：654321 

这是一个应用古典密码学的注册算法，是典型的代换密码，密鈅很容易找到，分析如下，“;”后是我加的注解 

  
00403215   .  8BF1          MOV ESI,ECX  
00403217   .  6A 01         PUSH 1  
00403219   .  E8 FF330100   CALL jump.0041661D                       ;  取注册码  
0040321E   .  8D9E 9C000000 LEA EBX,DWORD PTR DS:\[ESI+9C\]            ;  注册码入 ebx  
00403224   .  8BCB          MOV ECX,EBX  
00403226   .  E8 6FFF0000   CALL jump.0041319A　　　　　　　　　　　 ;小写字母变大写  
0040322B   .  6A 00         PUSH 0  
0040322D   .  8BCB          MOV ECX,EBX  
0040322F   .  E8 CFFE0000   CALL jump.00413103  
00403234   .  6A 0A         PUSH 0A                                  ; /pFileSystemNameSize = 0000000A  
00403236   .  8BF8          MOV EDI,EAX                              ; |  
00403238   .  6A 00         PUSH 0                                   ; |pFileSystemNameBuffer = NULL  
0040323A   .  6A 00         PUSH 0                                   ; |pFileSystemFlags = NULL  
0040323C   .  8D4424 1C     LEA EAX,DWORD PTR SS:\[ESP+1C\]            ; |  
00403240   .  6A 00         PUSH 0                                   ; |pMaxFilenameLength = NULL  
00403242   .  50            PUSH EAX                                 ; |pVolumeSerialNumber  
00403243   .  6A 0C         PUSH 0C                                  ; |MaxVolumeNameSize = C (12.)  
00403245   .  6A 00         PUSH 0                                   ; |VolumeNameBuffer = NULL  
00403247   .  68 88214200   PUSH jump.00422188                       ; |RootPathName = "c:\\"  
0040324C   .  FF15 60C14100 CALL DWORD PTR DS:\[<&KERNEL32.GetVolumeI>; \\GetVolumeInformationA  
00403252   .  8B4C24 10     MOV ECX,DWORD PTR SS:\[ESP+10\]            ;  取 C 盘序列号,ecx\=214414EA，转换为十进制（558109930）即为机器码  
00403256   .  85C9          TEST ECX,ECX                             ;  C 盘序列号是否为 0  
00403258   .  74 2F         JE SHORT jump.00403289  
0040325A   >  8BC1          MOV EAX,ECX                              ;  机器码入 eax  
0040325C   .  33D2          XOR EDX,EDX                              ;  edx 清空  
0040325E   .  BD 1A000000   MOV EBP,1A                               ;  1A 入 ebp  
00403263   .  F7F5          DIV EBP                                  ;  eax 除以 ebp，余数入 edx  
00403265   .  0FBE07        MOVSX EAX,BYTE PTR DS:\[EDI\]              ;  注册码一位入 eax  
00403268   .  8B1495 C02042>MOV EDX,DWORD PTR DS:\[EDX\*4+4220C0\]      ;  edx\=\[edx\*4+4220c0\]，即为取密码表  
0040326F   .  83C2 41       ADD EDX,41                               ;  edx\=edx+41，edx 转换为大写字母  
00403272   .  47            INC EDI                                  ;  下一位注册码  
00403273   .  3BD0          CMP EDX,EAX                              ;  运算后的 1 位机器码与注册码 1 位是否相等  
00403275      75 59         JNZ SHORT jump.004032D0                  ;  不等出错  
00403277   .  B8 4FECC44E   MOV EAX,4EC4EC4F                         ;  4EC4EC4F 入 eax  
0040327C   .  F7E1          MUL ECX                                  ;  乘以机器码，产生 16 位的大数，此数高 8 位入 edx,低 8 位入 eax  
0040327E   .  C1EA 03       SHR EDX,3                                ;  高 8 位右移 3 位，即除以 8  
00403281   .  8BCA          MOV ECX,EDX                              ;  ecx\=edx  
00403283   .  894C24 10     MOV DWORD PTR SS:\[ESP+10\],ECX            ;  把 ecx 压入\[esp+10\]中（用来替换机器码，）  
00403287   .^ 75 D1         JNZ SHORT jump.0040325A                  ;  循环，直到高 8 位数为 0  
00403289   >  E8 41650100   CALL jump.004197CF  
0040328E   .  8B0B          MOV ECX,DWORD PTR DS:\[EBX\]  
00403290   .  8B15 24394200 MOV EDX,DWORD PTR DS:\[423924\]  
00403296   .  8B40 04       MOV EAX,DWORD PTR DS:\[EAX+4\]  
00403299   .  51            PUSH ECX                                 ; /Arg3  
0040329A   .  8B0D 28394200 MOV ECX,DWORD PTR DS:\[423928\]            ; |  
004032A0   .  52            PUSH EDX                                 ; |Arg2 => 00E96EE8 ASCII "CODE"  
004032A1   .  51            PUSH ECX                                 ; |Arg1 => 00E96E98 ASCII "Jump"  
004032A2   .  8BC8          MOV ECX,EAX                              ; |  
004032A4   .  E8 88560100   CALL jump.00418931                       ; \\jump.00418931  
004032A9   .  6A 00         PUSH 0  
004032AB   .  68 50224200   PUSH jump.00422250  
004032B0   .  68 3C224200   PUSH jump.0042223C  
004032B5   .  8BCE          MOV ECX,ESI  
004032B7   .  E8 052C0100   CALL jump.00415EC1                       ;  显示注册成功  
004032BC   .  8BCE          MOV ECX,ESI  
004032BE   .  C746 5C 01000>MOV DWORD PTR DS:\[ESI+5C\],1 

  
注册过程为：  
先求机器码与 26 的模，据模数查找密钥，密钥＋41（即转换为大写字母） ，即为一位注册码，然后机器码与 4EC4EC4F（H）相乘，取 5－8 位数，此数右移 3 位，即得新的机器码，如此重复，直到新的机器码为 0，把各位注册码接起即为注册码。 

  
在 4220C0 处查看数，相距 4 个字节存放看 26 个密钥：  
密码表：  
004220C0  0C 00 00 00 10 00 00 00  .......  
004220C8  05 00 00 00 11 00 00 00  ......  
004220D0  06 00 00 00 15 00 00 00  ......  
004220D8  19 00 00 00 09 00 00 00  .......  
004220E0  03 00 00 00 0F 00 00 00  ......  
004220E8  0B 00 00 00 18 00 00 00   ......  
004220F0  0A 00 00 00 04 00 00 00  .......  
004220F8  0E 00 00 00 02 00 00 00  ......  
00422100  17 00 00 00 08 00 00 00  ......  
00422108  01 00 00 00 16 00 00 00  ......  
00422110  12 00 00 00 13 00 00 00  ......  
00422118  07 00 00 00 00 00 00 00  .......  
00422120  0D 00 00 00 14 00 00 00  ....... 

可以加 41 后转换为大写字母为：  
M,Q,F, .........,U 

  
\---------------------------------------------------------- 

在 00403273 处下断，在 softice 中下？edx，即可看到一位注册码，修正注册码后再用同样的方法会取得全部的注册码。 

  
机器码：558109930　　  
注册码：olfdusq 

  
注册信息保存在： 

  
\[HKEY\_CURRENT\_USER\\Software\\Lzly\\jump\\Jump\]  
"CODE"="OLFDUSQ" 

cracked by lordor\[BCG\]  
03.5.26

  
 4dd466d0e1e0cc123afbe504da2462ca