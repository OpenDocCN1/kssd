
# 《电视剧下载》V2.10算法分析

- 【破解作者】 gxjxdqk[DFCG][D.4s][CZG][OCN]POJE
- 【作者邮箱】 gxjxdqk
- 【作者主页】 gxjxdqk
- 【使用工具】 OD、PEID
- 【破解平台】 Win9x/NT/2000/XP
- 【软件名称】 电视剧下载 V2.10
- 【下载地址】 `a84K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8Y4k6$3N6Y4k6$3i4K6u0W2j5$3&6Q4x3V1k6\*7K9r3g2F1j5%4g2A6i4K6u0r3k6s2y4B7i4K6u0W2K9s2c8E0i4K6t1K6`
- 【软件简介】 《电视剧下载》:独一无二的电视剧高速下载和在线收看软件，汇集了400多个国内外电视台,1000多部|港台现代剧 | 港台古装剧 | 大陆电视剧 | 青春偶像剧 | 粤语原版剧 | 日韩电视剧 | 卡通漫画剧 | 欧美原版剧| ,速度快质量高的精彩视频，专人实时跟踪更新升级,本软件具有自动升级功能。 通过独特的合作资源，提供超过400套超高清晰的卫星电视节目，高速HBO大片，BBC新闻，日本MM、港台凤凰、华娱、星空、TVB系列，国内中央及各省卫星电视节目；
- 【加壳方式】 无
- 【破解声明】 我是一只小菜鸟，偶得一点心得，愿与大家分享：）

---

## 【破解内容】

PEID查看，无壳DELPHI编写运行试试，发现有提示信息好OD载入

查找字串“您输入的注册码无效，请重新输入”双击

向上一点下断运行程序被断下分析开始：）

首先我的机器码为`B8E02418`   试用的注册码为`76543210`

