
# 计算机软件水平考试测试系统 3.0

> 原文：[`www.pediy.com/kssd/pediy06/pediy6003.htm`](https://www.pediy.com/kssd/pediy06/pediy6003.htm)

计算机软件水平考试测试系统注册算法分析

* 作者声明：初学破解，纯属技术交流，无其它目的。
* 工具：`ollyDBg1.09`，`W32Dasm10`，脱壳工具。
* 基本知识：基础汇编知识，基本工具使用。
* 注册形式：注册申请码 + 注册认证码
* 软件介绍：

软件名称：计算机软件水平考试测试系统

整理日期：2003.4.29

最新版本：3.0 精华版(高级程序员)

文件大小：14659KB

软件授权：共享软件

使用平台：`Win9x`/`Me`/`NT`/`2000`/`XP`

发布公司：`Home Page`

软件简介：

这是一个关于“计算机软件专业技术资格与水平考试”测试系统软件(高级程序员)3.0 精华版。该软件收集了 1991 年--2001 年高级程序员软件水平考试的所有上午试题，和其它书籍上的一些重点试题。用户可以选择随机生成试卷来进行测试，也可以选择某一类型的题或某一年的题来测试。随机生成试卷是按照软件水平考试大纲上的要求来选题的，所选试题类型完全服合考试大纲规定。该软件还自动计算用户测试时间，到交卷时间就会提醒用户交卷。未注册的用户，只能做 2000 年试题。要想使用软件的所有功能，请先注册。

* 破解过程：我喜欢全部共享，let's go

假设：

注册申请码：`76C2-B045-0C72-3C2F`

注册认证码：`1234-2345-3456-4567`

### 一、用脱壳工具脱壳后，在 `W32Dasm` 中找出错信息：

```
* Referenced by a (U)nconditional or (C)onditional Jump at Address:

|:006168E0(C)

|

:0061699D 6A00                  push 00000000

* Possible StringData Ref from Code Obj ->"警告框"

                              |

:0061699F B9186B6100           mov ecx, 00616B18

* Possible StringData Ref from Code Obj ->"注册码错误，请与作者联系!"

                              |

:006169A4 BA206B6100           mov edx, 00616B20

:006168BE 83FA10               cmp edx, 00000010
:006168C1 0F85F0000000         jne 006169B7===
:006168C7 8D4DF0               lea ecx, dword ptr [ebp-10]
:006168CA 8B55FC               mov edx, dword ptr [ebp-04]
:006168CD 8B45F4               mov eax, dword ptr [ebp-0C]
:006168D0 E8E7ECFFFF          call 006155BC
:006168D5 8B45F0               mov eax, dword ptr [ebp-10]
:006168D8 8B55F8               mov edx, dword ptr [ebp-08]
:006168DB E8CCE1DEFF          call 00404AAC
:006168E0 0F85B7000000         jne 0061699D========>出错啦
:006168E6 8D55BC               lea edx, dword ptr [ebp-44]
```

### 二、用 `ollydbg` 载入程序，找到下面：

```
00616791  |.  8BD8             MOV EBX,EAX
00616793  |.  33C0             XOR EAX,EAX
00616795  |.  55               PUSH EBP
00616796  |.  68 2F6A6100      PUSH unpacked.00616A2F
0061679B  |.  64:FF30          PUSH DWORD PTR FS:[EAX]
0061679E  |.  64:8920         MOV DWORD PTR FS:[EAX],ESP
006167A1  |.  8D45 FC         LEA EAX,DWORD PTR SS:[EBP-4]
006167A4  |.  50               PUSH EAX
006167A5  |.  A1 9C0C6200      MOV EAX,DWORD PTR DS:[620C9C]          ; 注册申请码入 eax
006167AA  |.  8B00             MOV EAX,DWORD PTR DS:[EAX]             ; eax 为 76C2B0450C723C2F
006167AC  |.  B9 08000000      MOV ECX,8
006167B1  |.  BA 05000000      MOV EDX,5
006167B6  |.  E8 05E4DEFF      CALL unpacked.00404BC0                 ; 从注册申请码第 5 位开始，共
                                                                      ; 取 8 位，即 B0450C72
006167BB  |.  8D45 E8          LEA EAX,DWORD PTR SS:[EBP-18]          ; B0450C72 入 eax
006167BE  |.  50               PUSH EAX
006167BF  |.  A1 9C0C6200      MOV EAX,DWORD PTR DS:[620C9C]
006167C4  |.  8B00             MOV EAX,DWORD PTR DS:[EAX]             ; 注册申请码入 eax
006167C6  |.  B9 04000000      MOV ECX,4
006167CB  |.  BA 0D000000      MOV EDX,0D
006167D0  |.  E8 EBE3DEFF      CALL unpacked.00404BC0                 ; 从注册申请码第 13 位开始，
                                                                      ; 共取 4 位，即 3C2F
006167D5  |.  8B45 E8          MOV EAX,DWORD PTR SS:[EBP-18]          ; 3C2F 入 eax
006167D8  |.  50               PUSH EAX
006167D9  |.  8D45 E4          LEA EAX,DWORD PTR SS:[EBP-1C]
006167DC  |.  50               PUSH EAX
006167DD  |.  A1 9C0C6200      MOV EAX,DWORD PTR DS:[620C9C]
006167E2  |.  8B00             MOV EAX,DWORD PTR DS:[EAX]             ; eax 为 76C2B0450C723C2F
006167E4  |.  B9 04000000      MOV ECX,4
006167E9  |.  BA 01000000      MOV EDX,1
006167EE  |.  E8 CDE3DEFF      CALL unpacked.00404BC0                 ; 从注册申请码第 1 位开始，共
                                                                      ; 取 4 位，即 76c2
006167F3  |.  8B55 E4          MOV EDX,DWORD PTR SS:[EBP-1C]          ; 76C2 入 edx
006167F6  |.  8D45 F8          LEA EAX,DWORD PTR SS:[EBP-8]
006167F9  |.  59               POP ECX
006167FA  |.  E8 B5E1DEFF      CALL unpacked.004049B4                 ; 两串合并为一串，76c23c2f
006167FF  |.  8D55 DC          LEA EDX,DWORD PTR SS:[EBP-24]
00616802  |.  8B83 60030000    MOV EAX,DWORD PTR DS:[EBX+360]
00616808  |.  E8 AFD0E3FF      CALL unpacked.004538BC                 ; 取注册认证码第一串，即
                                                                      ; 1234
0061680D  |.  8B45 DC          MOV EAX,DWORD PTR SS:[EBP-24]         ; 1234 入 eax
00616810  |.  8D55 E0          LEA EDX,DWORD PTR SS:[EBP-20]
00616813  |.  E8 0C2CDFFF      CALL unpacked.00409424                 ; 取注册认证码第一串，即
                                                                      ; 1234
00616818  |.  FF75 E0          PUSH DWORD PTR SS:[EBP-20]
0061681B  |.  8D55 D4          LEA EDX,DWORD PTR SS:[EBP-2C]
0061681E  |.  8B83 64030000    MOV EAX,DWORD PTR DS:[EBX+364]
00616824  |.  E8 93D0E3FF      CALL unpacked.004538BC                 ; 取注册认证码第二串，即
                                                                      ; 2345
00616829  |.  8B45 D4          MOV EAX,DWORD PTR SS:[EBP-2C]         ; 2345 入 eax
0061682C  |.  8D55 D8          LEA EDX,DWORD PTR SS:[EBP-28]
0061682F  |.  E8 F02BDFFF      CALL unpacked.00409424                 ; 取注册认证码第二串，即
                                                                      ; 2345
00616834  |.  FF75 D8          PUSH DWORD PTR SS:[EBP-28]
00616837  |.  8D55 CC          LEA EDX,DWORD PTR SS:[EBP-34]
0061683A  |.  8B83 68030000    MOV EAX,DWORD PTR DS:[EBX+368]
00616840  |.  E8 77D0E3FF      CALL unpacked.004538BC                 ; 取注册认证码第三串，即
                                                                      ; 3456
00616845  |.  8B45 CC          MOV EAX,DWORD PTR SS:[EBP-34]
00616848  |.  8D55 D0          LEA EDX,DWORD PTR SS:[EBP-30]
0061684B  |.  E8 D42BDFFF      CALL unpacked.00409424                 ; 取注册认证码第三串，即
                                                                      ; 3456
00616850  |.  FF75 D0          PUSH DWORD PTR SS:[EBP-30]
00616853  |.  8D55 C4          LEA EDX,DWORD PTR SS:[EBP-3C]
00616856  |.  8B83 6C030000    MOV EAX,DWORD PTR DS:[EBX+36C]
0061685C  |.  E8 5BD0E3FF      CALL unpacked.004538BC                 ; 取注册认证码第四串，即
                                                                      ; 4567
00616861  |.  8B45 C4          MOV EAX,DWORD PTR SS:[EBP-3C]
00616864  |.  8D55 C8          LEA EDX,DWORD PTR SS:[EBP-38]
00616867  |.  E8 B82BDFFF      CALL unpacked.00409424                 ; 取注册认证码第四串，即
                                                                      ; 4567
0061686C  |.  FF75 C8          PUSH DWORD PTR SS:[EBP-38]
0061686F  |.  8D45 F4          LEA EAX,DWORD PTR SS:[EBP-C]
00616872  |.  BA 04000000      MOV EDX,4
00616877  |.  E8 ACE1DEFF      CALL unpacked.00404A28                 ; 四串注册申请码合并为一串
                                                                      ; ，即 1234234534564567
0061687C  |.  8B45 F4          MOV EAX,DWORD PTR SS:[EBP-C]
0061687F  |.  E8 E4E0DEFF      CALL unpacked.00404968                 ; 取位数
00616884  |.  8BC8             MOV ECX,EAX
00616886  |.  85C9             TEST ECX,ECX
00616888  |.  75 0F            JNZ SHORT unpacked.00616899            ; 如为空则跳出错
0061688A  |.  B8 446A6100      MOV EAX,unpacked.00616A44
0061688F  |.  E8 705CE3FF      CALL unpacked.0044C504
00616894  |.  E9 36010000      JMP unpacked.006169CF
00616899  |>  33D2             XOR EDX,EDX
0061689B  |.  8B45 F4          MOV EAX,DWORD PTR SS:[EBP-C]           ; 注册申请码
                                                                      ; 1234234534564567 入 eax
0061689E  |.  8A0410           MOV AL,BYTE PTR DS:[EAX+EDX]           ; 注册申请码依次入 al
006168A1  |.  EB 07            JMP SHORT unpacked.006168AA
006168A3  |>  42               /INC EDX
006168A4  |.  8B45 F4          |MOV EAX,DWORD PTR SS:[EBP-C]         ; 注册申请码
                                                                      ; 1234234534564567 入 eax
006168A7  |.  8A0410           |MOV AL,BYTE PTR DS:[EAX+EDX]         ; 注册申请码依次入 al
006168AA  |>  3C 30            CMP AL,30                               ; 注册申请码一位是否为 0
006168AC  |.  72 04           |JB SHORT unpacked.006168B2
006168AE  |.  3C 39           |CMP AL,39                               ; 注册申请码一位是否为 9
006168B0  |.  76 08           |JBE SHORT unpacked.006168BA
006168B2  |>  3C 41           |CMP AL,41                               ; 注册申请码一位是否为 A
006168B4  |.  72 08           |JB SHORT unpacked.006168BE
006168B6  |.  3C 46           |CMP AL,46                               ; 注册申请码一位是否为 Z
006168B8  |.  77 04           |JA SHORT unpacked.006168BE
006168BA  |>  3BCA            |CMP ECX,EDX
```
