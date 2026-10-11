
## 汇编代码分析

```assembly
005E9D99   |.^ 75 D9          \JNZ SHORT Contacts.005E9D74             ; 上面这个循环的作用就是把字符串倒序  

005E9D9B   |>  8D45 F8        LEA EAX,DWORD PTR SS:\[EBP-8\]  

005E9D9E   |.  50            PUSH EAX  

005E9D9F   |.  B9 04000000   MOV ECX,4                                  ; 使 ECX=4  

005E9DA4   |.  BA 01000000   MOV EDX,1                                  ; 使 EDX=1  

005E9DA9   |.  8B45 F4       MOV EAX,DWORD PTR SS:\[EBP-C\]             ; EBP-C=倒序后的字符串，设为字符串 A  

005E9DAC   |.  E8 47B1E1FF   CALL Contacts.00404EF8                    ; 取字符串 A 前 4 位  

005E9DB1   |.  8D45 F4       LEA EAX,DWORD PTR SS:\[EBP-C\]  

005E9DB4   |.  50            PUSH EAX  

005E9DB5   |.  B9 04000000   MOV ECX,4                                  ; 使 ECX=4  

005E9DBA   |.  BA 05000000   MOV EDX,5                                  ; 使 EDX=5  

005E9DBF   |.  8B45 F4       MOV EAX,DWORD PTR SS:\[EBP-C\]             ; EBP-C=倒序后的字符串  

005E9DC2   |.  E8 31B1E1FF   CALL Contacts.00404EF8                    ; 取字符串 A 的第 5 位到第八位  

005E9DC7   |.  8B45 F8       MOV EAX,DWORD PTR SS:\[EBP-8\]  

005E9DCA   |.  E8 C9AEE1FF   CALL Contacts.00404C98  

005E9DCF   |.  83F8 04       CMP EAX,4                                  ; 比较所得新的字符串是否为 4 位  

005E9DD2   |.  7D 2F         JGE SHORT Contacts.005E9E03               ; 大于四位就跳走直接进行运算  

005E9DD4   |.  8B45 F8       MOV EAX,DWORD PTR SS:\[EBP-8\]  

005E9DD7   |.  E8 BCAEE1FF   CALL Contacts.00404C98  

005E9DDC   |.  8BD8          MOV EBX,EAX  

005E9DDE   |.  83FB 03       CMP EBX,3                                  ; 比较新字符串是否为 3 位  

005E9DE1   |.  7F 20         JG SHORT Contacts.005E9E03  

005E9DE3   |>  8D4D E4       /LEA ECX,DWORD PTR SS:\[EBP-1C\]  

005E9DE6   |.  8BC3          |MOV EAX,EBX                                ; EBX=新字符串长度  

005E9DE8   |.  C1E0 02       |SHL EAX,2                                  ; EAX=EBX 乘以 2 的平方  

005E9DEB   |.  33D2          |XOR EDX,EDX                                ; EDX 清零  

005E9DED   |.  E8 6202E2FF   |CALL Contacts.0040A054  

005E9DF2   |.  8B55 E4       |MOV EDX,DWORD PTR SS:\[EBP-1C\]  

005E9DF5   |.  8D45 F8       |LEA EAX,DWORD PTR SS:\[EBP-8\]  

005E9DF8   |.  E8 A3AEE1FF   |CALL Contacts.00404CA0  

005E9DFD   |.  43            |INC EBX                                     ; 每计算一次 EBX+1  

005E9DFE   |.  83FB 04       |CMP EBX,4                                     ; 比较 EBX 是否等于 4  

005E9E01   |.^ 75 E0         \JNZ SHORT Contacts.005E9DE3               ; 小于就继续计算  

005E9E03   |>  8B45 F4       MOV EAX,DWORD PTR SS:\[EBP-C\]             ; 字符串 A 与计算结果合并  

005E9E06   |.  E8 8DAEE1FF   CALL Contacts.00404C98  

005E9E0B   |.  83F8 04       CMP EAX,4  

005E9E0E   |.  7D 2F         JGE SHORT <Contacts.输出计算结果>  

005E9E10   |.  8B45 F4       MOV EAX,DWORD PTR SS:\[EBP-C\]  

005E9E13   |.  E8 80AEE1FF   CALL Contacts.00404C98  

005E9E18   |.  8BD8          MOV EBX,EAX  

005E9E1A   |.  83FB 03       CMP EBX,3  

005E9E1D   |.  7F 20         JG SHORT <Contacts.输出计算结果>  

005E9E1F   |>  8D4D E0       /LEA ECX,DWORD PTR SS:\[EBP-20\]  

005E9E22   |.  8BC3          |MOV EAX,EBX                                 ; 使 EAX=EBX  

005E9E24   |.  C1E0 02       |SHL EAX,2                                  ; EAX=EBX 乘以 2 的平方  

005E9E27   |.  33D2          |XOR EDX,EDX  

005E9E29   |.  E8 2602E2FF   |CALL Contacts.0040A054  

005E9E2E   |.  8B55 E0       |MOV EDX,DWORD PTR SS:\[EBP-20\]  

005E9E31   |.  8D45 F4       |LEA EAX,DWORD PTR SS:\[EBP-C\]  

005E9E34   |.  E8 67AEE1FF   |CALL Contacts.00404CA0  

005E9E39   |.  43            |INC EBX                                     ; 每计算一次 EBX+1  

005E9E3A   |.  83FB 04       |CMP EBX,4                                     ; 比较 EBX 是否等于 4  

005E9E3D   |.^ 75 E0         \JNZ SHORT Contacts.005E9E1F               ; 小于就继续计算  

005E9E3F   |>|>  8D45 F0     LEA EAX,DWORD PTR SS:\[EBP-10\]           ; 输出计算结果  

005E9E42   |.  BA CC9E5E00   MOV EDX,Contacts.005E9ECC                ; con62bsar698  

005E9E47   |.  E8 24ACE1FF   CALL Contacts.00404A70                   ; 固定字符串 Con62bsae698  

005E9E4C   |.  8D45 DC       LEA EAX,DWORD PTR SS:\[EBP-24\]  

005E9E4F   |.  50            PUSH EAX  

005E9E50   |.  B9 04000000   MOV ECX,4                                  ; 使 ECX=4  

005E9E55   |.  BA 01000000   MOV EDX,1                                  ; EDX=1  

005E9E5A   |.  8B45 F0       MOV EAX,DWORD PTR SS:\[EBP-10\]  

005E9E5D   |.  E8 96B0E1FF   CALL Contacts.00404EF8                   ; 取固定字符串的前四位  

005E9E62   |.  FF75 DC       PUSH DWORD PTR SS:\[EBP-24\]  

005E9E65   |.  68 E49E5E00   PUSH Contacts.005E9EE4                   ; -  

005E9E6A   |.  FF75 F8       PUSH DWORD PTR SS:\[EBP-8\]  

005E9E6D   |.  8D45 D8       LEA EAX,DWORD PTR SS:\[EBP-28\]  

005E9E70   |.  50            PUSH EAX  

005E9E71   |.  B9 05000000   MOV ECX,5                                  ; 使 ECX=5  

005E9E76   |.  BA 05000000   MOV EDX,5                                  ; 使 EDX=5  

005E9E7B   |.  8B45 F0       MOV EAX,DWORD PTR SS:\[EBP-10\]  

005E9E7E   |.  E8 75B0E1FF   CALL Contacts.00404EF8  

005E9E83   |.  FF75 D8       PUSH DWORD PTR SS:\[EBP-28\]              ; 取固定字符串的 5 到 9 位  

005E9E86   |.  68 E49E5E00   PUSH Contacts.005E9EE4                   ; -  

省略一些代码。。。  

005E9EBC    .  5F            POP EDI  

005E9EBD    .  5E            POP ESI  

005E9EBE    .  5B            POP EBX  

005E9EBF    .  8BE5         MOV ESP,EBP  

005E9EC1    .  5D            POP EBP                                   ; 合并注册码并出栈  

005E9EC2    .  C3            RETN  
```

------------------------------------------------------------------------

## 算法总结

1. 取注册名每一位的 ASCII 值联成一个字符串。
2. 把所得到的字符串倒序。
3. 取字符串的前四位和 5 到 8 位，与固定字符串的 `con62bsar698` 的前四位和 5 到 9 位交叉组成注册码。

### 特殊情况处理

注意：如果注册名每一位的 ASCII 值联成的字符串小于 4 位或小于 8 位，就会进一步处理。

#### 小于 8 位的情况

- `0 * 2^2`
- `1 * 2^2`
- `2 * 2^2`
- `3 * 2^2`

得到一个四位的字符串。

#### 小于 4 位的情况（即输入的注册名只有一位）

1. 注册名的 ASCII 值倒序的结果与 `2 * 2^2`、`3 * 2^2` 所得的两个结果相合并，得到第一个四位字符串。
2. 再重复“小于 8 位”时的处理方法，得到第二个字符串。
3. 最后与固定字符串的 `con62bsar698` 交叉得到注册码。

### 示例

**注册名：** `tcxb`  
**注册码：** `Con6-26872bsar-3647`  

`4dd466d0e1e0cc123afbe504da2462ca`
