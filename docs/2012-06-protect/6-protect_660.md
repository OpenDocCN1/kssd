
# Asprotect 中的 X86 的 VM 分析

> 原文：[`www.pediy.com/kssd/pediy08/pediy8-640.htm`](https://www.pediy.com/kssd/pediy08/pediy8-640.htm)

`aspr` 的 VM 还是比较简单的

不过把 `USER_BUFFER` 完整的变成 VM，除了主程序我还没见过别的试练品，别的多是代码变形~~~

其实，你可以一行一行的解析，不用去管跳转，这样就全部恢复了，另外 `aspr` 主程序中的 VM 还有二次 VM（只是抽了 `ADD`, `MOV`, `SUB` 二次处理）

贴一个以前我写的修复 `aspr 2.3 SKE 06.26` 主程序中一处 VM 的例子

```c
// Aspr 2.3 SKE 06.26 fixer

//

#include <windows.h>

#include <stdio.h>

DWORD  VMDataSize;

LPBYTE  VMData;

LPBYTE  OutData;

DWORD  PCODELEN = 0x18;

DWORD  PCODENUM = 0x15;

DWORD  MAGIC = 0x7fcf10fc;

DWORD  BASE = 0x400000;

BYTE  TABLE[0x10] = {0x63, 0x9F, 0x82, 0x71, 0xC3, 0xB2, 0xA1, 0x51, 0x01, 0xED, 0x9A, 0x23, 0x50, 0xF3, 0x58, 0x09};

#pragma warning(disable:4244)

/*

 +0x00 BYTE pcode1a

 +0x01 BYTE pcode2a

 +0x02 BYTE symbol1

 +0x03 BYTE isreloc

 +0x04 DWORD address

 +0x08 BYTE  r32fix

 +0x09 BYTE data3

 +0x0A DWORD data1

 +0x0E DWORD data2

 +0x12 BYTE r32

 +0x13 BYTE mark1

 +0x14 BYTE  symbol2

 +0x15 BYTE mark1a

 +0x16 BYTE pcode1

 +0x17 BYTE pcode2

*/

/* 对应表

 0x00  0x63

 0x01  0x9F

 0x02  0x82

 0x03  0x71

 0x04  0xC3

 0x05  0xB2

 0x06  0xA1

 0x07  0x51

 0x08  0x01

 0x09  0xED

 0x0A  0x9A

 0x0B  0x23

 0x0C  0x50

 0x0D  0xF3

 0x0E  0x58

 0x0F  0x09

*/

#pragma pack(1)

typedef struct SS

{

  BYTE  pcode1a;

  BYTE  pcode2a;

  BYTE  symbol1;

  BYTE  isreloc;

  DWORD  address;

  BYTE  r32fix;

  BYTE  data3;

  DWORD  data1;

  DWORD  data2;

  BYTE  r32;

  BYTE  mark1;

  BYTE  symbol2;

  BYTE  mark1a;

  BYTE  pcode1;

  BYTE  pcode2;

}SS, *PSS;

typedef struct DD

{

  DWORD  address;

  BYTE  length;

  BYTE  done;

  BYTE  undo;

  BYTE  lost;

  BYTE  data[24];

}DD, *PDD;

#pragma pack(1)

typedef struct S2

{

  BYTE  work;

  BYTE  type;

  BYTE  mark;

  BYTE  dst;

  BYTE  src;

  DWORD  dstplus;

  DWORD  srcplus;

}S2, *PS2;

void ReadVMData()

{

  HANDLE  hFile;

  DWORD  temp;

  hFile = CreateFile("vm.bin", GENERIC_READ, 0, NULL, OPEN_EXISTING, 0, NULL);

  VMDataSize = GetFileSize(hFile, NULL);

  VMData = (LPBYTE)VirtualAlloc(NULL, VMDataSize, MEM_COMMIT, PAGE_READWRITE);

  ReadFile(hFile, VMData, VMDataSize, &temp, NULL);

  CloseHandle(hFile);

  OutData = (LPBYTE)VirtualAlloc(NULL, PCODENUM*sizeof(DD), MEM_COMMIT, PAGE_READWRITE);

}

void ParseR32(PSS pss, PDD pdd)

{

  BYTE  data;

  DWORD  data4;

  pdd->data[pdd->length] = pss->r32;

  pdd->length++;

  if ( ((pss->r32)&0x80) == 0 && ((pss->r32)&0x40) == 0 )

  {

    data = (pss->r32)&0x07;

    if (data == 0x04)

    {

      data = pss->r32fix;

      pdd->data[pdd->length] = data;

      pdd->length++;

    }

    if (data == 0x05)

    {

      data4 = pss->data1 + pss->isreloc*BASE;

      data = (data4>>0x00)&0xFF;

      pdd->data[pdd->length] = data;

      pdd->length++;

      data = (data4>>0x08)&0xFF;

      pdd->data[pdd->length] = data;

      pdd->length++;

      data = (data4>>0x10)&0xFF;

      pdd->data[pdd->length] = data;

      pdd->length++;

      data = (data4>>0x18)&0xFF;

      pdd->data[pdd->length] = data;

      pdd->length++;

    }

  }

  if ( ((pss->r32)&0x80) == 0 && ((pss->r32)&0x40) != 0 )

  {

    data = (pss->r32)&0x07;

    if (data == 0x04)

    {

      data = pss->r32fix;

      pdd->data[pdd->length] = data;

      pdd->length++;

    }   

    data4 = pss->data1;

    data = data4 & 0xFF;

    pdd->data[pdd->length] = data;

    pdd->length++;

  }

  if ( ((pss->r32)&0x80) != 0 && ((pss->r32)&0x40) == 0 )

  {

    data = (pss->r32)&0x07;

    if (data == 0x04)

    {

      data = pss->r32fix;

      pdd->data[pdd->length] = data;

      pdd->length++;

    }

    data4 = pss->data1 + pss->isreloc*BASE;

    data = (data4>>0x00)&0xFF;

    pdd->data[pdd->length] = data;

    pdd->length++;

    data = (data4>>0x08)&0xFF;

    pdd->data[pdd->length] = data;

    pdd->length++;

    data = (data4>>0x10)&0xFF;

    pdd->data[pdd->length] = data;

    pdd->length++;

    data = (data4>>0x18)&0xFF;

    pdd->data[pdd->length] = data;

    pdd->length++;

  }

}

void ParseDATA2(PSS pss, PDD pdd, DWORD num)

{

  BYTE  data;

  DWORD  data4;

  if (num == 1)

  {

    data4 = pss->data2;

    if (data4 != 0)

      data4 += MAGIC;

    data = data4&0xFF;

    pdd->data[pdd->length] = data;

    pdd->length++;

  }

  else if (num == 2)

  {

    data4 = pss->data2;

    if (data4 != 0)

      data4 += MAGIC;

    data = (data4>>0x00)&0xFF;

    pdd->data[pdd->length] = data;

    pdd->length++;

    data = (data4>>0x08)&0xFF;

    pdd->data[pdd->length] = data;

    pdd->length++;

  }

  else if (num == 4)

  {

    data4 = pss->data2;

    if (data4 != 0)

      data4 += MAGIC+pss->isreloc*BASE;

    data = (data4>>0x00)&0xFF;

    pdd->data[pdd->length] = data;

    pdd->length++;

    data = (data4>>0x08)&0xFF;

    pdd->data[pdd->length] = data;

    pdd->length++;

    data = (data4>>0x10)&0xFF;

    pdd->data[pdd->length] = data;

    pdd->length++;

    data = (data4>>0x18)&0xFF;

    pdd->data[pdd->length] = data;

    pdd->length++;

  }

}

BYTE FindTable(BYTE Value)

{

  BYTE i = 0xFF;

  for (i=0; i<0x10; i++)

  {

    if (TABLE[i] == Value)

      break ;

  }

  return i;

}

void ParseVM()

{

  DWORD  NumberOfPcode;

  DWORD  i;

  PSS    pss;

  PDD    pdd;

  BYTE  pcode1;

  BYTE  pcode2;

  BYTE  mark;

  BYTE  data;     //单字临时数据

  DWORD  data4;    //双字临时数据

  DWORD  address;

  DWORD  prefix;

  NumberOfPcode = PCODENUM;

  pss = (PSS)VMData;

  pdd = (PDD)OutData;

  prefix = 2;

  for (i=0; i<NumberOfPcode; i++)

  {

    if (prefix == 0)

    {

      prefix = 0;

      pss++;

      pdd++;

      pcode1 = pss->pcode1 + (MAGIC & 0xFF);

      pcode2 = pss->pcode2 + (MAGIC & 0xFF);

      mark = pss->mark1;

      address = pss->address + MAGIC + BASE;

      pdd->address = address;

    }

    else if (prefix == 1)

    {

      prefix = 0;

      i--;

      pcode1 = pss->pcode1a + (MAGIC & 0xFF);

      pcode2 = pss->pcode2a + (MAGIC & 0xFF);

      mark = pss->mark1a;

    }

    else

    {

      prefix = 0;

      pcode1 = pss->pcode1 + (MAGIC & 0xFF);

      pcode2 = pss->pcode2 + (MAGIC & 0xFF);

      mark = pss->mark1;

      address = pss->address + MAGIC + BASE;

      pdd->address = address;

    }

    // 二次 stolen

    if (pss->symbol2)

    {

      pdd->lost = 1;

      pdd->length = pss->r32;

      memcpy(pdd->data, pss, PCODELEN);

      continue ;

    }

    pcode1 = FindTable(pcode1);

    //   0 族

    //

    //   00        ADD BYTE *R32, R8

    //   01        ADD DWORD *R32, R32

    //   02        ADD R8, BYTE *R32

    //   03        ADD R32, DOWRD *R32

    //   04        ADD AL, BYTE

    //   05        ADD EAX, DWORD

    //   06        PUSH ES

    //   07        POP ES

    //   08        OR BYTE *R32, R8

    //   09        OR DWORD *R32, R32

    //   0A        OR R8, BYTE *R32

    //   0B        OR R32, DOWRD *R32

    //   0C        OR AL, BYTE

    //   0D        OR EAX, DWORD

    //   0E        PUSH CS

    //   0F        很多

    if (pcode1 == 0x00)

    {

      if ( (mark&0x01) != 0 && (mark&0x02) != 0)

      {

        data = (pcode1<<4) | pcode2;

        if (pcode2 >= 0x00 && pcode2 <= 0x03)            //00-03

        {

          pdd->data[pdd->length] = data;

          pdd->length++;
```



```c
ParseR32(pss, pdd);
pdd->done = 1;
continue ;
}
else if (pcode2 == 0x04)                   //04
{
  pdd->data[pdd->length] = data;
  pdd->length++;
  ParseDATA2(pss, pdd, 1);
  pdd->done = 1;
  continue ;
}
else if (pcode2 == 0x05)                   //05
{
  pdd->data[pdd->length] = data;
  pdd->length++;
  ParseDATA2(pss, pdd, 4);
  pdd->done = 1;
  continue ;
}
}
else if ( (mark&0x01) != 0 && (mark&0x02) == 0)
{
  pcode2 += 8;
  data = (pcode1<<4) | pcode2;
  if (pcode2 >= 0x08 && pcode2 <= 0x0B)                //08-0B
  {
    pdd->data[pdd->length] = data;
    pdd->length++;
    ParseR32(pss, pdd);
    pdd->done = 1;
    continue ;
  }
  else if (pcode2 == 0x0C)                   //0C
  {
    pdd->data[pdd->length] = data;
    pdd->length++;
    ParseDATA2(pss, pdd, 1);
    pdd->done = 1;
    continue ;
  }
  else if (pcode2 == 0x0D)                   //0D
  {
    pdd->data[pdd->length] = data;
    pdd->length++;
    ParseDATA2(pss, pdd, 4);
    pdd->done = 1;
    continue ;
  }
}
else if ( (mark&0x01) == 0)
{
  pcode2 = FindTable(pcode2);
  data = (pcode1<<4) | pcode2;
  if (pcode2 == 0x06 || pcode2 == 0x07 || pcode2 == 0x0E) // 06 07 0E
  {
    pdd->data[pdd->length] = data;
    pdd->length++;
    pdd->done = 1;
    continue ;
  }
  else if (pcode2 == 0x0F)                  //很多
  {
    pdd->data[pdd->length] = data;
    pdd->length++;
    pcode2 = pss->data3 + (MAGIC &0xFF);
    if (pcode2 >= 0x80 && pcode2 <= 0x8F)                //0F80-0F8F
    {
      data = pcode2;
      pdd->data[pdd->length] = data;
      pdd->length++;
      ParseDATA2(pss, pdd, 4);
      pdd->done = 1;
      continue ;
    }
    else if (pcode2 == 0xB6)                  //MOVZX
    {
      data = pcode2;
      pdd->data[pdd->length] = data;
      pdd->length++;
      ParseR32(pss, pdd);
      pdd->done = 1;
      continue ;
    }
    else if (pcode2 >= 0x90 && pcode2 <= 0x9F)          //0F90-0F9F
    {
      data = pcode2;
      pdd->data[pdd->length] = data;
      pdd->length++;
      ParseR32(pss, pdd);
      pdd->done = 1;
      continue ;
    }
  }
}
}

//   1 族       done
//
//   10        ADC BYTE *R32, R8
//   11        ADC DWORD *R32, R32
//   12        ADC R8, BYTE *R32
//   13        ADC R32, DOWRD *R32
//   14        ADC AL, BYTE
//   15        ADC EAX, DWORD
//   16        PUSH SS
//   17        POP SS
//   18        SBB BYTE *R32, R8
//   19        SBB DWORD *R32, R32
//   1A        SBB R8, BYTE *R32
//   1B        SBB R32, DOWRD *R32
//   1C        SBB AL, BYTE
//   1D        SBB EAX, DWORD
//   1E        PUSH DS
//   1F        POP DS

else if (pcode1 == 0x01)
{
  if ( (mark&0x01) != 0 && (mark&0x02) != 0)
  {
    data = (pcode1<<4) | pcode2;
    if (pcode2 >= 0x00 && pcode2 <= 0x03)                //10-13
    {
      pdd->data[pdd->length] = data;
      pdd->length++;
      ParseR32(pss, pdd);
      pdd->done = 1;
      continue ;
    }
    else if (pcode2 == 0x04)                   //14
    {
      pdd->data[pdd->length] = data;
      pdd->length++;
      data4 = pss->data2;
      ParseDATA2(pss, pdd, 1);
      pdd->done = 1;
      continue ;
    }
    else if (pcode2 == 0x05)                   //15
    {
      pdd->data[pdd->length] = data;
      pdd->length++;
      ParseDATA2(pss, pdd, 4);
      pdd->done = 1;
      continue ;
    }
  }
  else if ( (mark&0x01) != 0 && (mark&0x02) == 0)
  {
    pcode2 += 8;
    data = (pcode1<<4) | pcode2;
    if (pcode2 >= 0x08 && pcode2 <= 0x0B)                //18-1B
    {
      pdd->data[pdd->length] = data;
      pdd->length++;
      ParseR32(pss, pdd);
      pdd->done = 1;
      continue ;
    }
    else if (pcode2 == 0x0C)                   //1C
    {
      pdd->data[pdd->length] = data;
      pdd->length++;
      ParseDATA2(pss, pdd, 1);
      pdd->done = 1;
      continue ;
    }
    else if (pcode2 == 0x0D)                   //1D
    {
      pdd->data[pdd->length] = data;
      pdd->length++;
      ParseDATA2(pss, pdd, 4);
      pdd->done = 1;
      continue ;
    }
  }
  else if ( (mark&0x01) == 0)
  {
    pcode2 = FindTable(pcode2);
    data = (pcode1<<4) | pcode2;
    if (pcode2 == 0x06 || pcode2 == 0x07 || pcode2 == 0x0E || pcode2 == 0x0F) //16 17 1E 1F
    {
      pdd->data[pdd->length] = data;
      pdd->length++;
      pdd->done = 1;
      continue ;
    }
  }
}

//   2 族       done
//
//   20        AND BYTE *R32, R8
//   21        AND DWORD *R32, R32
//   22        AND R8, BYTE *R32
//   23        AND R32, DOWRD *R32
//   24        AND AL, BYTE
//   25        AND EAX, DWORD
//   26        前缀
//   27        DAA
//   28        SUB BYTE *R32, R8
//   29        SUB DWORD *R32, R32
//   2A        SUB R8, BYTE *R32
//   2B        SUB R32, DOWRD *R32
//   2C        SUB AL, BYTE
//   2D        SUB EAX, DWORD
//   2E        前缀
//   2F        DAS

else if (pcode1 == 0x02)
{
  if ( (mark&0x01) != 0 && (mark&0x02) != 0)
  {
    data = (pcode1<<4) | pcode2;
    if (pcode2 >= 0x00 && pcode2 <= 0x03)                //20-23
    {
      pdd->data[pdd->length] = data;
      pdd->length++;
      ParseR32(pss, pdd);
      pdd->done = 1;
      continue ;
    }
    else if (pcode2 == 0x04)                   //24
    {
      pdd->data[pdd->length] = data;
      pdd->length++;
      ParseDATA2(pss, pdd, 1);
      pdd->done = 1;
      continue ;
    }
    else if (pcode2 == 0x05)                   //25
    {
      pdd->data[pdd->length] = data;
      pdd->length++;
      ParseDATA2(pss, pdd, 4);
      pdd->done = 1;
      continue ;
    }
  }
  else if ( (mark&0x01) != 0 && (mark&0x02) == 0)
  {
    pcode2 += 8;
    data = (pcode1<<4) | pcode2;
    if (pcode2 >= 0x08 && pcode2 <= 0x0B)                //28-2B
    {
      pdd->data[pdd->length] = data;
      pdd->length++;
      ParseR32(pss, pdd);
      pdd->done = 1;
      continue ;
    }
    else if (pcode2 == 0x0C)                   //2C
    {
```



```
    pdd->data[pdd->length] = data;
    pdd->length++;
    ParseDATA2(pss, pdd, 1);
    pdd->done = 1;
    continue;
}
else if (pcode2 == 0x0D)                 //2D
{
    pdd->data[pdd->length] = data;
    pdd->length++;
    ParseDATA2(pss, pdd, 4);
    pdd->done = 1;
    continue;
}
}
else if ((mark&0x01) == 0)
{
    pcode2 = FindTable(pcode2);
    data = (pcode1<<4) | pcode2;
    if (pcode2 == 0x06 || pcode2 == 0x0E)
    {
        pdd->data[pdd->length] = data;
        pdd->length++;
        prefix = 1;
        continue;
    }
    else if (pcode2 == 0x07 || pcode2 == 0x0F)         //27 2F
    {
        pdd->data[pdd->length] = data;
        pdd->length++;
        pdd->done = 1;
        continue;
    }
}
}
//   3 族       done
//
//   30        XOR BYTE *R32, R8
//   31        XOR DWORD *R32, R32
//   32        XOR R8, BYTE *R32
//   33        XOR R32, DOWRD *R32
//   34        XOR AL, BYTE
//   35        XOR EAX, DWORD
//   36        前缀
//   37        AAA
//   38        CMP BYTE *R32, R8
//   39        CMP DWORD *R32, R32
//   3A        CMP R8, BYTE *R32
//   3B        CMP R32, DOWRD *R32
//   3C        CMP AL, BYTE
//   3D        CMP EAX, DWORD
//   3E        前缀
//   3F        AAS
else if (pcode1 == 0x03)
{
    if ((mark&0x01) != 0 && (mark&0x02) != 0)
    {
        data = (pcode1<<4) | pcode2;
        if (pcode2 >= 0x00 && pcode2 <= 0x03)          //30-33
        {
            pdd->data[pdd->length] = data;
            pdd->length++;
            ParseR32(pss, pdd);
            pdd->done = 1;
            continue;
        }
        else if (pcode2 == 0x04)                 //34
        {
            pdd->data[pdd->length] = data;
            pdd->length++;
            ParseDATA2(pss, pdd, 1);
            pdd->done = 1;
            continue;
        }
        else if (pcode2 == 0x05)                 //35
        {
            pdd->data[pdd->length] = data;
            pdd->length++;
            ParseDATA2(pss, pdd, 4);
            pdd->done = 1;
            continue;
        }
    }
    else if ((mark&0x01) != 0 && (mark&0x02) == 0)
    {
        pcode2 += 8;
        data = (pcode1<<4) | pcode2;
        if (pcode2 >= 0x08 && pcode2 <= 0x0B)          //38-3B
        {
            pdd->data[pdd->length] = data;
            pdd->length++;
            ParseR32(pss, pdd);
            pdd->done = 1;
            continue;
        }
        else if (pcode2 == 0x0C)                 //3C
        {
            pdd->data[pdd->length] = data;
            pdd->length++;
            data4 = pss->data2;
            ParseDATA2(pss, pdd, 1);
            pdd->done = 1;
            continue;
        }
        else if (pcode2 == 0x0D)                 //3D
        {
            pdd->data[pdd->length] = data;
            pdd->length++;
            ParseDATA2(pss, pdd, 4);
            pdd->done = 1;
            continue;
        }
    }
    else if ((mark&0x01) == 0)
    {
        pcode2 = FindTable(pcode2);
        data = (pcode1<<4) | pcode2;
        if (pcode2 == 0x06 || pcode2 == 0x0E)          //36 3E
        {
            pdd->data[pdd->length] = data;
            pdd->length++;
            prefix = 1;
            continue;
        }
        else if (pcode2 == 0x07 || pcode2 == 0x0F)     //37 3F
        {
            pdd->data[pdd->length] = data;
            pdd->length++;
            pdd->done = 1;
            continue;
        }
    }
}
//   4 族       done
//
//   40        INC EAX
//   41        INC ECX
//   42        INC EDX
//   43        INC EBX
//   44        INC ESP
//   45        INC EAP
//   46        INC ESI
//   47        INC EDI
//   48        DEC EAX
//   49        DEC ECX
//   4A        DEC EDX
//   4B        DEC EBX
//   4C        DEC ESP
//   4D        DEC EBP
//   4E        DEC ESI
//   4F        DEC EDI
else if (pcode1 == 0x04)
{
    if ((mark&0x01) != 0)
    {
        data = (pcode1<<4) | pcode2;
        if (pcode2 >= 0x00 && pcode2 <= 0x07)    //40-47
        {
            pdd->data[pdd->length] = data;
            pdd->length++;
            pdd->done = 1;
            continue;
        }
    }
    else if ((mark&0x01) == 0)
    {
        pcode2 += 8;
        data = (pcode1<<4) | pcode2;
        if (pcode2 >= 0x08 && pcode2 <= 0x0F)    //48-4F
        {
            pdd->data[pdd->length] = data;
            pdd->length++;
            pdd->done = 1;
            continue;
        }
    }
}
//   5 族       done
//
//   50        PUSH EAX
//   51        PUSH ECX
//   52        PUSH EDX
//   53        PUSH EBX
//   54        PUSH ESP
//   55        PUSH EAP
//   56        PUSH ESI
//   57        PUSH EDI
//   58        POP EAX
//   59        POP ECX
//   5A        POP EDX
//   5B        POP EBX
//   5C        POP ESP
//   5D        POP EBP
//   5E        POP ESI
//   5F        POP EDI
else if (pcode1 == 0x05)
{
    if ((mark&0x01) != 0)
    {
        data = (pcode1<<4) | pcode2;
        if (pcode2 >= 0x00 && pcode2 <= 0x07)    //50-57
        {
            pdd->data[pdd->length] = data;
            pdd->length++;
            pdd->done = 1;
            continue;
        }
    }
    else if ((mark&0x01) == 0)
    {
        pcode2 += 8;
        data = (pcode1<<4) | pcode2;
        if (pcode2 >= 0x08 && pcode2 <= 0x0F)    //58-5F
        {
            pdd->data[pdd->length] = data;
            pdd->length++;
            pdd->done = 1;
            continue;
        }
    }
}
//   6 族       done
//
//   60        PUSHAD
//   61        POPAD
//   62        不支持
//   63        不支持
//   64        前缀
//   65        前缀
//   66        前缀
//   67        前缀
//   68        PUSH DWORD
//   69        IMUL DWORD *R32, DWORD
//   6A        PUSH BYTE
//   6B        IMUL DWORD *R32, BYTE
//   6C        INS BYTE PTR[EDI], DX
//   6D        INS DWORD PTR[EDI], DX
//   6E        OUTS DX, BYTE PTR[EDI]
//   6F        OUTS DX, DWORD PTR[EDI]
else if (pcode1 == 0x06)
{
    pcode2 = FindTable(pcode2);
    data = (pcode1<<4) | pcode2;
    if (pcode2 == 0x00 || pcode2 == 0x01)      //60 61
    {
        pdd->data[pdd->length] = data;
        pdd->length++;
        pdd->done = 1;
        continue;
    }
    else if (pcode2 >= 0x04 && pcode2 <= 0x07) //64-67
    {
        pdd->data[pdd->length] = data;
        pdd->length++;
        prefix = 1;
        continue;
    }
    else if (pcode2 == 0x08)                 //68
    {
        pdd->data[pdd->length] = data;
        pdd->length++;
        ParseDATA2(pss, pdd, 4);
        pdd->done = 1;
        continue;
    }
    else if (pcode2 == 0x09)                 //69
    {
        pdd->data[pdd->length] = data;
        pdd->length++;
```



```
ParseR32(pss, pdd);

ParseDATA2(pss, pdd, 4);

pdd->done = 1;

continue;
}
else if (pcode2 == 0x0A) //6A
{
  pdd->data[pdd->length] = data;
  pdd->length++;
  ParseDATA2(pss, pdd, 1);
  pdd->done = 1;
  continue;
}
else if (pcode2 == 0x0B) //6B
{
  pdd->data[pdd->length] = data;
  pdd->length++;
  ParseR32(pss, pdd);
  ParseDATA2(pss, pdd, 1);
  pdd->done = 1;
  continue;
}
else if (pcode2 >= 0x0C && pcode2 <= 0x0F) //6C-6F
{
  pdd->data[pdd->length] = data;
  pdd->length++;
  pdd->done = 1;
  continue;
}
}

// 7 族 done
//
// 70 JO
// 71 JNO
// 72 JB
// 73 JNB
// 74 JE
// 75 JNZ
// 76 JBE
// 77 JA
// 78 JS
// 79 JNS
// 7A JPE
// 7B JPO
// 7C JL
// 7D JGE
// 7E JLE
// 7F JG
else if (pcode1 == 0x07)
{
  pcode2 = FindTable(pcode2);
  data = (pcode1 << 4) | pcode2;
  if (pcode2 >= 0x00 && pcode2 <= 0x0F) //70-7F
  {
    pdd->data[pdd->length] = data;
    pdd->length++;
    ParseDATA2(pss, pdd, 1);
    pdd->done = 1;
    continue;
  }
}

// 8 族 done
//
// 80 ADD/OR/ADC/SBB/AND/CMP BYTE *R32, BYTE
// 81 ADD/OR/ADC/SBB/AND/CMP DWORD *R32, DWORD
// 82 ADD/OR/ADC/SBB/AND/CMP BYTE *R32, SIGNBYTE
// 83 ADD/OR/ADC/SBB/AND/CMP DWORD *R32, SIGNBYTE
// 84 TEST BYTE *R32, R8
// 85 TEST DWORD *R32, R32
// 86 XCHG BYTE *R32, R8
// 87 XCHG DWORD *R32, R32
// 88 MOV BYTE *R32, R8
// 89 MOV DWORD *R32, R32
// 8A MOV R8, BYTE *R32
// 8B MOV R32, DWORD *R32
// 8C MOV WORD *R32, XS
// 8D LEA R32, DWORD *R32
// 8E MOV XS, WORD *R32
// 8F POP DWORD *R32
else if (pcode1 == 0x08) //done
{
  if ((mark & 0x02) != 0 && (mark & 0x04) == 0 && (mark & 0x08) == 0 && (mark & 0x01) == 0)
  {
    pcode2 = 0; //80
    data = (pcode1 << 4) | pcode2;
    pdd->data[pdd->length] = data;
    pdd->length++;
    ParseR32(pss, pdd);
    ParseDATA2(pss, pdd, 1);
    pdd->done = 1;
    continue;
  }
  else if ((mark & 0x02) != 0 && (mark & 0x04) != 0 && (mark & 0x08) == 0 && (mark & 0x01) != 0)
  {
    pcode2 = 1; //81
    data = (pcode1 << 4) | pcode2;
    pdd->data[pdd->length] = data;
    pdd->length++;
    ParseR32(pss, pdd);
    ParseDATA2(pss, pdd, 4);
    pdd->done = 1;
    continue;
  }
  else if ((mark & 0x02) != 0 && (mark & 0x04) == 0 && (mark & 0x08) != 0 && (mark & 0x01) == 0)
  {
    pcode2 = 2; //82
    data = (pcode1 << 4) | pcode2;
    pdd->data[pdd->length] = data;
    pdd->length++;
    ParseR32(pss, pdd);
    ParseDATA2(pss, pdd, 1);
    pdd->done = 1;
    continue;
  }
  else if ((mark & 0x02) != 0 && (mark & 0x04) == 0 && (mark & 0x08) != 0 && (mark & 0x01) != 0)
  {
    pcode2 = 3; //83
    data = (pcode1 << 4) | pcode2;
    pdd->data[pdd->length] = data;
    pdd->length++;
    ParseR32(pss, pdd);
    ParseDATA2(pss, pdd, 1);
    pdd->done = 1;
    continue;
  }
  else if ((mark & 0x02) == 0 && (mark & 0x04) != 0 && (mark & 0x08) != 0)
  {
    if ((mark & 0x01) == 0) //84
    {
      pcode2 = 4;
      data = (pcode1 << 4) | pcode2;
      pdd->data[pdd->length] = data;
      pdd->length++;
      ParseR32(pss, pdd);
      pdd->done = 1;
      continue;
    }
    else if ((mark & 0x01) != 0) //85
    {
      pcode2 = 5;
      data = (pcode1 << 4) | pcode2;
      pdd->data[pdd->length] = data;
      pdd->length++;
      ParseR32(pss, pdd);
      pdd->done = 1;
      continue;
    }
  }
  else if ((mark & 0x02) == 0 && (mark & 0x04) != 0 && (mark & 0x08) == 0)
  {
    if ((mark & 0x01) == 0) //86
    {
      pcode2 = 6;
      data = (pcode1 << 4) | pcode2;
      pdd->data[pdd->length] = data;
      pdd->length++;
      ParseR32(pss, pdd);
      pdd->done = 1;
      continue;
    }
    else if ((mark & 0x01) != 0) //87
    {
      pcode2 = 7;
      data = (pcode1 << 4) | pcode2;
      pdd->data[pdd->length] = data;
      pdd->length++;
      ParseR32(pss, pdd);
      pdd->done = 1;
      continue;
    }
  }
  else if ((mark & 0x02) == 0 && (mark & 0x04) == 0 && (mark & 0x08) != 0)
  {
    pcode2 += 0x08;
    data = (pcode1 << 4) | pcode2;
    if (pcode2 >= 0x08 && pcode2 <= 0x0B) //88-8B
    {
      pdd->data[pdd->length] = data;
      pdd->length++;
      ParseR32(pss, pdd);
      pdd->done = 1;
      continue;
    }
  }
  else if ((mark & 0x02) == 0 && (mark & 0x04) == 0 && (mark & 0x08) == 0)
  {
    pcode2 = FindTable(pcode2);
    data = (pcode1 << 4) | pcode2;
    if (pcode2 >= 0x0C && pcode2 <= 0x0F) //8C-8F
    {
      pdd->data[pdd->length] = data;
      pdd->length++;
      ParseR32(pss, pdd);
      pdd->done = 1;
      continue;
    }
  }
}

// 9 族 done
//
// 90 NOP
// 91 XCHG EAX, ECX
// 92 XCHG EAX, EDX
// 93 XCHG EAX, EBX
// 94 XCHG EAX, ESP
// 95 XCHG EAX, EBP
// 96 XCHG EAX, ESI
// 97 XCHG EAX, EDI
// 98 CWDE
// 99 CDQ
// 9A 未实现
// 9B 未实现
// 9C PUSHFD
// 9D POPFD
// 9E SAHF
// 9F LAHF
else if (pcode1 == 0x09)
{
  if ((mark & 0x01) != 0) //91-97
  {
    data = (pcode1 << 4) | pcode2;
    pdd->data[pdd->length] = data;
    pdd->length++;
    pdd->done = 1;
    continue;
  }
  else if ((mark & 0x01) == 0) //98 99 9C 9D 9E 9F
  {
    pcode2 = FindTable(pcode2);
    data = (pcode1 << 4) | pcode2;
    if (pcode2 == 0x08 || pcode2 == 0x09 || pcode2 == 0x0C
        || pcode2 == 0x0D || pcode2 == 0x0E || pcode2 == 0x0F)
    {
      pdd->data[pdd->length] = data;
      pdd->length++;
      pdd->done = 1;
      continue;
    }
  }
}

// A 族 done
//
// A0 MOV AL, [DWORD]
// A1 MOV EAX, [DWORD]
// A2 MOV [DWORD], AL
// A3 MOV [DWORD], EAX
// A4 MOVSB
// A5 MOVSD
// A6 CMPSB
// A7 CMPSD
```



```
//   A8        TEST AL, BYTE

//   A9        TEST EAX, DWORD

//   AA        STOS BYTE PTR[EDI]

//   AB        STOS DWORD PTR[EDI]

//   AC        LODS BYTE PTR[EDI]

//   AD        LODS DWORD PTR[EDI]

//   AE        SCAS BYTE PTR[EDI]

//   AF        SCAS DWORD PTR[EDI]

else if (pcode1 == 0x0A)

{

  pcode2 = FindTable(pcode2);

  data = (pcode1<<4) | pcode2;

  if (pcode2 >= 0x00 && pcode2 <=0x03)          //A0-A3

  {

    data = (pcode1<<4) | pcode2;

    pdd->data[pdd->length] = data;

    pdd->length++;

    ParseDATA2(pss, pdd, 4);

    pdd->done = 1;

    continue ;

  }

  else if (pcode2 >= 0x04 && pcode2 <=0x07)     //A4-A7

  {

    data = (pcode1<<4) | pcode2;

    pdd->data[pdd->length] = data;

    pdd->length++;

    pdd->done = 1;

    continue ;

  }

  else if (pcode2 == 0x08)                   //A8

  {

    data = (pcode1<<4) | pcode2;

    pdd->data[pdd->length] = data;

    pdd->length++;

    ParseDATA2(pss, pdd, 1);

    pdd->done = 1;

    continue ;

  }

  else if (pcode2 == 0x09)                   //A9

  {

    data = (pcode1<<4) | pcode2;

    pdd->data[pdd->length] = data;

    pdd->length++;

    ParseDATA2(pss, pdd, 4);   

    pdd->done = 1;

    continue ;

  }

  else if (pcode2 >= 0x0A && pcode2 <=0x0F)     //AA-AF

  {

    data = (pcode1<<4) | pcode2;

    pdd->data[pdd->length] = data;

    pdd->length++;

    pdd->done = 1;

    continue ;

  }

}

//   B 族       done

//

//   B0        MOV AL, BYTE

//   B1        MOV CL, BYTE

//   B2        MOV DL, BYTE

//   B3        MOV BL, BYTE

//   B4        MOV AH, BYTE

//   B5        MOV DH, BYTE

//   B6        MOV CH, BYTE

//   B7        MOV BH, BYTE

//   B8        MOV EAX, DWORD

//   B9        MOV ECX, DWORD

//   BA        MOV EDX, DWORD

//   BB        MOV EBX, DWORD

//   BC        MOV ESP, DWORD

//   BD        MOV EBP, DWORD

//   BE        MOV ESI, DWORD

//   BF        MOV EDI, DWORD

else if (pcode1 == 0x0B)

{

  if ( (mark&0x01) != 0 && (mark&0x02) != 0)    

  {

    data = (pcode1<<4) | pcode2;

    if (pcode2 >= 0x00 && pcode2 <=0x03)    //B0-B3

    {

      pdd->data[pdd->length] = data;

      pdd->length++;

      ParseDATA2(pss, pdd, 1);

      pdd->done = 1;

      continue ;

    }

  }

  else if ( (mark&0x01) != 0 && (mark&0x02) == 0)

  {

    pcode2 += 4;

    data = (pcode1<<4) | pcode2;

    if (pcode2 >= 0x04 && pcode2 <=0x07)    //B4-B7

    {

      pdd->data[pdd->length] = data;

      pdd->length++;

      ParseDATA2(pss, pdd, 1);

      pdd->done = 1;

      continue ;

    }

  }

  else if ( (mark&0x01) == 0)

  {

    pcode2 += 8;

    data = (pcode1<<4) | pcode2;

    if (pcode2 >= 0x08 && pcode2 <=0x0F)    //B8-BF

    {

      pdd->data[pdd->length] = data;

      pdd->length++;

      ParseDATA2(pss, pdd, 4);

      pdd->done = 1;

      continue ;

    }

  }

}

//   C 族       done

//

//   C0        ROL/ROR/RCL/RCR/SHL/SHR/SAL/SAR BYTE *R32, BYTE

//   C1        ROL/ROR/RCL/RCR/SHL/SHR/SAL/SAR DWORD *R32, BYTE

//   C2        RET  WORD

//   C3        RETN

//   C4        LES R32, DWORD *R32 未实现

//   C5        LDS R32, DWORD *R32 未实现

//   C6        MOV BYTE *R32, BYTE

//   C7        MOV DWORD *R32, DWORD

//   C8        ENTER WORD, BYTE

//   C9        LEAVE

//   CA        RETF WORD 未实现

//   CB        RETF 未实现

//   CC        INT3 未实现

//   CD        INT BYTE 未实现

//   CE        INTO 未实现

//   CF        IRETD 未实现

else if (pcode1 == 0x0C)

{

  pcode2 = FindTable(pcode2);

  data = (pcode1<<4) | pcode2;

  if (pcode2 == 0x00 || pcode2 == 0x01)         //C0-C1

  {

    data = (pcode1<<4) | pcode2;

    pdd->data[pdd->length] = data;

    pdd->length++;

    ParseR32(pss, pdd);

    ParseDATA2(pss, pdd, 1);

    pdd->done = 1;

    continue ;

  }

  else if (pcode2 == 0x02)                   //C2

  {

    data = (pcode1<<4) | pcode2;

    pdd->data[pdd->length] = data;

    pdd->length++;

    ParseDATA2(pss, pdd, 2);

    pdd->done = 1;

    continue ;   

  }

  else if (pcode2 == 0x03)                   //C3

  {

    data = (pcode1<<4) | pcode2;

    pdd->data[pdd->length] = data;

    pdd->length++;

    pdd->done = 1;

    continue ;

  }

  else if (pcode2 == 0x06)                   //C6

  {

    data = (pcode1<<4) | pcode2;

    pdd->data[pdd->length] = data;

    pdd->length++;

    ParseR32(pss, pdd);

    ParseDATA2(pss, pdd, 1);

    pdd->done = 1;

    continue ;

  }

  else if (pcode2 == 0x07)                   //C7

  {

    data = (pcode1<<4) | pcode2;

    pdd->data[pdd->length] = data;

    pdd->length++;

    ParseR32(pss, pdd);

    ParseDATA2(pss, pdd, 4);

    pdd->done = 1;

    continue ;

  }

  else if (pcode2 == 0x08)                   //C8

  {

    data = (pcode1<<4) | pcode2;

    pdd->data[pdd->length] = data;

    pdd->length++;

    ParseDATA2(pss, pdd, 2);

    data = pss->data3;

    pdd->data[pdd->length] = data;

    pdd->length++;

    pdd->done = 1;

    continue ;

  }

  else if (pcode2 == 0x09)                   //C9

  {

    data = (pcode1<<4) | pcode2;

    pdd->data[pdd->length] = data;

    pdd->length++;

    pdd->done = 1;

    continue ;

  }

}

//   D 族       done

//

//   D0        ROL/ROR/RCL/RCR/SHL/SHR/SAL/SAR BYTE *R32, 1

//   D1        ROL/ROR/RCL/RCR/SHL/SHR/SAL/SAR DWORD *R32, 1

//   D2        ROL/ROR/RCL/RCR/SHL/SHR/SAL/SAR BYTE *R32, CL

//   D3        ROL/ROR/RCL/RCR/SHL/SHR/SAL/SAR DWORD *R32, CL

//   D4        AAM BYTE

//   D5        AAD BYTE

//   D6        SALC 未实现

//   D7        XLAT 未实现

//   D8        浮点未实现

//   D9        浮点未实现

//   DA        浮点未实现

//   DB        浮点未实现

//   DC        浮点未实现

//   DD        浮点未实现

//   DE        浮点未实现

//   DF        浮点未实现

else if (pcode1 == 0x0D)

{

  pcode2 = FindTable(pcode2);

  data = (pcode1<<4) | pcode2;

  if (pcode2 >= 0x00 && pcode2 <= 0x03)        //D0-D3

  {

    data = (pcode1<<4) | pcode2;

    pdd->data[pdd->length] = data;

    pdd->length++;

    ParseR32(pss, pdd);

    pdd->done = 1;

    continue ;

  }

  else if (pcode2 >= 0x04 && pcode2 <= 0x05)    //D4-D5

  {

    data = (pcode1<<4) | pcode2;

    pdd->data[pdd->length] = data;

    pdd->length++;

    ParseDATA2(pss, pdd, 1);

    pdd->done = 1;

    continue ;

  }

}

//   E 族       done

//

//   E0        LOOPDNE BYTE

//   E1        LOOPDE BYTE

//   E2        LOOPD BYTE

//   E3        JECXZ BYTE

//   E4        IN AL, BYTE 未实现
```



// E5 IN EAX, BYTE 未实现  
// E6 OUT BYTE, AL 未实现  
// E7 OUT BYTE, EAX 未实现  
// E8 CALL DWORD  
// E9 JMP DWORD  
// EA JMP FAR DWORD WORD  
// EB JMP BYTE  
// EC IN AL, DX 未实现  
// ED IN EAX, DX 未实现  
// EE OUT DX, AL 未实现  
// EF OUT DX, EAX 未实现  

```c
else if (pcode1 == 0x0E)
{
  if ((mark & 0x01) != 0 && (mark & 0x02) != 0 && (mark & 0x04) == 0 && (mark & 0x08) != 0)
  {
    pcode2 = 0;                              // E0
    data = (pcode1 << 4) | pcode2;
    pdd->data[pdd->length] = data;
    pdd->length++;
    data4 = pss->data1;
    data = (data4 >> 0x00) & 0xFF;
    pdd->data[pdd->length] = data;
    pdd->length++;
    pdd->done = 1;
    continue;
  }
  else if ((mark & 0x01) != 0 && (mark & 0x02) != 0 && (mark & 0x04) == 0 && (mark & 0x08) == 0)
  {
    pcode2 = 1;                              // E1
    data = (pcode1 << 4) | pcode2;
    pdd->data[pdd->length] = data;
    pdd->length++;
    data4 = pss->data1;
    data = (data4 >> 0x00) & 0xFF;
    pdd->data[pdd->length] = data;
    pdd->length++;
    pdd->done = 1;
    continue;
  }
  else if ((mark & 0x01) != 0 && (mark & 0x02) != 0 && (mark & 0x04) != 0)
  {
    pcode2 = 2;                              // E2
    data = (pcode1 << 4) | pcode2;
    pdd->data[pdd->length] = data;
    pdd->length++;
    data4 = pss->data1;
    data = (data4 >> 0x00) & 0xFF;
    pdd->data[pdd->length] = data;
    pdd->length++;
    pdd->done = 1;
    continue;
  }
  else if ((mark & 0x01) != 0 && (mark & 0x02) == 0)
  {
    pcode2 = 3;                              // E3
    data = (pcode1 << 4) | pcode2;
    pdd->data[pdd->length] = data;
    pdd->length++;
    data4 = pss->data1;
    data = (data4 >> 0x00) & 0xFF;
    pdd->data[pdd->length] = data;
    pdd->length++;
    pdd->done = 1;
    continue;
  }
  else if ((mark & 0x01) == 0)
  {
    pcode2 = FindTable(pcode2);
    data = (pcode1 << 4) | pcode2;
    if (pcode2 == 0x0B)                      // EB
    {
      pdd->data[pdd->length] = data;
      pdd->length++;
      data4 = pss->data1;
      data = (data4 >> 0x00) & 0xFF;
      pdd->data[pdd->length] = data;
      pdd->length++;
      pdd->done = 1;
      continue;
    }
    else if (pcode2 == 0x08 || pcode2 == 0x09) // E8 E9
    {
      pdd->data[pdd->length] = data;
      pdd->length++;
      data4 = pss->data1;
      data = (data4 >> 0x00) & 0xFF;
      pdd->data[pdd->length] = data;
      pdd->length++;
      data = (data4 >> 0x08) & 0xFF;
      pdd->data[pdd->length] = data;
      pdd->length++;
      data = (data4 >> 0x10) & 0xFF;
      pdd->data[pdd->length] = data;
      pdd->length++;
      data = (data4 >> 0x18) & 0xFF;
      pdd->data[pdd->length] = data;
      pdd->length++;
      pdd->done = 1;
      continue;
    }
  }
}

// F 族       done
//
// F0        前缀
// F1        INT1 未实现
// F2        前缀
// F3        前缀
// F4        HLT 未实现
// F5        CMC
// F6        TEST/NOT/NEG/MUL/IMUL/DIV/iDIV BYTE *R32
// F7        TEST/NOT/NEG/MUL/IMUL/DIV/iDIV DWORD *R32
// F8        CALL DWORD
// F9        JMP DWORD
// FA        JMP FAR DWORD WORD
// FB        JMP BYTE
// FC        IN AL, DX 未实现
// FD        IN EAX, DX 未实现
// FE        无
// FF        INC/DEC/CALL/CALL FAR/JMP/JMP FAR/PUSH DWORD *R32
```

```c
else if (pcode1 == 0x0F)
{
  if ((mark & 0x01) != 0 && (mark & 0x02) != 0 && (mark & 0x04) != 0)
  {
    data = 0xF6;                             // F6
    pdd->data[pdd->length] = data;
    pdd->length++;
    ParseR32(pss, pdd);
    if (pss->data3 == 0)
      ParseDATA2(pss, pdd, 1);
    pdd->done = 1;
    continue;
  }
  else if ((mark & 0x01) != 0 && (mark & 0x02) != 0 && (mark & 0x04) == 0)
  {
    data = 0xF7;                             // F7
    pdd->data[pdd->length] = data;
    pdd->length++;
    ParseR32(pss, pdd);
    if (pss->data3 == 0)
      ParseDATA2(pss, pdd, 4);
    pdd->done = 1;
    continue;
  }
  else if ((mark & 0x01) != 0 && (mark & 0x02) == 0)
  {
    data = 0xFF;                             // FF
    pdd->data[pdd->length] = data;
    pdd->length++;
    ParseR32(pss, pdd);
    pdd->done = 1;
    continue;
  }
  else if ((mark & 0x01) == 0)
  {
    pcode2 = FindTable(pcode2);
    data = (pcode1 << 4) | pcode2;
    if (pcode2 == 0x00 || pcode2 == 0x02 || pcode2 == 0x03) // F0 F2 F3
    {
      pdd->data[pdd->length] = data;
      pdd->length++;
      prefix = 1;
      continue;
    }
    else if (pcode2 == 0x05)                 // F5
    {
      pdd->data[pdd->length] = data;
      pdd->length++;
      pdd->done = 1;
      continue;
    }
    else if (pcode2 >= 0x08 && pcode2 <= 0x0E) // F8 F9 FA FB FC FD
    {
      pdd->data[pdd->length] = data;
      pdd->length++;
      pdd->done = 1;
      continue;
    }
  }
}

pdd->undo = 1;
memcpy(pdd->data, pss, PCODELEN);
}
```

```c
void Check()
{
  FILE *fp;
  PDD pdd;
  int i;
  fp = fopen("out.txt", "w");
  pdd = (PDD)OutData;
  while (pdd->address != 0)
  {
    fprintf(fp, "%08X ", pdd->address);
    if (pdd->done)
    {
      for (i = 0; i < pdd->length; i++)
        fprintf(fp, "%02X ", pdd->data[i]);
      if ((pdd + 1)->address != 0)
      {
        if (pdd->address + pdd->length < (pdd + 1)->address)
          fprintf(fp, "  (S)");
        else if (pdd->address + pdd->length > (pdd + 1)->address)
          fprintf(fp, "  (L)");
      }
    }
    else if (pdd->lost)
    {
      fprintf(fp, "LOST %02X BYTES    ", pdd->length);
      for (i = 0; i < PCODELEN; i++)
        fprintf(fp, "%02X ", pdd->data[i]);
    }
    else if (pdd->undo)
    {
      fprintf(fp, "UNKNOWN COMMAND  ");
      for (i = 0; i < PCODELEN; i++)
        fprintf(fp, "%02X ", pdd->data[i]);
    }
    fprintf(fp, "\n");
    pdd++;
  }
  fclose(fp);
}

void Dump()
{
  FILE *fp;
  PDD pdd;
  int i;
  fp = fopen("out.txt", "a");
  pdd = (PDD)OutData;
  fprintf(fp, "\n\n%08X:\n", pdd->address);
  while (pdd->address != 0)
  {
    if (pdd->done)
    {
      if ((pdd + 1)->address != 0)
      {
        for (i = 0; i < (pdd + 1)->address - pdd->address; i++)
          fprintf(fp, "%02X", pdd->data[i]);
      }
      else
      {
        for (i = 0; i < pdd->length; i++)
          fprintf(fp, "%02X", pdd->data[i]);
      }
    }
    if (pdd->lost)
    {
      for (i = 0; i < pdd->length; i++)
        fprintf(fp, "90");
    }
    pdd++;
  }
  fprintf(fp, "\n\n");
  fclose(fp);
}

void ParseStolen(FILE *fp, LPBYTE data)
{
  PSS pss;
  PS2 ps2;
  int i;
  DWORD data4;
  DWORD stolenbase;
```



```c
  stolenbase = 0x400000;  

  pss = (PSS)data;  

  ps2 = (PS2)(VMData + PCODENUM*PCODELEN + 4 + ((pss->data1)&0xFF)*0x0D);  

  for (i=0; i<pss->symbol1; i++, ps2++)  

  {  

    if (ps2->work == 0x00 || ps2->work == 0x02)  

      fprintf(fp, "SUB ");  

    else if (ps2->work == 0x01)  

      fprintf(fp, "ADD ");  

    else if (ps2->work == 0x03)  

      fprintf(fp, "MOV ");  

    else  

      fprintf(fp, "??? ");  

    if ( ((ps2->mark)&0x02) != 0)  

    {  

      if (ps2->type == 0)  

        fprintf(fp, "DWORD PTR[");  

      else if (ps2->type == 2)  

        fprintf(fp, "BYTE PTR[");  

      if (ps2->dst == 0)  

        fprintf(fp, "EAX");  

      else if (ps2->dst == 1)  

        fprintf(fp, "ECX");  

      else if (ps2->dst == 2)  

        fprintf(fp, "EDX");  

      else if (ps2->dst == 3)  

        fprintf(fp, "EBX");  

      else if (ps2->dst == 4)  

        fprintf(fp, "ESP");  

      else if (ps2->dst == 5)  

        fprintf(fp, "EBP");  

      else if (ps2->dst == 6)  

        fprintf(fp, "ESI");  

      else if (ps2->dst == 7)  

        fprintf(fp, "EDI");  

      if (((ps2->mark)&0x08) != 0)  

      {  

        data4 = ps2->dstplus;  

        if (((ps2->mark)&0x20) != 0)  

          data4 += stolenbase;  

        fprintf(fp, "+%09X", data4);  

      }  

      if (ps2->type == 0)  

        fprintf(fp, "]");  

      else if (ps2->type == 2)  

        fprintf(fp, "]");  

    }  

    else if ( ((ps2->mark)&0x02) == 0)  

    {  

      if (ps2->type == 0)  

        fprintf(fp, "DWORD PTR[");  

      else if (ps2->type == 2)  

        fprintf(fp, "BYTE PTR[");  

      if (((ps2->mark)&0x08) != 0)  

      {  

        data4 = ps2->dstplus;  

        if (((ps2->mark)&0x20) != 0)  

          data4 += stolenbase;  

        fprintf(fp, "%09X", data4);  

      }  

      if (ps2->type == 0)  

        fprintf(fp, "]");  

      else if (ps2->type == 2)  

        fprintf(fp, "]");  

    }  

    fprintf(fp, ", ");  

    if ( ((ps2->mark)&0x04) != 0)  

    {  

      if (ps2->type == 1)  

        fprintf(fp, "DWORD PTR[");  

      else if (ps2->type == 3)  

        fprintf(fp, "BYTE PTR[");  

      if (ps2->src == 0)  

        fprintf(fp, "EAX");  

      else if (ps2->src == 1)  

        fprintf(fp, "ECX");  

      else if (ps2->src == 2)  

        fprintf(fp, "EDX");  

      else if (ps2->src == 3)  

        fprintf(fp, "EBX");  

      else if (ps2->src == 4)  

        fprintf(fp, "ESP");  

      else if (ps2->src == 5)  

        fprintf(fp, "EBP");  

      else if (ps2->src == 6)  

        fprintf(fp, "ESI");  

      else if (ps2->src == 7)  

        fprintf(fp, "EDI");  

      if (((ps2->mark)&0x10) != 0)  

      {  

        data4 = ps2->srcplus;  

        if (((ps2->mark)&0x40) != 0)  

          data4 += stolenbase;  

        fprintf(fp, "+%09X", data4);  

      }  

      if (ps2->type == 1)  

        fprintf(fp, "]");  

      else if (ps2->type == 3)  

        fprintf(fp, "]");  

    }  

    else if ( ((ps2->mark)&0x04) == 0)  

    {  

      if (ps2->type == 1)  

        fprintf(fp, "DWORD PTR[");  

      else if (ps2->type == 3)  

        fprintf(fp, "BYTE PTR[");  

      if (((ps2->mark)&0x10) != 0)  

      {  

        data4 = ps2->srcplus;  

        if (((ps2->mark)&0x40) != 0)  

          data4 += stolenbase;  

        fprintf(fp, "%09X", data4);  

      }  

      if (ps2->type == 1)  

        fprintf(fp, "]");  

      else if (ps2->type == 3)  

        fprintf(fp, "]");  

    }  

    fprintf(fp, "\n");  

  }  

}  

void DumpStolen()  

{  

  FILE  *fp;  

  PDD    pdd;  

  fp = fopen("out.txt", "a");  

  pdd = (PDD)OutData;  

  while (pdd->address != 0)  

  {  

    if (pdd->lost)  

    {  

      fprintf(fp, "%08X:\n", pdd->address);  

      ParseStolen(fp, pdd->data);  

    }  

    pdd++;  

  }  

  fprintf(fp, "\n\n");  

  fclose(fp);  

}  

void main()  

{  

  ReadVMData();  

  ParseVM();  

  Check();  

  Dump();  

  DumpStolen();  

} 
```

附件下载：`fkvm aspr23beta0626.rar`

`4dd466d0e1e0cc123afbe504da2462ca`
