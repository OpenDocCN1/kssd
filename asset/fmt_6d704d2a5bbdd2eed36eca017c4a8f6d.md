
# 第三题，抛砖引玉

> 原文：[`www.pediy.com/kssd/pediy11/123370.html`](https://www.pediy.com/kssd/pediy11/123370.html)

验证过程：

1. 计算注册码的格式：`xxxxxxxx-xxxxxxxx-xxxxxxxx-xxxxxxxx`以及字符集合法性，见如下`letter`数组
2. 一些移位操作计算出一个20字节的`HASH`值
3. 对输入的用户名，后面追加`C`盘逻辑卷的序列号以及一个字符串`"Tencent"`，用变形的类`SHA1`算法计算一个`HASH`值
4. 比较前后两个，相同则`ok`

能看到注册码的格式把注册码分为四部分，其中每一组`8Byte`计算出`5`个`BYTE`，对应于用户名的`HASH`值。

计算部分如下，用户名那部分的代码就不贴了，抠出来就是。

`DWORD` `USER_HASH[5]`; //这里记录的是用户名+VolumeSerialNumber+Tencent 最终生成的 hash
`char` `letter[]` = `"ABCDEFGHJKMNPQRSTVWXYZ1234567890"`;
`char` `reg_code[4][9]` = `{0}`;
`void` `CalcReg()`

```
{
    bool succ = false;
    BYTE* byte_USER_HASH = (BYTE*)USER_HASH;
    for(int i=0; i<4; i++)
    {
        int pos = i*5;
        succ = false;
        for(int a0=0; a0<32 && !succ; a0++)
        {
            for(int a1=0; a1<32 && !succ; a1++)
            {
                if ((((a0<<3)|(a1>>2))&0xFF) != byte_USER_HASH[pos])
                    continue;
                for(int a2=0; a2<32 && !succ; a2++)
                {
                    for(int a3=0; a3<32 && !succ; a3++)
                    {
                        if ((((a1<<6)|(a2<<1)|(a3>>4))&0xFF) != byte_USER_HASH[pos+1])
                            continue;
                        for(int a4=0; a4<32 && !succ; a4++)
                        {
                            if ((((a3<<4)|(a4>>1))&0xFF)!= byte_USER_HASH[pos+2])
                                continue;
                            for(int a5=0; a5<32 && !succ; a5++)
                            {
                                for(int a6=0; a6<32 && !succ; a6++)
                                {
                                    if ((((a4<<7)|(a5<<2)|(a6>>3))&0xFF) != byte_USER_HASH[pos+3])
                                        continue;
                                    for(int a7=0; a7<32; a7++)
                                    {
                                        if((((a6<<5)|a7)&0xFF) == byte_USER_HASH[pos+4])
                                        {
                                            reg_code[i][0] = letter[a0];
                                            reg_code[i][1] = letter[a1];
                                            reg_code[i][2] = letter[a2];
                                            reg_code[i][3] = letter[a3];
                                            reg_code[i][4] = letter[a4];
                                            reg_code[i][5] = letter[a5];
                                            reg_code[i][6] = letter[a6];
                                            reg_code[i][7] = letter[a7];
                                            succ = true;
                                            break;
                                        }
                                    }
                                }
                            }
                        }
                    }
                }
            }
        }
    }
}
```

我也来一段计算的代码

代码:

```
void GetCode(unsigned char p[20], char code[32])
{
  int ebp = 0;
  int ebx = 0;
  int esi = 0;
  char str[33] = "ABCDEFGHJKMNPQRSTVWXYZ1234567890";

  int flag[][2] =
  {

    {0xF8, -3},

    {0x07,  2}, //+
    {0xC0, -6},

    {0x3E, -1},

    {0x01,  4}, //+
    {0xF0, -4},

    {0x0F,  1}, //+
    {0x80, -7},

    {0x7C, -2},

    {0x03,  3}, //+
    {0xE0, -5},

    {0x1F,  0}, //+

    {0xF8, -3},

    {0x07,  2}, //+
    {0xC0, -6},

    {0x3E, -1},

    {0x01,  4}, //+
    {0xF0, -4},

    {0x0F,  1}, //+
    {0x80, -7},

    {0x7C, -2},

    {0x03,  3}, //+
    {0xE0, -5},

    {0x1F,  0}, //+

    {0xF8, -3},

    {0x07,  2}, //+
    {0xC0, -6},

    {0x3E, -1},

    {0x01,  4}, //+
    {0xF0, -4},

    {0x0F,  1}, //+
    {0x80, -7},

    {0x7C, -2},

    {0x03,  3}, //+
    {0xE0, -5},

    {0x1F,  0}, //+

    {0xF8, -3},

    {0x07,  2}, //+
    {0xC0, -6},

    {0x3E, -1},

    {0x01,  4}, //+
    {0xF0, -4},

    {0x0F,  1}, //+
    {0x80, -7},

    {0x7C, -2},

    {0x03,  3}, //+
    {0xE0, -5},

    {0x1F,  0}, //+
  };

  int t20 = 0;
  int t32 = 0;
  int da = 0;
  while (esi<20)
  {
    t20 |= flag[ebx][0];
    t32 |= flag[ebx][1]>0?(flag[ebx][0]<<flag[ebx][1]):(flag[ebx][0]>>-flag[ebx][1]);
    int t = (flag[ebx][0]&p[esi]);
    da |= flag[ebx][1]>0?(t<<flag[ebx][1]):(t>>-flag[ebx][1]);

    if (t20==0xff)
    {
      t20 = 0;
      esi++;
    }
    if (t32==0x1f)
    {
      code[ebp] = str[da];
      ebp++;
      da = 0;
      t32 = 0;
    }

    ebx++;
  }

}
```

完全扣出来的代码，虽然不知道它在干什么![](img/234a76e67df951b6551d305c177a8b78.jpg "扮鬼脸")

代码:
