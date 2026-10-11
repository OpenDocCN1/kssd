
### 代码段分析 (6)

```
00404634  |. 5D             POP EBP  
00404635  |> 8BC7           MOV EAX,EDI                     //计算结果送 EAX。  
00404637  |. 5F             POP EDI  
00404638  |. C1E0 10        SHL EAX,10                      //EAX 左移 10h 位。  
0040463B  |. 5E             POP ESI  
0040463C  |. 03C1           ADD EAX,ECX                     //EAX=EAX+ECX。  
0040463E  |. 5B             POP EBX  
0040463F  \. C3             RETN  
```

#### 关键调用：`FIFA_200.004046C0`

下面分析 `0040490F CALL FIFA_200.004046C0` 的代码。这个地方关键！！（6）

```
004046C0  /$ 8B4424 04      MOV EAX,DWORD PTR SS:[ESP+4]    //目标地址送 EAX。  
004046C4  |. 8B4C24 08      MOV ECX,DWORD PTR SS:[ESP+8]     //ECX=前面 004048FE 处函数计算的结果。  
004046C8  |. 8AD1           MOV DL,CL                       //CL 送 DL。          
004046CA  |. 80E2 1F        AND DL,1F                       //DL 与 1F 做“与运算”  
004046CD  |. 8850 06        MOV BYTE PTR DS:[EAX+6],DL      //保存结果。  
004046D0  |. 8BD1           MOV EDX,ECX                     //ECX 送 EDX。  
004046D2  |. C1EA 05        SHR EDX,5                       //右移 5 次。  
004046D5  |. 80E2 1F        AND DL,1F                       //DL 与 1F 做“与运算”  
004046D8  |. 8850 05        MOV BYTE PTR DS:[EAX+5],DL      //保存结果。  
004046DB  |. 8BD1           MOV EDX,ECX                     //ECX 送 EDX。  
004046DD  |. C1EA 0A        SHR EDX,0A                      //右移 A 次。  
004046E0  |. 80E2 1F        AND DL,1F                       //DL 与 1F 做“与运算”  
004046E3  |. 8850 04        MOV BYTE PTR DS:[EAX+4],DL      //保存结果。  
004046E6  |. 8BD1           MOV EDX,ECX                     //ECX 送 EDX。  
004046E8  |. C1EA 0F        SHR EDX,0F                      //右移 F 次。  
004046EB  |. 80E2 1F        AND DL,1F                       //DL 与 1F 做“与运算”  
004046EE  |. 8850 03        MOV BYTE PTR DS:[EAX+3],DL      //保存结果。  
004046F1  |. 8BD1           MOV EDX,ECX                     //ECX 送 EDX。  
004046F3  |. C1EA 14        SHR EDX,14                      //右移 14h 次。  
004046F6  |. 80E2 1F        AND DL,1F                       //DL 与 1F 做“与运算”  
004046F9  |. 8850 02        MOV BYTE PTR DS:[EAX+2],DL      //保存结果。  
004046FC  |. 8BD1           MOV EDX,ECX                     //ECX 送 EDX。  
004046FE  |. C1E9 1E        SHR ECX,1E                      //ECX 右移 1Eh 次。  
00404701  |. C1EA 19        SHR EDX,19                      //EDX 右移 19h 次。  
00404704  |. 80E1 1F        AND CL,1F                       //DL 与 1F 做“与运算”  
00404707  |. 8808           MOV BYTE PTR DS:[EAX],CL        //保存结果。  
00404709  |. 80E2 1F        AND DL,1F                       //DL 与 1F 做“与运算”  
0040470C  |. 8850 01        MOV BYTE PTR DS:[EAX+1],DL      //保存结果。  
0040470F  |. 8A48 06        MOV CL,BYTE PTR DS:[EAX+6]      //取出第 6 位送 CL。  
00404712  |. 8A10           MOV DL,BYTE PTR DS:[EAX]        //取出第 0 位送 DL。  
00404714  |. 80E1 07        AND CL,7                        //与 7 相与。  
00404717  |. C0E1 02        SHL CL,2                        //左移 2 位。  
0040471A  |. 0AD1           OR DL,CL                        //DL 与 CL 或运算。  
0040471C  |. 8810           MOV BYTE PTR DS:[EAX],DL        //保存结果。  
0040471E  |. 0FBED2         MOVSX EDX,DL                    //把 DL 送 EDX。  
00404721  |. 8A8A B8BC4200  MOV CL,BYTE PTR DS:[EDX+42BCB8] //查表得一数送 CL。  
00404727  |. 0FBE50 01      MOVSX EDX,BYTE PTR DS:[EAX+1]   //把第 1 位送 EDX。  
0040472B  |. 8808           MOV BYTE PTR DS:[EAX],CL        //保存前面查表结果。  
0040472D  |. 8A8A B8BC4200  MOV CL,BYTE PTR DS:[EDX+42BCB8] //查表得一数送 CL。  
00404733  |. 0FBE50 02      MOVSX EDX,BYTE PTR DS:[EAX+2]   //把第 2 位送 EDX。  
00404737  |. 8848 01        MOV BYTE PTR DS:[EAX+1],CL      //保存前面查表结果。  
0040473A  |. 8A8A B8BC4200  MOV CL,BYTE PTR DS:[EDX+42BCB8] //查表得一数送 CL。  
00404740  |. 0FBE50 03      MOVSX EDX,BYTE PTR DS:[EAX+3]   //把第 3 位送 EDX。  
00404744  |. 8848 02        MOV BYTE PTR DS:[EAX+2],CL      //保存前面查表结果。  
00404747  |. 8A8A B8BC4200  MOV CL,BYTE PTR DS:[EDX+42BCB8] //查表得一数送 CL。  
0040474D  |. 0FBE50 04      MOVSX EDX,BYTE PTR DS:[EAX+4]   //把第 4 位送 EDX。  
00404751  |. 8848 03        MOV BYTE PTR DS:[EAX+3],CL      //保存前面查表结果。  
00404754  |. 8A8A B8BC4200  MOV CL,BYTE PTR DS:[EDX+42BCB8] //查表得一数送 CL。  
0040475A  |. 0FBE50 05      MOVSX EDX,BYTE PTR DS:[EAX+5]   //把第 5 位送 EDX。  
0040475E  |. 8848 04        MOV BYTE PTR DS:[EAX+4],CL      //保存前面查表结果。  
00404761  |. 8A8A B8BC4200  MOV CL,BYTE PTR DS:[EDX+42BCB8] //查表得一数送 CL。  
00404767  |. 0FBE50 06      MOVSX EDX,BYTE PTR DS:[EAX+6]   //把第 6 位送 EDX。  
0040476B  |. 8848 05        MOV BYTE PTR DS:[EAX+5],CL      //保存前面查表结果。  
0040476E  |. 8A8A B8BC4200  MOV CL,BYTE PTR DS:[EDX+42BCB8] //查表得一数送 CL。  
00404774  |. 8848 06        MOV BYTE PTR DS:[EAX+6],CL      //保存前面查表结果。  
00404777  |. C640 07 00     MOV BYTE PTR DS:[EAX+7],0       //最后一位置 0。  
0040477B  \. C3             RETN  
```

