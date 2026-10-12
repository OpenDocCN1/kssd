
```
005319EC  |.  BA D81A5300    MOV EDX,糖尿病康.00531AD8  

005319F1  |.  E8 2206F8FF    CALL 糖尿病康.004B2018  

005319F6  |.  BA E41A5300    MOV EDX,糖尿病康.00531AE4                  ;  ASCII "11"  

005319FB  |.  8B08           MOV ECX,DWORD PTR DS:[EAX]  

005319FD  |.  FF91 B0000000  CALL DWORD PTR DS:[ECX+B0]  

00531A03  |.  A1 54525600    MOV EAX,DWORD PTR DS:[565254]  

00531A08  |.  8B00           MOV EAX,DWORD PTR DS:[EAX]  

00531A0A  |.  8B80 F8020000  MOV EAX,DWORD PTR DS:[EAX+2F8]  

00531A10  |.  8B10           MOV EDX,DWORD PTR DS:[EAX]  

00531A12  |.  FF92 4C020000  CALL DWORD PTR DS:[EDX+24C]  

00531A18  |.  B8 F01A5300    MOV EAX,糖尿病康.00531AF0  

00531A1D  |.  E8 B66CF0FF    CALL 糖尿病康.004386D8  

00531A22  |.  EB 0A          JMP SHORT 糖尿病康.00531A2E  

00531A24  |>  B8 141B5300    MOV EAX,糖尿病康.00531B14  

00531A29  |.  E8 AA6CF0FF    CALL 糖尿病康.004386D8  

00531A2E  |>  33C0           XOR EAX,EAX  

00531A30  |.  5A             POP EDX  

00531A31  |.  59             POP ECX  

00531A32  |.  59             POP ECX  

00531A33  |.  64:8910        MOV DWORD PTR FS:[EAX],EDX  

00531A36  |.  68 681A5300    PUSH 糖尿病康.00531A68  

00531A3B  |>  8D45 DC        LEA EAX,DWORD PTR SS:[EBP-24]  

00531A3E  |.  E8 A128EDFF    CALL 糖尿病康.004042E4  

00531A43  |.  8D45 E0        LEA EAX,DWORD PTR SS:[EBP-20]  

00531A46  |.  E8 9928EDFF    CALL 糖尿病康.004042E4  

00531A4B  |.  8D45 E4        LEA EAX,DWORD PTR SS:[EBP-1C]  

00531A4E  |.  E8 9128EDFF    CALL 糖尿病康.004042E4  

00531A53  |.  8D45 E8        LEA EAX,DWORD PTR SS:[EBP-18]  

00531A56  |.  BA 06000000    MOV EDX,6  

00531A5B  |.  E8 A828EDFF    CALL 糖尿病康.00404308  

00531A60  \.  C3             RETN  

00531A61   .^ E9 8222EDFF    JMP 糖尿病康.00403CE8  

00531A66   .^ EB D3          JMP SHORT 糖尿病康.00531A3B  

00531A68   .  5E             POP ESI  

00531A69   .  5B             POP EBX  

00531A6A   .  8BE5           MOV ESP,EBP  

00531A6C   .  5D             POP EBP  

00531A6D   .  C3             RETN
```

### 算法小结

1. 机器码从第三位开始取. 参与注册码运算
2. 机器码加 `13217D3` 再减去 `.12CEAC5` 记做为 `A` 得出的 `A` 乘以 `4`，再加上 `A`，再转换成 `10` 进制数，记做为 `B`
3. 取 `B` 的前三位与固定数 `500` 比较. 小于等于就与字符串 `"KH"` 相连. 反之就与 `"Ly"` 相连. 记做为 `C`
4. 取 `B` 的后三位与固定数 `500` 比较. 小于等于就与字符串 `"RD"` 相连. 反之就与 `"DJ"` 相连. 记做为 `D`
5. `C` 的结果与 `D` 的结果相连. 就是软件的注册码!!

### 破解总结

软件算法比较简单. 适合新手练习.. 我也是莱鸟，算法与总结都感觉写的不太好! 希望大家支持!

### 版权声明

本文纯属技术交流，转载请注明作者信息并保持文章的完整，谢谢!

`4dd466d0e1e0cc123afbe504da2462ca`
