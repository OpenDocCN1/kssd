
```c++
void \_\_fastcall OO0OoOOo\_Oo0\_\_oOOoO0o0(OO0OoOOo\_Oo0\_o \*this, const MethodInfo \*method)
{
OO0OoOOo\_Oo0\_o \*v2; // x19
\_\_int64 v3; // x1
struct System\_UInt16\_array \*aa1; // x8
\_\_int64 aa5; // x9
OO0OoOOo\_oO0OoOOo\_c \*v6; // x0
int32\_t v7; // w20
struct System\_UInt16\_array \*v8; // x9
int32\_t v9; // w8
System\_Collections\_Generic\_Dictionary\_TKey\_\_TValue\_\_o \*aa8; // x0
Il2CppObject \*Item; // x0
v2 = this;
aa1 = v2->fields.aa1;
aa5 = v2->fields.aa5;
while ( 1 )
{
v6 = OO0OoOOo\_oO0OoOOo\_TypeInfo;
v7 = aa1->m\_Items[aa5 + 2];//v7 又来源于aa1
if ( v6->static\_fields->a18 == v7 )
break;
v8 = v2->fields.aa1;
v9 = v2->fields.aa5;
aa8 = (System\_Collections\_Generic\_Dictionary\_TKey\_\_TValue\_\_o \*)v2->fields.aa8;
v2->fields.aa5 = v9 + 1;
Item = System\_Collections\_Generic\_Dictionary\_int\_\_object\_\_\_get\_Item(
aa8,
v7,
(const MethodInfo\_7C771C \*)Method\_System\_Collections\_Generic\_Dictionary\_int\_\_Action\_\_get\_Item\_\_);
//Item 的来源和 v7 和 this.fields.aa8 有关
this = (OO0OoOOo\_Oo0\_o \*)((\_\_int64 (\_\_fastcall \*)(Il2CppClass \*, void \*))Item[1].monitor)(
Item[4].klass,
Item[2].monitor);//调用 Item 的函数
aa1 = v2->fields.aa1;
aa5 = v2->fields.aa5;
}
}
```

有点 vm 的意思，一个 while 循环，然后不断从表里拿东西，那么`aa5`就是`eip`，`aa1`就是`cmd`。

`cmd`在初始化中来源于第二个参数。

![](upload/attach/202304/888558_A9GWFJV8GWWXP3P.png)

第二个参数来源于一个 199 的 array，验证了猜想，可以通过动调或者 hook 函数的方式 dump 下来。

![](upload/attach/202304/888558_HTCBUHXZBYE6YUM.png)

同时`eip`又是等于`aa7`，`aa7`是初始化的第 3 个参数，为 0。

![](upload/attach/202304/888558_NFYR8MHPZTPQM6U.png)

vm 题型里有了指令，还需要解决 handler。

`ctor`中还有一个函数，似乎设置了一些函数。

![](upload/attach/202304/888558_FB4JT84HD7ZR795.png)

设置值来源于`OO0OoOOo_oO0OoOOo_TypeInfo.static_fields`，通过索引可以看见，是设置为了 1~22。

![](upload/attach/202304/888558_FRRQ6TCRFXX5YRK.png)

那就可以合理推测这里是设置了 1 ~ 22 对应的函数了。

逐个分析，然后通过指令列表回推算法即可，偷懒直接把上次做的粘贴过来。

