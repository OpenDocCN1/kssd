
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



```char  * SHA(char *UserName, char *Key ,int len)
{

  char wap[0x13C]={0};
  char *pwap=wap;
    //预处理 补齐数据
    *(BYTE*)(UserName+len)=0x80;
  if(len<0x20)
  {
    *(BYTE*)(UserName+0x3F)=8*len;
  }
  else
  {
    *(BYTE*)(UserName+0x3F)=8*len%0x20;
    *(BYTE*)(UserName+0x3E)=len/0x20;
  }

  //改变字符串的字节序
  _asm
  {
    xor    ecx, ecx;
    mov    esi, UserName;
    mov    edi, pwap;
lab1:
    mov    eax,dword ptr [esi+ecx*4];
        bswap  eax;
    mov    dword ptr [edi+ecx*4], eax;
    add    ecx, 1;
    cmp    ecx, 0x10;
    jl     short lab1;
  }

  //生成表

  _asm
  {
     mov     edx,0x40;
     mov     eax,pwap;
     add     eax,8;
lab2:
     mov    ecx,dword ptr [eax+0x2C];
     xor    ecx,dword ptr [eax+0x18];
     add    eax,4;
     xor    ecx,dword ptr [eax-0xC];
     xor    ecx,dword ptr [eax-4];
     rol    ecx,1;
     sub    edx,1;
     mov    dword ptr [eax+0x34],ecx;
     jnz    short lab2;
  }

  //SHA

  _asm
  {
    push    ebp;
    mov     eax,Key;
    mov     edx,dword ptr [eax+4];
    mov     ebx,dword ptr [eax+0xC];
    mov     esi,dword ptr [eax+8];
    mov     edi,dword ptr [eax];
    mov     eax,dword ptr [eax+0x10];
    push    eax;
    push    edi;
    xor     eax,eax;

lab4:
    rol     edi,5;
    mov     ebp,esi;
    and     ebp,edx;
    mov     ecx,edx;
    not     ecx;
    and     ecx,ebx;
    xor     ecx,ebp;
    add     edi,ecx;

    push    ebp;
    mov     ebp,dword ptr [esp+0xC];
    add     edi,dword ptr [wap+eax*4];
    pop     ebp;

    mov     ecx,dword ptr [esp+4];
    lea     ecx,dword ptr [edi+ecx+0x5A827999];
    mov     edi,dword ptr [esp];
    mov     dword ptr [esp+4],ebx;
    ror     edx,2;
    mov     dword ptr [esp],ecx;
    rol     ecx,5;
    mov     ebx,edi;
    not     ebx;
    and     ebx,esi;
    mov     ebp,edx;
    and     ebp,edi;
    xor     ebx,ebp;
    add     ecx,ebx;

    push    ebp;
    mov     ebp,dword ptr [esp+0xC];
    add     ecx,dword ptr [wap+eax*4+4];
    pop     ebp;

    mov     ebx,dword ptr [esp+4];
    lea     ecx,dword ptr [ecx+ebx+0x5A827999];
    ror     edi,2;
    mov     dword ptr [esp+4],esi;
    mov     ebx,edx;
    mov     edx,dword ptr [esp];
    mov     dword ptr [esp],ecx;
    rol     ecx,5;
    mov     ebp,edi;
    and     ebp,edx;
    mov     esi,edx;
    not     esi;
    and     esi,ebx;
    xor     esi,ebp;
    add     ecx,esi;

    push    ebp;
    mov     ebp,dword ptr [esp+0xC];
    add     ecx,dword ptr [wap+eax*4+8];
    pop     ebp;
    mov     esi,dword ptr [esp+4];
    lea     ecx,dword ptr [ecx+esi+0x5A827999];
    mov     esi,dword ptr [esp];
    ror     edx,2;
    mov     dword ptr [esp+4],ebx;
    mov     dword ptr [esp],ecx;
    rol     ecx,5;
    mov     ebx,esi;
    not     ebx;
    and     ebx,edi;
    mov     ebp,edx;
    and     ebp,esi;
    xor     ebx,ebp;
    add     ecx,ebx;

    push    ebp;
    mov     ebp,dword ptr [esp+0xC];
    add     ecx,dword ptr [wap+eax*4+0xC];
    pop     ebp;

    mov     ebx,dword ptr [esp+4];
    lea     ecx,dword ptr [ecx+ebx+0x5A827999];
    ror     esi,2;
    mov     dword ptr [esp+4],edi;
    mov     ebx,esi;
    mov     esi,dword ptr [esp];
    mov     edi,esi;
    not     edi;
    and     edi,edx;
    mov     dword ptr [esp],ecx;
    rol     ecx,5;
    mov     ebp,ebx;
    and     ebp,esi;
    xor     edi,ebp;
    add     ecx,edi;

    push    ebp;
    mov     ebp,dword ptr [esp+0xC];
    add     ecx,dword ptr [wap+eax*4+0x10];
    pop     ebp;

    mov     edi,dword ptr [esp+4];
    lea     ecx,dword ptr [ecx+edi+0x5A827999];
    mov     ebp,edx;
    mov     edx,dword ptr [esp];
    mov     edi,ecx;
    add     eax,5;
    ror     esi,2;
    mov     dword ptr [esp+4],ebp;
    mov     dword ptr [esp],edi;
    cmp     eax,0x14;
    jl      lab4;

    cmp     eax,0x28;
    jge     short lab5;
lab6:
    mov      ebp,ebx;
    xor      ebp,esi;
    xor      ebp,edx;
    rol      ecx,5;
    add      ecx,ebp;

    push    ebp;
    mov     ebp,dword ptr [esp+0xC];
    add     ecx,dword ptr [wap+eax*4];
    pop     ebp;

    mov     ebp,dword ptr [esp+4];
    lea     ecx,dword ptr [ecx+ebp+0x6ED9EBA1];
    ror     edx,2;
    mov     ebp,ebx;
    mov     ebx,esi;
    add     eax,1;
    cmp     eax,0x28;
    mov     esi,edx;
    mov     edx,edi;
    mov     dword ptr [esp+4],ebp;
    mov     edi,ecx;
    jl      short lab6;
    mov     dword ptr [esp],ecx;
lab5:
    cmp     eax,0x3C;
    jge     short lab7;
lab8:
    mov     edi,esi;
    xor     edi,edx;
    and     edi,ebx;
    mov     ebp,esi;
    and     ebp,edx;
    rol     ecx,5;
    add     ecx,dword ptr [esp+4];
    xor     edi,ebp;

    push    ebp;
    mov     ebp,dword ptr [esp+0xC];
    add     edi,dword ptr [wap+eax*4];
    pop     ebp;

    ror     edx,2;
    mov     ebp,ebx;
    mov     ebx,esi;
    add     eax,1;
    cmp     eax,0x3C;
    lea     ecx,dword ptr [edi+ecx+0x8F1BBCDC];
    mov     esi,edx;
    mov     edx,dword ptr [esp];
    mov     dword ptr [esp+4],ebp;
    mov     dword ptr [esp],ecx;
    jl      short lab8;
lab7:
    cmp     eax,0x50;
    jge     short lab9;
lab10:
    mov     edi,ebx;
    xor     edi,esi;
    rol     ecx,5;
    xor     edi,edx;

    push    ebp;
    mov     ebp,dword ptr [esp+0xC];
    add     edi,dword ptr [wap+eax*4];
    pop     ebp;

    add     ecx,ebp;
    ror     edx,2;
    mov     ebp,ebx;
    mov     ebx,esi;
    add     eax,1;
    cmp     eax,0x50;
    lea     ecx,dword ptr [edi+ecx+0xCA62C1D6];
    mov     esi,edx;
    mov     edx,dword ptr [esp];
    mov     dword ptr [esp],ecx;
    jl      short lab10;
lab9:

    push    ebp;
    mov     ebp,dword ptr [esp+0xC];
    mov     eax,Key;
    pop     ebp;

    mov     edi,dword ptr [eax];
    add     edi,ecx;
    mov     ecx,dword ptr [eax+4];
    add     ecx,edx;
    mov     dword ptr [eax+4],ecx;
    mov     ecx,dword ptr [eax+8];
    add     ecx,esi;
    mov     dword ptr [eax+8],ecx;
    mov     ecx,dword ptr [eax+0xC];
    add     ecx,ebx;
    mov     dword ptr [eax],edi;
    mov     dword ptr [eax+0xC],ecx;
    mov     ecx,dword ptr [eax+0x10];
    add     ecx,ebp;
    mov     dword ptr [eax+0x10],ecx;

    pop     edi;
    pop     eax;
    pop     ebp;
  }

  //翻转字节序

  _asm
  {
    xor    ecx, ecx;
    mov    esi, Key;
lab11:
    mov    eax,dword ptr [esi+ecx*4];
        bswap  eax;
    mov    dword ptr [esi+ecx*4], eax;
    add    ecx, 1;
    cmp    ecx, 5;
    jl     short lab11;
  }
  return Key;

}
```

