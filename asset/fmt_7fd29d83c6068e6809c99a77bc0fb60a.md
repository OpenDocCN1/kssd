
# 通用文档微机管理系统 V1.0

> 原文：[`www.pediy.com/kssd/pediy03/forum1089.htm`](https://www.pediy.com/kssd/pediy03/forum1089.htm)

**下手软件：**

通用文档微机管理系统 V1.0 单机版

**下载地址：**
`http://www.skycn.com/down1.php?id=5811&url=http://202.107.104.34/soft/down/xhonghuisetupV1.0.exe`

**软件用途：**

年终岁首，文书归档立卷工作是办公室一项十分头痛的事，传统的手工抄写文件级目录不仅劳动量大，而且难以规范统一，更重要的是即使立卷工作完成了，但查档并不轻松，为此我们严格依照最新的档案管理国家标准，开发了这套软件，立足档案室工作实际，解决基层档案工作具体问题，做到打印卷内目录整洁美观，查找档案既快又准，为促进档案管理的规范化和正规化打好基础。

**破解原因：**

为了加入 FCG，成为“三冠王”

**使用工具：**
`soft-ice`，`ursoft w32dasm` (kanker 赠送版，反编译中文效果奇好！！)，`hiew 640 注册版`

**破解过程：**

首先启动软件，观察软件的注册方式，发现这个软件采用的是一机一码的方式，首先看见了机器码（准备暑假学习，机器码的产生方式和学习注册机的制作了）在注册码的位置填写`copyyour`，这个任意写，`ctrl+d` 调用 `soft-ice`

下断点`hmemcpy`按`f12`一共 12 次，第 13 次时会出错，好，我们现在通过反编译来看看代码：

如下：

```
:005FF1A1 8D55F4                   lea edx, dword ptr [ebp-0C]     ======>>>>进入后光标在这里
:005FF1A4 8B87DC020000             mov eax, dword ptr [edi+000002DC]
:005FF1AA E8E5A1E3FF               call 00439394
:005FF1AF 8B45F4                   mov eax, dword ptr [ebp-0C]
:005FF1B2 8D55F8                   lea edx, dword ptr [ebp-08]
:005FF1B5 E8C2ACE0FF               call 00409E7C
:005FF1BA 8B45F8                   mov eax, dword ptr [ebp-08]     ===========此处是错误的注册码
:005FF1BD 8B55FC                   mov edx, dword ptr [ebp-04]
:005FF1C0 E8E350E0FF               call 004042A8                   ===========此处 edx 发现正确注册码 没能进入此 call 观察，注册码的算法应该在这个 call 里面 初学者可按 f8 跟入
:005FF1C5 7419                     je 005FF1E0                     ===========千万不能让它跳

* Possible StringData Ref from Code Obj ->"注册失败"

                                                  |
:005FF1C7 B8D0F25F00                 mov eax, 005FF2D0
:005FF1CC E85355E6FF                 call 00464724
:005FF1D1 8B87DC020000               mov eax, dword ptr [edi+000002DC]
:005FF1D7 33D2                       xor edx, edx
:005FF1D9 E8E6A1E3FF                 call 004393C4
:005FF1DE EB52                       jmp 005FF232

* Referenced by a (U)nconditional or (C)onditional Jump at Address:
|:005FF1C5(C)
|
:005FF1E0 B201                       mov dl, 01
:005FF1E2 A1DCEC4F00                 mov eax, dword ptr [004FECDC]
:005FF1E7 E85CFCEFFF                 call 004FEE48
:005FF1EC 8BD8                       mov ebx, eax
:005FF1EE BA02000080                 mov edx, 80000002
:005FF1F3 8BC3                       mov eax, ebx
:005FF1F5 E82AFDEFFF                 call 004FEF24
:005FF1FA B101                       mov cl, 01

* Possible StringData Ref from Code Obj ->"Software\Microsoft\Windows\CurrentVersion\Gene"
                                                  ->"ral document"      =========== 从这里我们很轻松的就知道注册成功后 会写入注册表的哪个键值 从下面得知是 将在 gc_id 中发生变化，由于我开始没有注意，所以没能发现是什么变化

                                                  |
:005FF1FC BA6CF25F00                 mov edx, 005FF26C
:005FF201 8BC3                       mov eax, ebx
:005FF203 E860FEEFFF                 call 004FF068
:005FF208 84C0                       test al, al
:005FF20A 7426                       je 005FF232
:005FF20C 33C9                       xor ecx, ecx

* Possible StringData Ref from Code Obj ->"gc_id"

                                                  |
:005FF20E BAB0F25F00                 mov edx, 005FF2B0
:005FF213 8BC3                       mov eax, ebx
:005FF215 E8FE03F0FF                 call 004FF618
:005FF21A 8BC3                       mov eax, ebx
:005FF21C E8D3FCEFFF                 call 004FEEF4
:005FF221 8BC3                       mov eax, ebx
:005FF223 E8C83EE0FF                 call 004030F0
```

好了，你要爆破的话就用 `hiew` 修改`:005FF1C5 7419                     je 005FF1E0`

`74` 改 `75`

想要得到注册码的话就在`:005FF1C0 E8E350E0FF               call 004042A8`

下 `d edx` 将看到注册码

破解完成后，采用 `CodeFusion 中文版`制作补丁，这样会使补丁文件很小

补丁将在我朋友老虫网站发布 `http://bugbug.6to23.com`

另外这篇教程是破解后三小时写的，全凭记忆，可能会有出错的地方，请指正，谢谢

此篇文章版权为 CCG BCG FCG 共同拥有

`cracker by copyyour`

```c
#include <stdio.h>

#include <stdlib.h>

#include <ctype.h>

#include <dos.h>

long sturn(char a[80])

{

  int i,j,l;

  long d,s=0;

  l=strlen(a);

  for(i=0;i<l;i++)

  {if(a[i]>47&&a[i]<58) d=a[i]-48;

  if(a[i]>64&&a[i]<71) d=a[i]-55;

  if(a[i]>96&&a[i]<103) d=a[i]-87;

  for(j=l-i;j>1;j--)

  d*=16;

  s+=d;}

  return s;

  }

main(void)

{

long sn;

char cc[4];

char serial_no[10];

union REGS r;

struct SREGS s;

unsigned sno1, sno2;

struct date today;

getdate(&today);

cc[0]=(today.da_mon/10)%10+65;

cc[1]=(today.da_mon/1)%10+66;

cc[2]=(today.da_mon/10)%10+67;

cc[3]=(today.da_mon/1)%10+68;

r.x.ax = 0x6900;

r.h.bl = 3; /* A:=1, B:=2, C:=3 etc. */

segread(&s);

intdosx(&r, &r, &s);

if (r.x.cflag)

*serial_no = "";

else

{

sno2 = *((unsigned far *)MK_FP(s.ds, r.x.dx+2));

sno1 = *((unsigned far *)MK_FP(s.ds, r.x.dx+4));

  sprintf(serial_no, "%04X%04X", sno1, sno2);

}

/*printf("The Serial Number of Login Disk [C] is %s\n", serial_no );*/

sn=sturn(serial_no);

sprintf(serial_no,"%ld",sn);

/*printf("%s\n",serial_no);*/

printf(" ******FJWDGL 2.1******\n This keygen is made by Pretender\n ");

printf("Your Register code is:  ");

for(sn=0;sn<strlen(serial_no);sn++)

printf("%c",serial_no[sn]+17);

printf("%c",cc[0]);

printf("%c",cc[1]);printf("%c",cc[2]);printf("%c",cc[3]+1);

printf("\n *************************************      ---    ---    ---\n *Welcome to WWW.CRACKNOW.COM*      /      /      / --\n *************************************      ---    ---    --/\n");}
```

`4dd466d0e1e0cc123afbe504da2462ca`
