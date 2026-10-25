
`if (var1==0)`

`var1=0x11;//如果加数的位置为第一个字符，改变指针到最后一个字符`

`var1-- //循环减一`

`}`

500次循环做完，得到的结果就是最后的注册码，就是取在数组中的前6个Byte对应的数字

`****************`

```asm
00401A51                mov
esi, offset a0123456789abcd ; "0123456789abcdef"
00401A56                lea
edi, [ebp+var_1C]
00401A59                movsd
00401A5A                movsd
00401A5B                movsd
00401A5C                and
dword_44E168, 0
00401A63                movsd
00401A64                movsb
00401A65                mov
esi, [ebp+arg_4]
00401A68                xor
edi, edi
00401A6A

00401A6A loc_401A6A:
            ; CODE XREF: sub_401940+15A j
00401A6A                movzx
eax, byte ptr [edi+ebx]
00401A6E                mov
dl, [esi]
00401A70                mov
ecx, eax
00401A72                shr
ecx, 4
00401A75                cmp
dl, [ebp+ecx+var_1C]
00401A79                jnz
short loc_401A8B ;right can't jump
00401A7B                mov
cl, [esi+1]
00401A7E                and
eax, 0Fh
00401A81                cmp
cl, [ebp+eax+var_1C];比较你的Key跟计算结果相等吗？
00401A85                jnz
short loc_401A8B;在这里正确都不跳，
；不然就置错误标志
00401A87                xor
eax, eax；标志0是正确
00401A89                jmp
short loc_401A8E; must to there
00401A8B ; ---------------------------------------------------------------------------

00401A8B
00401A8B loc_401A8B:
            ; CODE XREF: sub_401940+139 j
00401A8B
                    ; sub_401940+145
j
00401A8B                push
1; 要是在这里就错了
00401A8D                pop
eax
00401A8E

00401A8E loc_401A8E:
            ; CODE XREF: sub_401940+149 j
00401A8E                or
  dword_44E168, eax
00401A94                inc
edi
00401A95                inc
esi
00401A96                inc
esi
00401A97                cmp
edi, 6；比较前6个byte的信息。
00401A9A                jl
  short loc_401A6A；
00401A9C                mov
eax, dword_453670
00401AA1                pop
edi
00401AA2                pop
esi
00401AA3

00401AA3 loc_401AA3:
            ; CODE XREF: sub_401940+31 j
00401AA3
                    ; sub_401940+71
j
00401AA3                pop
ebx
00401AA4                leave
00401AA5                retn
00401AA5 sub_401940      endp
00401AA5
```

程序将这些信息写入注册表的`HLM\Software\Centipede\Rops\rops`中，每次启动时判断。注册机付在后边。

（转贴请保持完整）

`AllRight reserved :[CCG]`

### VC 控制台下注册机

嘿嘿，小弟太菜。注册机中好多BVG,谢谢大哥指出。首先，数组变量的分配因该在主程序外边，在里边定义会 在`strcat(temp,name)`时当名字过长时，覆盖数据为啥会出现这样的情况小弟原因不明，希望哪个大哥指教;
第二，字符结尾要用`0x00`结束。不然会在后边显示一些怪字符.
第三，这个注册机不完美，还有些路没有去走，嘿嘿，哪个大哥把他补全。
只希望初学者去走做，对于高手这个东西也太简单了。大家一起学习。去走走看。
小弟献丑了。

```c
include<windows.h>
  char name[100],user[10],temp[18],covert[13]; //BUG修改定义成全局变量
  int i,var,var1,var2;
  BYTE lena,lenb;

main()
{

  printf("[CCG] KeyGen\n");
  printf("please input your name:\n");
  scanf("%s",name);
    getchar();
  printf("please input user number:\n");
  scanf("%s",user);
    getchar();
    var=strlen(name);
    memset(temp,0x00,18);
    if(var<17)
    {
        var=17/var+1;
      for(i=0;i<var;i++)
      strcat(temp,name);
    }
    else
      strncpy(temp,name,17);

    temp[17]=0x00;
    var=strlen(user);
    if(var>10)
        temp[16]=0x30;
    else
      memcpy(&temp[17-var],user,var);
//    printf("%s\n",temp);
    var1=0xc;
    var2=1;
    for(i=0;i<500;i++)
    {
     var2--;
    temp[var2]=temp[var2]+temp[var1];
     if(var2==0)
         var2=0x11;

      if(var1==0)
         var1=0x11;
        var1--;
    }
for(i=0;i<6;i++)
{
lena=(unsigned char)temp[i]/0x10;
lenb=(unsigned char)temp[i]&0xf;
  if(lena<10)
  covert[2*i]=lena+0x30;
  else
  covert[2*i]=lena+0x57;
  if(lenb<10)
  covert[2*i+1]=lenb+0x30;
  else
  covert[2*i+1]=lenb+0x57;

}
  covert[12]=0x00; //BUG最后加上结尾不然会显示错误
  printf(" key is:%s",covert);
  }
```
