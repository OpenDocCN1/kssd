

# 去花的 idc

> 原文：[`www.pediy.com/kssd/pediy10/71723.html`](https://www.pediy.com/kssd/pediy10/71723.html)

无意看了一个 e 语言的东东， e 语言自带了很多花， 写个脚本在 ida 中删除  
使用很简单， 只需要把 main 函数的花串改掉就行。  
  patchspec("f8 73 01", 4, begin, end);  
第 1 个参数是花串， 第 2 个参数是 patch90 的长度， 后面是开始位置和结束位置  
  
// begin copy here  
// author jjnet  
// file: flowerpatch.idc  
  
static \_hexval(str)  
{  
  auto v;  
  v = ord(str);  
  if (0x30<=v && v<=0x39)  
    return v-0x30;  
  if ('a'<=v && v<='z')  
    return v-'a'+10;  
  if ('A'<=v && v<='Z')  
    return v-'A'+10;  
  return 0;  
}  
  
  
static hexval(str)  
{  
   return (\_hexval(str)<<4) + \_hexval(substr(str, 1, 2));  
}  
  
static format\_hexstr(str)  
{  
  auto lens, i, b, ret;  
  i = 0;  
  lens = strlen(str);  
  ret = "";  
    
  while(i+2 <= lens)  
  {  
    if (' ' ==   ord(substr(str, i, i+1)) )  
    {  
       i = i+1;  
       continue;  
     }  
    b = hexval(substr(str, i, i+2));  
    if ('\\\\' == b)  
    {  
       ret = ret + char('\\\\')+char('\\\\');  
    }  
    else if (0 == b)  
    {  
       ret = ret + char('\\\\')+char(1);  
    }  
    else  
    {  
       ret = ret + char(b);  
    }  
    i = i+2;  
  }  
  return ret;  
}  
  
static fmtstrlen(fmtstr)  
{  
  auto i, lens, ret;  
  lens = strlen(fmtstr);  
  i = 0;  
  ret = 0;  
  while(i<lens)  
  {  
     if ('\\\\' == (ord(substr(fmtstr, i, i+1))&0xff) )  
     {  
        i=i+2;  
     }  
     else  
     {  
        i=i+1;  
     }  
     ret = ret+1;  
  }  
  return ret;  
}  
  
  
  
static patchspec(hexstr, patchmany, begin, end)  
{  
  auto fmtstr, fmtstr\_len, i, j, count, c1, c2;  
  fmtstr = format\_hexstr(hexstr);  
  fmtstr\_len = fmtstrlen(fmtstr);  
  count = 0;  
  
  while(begin+fmtstr\_len<=end)  
  {  
    j = 0;  
    for(i=0; i<fmtstr\_len; ++i)  
    {  
      c1 = Byte(begin+i);  
      c2 = ord(substr(fmtstr,j,j+1))&0xff;  
      if ('\\\\' == c2)  
      {  
        j=j+1;  
        c2 = ord(substr(fmtstr,j,j+1))&0xff;  
        if (1 == c2) c2 = 0;  // escape charset  
      }  
      if (c1 != c2) break;  
      j = j+1;   
    }  
    if (i==fmtstr\_len) // found  
    {  
      for(i=0; i<patchmany; ++i)  
      {  
        MakeUnkn(begin+i,0);  
        PatchByte(begin+i,0x90);  
        MakeCode(begin+i);  
      }  
      Message("%x\\n", begin);  
      begin = begin+patchmany;  
      count = count+1;  
    }  
    else   // not found  
    {  
      begin = begin+1;  
    }  
  }  
  
  Message("total patched: %d\\n", count);  
}  
  
static main()  
{  
  auto begin, end;  
  begin = 0xd00000;  
  end = 0xe00000;  
  
  // e 语言带的  
  patchspec("f8 73 01", 4, begin, end);  
  patchspec("f9 72 01", 4, begin, end);  
  patchspec("e8 00 00 00 00 83 04 24 06 c3", 11, begin, end);  
  patchspec("E9 00 00 00 00", 5, begin, end);  
  patchspec("EB 01", 3, begin, end); //会不会误 patch？  
}