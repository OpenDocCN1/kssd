
```
004155F5  |.  E8 A6810200    |call MultiDic.0043D7A0                     ;连接字符  
004155FA  |.  8BFB           |mov edi,ebx                                ;机器码先保存起来  
004155FC  |.  83C9 FF       |or ecx,FFFFFFFF      
004155FF  |.  33C0           |xor eax,eax                                ;清零  
00415601  |.  83C4 0C       |add esp,0C  
00415604  |.  83C6 02       |add esi,2                                  ;step=2  
00415607  |.  F2:AE         |repne scas byte ptr es:\[edi\]             ;偶菜不知道这是什么  
00415609  |.  F7D1          |not ecx  
0041560B  |.  49            |dec ecx                                    ;记数器-1  
0041560C  |.  3BF1         |cmp esi,ecx                                ;循环完了吗？  
0041560E  |.^ 72 C3        \\jb short MultiDic.004155D3  
00415610  |.  5D           pop ebp                                    ;这里可以做内存注册机  
00415611  |>   5E           pop esi  
00415612  |.  5F           pop edi  
00415613  |.  B0 01         mov al,1  
00415615  |.  5B           pop ebx  
00415616  |.  81C4 14020000 add esp,214  
0041561C  \\.  C2 0800      retn 8  
```

好了，基本上算法大概就在上面的  

下面用高级语言描述一下算法（顺便复习一下 `C` `^\_^`），注册机如下：  

```c
#include <stdio.h>  

void main()  

{  

/* KeyGen By XJTL   www.CnXHacker.com */  

  int j,k,t=0,l=0,s[20];  

  char n[20];  

  printf("******  KeyGen For KCDICT2004  ******\n");  

  printf("Please Input Your Serial Num:");  

  scanf("%s",n);  

  while(n[l]!='\0') l++;  

  printf("\n Your License Num is : ");  

  for(k=0;k<l;k+=2)  

  {  

    s[t]=n[k+1]^n[k];  

    s[t]%=0x1A;  

    s[t]+=0x41;  

    printf("%c",s[t]);  

    t++;  

  }  

    printf("\n");  

}  
```

———————————————————————————————————————————  

【最后总结】：  

这里介绍了如何找到算法核心，方便我等菜鸟们步入算法分析时代。希望各位小鸟们加油哟~ 

4dd466d0e1e0cc123afbe504da2462ca
