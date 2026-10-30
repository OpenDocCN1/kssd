
这个注册码一共经历了四次变换（`First`、`Second`、`Third`、`Fourth`）。

其中，`First` 运算可逆，`Second`、`Third`、`Fourth` 都不是可逆的函数。

观察一下计算后的数据，容易发现注册码被分成了如下的五个部分：

![这里写图片描述](upload/attach/202002/759802_TBKA8RPK9DQMDEU.jpg)

其中有高亮的（第二个到第四个）都是被替换后的，而没有高亮的（第一个和最后一个）都是不被替换的，因此没有高亮的部分可以直接用 `First` 运算来还原。

而有高亮的部分我没有找到好的办法去还原，只有枚举了。

总体的计算思路如下：

1. 先不考虑没有高亮的部分（第一个和最后一个），枚举第二个，用 `Second` 运算来枚举，尝试一个字节一个字节枚举，考虑到注册码都是可显示字符，`scanf` 又有遇到空格截断的特性，因此从 `0x21` 开始枚举。若 `Second` 返回的数据是正确的，则开始第二个字节的枚举。
2. 枚举第三个，用 `Third` 运算来枚举，尝试一个字节一个字节枚举，若 `Third` 返回的数据是正确的，则开始第二个字节的枚举。
3. 枚举第四个，用 `Fourth` 运算来枚举，尝试一个字节一个字节枚举，若 `Fourth` 返回的数据是正确的，则开始第二个字节的枚举。
4. 这步中的数据应该是正确的 `Key` 经过 `First` 运算后的数据了，因为 `First` 运算是 `xor` 运算，故将这部分的数据再用 `First` 运算一次即可得到 `Key`。

计算代码如下：

```
#include <stdio.h>
#include <stdlib.h>
void First(unsigned char *Output,unsigned char *Input,int size)//Key的第一次变换
{
for(int i = 0;i < size;i++)
Output[i] = Input[i] ^ 0x76;
}
void Second(unsigned char *Output,unsigned char *Input,int size)//Key的第二次变换
{
if(size > 7)
return;
for(int i = 0;i < size;i++)
{
int tmp;
int tmp1;
Output[i] = Input[i] ^ 0xAD;
tmp = (Output[i] & 0xAA) >> 1;
tmp1 = 2 * Output[i] & 0xAA;
Output[i] = tmp1 | tmp;
}
}
void Third(unsigned char *Output,unsigned char *Input,int size)//Key的第三次变换
{
if(size > 7)
return;
for(int i = 0;i < size;i++)
{
int tmp;
int tmp1;
Output[i] = Input[i] ^ 0xBE;
tmp = (Output[i] & 0xCC) >> 2;
tmp1 = 4 * Output[i] & 0xCC;
Output[i] = tmp1 | tmp;
}
}
void Fourth(unsigned char *Output,unsigned char *Input,int size)//Key的第四次变换
{
if(size > 7)
return;
for(int i = 0;i < size;i++)
{
int tmp;
int tmp1;
Output[i] = Input[i] ^ 0xEF;
tmp = (Output[i] & 0xF0) >> 4;
tmp1 = 16 * Output[i] & 0xF0;
Output[i] = tmp1 | tmp;
}
}
int main(void)
{
int j;
unsigned char ch;
unsigned char Data1[] = //第二个key
{
0x1E,0x15,0x02,0x10,0x0D,0x48,0x48,
0x6F,0xDD,0xDD,0x48,0x64,0x63,0xD7,
0x2E,0x2C,0xFE,0x6A,0x6D,0x2A,0xF2,
0x6F,0x9A,0x4D,0x8B,0x4B,0x9A,0xAA,
0x41,0x42,0x42,0x42,0x13,0x14,0x0B
};
/*
计算第二个Key
这部分的Key一共有35个字节,若每七个字节是一块则一共有五块.
*/
j = 0;//第二块
ch = 0x21;
while(true)
{
unsigned char a;
if(j == 7)
break;
Second(&a,&ch,1);//第二次变换
if(a != Data1[j + 7])
{
ch++;
continue;
}
Data1[j + 7] = ch;
ch = 0x21;
j++;
}
j = 0;//第三块
ch = 0x21;
while(true)
{
unsigned char a;
if(j == 7)
break;
Third(&a,&ch,1);//第三次变换
if(a != Data1[j + 14])
{
ch++;
continue;
}
Data1[j + 14] = ch;
ch = 0x21;
j++;
}
j = 0;//第四块
ch = 0x21;
while(true)
{
unsigned char a;
if(j == 7)
break;
Fourth(&a,&ch,1);//第四次变换
if(a != Data1[j + 21])
{
ch++;
continue;
}
Data1[j + 21] = ch;
ch = 0x21;
j++;
}
First(Data1,Data1,sizeof(Data1));//第一次变换
/* Output:hctf{>>D55_CH0CK3R_B0o0M!-037444eb} */
printf("%0.35s\n",Data1);
system("pause");
return 0;
}
```

