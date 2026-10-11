
# 亿虎 Email 邮差 2003c

> 原文：[`www.pediy.com/kssd/pediy05/pediy50669.htm`](https://www.pediy.com/kssd/pediy05/pediy50669.htm)

## 亿虎 Email 邮差注册算法分析

### 一、基础信息

- **作者声明**：初学破解，纯属技术交流，无其它目的。
- **工具**：`ollyDBg1.09`, `W32Dasm10`, 脱壳工具。
- **基本知识**：基础汇编知识，基本工具使用。
- **注册形式**：机器码 + 注册码
- **软件介绍**：

`亿虎 Email 邮差` 是基于 Internet 上标准邮件服务器的专业 Email 群发程序，专为网络营销及电子商务服务。它不同于普通的邮件群发软件，是真正简单、容易使用的邮件发送软件。首先内置高效快速的 `SMTP` 服务器，从而不需要其他 `SMTP` 服务器中转，即可将邮件直接发送到每个收件人信箱，对邮件发送是否成功一目了然。其次 `亿虎 Email 邮差` 采用智能变量替代功能，即同一封信对不同的收件人可以产生不同的内容，这样才可以真正做到一信多发，增加了对收件人友好性。另外全面支持继续发送功能，随时可以想发就发。内置 `1~512` 可调节线程数，让您在短短的时间内快速、准确、直接发送邮件，是信息发布的最佳助手。

- **机器码**：`501229`
- **注册码**：`12345-23456-34567-45678`

### 二、在 W32Dasm 中查找关键信息

在 `W32Dasm` 中找出信息，很容易找到关键部分。

```
:0049B5C6 8B45E0                 mov eax, dword ptr [ebp-20]
:0049B5C9 E8AAD8F6FF             call 00408E78
:0049B5CE 8BD0                   mov edx, eax
:0049B5D0 B98F000000             mov ecx, 0000008F
:0049B5D5 8B45FC                 mov eax, dword ptr [ebp-04]
:0049B5D8 E8B7F5FFFF             call 0049AB94
:0049B5DD 8BD8                   mov ebx, eax
:0049B5DF A1580B4A00             mov eax, dword ptr [004A0B58]
:0049B5E4 885854                 mov byte ptr [eax+54], bl
:0049B5E7 A1580B4A00             mov eax, dword ptr [004A0B58]
:0049B5EC 84DB                   test bl, bl
:0049B5EE 752C                   jne 0049B61C
:0049B5F0 8D45F8                 lea eax, dword ptr [ebp-08]
* Possible StringData Ref from Code Obj ->"无效的注册码!  "
                                   |
:0049B5F3 BA90B64900             mov edx, 0049B690
:0049B5F8 E83387F6FF             call 00403D30
:0049B5FD 6A30                   push 00000030
:0049B5FF 8B45F8                 mov eax, dword ptr [ebp-08]
:0049B602 E8D58AF6FF             call 004040DC
:0049B607 8BD0                   mov edx, eax
```

### 三、用 ollydbg 动态分析

用 `ollydbg` 载入程序，进行动态分析。

```
0049B551  .: 64:FF30     PUSH DWORD PTR FS:[EAX]
0049B554  .: 64:8920     MOV DWORD PTR FS:[EAX],ESP
0049B557  .: 8D55 F4     LEA EDX,DWORD PTR SS:[EBP-C]
0049B55A  .: 8B86 E0020000 MOV EAX,DWORD PTR DS:[ESI+2E0]
0049B560  .: E8 AB51F9FF   CALL Esend.00430710                  ; 取 12345e
0049B565  .: FF75 F4     PUSH DWORD PTR SS:[EBP-C]
0049B568  .: 8D55 F0     LEA EDX,DWORD PTR SS:[EBP-10]
0049B56B  .: 8B86 E8020000 MOV EAX,DWORD PTR DS:[ESI+2E8]
0049B571  .: E8 9A51F9FF   CALL Esend.00430710                  ; 取 23456e
0049B576  .: FF75 F0     PUSH DWORD PTR SS:[EBP-10]
0049B579  .: 8D55 EC     LEA EDX,DWORD PTR SS:[EBP-14]
0049B57C  .: 8B86 EC020000 MOV EAX,DWORD PTR DS:[ESI+2EC]
0049B582  .: E8 8951F9FF   CALL Esend.00430710                  ; 取 34567e
0049B587  .: FF75 EC     PUSH DWORD PTR SS:[EBP-14]
0049B58A  .: 8D55 E8     LEA EDX,DWORD PTR SS:[EBP-18]
0049B58D  .: 8B86 F0020000 MOV EAX,DWORD PTR DS:[ESI+2F0]
0049B593  .: E8 7851F9FF   CALL Esend.00430710                  ; 取 45678e
0049B598  .: FF75 E8     PUSH DWORD PTR SS:[EBP-18]
0049B59B  .: 8D45 FC     LEA EAX,DWORD PTR SS:[EBP-4]
0049B59E  .: BA 04000000   MOV EDX,4
0049B5A3  .: E8 308AF6FF   CALL Esend.00403FD8                  ; 把上面取得的 4 串注册合并成
                                                               ; 一串
0049B5A8  .: 8D55 E4     LEA EDX,DWORD PTR SS:[EBP-1C]
0049B5AB  .: 8B45 FC     MOV EAX,DWORD PTR SS:[EBP-4]
0049B5AE  .: E8 0DD3F6FF   CALL Esend.004088C0                  ; 把串由小写转为大写
0049B5B3  .: 8B55 E4     MOV EDX,DWORD PTR SS:[EBP-1C]
0049B5B6  .: 8D45 FC     LEA EAX,DWORD PTR SS:[EBP-4]
0049B5B9  .: E8 7287F6FF   CALL Esend.00403D30
0049B5BE  .: 8D45 E0     LEA EAX,DWORD PTR SS:[EBP-20]
0049B5C1  .: E8 7EF3FFFF   CALL Esend.0049A944                  ; 取机器码，产生 501229
0049B5C6  .: 8B45 E0     MOV EAX,DWORD PTR SS:[EBP-20]
0049B5C9  .: E8 AAD8F6FF   CALL Esend.00408E78                  ; 501229 转换为十六进制，
                                                               ; eax=0007A5ED
0049B5CE  .: 8BD0        MOV EDX,EAX
0049B5D0  .: B9 8F000000   MOV ECX,8F                           ; 8f 入 ecx
0049B5D5  .: 8B45 FC     MOV EAX,DWORD PTR SS:[EBP-4]          ; 注册码合并串入 eax
0049B5D8  .: E8 B7F5FFFF   CALL Esend.0049AB94                  ; 关键，进入
0049B5DD  .: 8BD8        MOV EBX,EAX
0049B5DF  .: A1 580B4A00   MOV EAX,DWORD PTR DS:[4A0B58]
0049B5E4  .: 8858 54       MOV BYTE PTR DS:[EAX+54],BL
0049B5E7  .: A1 580B4A00   MOV EAX,DWORD PTR DS:[4A0B58]
0049B5EC  .: 84DB        TEST BL,BL
0049B5EE      75 2C       JNZ SHORT Esend.0049B61C             ; 关键跳转
0049B5F0  .: 8D45 F8     LEA EAX,DWORD PTR SS:[EBP-8]
0049B5F3  .: BA 90B64900   MOV EDX,Esend.0049B690
0049B5F8  .: E8 3387F6FF   CALL Esend.00403D30
0049B5FD  .: 6A 30        PUSH 30
0049B5FF  .: 8B45 F8     MOV EAX,DWORD PTR SS:[EBP-8]
0049B602  .: E8 D58AF6FF   CALL Esend.004040DC
0049B607  .: 8BD0        MOV EDX,EAX
0049B609  .: B9 A0B64900   MOV ECX,Esend.0049B6A0
0049B60E  .: A1 8C0A4A00   MOV EAX,DWORD PTR DS:[4A0A8C]
0049B613  .: 8B00        MOV EAX,DWORD PTR DS:[EAX]
0049B615  .: E8 2E38FBFF   CALL Esend.0044EE48                  ; 出错框
0049B61A  .: EB 28        JMP SHORT Esend.0049B644
0049B61C  |> A1 580B4A00   MOV EAX,DWORD PTR DS:[4A0B58]
-------------------------------------
```

#### 关键 CALL 分析

关键 `CALL` 位于 `Esend.0049AB94`

```
0049ABAA  .: 894D F4     MOV DWORD PTR SS:[EBP-C],ECX
0049ABAD  .: 8955 F8     MOV DWORD PTR SS:[EBP-8],EDX
0049ABB0  .: 8945 FC     MOV DWORD PTR SS:[EBP-4],EAX
0049ABB3  .: 8B45 FC     MOV EAX,DWORD PTR SS:[EBP-4]          ; 注册码入 eax
0049ABB6  .: E8 1195F6FF   CALL Esend.004040CC
0049ABBB  .: 33C0        XOR EAX,EAX
0049ABBD  .: 55          PUSH EBP
0049ABBE  .: 68 9AB14900   PUSH Esend.0049B19A
0049ABC3  .: 64:FF30     PUSH DWORD PTR FS:[EAX]
0049ABC6  .: 64:8920     MOV DWORD PTR FS:[EAX],ESP
0049ABC9  .: C645 F3 00    MOV BYTE PTR SS:[EBP-D],0
0049ABCD  .: 8B45 FC     MOV EAX,DWORD PTR SS:[EBP-4]          ; 注册码入 eax
0049ABD0  .: E8 4393F6FF   CALL Esend.00403F18                  ; 取注册码位数
0049ABD5  .: 83F8 14     CMP EAX,14                             ; 注册码是否为 20 位( 5*4=20)
0049ABD8  .: 0F85 89050000 JNZ Esend.0049B167
0049ABDE  .: 8D45 EC     LEA EAX,DWORD PTR SS:[EBP-14]
0049ABE1  .: 8B55 FC     MOV EDX,DWORD PTR SS:[EBP-4]          ; 注册码入 edx
0049ABE4  .: E8 4791F6FF   CALL Esend.00403D30
0049ABE9  .: E8 4AF3F6FF   CALL Esend.00409F38                  ; 取时间
0049ABEE  .: 83C4 F8     ADD ESP,-8                             ; /
0049ABF1  .: DD1C24      FSTP QWORD PTR SS:[ESP]                ; |Arg1 (8-byte)
0049ABF4  .: 9B          WAIT                                   ; |
0049ABF5  .: 8D55 C4     LEA EDX,DWORD PTR SS:[EBP-3C]         ; |
0049ABF8  .: B8 B4B14900   MOV EAX,Esend.0049B1B4              ; |ASCII "hhnnss"
0049ABFD  .: E8 76FFF6FF   CALL Esend.0040AB78                  ; \Esend.0040AB78
0049AC02  .: 8B45 C4     MOV EAX,DWORD PTR SS:[EBP-3C]         ; 取当前时间，为时分秒
0049AC05  .: E8 6EE2F6FF   CALL Esend.00408E78                  ; 时分秒转为十六进制，
                                                               ; eax=0002F775
0049AC0A  .: 8BF8        MOV EDI,EAX
0049AC0C  .: BB 01000000   MOV EBX,1
0049AC11  |> 83FF 0A     /CMP EDI,0A
0049AC14  .: 7C 59       |JL SHORT Esend.0049AC6F
0049AC16  .: 8B45 EC     |MOV EAX,DWORD PTR SS:[EBP-14]        ; 注册码入 eax
0049AC19  .: E8 FA92F6FF   |CALL Esend.00403F18                 ; 取注册码位数
0049AC1E  .: 85C0        |TEST EAX,EAX
0049AC20  .: 7E 2E       |JLE SHORT Esend.0049AC50
0049AC22  .: E8 11F3F6FF   |CALL Esend.00409F38
0049AC27  .: 83C4 F8     |ADD ESP,-8                            ; /
```



```
0049AC2A  .|  DD1C24       |FSTP QWORD PTR SS:[ESP]                  ; |Arg1 (8-byte)  

0049AC2D  .|  9B           |WAIT                                     ; |  

0049AC2E  .|  8D55 C0      |LEA EDX,DWORD PTR SS:[EBP-40]            ; |  

0049AC31  .|  B8 B4B14900  |MOV EAX,Esend.0049B1B4                  ; |ASCII "hhnnss"  

0049AC36  .|  E8 3DFFF6FF  |CALL Esend.0040AB78                     ; \\Esend.0040AB78  

0049AC3B  .|  8B45 C0      |MOV EAX,DWORD PTR SS:[EBP-40]            ;  取当前时间 AX,DW  

0049AC3E  .|  E8 35E2F6FF  |CALL Esend.00408E78                     ;  时间转换为十六进制，

eax=0002F911  

0049AC43  .|  2BC7         |SUB EAX,EDI                              ;  eax 与 edi 相减，即

eax=0002F911-0002F775=19c    I  

0049AC45  .|  83F8 0A      |CMP EAX,0A                               ;  eax 与 0a 是否相等  

0049AC48      0F8F 19050000 JG Esend.0049B167                      ;  如果 eax 大于 0a 则跳走  

0049AC4E  .|  EB 1F        |JMP SHORT Esend.0049AC6F  

0049AC50  |>  8B45 EC      |MOV EAX,DWORD PTR SS:[EBP-14]  

0049AC53  .|  E8 C092F6FF  |CALL Esend.00403F18  

0049AC58  .|  3BD8         |CMP EBX,EAX  

0049AC5A  .|  7D 13        |JGE SHORT Esend.0049AC6F  

0049AC5C  .|  8D45 E8      |LEA EAX,DWORD PTR SS:[EBP-18]  

0049AC5F  .|  50           |PUSH EAX  

0049AC60  .|  B9 01000000  |MOV ECX,1  

0049AC65  .|  8BD3         |MOV EDX,EBX  

0049AC67  .|  8B45 EC      |MOV EAX,DWORD PTR SS:[EBP-14]  

0049AC6A  .|  E8 B194F6FF  |CALL Esend.00404120  

0049AC6F  |>  43           |INC EBX  

0049AC70  .|  81FB F5010000 |CMP EBX,1F5  

0049AC76  |.^ 75 99        \\JNZ SHORT Esend.0049AC11               ;  以上为取得时间，如两时间

相差 0a，则跳走，所以应在下面断                I?  

0049AC78  .|  BB 01000000  MOV EBX,1  

0049AC7D  |>  8D45 BC      /LEA EAX,DWORD PTR SS:[EBP-44]  

0049AC80  .|  50           |PUSH EAX  

0049AC81  .|  B9 01000000  |MOV ECX,1  

0049AC86  .|  8BD3         |MOV EDX,EBX  

0049AC88  .|  8B45 EC      |MOV EAX,DWORD PTR SS:[EBP-14]           ;  注册码入 eax  

0049AC8B  .|  E8 9094F6FF  |CALL Esend.00404120                     ;  依次取注册码第一位、第二

位...  

0049AC90  .|  8B45 BC      |MOV EAX,DWORD PTR SS:[EBP-44]  

0049AC93  .|  BA C4B14900  |MOV EDX,Esend.0049B1C4                  ;  Z 入 edxD  

0049AC98  .|  E8 8B93F6FF  |CALL Esend.00404028                     ;  注册码各位与 Z 比较是否相等

404028  

0049AC9D  .|  0F87 C4040000 |JA Esend.0049B167  

0049ACA3  .|  8D45 B8      |LEA EAX,DWORD PTR SS:[EBP-48]  

0049ACA6  .|  50           |PUSH EAX  

0049ACA7  .|  B9 01000000  |MOV ECX,1  

0049ACAC  .|  8BD3         |MOV EDX,EBX  

0049ACAE  .|  8B45 EC      |MOV EAX,DWORD PTR SS:[EBP-14]           ;  注册码入 eax  

0049ACB1  .|  E8 6A94F6FF  |CALL Esend.00404120                     ;  依次取注册码第一位、第二

位...  

0049ACB6  .|  8B45 B8      |MOV EAX,DWORD PTR SS:[EBP-48]  

0049ACB9  .|  BA D0B14900  |MOV EDX,Esend.0049B1D0                  ;  A 入 edx  

0049ACBE  .|  E8 6593F6FF  |CALL Esend.00404028                     ;  比较 1 与 A 是否相等 d.00404  

0049ACC3  .|  0F82 9E040000 |JB Esend.0049B167  

0049ACC9  .|  43           |INC EBX  

0049ACCA  .|  83FB 15      |CMP EBX,15                               ;  以上循环 21 次，检查注册码

是否是 A-Z 间的字母。                 I?  

0049ACCD  |.^ 75 AE        \\JNZ SHORT Esend.0049AC7D  

0049ACCF  .|  8D45 E8      LEA EAX,DWORD PTR SS:[EBP-18]  

0049ACD2  .|  8B55 FC      MOV EDX,DWORD PTR SS:[EBP-4]              ;  注册码 edx  

0049ACD5  .|  E8 5690F6FF  CALL Esend.00403D30  

0049ACDA  .|  33F6         XOR ESI,ESI  

0049ACDC  .|  BB 01000000  MOV EBX,1  

0049ACE1  |>  8D45 E4      /LEA EAX,DWORD PTR SS:[EBP-1C]  

0049ACE4  .|  50           |PUSH EAX  

0049ACE5  .|  B9 01000000  |MOV ECX,1  

0049ACEA  .|  8BD3         |MOV EDX,EBX  

0049ACEC  .|  8B45 E8      |MOV EAX,DWORD PTR SS:[EBP-18]           ;  注册码入 eax  

0049ACEF  .|  E8 2C94F6FF  |CALL Esend.00404120                     ;  依次取 1 位注册码  

0049ACF4  .|  BA DCB14900  |MOV EDX,Esend.0049B1DC                  ;  串

GFEDCBANMLKJIHTSRQPOZYXWVU 入 edx  

0049ACF9  .|  8B45 E4      |MOV EAX,DWORD PTR SS:[EBP-1C]           ;  串

GFEDCBANMLKJIHTSRQPOZYXWVU 入 eax  

0049ACFC  .|  E8 0395F6FF  |CALL Esend.00404204                     ;  检查注册码第 N 位在上面串的

位置，位置数入 eax                    I?  

0049AD01  .|  8BF8         |MOV EDI,EAX                              ;  eax 入 edi,  

0049AD03  .|  4F           |DEC EDI                                  ;  edi 减 1D  

0049AD04  .|  0FAFFB       |IMUL EDI,EBX                             ;  edi 乘注册码第 N 位 BX  

0049AD07  .|  03F7         |ADD ESI,EDI                              ;  把结果相加 SI,ED  

0049AD09  .|  43           |INC EBX  

0049AD0A  .|  83FB 14      |CMP EBX,14                               ;  循环 20 次 BX,  

0049AD0D  |.^ 75 D2        \\JNZ SHORT Esend.0049ACE1                ;  (第 N 位注册码在密码串中的

位置-1)乘注册码第 N 位，各位相加，并入 esi    I?  

0049AD0F  .|  8BC6         MOV EAX,ESI                                ;  把 esi 入 eaxSI  

0049AD11  .|  B9 1A000000  MOV ECX,1A                                 ;  1A 入 ecx,  

0049AD16  .|  99           CDQ  

0049AD17  .|  F7F9         IDIV ECX                                   ;  ESI 除 1A,余数入 edx  

0049AD19  .|  42           INC EDX                                    ;  余数 edx 增 1  

0049AD1A  .|  8BF2         MOV ESI,EDX                                ;  edx 入 esiE  

0049AD1C  .|  8D45 E4      LEA EAX,DWORD PTR SS:[EBP-1C]  

0049AD1F  .|  50           PUSH EAX  

0049AD20  .|  B8 DCB14900  MOV EAX,Esend.0049B1DC                     ;  串

GFEDCBANMLKJIHTSRQPOZYXWVU 入 eax  

0049AD25  .|  B9 01000000  MOV ECX,1  

0049AD2A  .|  8BD6         MOV EDX,ESI                                ;  上面计算的模数减 1 的值入

edx  

0049AD2C  .|  E8 EF93F6FF  CALL Esend.00404120                       ;  取第 edx 位在密码串中的值

04120  

0049AD31  .|  8D45 B4      LEA EAX,DWORD PTR SS:[EBP-4C]  

0049AD34  .|  50           PUSH EAX  

0049AD35  .|  B9 01000000  MOV ECX,1  

0049AD3A  .|  BA 14000000  MOV EDX,14  

0049AD3F  .|  8B45 E8      MOV EAX,DWORD PTR SS:[EBP-18]             ;  注册码入 eaxDWOR  

0049AD42  .|  E8 D993F6FF  CALL Esend.00404120                       ;  取注册码最后一位 d.004041  

0049AD47  .|  8B55 B4      MOV EDX,DWORD PTR SS:[EBP-4C]             ;  注册码最后一位入 edxD PTR 

SS  

0049AD4A  .|  8B45 E4      MOV EAX,DWORD PTR SS:[EBP-1C]             ;  上面在密码表中取得的值入

eaxR SS:[EBP-1C  

0049AD4D  .|  E8 D692F6FF  CALL Esend.00404028                       ;  比较是否相等 end.00  

0049AD52      0F85 0F040000 JNZ Esend.0049B167  

0049AD58  .|  33F6         XOR ESI,ESI  

0049AD5A  .|  BB 01000000  MOV EBX,1  

0049AD5F  |>  8D45 E4      /LEA EAX,DWORD PTR SS:[EBP-1C]  

0049AD62  .|  50           |PUSH EAX  

0049AD63  .|  B9 01000000  |MOV ECX,1  

0049AD68  .|  8BD3         |MOV EDX,EBX  

0049AD6A  .|  8B45 E8      |MOV EAX,DWORD PTR SS:[EBP-18]           ;  注册码入 eax  

0049AD6D  .|  E8 AE93F6FF  |CALL Esend.00404120  

0049AD72  .|  BA DCB14900  |MOV EDX,Esend.0049B1DC                  ;  ASCII 

"GFEDCBANMLKJIHTSRQPOZYXWVU"  

0049AD77  .|  8B45 E4      |MOV EAX,DWORD PTR SS:[EBP-1C]  

0049AD7A  .|  E8 8594F6FF  |CALL Esend.00404204  

0049AD7F  .|  8BF8         |MOV EDI,EAX  

0049AD81  .|  4F           |DEC EDI  

0049AD82  .|  03F7         |ADD ESI,EDI  

0049AD84  .|  43           |INC EBX  

0049AD85  .|  83FB 13      |CMP EBX,13  

0049AD88  |.^ 75 D5        \\JNZ SHORT Esend.0049AD5F                ;  注册码前面 19 位按上面生成

一值  

0049AD8A  .|  8BC6         MOV EAX,ESI  

0049AD8C  .|  B9 1A000000  MOV ECX,1A  

0049AD91  .|  99           CDQ  

0049AD92  .|  F7F9         IDIV ECX  

0049AD94  .|  42           INC EDX  

0049AD95  .|  8BF2         MOV ESI,EDX  

0049AD97  .|  8D45 E4      LEA EAX,DWORD PTR SS:[EBP-1C]  

0049AD9A  .|  50           PUSH EAX  

0049AD9B  .|  B8 DCB14900  MOV EAX,Esend.0049B1DC                    ;  ASCII 

"GFEDCBANMLKJIHTSRQPOZYXWVU"  
```



```
0049ADA0  |.  B9 01000000   MOV ECX,1  

0049ADA5  |.  8BD6          MOV EDX,ESI  

0049ADA7  |.  E8 7493F6FF   CALL Esend.00404120  

0049ADAC  |.  8D45 B0       LEA EAX,DWORD PTR SS:\[EBP\-50\]  

0049ADAF  |.  50            PUSH EAX  

0049ADB0  |.  B9 01000000   MOV ECX,1  

0049ADB5  |.  BA 13000000   MOV EDX,13  

0049ADBA  |.  8B45 E8       MOV EAX,DWORD PTR SS:\[EBP\-18\]          ;  注册码入 eax  

0049ADBD  |.  E8 5E93F6FF   CALL Esend.00404120                     ;  取注册码第 19 位  

0049ADC2  |.  8B55 B0       MOV EDX,DWORD PTR SS:\[EBP\-50\]  

0049ADC5  |.  8B45 E4       MOV EAX,DWORD PTR SS:\[EBP\-1C\]  

0049ADC8  |.  E8 5B92F6FF   CALL Esend.00404028  

0049ADCD     0F85 94030000 JNZ Esend.0049B167  

0049ADD3  |.  8D45 E8       LEA EAX,DWORD PTR SS:\[EBP\-18\]  

0049ADD6  |.  50            PUSH EAX  

0049ADD7  |.  B9 12000000   MOV ECX,12  

0049ADDC  |.  BA 01000000   MOV EDX,1  

0049ADE1  |.  8B45 FC       MOV EAX,DWORD PTR SS:\[EBP\-4\]           ;  注册码入 eax  

0049ADE4  |.  E8 3793F6FF   CALL Esend.00404120                     ;  取注册码 18 位在密码表中的

位置  

0049ADE9  |.  8D45 EC       LEA EAX,DWORD PTR SS:\[EBP\-14\]  

0049ADEC  |.  E8 A78EF6FF   CALL Esend.00403C98  

0049ADF1  |.  BB 01000000   MOV EBX,1  

0049ADF6  |>  8D45 AC       /LEA EAX,DWORD PTR SS:\[EBP\-54\]  

0049ADF9  |.  50            |PUSH EAX  

0049ADFA  |.  BA 12000000   |MOV EDX,12  

0049ADFF  |.  2BD3          |SUB EDX,EBX  

0049AE01  |.  42            |INC EDX  

0049AE02  |.  B9 01000000   |MOV ECX,1  

0049AE07  |.  8B45 E8       |MOV EAX,DWORD PTR SS:\[EBP\-18\]  

0049AE0A  |.  E8 1193F6FF   |CALL Esend.00404120  

0049AE0F  |.  8B55 AC       |MOV EDX,DWORD PTR SS:\[EBP\-54\]  

0049AE12  |.  8D45 EC       |LEA EAX,DWORD PTR SS:\[EBP\-14\]  

0049AE15  |.  E8 0691F6FF   |CALL Esend.00403F20  

0049AE1A  |.  43            |INC EBX  

0049AE1B  |.  83FB 13       |CMP EBX,13  

0049AE1E  |.^ 75 D6          \JNZ SHORT Esend.0049ADF6  

0049AE20  |.  8D45 E8       LEA EAX,DWORD PTR SS:\[EBP\-18\]  

0049AE23  |.  8B55 EC       MOV EDX,DWORD PTR SS:\[EBP\-14\]  

0049AE26  |.  E8 058FF6FF   CALL Esend.00403D30  

0049AE2B  |.  8D45 EC       LEA EAX,DWORD PTR SS:\[EBP\-14\]  

0049AE2E  |.  E8 658EF6FF   CALL Esend.00403C98  

0049AE33  |.  BB 01000000   MOV EBX,1  

0049AE38  |>  FF75 EC       /PUSH DWORD PTR SS:\[EBP\-14\]  

0049AE3B  |.  8D45 A8       |LEA EAX,DWORD PTR SS:\[EBP\-58\]  

0049AE3E  |.  50            |PUSH EAX  

0049AE3F  |.  B9 01000000   |MOV ECX,1  

0049AE44  |.  8BD3          |MOV EDX,EBX  

0049AE46  |.  8B45 E8       |MOV EAX,DWORD PTR SS:\[EBP\-18\]  

0049AE49  |.  E8 D292F6FF   |CALL Esend.00404120  

0049AE4E  |.  FF75 A8       |PUSH DWORD PTR SS:\[EBP\-58\]  

0049AE51  |.  8D45 A4       |LEA EAX,DWORD PTR SS:\[EBP\-5C\]  

0049AE54  |.  50            |PUSH EAX  

0049AE55  |.  8D53 09       |LEA EDX,DWORD PTR DS:\[EBX+9\]  

0049AE58  |.  B9 01000000   |MOV ECX,1  

0049AE5D  |.  8B45 E8       |MOV EAX,DWORD PTR SS:\[EBP\-18\]  

0049AE60  |.  E8 BB92F6FF   |CALL Esend.00404120  

0049AE65  |.  FF75 A4       |PUSH DWORD PTR SS:\[EBP\-5C\]  

0049AE68  |.  8D45 EC       |LEA EAX,DWORD PTR SS:\[EBP\-14\]  

0049AE6B  |.  BA 03000000   |MOV EDX,3  

0049AE70  |.  E8 6391F6FF   |CALL Esend.00403FD8  

0049AE75  |.  43            |INC EBX  

0049AE76  |.  83FB 13       |CMP EBX,13  

0049AE79  |.^ 75 BD          \JNZ SHORT Esend.0049AE38  

0049AE7B  |.  8D45 EC       LEA EAX,DWORD PTR SS:\[EBP\-14\]  

0049AE7E  |.  50            PUSH EAX  

0049AE7F  |.  B9 0F000000   MOV ECX,0F
```

### 三、总结

这是一个不错的国产软件，为了防止破解扩散，只公开前面两部分的注册算法分析。

这个软件有一处反跟踪代码，是通过判断两次时间差，如发现有两次时间差超过 `10` 秒，则会直接出错。

密码表：`GFEDCBANMLKJIHTSRQPOZYXWVU`

```
      m1m2m3''MI''''m25m26
注册码：s1s2s3'''sN''s18s19s20
```

1. 注册码第 `N` 位在密码串的位置数 `I`，位置数 `I` 入 `eax`，并计算：

```
X=0
for(i=0;i<21;i++)
{
X=X+(I-1)*注册码第 N 位;
}
```

注册码依次查表，并把各位相加为 `X`，最后取`X/26`的模数，然后模数加 `1`，得数 `Y`，在密码表中取第 `Y` 位字母，与注册码第 `20` 位比较

2. 注册码第 `N` 位在密码串的位置数 `I`，位置数 `I` 入 `eax`，并计算：

```
X=0;
for(i=0;i<20;i++)
{
X=X+(I-1)
}
```

（注册码第 `N` 位在密码串中的位置数 - 1）并各相加，注册码第 `20` 位不参与运算。最后取`X/26`的模数，然后模数加 `1`，得数 `Y`，密码表中取第 `Y` 位字母，与注册码第 `19` 位比较.

`4dd466d0e1e0cc123afbe504da2462ca`