我也一段 Name 的计算代码

代码:



```
int sub\_4078E0(char\* a0)
{
  char \*p = a0;
  while (\*p)
  {
    p++; 
    p++;
    if (\*(p-1) == '\\0')
    {
      return p-a0-1;
    }

    p++;
    if (\*(p-1) == '\\0')
    {
      return p-a0-1;
    }

    p++;
    if (\*(p-1) == '\\0')
    {
      return p-a0-1;
    }
  }

  return p+1-a0-1;
}

char \*memcpy\_r(char\* a0, char\* a4, int a8)
{
  char \*p = a0;
  while (a8>0)
  {
    \*p = \*(a4-a0+p);
    p++;
    a8--;
  }
  
  return a0;
}

int bswap(int n)
{
  \_asm {
    mov eax, n
    bswap eax
    mov n, eax
  }
  return n;
}


void my\_sha1\_compile(sha1\_ctx ctx\[1\])
{
  sha1\_compile(ctx);
  return;

}

void my\_sha1\_hash(sha1\_ctx ecx\[1\], char\* a0, int a4)
{
  int eax = ((ecx->count\[0\]>>3) & 0x3f);
  ecx->count\[0\] += a4\*8;
  if (ecx->count\[0\] < (sha1\_32t)a4\*8)
  {
    ecx->count\[1\]++;
  }
  ecx->count\[1\] += (a4>>29);
  if (a4 < (0x40 - eax))
  {
    memcpy\_r((char\*)ecx->wbuf+eax, a0, a4);
    return;
  }

  memcpy\_r((char\*)ecx->wbuf+eax, a0, (0x40 - eax));
  my\_sha1\_compile(ecx);
  int va4 = (0x40 - eax);
  int ebp = va4+0x3f;
  char\* ebx = (char\*)ecx->wbuf+0x100;

  if (ebp < a4)
  {
    do {
      my\_sha1\_compile(ecx);
      va4 += 0x40;
      ebp += 0x40;
      ebx += 0x100;
    } while(ebp < a4);
  }

  memcpy\_r((char\*)ecx->wbuf, a0+va4, a4-va4);
  return;

}

void my\_sha1\_end(sha1\_ctx ecx\[1\], char\* a0)
{
  char v1c\[8\] = {0};
  \*(int\*)(v1c+4) = ecx->count\[0\];
  \*(int\*)(v1c) = ecx->count\[1\];

  int ecx1;
  for (ecx1=0; ecx1<2; ecx1++)
  {
    \*(int\*)(v1c+ecx1\*4) = bswap(\*(int\*)(v1c+ecx1\*4));
  }

  sha1\_32t    i = (sha1\_32t)((ecx->count\[0\]>>3) & 0x3f);
  static char s\_byte\_40CD50\[64\] = {'\\x80'};
  my\_sha1\_hash(ecx, s\_byte\_40CD50, (i<56?56-i:120-56));
  my\_sha1\_hash(ecx, v1c, 8);
  char v14\[20\];
  for (ecx1=0; ecx1<5; ecx1++)
  {
    \*(int\*)(v14+ecx1\*4) = bswap(ecx->hash\[ecx1\]);
  }
  memcpy\_r(a0, v14, 20);
  memset(ecx->hash, 0, 20);
  return;
}

bool sub\_401000(int ecx, char \*edi, char \*a0)
{
  char v30\[32+12\] = {0};

  if (edi == NULL || a0 == NULL)
  {
    return false;
  }

  DWORD dwVolumeSerialNumber;
  GetVolumeInformationA("C:\\\\", NULL, 0, &dwVolumeSerialNumber, NULL,NULL, NULL, 0);
  memcpy\_r(v30, a0, ecx);
  \*(DWORD\*)(v30+ecx) = dwVolumeSerialNumber;
  memcpy\_r(v30+ecx+4, "Tencent", sub\_4078E0("Tencent"));
  sha1\_ctx v8c;
  sha1\_begin(&v8c);
  v8c.hash\[0\] = 0xB1CAB1CA;
  v8c.hash\[1\] = 0xCCBFCCBF;
  v8c.hash\[2\] = 0xBFB2D6BE;
  v8c.hash\[3\] = 0xF8C7D8B5;
  v8c.hash\[4\] = 0xEEC7BCCD;
  my\_sha1\_hash(&v8c, v30, ecx+11);
  my\_sha1\_end(&v8c, edi);
  return true;
}
```

`sha1` 系列函数的代码，到网上搜吧

```
if(len<0x20)  
  {  
    *(BYTE*)(UserName+0x3F)=8*len;  
  }  
  else  
  {  
    *(BYTE*)(UserName+0x3F)=8*len%0x20;  
    *(BYTE*)(UserName+0x3E)=len/0x20;  
  }
```

应该为 `*(BYTE*)(UserName+0x3F)=(8*len)%(8*0x20)`；