```c
dispatcher
{
a5++;
get\_Item(key: cmd->m\_Items[a5 - 1]).method();
}
1//add
{
v6 = buffer->m\_Items[a4];
v8 = buffer->m\_Items[a4 - 1];
buffer->m\_Items[--a4] = v8 + v6;
}
2//sub
{
v6 = buffer->m\_Items[a4];
v8 = buffer->m\_Items[a4 - 1];
buffer->m\_Items[--a4] = v8 - v6;
}
4//<
{
v6 = buffer->m\_Items[a4];
v8 = buffer->m\_Items[a4 - 1];
buffer->m\_Items[--a4] = v8 < v6;
}
5//==
{
v6 = buffer->m\_Items[a4];
v8 = buffer->m\_Items[a4 - 1];
buffer->m\_Items[--a4] = v8 == v6;
}
7//je
{
if ( buffer->m\_Items[a4--] == 1 )
v7 = cmd->m\_Items[v5];
else
v7 = a5 + 1;
a5 = v7;
}
8//je
{
if ( buffer->m\_Items[a4--] )
v7 = a5 + 1;
else
v7 = cmd->m\_Items[v5];
a5 = v7;
}
9//push from cmd[]
{
buffer->m\_Items[++a4] = cmd->m\_Items[a5++];
}
a//[]
{
buffer->m\_Items[a4] = input->m\_Items[buffer->m\_Items[a4]];
}
b//push from input[cmd[]]
{
buffer->m\_Items[++a4] = input->m\_Items[cmd->m\_Items[a5++]];
}
d//pop
{
input->m\_Items[cmd->m\_Items[a5++]] = buffer->m\_Items[a4--];
}
13//>>
{
v6 = buffer->m\_Items[a4];
v8 = buffer->m\_Items[a4 - 1];
buffer->m\_Items[--a4] = v8 >> v6;
}
14//<<
{
v6 = buffer->m\_Items[a4];
v8 = buffer->m\_Items[a4 - 1];
buffer->m\_Items[--a4] = v8 >> v6;
}
15//and
{
v6 = buffer->m\_Items[a4];
v8 = buffer->m\_Items[a4 - 1];
buffer->m\_Items[--a4] = v6 & v8;
}
16//xor
{
v6 = buffer->m\_Items[a4];
v8 = buffer->m\_Items[a4 - 1];
buffer->m\_Items[--a4] = v6 ^ v8;
}
```

具体推导算法过程我也偷个懒不重复推导了，到此所有算法都出来了，直接贴计算脚本。

```c
#include<stdio.h>
#include <stdlib.h>
void vmDecode(int\* flag)
{
for(int i = 0; i <= 3; ++i)
((char\*)flag)[4 + i] -= i \* 8;
((char\*)flag)[7] ^= 0x98;
((char\*)flag)[6] -= 0x37;
((char\*)flag)[5] ^= 0xb6;
((char\*)flag)[4] += 0x2f;
for(int i = 0; i <= 3; ++i)
((char\*)flag)[i] ^= i \* 8;
((char\*)flag)[3] ^= 0x36;
((char\*)flag)[2] -= 0xa8;
((char\*)flag)[1] ^= 0xc2;
((char\*)flag)[0] += 0x1b;
}
void Decode2(int\* flag)
{
char temp[] = {50, -51, -1, -104, 25, -78, 0x7C, -102};
for(int j = 0; j < 2; ++j)
{
unsigned int v7 = flag[j];
for(int i = 0; i < 4; ++i)
{
((char \*)&v7)[i] -= i;
((char \*)&v7)[i] ^= temp[i % 8];
}
char v77[4] = {};
v77[0] = ((char \*)&v7)[3];
v77[1] = ((char \*)&v7)[2];
v77[2] = ((char \*)&v7)[1];
v77[3] = ((char \*)&v7)[0];
v7 = \*(int\*)v77;
//printf("0x%x\n", v7);
v7 = (unsigned int)(v7 << 7) | (v7 >> 25);
v77[0] = ((char \*)&v7)[3];
v77[1] = ((char \*)&v7)[2];
v77[2] = ((char \*)&v7)[1];
v77[3] = ((char \*)&v7)[0];
v7 = \*(int\*)v77;
//printf("0x%x\n", v7);
flag[j] = v7;
}
}
void Decode1(unsigned int\* flag)
{
for(int j = 0; j < 2; ++j)
{
unsigned int v7 = flag[j];
//v7 += 0x18100800;
((unsigned char \*)&v7)[0] += 0x00;
((unsigned char \*)&v7)[1] += 0x08;
((unsigned char \*)&v7)[2] += 0x10;
((unsigned char \*)&v7)[3] += 0x18;
((unsigned char \*)&v7)[3] ^= 0x86;
((unsigned char \*)&v7)[2] += 94;
((unsigned char \*)&v7)[1] ^= 0xD3;
((unsigned char \*)&v7)[0] += 28;
for(int i = 3; i >= 0; --i)
{
((unsigned char\*)flag)[j \* 4 + i] = (unsigned char)(v7 >> (i \* 8)) ^ i;
}
}
}
void ObfuDecode(unsigned int\* flag)//sub\_3B8CC
{
vmDecode(flag);
Decode2(flag);//sub\_3A054
char v77[4] = {};
v77[0] = ((char \*)flag)[3];
v77[1] = ((char \*)flag)[2];
v77[2] = ((char \*)flag)[1];
v77[3] = ((char \*)flag)[0];
flag[0] = \*(int\*)v77;
v77[0] = ((char \*)flag)[7];
v77[1] = ((char \*)flag)[6];
v77[2] = ((char \*)flag)[5];
v77[3] = ((char \*)flag)[4];
flag[1] = \*(int\*)v77;
Decode1(flag);//sub\_3B9D4
}
int\* DecodeTea(int RandNum)
{
unsigned int v20 = 0xBEEFBEEF - 0x21524111 \* 64;
unsigned int v21 = 0x9D9D7DDE - 0x21524111 \* 64;
int v22 = 64;
char key\_char[] = {0x63,0x7c,0x77,0x7b,0xf2,0x6b,0x6f,0xc5,0x30,0x01,0x67,0x2b,0xfe,0xd7,0xab,0x76};
int\* key = (int\*)key\_char;
unsigned int v16 = RandNum;
unsigned int v17 = 0;
do
{
v21 += 0x21524111;
v17 -= (v21 + key[(v21 >> 13) & 3]) ^ (((v16 << 8) ^ (v16 >> 7)) - v16);
v20 += 0x21524111;
v16 -= (v20 - key[v20 & 3]) ^ (((v17 << 7) ^ (v17 >> 8)) + v17);
--v22;
}
while ( v22 );
unsigned int\* temp = (int\*)malloc(8);
temp[0] = v16;
temp[1] = v17;
return temp;
}
int main()
{
printf("pls input rand:");
int rand;
scanf("%d", &rand);
int\* temp = DecodeTea(rand);
//unsigned int\* temp = malloc(10);
//temp[0] = 0x4d09f96f;
//temp[1] = 0x2d55b70a;
ObfuDecode(temp);
printf("%llu\n", (((unsigned long long)temp[0]<<32) | (temp[1] & 0xffffffff)));
free(temp);
}
```

