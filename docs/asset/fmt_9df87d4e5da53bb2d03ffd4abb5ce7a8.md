
# 汇编代码分析

```
0041F7AB   895D FC          MOV DWORD PTR SS:[EBP-4],EBX

0041F7AE   FFD6             CALL ESI                           ; EAX A 用户名长度,EDX "FTBirthday"

0041F7B0   8BF0             MOV ESI,EAX                        ; ESI=A

0041F7B2   85F6             TEST ESI,ESI                       ; 判断用户名是否为空 ESI

0041F7B4   8975 F8          MOV DWORD PTR SS:[EBP-8],ESI       ; [EBP-8]=0012E430=0A 00 00 00

0041F7B7   75 08            JNZ SHORT FTBirthd.0041F7C1        ; 不为零则跳，为空 OVER

0041F7B9   8B45 10          MOV EAX,DWORD PTR SS:[EBP+10]

0041F7BC   8020 00          AND BYTE PTR DS:[EAX],0

0041F7BF   EB 4E            JMP SHORT FTBirthd.0041F80F

0041F7C1   57               PUSH EDI                           ; 一切正常,就到这里

0041F7C2   8B7D 10          MOV EDI,DWORD PTR SS:[EBP+10]

0041F7C5   FF75 0C          PUSH DWORD PTR SS:[EBP+C]

0041F7C8   57               PUSH EDI

0041F7C9   FF15 40114000    CALL DWORD PTR DS:[<&KERNEL32.ls]; 把"_r <@~=nj/2[l5,^"复制到 EAX

0041F7CF   3BF3             CMP ESI,EBX                        ; 判断用户名长度是否大于注册码

0041F7D1   8975 0C          MOV DWORD PTR SS:[EBP+C],ESI       ; [EBP+C]=0A

0041F7D4   7F 03            JG SHORT FTBirthd.0041F7D9         ; 大于则跳 HORT

0041F7D6   895D 0C          MOV DWORD PTR SS:[EBP+C],EBX       ; [EBP+C]=10

0041F7D9   33F6             XOR ESI,ESI                        ; ESI 清零

0041F7DB   3975 0C          CMP DWORD PTR SS:[EBP+C],ESI       ; 判断有没有输入注册码

0041F7DE   7E 2C            JLE SHORT FTBirthd.0041F80C

0041F7E0   8BC6             MOV EAX,ESI                        ; 算法循环开始,EAX 初值为 0

0041F7E2   6A 19            PUSH 19

0041F7E4   99               CDQ                                ; EDX 置零

0041F7E5   F77D FC          IDIV DWORD PTR SS:[EBP-4]          ; EAX idiv 注册码长度(EBP-4=10),商放入 EAX,余数放入 EDX

0041F7E8   8BC6             MOV EAX,ESI

0041F7EA   5B               POP EBX

0041F7EB   8D0C3A           LEA ECX,DWORD PTR DS:[EDX+EDI]     ; 字符串第[余数+1]位开始给 ECX

0041F7EE   99               CDQ                                ; EDX 置零

0041F7EF   F77D F8          IDIV DWORD PTR SS:[EBP-8]          ; EAX idiv 用户名长度(EBP-8=A),商放入 EAX,余数放入 EDX

0041F7F2   8B45 08          MOV EAX,DWORD PTR SS:[EBP+8]       ; "FTBirthday"给 EAXR

0041F7F5   0FB60402         MOVZX EAX,BYTE PTR DS:[EDX+EAX]    ; 取用户名的第[余数+1]位

0041F7F9   0FB611           MOVZX EDX,BYTE PTR DS:[ECX]        ; 取 ECX 处字符串的第一位

0041F7FC   33C2             XOR EAX,EDX                        ; 异或，结果放入 EAX

0041F7FE   99               CDQ                                ; EDX 置零

0041F7FF   F7FB             IDIV EBX                           ; 异或结果 idiv19,商放入 EAX,余数放入 EDX

0041F801   80C2 41          ADD DL,41                          ; 余数＋41

0041F804   46               INC ESI                            ; 循环指针加 1

0041F805   3B75 0C          CMP ESI,DWORD PTR SS:[EBP+C]       ; 比较循环是否结束,[EBP+C]=0012E444="10 00 00 00"

0041F808   8811             MOV BYTE PTR DS:[ECX],DL           ; DL 即为注册码的下一位

0041F80A   7C D4            JL SHORT FTBirthd.0041F7E0         ; 小于则跳回继续

0041F80C   8BC7             MOV EAX,EDI                        ; 结束后把最终结果放入 EAX

0041F80E   5F               POP EDI

0041F80F   5E               POP ESI

0041F810   5B               POP EBX

0041F811   C9               LEAVE

0041F812   C3               RETN
```

## 模拟算法验证

用户名 `"FTBirthday"` 即 `"46 54 42 69 72 74 68 64 61 79"` = `A[i]`

字符串 `"_r <@~=nj/2[l5,^"` 即 `"5F 72 20 3C 40 7E 3D 6E 6A 2F 32 5B 31 35 2C 5E"` = `B[i]`

初始 `ESI=0` `EAX=0`

### 注册码生成规则

**注册码第一位**

`{A[0 idiv A 的余数] xor B[0 idiv 10 的余数]}idiv 19 的余数＋41`

`＝{A[0] xor B[0]}idiv 19 的余数＋41`

`＝{46 xor 5F}idiv 19 的余数＋41`

`＝19 idiv 19 的余数＋41=0+41=41="A"`

**注册码第二位**

`{A[1 idiv A 的余数] xor B[1 idiv 10 的余数]}idiv 19 的余数＋41`

`＝{A[1] xor B[1]}idiv 19 的余数＋41`

`＝{54 xor 72}idiv 19 的余数＋41`

`＝26 idiv 19 的余数＋41=D+41=4E="N"`

**注册码第三位**

`{A[2 idiv A 的余数] xor B[2 idiv 10 的余数]}idiv 19 的余数＋41`

`＝{A[2] xor B[2]}idiv 19 的余数＋41`

`＝{42 xor 20}idiv 19 的余数＋41`

`＝62 idiv 19 的余数＋41=17+41=58="X"`

**注册码第四位**

`{A[3 idiv A 的余数] xor B[3 idiv 10 的余数]}idiv 19 的余数＋41`

`＝{A[3] xor B[3]}idiv 19 的余数＋41`

`＝{69 xor 3C}idiv 19 的余数＋41`

`＝55 idiv 19 的余数＋41=A+41=4B="K"`

………………

**注册码第十一位**

`{A[A idiv A 的余数] xor B[A idiv 10 的余数]}idiv 19 的余数＋41`

`＝{A[0] xor B[A]}idiv 19 的余数＋41`

`＝{46 xor 32}idiv 19 的余数＋41`

`＝74 idiv 19 的余数＋41=10+41=51="Q"`

………………

**注册码第十六位**

`{A[F idiv A 的余数] xor B[F idiv 10 的余数]}idiv 19 的余数＋41`

`＝{A[5] xor B[F]}idiv 19 的余数＋41`

`＝{74 xor 5E}idiv 19 的余数＋41`

`＝2A idiv 19 的余数＋41=11+41=52="R"`

### 结果

所以示例注册码为

用户名:`"FTBirthday"`

注册码:`"ANXK-AKKK-LLQP-VRTR"`

`4dd466d0e1e0cc123afbe504da2462ca`
