
将要跳转的地址计算填入特征位置，跳转回去的地址填入位置，计算位置这个看代码吧不好描述。

```c
bool ChangePeopleMem(char* changePeopleMem)

{

 HANDLE selfProHandle = GetCurrentProcess();

 DWORD dwOffset;

 /*

 address + 0000 0000 - 55                    - push ebp

 address + 0000 0001 - 8B EC                 - mov ebp,esp

 address + 0000 0003 - 83 EC 08              - sub esp,08

 address + 0000 0006 - 8B 45 08              - mov eax,[ebp+08]

 address + 0000 0009 - 8B 40 08              - mov eax,[eax+08]

 address + 0000 000C - 8B 40 64              - mov eax,[eax+64]

 address + 0000 000F - 8B 40 10              - mov eax,[eax+10]  ---------------------------》改到自己的内存 E9 ?? ?? ?? ??  

 address + 0000 0012 - 85 C0                 - test eax,eax  

 address + 0000 0014 - 75 08                 - jne 24D70D26    《----------------------------运行完自己的逻辑在跳转回来

 */

 char changePeopleByte[] = { 0xE9,0x90,0x90 ,0x90 ,0x90 };

 bool flag_research = true;

 int addressOffset = 0;

 int offset_addr = 0;

 if (EnumAllMemoryBlocks(selfProHandle, vec)) {

  for (int i = 0; i < vec.size(); i++) {

   // 查找特征

   addressOffset = MemoryCmpToFeatureCode(vec[i].BaseAddress, vec[i].RegionSize, mem_of_callpeople);

   if (addressOffset != -1)

   {

    for (int j = 0; j < vecCodeAddr.size(); j++) {

     int int_value = (int)vec[i].BaseAddress + (int)vecCodeAddr[j];

     g_changePeopleAddress = (PDWORD)(int_value + 0xF);//定位到修改

     dwOffset = ((DWORD)changePeopleMem - ((DWORD)g_changePeopleAddress)) - 0x5;//计算位置

     *(DWORD*)(changePeopleByte + 0x1) = dwOffset;

     memcpy(g_changePeopleAddress, changePeopleByte, 0x5);

     g_changePeopleAddress = (PDWORD)(int_value + 0x14);

     dwOffset =(DWORD)g_changePeopleAddress - (DWORD)(changePeopleMem + 0x173)-0x5;//跳转回来

     *(DWORD*)(changePeopleMem + 0x173 + 0x1) = dwOffset;

     return true;

    }

   }

  }

 }

 else

 {

  MessageBox(NULL, "memallpeople failed.", "failed", NULL);

  return false;

 }

}
```

对应图：

