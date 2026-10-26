
```
004AE85C  |> 8D45 F4        LEA EAX,DWORD PTR SS:[EBP-C]
004AE85F  |. BA 03000000    MOV EDX,3
004AE864  |. E8 BB54F5FF    CALL CACHEMAN.00403D24
004AE869  \. C3             RETN
```

---

### 下面分析 `004AE7ED CALL CACHEMAN.0040426C`（**）

```
0040426C  /$ 85C0           TEST EAX,EAX
0040426E  |. 74 40          JE SHORT CACHEMAN.004042B0
00404270  |. 85D2           TEST EDX,EDX
00404272  |. 74 31          JE SHORT CACHEMAN.004042A5
00404274  |. 53             PUSH EBX
00404275  |. 56             PUSH ESI
00404276  |. 57             PUSH EDI
00404277  |. 89C6           MOV ESI,EAX
00404279  |. 89D7           MOV EDI,EDX
0040427B  |. 8B4F FC        MOV ECX,DWORD PTR DS:[EDI-4]
0040427E  |. 57             PUSH EDI
0040427F  |. 8B56 FC        MOV EDX,DWORD PTR DS:[ESI-4]
00404282  |. 4A             DEC EDX
00404283  |. 78 1B          JS SHORT CACHEMAN.004042A0
00404285  |. 8A06           MOV AL,BYTE PTR DS:[ESI]
00404287  |. 46             INC ESI
00404288  |. 29D1           SUB ECX,EDX
0040428A  |. 7E 14          JLE SHORT CACHEMAN.004042A0
0040428C  |> F2:AE          /REPNE SCAS BYTE PTR ES:[EDI]   //用下划线字符依次与ID码字符比较。
0040428E  |. 75 10          |JNZ SHORT CACHEMAN.004042A0
00404290  |. 89CB           |MOV EBX,ECX
00404292  |. 56             |PUSH ESI
00404293  |. 57             |PUSH EDI
00404294  |. 89D1           |MOV ECX,EDX
00404296  |. F3:A6          |REPE CMPS BYTE PTR ES:[EDI],BYTE PTR DS:[ESI]
00404298  |. 5F             |POP EDI
00404299  |. 5E             |POP ESI
0040429A  |. 74 0C          |JE SHORT CACHEMAN.004042A8    //如果存在下划线则OK。
0040429C  |. 89D9           |MOV ECX,EBX
0040429E  |.^EB EC          \JMP SHORT CACHEMAN.0040428C
004042A0  |> 5A             POP EDX
004042A1  |. 31C0           XOR EAX,EAX
004042A3  |. EB 08          JMP SHORT CACHEMAN.004042AD
004042A5  |> 31C0           XOR EAX,EAX
004042A7  |. C3             RETN
004042A8  |> 5A             POP EDX
004042A9  |. 89F8           MOV EAX,EDI
004042AB  |. 29D0           SUB EAX,EDX                    //下划线字符地址与首字符地址相减得子串长度。即
包含下划线字符在内字符数。返回到004AE7F2地址处与4比较，所以这里一定让第四位是下划线。否则返回值不匹配。
004042AD  |> 5F             POP EDI
004042AE  |. 5E             POP ESI
004042AF  |. 5B             POP EBX
004042B0  \> C3             RETN
```

---

### 下面分析 `004AE819 CALL CACHEMAN.00408C8C`

这个函数处理前3位ID码。（***）

```
00408C8C  /$ 53             PUSH EBX
00408C8D  |. 51             PUSH ECX
00408C8E  |. 8BDA           MOV EBX,EDX                   //EDX=0，并给EBX赋值。
00408C90  |. 8BD4           MOV EDX,ESP
00408C92  |. E8 2D9FFFFF    CALL CACHEMAN.00402BC4        //我们跟入这个函数看看。（****）
00408C97  |. 833C24 00      CMP DWORD PTR SS:[ESP],0      //这里[ESP]内返回0。
00408C9B  |. 74 02          JE SHORT CACHEMAN.00408C9F    //这里则跳。
00408C9D  |. 8BC3           MOV EAX,EBX     //否则下面EBX把EAX覆盖为0。
00408C9F  |> 5A             POP EDX
00408CA0  |. 5B             POP EBX
00408CA1  \. C3             RETN
```

---

### 下面是 `00408C92 CALL CACHEMAN.00402BC4` 代码分析（****）

