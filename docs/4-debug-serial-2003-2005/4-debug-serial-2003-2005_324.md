
# 仓管王 V12.6 单机版

> 原文：[`www.pediy.com/kssd/pediy05/pediy50626.htm`](https://www.pediy.com/kssd/pediy05/pediy50626.htm)

**软件信息**

- 软件大小： 4399 KB
- 软件语言： 简体中文
- 软件类别： 国产软件 / 共享版 / 商业贸易
- 应用平台： `Win9x/NT/2000/XP`
- 界面预览： 无
- 加入时间： 2003-03-23 10:02:16
- 下载次数： 15385
- 推荐等级：
- 联系人： `tech@shd.com.cn`
- 开发商： [`www.shd.com.cn/`](http://www.shd.com.cn/)

**软件介绍**

全面优化、占用资源极少、简便实用、完全符合国家规范。完全满足企事业、行政单位的仓库、财产、物资管理的要求。可选择金额、数量记帐法。除设有基本的入库单、出库单、调拨单、报废单、盘点单外，尚有功能强大的单据修改、单据撤销、单据审批、分类统计等全自动的统计功能，是同类产品中功能最强、价格最低的优秀产品。拥有全国各地近 10 万用户。

**作者声明**

初学破解，纯属技术交流，无其它目的。

**工具**

`ollyDBg1.09`、`W32Dasm10`、`Caspr110` 脱壳工具。

**假设数据**

- 系列号：`166697841106`
- 注册号(假码)：`1234-2345-3456-4567`

先用脱壳工具脱壳。

### 一、查找出错信息

```
:00653080 E87B24E4FF             call 00495500
:00653085 8B45E8                 mov eax, dword ptr [ebp-18]
:00653088 5A                     pop edx
:00653089 E802620400             call 00699290====>关键 call
:0065308E 84C0                   test al, al
:00653090 7530                   jne 006530C2====>此改为 je 即爆破
:00653092 6A10                   push 00000010

* Possible StringData Ref from Code Obj ->"´错误"
                       |
:00653094 B9E4326500             mov ecx, 006532E4

* Possible StringData Ref from Code Obj ->"注册号错误 "
                       |
:00653099 BAEC326500             mov edx, 006532EC
:0065309E A1D45D6B00             mov eax, dword ptr [006B5DD4]
:006530A3 8B00                   mov eax, dword ptr [eax]
:006530A5 E8DE3AE3FF             call 00486B88
```

### 二、用 `ollydbg` 载入

```
0065301C  |. E8 DF24E4FF    CALL aa.00495500                        ;  取得 1234.0
00653021  |. FF75 F8        PUSH DWORD PTR SS:[EBP-8]               ;  push1234
00653024  |. 68 E0326500    PUSH aa.006532E0
00653029  |. 8D55 F4        LEA EDX,DWORD PTR SS:[EBP-C]
0065302C  |. 8B83 54030000  MOV EAX,DWORD PTR DS:[EBX+354]
00653032  |. E8 C924E4FF    CALL aa.00495500                        ;  取得 2345.0
00653037  |. FF75 F4        PUSH DWORD PTR SS:[EBP-C]
0065303A  |. 68 E0326500    PUSH aa.006532E0
0065303F  |. 8D55 F0        LEA EDX,DWORD PTR SS:[EBP-10]
00653042  |. 8B83 58030000  MOV EAX,DWORD PTR DS:[EBX+358]
00653048  |. E8 B324E4FF    CALL aa.00495500                        ;  取 3456a
0065304D  |. FF75 F0        PUSH DWORD PTR SS:[EBP-10]
00653050  |. 68 E0326500    PUSH aa.006532E0
00653055  |. 8D55 EC        LEA EDX,DWORD PTR SS:[EBP-14]
00653058  |. 8B83 5C030000  MOV EAX,DWORD PTR DS:[EBX+35C]
0065305E  |. E8 9D24E4FF    CALL aa.00495500                        ;  取 4567a
00653063  |. FF75 EC        PUSH DWORD PTR SS:[EBP-14]
00653066  |. 8D45 FC        LEA EAX,DWORD PTR SS:[EBP-4]
00653069  |. BA 07000000    MOV EDX,7
0065306E  |. E8 B121DBFF    CALL aa.00405224                        ;  形成串：
1234-2345-3456-4567
00653073  |. 8B45 FC        MOV EAX,DWORD PTR SS:[EBP-4]
00653076  |. 50             PUSH EAX
00653077  |. 8D55 E8        LEA EDX,DWORD PTR SS:[EBP-18]
0065307A  |. 8B83 4C030000  MOV EAX,DWORD PTR DS:[EBX+34C]
00653080  |. E8 7B24E4FF    CALL aa.00495500                        ;  取系列号 aa.0
00653085  |. 8B45 E8        MOV EAX,DWORD PTR SS:[EBP-18]          ;  系列号 EAX
00653088  |. 5A             POP EDX                                 ;  取出输入的注册码
00653089  |. E8 02620400    CALL aa.00699290                        ;  关键，F7 进入 00699
0065308E  |. 84C0           TEST AL,AL
00653090  |. 7530           jne 006530C2====>此改为 je 即爆破
00653092  |. 6A 10          PUSH 10
```

### 三、关键 `call` 分析

```
\-----------------------------------------
006992A1  |. 56             PUSH ESI
006992A2  |. 8955 F8        MOV DWORD PTR SS:[EBP-8],EDX            ;  输入串保存到 ebp-8
006992A5  |. 8945 FC        MOV DWORD PTR SS:[EBP-4],EAX            ;  系列号保存到 ebp-4
006992A8  |. 8B45 FC        MOV EAX,DWORD PTR SS:[EBP-4]
006992AB  |. E8 9CC0D6FF    CALL aa.0040534C
006992B0  |. 8B45 F8        MOV EAX,DWORD PTR SS:[EBP-8]
006992B3  |. E8 94C0D6FF    CALL aa.0040534C
006992B8  |. 33C0           XOR EAX,EAX
006992BA  |. 55             PUSH EBP
006992BB  |. 68 02956900    PUSH aa.00699502
006992C0  |. 64:FF30        PUSH DWORD PTR FS:[EAX]
006992C3  |. 64:8920        MOV DWORD PTR FS:[EAX],ESP
006992C6  |. 8D45 F4        LEA EAX,DWORD PTR SS:[EBP-C]
006992C9  |. E8 DEBBD6FF    CALL aa.00404EAC
006992CE  |. 66:BB 0100     MOV BX,1
006992D2  |> 8BF3           /MOV ESI,EBX
006992D4  |. 03F6           |ADD ESI,ESI
006992D6  |. 4E             |DEC ESI
006992D7  |. 8D45 E0        |LEA EAX,DWORD PTR SS:[EBP-20]
006992DA  |. 50             |PUSH EAX
006992DB  |. 0FB7D6         |MOVZX EDX,SI
006992DE  |. B9 01000000    |MOV ECX,1
006992E3  |. 8B45 FC        |MOV EAX,DWORD PTR SS:[EBP-4]          ;  系列号入 eax
006992E6  |. E8 D1C0D6FF    |CALL aa.004053BC
006992EB  |. 8B55 E0        |MOV EDX,DWORD PTR SS:[EBP-20]
006992EE  |. 8D45 F4        |LEA EAX,DWORD PTR SS:[EBP-C]
006992F1  |. E8 76BED6FF    |CALL aa.0040516C
006992F6  |. 43             |INC EBX
006992F7  |. 66:83FB 05     |CMP BX,5
006992FB  |. JNZ SHORT aa.006992D2                    ;  以上为依次取系列号的 1、3、5、7 位
006992FD  |. 8D45 F0        LEA EAX,DWORD PTR SS:[EBP-10]
00699300  |. 50             PUSH EAX
00699301  |. B9 04000000    MOV ECX,4
00699306  |. BA 01000000    MOV EDX,1                 ;
0069930B  |. 8B45 F8        MOV EAX,DWORD PTR SS:[EBP-8]      ;  注册码串入 eax
0069930E  |. E8 A9C0D6FF    CALL aa.004053BC                      ;  取串（1）1234
00699313  |. 8D45 EC        LEA EAX,DWORD PTR SS:[EBP-14]
00699316  |. 50             PUSH EAX
00699317  |. B9 04000000    MOV ECX,4
0069931C  |. BA 06000000    MOV EDX,6                 ;  表示从串第六位取四位
00699321  |. 8B45 F8        MOV EAX,DWORD PTR SS:[EBP-8]
00699324  |. E8 93C0D6FF    CALL aa.004053BC                      ;  取串（2）2345
00699329  |. 8D45 E8        LEA EAX,DWORD PTR SS:[EBP-18]
0069932C  |. 50             PUSH EAX
0069932D  |. B9 04000000    MOV ECX,4
00699332  |. BA 0B000000    MOV EDX,0B
00699337  |. 8B45 F8        MOV EAX,DWORD PTR SS:[EBP-8]
0069933A  |. E8 7DC0D6FF    CALL aa.004053BC                      ;  取串（3）3456
0069933F  |. 8D45 E4        LEA EAX,DWORD PTR SS:[EBP-1C]
00699342  |. 50             PUSH EAX
00699343  |. B9 04000000    MOV ECX,4
00699348  |. BA 10000000    MOV EDX,10
0069934D  |. 8B45 F8        MOV EAX,DWORD PTR SS:[EBP-8]
00699350  |. E8 67C0D6FF    CALL aa.004053BC                      ;  取串（4）4567
00699355  |. 8B45 FC        MOV EAX,DWORD PTR SS:[EBP-4]          ;  系列号入 eax
00699358  |. E8 07BED6FF    CALL aa.00405164                      ;  取系列号位数
0069935D  |. 8BD8           MOV EBX,EAX
0069935F  |. 8D45 D8        LEA EAX,DWORD PTR SS:[EBP-28]
00699362  |. 50             PUSH EAX
00699363  |. B9 01000000    MOV ECX,1
00699368  |. BA 01000000    MOV EDX,1
0069936D  |. 8B45 F0        MOV EAX,DWORD PTR SS:[EBP-10]         ;  输入串 1234 入 eax
00699370  |. E8 47C0D6FF    CALL aa.004053BC                      ;  取串第一位，保存在 ebp-28 地址处
00699375  |. FF75 D8        PUSH DWORD PTR SS:[EBP-28]
00699378  |. 8D45 D4        LEA EAX,DWORD PTR SS:[EBP-2C]
0069937B  |. 50             PUSH EAX
0069937C  |. B9 01000000    MOV ECX,1
00699381  |. BA 02000000    MOV EDX,2                 ;
00699386  |. 8B45 EC        MOV EAX,DWORD PTR SS:[EBP-14]         ;  串（2）2345 入 eax
00699389  |. E8 2EC0D6FF    CALL aa.004053BC                      ;  取得串 2 的第二位即 3
0069938E  |. FF75 D4        PUSH DWORD PTR SS:[EBP-2C]
00699391  |. 8D45 D0        LEA EAX,DWORD PTR SS:[EBP-30]
```



```
00699394  . 50                PUSH EAX
00699395  . B9 01000000       MOV ECX,1
0069939A  . BA 03000000       MOV EDX,3
0069939F  . 8B45 E8           MOV EAX,DWORD PTR SS:[EBP-18]          ; 串（3）3456
006993A2  . E8 15C0D6FF       CALL aa.004053BC                        ;取串（3）第 3 位
006993A7  . FF75 D0           PUSH DWORD PTR SS:[EBP-30]              ; 取得数字 5
006993AA  . 8D45 CC           LEA EAX,DWORD PTR SS:[EBP-34]
006993AD  . 50                PUSH EAX
006993AE  . B9 01000000       MOV ECX,1
006993B3  . BA 04000000       MOV EDX,4
006993B8  . 8B45 E4           MOV EAX,DWORD PTR SS:[EBP-1C]          ; 串（4）4567 入 eax
006993BB  . E8 FCBFD6FF       CALL aa.004053BC                        ; 取串（4）第 4 位，取得数字 7
006993C0  . FF75 CC           PUSH DWORD PTR SS:[EBP-34]
006993C3  . 8D45 DC           LEA EAX,DWORD PTR SS:[EBP-24]
006993C6  . BA 04000000       MOV EDX,4
006993CB  . E8 54BED6FF       CALL aa.00405224                        ; 取得的各位形成串，即为 1357
006993D0  . 8B55 DC           MOV EDX,DWORD PTR SS:[EBP-24]
006993D3  . 8B45 F4           MOV EAX,DWORD PTR SS:[EBP-C]            ; 系列号的 1、3、5、7 位，即 1698
006993D6  . E8 CDBED6FF       CALL aa.004052A8                        ; 系列号的 1、3、5、7 位，即 1698，与串（1）的第 1 位，串（2）第 2 位，串（3）第 3 位，串（4）第 4 位各位比较
006993DB  . 0F85 F2000000     JNZ aa.006994D3                         ; 关键判断 1
006993E1  . 8D45 C8           LEA EAX,DWORD PTR SS:[EBP-38]
006993E4  . 50                PUSH EAX
006993E5  . B9 01000000       MOV ECX,1
006993EA  . BA 04000000       MOV EDX,4
006993EF  . 8B45 F0           MOV EAX,DWORD PTR SS:[EBP-10]          ; 串（1）1234 入 eax
006993F2  . E8 C5BFD6FF       CALL aa.004053BC                        ; 从串的第 4 位，即为 4
006993F7  . 8B45 C8           MOV EAX,DWORD PTR SS:[EBP-38]
006993FA  . 50                PUSH EAX
006993FB  . 8D45 C4           LEA EAX,DWORD PTR SS:[EBP-3C]
006993FE  . 50                PUSH EAX
006993FF  . 0FB7D3            MOVZX EDX,BX                            ; 系列号的位数入 edx,即为 12
00699402  . 83EA 03           SUB EDX,3                               ; 位数－3，edx=9
00699405  . B9 01000000       MOV ECX,1                               ; ecx 为 1
0069940A  . 8B45 FC           MOV EAX,DWORD PTR SS:[EBP-4]            ; 系列号入 eax
0069940D  . E8 AABFD6FF       CALL aa.004053BC                        ; 取系列号的第 9 位，即 1
00699412  . 8B55 C4           MOV EDX,DWORD PTR SS:[EBP-3C]
00699415  . 58                POP EAX
00699416  . E8 8DBED6FF       CALL aa.004052A8                        ; 系列号的第 9 位与串 1 的第 4 位比较
0069941B  . 0F85 B2000000     JNZ aa.006994D3                         ; 关键判断 2
00699421  . 8D45 C0           LEA EAX,DWORD PTR SS:[EBP-40]
00699424  . 50                PUSH EAX
00699425  . B9 01000000       MOV ECX,1
0069942A  . BA 03000000       MOV EDX,3
0069942F  . 8B45 EC           MOV EAX,DWORD PTR SS:[EBP-14]          ; 串（2）2645
00699432  . E8 85BFD6FF       CALL aa.004053BC                        ; 取串（2）第三位
00699437  . 8B45 C0           MOV EAX,DWORD PTR SS:[EBP-40]
0069943A  . 50                PUSH EAX
0069943B  . 8D45 BC           LEA EAX,DWORD PTR SS:[EBP-44]
0069943E  . 50                PUSH EAX
0069943F  . 0FB7D3            MOVZX EDX,BX                            ; 系列号位数入 edx
00699442  . 83EA 02           SUB EDX,2                               ; edx 减 2，edx 为 10
00699445  . B9 01000000       MOV ECX,1
0069944A  . 8B45 FC           MOV EAX,DWORD PTR SS:[EBP-4]            ; 系列号入 eax
0069944D  . E8 6ABFD6FF       CALL aa.004053BC                        ; 取得系列号第 10 位
00699452  . 8B55 BC           MOV EDX,DWORD PTR SS:[EBP-44]
00699455  . 58                POP EAX
00699456  . E8 4DBED6FF       CALL aa.004052A8                        ; 系列号第 10 位与串（2）第三位比较
0069945B  . 75 76             JNZ SHORT aa.006994D3                   ; 关键判断 3
0069945D  . 8D45 B8           LEA EAX,DWORD PTR SS:[EBP-48]
00699460  . 50                PUSH EAX
00699461  . B9 01000000       MOV ECX,1
00699466  . BA 02000000       MOV EDX,2
0069946B  . 8B45 E8           MOV EAX,DWORD PTR SS:[EBP-18]          ; 串（3）3459 入 eax
0069946E  . E8 49BFD6FF       CALL aa.004053BC                        ; 取串（3）第二位即为 4
00699473  . 8B45 B8           MOV EAX,DWORD PTR SS:[EBP-48]
00699476  . 50                PUSH EAX
00699477  . 8D45 B4           LEA EAX,DWORD PTR SS:[EBP-4C]
0069947A  . 50                PUSH EAX
0069947B  . 0FB7D3            MOVZX EDX,BX                            ; 系列号位数入 edx
0069947E  . 4A                DEC EDX                                 ; 减 1 即为第 11 位
0069947F  . B9 01000000       MOV ECX,1
00699484  . 8B45 FC           MOV EAX,DWORD PTR SS:[EBP-4]
00699487  . E8 30BFD6FF       CALL aa.004053BC                        ; 取系列号的第 11 位
0069948C  . 8B55 B4           MOV EDX,DWORD PTR SS:[EBP-4C]
0069948F  . 58                POP EAX
00699490  . E8 13BED6FF       CALL aa.004052A8                        ; 取系列号的第 11 位与串（3）第二位比较
00699495  . 75 3C             JNZ SHORT aa.006994D3                   ; 关键判断 4
00699497  . 8D45 B0           LEA EAX,DWORD PTR SS:[EBP-50]
0069949A  . 50                PUSH EAX
0069949B  . B9 01000000       MOV ECX,1
006994A0  . BA 01000000       MOV EDX,1
006994A5  . 8B45 E4           MOV EAX,DWORD PTR SS:[EBP-1C]          ; 串（4）入 eax
006994A8  . E8 0FBFD6FF       CALL aa.004053BC                        ; 取串（4）第 1 位
006994AD  . 8B45 B0           MOV EAX,DWORD PTR SS:[EBP-50]
006994B0  . 50                PUSH EAX
006994B1  . 8D45 AC           LEA EAX,DWORD PTR SS:[EBP-54]
006994B4  . 50                PUSH EAX
006994B5  . 0FB7D3            MOVZX EDX,BX
006994B8  . 83EA 00           SUB EDX,0
006994BB  . B9 01000000       MOV ECX,1
006994C0  . 8B45 FC           MOV EAX,DWORD PTR SS:[EBP-4]
006994C3  . E8 F4BED6FF       CALL aa.004053BC                        ; 取系列号第 12 位
006994C8  . 8B55 AC           MOV EDX,DWORD PTR SS:[EBP-54]
006994CB  . 58                POP EAX
006994CC  . E8 D7BDD6FF       CALL aa.004052A8                        ; 系列号第 12 位与串（4）第 1 位比较
006994D1  . 74 04             JE SHORT aa.006994D7                    ; 关键判断 5
006994D3  > 33DB              XOR EBX,EBX
006994D5  . EB 02             JMP SHORT aa.006994D9
```

### 算法总结

1. 系列号的 1、3、5、7 位，即 `1698`，与串（1）的第 1 位，串（2）第 2 位，串（3）第 3 位，串（4）第 4 位相等
2. 系列号的第 9 位与串(1)的第 4 位相等
3. 系列号第 10 位与串（2）第 3 位相等
4. 系列号第 11 位与串（3）第 2 位相等
5. 系列号第 12 位与串（4）第 1 位相等

### 总结

系列号：`166697841106`

注册号：`1231-2615-3096-6568`

`lordor`

`4.20`

`4dd466d0e1e0cc123afbe504da2462ca`
