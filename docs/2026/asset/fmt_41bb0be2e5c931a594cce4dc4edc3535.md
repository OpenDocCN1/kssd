
![图片描述](upload/attach/202009/590753\_3ZKG8NA22XFHKF5.png)

这里需要动态调试获取值

![图片描述](upload/attach/202009/590753\_K2PUUV9GRUDMPX7.png)

调试的时候断点参考 `loc_11B0` `loc_10AA`

## (2) 加密算法

加密过程

```
data1 = [0x11, 0x22, 0x33, 0x44, 0x55, 0x66, 0x77, 0x88]
data2 = [0x77, 0x23, 0x9D, 0xAC, 0x13, 0x27, 0xCF, 0xFE]
data3 = [0xA8, 0xAF, 0x56, 0x98, 0x88, 0xEF, 0x40, 0x06, 0xFD, 0xAE, 0xE9, 0x9E, 0xB9, 0xEA, 0xAD, 0x52,
0xCC, 0xAB, 0x04, 0xCA, 0xEC, 0xEB, 0x12, 0x54, 0x99, 0xAA, 0xBB, 0xCC, 0xDD, 0xEE, 0xFF, 0x00,
0xA8, 0xAF, 0x56, 0x98, 0x88, 0xEF, 0x40, 0x06, 0xFD, 0xAE, 0xE9, 0x9E, 0xB9, 0xEA, 0xAD, 0x52,
0xCB, 0xAA, 0x19, 0xE5, 0xEC, 0xEB, 0x12, 0x5E, 0xA0, 0xA6, 0xA8, 0xF3, 0x8B, 0xD6, 0xCD, 0x6F,
0xCB, 0xC0, 0x4F, 0xC1, 0xE0, 0xDA, 0x74, 0x00, 0x91, 0xBE, 0xC6, 0x83, 0xD0, 0xA6, 0x8D, 0x2E,
0xBE, 0xFC, 0x3F, 0xAD, 0x9B, 0xE0, 0x26, 0x52, 0xF5, 0xBA, 0x94, 0xD1, 0xB4, 0xA2, 0xDF, 0x7C,
0xDA, 0xF8, 0x6D, 0xFF, 0xFF, 0xE4, 0x74, 0x00, 0x91, 0xBE, 0xC6, 0x83, 0xD0, 0xA6, 0x8D, 0x2E,
0xBE, 0xFC, 0x3F, 0xAD, 0x9B, 0xE0, 0x26, 0x52, 0xF5, 0xBA, 0x94, 0xD1, 0xB4, 0xA2, 0xDF, 0x7C]
```

1、输入字符串

2、按首字符填充长度为首字符10进制长度 + 字符串长度 + 尾字符填充 = 128长度

3、循环128位字符 先与`data1`和`data2`进行异或，再循环 28x16（1-7 6-0） 次 进行高/低四位保留（0xf0 0xf 与运算），最后异或0x88

4、结果与`data3`进行比较

## (3) 算法还原

**加密**

```
int sub_F34(char *s){
char *v1; // r8
char *v2;
size_t v3; // r0
int v4; // r9
unsigned int v5; // r3
int v6; // r0
int v7; // r1
char *v8; // r2
int v9; // r4
signed int v10; // r3
int v11; // r6
int v12; // r10
unsigned int v13; // r6
int v14; // r3
char *v15; // r11
char *v16;
int i; // r0
unsigned int v17; // r1
int v18; // r10
char *v19; // r2
int v20; // t1
int *v21; // r2
int v22; // lr
int v23; // r1
char *v24; // r8
int v25; // r12
int j; // r4
int k; // r4
int v28; // r9
int v29; // r5
int v30; // r1
int v31; // r4
int v32; // r4
int v33; // r5
int v34; // r0
int v59;
v1 = s;
v3 = strlen(s);
if ( v3 <= 128 )
{
v4 = v3;
v5 = 0;
v6 = 0x1021;
v7 = v4;
v8 = v1;
while ( v7 >= 1 )
{
v9 = v5 ^ (*v8 << 8);
v10 = 8;
while ( v10 )
{
v11 = v6 ^ 2 * v9;
if ( !(v9 & 0x8000) )
v11 = 2 * v9;
--v10;
v9 = v11;
}
--v7;
v5 = v9;
++v8;
}
v59 = 128;
v12 = v5 & 0xF;
v13 = (v5 >> 8) & 0x3F;
v15 = (char *)malloc(128);
for ( i = 0; v13 != i; ++i )
v15[i] = *v1;
v17 = v13 + v4;
v18 = v12 + 16;
v19 = v1;
printf("before %s %d \n",v15, strlen(v15));
while ( v13 < v17 )
{
v20 = *v19++;
v14 = v20;
v15[v13++] = v20;
}
printf("input %s %d \n",v15, strlen(v15));
while ( v13 <= 0x7F )
v15[v13++] = v1[v4 - 1];
printf("after %s %d \n",v15, strlen(v15));
unsigned char byte_2B98[8] = {0x77, 0x23, 0x9D, 0xAC, 0x13, 0x27, 0xCF, 0xFE};
unsigned char v21[8] = {0x11, 0x22, 0x33, 0x44, 0x55, 0x66, 0x77, 0x88};
while ( v22 != 16 )
{
v25 = 8 * v22;
for ( j = 0; j != 8; ++j )
v21[j] ^= v15[v25 + j];
for ( k = 0; k != 8; ++k )
*(v21 + k) ^= byte_2B98[k];
v28 = 0;
while ( v28 != v18 )
{
v14 = v21[7];
v29 = 7;
while ( v29 >= 1 )
{
v21[v29] = (v21[v29]&0xf0) + (v21[v29-1]&0xf);
--v29;
}
v32 = 0;
v33 = v21[0] & 0xF0;
v21[0] = (v14 & 0xF) + v33;
v34 = v21[0];
while ( v32 != 7 )
{
v21[v32] = (v21[v32]&0xf) + (v21[v32+1]&0xf0);
++v32;
}
v21[7] = (v21[7]&0xf) + (v34&0xf0);
++v28;
}
v30 = 0;
while(v30 != 8){
printf("0x%x ", v21[v30] ^ 0x88);
++v30;
}
printf("\n");
++v22;
}
}
return 0;
}
```

