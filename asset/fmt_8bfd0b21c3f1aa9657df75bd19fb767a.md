
### 汇编代码片段

```asm
004D088B  |. E8 F888F3FF    CALL SM6.00409188                     生成 M2  
004D0890  |. FF75 DC        PUSH DWORD PTR SS:[EBP-24]  
004D0893  |. 68 20094D00    PUSH SM6.004D0920                     ASCII "721"  
004D0898  |. 8D45 E4        LEA EAX,DWORD PTR SS:[EBP-1C]  
004D089B  |. BA 06000000    MOV EDX,6  
004D08A0  |. E8 1345F3FF    CALL SM6.00404DB8                     计算组装注册码的 call  
004D08A5  |. 8B55 E4        MOV EDX,DWORD PTR SS:[EBP-1C]  
004D08A8  |. 58             POP EAX  
004D08A9  |. E8 8E45F3FF    CALL SM6.00404E3C                     关键注册码比较 call  
004D08AE  |. 75 04          JNZ SHORT SM6.004D08B4  
```

### 五、【注册机和算法】

*   机器码 = `"S6L"` + 变 `code1` + `"J"` + 变 `code2` + `"L"`（21 位）
*   注册码 = `"L6A"` + `M1` + `"1213XU"` + `M2` + `"721"`
*   认证码 = `code` + `"0381"`
*   `Sum` = 机器码的字节和
*   `M1` - 参算码 = `sum * 3`
*   `M2` - 参算码 = `sum * 2`
*   `code1` = （C 盘卷标值）

`M1`、`M2` 和（`code1` 每位）都是参与计算注册码和认证码的部分。

**公共计算参算码的代码部分：**

省略大量代码。。。跟踪算码部分来到这个关键地方。。。

`eax` 装入参算码(被除数)

```asm
00409DEE  |$ B9 0A000000    MOV ECX,0A                     置除数 Ah  
00409DF3  |> 8D75 9F        LEA ESI,DWORD PTR SS:[EBP-61]  
00409DF6  |> 31D2           /XOR EDX,EDX                     余数位置清零  
00409DF8  |. F7F1           |DIV ECX                         eax/A  
00409DFA  |. 80C2 30        |ADD DL,30                        余数+30h  
00409DFD  |. 80FA 3A        |CMP DL,3A  
00409E00  |. 72 03          |JB SHORT SM6.00409E05            >=3Ah，dl+7h  
00409E02  |. 80C2 07        |ADD DL,7                        
00409E05  |> 4E            |DEC ESI  
00409E06  |. 8816           |MOV BYTE PTR DS:[ESI],DL         结果放入预定位置（注意：顺序为倒序）  
00409E08  |. 09C0           |OR EAX,EAX  
00409E0A  |. 75 EA          \JNZ SHORT SM6.00409DF6           被除数为 0 就结束  
```

下面是我在 Turbo C 下勉强通过的算法注册机：欢迎指正（语病、精简、更好的便径）

```c
#include <stdio.h>
#include <string.h>

unsigned int r,i;
unsigned char q[20];

unsigned int f(int n)
{i=0;
 while(n!=0)
   {r=n%0xa+0x30;
    n=n/0xa;
    if(r>=0x3A)
    {r+=7;}
    q[i]=r;
    i+=1;
   }
 return(q[0]);
}

main()
{unsigned int code_len,sum=0,r,i,j,n,m1,m2,temp;
unsigned char code[21],x,y,z,resn[8];

start: printf("\n");
       printf(" *******************************************************\n");
       printf("*              XXXX 网页特效精灵 6.0 XP 特别版 KeyGen           *\n");
       printf("*               Code by NewHand[BCG]                     *\n");
       printf("*   Copyright-2003 [BCG] All Rights Reserved    2003-10-30    *\n");
       printf(" *******************************************************\n");
       
printf("提示：\n 注册码在主程序 sm6.exe 注册框输入!\n 认证码在 sm6reg.exe 认证框输入!\n\n");
printf("请输入你的机器码：");
gets(code);
printf("\n");

code_len=strlen(code);
if(code_len!=21)
{
printf("\n 你输入的机器码不对！\n\n");
goto start;
}
else
{for(i=0;i<=code_len;i++)
sum=sum+code[i];
m1=sum*3;
m2=sum*2;
printf("\n 你的认证码是：");
for(i=0;i<=7;i++)
{resn[i]=code[3+i];
z=f(resn[i]);
printf("%c%c",q[1],q[0]);
}
printf("0381\n");

x=f(m1);
for(j=0;j<=strlen(q)/2;j++)
{temp=q[j];q[j]=q[strlen(q)-1-j];q[strlen(q)-1-j]=temp;
}
printf("\n 你的注册码是：L6A");
printf("%s",q);
printf("1213XU");

y=f(m2);
for(j=0;j<=strlen(q)/2;j++)
{temp=q[j];q[j]=q[strlen(q)-1-j];q[strlen(q)-1-j]=temp;
}
printf("%s",q);
printf("721");
printf("\n\n\n");
printf("                 _/__/  _/__/  _/__/ \n");
printf("                /    /  /        /  \n");
printf("                _/__/  /        /  _/__/ \n");
printf("                 /    /  /        /    /    \n");
printf("                _/__/  _/__/  _/__/     \n");
}
}
```

`4dd466d0e1e0cc123afbe504da2462ca`