运行以后得到结果：`hctf{>>D55_CH0CK3R_B0o0M!-037444eb}`

# 4x0 最后一个输入

这个就很简单了，问你输入 `Y` 或者 `N`，若输入 `Y` 则成功（不区分大小写），否则失败。

![这里写图片描述](upload/attach/202002/759802_NGM8BTMFBPPEE52.jpg)

## 4x1 完整的计算代码

将上述代码写到一起，然后再加上输出最后一个注册码的代码即可。

代码如下：

```
#include <stdio.h>
#include <stdlib.h>
void First(unsigned char *Output,unsigned char *Input,int size)//Key的第一次变换
{
for(int i = 0;i < size;i++)
Output[i] = Input[i] ^ 0x76;
}
void Second(unsigned char *Output,unsigned char *Input,int size)//Key的第二次变换
{
if(size > 7)
return;
for(int i = 0;i < size;i++)
{
int tmp;
int tmp1;
Output[i] = Input[i] ^ 0xAD;
tmp = (Output[i] & 0xAA) >> 1;
tmp1 = 2 * Output[i] & 0xAA;
Output[i] = tmp1 | tmp;
}
}
void Third(unsigned char *Output,unsigned char *Input,int size)//Key的第三次变换
{
if(size > 7)
return;
for(int i = 0;i < size;i++)
{
int tmp;
int tmp1;
Output[i] = Input[i] ^ 0xBE;
tmp = (Output[i] & 0xCC) >> 2;
tmp1 = 4 * Output[i] & 0xCC;
Output[i] = tmp1 | tmp;
}
}
void Fourth(unsigned char *Output,unsigned char *Input,int size)//Key的第四次变换
{
if(size > 7)
return;
for(int i = 0;i < size;i++)
{
int tmp;
int tmp1;
Output[i] = Input[i] ^ 0xEF;
tmp = (Output[i] & 0xF0) >> 4;
tmp1 = 16 * Output[i] & 0xF0;
Output[i] = tmp1 | tmp;
}
}
int main(void)
{
int j;
unsigned char ch;
unsigned char Data[] = {0xA4,0xA9,0xAA,0xBE,0xBC,0xB9,0xB3,0xA9,0xBE,0xD8,0xBE};//第一个key
unsigned char Data1[] = //第二个key
{
0x1E,0x15,0x02,0x10,0x0D,0x48,0x48,
0x6F,0xDD,0xDD,0x48,0x64,0x63,0xD7,
0x2E,0x2C,0xFE,0x6A,0x6D,0x2A,0xF2,
0x6F,0x9A,0x4D,0x8B,0x4B,0x9A,0xAA,
0x41,0x42,0x42,0x42,0x13,0x14,0x0B
};
/* 计算第一个key */
for(int i = 0;i < sizeof(Data);i++)
Data[i] = (((((i ^ 0x76) - 52) ^ 0x80) + 43) ^ Data[i]);
for(int i = 0;i < sizeof(Data) / 2;i++)
{
Data[i] ^= Data[sizeof(Data) - i - 1];
Data[sizeof(Data) - i - 1] ^= Data[i];
Data[i] ^= Data[sizeof(Data) - i - 1];
}
/* Output:M.KATSURAGI */
printf("%0.11s\n",Data);
/*
计算第二个Key
这部分的Key一共有35个字节,若每七个字节是一块则一共有五块.
*/
j = 0;//第二块
ch = 0x21;
while(true)
{
unsigned char a;
if(j == 7)
break;
Second(&a,&ch,1);//第二次变换
if(a != Data1[j + 7])
{
ch++;
continue;
}
Data1[j + 7] = ch;
ch = 0x21;
j++;
}
j = 0;//第三块
ch = 0x21;
while(true)
{
unsigned char a;
if(j == 7)
break;
Third(&a,&ch,1);//第三次变换
if(a != Data1[j + 14])
{
ch++;
continue;
}
Data1[j + 14] = ch;
ch = 0x21;
j++;
}
j = 0;//第四块
ch = 0x21;
while(true)
{
unsigned char a;
if(j == 7)
break;
Fourth(&a,&ch,1);//第四次变换
if(a != Data1[j + 21])
{
ch++;
continue;
}
Data1[j + 21] = ch;
ch = 0x21;
j++;
}
First(Data1,Data1,sizeof(Data1));//第一次变换
/* Output:hctf{>>D55_CH0CK3R_B0o0M!-037444eb} */
printf("%0.35s\n",Data1);
/*
计算第三个key
Output:y
*/
puts("y");
system("pause");
return 0;
}
```