该函数做完上述工作，得到一个 7 位的字符串。记为 `S1=2K59X2T`。

### 代码段分析 (7)

下面是 `004049F4 CALL FIFA_200.00404780` 代码，跟上面这部分类似。（7）

```
00404780  /$ 8B4424 04      MOV EAX,DWORD PTR SS:[ESP+4]     //S1 串地址送 EAX。  
00404784  |. 8B4C24 08      MOV ECX,DWORD PTR SS:[ESP+8]     //4049EE 处的结果送 ECX。  
00404788  |. 8AD1           MOV DL,CL  
0040478A  |. 80E2 03        AND DL,3  
0040478D  |. C0E2 03        SHL DL,3  
00404790  |. 8850 06        MOV BYTE PTR DS:[EAX+6],DL      //下面总共计算出 7 个 DL 值依次覆盖原来数值。  
00404793  |. C1E9 02        SHR ECX,2  
00404796  |. 8AD1           MOV DL,CL  
00404798  |. 80E2 1F        AND DL,1F  
0040479B  |. 8850 05        MOV BYTE PTR DS:[EAX+5],DL  
0040479E  |. 8BD1           MOV EDX,ECX  
004047A0  |. C1EA 05        SHR EDX,5  
004047A3  |. 80E2 1F        AND DL,1F  
004047A6  |. 8850 04        MOV BYTE PTR DS:[EAX+4],DL  
004047A9  |. 8BD1           MOV EDX,ECX  
004047AB  |. C1EA 0A        SHR EDX,0A  
004047AE  |. 80E2 1F        AND DL,1F  
004047B1  |. 8850 03        MOV BYTE PTR DS:[EAX+3],DL  
004047B4  |. 8BD1           MOV EDX,ECX  
004047B6  |. C1EA 0F        SHR EDX,0F  
004047B9  |. 80E2 1F        AND DL,1F  
004047BC  |. 8850 02        MOV BYTE PTR DS:[EAX+2],DL  
004047BF  |. 8BD1           MOV EDX,ECX  
004047C1  |. C1EA 14        SHR EDX,14  
004047C4  |. C1E9 19        SHR ECX,19  
004047C7  |. 80E2 1F        AND DL,1F  
004047CA  |. 8850 01        MOV BYTE PTR DS:[EAX+1],DL  
004047CD  |. 80E1 1F        AND CL,1F  
004047D0  |. 8808           MOV BYTE PTR DS:[EAX],CL  
004047D2  |. 8A50 06        MOV DL,BYTE PTR DS:[EAX+6]  
004047D5  |. 80E1 07        AND CL,7  
004047D8  |. 0AD1           OR DL,CL  
004047DA  |. 8850 06        MOV BYTE PTR DS:[EAX+6],DL     
004047DD  |. 0FBE10         MOVSX EDX,BYTE PTR DS:[EAX]     //下面代码依次取出上面刚刚得到的 7 位数值送 EDX。  
004047E0  |. 8A8A 98BC4200  MOV CL,BYTE PTR DS:[EDX+42BC98] //用 EDX 作为地址偏移查表，结果送 CL。  
004047E6  |. 0FBE50 01      MOVSX EDX,BYTE PTR DS:[EAX+1]  
004047EA  |. 8808           MOV BYTE PTR DS:[EAX],CL         //保存 CL。  
004047EC  |. 8A8A 98BC4200  MOV CL,BYTE PTR DS:[EDX+42BC98]  
004047F2  |. 0FBE50 02      MOVSX EDX,BYTE PTR DS:[EAX+2]  
004047F6  |. 8848 01        MOV BYTE PTR DS:[EAX+1],CL  
004047F9  |. 8A8A 98BC4200  MOV CL,BYTE PTR DS:[EDX+42BC98]  
004047FF  |. 0FBE50 03      MOVSX EDX,BYTE PTR DS:[EAX+3]  
00404803  |. 8848 02        MOV BYTE PTR DS:[EAX+2],CL  
00404806  |. 8A8A 98BC4200  MOV CL,BYTE PTR DS:[EDX+42BC98]  
0040480C  |. 0FBE50 04      MOVSX EDX,BYTE PTR DS:[EAX+4]  
00404810  |. 8848 03        MOV BYTE PTR DS:[EAX+3],CL  
00404813  |. 8A8A 98BC4200  MOV CL,BYTE PTR DS:[EDX+42BC98]  
00404819  |. 0FBE50 05      MOVSX EDX,BYTE PTR DS:[EAX+5]  
```