![](https://bbs.kanxue.com/upload/attach/202005/845934_39K9WZM3X2399FQ.jpg)

## 快速研究及建造

研究也是按照同样的方法搜索，不知道类型就搜索所有类型根据改变一点一点确定范围，找到访问的位置

![](https://bbs.kanxue.com/upload/attach/202005/845934_QTSJXMQ8W6MF88U.jpg)

### 函数特征

```c
/*访问研究数值修改位置的特征

1607BBC5 - 8D 4C B1 10           - lea ecx,[ecx+esi*4+10]

1607BBC9 - D9 19                 - fstp dword ptr [ecx]

1607BBCB - 8B 48 30              - mov ecx,[eax+30]  -------》跳到我们地址

1607BBCE - 41                    - inc ecx 

1607BBCF - 89 48 30              - mov [eax+30],ecx  

1607BBD2 - 8D 65 F4              - lea esp,[ebp-0C]

1607BBD5 - 5E                    - pop esi

1607BBD6 - 5F                    - pop edi

1607BBD7 - 5B                    - pop ebx

1607BBD8 - C9                    - leave 

1607BBD9 - C3                    - ret 

  */  

char mem_of_fast_research[] = { 0x8D ,0x4C, 0xB1, 0x10,0xD9, 0x19,0x8B, 0x48 ,0x30,0x41,0x89, 0x48, 0x30,0x8D, 0x65, 0xF4,0x5E,0x5F,0x5B,0xC9,0xC3,0x00 };
```

但是这个访问的位置不一定是研究，还有其他的数据，我们只修改研究，所以要先判断修改的数据地址是否在数据地址附近，说实话这个方法不好，正常应该找到像人物数据一样的结构存放，我这里偷懒值直接特征码数据获取地址进行判断。

同样跳转到自己的代码

```c
void InitFastResearchData()

{

 fastResearchMem = (CHAR*)malloc(0x2A);

 /*

  0208153B - D9 45 10              - fld dword ptr [ebp+10]

  0208153E - 39 71 0C              - cmp [ecx+0C],esi

  02081541 - 0F86 3F000000         - jbe 02081586

  --------跳到下面这

  02081547 - 8D 4C B1 10           - lea ecx,[ecx+esi*4+10]

  0208154B - D9 19                 - fstp dword ptr [ecx]

  0040EB19      8B09     - mov ecx,dword ptr ds:[ecx]

  0040EB1B      81C1 FFFF0000   - add ecx,0xFFFF

  0208154D - 8B 48 30              - mov ecx,[eax+30]

  02081550 - 41                    - inc ecx

  02081551 - 89 48 30              - mov [eax+30],ecx

  02081554 - 8D 65 F4              - lea esp,[ebp-0C]

  02081557 - 5E                    - pop esi

  02081558 - 5F                    - pop edi

  02081559 - 5B                    - pop ebx

  0208155A - C9                    - leave 

  0208155B - C3                    - ret

  0040EAEA      81F9 ????????    cmp ecx,0x12345678 大 判断是不是数据的地址不是跳过 

  0040EAF0      77 10            ja short 0040EB02

  0040EAF2      81F9 ????????    cmp ecx,0x12345678 小 判断是不是数据的地址不是跳过

  0040EAF8      76 08            jbe short 0040EB02

  0040EAFA      8B09             mov ecx,dword ptr ds:[ecx]

  0040EAFC      81C1 FFFF0000    add ecx,0xFFFF

  0040EB02      90               nop

  0040EB03      8B48 30          mov ecx,dword ptr ds:[eax+0x30]

  0040EB06      41               inc ecx

  0040EB07      8948 30          mov dword ptr ds:[eax+0x30],ecx

  0040EB0A      8D65 F4          lea esp,dword ptr ss:[ebp-0xC]

  0040EB0D      5E               pop esi

  0040EB0E      5F               pop edi

  0040EB0F      5B               pop ebx

  0040EB10      C9               leave

  0040EB11      C3               retn

  81F978563412771081F97856341276088B0981C1FFFF0000908B4830418948308D65F45E5F5BC9C3

 */

 char changeByte[] = { 0x81,0xF9,0x78,0x56,0x34,0x12,0x77,0x13,0x81,0xF9,0x78,0x56,0x34,0x12,0x76,0x0B,0x50,0x8B,0x01,0x05,0xFF,0xFF,0x00,0x00,0x89,0x01,0x58,0x8B,0x48,0x30,0x41,0x89,0x48,0x30,0x8D,0x65,0xF4,0x5E,0x5F,0x5B,0xC9,0xC3 };

 //char changeByte[] = { 0x8D ,0x4C ,0xB1 ,0x10 ,0xD9 ,0x19 ,0x8B,0x09 ,0x81,0xC1 ,0xFF,0xFF,0x00,0x00,0x8B ,0x48 ,0x30 ,0x41 ,0x89 ,0x48 ,0x30,0x8d,0x65,0xf4,0x5e,0x5f,0x5b,0xc9,0xc3 };

 memcpy(fastResearchMem, changeByte, 0x2A);

 DWORD dwOldAttrubet;

 VirtualProtect(fastResearchMem,

  0x2A,

  PAGE_EXECUTE_READWRITE,

  &dwOldAttrubet);

}
```

最后是快速建造，方法大家研究吧，因为写这个的时候也没怎么做笔记，从新记录下来会丢失很多细节，这些都是再边逆向，边编程解决的，不过大致方法和原理都是一样的。

找到关键函数，直接`NOP`掉2中的代码就会建造成功。

![](https://bbs.kanxue.com/upload/attach/202005/845934_K3VAMYZYPB3BBDK.jpg)

### 特征

```c
/*

seg000:0C4EEE56 D9 43 58                                      fld     dword ptr [ebx+58h]

seg000:0C4EEE59 D9 45 F0                                      fld     [ebp+var_10]

seg000:0C4EEE5C DE C1                                         faddp   st(1), st

seg000:0C4EEE5E D9 5B 58                                      fstp    dword ptr [ebx+58h]

seg000:0C4EEE61 D9 43 58                                      fld     dword ptr [ebx+58h]

seg000:0C4EEE64 D9 45 E0                                      fld     [ebp+var_20]

seg000:0C4EEE67 DF F1                                         fcomip  st, st(1)

seg000:0C4EEE69 DD D8                                         fstp    st

seg000:0C4EEE6B 7A 23                                         jp      short loc_C4EEE90 ---nop

seg000:0C4EEE6D 77 21                                         ja      short loc_C4EEE90 ---nop

seg000:0C4EEE6F 83 EC 08                                      sub     esp, 8

seg000:0C4EEE72 56                                            push    esi

seg000:0C4EEE73 53                                            push    ebx

seg000:0C4EEE74 39 1B                                         cmp     [ebx], ebx

*/

char mem_of_fast_bulid[] = { 0xD9, 0x43, 0x58, 0xD9, 0x45, 0xF0, 0xDE, 0xC1, 0xD9, 0x5B, 0x58, 0xD9, 0x43, 0x58, 0xD9, 0x45, 0xE0, 0xDF, 0xF1, 0xDD, 0xD8, 0x7A, 0x23, 0x77, 0x21, 0x83, 0xEC, 0x08, 0x56, 0x53, 0x39, 0x1B ,0x00 };
```

对应图：

![](https://bbs.kanxue.com/upload/attach/202005/845934_V7RAA85FYSBFCFF.jpg)

## 值得注意的点

快速研究中游戏必须要有工作台才能分析，特征中不能含有`00`，我的是遍历字符串自动算长度，所以我改成了如特征中由`00`的话，需要传特征长度到函数`MemoryCmpToFeatureCode`

```c
//匹配内存中参数特征的位置

long MemoryCmpToFeatureCode(PVOID BaseAddress, long mem_size, PCHAR Strings, int isstrlen = 0)

{

 vecCodeAddr.clear();

 vecCodeAddr.reserve(200);

 int index = 0;

 int flag = -1;

 int strlens;

 //获取特征长度

 if (isstrlen == 0)

 {

  strlens = strlen(Strings);

 }

 else

 {

  strlens = isstrlen;

 }

 for (long j = 0; j < mem_size - strlens; j++)

 {

  index = 0;

  for (int i = 0; i < strlens; i++)

  {

   char* compare = (char*)BaseAddress;

   //判断内存特征与传进来的特征对比

   if (compare[j + i] != Strings[i])

   {

    break;

   }

   index++;

  }

  if (index == strlens)

  {

   vecCodeAddr.push_back(j);

   flag = 1;

  }

 }

 return flag;

}
```