运行以后得到完整的 `Key`：

![这里写图片描述](upload/attach/202002/759802_2KR52KX2UZ96HVF.jpg)

用原版 CM 注册，得到结果如下：

![这里写图片描述](upload/attach/202002/759802_Y99V4XAD6RP4JKC.jpg)

全文完。

附件中的 `RuntimeLibrary.zip` 是 CM 的运行库，作者没有打包在 CM 中，如果打开 CM 提示缺少 `DLL` 的话就下载 `RuntimeLibrary.zip` 中的文件解压到 CM 目录下。

# 0x0 初探

先用 PEID 查看一下 CM 的信息。

![这里写图片描述](https://bbs.kanxue.com/upload/attach/202002/759802_BXHQHTMA6AQZKNA.jpg)

![这里写图片描述](https://bbs.kanxue.com/upload/attach/202002/759802_XM8JXXQQTEP5B2J.jpg)

从图中可以发现，程序使用 Debug 的编译方式编译且有一个 TLS 回调函数。

因为程序有 ASLR，不方便分析，因此我用 FFI 去掉后再继续分析。

![这里写图片描述](https://bbs.kanxue.com/upload/attach/202002/759802_3KJVU2FXMYTGHPR.jpg)

# 1x0 分析TLS回调函数

OD 载入，断在 TLS 入口处。

![这里写图片描述](https://bbs.kanxue.com/upload/attach/202002/759802_UD3A4PTXPQBK6FE.jpg)

一共有四个函数，其中前两个我们需要关注一下。因为这两个函数是反调试的函数。

## 1x1 分析函数 0041141A

CM 先创建一个进程快照。

![这里写图片描述](https://bbs.kanxue.com/upload/attach/202002/759802_KSRGGMCHHY73UXX.jpg)

然后就开始枚举进程，取出进程名，如果进程名字符合如下进程名中的其中一个，就直接报错退出。

> `ollyice.exe`
> `ollydbg.exe`
> `peid.exe`
> `ida.exe`
> `idaq.exe`

对应的检测代码如下：
