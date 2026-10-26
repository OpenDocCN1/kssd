
0040252F  |>  66:83FB 71    cmp     bx, 71
00402533  |.  75 05         jnz     short 0040253A
00402535  |.  83C8 FF       or      eax, FFFFFFFF
00402538  |.  EB 50         jmp     short 0040258A
0040253A  |>  66:81FB B100  cmp     bx, 0B1
0040253F  |.  75 05         jnz     short 00402546
00402541  |.  83C8 FF       or      eax, FFFFFFFF
00402544  |.  EB 44         jmp     short 0040258A
00402546  |>  66:81FB BA00  cmp     bx, 0BA
0040254B  |.  75 06         jnz     short 00402553
0040254D  |.  66:B8 0100    mov     ax, 1
00402551  |.  EB 37         jmp     short 0040258A
00402553  |>  66:81FB 8211  cmp     bx, 1182
00402558  |.  75 06         jnz     short 00402560
0040255A  |.  66:B8 0500    mov     ax, 5
0040255E  |.  EB 2A         jmp     short 0040258A
00402560  |>  0FB7D3        movzx   edx, bx                          ; 取注册码转为十六进制数后的低8位
00402563  |.  0FBFC8        movsx   ecx, ax                          ; 取注册名经过运算后的低8位
00402566  |.  81C1 10270000 add     ecx, 2710                        ; 余数加上0x2710
0040256C  |.  3BD1          cmp     edx, ecx                         ; ①比较是否相等
0040256E  |.  0F94C0        sete    al
00402571  |.  83E0 01       and     eax, 1
00402574  |.  66:83FE 06    cmp     si, 6                            ; ②注册名不能少于6个字符
00402578  |.  0F9DC2        setge   dl
0040257B  |.  83E2 01       and     edx, 1
0040257E  |.  23C2          and     eax, edx                         ; ①②同时满足，注册成功
00402580  |.  74 06         je      short 00402588
00402582  |.  66:B8 0700    mov     ax, 7
00402586  |.  EB 02         jmp     short 0040258A
00402588  |>  33C0          xor     eax, eax
0040258A  |>  5E            pop     esi
0040258B  |.  5B            pop     ebx
0040258C  |.  5D            pop     ebp
0040258D  \.  C3            retn

### 一组可用的注册码
`regist=22765`
`username=PantsT`

注册成功后，“关于”里面显示 Special User。

本小鸟第一次发帖，如果分析过程有问题，请各位大侠指正。谢谢！

---

## 【经验总结】

贴上 C 编写的注册机。

本人编程习惯不是太好，代码没优化，望各位看客多多包涵。

```c
#include<stdio.h>
#include<string.h>
main()
{
char name[20];
unsigned int serial=0xffff;
unsigned int i,j,temp;
re: printf("Name:\t");
gets(name);
if(strlen(name)<6)
  {
  printf("Error!\n\n");
  goto re;
  }
for(i=0;i<strlen(name);i++)
  {
  temp=name[i];
  temp<<=8;
  serial^=temp;
  for(j=0;j<8;j++)
    {
    if(serial>>15)
      {
       serial+=serial;
       serial^=0x1021;
      }
    else serial+=serial;
    }
  }
serial=~serial;
serial&=0xffff;
serial%=0x58f0;
serial+=0x2710;
printf("Serial:\t%d\n",serial);
getchar();
}
```

---

【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

2006年10月24日 下午 9:49:12

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