```
00402BC4  /$ 53             PUSH EBX
00402BC5  |. 56             PUSH ESI
00402BC6  |. 57             PUSH EDI
00402BC7  |. 89C6           MOV ESI,EAX
00402BC9  |. 50             PUSH EAX
00402BCA  |. 85C0           TEST EAX,EAX
00402BCC  |. 74 73          JE SHORT CACHEMAN.00402C41
00402BCE  |. 31C0           XOR EAX,EAX
00402BD0  |. 31DB           XOR EBX,EBX
00402BD2  |. BF CCCCCC0C    MOV EDI,0CCCCCCC
00402BD7  |> 8A1E           /MOV BL,BYTE PTR DS:[ESI]    //先判断注册码是否为空格。
00402BD9  |. 46             |INC ESI
00402BDA  |. 80FB 20        |CMP BL,20
00402BDD  |.^74 F8          \JE SHORT CACHEMAN.00402BD7
00402BDF  |. B5 00          MOV CH,0
00402BE1  |. 80FB 2D        CMP BL,2D                   //判断是否为减号。
00402BE4  |. 74 69          JE SHORT CACHEMAN.00402C4F
00402BE6  |. 80FB 2B        CMP BL,2B                   //判断是否为加号。
00402BE9  |. 74 66          JE SHORT CACHEMAN.00402C51
00402BEB  |. 80FB 24        CMP BL,24                   //判断是否为$号。
00402BEE  |. 74 66          JE SHORT CACHEMAN.00402C56
00402BF0  |. 80FB 78        CMP BL,78                   //判断是否为x。
00402BF3  |. 74 61          JE SHORT CACHEMAN.00402C56
00402BF5  |. 80FB 58        CMP BL,58                   //判断是否为X。
00402BF8  |. 74 5C          JE SHORT CACHEMAN.00402C56
00402BFA  |. 80FB 30        CMP BL,30                   //判断是否为0。
00402BFD  |. 75 13          JNZ SHORT CACHEMAN.00402C12 //不为0则跳到下面处理。
\*略去几行\*
00402C12  |> 84DB           TEST BL,BL
00402C14  |. 74 34          JE SHORT CACHEMAN.00402C4A
00402C16  |> 80EB 30        /SUB BL,30                  //减去30h。
00402C19  |. 80FB 09        |CMP BL,9                   //与9比较。
00402C1C  |. 77 2C          |JA SHORT CACHEMAN.00402C4A //大于9则跳走。
00402C1E  |. 39F8           |CMP EAX,EDI
00402C20  |. 77 28          |JA SHORT CACHEMAN.00402C4A
00402C22  |. 8D0480         |LEA EAX,DWORD PTR DS:[EAX+EAX*4]  //EAX=5*EAX。
00402C25  |. 01C0           |ADD EAX,EAX                //EAX=2*EAX。
00402C27  |. 01D8           |ADD EAX,EBX                //把EBX累加到EAX。EBX内为转换为16进制的数。
00402C29  |. 8A1E           |MOV BL,BYTE PTR DS:[ESI]   //取下一个数。
00402C2B  |. 46             |INC ESI
00402C2C  |. 84DB           |TEST BL,BL
00402C2E  |.^75 E6          \JNZ SHORT CACHEMAN.00402C16  //不等于0继续循环。
00402C30  |> FECD           DEC CH
00402C32  |. 74 10          JE SHORT CACHEMAN.00402C44    //不跳。
00402C34  |. 85C0           TEST EAX,EAX
00402C36  |. 7C 12          JL SHORT CACHEMAN.00402C4A    //不跳。
00402C38  |> 59             POP ECX
00402C39  |. 31F6           XOR ESI,ESI
00402C3B  |> 8932           MOV DWORD PTR DS:[EDX],ESI    //ESI=0，把[EDX]单元清0。EDX=ESP（看408C90处即
明白）。
00402C3D  |. 5F             POP EDI
00402C3E  |. 5E             POP ESI
00402C3F  |. 5B             POP EBX
00402C40  |. C3             RETN
```

---

## 后记

这个小软件的注册码算法比较简单，最后一位数的ASSIC码经过乘以6后与`14H`异或，所得的16进制数与前3位数字的16进制相比较，如果相等则OK。注册码算法只与前3位与最后一位有关，与其他位无关。所以比较简单。下面给出一个注册机。

```c
#include "stdlib.h"
#include "time.h"
main()
{
int a;
char b[9]={0};
int s,i;
randomize();   //随机数种子。
a=random(10); //取到随机数
s=6*(a+0x30);  //变成ASSIC码。
s=s^0x14;       //异或运算。
for(i=0;i<7;i++)
b[i]=random(10)+0x30;  //填充中间7位。
b[7]=a+0x30;      //加上最后一位。
printf("the key is \n");
printf("%d_%s",s,b);
}
```

感谢您的阅读与大力支持！！

qduwg@163.com

qduwg

2006年1月30 日晚结稿

[[招生]科锐逆向工程师培训(2026年11月13日实地，远程教学同时开班, 第57期)](https://bbs.kanxue.com/thread-51839.htm)
