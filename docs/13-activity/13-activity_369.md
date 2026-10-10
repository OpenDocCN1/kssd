

# 【讨论】Tx 第三题的两个正向函数

> 原文：[`www.pediy.com/kssd/pediy11/123396.html`](https://www.pediy.com/kssd/pediy11/123396.html)

基本上两个正向函数逆了出来，也没花多少时间，但是在 reversekey 算法的时候卡死了，穷举当然可以，但 32\*\*32，不知道穷举下来我这破电脑要几天，而且还是想学习下 kengen；  
  
由于右手石膏 ing，代码逆的时候没太注意格式，试了下 base32 直接 encode username 的 hash，结果不对，贴下代码，往大牛指点指点   
  

代码:

```
// kengon.cpp : Defines the entry point for the console application.
//

#include "stdafx.h"
#include <string.h>
#include <Windows.h>
#include <iostream>
#include "sha1.h"
#include "Base32.h"
using namespace std;
/\*
\*  Function prototype
\*/
void DisplayMessageDigest(unsigned \*message\_digest,unsigned len);

int StrCpy(BYTE Dst\[\],BYTE Src\[\],DWORD len)
{
  DWORD iLen = len;
  DWORD iSrc = (DWORD)&Src\[0\];
  DWORD iDst = (DWORD)&Dst\[0\];
  DWORD disTen = iSrc - iDst;
  DWORD Ret = iDst;
  if ( iLen!=0 )
  {
    do 
    {
      --iLen;
      \*((BYTE \*)iDst) = (\*(BYTE \*)(iDst + disTen));
      ++iDst;
    } while (iLen);

  }
  return iDst;
}

/\*\* \\fn  int GetHash(BYTE \*username,BYTE userkey\[\])
 \*  \\brief 用户名生成 20 byte
 \*/
int GetHash(BYTE \*username,BYTE userkey\[\])
{
  DWORD a = 0;
  DWORD VolumeNumber = 0;
  BYTE tail\[\] = "Tencent";
  BYTE userbuf\[32\] = {0};
  BYTE Det\[0x2B\] ;
  memset(Det,0,0x2B);
  int iuserlen = strlen((char\*)username);
  GetVolumeInformation("C:\\\\",NULL,NULL,&VolumeNumber,NULL,NULL,NULL,NULL);
  StrCpy(userbuf,username,iuserlen);
  memcpy(userbuf+iuserlen,&VolumeNumber,4);

  memcpy(userbuf+iuserlen+4,tail,7);
  iuserlen +=0xB;
  SHA1Context sha;
  SHA1Reset(&sha);
   // SHA1Input(&sha,  userbuf, strlen((char \*)userbuf));
  SHA1Input(&sha,  userbuf, iuserlen);
    if (!SHA1Result(&sha))
    {
        fprintf(stderr, "ERROR-- could not compute message digest\\n");
    }
    else
    {
        printf("\\t");  
        for(int i = 0; i < 5 ; i++)
        {
            printf("%X ", sha.Message\_Digest\[i\]);
        }
        printf("\\n");
        
  }
  memcpy(userkey,sha.Message\_Digest,20);
  for (int i =0;i<5;i++)
  {
    DWORD temp = \*(unsigned\*)(userkey + i\*4);
    \_asm {
      pushad
      mov eax,temp
      bswap eax
      mov temp,eax
      popad
    }
    memcpy(userkey + i\*4,&temp,4);
  }
  return 0;

}

bool CreateTable(BYTE \* Table,BYTE \*lpinit)
{
  BYTE Value = 0XFF;
  BYTE index = 0;
  memset(Table,0xFF,256);
  for (int i = 0;i <32;i++)
  {
    index = lpinit\[i\];
    Table\[index\] = i;

  }
  Table\[0x3D\] = 0x20;
  return true;
}
BYTE FindCharleft(BYTE Hash,BYTE proHash,BYTE cl)
{
  BYTE al = 0xFF;
  for(int i = 0;i<32;i++)
  {
    al = i;
    al = al |(al<< cl);
    if (al == Hash)
    {
      return i;
    }
  }
  return al;
}
BYTE FindSingle(BYTE Hash,BYTE proHash)
{
  BYTE al = 0xFF;
  for(int i = 0;i<32;i++)
  {
    al = i;
    al = al | proHash;
    if (al == Hash)
    {
      return i;
    }
  }
  return al;
}
BYTE FindCharRight(BYTE Hash,BYTE proHash,BYTE cl,BYTE \*p)
{
  BYTE al = 0xFF;
  BYTE dl = \*p;
  for(BYTE i = 0;i<32;i++)
  {
    al = i; 
    al = al<<cl;
    for (BYTE j = 0;j<32;j++)
    {
      BYTE  d = j;
      d = d >> dl;
      BYTE temp = al | d |proHash; 
      if (temp  == Hash)
      {
        \*p = j;
        return i;
      }  
    }
    
    
  
  }
  return al;
}
int ReverseKey(BYTE \*userHash,BYTE \* key,BYTE \*Table,BYTE \*charset)
{
  DWORD len = 32;
  DWORD iebx = 0,
    iesi = 0,
    iebp = 0,
    iecx = 0;
  BYTE ProChar\[32\] = {0};
  BYTE Hashbuf\[33\] = {0};
  BYTE Index\[32\] = {0};

  BYTE  cl = 0,
      index ;
  DWORD count = 0;
   while(iebp <32 )
  {

    index = 0xFF;
    BYTE  hash = userHash\[iebp\];
    if (iebx <3)
    {
      iebx = (iebx - 3)&7;
      if (iebx != 0)
      {
        iecx = 8;
        iecx -= iebx;
        cl = BYTE(0x000000FF & iecx);
        ProChar\[count\] = cl;
        count++;

      }else{
        index = FindSingle(hash,Hashbuf\[iebp\]);
        if ((index!= 0xFF)&&(index < 32))
        {
          key\[iebp\] = charset\[index\];
          printf("%c",charset\[key\[0\]\]);
          Index\[count\] = charset\[index\];
          iebp++;
          count++;
        }

      }
    }else if (iebx >= 3)
    {
      iebx = (iebx - 3)&7;
        BYTE  dl = BYTE(0x000000FF&iebx);
      index = FindCharRight(hash,Hashbuf\[iebp\],cl,&dl);
      if ((index!= 0xFF)&&(index < 32))
      {
        Index\[count--\] = index;
        key\[iebp\] = charset\[index\];
        printf("%c",charset\[index\]);
        iebp++;
        iecx = 8;
        iecx -= iebx;
        Index\[count\] = dl;
        BYTE bl = BYTE(0x000000FF & iecx);
        dl = dl << bl;
        Hashbuf\[iebp\] = Hashbuf\[iebp\] | dl;
        ProChar\[iebp\] = bl;
        count++;
      }
    
      
    

    }

 
   }
  printf("\\n");
  iebx = 0;
  for (int k = 0;k<32;k++)
  {
    
    printf("%d\\t",iebx);
    iebx = (iebx - 3)&7;
    
  }
  return 0;
}

/\*\* \\fn  int CreateHashKey(BYTE \*hashkey,BYTE \* key,BYTE \*Table)
 \*  \\brief 由 32 字节生成 20 字节
 \*  \\param hashkey 生成的 Hash 值
 \*  \\param key 去除‘-’字符的 key
 \*/
int CreateHashKey(BYTE \*hashkey,BYTE \* key,BYTE \*Table)
{
  DWORD len = 32;
  DWORD i = 0,
      index = 0,
      count = 0;
  BYTE  cl = 0;
  while(count < 32)
  {
    DWORD item = key\[count\];
    BYTE  al = Table\[item\];
    if (i < 3)
    {
      i = (i - 3)&7;
      if (i != 0)
      {
        item = 8;
        item -= i;
        cl = BYTE(0x000000FF & item);
        al = al << cl;
        hashkey\[index\] = hashkey\[index\] | al;
        count++;
      }else{
          hashkey\[index\] = hashkey\[index\] | al;
          index++;
          count++;
      }
   
    }else if(i>=3){
      i = (i - 3)&7;
      BYTE dl = al;
      cl = BYTE(0x000000FF&i);
      dl = dl >>cl;
      hashkey\[index\] = hashkey\[index\] | dl;
      index++;
      item = 8;
      item -= i;
      cl = BYTE(0x000000FF & item);
      al = al << cl;
      hashkey\[index\] = hashkey\[index\] | al;
      count++;

    }

  }
  return 0;
}

int main(int argc, BYTE\* argv\[\])
{
  BYTE username\[64\] = {0};
  BYTE userkey\[20\]= {0};
  char key\[36\] = {0};
  BYTE keykey\[20\] = {0};
  BYTE table\[256\] = {0};
  BYTE lpinit\[\] ="ABCDEFGHJKMNPQRSTVWXYZ1234567890";
  BYTE lpkey\[\] ="12345678123456781234567812345678";
  BYTE buf\[32\] ={0};
  BYTE lpname\[\]="kanghtta";
  StrCpy(username,lpname,strlen((char\*)lpname));
  GetHash(username,userkey);
  
//  DisplayMessageDigest((unsigned \*)userkey,20);
  bool result = CreateTable(table,lpinit);
//  DisplayMessageDigest((unsigned \* )table,256);
  BYTE Table\[256\] = {0};

  Base32 base32 ;
  base32.GetEncode32Length(20);
  base32.Encode32(userkey,20,buf);
  base32.Map32(buf,32,lpinit);



  //CreateKey(userkey,buf,table,lpinit);
  memcpy(key,buf,8);
  strcat(key,"-");
  memcpy(key + 9,buf+8,8);
  strcat(key,"-");
  memcpy(key + 0x12,buf+0x11,8);
  strcat(key,"-");
  memcpy(key + 0x1B,buf+0x1A,8);
  printf("\\tusername:\\n\\t\\t%s\\n",lpname);
  printf("\\tkey:\\n");
  printf("\\t\\t%s \\n ",key);
  
//   CreateHashKey(keykey,lpkey,table);
//   DisplayMessageDigest((unsigned \*)keykey,20);


  return 0;
}



/\*  
\*  DisplayMessageDigest
\*
\*  Description:
\*      Display Message Digest array
\*
\*  Parameters:
\*      None.
\*
\*  Returns:
\*      Nothing.
\*
\*  Comments:
\*
\*/
void DisplayMessageDigest(unsigned \*message\_digest,unsigned len)
{
    ios::fmtflags   flags;
  
    cout << '\\t';
  
    flags = cout.setf(ios::hex|ios::uppercase,ios::basefield);
    cout.setf(ios::uppercase);
  
    for(int i = 0; i < len/4 ; i++)
    {
        cout << message\_digest\[i\] << ' ';
    if((i%5 == 0)&&i>=5)
    {
      cout << endl << '\\t';
    }
    }
  
    cout << endl;
  
    cout.setf(flags);
}

```

上传的附件
| ![](img/0623e01110d741fd20f66acbc8ec6f9d.jpg) || kengon.rar  
 |

memcpy(key + 0x12,buf+0x10,8);  
  strcat(key,"-");  
  memcpy(key + 0x1B,buf+0x18,8);  
  
..... 

我穷举的, 用时接近 0ms, 正向算法出来了, 应该能看出来可以按字节穷举, 我倒是没看出来是 base32~~  
  

代码:

```
BOOL SearchKeyPair(char key\[8\], BYTE code\[5\])
{
  BYTE c\[5\];
  for (int a0=0; a0<32; a0++) 
  for (int a1=0; a1<32; a1++) {
    key\[0\] = code\_char\[a0\];
    key\[1\] = code\_char\[a1\];
    Encode(key, c, 2);

    if (c\[0\] != code\[0\]) continue;

    for (int a2=0; a2<32; a2++) 
    for (int a3=0; a3<32; a3++) {
      key\[2\] = code\_char\[a2\];
      key\[3\] = code\_char\[a3\];
      Encode(key, c, 4);
      if (c\[1\] != code\[1\]) continue;

      for (int a4=0; a4<32; a4++) 
      for (int a5=0; a5<32; a5++) {
        key\[4\] = code\_char\[a4\];
        key\[5\] = code\_char\[a5\];
        Encode(key, c, 6);
        if (c\[2\] != code\[2\]) continue;

        for (int a6=0; a6<32; a6++) 
        for (int a7=0; a7<32; a7++) {

          key\[6\] = code\_char\[a6\];
          key\[7\] = code\_char\[a7\];
          Encode(key, c, 8);
          if (c\[3\] == code\[3\] && c\[4\] == code\[4\]) {
            return TRUE;
          }
        }
      }
    }
  }
  return FALSE;
}

```