# 后记

决赛的放下一篇明天再写

# 前言

虽然这次很不走运没拿名次，但是既然都花了这么长时间，还是得好好复现总结总结所学，尤其是才接触安卓两三个月，基础有点不牢固，于是准备重做一遍并记录下过程，分享出来与各逆向大佬共勉。

# 预备工作

## 启发式扫描

拿到题目首先用 WinRAR 打开，看 lib：

- `lib\arm64-v8a\libil2cpp.so`
- `lib\arm64-v8a\libil2cpp.so.merged.result`
- `lib\arm64-v8a\libmain.so`
- `lib\arm64-v8a\libsec2023.so`
- `lib\arm64-v8a\libunity.so`

说明这个是一个 unity il2cpp 框架下的题目，那么立马就想到 [il2cppdumpper](https://bbs.kanxue.com/elink@1a2K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6b7k6i4u0X3j5i4u0W2i4K6u0r3d9h3H3J5b7%4m8H3c8s2g2E0M7r3g2J5)。

提取文件夹下的`assets\bin\Data\Managed\Metadata\global-metadata.dat`。

不出所料失败了：

```
Initializing metadata...
Metadata Version: 29
Initializing il2cpp file...
Applying relocations...
WARNING: find .init_proc
ERROR: This file may be protected.
Il2Cpp Version: 29
Detected this may be a dump file.
Input il2cpp dump address or input 0 to force continue:
0
Searching...
CodeRegistration : 0
MetadataRegistration : 0
ERROR: No symbol is detected
ERROR: Can't use auto mode to process file, try manual mode.
Input CodeRegistration:
```

尝试使用 [Zygisk-Il2CppDumper](https://bbs.kanxue.com/elink@f35K9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6Y4K9i4c8Z5N6h3u0Q4x3X3g2U0L8$3#2Q4x3V1k6b7k6i4u0X3j5i4u0W2i4K6u0r3h3Y4W2Y4K9i4y4C8i4K6u0V1d9h3H3J5b7%4m8H3c8s2g2E0M7r3g2J5)，也失败了，那么到此只能使用终极 dump 方案。

`adb install` apk 之后用 gg 在内存中 dump 解密之后的`libil2cpp.so`。

dump 过程如下，选择起始：

![](https://bbs.kanxue.com/upload/attach/202304/888558_65RHZ9Y9DGQUSG9.png)

以及结尾，然后保存就可以了：

![](https://bbs.kanxue.com/upload/attach/202304/888558_DQXFA9E5YZY7QXM.png)
