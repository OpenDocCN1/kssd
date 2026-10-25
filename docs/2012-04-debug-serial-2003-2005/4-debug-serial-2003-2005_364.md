
# 公务员考试记忆指南(全国版)3.0

> 原文：[`www.pediy.com/kssd/pediy06/pediy6079.htm`](https://www.pediy.com/kssd/pediy06/pediy6079.htm)

对象：公务员考试记忆指南(全国版)3.0  
作者：`lordor[BCG]`  
目的：属技术交流，无其它目的，请不要任意散布或用用商业用途。初学破解,如有不对的地方欢迎批评指出。  
工具：`softice`, `w32Dasm`, `ollydbg`

### 试炼码

- 机器码：`{264443CE-814B-4915-B8DF-591BF3FF27A8}`
- 注册码：`654321`

### 汇编代码

```
0053BAE6     8BC0            MOV EAX,EAX
0053BAE8   .  A1 10545400    MOV EAX,DWORD PTR DS:[545410]
0053BAED   .  8B00           MOV EAX,DWORD PTR DS:[EAX]
0053BAEF   .  E8 C4210000    CALL 公务员考.0053DCB8 DS:[  ====》关键 call，跟入
0053BAF4   .  84C0           TEST AL,AL                  ====》成功标志
0053BAF6   .  74 4E          JE SHORT 公务员考.0053BB46
0053BAF8   .  A1 10545400    MOV EAX,DWORD PTR DS:[545410]
```

### 关键 Call 分析

```
0053DCD0  |.  64:8920        MOV DWORD PTR FS:[EAX],ESP
0053DCD3  |.  33DB           XOR EBX,EBX
0053DCD5  |.  8D55 F8        LEA EDX,DWORD PTR SS:[EBP-8]
0053DCD8  |.  A1 08505400    MOV EAX,DWORD PTR DS:[545008]
0053DCDD  |.  8B00           MOV EAX,DWORD PTR DS:[EAX]
0053DCDF  |.  8B80 FC020000  MOV EAX,DWORD PTR DS:[EAX+2FC]
0053DCE5  |.  E8 CAC9F2FF    CALL 公务员考.0046A6B4
0053DCEA  |.  8B45 F8        MOV EAX,DWORD PTR SS:[EBP-8]         ;  注册码入 eax
0053DCED  |.  E8 A26CECFF    CALL 公务员考.00404994
0053DCF2  |.  85C0           TEST EAX,EAX
0053DCF4  |.  75 09          JNZ SHORT 公务员考.0053DCFF
0053DCF6  |.  C645 FF 00     MOV BYTE PTR SS:[EBP-1],0
0053DCFA  |.  E9 94000000    JMP 公务员考.0053DD93
0053DCFF  |>  8BF0           MOV ESI,EAX
0053DD01  |.  85F6           TEST ESI,ESI
0053DD03  |.  7E 42          JLE SHORT 公务员考.0053DD47
0053DD05  |.  C745 F4 01000> MOV DWORD PTR SS:[EBP-C],1
0053DD0C  |>  8D45 EC        /LEA EAX,DWORD PTR SS:[EBP-14]
0053DD0F  |.  50             |PUSH EAX
0053DD10  |.  B9 01000000    |MOV ECX,1
0053DD15  |.  8B55 F4        |MOV EDX,DWORD PTR SS:[EBP-C]
0053DD18  |.  8B45 F8        |MOV EAX,DWORD PTR SS:[EBP-8]           ;  注册码入 eax
0053DD1B  |.  E8 D46EECFF    |CALL 公务员考.00404BF4                 ;  取 1 位注册码
0053DD20  |.  8B45 EC        |MOV EAX,DWORD PTR SS:[EBP-14]
0053DD23  |.  E8 6C6EECFF    |CALL 公务员考.00404B94
0053DD28  |.  8A00           |MOV AL,BYTE PTR DS:[EAX]
0053DD2A  |.  25 FF000000    |AND EAX,0FF                             ;  以上为取一位注册码，十六进制值入 eax
0053DD2F  |.  69C0 6442FB8C  |IMUL EAX,EAX,8CFB4264                   ;  eax=eax*8CFB4264，带符号相乘，产生 8 个字节的数，只取 0－4 字节的数
0053DD35  |.  03D8           |ADD EBX,EAX                             ;  然后加起来
0053DD37  |.  8D55 F0        |LEA EDX,DWORD PTR SS:[EBP-10]
0053DD3A  |.  8BC3           |MOV EAX,EBX
0053DD3C  |.  E8 43B5ECFF    |CALL 公务员考.00409284                 ;  转换为十进制
0053DD41  |.  FF45 F4        |INC DWORD PTR SS:[EBP-C]               ;  加 1
0053DD44  |.  4E             |DEC ESI
0053DD45  |.^ 75 C5          \JNZ SHORT 公务员考.0053DD0C
0053DD47  |>  8D45 F0        LEA EAX,DWORD PTR SS:[EBP-10]
0053DD4A  |.  BA D8DD5300    MOV EDX,公务员考.0053DDD8               ;  字母 A 入 edx
0053DD4F  |.  E8 486CECFF    CALL 公务员考.0040499C                  ;  在刚才生成的数字后追加 1 位字母 A,此即为真码
0053DD54  |.  8D55 E8        LEA EDX,DWORD PTR SS:[EBP-18]
0053DD57  |.  A1 08505400    MOV EAX,DWORD PTR DS:[545008]
0053DD5C  |.  8B00           MOV EAX,DWORD PTR DS:[EAX]
0053DD5E  |.  8B80 00030000  MOV EAX,DWORD PTR DS:[EAX+300]
0053DD64  |.  E8 4BC9F2FF    CALL 公务员考.0046A6B4                  ;  取得输入的注册码
0053DD69  |.  8B45 E8        MOV EAX,DWORD PTR SS:[EBP-18]           ;  注册码入 eax
0053DD6C  |.  8B55 F0        MOV EDX,DWORD PTR SS:[EBP-10]           ;  真码入 edx
0053DD6F  |.  E8 6C6DECFF    CALL 公务员考.00404AE0                  ;  关键比较
0053DD74  |.  75 19          JNZ SHORT 公务员考.0053DD8F             ;  不等跳走
0053DD76  |.  C645 FF 01     MOV BYTE PTR SS:[EBP-1],1
0053DD7A  |.  B8 246E5400    MOV EAX,公务员考.00546E24
0053DD7F  |.  8B55 F8        MOV EDX,DWORD PTR SS:[EBP-8]
0053DD82  |.  E8 A169ECFF    CALL 公务员考.00404728
```

### 总结

机器码从第 1 位开始，各位十六进制值与 `8CFB4264`（H）进行带符号相乘，得 8 个字节的数（即为 64 位），只取低 32 位，然后取得的数相加起来（只取 32 位），如此重复，直到最后 1 位的机器码。然后把所得的相加数转换为十进制数，再在后面加上 `A` 就是注册码。

- 机器码：`{264443CE-814B-4915-B8DF-591BF3FF27A8}`
- 注册码：`717393156A`

`lordor[BCG]`
`4dd466d0e1e0cc123afbe504da2462ca`
