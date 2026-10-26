
### 反汇编

```
00402737    C64424 1B 4D         mov byte ptr ss:[esp+1B],4D
0040273C    C64424 1C 65         mov byte ptr ss:[esp+1C],65
00402741    C64424 1D FD         mov byte ptr ss:[esp+1D],0FD
00402746    C64424 1E A5         mov byte ptr ss:[esp+1E],0A5
0040274B    C64424 1F 7B         mov byte ptr ss:[esp+1F],7B
00402750    894C24 10            mov dword ptr ss:[esp+10],ecx
00402754    8A08                 mov cl,byte ptr ds:[eax]
00402756    83C0 01              add eax,1
00402759    84C9                 test cl,cl
0040275B  ^ 75 F7                jnz short LingoLes.00402754
0040275D    2B4424 10            sub eax,dword ptr ss:[esp+10]                        ; //得到Email的位数
00402761    74 41                je short LingoLes.004027A4
00402763    8B06                 mov eax,dword ptr ds:[esi]
00402765    8A0C17               mov cl,byte ptr ds:[edi+edx]                         ; //循环取Email的各位到cl
00402768    0FB66C04 18          movzx ebp,byte ptr ss:[esp+eax+18]                     // 从[ESI]＋1位开始循环取数组中的元素到EBP
0040276D    0FB6C9               movzx ecx,cl
00402770    33E9                 xor ebp,ecx                                          ; // Email的各位^EBP
00402772    0FAF6C24 40          imul ebp,dword ptr ss:[esp+40]                       ; // (Email的各位^EBP)\*[ESP+40]
00402777    83C0 01              add eax,1
0040277A    03DD                 add ebx,ebp                                          ; // 累加在EBX
0040277C    83F8 08              cmp eax,8
0040277F    8906                 mov dword ptr ds:[esi],eax                             // [ESI]=EAX  保存EAX的值
00402781    7C 06                jl short LingoLes.00402789                             // 取完数组A的最后一位再从第一位开始取
00402783    C706 00000000        mov dword ptr ds:[esi],0
00402789    8BC7                 mov eax,edi
0040278B    83C2 01              add edx,1
0040278E    8D68 01              lea ebp,dword ptr ds:[eax+1]
00402791    8A08                 mov cl,byte ptr ds:[eax]
00402793    83C0 01              add eax,1
00402796    84C9                 test cl,cl
00402798  ^ 75 F7                jnz short LingoLes.00402791
0040279A    2BC5                 sub eax,ebp
0040279C    3BD0                 cmp edx,eax
0040279E  ^ 72 C3                jb short LingoLes.00402763
004027A0    8B6C24 14            mov ebp,dword ptr ss:[esp+14]
004027A4    81E3 FFFF0000        and ebx,0FFFF                                        ; //计算结果^=0xffff
004027AA    53                   push ebx
004027AB    8D5424 24            lea edx,dword ptr ss:[esp+24]
004027AF    68 64074400          push LingoLes.00440764                               ; ASCII "%04x"
004027B4    52                   push edx
004027B5    E8 C0550200          call LingoLes.00427D7A
```

---

# 经验总结

## 总结

算法蛮简单的,`keygen`代码

刚学C不久,代码写得很乱。。。。。。。。

```c
#include<stdio.h>
#include<string.h>
void main()
{    char email[64]={0};
     long sn[3]={0};
     int esp,esi=0,i,j,z,len,k[]={0x17,0x47,9};
     int a[]={0xc,0xe8,0x2b,0x4d,0x65,0xfd,0xa5,0x7b};  //初始化数组
     printf("请输入Email地址:\n");
     gets(email);
     len=strlen(email);
     for(z=0,j=esi;z<3;z++)  //循环计算三次
     {
      for(i=0;i<len;i++)
      {
      sn[z]+=(email[i]^a[j])*k[z];
      j++;
      esi=j;
      if(j>=8)
      j=0;
      }
      sn[z]&=0xffff;
    }
     printf("序列号:\n");
     printf("%lx-%lx-%lx",sn[0],sn[1],sn[2]);
     getch();
}
```

在`Dev-C++` 和`winxp2`下编译成功！

---

【版权声明】: 本文原创于看雪技术论坛, 转载请注明作者并保持文章的完整, 谢谢!

2007年04月10日 23:38:33

[冰与火的战歌：Windows内核攻防实战高级班！从零到实战，融合AI与Windows内核攻防全技术栈，打造具备自动化能力的内核开发高手。](https://www.kanxue.com/book-section_list-227.htm)