```
0049E29E   .  E8 4D7EFAFF      CALL 电视剧下.004460F0
0049E2A3   .  837D F0 00       CMP DWORD PTR SS:[EBP-10],0         ;  假码跟0比
0049E2A7   .  74 1E            JE SHORT 电视剧下.0049E2C7              ;  等于0就跳
0049E2A9   .  8D55 EC          LEA EDX,DWORD PTR SS:[EBP-14]
0049E2AC   .  8B45 FC          MOV EAX,DWORD PTR SS:[EBP-4]
0049E2AF   .  8B80 3C030000    MOV EAX,DWORD PTR DS:[EAX+33C]
0049E2B5   .  E8 367EFAFF      CALL 电视剧下.004460F0
0049E2BA   .  8B45 EC          MOV EAX,DWORD PTR SS:[EBP-14]       ;  假码入EAX
0049E2BD   .  E8 8E61F6FF      CALL 电视剧下.00404450                  ;  计算假码位数的CALL
0049E2C2   .  83F8 08          CMP EAX,8                           ;  假码位数比8
0049E2C5   .  7E 0F            JLE SHORT 电视剧下.0049E2D6             ;  注册码必需小于等于8位
0049E2C7   >  B8 30E54900      MOV EAX,电视剧下.0049E530
0049E2CC   .  E8 7716F9FF      CALL 电视剧下.0042F948                  ;  出错框调用的CALL
0049E2D1   .  E9 BA010000      JMP 电视剧下.0049E490
0049E2D6   >  8D45 E4          LEA EAX,DWORD PTR SS:[EBP-1C]
0049E2D9   .  50               PUSH EAX
0049E2DA   .  8D55 E0          LEA EDX,DWORD PTR SS:[EBP-20]
0049E2DD   .  8B45 FC          MOV EAX,DWORD PTR SS:[EBP-4]
0049E2E0   .  8B80 40030000    MOV EAX,DWORD PTR DS:[EAX+340]
0049E2E6   .  E8 057EFAFF      CALL 电视剧下.004460F0
0049E2EB   .  8B45 E0          MOV EAX,DWORD PTR SS:[EBP-20]       ;  机器码入EAX
0049E2EE   .  B9 06000000      MOV ECX,6
0049E2F3   .  BA 01000000      MOV EDX,1
0049E2F8   .  E8 B363F6FF      CALL 电视剧下.004046B0
0049E2FD   .  8B4D E4          MOV ECX,DWORD PTR SS:[EBP-1C]       ;  机器码前6位给ECX
0049E300   .  8D45 E8          LEA EAX,DWORD PTR SS:[EBP-18]
0049E303   .  BA 5CE54900      MOV EDX,电视剧下.0049E55C               ;  ASCII "0x"
0049E308   .  E8 8F61F6FF      CALL 电视剧下.0040449C                  ;  在前六位机器码前加0x的CALL
0049E30D   .  8B45 E8          MOV EAX,DWORD PTR SS:[EBP-18]       ;  0xB8E024入EAX
0049E310   .  E8 8FA5F6FF      CALL 电视剧下.004088A4                   ;  把机器码前六位的前面加00的CALL
0049E315   .  8BF0             MOV ESI,EAX                          ;  00B8E024给ESI
0049E317   .  33C0             XOR EAX,EAX                          ;  EAX清0
0049E319   .  55               PUSH EBP
0049E31A   .  68 47E44900      PUSH 电视剧下.0049E447
0049E31F   .  64:FF30          PUSH DWORD PTR FS:[EAX]
0049E322   .  64:8920          MOV DWORD PTR FS:[EAX],ESP
0049E325   .  8D55 DC          LEA EDX,DWORD PTR SS:[EBP-24]
0049E328   .  8B45 FC          MOV EAX,DWORD PTR SS:[EBP-4]
0049E32B   .  8B80 3C030000    MOV EAX,DWORD PTR DS:[EAX+33C]
0049E331   .  E8 BA7DFAFF      CALL 电视剧下.004460F0
0049E336   .  8B45 DC          MOV EAX,DWORD PTR SS:[EBP-24]     ;  假码给EAX
0049E339   .  E8 66A5F6FF      CALL 电视剧下.004088A4                ;  算法CALL跟进
0049E33E   .  8BD8             MOV EBX,EAX                       ;  假码算出的值EAX给EBX
0049E340   .  8BC3             MOV EAX,EBX                       ;  假码算出的值EBX给EAX
0049E342   .  2BC6             SUB EAX,ESI                       ;  EAX-ESI
0049E344   .  3B05 70A14C00    CMP EAX,DWORD PTR DS:[4CA170]     ;  把EAX-ESI得到的值称如果=00023F3A(147258)
0049E34A   .  74 38            JE SHORT 电视剧下.0049E384            ;  等于就跳向正确
```

跟进`0049E339`的`CALL`

```
004088A4  /$  53               PUSH EBX
004088A5  |.  56               PUSH ESI
004088A6  |.  83C4 F4          ADD ESP,-0C                             ;  ESP指向假码地址
004088A9  |.  8BD8             MOV EBX,EAX
004088AB  |.  8BD4             MOV EDX,ESP
004088AD  |.  8BC3             MOV EAX,EBX
004088AF  |.  E8 20A5FFFF      CALL 电视剧下.00402DD4
004088B4  |.  8BF0             MOV ESI,EAX                             ;  假码算出的值给ESI
004088B6  |.  833C24 00        CMP DWORD PTR SS:[ESP],0
004088BA  |.  74 19            JE SHORT 电视剧下.004088D5
004088BC  |.  895C24 04        MOV DWORD PTR SS:[ESP+4],EBX
004088C0  |.  C64424 08 0B     MOV BYTE PTR SS:[ESP+8],0B
004088C5  |.  8D5424 04        LEA EDX,DWORD PTR SS:[ESP+4]
004088C9  |.  A1 30A44C00      MOV EAX,DWORD PTR DS:[4CA430]
004088CE  |.  33C9             XOR ECX,ECX
004088D0  |.  E8 33F9FFFF      CALL 电视剧下.00408208
004088D5  |>  8BC6             MOV EAX,ESI                             ;  假码算出的值给EAX
004088D7  |.  83C4 0C          ADD ESP,0C                              ;  ESP+0C指向假码算出的值
004088DA  |.  5E               POP ESI                                 ;  00B8E024出堆栈
004088DB  |.  5B               POP EBX
004088DC  \.  C3               RETN
```

