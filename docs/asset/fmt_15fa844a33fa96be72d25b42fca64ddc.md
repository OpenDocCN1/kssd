
## 汇编代码

```asm
  0049E4E8  |.  75 68             JNZ SHORT clipboar.0049E552           ; 不相等跳  
  0049E4EA  |.  B2 01             MOV DL,1  
  0049E4EC  |.  A1 64604300       MOV EAX,DWORD PTR DS:[436064]  
  0049E4F1  |.  E8 6E7CF9FF       CALL clipboar.00436164  
  0049E4F6  |.  8BD8              MOV EBX,EAX  
  0049E4F8  |.  BA 00000080       MOV EDX,80000000  
  0049E4FD  |.  8BC3              MOV EAX,EBX  
  0049E4FF  |.  E8 007DF9FF       CALL clipboar.00436204  
  0049E504  |.  B1 01             MOV CL,1  
  0049E506  |.  BA BCE54900       MOV EDX,clipboar.0049E5BC           ;  ASCII ".lwx1"  
  0049E50B  |.  8BC3              MOV EAX,EBX  
  0049E50D  |.  E8 567DF9FF       CALL clipboar.00436268  
  0049E512  |.  84C0              TEST AL,AL  
  0049E514  |.  74 11             JE SHORT clipboar.0049E527  
  0049E516  |.  B9 CCE54900       MOV ECX,clipboar.0049E5CC           ;  ASCII "!oncemoreagain"  
  0049E51B  |.  BA E4E54900       MOV EDX,clipboar.0049E5E4           ;  ASCII "registerok"  
  0049E520  |.  8BC3              MOV EAX,EBX  
  0049E522  |.  E8 FD7EF9FF       CALL clipboar.00436424  
  0049E527  |>  6A 00             PUSH 0  
  0049E529  |.  B9 F0E54900       MOV ECX,clipboar.0049E5F0           ;  ASCII "Register OK"  
  0049E52E  |.  BA FCE54900       MOV EDX,clipboar.0049E5FC           ;  ASCII "Register OK,Enjoy it!"  
  0049E533  |.  A1 B83A4A00       MOV EAX,DWORD PTR DS:[4A3AB8]  
  0049E538  |.  8B00              MOV EAX,DWORD PTR DS:[EAX]  
  0049E53A  |.  E8 8960FBFF       CALL clipboar.004545C8  
  0049E53F  |.  A1 7C4D4A00       MOV EAX,DWORD PTR DS:[4A4D7C]  
  0049E544  |.  E8 3727FBFF       CALL clipboar.00450C80  
  0049E549  |.  8BC3              MOV EAX,EBX  
  0049E54B  |.  E8 3455F6FF       CALL clipboar.00403A84  
  0049E550  |.  EB 18             JMP SHORT clipboar.0049E56A  
  0049E552  |>  6A 00             PUSH 0  
  0049E554  |.  B9 14E64900       MOV ECX,clipboar.0049E614           ;  ASCII "Warning"  
  0049E559  |.  BA 1CE64900       MOV EDX,clipboar.0049E61C           ;  ASCII "Invalid registration!!"  
  0049E55E  |.  A1 B83A4A00       MOV EAX,DWORD PTR DS:[4A3AB8]  
  0049E563  |.  8B00              MOV EAX,DWORD PTR DS:[EAX]  
  0049E565  |.  E8 5E60FBFF       CALL clipboar.004545C8  
  0049E56A  |>  33C0              XOR EAX,EAX  
  0049E56C  |.  5A                POP EDX  
  0049E56D  |.  59                POP ECX  
  0049E56E  |.  59                POP ECX  
  0049E56F  |.  64:8910           MOV DWORD PTR FS:[EAX],EDX  
  0049E572  |.  68 9CE54900       PUSH clipboar.0049E59C  
  0049E577  |>  8D45 E0           LEA EAX,DWORD PTR SS:[EBP-20]  
  0049E57A  |.  E8 7562F6FF       CALL clipboar.004047F4  
  0049E57F  |.  8D45 E4           LEA EAX,DWORD PTR SS:[EBP-1C]  
  0049E582  |.  BA 04000000       MOV EDX,4  
  0049E587  |.  E8 8C62F6FF       CALL clipboar.00404818  
  0049E58C  |.  8D45 F8           LEA EAX,DWORD PTR SS:[EBP-8]  
  0049E58F  |.  E8 6062F6FF       CALL clipboar.004047F4  
  0049E594  \\.  C3               RETN  
  0049E595   .^ E9 3E5CF6FF       JMP clipboar.004041D8  
  0049E59A   .^ EB DB            JMP SHORT clipboar.0049E577  
  0049E59C   .  5E                POP ESI  
  0049E59D   .  5B                POP EBX  
  0049E59E   .  8BE5              MOV ESP,EBP  
  0049E5A0   .  5D                POP EBP  
  0049E5A1   .  C3                RETN  
```

---

## 经验总结

### 算法总结

1.  程序内置一个串 `820109`；
2.  紧接着用 `6D` 除以 `A` 计算出另一个串 `109`；
3.  然后依次取注册名，从第二位开始取；位数大于等于 `5` 的话计算 `4` 次，位数小于 `5` 的计算 `strlen(name)-1` 次；
    *   以注册名大于等于 `5` 为例：
        *   从第二位开始取出对应的字母的 `ASCII` 值，余数作为注册码，商作为循环条件，直到商为 `0` 停止，直到取完；
        *   注册码的形式是 `820109109` + 上述计算结果。

```c
#include <stdio.h>
#include <string.h>
#include <math.h>

main()
{
    char name[50];
    long i,j,l,k=0;
    int temp[50];
    
    printf("////////////////////////////////////////////////////\n");
    printf("//        Clipboard Box V2.8      Key Generator //\n");
    printf("//                                                //\n");
    printf("//        Author: ocnzhao[OCN]          //\n");
    printf("//                                                //\n");
    printf("//        E-mail: ocnzhao@163.com       //\n");
    printf("//                                                //\n");
    printf("//   OS : WinXP, PEiD, Ollydbg, C-Free3.5     //\n");
    printf("//                                                //\n");
    printf("//       Date :  2006-05-16          //\n");
    printf("//////////////////////////////////////////////////\n\n");
    
    printf("请输入注册名: ");
    scanf("%s", &name);
    printf("注册码是");
    printf("\n");
    
    if ( strlen(name) >= 5)
    {
        printf("820109109");
        for(i=1; i<5; i++)
        {
            j=0;
            do
            {
                temp[j] = name[i] % 0xA;
                name[i] = name[i] / 0xA;
                j++;
            }
            while(name[i] != 0);
            
            for(k=j-1; k>=0; k--)
            {
                printf("%d", temp[k]);
            }
        }
    }
    else
    {
        if(strlen(name) < 5)
        {
            l = strlen(name);
            l--;
            printf("820109109");
            for(i=1; i<=l; i++)
            {
                j=0;
                do
                {
                    temp[j] = name[i] % 0xA;
                    name[i] = name[i] / 0xA;
                    j++;
                }
                while(name[i] != 0);
                
                for(k=j-1; k>=0; k--)
                {
                    printf("%d", temp[k]);
                }
            }
        }
    }
    
    printf("\n");
}
```

> 注：`CMD` 窗口出现的注册码直接不好复制，可以点鼠标右键选择“标记”再复制，省得照着往里输，麻烦。

---

## 版权声明

本文原创于看雪技术论坛，转载请注明作者并保持文章的完整，谢谢！

2006 年 05 月 16 日 21:03:08

`4dd466d0e1e0cc123afbe504da2462ca`
