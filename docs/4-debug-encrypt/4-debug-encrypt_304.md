

# 一个软件注册算法分析及注册机, 加密算法:SHA1, Tiger, Base64, Blowfish

> 原文：[`www.pediy.com/kssd/pediy07/pediy7-461.htm`](https://www.pediy.com/kssd/pediy07/pediy7-461.htm)

【原创】一个软件注册算法分析及注册机, 加密算法:SHA1, Tiger, Base64, Blowfish  
  
\--------------------------------------------------------------------------------  
【软件名称】：雪狐提醒簿 V3.0  
  
【软件下载】：http://www.onlinedown.net/soft/2469.htm  
  
【软件大小】：4644KB  
  
【软件语言】：简体中文  
  
【软件类别】：国产软件/免费版/闹铃时钟  
  
【软件限制】：注册码   
  
【破解工具】：PEiD, DeDe, OllyDbg, W32DSM  
  
【破解作者】：blackeyes  
  
【作者声明】：初学 Crack，只是感兴趣，没有其它目的。失误之处敬请诸位大侠赐教！   
  
【破解过程】:   
  
<1> 脱壳请参照 <<脱壳论坛>>上的"UltraProtect 1.x 的脱壳与修复"  
  
<2> 用 DeDe3.50 分析, 再配合 OLLYDBG 动态跟踪:  
  
输入 UserName & LicenseKey  
UserName:   blackeyes@abc.com  
LicenseKey: RMDBK0123456789ABCDEF0123456789ABCDEF0123456789ABCDEF0123456789ABCDEF01  
  
007211BC  /$  55            PUSH EBP  
007211BD  |.  8BEC          MOV EBP,ESP  
007211BF  |.  B9 04000000   MOV ECX,4  
007211C4  |>  6A 00         /PUSH 0  
007211C6  |.  6A 00         |PUSH 0  
007211C8  |.  49            |DEC ECX  
007211C9  |.^ 75 F9         \\JNZ SHORT RemindBo.007211C4  
007211CB  |.  51            PUSH ECX  
007211CC  |.  33C0          XOR EAX,EAX  
007211CE  |.  55            PUSH EBP  
007211CF  |.  68 75137200   PUSH RemindBo.00721375  
007211D4  |.  64:FF30       PUSH DWORD PTR FS:\[EAX\]  
007211D7  |.  64:8920       MOV DWORD PTR FS:\[EAX\],ESP  
007211DA  |.  6A 00         PUSH 0  
007211DC  |.  8D45 F8       LEA EAX,DWORD PTR SS:\[EBP-8\]  
007211DF  |.  50            PUSH EAX  
007211E0  |.  33C9          XOR ECX,ECX  
007211E2  |.  BA 8C137200   MOV EDX,RemindBo.0072138C                ;  ASCII "UserName"  
007211E7  |.  B8 A0137200   MOV EAX,RemindBo.007213A0                ;  ASCII "system"  
007211EC  |.  E8 1F6BD9FF   CALL RemindBo.004B7D10                   ;  读 UserName  
007211F1  |.  6A 00         PUSH 0  
007211F3  |.  8D45 EC       LEA EAX,DWORD PTR SS:\[EBP-14\]  
007211F6  |.  50            PUSH EAX  
007211F7  |.  33C9          XOR ECX,ECX  
007211F9  |.  BA B0137200   MOV EDX,RemindBo.007213B0                ;  ASCII "LicenseKey"  
007211FE  |.  B8 A0137200   MOV EAX,RemindBo.007213A0                ;  ASCII "system"  
00721203  |.  E8 086BD9FF   CALL RemindBo.004B7D10                   ;  读 LicenseKey  
00721208  |.  8B45 EC       MOV EAX,DWORD PTR SS:\[EBP-14\]  
0072120B  |.  8D55 F4       LEA EDX,DWORD PTR SS:\[EBP-C\]  
0072120E  |.  E8 9530D9FF   CALL RemindBo.004B42A8                   ; LicenseKey 解码  
00721213  |.  837D F8 00    CMP DWORD PTR SS:\[EBP-8\],0  
00721217  |.  74 0C         JE SHORT RemindBo.00721225  
00721219  |.  837D F4 00    CMP DWORD PTR SS:\[EBP-C\],0  
0072121D  |.  74 06         JE SHORT RemindBo.00721225  
0072121F  |.  C645 F3 01    MOV BYTE PTR SS:\[EBP-D\],1  
00721223  |.  EB 04         JMP SHORT RemindBo.00721229  
00721225  |>  C645 F3 00    MOV BYTE PTR SS:\[EBP-D\],0  
00721229  |>  807D F3 00    CMP BYTE PTR SS:\[EBP-D\],0  
0072122D  |.  74 6A         JE SHORT RemindBo.00721299  
0072122F  |.  8B45 F4       MOV EAX,DWORD PTR SS:\[EBP-C\]  
00721232  |.  E8 2944CEFF   CALL RemindBo.00405660  
00721237  |.  83F8 47       CMP EAX,47                               ;  LicenseKey 长是 0x47 吗?  
0072123A  |.  75 25         JNZ SHORT RemindBo.00721261  
0072123C  |.  8D45 E8       LEA EAX,DWORD PTR SS:\[EBP-18\]  
0072123F  |.  50            PUSH EAX  
00721240  |.  B9 05000000   MOV ECX,5  
00721245  |.  BA 01000000   MOV EDX,1  
0072124A  |.  8B45 F4       MOV EAX,DWORD PTR SS:\[EBP-C\]  
0072124D  |.  E8 6E46CEFF   CALL RemindBo.004058C0                   ;  取 1-5  
00721252  |.  8B45 E8       MOV EAX,DWORD PTR SS:\[EBP-18\]  
00721255  |.  BA C4137200   MOV EDX,RemindBo.007213C4                ;  ASCII "RMDBK"  
0072125A  |.  E8 4D45CEFF   CALL RemindBo.004057AC                   ;  是 "RMDBK" 吗?  
0072125F  |.  74 32         JE SHORT RemindBo.00721293  
00721261  |>  8B45 F4       MOV EAX,DWORD PTR SS:\[EBP-C\]  
00721264  |.  E8 F743CEFF   CALL RemindBo.00405660  
00721269  |.  83F8 0D       CMP EAX,0D  
0072126C  |.  75 2B         JNZ SHORT RemindBo.00721299  
...  
00721293  |>  C645 F2 01    MOV BYTE PTR SS:\[EBP-E\],1  
00721297  |.  EB 04         JMP SHORT RemindBo.0072129D  
00721299  |>  C645 F2 00    MOV BYTE PTR SS:\[EBP-E\],0  
0072129D  |>  8A45 F3       MOV AL,BYTE PTR SS:\[EBP-D\]  
007212A0  |.  2245 F2       AND AL,BYTE PTR SS:\[EBP-E\]  
007212A3  |.  0F84 95000000 JE RemindBo.0072133E  
007212A9  |.  8B45 F4       MOV EAX,DWORD PTR SS:\[EBP-C\]  
007212AC  |.  E8 AF43CEFF   CALL RemindBo.00405660  
007212B1  |.  83F8 47       CMP EAX,47  
007212B4  |.  75 39         JNZ SHORT RemindBo.007212EF  
007212B6  |.  68 E0137200   PUSH RemindBo.007213E0                   ;  ASCII "Remind Book"  
007212BB  |.  68 F4137200   PUSH RemindBo.007213F4                   ;  ASCII "3.0"  
007212C0  |.  8D45 E0       LEA EAX,DWORD PTR SS:\[EBP-20\]  
007212C3  |.  50            PUSH EAX  
007212C4  |.  8B45 F4       MOV EAX,DWORD PTR SS:\[EBP-C\]  
007212C7  |.  E8 9443CEFF   CALL RemindBo.00405660  
007212CC  |.  8BC8          MOV ECX,EAX  
007212CE  |.  83E9 05       SUB ECX,5  
007212D1  |.  BA 06000000   MOV EDX,6  
007212D6  |.  8B45 F4       MOV EAX,DWORD PTR SS:\[EBP-C\]  
007212D9  |.  E8 E245CEFF   CALL RemindBo.004058C0                   ;  取 LicenseKey 的第 6 位 到 最后一位  
007212DE  |.  8B4D E0       MOV ECX,DWORD PTR SS:\[EBP-20\]  
007212E1  |.  33D2          XOR EDX,EDX  
007212E3  |.  8B45 F8       MOV EAX,DWORD PTR SS:\[EBP-8\]  
007212E6  |.  E8 5524D9FF   CALL RemindBo.004B3740                   ; F7 !!!!  
007212EB  |.  84C0          TEST AL,AL                               ; 不能跳  
007212ED  |.  75 46         JNZ SHORT RemindBo.00721335  
  
  
004B3740   $  55            PUSH EBP  
004B3741   .  8BEC          MOV EBP,ESP  
004B3743   .  51            PUSH ECX  
004B3744   .  B9 1B000000   MOV ECX,1B  
004B3749   >  6A 00         PUSH 0  
004B374B   .  6A 00         PUSH 0  
004B374D   .  49            DEC ECX  
004B374E   .^ 75 F9         JNZ SHORT RemindBo.004B3749  
...  
004B37CA   .  33C0          XOR EAX,EAX  
004B37CC   .  55            PUSH EBP  
004B37CD   .  68 1E3C4B00   PUSH RemindBo.004B3C1E  
004B37D2   .  64:FF30       PUSH DWORD PTR FS:\[EAX\]  
004B37D5   .  64:8920       MOV DWORD PTR FS:\[EAX\],ESP  
004B37D8   .  8B45 EC       MOV EAX,DWORD PTR SS:\[EBP-14\]  
004B37DB   .  8B10          MOV EDX,DWORD PTR DS:\[EAX\]  
004B37DD   .  FF52 40       CALL DWORD PTR DS:\[EDX+40\]               ;  SHA1Reset()  
004B37E0   .  8D45 A8       LEA EAX,DWORD PTR SS:\[EBP-58\]  
004B37E3   .  8B4D 08       MOV ECX,DWORD PTR SS:\[EBP+8\]             ;  "3.0"  
004B37E6   .  8B55 FC       MOV EDX,DWORD PTR SS:\[EBP-4\]             ;  "blackeyes@abc.com"  
004B37E9   .  E8 BE1EF5FF   CALL RemindBo.004056AC  
004B37EE   .  8B55 A8       MOV EDX,DWORD PTR SS:\[EBP-58\]            ;  "blackeyes@abc.com3.0"  
004B37F1   .  8B45 EC       MOV EAX,DWORD PTR SS:\[EBP-14\]  
004B37F4   .  E8 638DFFFF   CALL RemindBo.004AC55C                   ;  SHA1Input()  
004B37F9   .  8D55 AC       LEA EDX,DWORD PTR SS:\[EBP-54\]  
004B37FC   .  8B45 EC       MOV EAX,DWORD PTR SS:\[EBP-14\]  
004B37FF   .  8B08          MOV ECX,DWORD PTR DS:\[EAX\]  
004B3801   .  FF51 44       CALL DWORD PTR DS:\[ECX+44\]               ;  SHA1Result()  
004B3804   .  8B45 EC       MOV EAX,DWORD PTR SS:\[EBP-14\]  
004B3807   .  8B10          MOV EDX,DWORD PTR DS:\[EAX\]  
004B3809   .  FF52 38       CALL DWORD PTR DS:\[EDX+38\]  
004B380C   .  8945 DC       MOV DWORD PTR SS:\[EBP-24\],EAX  
004B380F   .  8D45 D8       LEA EAX,DWORD PTR SS:\[EBP-28\]  
004B3812   .  E8 6D1BF5FF   CALL RemindBo.00405384  
004B3817   .  8B45 DC       MOV EAX,DWORD PTR SS:\[EBP-24\]            ;  bits\_num = A0  
004B381A   .  85C0          TEST EAX,EAX  
004B381C   .  79 03         JNS SHORT RemindBo.004B3821  
004B381E   .  83C0 07       ADD EAX,7  
004B3821   >  C1F8 03       SAR EAX,3  
004B3824   .  48            DEC EAX  
004B3825   .  85C0          TEST EAX,EAX  
004B3827   .  7C 33         JL SHORT RemindBo.004B385C  
004B3829   .  40            INC EAX  
004B382A   .  8945 CC       MOV DWORD PTR SS:\[EBP-34\],EAX            ;  bytes\_num = 14  
004B382D   .  C745 E0 00000>MOV DWORD PTR SS:\[EBP-20\],0  
004B3834   >  8D4D A4       LEA ECX,DWORD PTR SS:\[EBP-5C\]  
004B3837   .  8B45 E0       MOV EAX,DWORD PTR SS:\[EBP-20\]  
004B383A   .  0FB64405 AC   MOVZX EAX,BYTE PTR SS:\[EBP+EAX-54\]  
004B383F   .  BA 02000000   MOV EDX,2  
004B3844   .  E8 4F72F5FF   CALL RemindBo.0040AA98                   ;  printf("%02X", xx\[i\]) ?!  
004B3849   .  8B55 A4       MOV EDX,DWORD PTR SS:\[EBP-5C\]  
004B384C   .  8D45 D8       LEA EAX,DWORD PTR SS:\[EBP-28\]  
004B384F   .  E8 141EF5FF   CALL RemindBo.00405668                   ;  strcat() ?!  
004B3854   .  FF45 E0       INC DWORD PTR SS:\[EBP-20\]  
004B3857   .  FF4D CC       DEC DWORD PTR SS:\[EBP-34\]  
004B385A   .^ 75 D8         JNZ SHORT RemindBo.004B3834  
004B385C   >  8B45 F4       MOV EAX,DWORD PTR SS:\[EBP-C\]             ;  KeyLicense  
004B385F   .  E8 FC1DF5FF   CALL RemindBo.00405660                   ;  len = strlen()  
004B3864   .  8B55 F4       MOV EDX,DWORD PTR SS:\[EBP-C\]  
004B3867   .  8A4402 FF     MOV AL,BYTE PTR DS:\[EDX+EAX-1\]           ;  key\[len-1\]  
  
sha1("blackeyes@abc.com3.0") = "27 32 3F 43 79 AE 84 1F A0 DA 14 74 50 D3 39 F3 C8 66 A9 F2"  
004B386B   .  8B55 D8       MOV EDX,DWORD PTR SS:\[EBP-28\]            ;  SHA1\_hash\[...\]  
004B386E   .  3A42 11       CMP AL,BYTE PTR DS:\[EDX+11\]              ;  key\[len-1\] == S\[11\]?  
004B3871   .  74 0A         JE SHORT RemindBo.004B387D  
004B3873   .  E8 0C15F5FF   CALL RemindBo.00404D84  
004B3878   .  E9 A8030000   JMP RemindBo.004B3C25  
004B387D   >  8B45 E4       MOV EAX,DWORD PTR SS:\[EBP-1C\]            ;  Tiger\_Context  
004B3880   .  8B10          MOV EDX,DWORD PTR DS:\[EAX\]  
004B3882   .  FF52 40       CALL DWORD PTR DS:\[EDX+40\]               ;  Tiger\_init()  
004B3885   .  8D45 98       LEA EAX,DWORD PTR SS:\[EBP-68\]  
004B3888   .  8B55 D8       MOV EDX,DWORD PTR SS:\[EBP-28\]  
004B388B   .  8A52 08       MOV DL,BYTE PTR DS:\[EDX+8\]               ;  S\[8\]  
004B388E   .  8850 01       MOV BYTE PTR DS:\[EAX+1\],DL  
004B3891   .  C600 01       MOV BYTE PTR DS:\[EAX\],1  
004B3894   .  8D55 98       LEA EDX,DWORD PTR SS:\[EBP-68\]  
004B3897   .  8D45 94       LEA EAX,DWORD PTR SS:\[EBP-6C\]  
004B389A   .  E8 6DFEF4FF   CALL RemindBo.0040370C  
004B389F   .  8D45 90       LEA EAX,DWORD PTR SS:\[EBP-70\]  
004B38A2   .  8B55 D8       MOV EDX,DWORD PTR SS:\[EBP-28\]  
004B38A5   .  8A52 1D       MOV DL,BYTE PTR DS:\[EDX+1D\]              ;  S\[1D\]  
004B38A8   .  8850 01       MOV BYTE PTR DS:\[EAX+1\],DL  
004B38AB   .  C600 01       MOV BYTE PTR DS:\[EAX\],1  
004B38AE   .  8D55 90       LEA EDX,DWORD PTR SS:\[EBP-70\]  
004B38B1   .  8D45 94       LEA EAX,DWORD PTR SS:\[EBP-6C\]  
004B38B4   .  B1 02         MOV CL,2  
004B38B6   .  E8 21FEF4FF   CALL RemindBo.004036DC  
004B38BB   .  8D55 94       LEA EDX,DWORD PTR SS:\[EBP-6C\]  
004B38BE   .  8D45 9C       LEA EAX,DWORD PTR SS:\[EBP-64\]  
004B38C1   .  E8 221DF5FF   CALL RemindBo.004055E8  
004B38C6   .  FF75 9C       PUSH DWORD PTR SS:\[EBP-64\]  
004B38C9   .  FF75 F8       PUSH DWORD PTR SS:\[EBP-8\]  
004B38CC   .  8D45 8C       LEA EAX,DWORD PTR SS:\[EBP-74\]  
004B38CF   .  8B55 D8       MOV EDX,DWORD PTR SS:\[EBP-28\]  
004B38D2   .  8A52 11       MOV DL,BYTE PTR DS:\[EDX+11\]              ;  S\[11\]  
004B38D5   .  E8 921CF5FF   CALL RemindBo.0040556C  
004B38DA   .  FF75 8C       PUSH DWORD PTR SS:\[EBP-74\]  
004B38DD   .  8D45 88       LEA EAX,DWORD PTR SS:\[EBP-78\]  
004B38E0   .  8B55 D8       MOV EDX,DWORD PTR SS:\[EBP-28\]  
004B38E3   .  8A52 02       MOV DL,BYTE PTR DS:\[EDX+2\]               ;  S\[2\]  
004B38E6   .  E8 811CF5FF   CALL RemindBo.0040556C  
004B38EB   .  FF75 88       PUSH DWORD PTR SS:\[EBP-78\]  
004B38EE   .  FF75 0C       PUSH DWORD PTR SS:\[EBP+C\]  
004B38F1   .  8D45 84       LEA EAX,DWORD PTR SS:\[EBP-7C\]  
004B38F4   .  8B55 D8       MOV EDX,DWORD PTR SS:\[EBP-28\]  
004B38F7   .  8A52 26       MOV DL,BYTE PTR DS:\[EDX+26\]              ;  S\[26\]  
004B38FA   .  E8 6D1CF5FF   CALL RemindBo.0040556C  
004B38FF   .  FF75 84       PUSH DWORD PTR SS:\[EBP-7C\]  
004B3902   .  8D45 80       LEA EAX,DWORD PTR SS:\[EBP-80\]  
004B3905   .  8B55 D8       MOV EDX,DWORD PTR SS:\[EBP-28\]  
004B3908   .  8A52 08       MOV DL,BYTE PTR DS:\[EDX+8\]               ;  S\[8\]  
004B390B   .  E8 5C1CF5FF   CALL RemindBo.0040556C  
004B3910   .  FF75 80       PUSH DWORD PTR SS:\[EBP-80\]  
004B3913   .  8D45 A0       LEA EAX,DWORD PTR SS:\[EBP-60\]  
004B3916   .  BA 07000000   MOV EDX,7  
004B391B   .  E8 001EF5FF   CALL RemindBo.00405720  
004B3920   .  8B55 A0       MOV EDX,DWORD PTR SS:\[EBP-60\]            ;  "7907Remind BookF7"  
004B3923   .  8B45 E4       MOV EAX,DWORD PTR SS:\[EBP-1C\]  
004B3926   .  E8 318CFFFF   CALL RemindBo.004AC55C  
004B392B   .  8D55 AC       LEA EDX,DWORD PTR SS:\[EBP-54\]            ;  Tiger\_input()  
004B392E   .  8B45 E4       MOV EAX,DWORD PTR SS:\[EBP-1C\]  
004B3931   .  8B08          MOV ECX,DWORD PTR DS:\[EAX\]  
004B3933   .  FF51 44       CALL DWORD PTR DS:\[ECX+44\]               ;  Tiger\_result()  
004B3936   .  8B45 E4       MOV EAX,DWORD PTR SS:\[EBP-1C\]  
004B3939   .  8B10          MOV EDX,DWORD PTR DS:\[EAX\]  
004B393B   .  FF52 38       CALL DWORD PTR DS:\[EDX+38\]               ;  bits\_num = C0  
004B393E   .  8945 DC       MOV DWORD PTR SS:\[EBP-24\],EAX  
004B3941   .  8D45 D4       LEA EAX,DWORD PTR SS:\[EBP-2C\]  
004B3944   .  E8 3B1AF5FF   CALL RemindBo.00405384  
004B3949   .  8B45 DC       MOV EAX,DWORD PTR SS:\[EBP-24\]  
004B394C   .  85C0          TEST EAX,EAX  
004B394E   .  79 03         JNS SHORT RemindBo.004B3953  
004B3950   .  83C0 07       ADD EAX,7  
004B3953   >  C1F8 03       SAR EAX,3  
004B3956   .  48            DEC EAX  
004B3957   .  85C0          TEST EAX,EAX  
004B3959   .  7C 39         JL SHORT RemindBo.004B3994  
004B395B   .  40            INC EAX  
004B395C   .  8945 CC       MOV DWORD PTR SS:\[EBP-34\],EAX            ;  bytes\_num=18  
004B395F   .  C745 E0 00000>MOV DWORD PTR SS:\[EBP-20\],0  
004B3966   >  8D8D 7CFFFFFF LEA ECX,DWORD PTR SS:\[EBP-84\]  
004B396C   .  8B45 E0       MOV EAX,DWORD PTR SS:\[EBP-20\]  
004B396F   .  0FB64405 AC   MOVZX EAX,BYTE PTR SS:\[EBP+EAX-54\]  
004B3974   .  BA 02000000   MOV EDX,2  
004B3979   .  E8 1A71F5FF   CALL RemindBo.0040AA98                   ;  printf("%02X", xx\[i\]) ?!  
004B397E   .  8B95 7CFFFFFF MOV EDX,DWORD PTR SS:\[EBP-84\]  
004B3984   .  8D45 D4       LEA EAX,DWORD PTR SS:\[EBP-2C\]  
004B3987   .  E8 DC1CF5FF   CALL RemindBo.00405668  
004B398C   .  FF45 E0       INC DWORD PTR SS:\[EBP-20\]  
004B398F   .  FF4D CC       DEC DWORD PTR SS:\[EBP-34\]  
004B3992   .^ 75 D2         JNZ SHORT RemindBo.004B3966  
004B3994   >  8B45 F4       MOV EAX,DWORD PTR SS:\[EBP-C\]             ;  key\[...\]  
004B3997   .  8A00          MOV AL,BYTE PTR DS:\[EAX\]                 ;  key\[0\]  
  
Tiger("7903Remind BookF7") = "DEBC578DAF2E2830CF987AEEA535DF8EBF642CF697B59355"  
004B3999   .  8B55 D4       MOV EDX,DWORD PTR SS:\[EBP-2C\]            ;  Tiger\[0...2F\]  
004B399C   .  3A42 23       CMP AL,BYTE PTR DS:\[EDX+23\]              ;  K\[0\] == T\[23\]?  
004B399F   .  74 0A         JE SHORT RemindBo.004B39AB  
004B39A1   .  E8 DE13F5FF   CALL RemindBo.00404D84  
004B39A6   .  E9 7A020000   JMP RemindBo.004B3C25  
004B39AB   >  8D45 98       LEA EAX,DWORD PTR SS:\[EBP-68\]  
004B39AE   .  8B55 D8       MOV EDX,DWORD PTR SS:\[EBP-28\]  
004B39B1   .  8A52 06       MOV DL,BYTE PTR DS:\[EDX+6\]               ;  S\[6\]  
004B39B4   .  8850 01       MOV BYTE PTR DS:\[EAX+1\],DL  
004B39B7   .  C600 01       MOV BYTE PTR DS:\[EAX\],1  
004B39BA   .  8D55 98       LEA EDX,DWORD PTR SS:\[EBP-68\]  
004B39BD   .  8D45 94       LEA EAX,DWORD PTR SS:\[EBP-6C\]  
004B39C0   .  E8 47FDF4FF   CALL RemindBo.0040370C  
004B39C5   .  8D45 90       LEA EAX,DWORD PTR SS:\[EBP-70\]  
004B39C8   .  8B55 D4       MOV EDX,DWORD PTR SS:\[EBP-2C\]  
004B39CB   .  8A52 02       MOV DL,BYTE PTR DS:\[EDX+2\]               ; T\[2\]  
004B39CE   .  8850 01       MOV BYTE PTR DS:\[EAX+1\],DL  
004B39D1   .  C600 01       MOV BYTE PTR DS:\[EAX\],1  
004B39D4   .  8D55 90       LEA EDX,DWORD PTR SS:\[EBP-70\]  
004B39D7   .  8D45 94       LEA EAX,DWORD PTR SS:\[EBP-6C\]  
004B39DA   .  B1 02         MOV CL,2  
004B39DC   .  E8 FBFCF4FF   CALL RemindBo.004036DC  
004B39E1   .  8D55 94       LEA EDX,DWORD PTR SS:\[EBP-6C\]  
004B39E4   .  8D85 78FFFFFF LEA EAX,DWORD PTR SS:\[EBP-88\]  
004B39EA   .  E8 1DFDF4FF   CALL RemindBo.0040370C  
004B39EF   .  8D45 90       LEA EAX,DWORD PTR SS:\[EBP-70\]  
004B39F2   .  8B55 D8       MOV EDX,DWORD PTR SS:\[EBP-28\]  
004B39F5   .  8A52 1A       MOV DL,BYTE PTR DS:\[EDX+1A\]              ;  S\[1A\]  
004B39F8   .  8850 01       MOV BYTE PTR DS:\[EAX+1\],DL  
004B39FB   .  C600 01       MOV BYTE PTR DS:\[EAX\],1  
004B39FE   .  8D55 90       LEA EDX,DWORD PTR SS:\[EBP-70\]  
004B3A01   .  8D85 78FFFFFF LEA EAX,DWORD PTR SS:\[EBP-88\]  
004B3A07   .  B1 03         MOV CL,3  
004B3A09   .  E8 CEFCF4FF   CALL RemindBo.004036DC  
004B3A0E   .  8D95 78FFFFFF LEA EDX,DWORD PTR SS:\[EBP-88\]  
004B3A14   .  8D85 70FFFFFF LEA EAX,DWORD PTR SS:\[EBP-90\]  
004B3A1A   .  E8 EDFCF4FF   CALL RemindBo.0040370C  
004B3A1F   .  8D45 90       LEA EAX,DWORD PTR SS:\[EBP-70\]  
004B3A22   .  8B55 D4       MOV EDX,DWORD PTR SS:\[EBP-2C\]  
004B3A25   .  8A52 1D       MOV DL,BYTE PTR DS:\[EDX+1D\]              ;  T\[1D\]  
004B3A28   .  8850 01       MOV BYTE PTR DS:\[EAX+1\],DL  
004B3A2B   .  C600 01       MOV BYTE PTR DS:\[EAX\],1  
004B3A2E   .  8D55 90       LEA EDX,DWORD PTR SS:\[EBP-70\]  
004B3A31   .  8D85 70FFFFFF LEA EAX,DWORD PTR SS:\[EBP-90\]  
004B3A37   .  B1 04         MOV CL,4  
004B3A39   .  E8 9EFCF4FF   CALL RemindBo.004036DC  
004B3A3E   .  8D95 70FFFFFF LEA EDX,DWORD PTR SS:\[EBP-90\]  
004B3A44   .  8D85 68FFFFFF LEA EAX,DWORD PTR SS:\[EBP-98\]  
004B3A4A   .  E8 BDFCF4FF   CALL RemindBo.0040370C  
004B3A4F   .  8D45 90       LEA EAX,DWORD PTR SS:\[EBP-70\]  
004B3A52   .  8B55 D8       MOV EDX,DWORD PTR SS:\[EBP-28\]  
004B3A55   .  8A52 10       MOV DL,BYTE PTR DS:\[EDX+10\]              ;  S\[10\]  
004B3A58   .  8850 01       MOV BYTE PTR DS:\[EAX+1\],DL  
004B3A5B   .  C600 01       MOV BYTE PTR DS:\[EAX\],1  
004B3A5E   .  8D55 90       LEA EDX,DWORD PTR SS:\[EBP-70\]  
004B3A61   .  8D85 68FFFFFF LEA EAX,DWORD PTR SS:\[EBP-98\]  
004B3A67   .  B1 05         MOV CL,5  
004B3A69   .  E8 6EFCF4FF   CALL RemindBo.004036DC  
004B3A6E   .  8D95 68FFFFFF LEA EDX,DWORD PTR SS:\[EBP-98\]  
004B3A74   .  8D85 60FFFFFF LEA EAX,DWORD PTR SS:\[EBP-A0\]  
004B3A7A   .  E8 8DFCF4FF   CALL RemindBo.0040370C  
004B3A7F   .  8D45 90       LEA EAX,DWORD PTR SS:\[EBP-70\]  
004B3A82   .  8B55 D4       MOV EDX,DWORD PTR SS:\[EBP-2C\]  
004B3A85   .  8A52 07       MOV DL,BYTE PTR DS:\[EDX+7\]               ;  T\[7\]  
004B3A88   .  8850 01       MOV BYTE PTR DS:\[EAX+1\],DL  
004B3A8B   .  C600 01       MOV BYTE PTR DS:\[EAX\],1  
004B3A8E   .  8D55 90       LEA EDX,DWORD PTR SS:\[EBP-70\]  
004B3A91   .  8D85 60FFFFFF LEA EAX,DWORD PTR SS:\[EBP-A0\]  
004B3A97   .  B1 06         MOV CL,6  
004B3A99   .  E8 3EFCF4FF   CALL RemindBo.004036DC  
004B3A9E   .  8D95 60FFFFFF LEA EDX,DWORD PTR SS:\[EBP-A0\]  
004B3AA4   .  8D85 58FFFFFF LEA EAX,DWORD PTR SS:\[EBP-A8\]  
004B3AAA   .  E8 5DFCF4FF   CALL RemindBo.0040370C  
004B3AAF   .  8D45 90       LEA EAX,DWORD PTR SS:\[EBP-70\]  
004B3AB2   .  8B55 D8       MOV EDX,DWORD PTR SS:\[EBP-28\]  
004B3AB5   .  8A52 07       MOV DL,BYTE PTR DS:\[EDX+7\]               ;  S\[7\]  
004B3AB8   .  8850 01       MOV BYTE PTR DS:\[EAX+1\],DL  
004B3ABB   .  C600 01       MOV BYTE PTR DS:\[EAX\],1  
004B3ABE   .  8D55 90       LEA EDX,DWORD PTR SS:\[EBP-70\]  
004B3AC1   .  8D85 58FFFFFF LEA EAX,DWORD PTR SS:\[EBP-A8\]  
004B3AC7   .  B1 07         MOV CL,7  
004B3AC9   .  E8 0EFCF4FF   CALL RemindBo.004036DC  
004B3ACE   .  8D95 58FFFFFF LEA EDX,DWORD PTR SS:\[EBP-A8\]  
004B3AD4   .  8D85 4CFFFFFF LEA EAX,DWORD PTR SS:\[EBP-B4\]  
004B3ADA   .  E8 2DFCF4FF   CALL RemindBo.0040370C  
004B3ADF   .  8D45 90       LEA EAX,DWORD PTR SS:\[EBP-70\]  
004B3AE2   .  8B55 D8       MOV EDX,DWORD PTR SS:\[EBP-28\]  
004B3AE5   .  8A52 20       MOV DL,BYTE PTR DS:\[EDX+20\]              ;  S\[20\]  
004B3AE8   .  8850 01       MOV BYTE PTR DS:\[EAX+1\],DL  
004B3AEB   .  C600 01       MOV BYTE PTR DS:\[EAX\],1  
004B3AEE   .  8D55 90       LEA EDX,DWORD PTR SS:\[EBP-70\]  
004B3AF1   .  8D85 4CFFFFFF LEA EAX,DWORD PTR SS:\[EBP-B4\]  
004B3AF7   .  B1 08         MOV CL,8  
004B3AF9   .  E8 DEFBF4FF   CALL RemindBo.004036DC  
004B3AFE   .  8D95 4CFFFFFF LEA EDX,DWORD PTR SS:\[EBP-B4\]  
004B3B04   .  8D85 40FFFFFF LEA EAX,DWORD PTR SS:\[EBP-C0\]  
004B3B0A   .  E8 FDFBF4FF   CALL RemindBo.0040370C  
004B3B0F   .  8D45 90       LEA EAX,DWORD PTR SS:\[EBP-70\]  
004B3B12   .  8B55 D8       MOV EDX,DWORD PTR SS:\[EBP-28\]  
004B3B15   .  8A52 0E       MOV DL,BYTE PTR DS:\[EDX+E\]               ; S\[0E\]  
004B3B18   .  8850 01       MOV BYTE PTR DS:\[EAX+1\],DL  
004B3B1B   .  C600 01       MOV BYTE PTR DS:\[EAX\],1  
004B3B1E   .  8D55 90       LEA EDX,DWORD PTR SS:\[EBP-70\]  
004B3B21   .  8D85 40FFFFFF LEA EAX,DWORD PTR SS:\[EBP-C0\]  
004B3B27   .  B1 09         MOV CL,9  
004B3B29   .  E8 AEFBF4FF   CALL RemindBo.004036DC  
004B3B2E   .  8D95 40FFFFFF LEA EDX,DWORD PTR SS:\[EBP-C0\]  
004B3B34   .  8D85 34FFFFFF LEA EAX,DWORD PTR SS:\[EBP-CC\]  
004B3B3A   .  E8 CDFBF4FF   CALL RemindBo.0040370C  
004B3B3F   .  8D45 90       LEA EAX,DWORD PTR SS:\[EBP-70\]  
004B3B42   .  8B55 D4       MOV EDX,DWORD PTR SS:\[EBP-2C\]  
004B3B45   .  8A52 20       MOV DL,BYTE PTR DS:\[EDX+20\]              ;  T\[20\]  
004B3B48   .  8850 01       MOV BYTE PTR DS:\[EAX+1\],DL  
004B3B4B   .  C600 01       MOV BYTE PTR DS:\[EAX\],1  
004B3B4E   .  8D55 90       LEA EDX,DWORD PTR SS:\[EBP-70\]  
004B3B51   .  8D85 34FFFFFF LEA EAX,DWORD PTR SS:\[EBP-CC\]  
004B3B57   .  B1 0A         MOV CL,0A  
004B3B59   .  E8 7EFBF4FF   CALL RemindBo.004036DC  
004B3B5E   .  8D95 34FFFFFF LEA EDX,DWORD PTR SS:\[EBP-CC\]  
004B3B64   .  8D85 28FFFFFF LEA EAX,DWORD PTR SS:\[EBP-D8\]  
004B3B6A   .  E8 9DFBF4FF   CALL RemindBo.0040370C  
004B3B6F   .  8D45 90       LEA EAX,DWORD PTR SS:\[EBP-70\]  
004B3B72   .  8B55 D4       MOV EDX,DWORD PTR SS:\[EBP-2C\]  
004B3B75   .  8A52 10       MOV DL,BYTE PTR DS:\[EDX+10\]              ;  T\[10\]  
004B3B78   .  8850 01       MOV BYTE PTR DS:\[EAX+1\],DL  
004B3B7B   .  C600 01       MOV BYTE PTR DS:\[EAX\],1  
004B3B7E   .  8D55 90       LEA EDX,DWORD PTR SS:\[EBP-70\]  
004B3B81   .  8D85 28FFFFFF LEA EAX,DWORD PTR SS:\[EBP-D8\]  
004B3B87   .  B1 0B         MOV CL,0B  
004B3B89   .  E8 4EFBF4FF   CALL RemindBo.004036DC  
004B3B8E   .  8D95 28FFFFFF LEA EDX,DWORD PTR SS:\[EBP-D8\]            ;  "4BDFAD3C1BC"  
004B3B94   .  8D45 D0       LEA EAX,DWORD PTR SS:\[EBP-30\]  
004B3B97   .  E8 4C1AF5FF   CALL RemindBo.004055E8  
004B3B9C   .  8B0D 80CE4A00 MOV ECX,DWORD PTR DS:\[4ACE80\]            ;  RemindBo.004ACECC  
004B3BA2   .  8B55 D0       MOV EDX,DWORD PTR SS:\[EBP-30\]  
004B3BA5   .  8B45 E8       MOV EAX,DWORD PTR SS:\[EBP-18\]            ;  xxx\_context?  
004B3BA8   .  E8 4F8BFFFF   CALL RemindBo.004AC6FC                   ;  ??  
004B3BAD   .  8B45 E8       MOV EAX,DWORD PTR SS:\[EBP-18\]  
004B3BB0   .  8B10          MOV EDX,DWORD PTR DS:\[EAX\]               ;  RemindBo.004AF580  
004B3BB2   .  FF52 48       CALL DWORD PTR DS:\[EDX+48\]  
004B3BB5   .  8D85 24FFFFFF LEA EAX,DWORD PTR SS:\[EBP-DC\]  
004B3BBB   .  50            PUSH EAX  
004B3BBC   .  8B45 F4       MOV EAX,DWORD PTR SS:\[EBP-C\]             ;  key  
004B3BBF   .  E8 9C1AF5FF   CALL RemindBo.00405660  
004B3BC4   .  8BC8          MOV ECX,EAX  
004B3BC6   .  83E9 02       SUB ECX,2  
004B3BC9   .  BA 02000000   MOV EDX,2  
004B3BCE   .  8B45 F4       MOV EAX,DWORD PTR SS:\[EBP-C\]  
004B3BD1   .  E8 EA1CF5FF   CALL RemindBo.004058C0                   ; 切掉 Key 的第 1 位和最后 1 位  
004B3BD6   .  8B95 24FFFFFF MOV EDX,DWORD PTR SS:\[EBP-DC\]  
004B3BDC   .  8D4D D0       LEA ECX,DWORD PTR SS:\[EBP-30\]  
004B3BDF   .  8B45 E8       MOV EAX,DWORD PTR SS:\[EBP-18\]  
004B3BE2   .  8B18          MOV EBX,DWORD PTR DS:\[EAX\]  
004B3BE4   .  FF53 58       CALL DWORD PTR DS:\[EBX+58\]               ;  关键代码, F7  
004B3BE7   .  8B45 D4       MOV EAX,DWORD PTR SS:\[EBP-2C\]            ;  T\[0...2F\]  
004B3BEA   .  8B55 D0       MOV EDX,DWORD PTR SS:\[EBP-30\]            ;  计算结果  
004B3BED   .  E8 BA1BF5FF   CALL RemindBo.004057AC                   ;  比较  
004B3BF2   .  75 04         JNZ SHORT RemindBo.004B3BF8              ;  不能跳  
004B3BF4   .  C645 F3 01    MOV BYTE PTR SS:\[EBP-D\],1  
004B3BF8   >  33C0          XOR EAX,EAX  
004B3BFA   .  5A            POP EDX  
  
  
004ACBA4  /.  55            PUSH EBP  
004ACBA5  |.  8BEC          MOV EBP,ESP  
004ACBA7  |.  83C4 F4       ADD ESP,-0C  
004ACBAA  |.  53            PUSH EBX  
004ACBAB  |.  894D F4       MOV DWORD PTR SS:\[EBP-C\],ECX  
004ACBAE  |.  8955 F8       MOV DWORD PTR SS:\[EBP-8\],EDX  
004ACBB1  |.  8945 FC       MOV DWORD PTR SS:\[EBP-4\],EAX  
004ACBB4  |.  8B55 F4       MOV EDX,DWORD PTR SS:\[EBP-C\]  
004ACBB7  |.  8B45 F8       MOV EAX,DWORD PTR SS:\[EBP-8\]  
004ACBBA  |.  E8 D1F2FFFF   CALL RemindBo.004ABE90                   ;  DecodeBase64(Key)  
004ACBBF  |.  8B45 F4       MOV EAX,DWORD PTR SS:\[EBP-C\]  
004ACBC2  |.  8B00          MOV EAX,DWORD PTR DS:\[EAX\]  
004ACBC4  |.  E8 978AF5FF   CALL RemindBo.00405660  
004ACBC9  |.  50            PUSH EAX  
004ACBCA  |.  8B45 F4       MOV EAX,DWORD PTR SS:\[EBP-C\]  
004ACBCD  |.  E8 E68CF5FF   CALL RemindBo.004058B8  
004ACBD2  |.  50            PUSH EAX  
004ACBD3  |.  8B45 F4       MOV EAX,DWORD PTR SS:\[EBP-C\]  
004ACBD6  |.  E8 DD8CF5FF   CALL RemindBo.004058B8  
004ACBDB  |.  8BD0          MOV EDX,EAX  
004ACBDD  |.  8B45 FC       MOV EAX,DWORD PTR SS:\[EBP-4\]  
004ACBE0  |.  59            POP ECX  
004ACBE1  |.  8B18          MOV EBX,DWORD PTR DS:\[EAX\]  
004ACBE3  |.  FF93 80000000 CALL DWORD PTR DS:\[EBX+80\]               ;  RemindBo.004AEDAC, F7 跟进 !!!  
004ACBE9  |.  5B            POP EBX  
004ACBEA  |.  8BE5          MOV ESP,EBP  
004ACBEC  |.  5D            POP EBP  
004ACBED  \\.  C3            RETN  
  
  
004AEDAC  /.  55            PUSH EBP  
004AEDAD  |.  8BEC          MOV EBP,ESP  
004AEDAF  |.  83C4 D8       ADD ESP,-28  
004AEDB2  |.  53            PUSH EBX  
004AEDB3  |.  894D F4       MOV DWORD PTR SS:\[EBP-C\],ECX  
004AEDB6  |.  8955 F8       MOV DWORD PTR SS:\[EBP-8\],EDX  
004AEDB9  |.  8945 FC       MOV DWORD PTR SS:\[EBP-4\],EAX             ; Blowfish\_context  
004AEDBC  |.  8B45 FC       MOV EAX,DWORD PTR SS:\[EBP-4\]  
004AEDBF  |.  8078 30 00    CMP BYTE PTR DS:\[EAX+30\],0  
004AEDC3  |.  75 16         JNZ SHORT RemindBo.004AEDDB  
004AEDC5  |.  B9 5CEE4A00   MOV ECX,RemindBo.004AEE5C                ;  ASCII "Cipher not initialized"  
004AEDCA  |.  B2 01         MOV DL,1  
004AEDCC  |.  A1 9CC24A00   MOV EAX,DWORD PTR DS:\[4AC29C\]  
004AEDD1  |.  E8 9E00F6FF   CALL RemindBo.0040EE74  
004AEDD6  |.  E8 FD5EF5FF   CALL RemindBo.00404CD8  
004AEDDB  |>  8B45 F8       MOV EAX,DWORD PTR SS:\[EBP-8\]  
004AEDDE  |.  8945 EC       MOV DWORD PTR SS:\[EBP-14\],EAX  
004AEDE1  |.  8B45 F4       MOV EAX,DWORD PTR SS:\[EBP-C\]  
004AEDE4  |.  8945 E8       MOV DWORD PTR SS:\[EBP-18\],EAX  
004AEDE7  |.  8B45 08       MOV EAX,DWORD PTR SS:\[EBP+8\]  
004AEDEA  |.  85C0          TEST EAX,EAX  
004AEDEC  |.  76 5D         JBE SHORT RemindBo.004AEE4B  
004AEDEE  |.  8945 D8       MOV DWORD PTR SS:\[EBP-28\],EAX  
004AEDF1  |.  C745 F0 01000>MOV DWORD PTR SS:\[EBP-10\],1  
004AEDF8  |>  8B45 EC       /MOV EAX,DWORD PTR SS:\[EBP-14\]           ; pointer to Key\_bin  
004AEDFB  |.  8A00          |MOV AL,BYTE PTR DS:\[EAX\]  
004AEDFD  |.  8845 E7       |MOV BYTE PTR SS:\[EBP-19\],AL  
004AEE00  |.  8D4D DF       |LEA ECX,DWORD PTR SS:\[EBP-21\]           ; 指向输出的 xl, xr  
004AEE03  |.  8B45 FC       |MOV EAX,DWORD PTR SS:\[EBP-4\]            ; Blowfish\_context  
004AEE06  |.  8D50 40       |LEA EDX,DWORD PTR DS:\[EAX+40\]           ; 指向输入的 xl, xr  
004AEE09  |.  8B45 FC       |MOV EAX,DWORD PTR SS:\[EBP-4\]  
004AEE0C  |.  8B18          |MOV EBX,DWORD PTR DS:\[EAX\]  
004AEE0E  |.  FF53 6C       |CALL DWORD PTR DS:\[EBX+6C\]              ; Blowfish\_Encrypt()  
004AEE11  |.  8B45 EC       |MOV EAX,DWORD PTR SS:\[EBP-14\]           ;   
004AEE14  |.  8A00          |MOV AL,BYTE PTR DS:\[EAX\]                ; Key\_bin\[i\]  
004AEE16  |.  3245 DF       |XOR AL,BYTE PTR SS:\[EBP-21\]             ; XOR xl  
004AEE19  |.  8B55 E8       |MOV EDX,DWORD PTR SS:\[EBP-18\]  
004AEE1C  |.  8802          |MOV BYTE PTR DS:\[EDX\],AL                ; 保存结果  
004AEE1E  |.  8B45 FC       |MOV EAX,DWORD PTR SS:\[EBP-4\]  
004AEE21  |.  8D50 40       |LEA EDX,DWORD PTR DS:\[EAX+40\]  
004AEE24  |.  8B45 FC       |MOV EAX,DWORD PTR SS:\[EBP-4\]  
004AEE27  |.  83C0 41       |ADD EAX,41                              ; xl,xr 左移一 BYTE  
004AEE2A  |.  B9 07000000   |MOV ECX,7                               ; 即只取 7 BYTES   
004AEE2F  |.  E8 7840F5FF   |CALL RemindBo.00402EAC  
004AEE34  |.  8A45 E7       |MOV AL,BYTE PTR SS:\[EBP-19\]  
004AEE37  |.  8B55 FC       |MOV EDX,DWORD PTR SS:\[EBP-4\]  
004AEE3A  |.  8842 47       |MOV BYTE PTR DS:\[EDX+47\],AL             ; 加上 Key\_bin\[i\], 重新构成 8BYTES 的 xl, xr  
004AEE3D  |.  FF45 EC       |INC DWORD PTR SS:\[EBP-14\]  
004AEE40  |.  FF45 E8       |INC DWORD PTR SS:\[EBP-18\]  
004AEE43  |.  FF45 F0       |INC DWORD PTR SS:\[EBP-10\]  
004AEE46  |.  FF4D D8       |DEC DWORD PTR SS:\[EBP-28\]  
004AEE49  |.^ 75 AD         \\JNZ SHORT RemindBo.004AEDF8             ; 循环 0x30 遍  
004AEE4B  |>  5B            POP EBX  
004AEE4C  |.  8BE5          MOV ESP,EBP  
004AEE4E  |.  5D            POP EBP  
004AEE4F  \\.  C2 0400       RETN 4  
  
  
<3> 注册算法:  
  
1. KeyLicense 长 0x47, 以 RMDBK 开始  
2. 去掉 KeyLicense 开始的 RMDBK, 长 0x42  
3. UserName + 常串 "3.0" ==> buf1  
4. SHA1(buf1) ==> SHA1\_hash: 0x14 bytes HEX, 简称为 hash1  
5. SHA1\_hash 转换成 0x28 bytes 的 HEX 串, H1  
6. H1 的第 11 位==KeyLicense 的最后一位.  
7. 取 HEX 串的 08, 1D, 11, 02 BYTEs, 加常串 "Remind Book"   
8. 加 HEX 串的 26, 08 BYTES, ==> buf2  
9. Tiger(buf2) ==> Tiger\_hash: 0x18 bytes 的 HEX 串, 简称为 hash2  
10. Tiger\_hash 转换成 0x30 bytes 的 HEX 串, H2  
11. H2 的第 23 位==KeyLicense 的第一位.  
10. 取 H1\[6\],H2\[2\],H1\[1A\],H2\[1D\],H1\[10\],H2\[07\],H1\[07\],H1\[20\],H1\[0E\],H2\[20\],H2\[10\]  
    构成 buf3  
11. SHA1(buf3) ==> SHA1\_hash, 称为 hash3  
12. 用 hash3 作为 blowfish 的 init\_key, 初始化 Blowfish CTX  
13. 去掉 KeyLicense 的第一位和最后一位, 长 0x40  
14. DecodeBase64(KeyLicense) ==> Key\_bin, 长 0x30  
15. L=R=0, Blowfish\_Encrypt(&blowfish, &L, &R);  
  
16. 保存 L,R, CALL Blowfish\_Encrypt(&blowfish, &L, &R);  
17. 取 L 的第一 BYTE XOR Key\_bin\[i\] ==> Result\[i\];  
18. 取保存的 L,R,舍掉 L 的第一 BYTE, 右边补上 Key\_in\[i\], 即左移一 BYTE;  
19. 到 16 循环, 共 运行 0x30 次,   
20. 最后得到的 Result 为 0x30 BYTES, 应与 H2 一样.  
  
  
<4> 注册机:　只有注册算法的代码,   
base64, SHA1, Tiger, blowfish 这些加密算法的代码请到网上找吧.  
  
\=========================================================  
#include <stdio.h>  
#include <string.h>  
  
#include "base64.h"  
#include "sha1.h"  
#include "tiger.h"  
#include "blowfish.h"  
  
char str1\[\]="3.0";  
char str2\[\]="Remind Book";  
  
char Username\[\]="blackeyes@abc.com";  
char Key\[100\] = "RMDBK4MDEyMzQ1Njc4OUFCQ0RFRjAxMjM0NTY3ODlBQkNERUYwMTIzNDU2Nzg5QUJDREVG0";  
char Key\_base64\[100\];  
unsigned char Key\_bin\[100\];  
  
SHA1Context sha;  
uint8\_t     sha1\_hash1\[20\];  
char        sha1\_hash\_str1\[100\];  
uint8\_t     sha1\_hash2\[20\];  
char        sha1\_hash\_str2\[100\];  
  
TIGER\_CTX  tiger;  
uint8\_t    tiger\_hash\[24\];  
char       tiger\_hash\_str\[100\];  
  
BLOWFISH\_CTX blowfish;  
  
int main(void )  
{  
  int  i;  
  int  len;  
  char buf1\[100\], buf2\[100\], buf3\[100\], buf4\[100\];  
  int  seq1\[\]={0x08, 0x1D, 0x11, 0x02};  
  int  seq2\[\]={0x26, 0x8};  
  int  seq3\[\]={0x0006, 0x8002, 0x001A, 0x801D, 0x0010, 0x8007,  
    0x0007,0x0020,0x000E,0x8020, 0x8010};  
  
  unsigned long L, R, L0, R0;  
  
  printf("Input name:");  
  gets(Username);  
    
  strcpy(buf1, Username);  
  strcat(buf1, str1);  
    
  SHA1Reset(&sha);  
  SHA1Input(&sha, (const unsigned char \*) buf1, strlen(buf1));  
  SHA1Result(&sha, sha1\_hash1);  
  
  for(i = 0; i < 20 ; ++i)  
  {  
    sprintf(sha1\_hash\_str1+i\*2, "%02X ", sha1\_hash1\[i\]);  
  }  
  sha1\_hash\_str1\[i+20\] = '\\0';  
  
  printf("SHA1 input: %s\\n", buf1);  
  printf("SHA1 Hash:  %s\\n", sha1\_hash\_str1);  
  
  for(i=0;i<4;i++)  
    buf2\[i\] = sha1\_hash\_str1\[seq1\[i\]\];  
  buf2\[i\] = '\\0';  
  strcat(buf2, str2);  
  len=strlen(buf2);  
  for(i=0;i<2;i++)  
    buf2\[len+i\]=sha1\_hash\_str1\[seq2\[i\]\];  
  buf2\[len+i\] = '\\0';  
  
  Tiger\_init(&tiger);  
  Tiger\_Hash(&tiger, tiger\_hash, buf2, strlen(buf2));  
  
  for(i = 0; i < 24 ; ++i)  
  {  
    sprintf(tiger\_hash\_str+i\*2, "%02X ", tiger\_hash\[i\]);  
  }  
  tiger\_hash\_str\[i\*2\] = '\\0';  
  printf("Tiger input: %s\\n", buf2);  
  printf("Tiger Hash:  %s\\n", tiger\_hash\_str);  
  
  for(i=0;i<11;i++)  
    buf3\[i\] = (seq3\[i\] & 0x8000) ? tiger\_hash\_str\[seq3\[i\]&0xff\] : sha1\_hash\_str1\[seq3\[i\]&0xff\] ;  
  buf3\[i\]='\\0';  
  
  SHA1Reset(&sha);  
  SHA1Input(&sha, (const unsigned char \*) buf3, strlen(buf3));  
  SHA1Result(&sha, sha1\_hash2);  
  for(i = 0; i < 20 ; ++i)  
  {  
    sprintf(sha1\_hash\_str2+i\*2, "%02X ", sha1\_hash2\[i\]);  
  }  
  sha1\_hash\_str2\[i+20\] = '\\0';  
  printf("SHA1 input: %s\\n", buf3);  
  printf("SHA1 hash:  %s\\n", sha1\_hash\_str2);  
    
  Blowfish\_Init(&blowfish, sha1\_hash2, 0x14);  
  
  L = 0;  
  R = 0;  
  Blowfish\_Encrypt(&blowfish, &L, &R);  
  
    
/\*  验证代码 \*/  
/\*  
  strcpy(Key\_base64, Key+6);  
  Key\_base64\[0x40\] = '\\0';  
  len = DecodeBase64(Key\_base64, Key\_bin, 100);  
  
  for(i=0;i<0x30;i++)  
  {  
    L0 = L;  
    R0 = R;  
    Blowfish\_Encrypt(&blowfish, &L, &R);  
    buf4\[i\] = Key\_bin\[i\] ^ (unsigned char)(L>>24);  
    L = L0;  
    R = R0;  
    L = (L << 8) | (R>>24);  
    R = (R << 8) | ((unsigned long)Key\_bin\[i\]);  
  }  
  buf4\[i\] = '\\0';  
  if (strcmp(buf4, tiger\_hash\_str)==0){  
    printf("OK\\n");  
  }  
\*/  
  
/\* 反算求 KeyLicense \*/  
  for(i=0;i<0x30;i++)  
  {  
    L0 = L;  
    R0 = R;  
    Blowfish\_Encrypt(&blowfish, &L, &R);  
    Key\_bin\[i\] = tiger\_hash\_str\[i\] ^ (unsigned char)(L>>24);  
    L = L0;  
    R = R0;  
    L = (L << 8) | (R>>24);  
    R = (R << 8) | ((unsigned long)Key\_bin\[i\]);  
  }  
  Key\_bin\[i\] = '\\0';  
  len = EncodeBase64(Key\_bin, 0x30, Key\_base64, 100);  
  strcpy(Key, "RMDBK");  
  Key\[5\] = tiger\_hash\_str\[0x23\];  
  Key\[6\] = '\\0';  
  strcat(Key, Key\_base64);  
  len = strlen(Key);  
  Key\[len\] = sha1\_hash\_str1\[0x11\];  
  Key\[len+1\] = '\\0';  
  printf("User:%s\\n", Username);  
  printf("Key: %s\\n", Key);  
  
  return 0;  
} 

 4dd466d0e1e0cc123afbe504da2462ca