**解密**

```
void decrypt(){
char *v1; // r8
char *v2;
size_t v3; // r0
int v4; // r9
unsigned int v5; // r3
int v6; // r0
int v7; // r1
char *v8; // r2
int v9; // r4
signed int v10; // r3
int v11; // r6
int v12; // r10
unsigned int v13; // r6
int v14; // r3
char *v15; // r11
char *v16;
int i; // r0
unsigned int v17; // r1
int v18; // r10
char *v19; // r2
int v20; // t1
int v22; // lr
int v23; // r1
char *v24; // r8
int v25; // r12
int j; // r4
int k; // r4
int v28; // r9
int v29; // r5
int v32; // r4
int v33; // r5
int v34; // r0
unsigned char byte_2B98[8] = {0x77, 0x23, 0x9D, 0xAC, 0x13, 0x27, 0xCF, 0xFE};
unsigned char v21[8] = {0x11, 0x22, 0x33, 0x44, 0x55, 0x66, 0x77, 0x88};
unsigned char unk_2BA0[128] = {
0xA8, 0xAF, 0x56, 0x98, 0x88, 0xEF, 0x40, 0x06, 0xFD, 0xAE, 0xE9, 0x9E, 0xB9, 0xEA, 0xAD, 0x52,
0xCC, 0xAB, 0x04, 0xCA, 0xEC, 0xEB, 0x12, 0x54, 0x99, 0xAA, 0xBB, 0xCC, 0xDD, 0xEE, 0xFF, 0x00,
0xA8, 0xAF, 0x56, 0x98, 0x88, 0xEF, 0x40, 0x06, 0xFD, 0xAE, 0xE9, 0x9E, 0xB9, 0xEA, 0xAD, 0x52,
0xCB, 0xAA, 0x19, 0xE5, 0xEC, 0xEB, 0x12, 0x5E, 0xA0, 0xA6, 0xA8, 0xF3, 0x8B, 0xD6, 0xCD, 0x6F,
0xCB, 0xC0, 0x4F, 0xC1, 0xE0, 0xDA, 0x74, 0x00, 0x91, 0xBE, 0xC6, 0x83, 0xD0, 0xA6, 0x8D, 0x2E,
0xBE, 0xFC, 0x3F, 0xAD, 0x9B, 0xE0, 0x26, 0x52, 0xF5, 0xBA, 0x94, 0xD1, 0xB4, 0xA2, 0xDF, 0x7C,
0xDA, 0xF8, 0x6D, 0xFF, 0xFF, 0xE4, 0x74, 0x00, 0x91, 0xBE, 0xC6, 0x83, 0xD0, 0xA6, 0x8D, 0x2E,
0xBE, 0xFC, 0x3F, 0xAD, 0x9B, 0xE0, 0x26, 0x52, 0xF5, 0xBA, 0x94, 0xD1, 0xB4, 0xA2, 0xDF, 0x7C
};
unsigned char flag[128];
for(int i=0;i<16;i++){
unsigned char data[8];
unsigned char data1[8];
for(int j=0;j<8;j++){
data[j] = unk_2BA0[i*8 + j] ^ 0x88;
data1[j] = data[j];
}
v28 = 0;
while ( v28 != 28 )
{
v34 = data[0];
v32 = 0;
while ( v32 != 7 )
{
data[v32] = (data[v32]&0xf) + (data[v32+1]&0xf0);
++v32;
}
data[7] = (data[7]&0xf) + (v34&0xf0);
v14 = data[7];
v29 = 7;
while ( v29 >= 1 )
{
data[v29] = (data[v29]&0xf0) + (data[v29-1]&0xf);
--v29;
}
v33 = data[0] & 0xF0;
data[0] = (v14 & 0xF) + v33;
++v28;
}
for(j=0; j<8; j++) {
data[j] ^= byte_2B98[j];
data[j] ^= v21[j];
v21[j] = data1[j];
printf("%c", data[j]);
}
}
}
```

