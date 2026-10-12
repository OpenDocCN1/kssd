
# 未知壳乱序重构

> 原文：[`www.pediy.com/kssd/pediy10/58361.html`](https://www.pediy.com/kssd/pediy10/58361.html)

代码:

```
很有趣:eek: 
/\*
Coder: dummyz@126.com
\*/

#include <Windows.h>
#include <fstream>
#include <memory>
#include <list>
#include <algorithm>
#include "xde.h"

using namespace std;

typedef unsigned char ubyte;
typedef unsigned short ushort;
typedef unsigned int uint;
typedef unsigned long ulong;

ubyte\* load\_pefile(const char\* fname, ulong& size)
{
  IMAGE\_DOS\_HEADER dosheader;
  IMAGE\_NT\_HEADERS ntheader;
  ifstream ifile(fname, ios::binary);

  if ( !ifile )
    return NULL;

  ifile.read((char\*)&dosheader, sizeof (IMAGE\_DOS\_HEADER));
  if ( dosheader.e\_magic != IMAGE\_DOS\_SIGNATURE )
    return NULL;

  ifile.seekg(dosheader.e\_lfanew, ios::beg);

  ifile.read((char\*)&ntheader, sizeof (IMAGE\_NT\_HEADERS));
  if ( ntheader.Signature != IMAGE\_NT\_SIGNATURE || 
    ntheader.FileHeader.NumberOfSections == 0 )
    return NULL;

  size = ifile.seekg(0, ios::end).tellg();
  ifile.seekg(0, ios::beg);

  auto\_ptr<ubyte> buf(new ubyte\[size\]);
  ifile.read((char\*)buf.get(), size);

  return buf.release();
}

typedef struct \_instr
{
  ubyte\* old\_ip;
  ubyte\* new\_ip;
  xde\_instr instr;
} instr\_t;

bool is\_hasip(list<instr\_t>& instr\_lst, ubyte\* ip)
{
  list<instr\_t>::iterator i = instr\_lst.begin();
  while ( i != instr\_lst.end() )
  {
    instr\_t& p = \*i++;
    if ( p.old\_ip == ip )
    {
      return true;
    }
  }

  return false;
}

int main(int argc, char\* argv\[\])
{
  if ( argc < 2 )
  {
    return -1;
  }
  
  ulong file\_size;
  auto\_ptr<ubyte> file\_buf(load\_pefile(argv\[1\], file\_size));
  ubyte\* base = file\_buf.get();
  if ( base == NULL )
  {
    return -2;
  }
  
  list<instr\_t> instr\_lst; // 指令表
  list<ubyte\*> ep\_list; // 入口表
  instr\_t instr;

  PIMAGE\_DOS\_HEADER dosheader = (PIMAGE\_DOS\_HEADER)base;
  PIMAGE\_NT\_HEADERS ntheader = (PIMAGE\_NT\_HEADERS)(base + dosheader->e\_lfanew);
  PIMAGE\_SECTION\_HEADER sectheader = IMAGE\_FIRST\_SECTION(ntheader);
  
  ulong code\_size = ntheader->OptionalHeader.SizeOfCode;
  ulong code\_va = ntheader->OptionalHeader.ImageBase + sectheader->VirtualAddress;
  ulong oep\_va = ntheader->OptionalHeader.ImageBase + ntheader->OptionalHeader.AddressOfEntryPoint;

  ubyte\* code = base + sectheader->VirtualAddress;

  instr.old\_ip = base + ntheader->OptionalHeader.AddressOfEntryPoint - \\
    sectheader->VirtualAddress + sectheader->PointerToRawData;
  
  auto\_ptr<byte> code\_buf(new byte\[code\_size \* 2\]);
  memset(code\_buf.get(), 0x90, code\_size \* 2);

  instr.new\_ip = code\_buf.get() + ntheader->OptionalHeader.AddressOfEntryPoint - \\
    sectheader->VirtualAddress;
  
  // @1 重构串行指令序
  ep\_list.push\_back(instr.old\_ip);
  while ( true )
  {
    list<ubyte\*>::iterator it;
    
    printf("\\r\\bPasering eip = %08X...", instr.old\_ip - code + code\_va);
    
    if (
      instr.old\_ip > code + code\_size ||
      is\_hasip(instr\_lst, instr.old\_ip) // 检查是否已经处理过了, 从入口表中取出新的地址
      )
    {
      if ( ep\_list.empty() )
        break;

      instr.old\_ip = ep\_list.front();
      ep\_list.erase(ep\_list.begin());
    }

    if ( 0 == xde\_disasm(instr.old\_ip, &instr.instr) )
    {
      printf("xde\_disasm error!\\n");
      break;
    }

    /\*
0040102C > $   C705 00104000>mov     dword ptr [401000], 00401333
00401036   .  FF25 00104000 jmp     dword ptr [401000]
    \*/
    if ( instr.instr.opcode == 0xc7 &&
      instr.instr.modrm == 0x05 &&
      (instr.instr.addr\_l\[0\] >= code\_va && instr.instr.addr\_l\[0\] < oep\_va) 
      )
    {
      ubyte\* next\_ep\_ip = instr.old\_ip + instr.instr.len;
      if ( \*(ushort\*)next\_ep\_ip == 0x25ff &&
        \*(ulong\*)(next\_ep\_ip + 2) == instr.instr.addr\_l\[0\] 
        )
      {
        next\_ep\_ip += 6;
        if ( ep\_list.end() == find(ep\_list.begin(), ep\_list.end(), next\_ep\_ip) && 
          !is\_hasip(instr\_lst, next\_ep\_ip) 
          )
        {
          ep\_list.push\_back(next\_ep\_ip);
        }
        
        instr.old\_ip = instr.instr.data\_l\[0\] - code\_va + code;
        continue;
      }
    }

    if ( instr.instr.flag & C\_REL )
    {
      ubyte\* next\_ep\_ip = instr.old\_ip + instr.instr.len;
      switch ( instr.instr.datasize )
      {
      case 1: next\_ep\_ip = (ubyte\*)((ulong)next\_ep\_ip + (long)instr.instr.data\_c\[0\]); break;
      case 4: next\_ep\_ip = (ubyte\*)((ulong)next\_ep\_ip + (long)instr.instr.data\_l\[0\]); break;
      default: printf("error datasize!\\n");   
      }

      if ( ep\_list.end() == find(ep\_list.begin(), ep\_list.end(), next\_ep\_ip) && 
        !is\_hasip(instr\_lst, next\_ep\_ip) 
        )
      {
        ep\_list.push\_back(next\_ep\_ip);
      }

      if ( instr.old\_ip\[0\] == 0x8a )
      {
        instr.old\_ip\[0\] = 0x8a;
      }
    }
    
    it = find(ep\_list.begin(), ep\_list.end(), instr.old\_ip);
    if ( ep\_list.end() != it ) // 从入口表中踢出
    {
      ep\_list.erase(it);
    }

    instr\_lst.push\_back(instr);
    instr.new\_ip += instr.instr.len;

    if ( instr.old\_ip\[0\] == 0xc3 ||
      instr.old\_ip\[0\] == 0xc2 
      )
    {
      if ( ep\_list.empty() )
        break;

      instr.old\_ip = ep\_list.front();
      ep\_list.erase(ep\_list.begin());
    }
    else
    {
      instr.old\_ip += instr.instr.len;
    }
  }
  
  printf("\\ninstr sum = %d\\n", instr\_lst.size());
  
  // @2 拷贝 & 校正偏移
  list<instr\_t>::iterator instr\_lst\_i = instr\_lst.begin();
  while ( instr\_lst\_i != instr\_lst.end() )
  {
    instr\_t& instr1 = \*instr\_lst\_i++;
    
    printf("\\r\\bRelocing eip = %08X...", instr1.new\_ip - code\_buf.get() + code\_va);

    if ( instr1.instr.flag & C\_REL )
    {
      ubyte\* jmp\_ip = instr1.old\_ip + instr1.instr.len;
      switch ( instr1.instr.datasize )
      {
      case 1: jmp\_ip = (ubyte\*)((ulong)jmp\_ip + (long)instr1.instr.data\_c\[0\]); break;
      case 4: jmp\_ip = (ubyte\*)((ulong)jmp\_ip + (long)instr1.instr.data\_l\[0\]); break;
      default: printf("error datasize!\\n");
      }
       
      do 
      {
        if ( jmp\_ip\[0\] == 0xc7 && jmp\_ip\[1\] == 0x05 &&
          \*(ulong\*)(jmp\_ip + 2) >= code\_va && \*(ulong\*)(jmp\_ip + 2) < oep\_va
          )
        {
          if ( \*(ushort\*)(jmp\_ip + 2 + 4 + 4) == 0x25ff &&
            \*(ulong\*)(jmp\_ip + 2 + 4 + 4 + 2) == \*(ulong\*)(jmp\_ip + 2) )
          {
            jmp\_ip = \*(ulong\*)(jmp\_ip + 2 + 4) - code\_va + code;
            continue;
          }
        }

        break;
      } while( true );
       
      list<instr\_t>::iterator instr\_lst\_j = instr\_lst.begin();
      while ( instr\_lst\_j != instr\_lst.end() )
      {
        instr\_t& instr2 = \*instr\_lst\_j;
        if ( instr2.old\_ip == jmp\_ip )
        {
          xde\_instr tmp = instr1.instr;

          tmp.data\_l\[0\] = instr2.new\_ip - instr1.new\_ip + instr1.instr.len;
          xde\_asm(instr1.new\_ip, &tmp); // 汇编
          break;
        }

        instr\_lst\_j++;
      }

      if ( instr\_lst\_j == instr\_lst.end() )
      {
        printf("reloc jmp error!\\n");
        break;
      }
    }
    else
    {
      memcpy(instr1.new\_ip, instr1.old\_ip, instr1.instr.len);
    }
  }
  
  ofstream ofile("1.bin", ios::binary);
  ofile.write((char\*)code\_buf.get(), code\_size);
  
  printf("\\nFinished!\\n");

  return 0;
}
```

上传的附件

| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg) || `fucker.rar`
|---|---|
| ![](img/c8c74a380a3dea7fa2460187d80ee5e0.jpg)`mbrrootkit.rar` | [样本是木马，慎运行，请到论坛下载：`bbs.pediy.com/showthread.php?t=58361`](http://bbs.pediy.com/showthread.php?t=58361)