算法`CALL`跟进`004088AF`

```
00402DD4  /$  53               PUSH EBX
00402DD5  |.  56               PUSH ESI
00402DD6  |.  57               PUSH EDI
00402DD7  |.  89C6             MOV ESI,EAX
00402DD9  |.  50               PUSH EAX
00402DDA  |.  85C0             TEST EAX,EAX
00402DDC  |.  74 6C            JE SHORT 电视剧下.00402E4A
00402DDE  |.  31C0             XOR EAX,EAX
00402DE0  |.  31DB             XOR EBX,EBX
00402DE2  |.  BF CCCCCC0C      MOV EDI,0CCCCCCC                     ;  0CCCCCCC给EDI
00402DE7  |>  8A1E             /MOV BL,BYTE PTR DS:[ESI]            ;  取一位假码入BL
00402DE9  |.  46               |INC ESI                             ;  ESI加1
00402DEA  |.  80FB 20          |CMP BL,20                           ;  跟20比
00402DED  |.^ 74 F8            \JE SHORT 电视剧下.00402DE7              ;  等于20（空）就跳
00402DEF  |.  B5 00            MOV CH,0
00402DF1  |.  80FB 2D          CMP BL,2D                            ;  跟2D（-）比
00402DF4  |.  74 62            JE SHORT 电视剧下.00402E58
00402DF6  |.  80FB 2B          CMP BL,2B                            ;  跟2B（+）比
00402DF9  |.  74 5F            JE SHORT 电视剧下.00402E5A
00402DFB  |>  80FB 24          CMP BL,24                            ;  跟24（$）比
00402DFE  |.  74 5F            JE SHORT 电视剧下.00402E5F
00402E00  |.  80FB 78          CMP BL,78                            ;  跟78（x）比
00402E03  |.  74 5A            JE SHORT 电视剧下.00402E5F
00402E05  |.  80FB 58          CMP BL,58                            ;  跟58（X）比
00402E08  |.  74 55            JE SHORT 电视剧下.00402E5F
00402E0A  |.  80FB 30          CMP BL,30                            ;  跟30（0）比
00402E0D  |.  75 13            JNZ SHORT 电视剧下.00402E22              ;  不等于30（0）就跳
00402E0F  |.  8A1E             MOV BL,BYTE PTR DS:[ESI]
00402E11  |.  46               INC ESI
00402E12  |.  80FB 78          CMP BL,78
00402E15  |.  74 48            JE SHORT 电视剧下.00402E5F
00402E17  |.  80FB 58          CMP BL,58
00402E1A  |.  74 43            JE SHORT 电视剧下.00402E5F
00402E1C  |.  84DB             TEST BL,BL
00402E1E  |.  74 20            JE SHORT 电视剧下.00402E40
00402E20  |.  EB 04            JMP SHORT 电视剧下.00402E26
00402E22  |> \84DB             TEST BL,BL                           ;  测试BL是否为空
00402E24  |.  74 2D            JE SHORT 电视剧下.00402E53               ;  为0点
00402E26  |>  80EB 30          /SUB BL,30                           ;  假码的一位16进制-30
00402E29  |.  80FB 09          |CMP BL,9                            ;  跟9比
00402E2C  |.  77 25            |JA SHORT 电视剧下.00402E53              ;  大于9就跳
00402E2E  |.  39F8             |CMP EAX,EDI                         ;  EAX比0CCCCCCCC
00402E30  |.  77 21            |JA SHORT 电视剧下.00402E53              ;  大于0CCCCCCCC就跳
00402E32  |.  8D0480           |LEA EAX,DWORD PTR DS:[EAX+EAX\*4]    ;  取EAX\*5给EAX
```