运行结果：

![图片描述](upload/attach/202011/590753\_XJE7CKPE3QZ3D44.png)

## 0x03 总结

1、识别花指令

2、对Jumpout进行修复

3、动态调试

4、算法还原

## 0x00 前言

小伙伴给我发来一道android题，涉及花指令，记录一下去除花指令和算法还原的过程。

### 样本

强网杯 `flower.apk`

## 0x01 花指令

花指令是企图隐藏掉不想被逆向工程的代码块(或其它功能)的一种方法，在真实代码中插入一些垃圾代码的同时还保证原有程序的正确执行，而程序无法很好地反编译, 难以理解程序内容，达到混淆视听的效果。

**主要目的加大静态分析难度**

#### 1、不可执行的花指令

运行是不执行的汇编指令，会对反汇编造成影响，影响静态分析（如ida无法f5）

#### 2、可执行的花指令

可执行花指令一般都是无意义的汇编指令，会被反汇编正常识别

参考：

eb2K9s2c8@1M7q4)9K6b7g2)9J5c8W2)9J5c8X3c8&6k6W2)9J5k6h3W2F1K9#2)9J5c8Y4u0W2N6X3g2J5M7$3g2Q4x3V1k6%4K9h3&6V1L8%4N6K6i4K6u0r3j5h3&6@1K9g2)9J5k6r3c8W2j5Y4g2Y4i4K6u0r3K9Y4g2F1K9#2)9J5k6r3y4G2k6r3g2Q4x3V1j5`.

e1cK9s2c8@1M7s2y4Q4x3@1q4Q4x3V1k6Q4x3V1k6%4N6%4N6Q4x3X3g2S2L8Y4q4#2j5h3&6C8k6g2)9J5k6h3y4G2L8g2)9J5c8Y4m8G2M7%4c8Q4x3V1k6A6k6q4)9J5c8U0t1H3z5o6j5^5x3R3`.`.

## 0x02 算法还原

### 1、样本分析

#### (1) java层

![图片描述](https://bbs.kanxue.com/upload/attach/202009/590753_92EMSVDEKURZZ3N.png)

#### (2) native层

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14 | `int` `__fastcall Java_com_a_flower_MainActivity_check(` `int` `a1,` `int` `a2,` `int` `a3)`  `{`  `int` `v3;` `/``/` `r5`  `int` `v4;` `/``/` `r4`  `char` `*``v5;` `/``/` `r6`  `int` `v6;` `/``/` `r8`    `v3` `=` `a1;`  `v4` `=` `a3;`  `v5` `=` `(``*``(``*``a1` `+` `676``))(a1, a3,` `0``);`  `v6` `=` `sub_F34(v5);`  `(``*``(``*``v3` `+` `680``))(v3, v4, v5);`  `return` `v6;`  `}` |

###### sub_F34

|  |  |
| --- | --- |
| 1  2  3  4  5  6  7  8  9  10  11  12  13  14  15  16  17  18  19 | `int` `__fastcall sub_F34(char` `*``s)`  `{`  `char` `*``v1;` `/``/` `r8`  `size_t v2;` `/``/` `r0`  `int` `v3;` `/``/` `r4`    `v1` `=` `s;`  `v2` `=` `strlen(s);`  `if` `( v2 <``=` `0x80` `)`  `{`  `if` `( (signed` `int``)v2 >``=` `1` `)`  `{`  `v3` `=` `(unsigned __int8)``*``v1 <<` `8``;`  `sub_F8E((``int``)&dword_0` `+` `3``);`  `}`  `sub_FD0((``int``)&dword_0` `+` `3``);`  `}`  `return` `1``;`  `}` |

`sub_F8E`

 ![图片描述](https://bbs.kanxue.com/upload/attach/202009/590753_NM4W3HHPT8P3D8C.png)

`sub_FD0`

 ![图片描述](https://bbs.kanxue.com/upload/attach/202009/590753_RYX68CSTB9HY8WW.png)

遇到JUMPOUT要进行path修复

JUMPOUT修复参考：https://bbs.pediy.com/thread-259062.htm

###### 修复 sub_F8E

根据ida的提示`sub_F8E`应该跳转的位置是 `0xFA4` = `0xF98` + `0xC`

patch脚本

|  |  |
| --- | --- |
