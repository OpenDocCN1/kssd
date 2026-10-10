

# \[变形金刚之重拳出击(GunMetal)\]游戏作弊器

> 原文：[`www.pediy.com/kssd/pediy11/120880.html`](https://www.pediy.com/kssd/pediy11/120880.html)

一个老游戏了，最近想怀旧一下，懒得拼命，网上也没有秘籍，遂解密之，附件是自己写的作弊器，无敌模式+无限弹药，放到游戏目录运行即可，下面是跟踪思路：  
  
一、无敌模式  
  
配合金山游侠 V 跟踪 Health 的变化，分别在 Health = 100、99 时搜索浮点值 100.0、99.0，可以判断出一个地址：005E686C；  
OD 载入游戏，一定要删除所有断点，运行到进入正常战斗状态，在 OD 中查找所有常量 005E686C 并全部 F2 下断；  
运行并断下后注意浮点寄存器栈顶数值，如果小于等于 100.0 或 1.0 就要关注断点所在地址，可以找到一处：  
0041F5D4    DB86 94000000     fild dword ptr ds:\[esi+94\]  
0041F5DA    DAB6 9C000000     fidiv dword ptr ds:\[esi+9C\]  
0041F5E0    D80D 6C685E00     fmul dword ptr ds:\[5E686C\]  
  
在地址 ds:\[esi+94\] 下硬件写入断点，断在：  
0041F2F0    D94424 1C         fld dword ptr ss:\[esp+1C\]  
0041F2F4    E8 B39D1600       call 005890AC  
0041F2F9    8B8E 94000000     mov ecx,dword ptr ds:\[esi+94\]  
0041F2FF    2BC8              sub ecx,eax                        ; nop 掉，解除敌方攻击伤害  
0041F301    8BC1              mov eax,ecx  
0041F303    898E 94000000     mov dword ptr ds:\[esi+94\],ecx  
0041F309    85C0              test eax,eax  
0041F30B    7D 0A             jge short 0041F317  
0041F30D    C786 94000000 000>mov dword ptr ds:\[esi+94\],0  
0041F317    5F                pop edi  
0041F318    5E                pop esi  
0041F319    83C4 10           add esp,10  
0041F31C    C2 0800           retn 8  
  
查找指令 mov ecx,dword ptr ds:\[esi+94\]，找到和上面的代码类似的部分：  
0041DF5F    D91C24            fstp dword ptr ss:\[esp\]  
0041DF62    E8 499C0100       call 00437BB0  
0041DF67    E8 40B11600       call 005890AC  
0041DF6C    8B8E 94000000     mov ecx,dword ptr ds:\[esi+94\]  
0041DF72    2BC8              sub ecx,eax                        ; nop 掉，解除触碰伤害  
0041DF74    8BC1              mov eax,ecx  
0041DF76    898E 94000000     mov dword ptr ds:\[esi+94\],ecx  
0041DF7C    85C0              test eax,eax  
0041DF7E    7D 0A             jge short 0041DF8A  
0041DF80    C786 94000000 000>mov dword ptr ds:\[esi+94\],0  
0041DF8A    8B86 74010000     mov eax,dword ptr ds:\[esi+174\]  
0041DF90    5E                pop esi  
0041DF91    8B48 08           mov ecx,dword ptr ds:\[eax+8\]  
0041DF94    C781 40400000 000>mov dword ptr ds:\[ecx+4040\],0  
0041DF9E    59                pop ecx  
0041DF9F    C3                retn  
  
  
二、无限弹药  
  
以导弹为例，用金山游侠 V 搜索几次就能找到关键地址：0065B420，下硬件写入断点，断在  
004424A3    894C24 14         mov dword ptr ss:\[esp+14\],ecx  
004424A7    894F 0C           mov dword ptr ds:\[edi+C\],ecx  
004424AA    DB4424 14         fild dword ptr ss:\[esp+14\]  
004424AE    894424 14         mov dword ptr ss:\[esp+14\],eax  
004424B2    DA7424 14         fidiv dword ptr ss:\[esp+14\]  
004424B6    D95F 08           fstp dword ptr ds:\[edi+8\]  
004424B9    8B0D F0EB7200     mov ecx,dword ptr ds:\[72EBF0\]  
  
往上到  
00442481    8D5424 10         lea edx,dword ptr ss:\[esp+10\]  
00442485    8BCE              mov ecx,esi  
00442487    52                push edx  
00442488    E8 B33B1300       call 00576040                        ; 跟进  
  
来到  
00576040    E8 2BFFFFFF       call 00575F70  
00576045    85C0              test eax,eax  
00576047    74 1D             je short 00576066  
00576049    8B88 50030000     mov ecx,dword ptr ds:\[eax+350\]        ; 关键地址  
0057604F    56                push esi  
00576050    8B7424 08         mov esi,dword ptr ss:\[esp+8\]  
00576054    51                push ecx  
00576055    890E              mov dword ptr ds:\[esi\],ecx  
00576057    8BC8              mov ecx,eax  
00576059    E8 C2A0FFFF       call 00570120  
0057605E    8906              mov dword ptr ds:\[esi\],eax            ; eax = 剩余弹药量  
00576060    B0 01             mov al,1  
00576062    5E                pop esi  
00576063    C2 0400           retn 4  
00576066    32C0              xor al,al  
00576068    C2 0400           retn 4  
  
查找所有常量：350，找到一可疑处  
00570970    8B91 50030000     mov edx,dword ptr ds:\[ecx+350\]  
00570976    B0 01             mov al,1  
00570978    4A                dec edx                               ; 如果 nop 掉，弹药限制会解除，但游戏会崩溃  
00570979    8991 50030000     mov dword ptr ds:\[ecx+350\],edx  
0057097F    79 14             jns short 00570995  
00570981    8B4424 04         mov eax,dword ptr ss:\[esp+4\]  
00570985    C781 50030000 000>mov dword ptr ds:\[ecx+350\],0  
0057098F    50                push eax  
00570990    E8 6BFFFFFF       call 00570900  
00570995    8B5424 08         mov edx,dword ptr ss:\[esp+8\]  
00570999    C702 01000000     mov dword ptr ds:\[edx\],1  
0057099F    C781 54030000 000>mov dword ptr ds:\[ecx+354\],0  
005709A9    C2 0800           retn 8  
  
回到 00576049 处，eax 中的地址保存了战机数据，\[eax+20\]为初始弹药数，只要用该值覆盖\[eax+350\]，就可完美解除弹药限制；  
  
修改后：  
00576049    8B48 20           mov ecx,dword ptr ds:\[eax+20\]  
0057604C  ^ EB C4             jmp short 00576012  
0057604E    90                nop  
  
00576012    8988 50030000     mov dword ptr ds:\[eax+350\],ecx  
00576018    EB 35             jmp short 0057604F  
  
  
有兴趣的朋友可以放心使用作弊器，不会造成游戏和系统的不稳定，经本人 FX5200 烂卡亲测，游戏自始至终未出现异常。

上传的附件
| ![](img/1baac1a3eff5ac15dae6f33dccc134d9.jpg) || GMGod.zip  
 |