
## 四、算法小结

1. 注册码分两部分，“-”连接。由用户名计算而来。用户名位数必须介于`5`到`150`之间。
2. 算法`1`部分计算所得数值转为`10`进制，即为注册码第一部分。注册码第一部分主要由用户名前`4`位参与计算而得。
   如果用户名大于`9`位，则第`9`、`10`、`11`、`12`位可能参与计算（取决于用户名长度）。
3. 算法`2`部分计算所得数值转为`10`进制，即为注册码第二部分。注册码第二部分主要由用户名第`5`、`6`、`7`、`8`或`13`位开始参与计算而得。
   具体参与位数取决于用户名长度。

## 四、C 语言注册机代码

代码：

```c
#include "stdio.h"
#include "string.h"
int main()
{
    int len, reg1, reg2=0;
    char name[250];
    scanf("%s", name);
    len = strlen(name);
    if (len>=5 && len<=150)
    {
        __asm
        {
            xor esi, esi
            xor edx, edx
            xor edi, edi
            L0:
            movsx   ebx, byte ptr [name+edi]
            cmp     edx, 4
            jge L011
            push    3
            add     ebx, 10E1h
            pop     ecx
            sub     ecx, edx
            shl     ecx, 3
            shl     ebx, cl
            or      esi, ebx
            jmp L018
            L011:
            push    7
            add     ebx, 4D2h
            pop     ecx
            sub     ecx, edx
            shl     ecx, 3
            shl     ebx, cl
            or      reg2, ebx
            L018:
            inc     edx
            cmp     edx, 8
            jnz L022
            xor     edx, edx
            L022:
            add edi, 1
            cmp     byte ptr [name+edi], 0
            jnz L0
            mov reg1, esi
        }
        printf("%d-%d", reg1, reg2);
    }
    else
        printf("用户名至少为 5 位");
    return 0;
}
```
